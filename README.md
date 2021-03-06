# dncs_http3test
## Team
I membri del team sono Luigi Dorigatti e Giacomo Fedrigo

## Project
L'obiettivo del progetto è quello di costruire un framework virtualizzato con lo scopo di analizzare le prestazioni di HTTP/3 + QUIC, rispetto a HTTP/2 o TCP.

Software suggerito: Vagrant, OpenVSwitch, docker, o in alternativa mininet+docker (Comnetsemu) Software di riferimento: https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/

## Pre configurazione
Per compiere una valutazione delle prestazioni oggettiva e precisa, è necessario un laboratorio virtualizzato. Ovvero, nell'implementazione scelta, vengono utilizzati due software per impostare l'ambiente: Docker e Vagrant. Il setup sarà il seguente:  ci sono 2 host usati come server e appartenenti alla stessa subnet (diversa da quella del client) collegati ad uno switch e quindi al router.Poi un host, usato come client è collegato direttamente all'unico router del laboratorio.  Il primo host sarà chiamato client, e sopra di esso verrà eseguito il software necessario per la valutazione delle prestazioni. D'altra parte, il secondo host sarà chiamato web-server ed eseguirà 3 diversi contenitori Docker e allo stesso modo l'ultimo sarà chiamato video-server ed eseguirà i restanti 3 contenitori.
Abbiamo deciso di includere sia i contenuti statici delle pagine web che lo streaming video.Da qui la necessità di 6 diversi contenitori Docker: 3 protocolli da testare e 2 tipi di media. Pper far funzionare il contenitore HTTP/3+QUIC ha bisogno di usare le porte 80 e 443, e dato che ce ne sono 2, sarebbe un problema. Per questo motivo specifico i contenitori sono divisi in 2 host diversi.

Per quanto riguarda gli indirizzi IP assegnati da Vagrant ai diversi host, sono i seguenti: router::eth1 è 192.168.1.1, router::eth2 è 192.168.2.1, client::eth1 è 192.168.1.2 e web-server::eth1 è 192.168.2.2 e video-server::eth1 è 192.168.2.3.

Service	Protocol	IP address	Ports
Web page	TCP	192.168.2.2	82, 452
Web page	HTTP/2	192.168.2.2	81, 451
Web page	HTTP/3 + QUIC	192.168.2.2	80, 443
Video streaming	TCP	192.168.2.3	82, 452
Video streaming	HTTP/2	192.168.2.3	81, 451
Video streaming	HTTP/3 + QUIC	192.168.2.3	80, 443

## Vagrant 

Come mostrato in precedenza, Vagrant è usato per gestire la VM e il lato di rete dell'ambiente Lab. L'immagine utilizzata per il sistema operativo è ubuntu/bionic64. Il server X11 viene inoltrato al fine di utilizzare strumenti di valutazione delle prestazioni e browser dal client tramite X-server. Questo si ottiene aggiungendo nel Vagrantfile le seguenti linee:

  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true
  
Inoltre, sia il client che il server hanno 1024 MB di RAM per essere in grado di eseguire Google Chrome e ffmpeg . Tutti gli script di provisioning sono nella cartella vagrant e sono usati principalmente per il routing e l'installazione del software di base. Lo script di provisioning responsabile per il deployment di Docker (docker_run.sh) è anch'esso contenuto nella stessa cartella degli altri, ma sarà discusso più avanti. Ultimo ma non meno importante, è importante sapere che le immagini docker non saranno compilate ad ogni vagrant up, ma invece scaricate dal Docker Hub.

## Docker
L'approccio scelto è stato quello di costruire 2 diverse immagini Docker, distribuendo 6 contenitori: il primo serve allo scopo di eseguire un web-server, mentre l'ultimo è utilizzato per lo streaming video HLS. L'immagine per lo streaming video è una mod di quella per il web-server e sono entrambe basate sul server NGINX, più precisamente NGINX 1.16.1 . Infatti entrambe le immagini sono preconfigurate come HTTP/3, ma saranno limitate nel file di configurazione nginx.conf per funzionare su TCP, HTTP/2 e HTTP/3+QUIC . Importante notare che le immagini non verranno compilate ogni volta che il comando vagrant up sarà eseguito, ma saranno invece scaricate direttamente dal nostro Docker Hub (https://hub.docker.com/r/luigidoricats).

## Avvio

Per cambiare la configurazione delle porte utilizzate e i certificati SSL/TLS necessari, bisogna modificare docker/docker_deploy.sh o vagrant/docker_run.sh. Tutte le porte sono parametrizzate (tranne quelle HTTP/3, perché altrimenti non funzionerebbe), quindi la configurazione è abbastanza semplice.

Per l'avvio, è necessario lanciare l'ambiente di laboratorio discusso in precedenza con il comando vagrant up. Le ultime immagini Docker compilate saranno scaricate dal Docker Hub e distribuite automaticamente.



