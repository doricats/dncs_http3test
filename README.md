# dncs_http3test
## Team
I membri del team sono Luigi Dorigatti e Giacomo Fedrigo

## Project
L'obiettivo del progetto è quello di costruire un framework virtualizzato per analizzare le prestazioni di HTTP/3 + QUIC, rispetto a HTTP/2 o TCP.

Software suggerito: Vagrant, OpenVSwitch, docker, o in alternativa mininet+docker (Comnetsemu) Software di riferimento: https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/

## Lab Environment 🌍
Per essere il più imparziale possibile, e anche per rendere la valutazione delle prestazioni replicabile da tutti, è necessario un laboratorio virtualizzato. Più specificamente, nell'implementazione scelta, vengono utilizzati due software per impostare l'ambiente: Docker e Vagrant. Per replicare uno scenario realistico, il setup sarà il seguente: un host, usato come client è collegato direttamente all'unico router del laboratorio. Poi ci sono 2 host usati come server e appartenenti alla stessa subnet (diversa da quella del client) collegati ad uno switch e quindi al router. Il primo host sarà chiamato client, e sopra di esso verrà eseguito il software necessario per la valutazione delle prestazioni (che sarà discusso in una sezione dedicata). D'altra parte, il secondo host sarà chiamato web-server ed eseguirà 3 diversi contenitori Docker e allo stesso modo l'ultimo sarà chiamato video-server ed eseguirà i restanti 3 contenitori.


## Lab Configuration

### Results
https://hub.docker.com/r/luigidoricats
