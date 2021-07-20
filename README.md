# dncs_http3test
## Team
I membri del team sono Luigi Dorigatti e Giacomo Fedrigo

## Project
L'obiettivo del progetto è quello di costruire un framework virtualizzato per analizzare le prestazioni di HTTP/3 + QUIC, rispetto a HTTP/2 o TCP.

Software suggerito: Vagrant, OpenVSwitch, docker, o in alternativa mininet+docker (Comnetsemu) Software di riferimento: https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/

## Lab Configuration
Per essere il più imparziale possibile, e anche per rendere la valutazione delle prestazioni replicabile da tutti, è necessario un laboratorio virtualizzato. Più specificamente, nell'implementazione scelta, vengono utilizzati due software per impostare l'ambiente: Docker e Vagrant. Per replicare uno scenario realistico, il setup sarà il seguente: un host, usato come client è collegato direttamente all'unico router del laboratorio. Poi ci sono 2 host usati come server e appartenenti alla stessa subnet (diversa da quella del client) collegati ad uno switch e quindi al router. Il primo host sarà chiamato client, e sopra di esso verrà eseguito il software necessario per la valutazione delle prestazioni (che sarà discusso in una sezione dedicata). D'altra parte, il secondo host sarà chiamato web-server ed eseguirà 3 diversi contenitori Docker e allo stesso modo l'ultimo sarà chiamato video-server ed eseguirà i restanti 3 contenitori.
Affinché la valutazione delle prestazioni sia verosimilmente realistica, dovrebbe includere sia i contenuti statici delle pagine web che lo streaming video (il mezzo più popolare al giorno d'oggi). Da qui la necessità di 6 diversi contenitori Docker: (3 protocolli da testare) X (2 tipi di media). I contenitori potrebbero girare sullo stesso host, ma per far funzionare il contenitore HTTP/3+QUIC ha bisogno di usare le porte 80 e 443, e dato che ce ne sono 2, sarebbe un problema. Per questo motivo specifico i contenitori sono divisi in 2 host diversi (uno dedicato ai contenuti statici del web, e l'altro dedicato allo streaming video).

Service	Protocol	IP address	Ports
Web page	TCP	192.168.2.2	82, 452
Web page	HTTP/2	192.168.2.2	81, 451
Web page	HTTP/3 + QUIC	192.168.2.2	80, 443
Video streaming	TCP	192.168.2.3	82, 452
Video streaming	HTTP/2	192.168.2.3	81, 451
Video streaming	HTTP/3 + QUIC	192.168.2.3	80, 443

Per quanto riguarda gli indirizzi IP assegnati da Vagrant ai diversi host, sono i seguenti: router::eth1 è 192.168.1.1, router::eth2 è 192.168.2.1, client::eth1 è 192.168.1.2 e web-server::eth1 è 192.168.2.2 e video-server::eth1 è 192.168.2.3.



### Results
https://hub.docker.com/r/luigidoricats