## Video e Pagina web
Il protocollo di streaming scelto è stato HLS. HLS può riprodurre video codificati con i codec H.264 o HEVC/H.265. L'immagine Docker per lo streaming video è solo una mod di quella creata nella sezione precedente. L'unica differenza è un plugin RTMP per NGINX e l'installazione di ffmpeg (usato per codificare il file video e metterlo in loop, simulando uno streaming live)


L'immagine del web-server è costruita dal Dockerfile_TEXT che può essere trovato nella cartella docker. La distro Linux utilizzata come sottosistema è Ubuntu. Dopo aver installato tutte le dipendenze e NGINX, quest'ultimo viene patchato utilizzando la patch quiche di Cloudflare. https://github.com/cloudflare/quiche . Il web-server può funzionare su TCP, HTTP/2 o HTTP/3 come richiesto, utilizzando il file di configurazione, passato dall'opzione -v in Docker.


## Risultati

Ci siamo avvalsi del comando httpstat per l'estrazione dei dati necessari ed una visualizzazione chiara e comprensiva dei risultati. 

### HTTP/3 + QUIC web page:
```console
vagrant@client:~$ httpstat https://web.doricats.dev:443
Connected to 192.168.2.2:443 from 192.168.1.2:60864

HTTP/2 200
server: nginx/1.16.1
content-type: text/html
content-length: 84200
etag: "60251560-19ed8"
alt-svc: h3-27=":443"; ma=86400
accept-ranges: bytes

Body stored in: /tmp/tmp1G1tZP

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[     4ms    |       2ms      |     17ms      |        2ms        |        9ms       ]
             |                |               |                   |                  |
    namelookup:4ms            |               |                   |                  |
                        connect:6ms           |                   |                  |
                                    pretransfer:23ms              |                  |
                                                      starttransfer:25ms             |
                                                                                 total:34ms
   
```
### HTTP/2 + SSL web page:
```console
vagrant@client:~$ httpstat https://web.doricats.dev:451
Connected to 192.168.2.2:451 from 192.168.1.2:43398

HTTP/2 200
server: nginx/1.16.1
content-type: text/html
content-length: 84200
etag: "60251560-19ed8"
accept-ranges: bytes

Body stored in: /tmp/tmp7A0ND6

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[     4ms    |       1ms      |     14ms      |        3ms        |        7ms       ]
             |                |               |                   |                  |
    namelookup:4ms            |               |                   |                  |
                        connect:5ms           |                   |                  |
                                    pretransfer:19ms              |                  |
                                                      starttransfer:22ms             |
                                                                                 total:29ms

```
### TCP + SSL web page:
```console
vagrant@client:~$ httpstat https://web.doricats.dev:452
Connected to 192.168.2.2:452 from 192.168.1.2:32988

HTTP/1.1 200 OK
Server: nginx/1.16.1
Content-Type: text/html
Content-Length: 84200
Connection: keep-alive
ETag: "60251560-19ed8"
Accept-Ranges: bytes

Body stored in: /tmp/tmphG9UJe

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[     4ms    |       2ms      |     14ms      |        3ms        |        4ms       ]
             |                |               |                   |                  |
    namelookup:4ms            |               |                   |                  |
                        connect:6ms           |                   |                  |
                                    pretransfer:20ms              |                  |
                                                      starttransfer:23ms             |
                                                                                 total:27ms

```

Un riassunto dei dati raccolti con l'uso del comando httpstat.
| Protocol      | Page weight | TTFB      | Load time | # requests | # tcp connections |
| ------------- | ----------- | --------- | --------- | ---------- | ----------------- |
| HTTP/3 + QUIC | 9.96 MB      | 5.51 msec | 3.96 sec  | 140        | 0                 |
| HTTP/2        | 9.96 MB      | 5.63 msec | 2.57 sec  | 129        | 1                 |
| TCP           | 9.96 MB      | 4.64 msec | 2.49 sec  | 125        | 6                 |


Un riassunto dei dati raccolti con l'uso di https://hls-js.netlify.app/demo/ e https://demo.theoplayer.com/test-your-stream-with-statistics per lo streaming dei contenuti video.
|  Protocol Startup | time      | Avg latency | Avg bitrate | Dropped Frames |
|-------------------|-----------|-------------|-------------|----------------|
| HTTP/3 + QUIC     | 1356 msec | 48,35 msec  | 64,17 MB/s  | 352            |
| HTTP/2            | 1094 msec | 28,32 msec  | 139,57 MB/s | 302            |
| TCP               | 876 msec  | 25,78 msec  | 57,44 MB/s  | 705            |


## Conclusions
Dai risultati ottenuti nello streaming del video e nel caricamento dell pagina web, si evince chiaramente che HTTP/3 è più lento di HTTP/2.
