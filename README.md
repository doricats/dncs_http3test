# dncs_http3test
## Team
Team members are Baccichet Giovanni (202869) and Parpinello Davide (201494).

## Project
The goal of the project is to build a virtualized framework for analyzing the performance of HTTP/3 + QUIC, with respect to HTTP/2 or TCP.

Suggested software: Vagrant, OpenVSwitch, docker, or alternatively mininet+docker (Comnetsemu) Reference software: https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/

## Lab Environment 🌍
In order to be as unbiased as possibile, and also to make the performance evaluation replicable by everyone, it is necessary a virtualized lab. More specifically, in the implementation chosen, two softwares are used to set up the environment: Docker and Vagrant. In order to replicate a realistic scenario, the setup will be the following: one host, used as a client is connected directly to the only router of the lab. Than there are 2 hosts used as servers and belonging to the same subnet (different from the client one) connected to a switch and so to the router. The fist host will be named client, and on top of it will run the software needed for the performance evaluation (that will be discussed in a dedicated section). On the other hand, the second host will be called web-server and will run 3 different Docker containers and similarly the last one will be called video-server and it will run the remaining 3 containers.


## Lab Configuration

### Results
https://hub.docker.com/r/luigidoricats
