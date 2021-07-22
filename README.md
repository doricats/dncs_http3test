# dncs_http3test
## Team
I membri del team sono Luigi Dorigatti e Giacomo Fedrigo

## Project
L'obiettivo del progetto è quello di costruire un framework virtualizzato con lo scopo di analizzare le prestazioni di HTTP/3 + QUIC, rispetto a HTTP/2 o TCP.

Software suggerito: Vagrant, OpenVSwitch, docker, o in alternativa mininet+docker (Comnetsemu) Software di riferimento: https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/

## Pre configurazione
Per compiere una valutazione delle prestazioni oggettiva e precisa, è necessario un laboratorio virtualizzato. Ovvero, nell'implementazione scelta, vengono utilizzati due software per impostare l'ambiente: Docker e Vagrant. Per replicare uno scenario realistico, il setup sarà il seguente:  ci sono 2 host usati come server e appartenenti alla stessa subnet (diversa da quella del client) collegati ad uno switch e quindi al router.Poi un host, usato come client è collegato direttamente all'unico router del laboratorio.  Il primo host sarà chiamato client, e sopra di esso verrà eseguito il software necessario per la valutazione delle prestazioni. D'altra parte, il secondo host sarà chiamato web-server ed eseguirà 3 diversi contenitori Docker e allo stesso modo l'ultimo sarà chiamato video-server ed eseguirà i restanti 3 contenitori.
Affinché la valutazione delle prestazioni sia realistica, dovrebbe includere sia i contenuti statici delle pagine web che lo streaming video,scelto come esempio principale. Da qui la necessità di 6 diversi contenitori Docker: (3 protocolli da testare) X (2 tipi di media). I contenitori potrebbero girare sullo stesso host, ma per far funzionare il contenitore HTTP/3+QUIC ha bisogno di usare le porte 80 e 443, e dato che ce ne sono 2, sarebbe un problema. Per questo motivo specifico i contenitori sono divisi in 2 host diversi per il web e per il video.

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

## Sviluppo
Per l'avvio, è necessario lanciare l'ambiente di laboratorio discusso in precedenza con il comando vagrant up. Le ultime immagini Docker compilate saranno scaricate dal Docker Hub e distribuite automaticamente.

Alcune note vanno fatte: per cambiare la configurazione delle porte utilizzate e i certificati SSL/TLS necessari, a seconda del metodo di distribuzione scelto, bisogna modificare docker/docker_deploy.sh o vagrant/docker_run.sh. Tutte le porte sono parametrizzate (tranne quelle HTTP/3, perché altrimenti non funzionerebbe), quindi la configurazione è abbastanza semplice.


## Video
Il protocollo di streaming scelto è stato HLS per la sua grande diffusione e per le sue prestazioni.  HLS può riprodurre video codificati con i codec H.264 o HEVC/H.265. L'immagine Docker per lo streaming video è solo una mod di quella creata nella sezione precedente. L'unica differenza è un plugin RTMP per NGINX e l'installazione di ffmpeg (usato per codificare il file video e metterlo in loop, simulando uno streaming live)

## Pagina web
L'immagine del web-server è costruita dal Dockerfile_TEXT che può essere trovato nella cartella docker. La distro Linux utilizzata come sottosistema è Ubuntu. Dopo aver installato tutte le dipendenze e NGINX, quest'ultimo viene patchato utilizzando la patch quiche di Cloudflare. Come riportato in precedenza, l'immagine di base è capace di HTTP/3, ma il web-server può funzionare su TCP, HTTP/2 o HTTP/3 come richiesto, utilizzando il file di configurazione, passato dall'opzione -v in Docker.


### Results
```console
vagrant@client:~$ httpstat https://web.bacci.dev:443
Connected to 192.168.2.2:443 from 192.168.1.2:60864

HTTP/2 200
server: nginx/1.16.1
content-type: text/html
content-length: 106200
etag: "60251560-19ed8"
alt-svc: h3-27=":443"; ma=86400
accept-ranges: bytes

Body stored in: /tmp/tmp1G1tZP

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[     4ms    |       2ms      |     17ms      |        2ms        |        8ms       ]
             |                |               |                   |                  |
    namelookup:4ms            |               |                   |                  |
                        connect:6ms           |                   |                  |
                                    pretransfer:23ms              |                  |
                                                      starttransfer:25ms             |
                                                                                 total:33ms
   
```

