# docker-nomad

##Nomad Agent in Docker
This project is a Docker container for [Nomad](https://www.nomadproject.io) that is pre-configured Nomad Agent made specifically to work in the Docker ecosystem.

##Getting the container
The container is very small and based on Alpine base image.
```
docker pull aatarasoff/nomad           //base image
docker pull aatarasoff/nomad-server    //server pre-configured image
docker pull aatarasoff/nomad-client    //client pre-configured image
```

##Using the container

###Standalone
```
docker run -d --net host --name nomad \
  -p 4646:4646 -p 4646:4646/udp \
  -p 4647:4647 -p 4647:4647/udp \
  -p 4648:4648 -p 4648:4648/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aatarasoff/nomad -dev -bind=<your_binding_ip_address>
```
Standalone configuration is recommended for developing only.

###Clustering
First you need to run three server nodes on different machines. Each node is running like:
```
docker run -d --net host --name nomad \
  -p 4646:4646 -p 4646:4646/udp \
  -p 4647:4647 -p 4647:4647/udp \
  -p 4648:4648 -p 4648:4648/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aatarasoff/nomad-server -bind=<your_binding_ip_address>
```
Next step is joining nodes number two and three to node number one:
```
docker exec nomad nomad server-join -address http://<node2_ip>:4646 <node1_ip>:4648
docker exec nomad nomad server-join -address http://<node3_ip>:4646 <node1_ip>:4648
```
Or you may use [HTTP API](https://www.nomadproject.io/docs/http/index.html) for this.
Next step is launch clients:
```
docker run -d --net host --name nomad \
  -p 4646:4646 -p 4646:4646/udp \
  -p 4647:4647 -p 4647:4647/udp \
  -p 4648:4648 -p 4648:4648/udp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aatarasoff/nomad-client -bind=<your_binding_ip_address> -servers http://<node1_ip>:4647
```
Check that client nodes is available:
```
curl <node1_ip>/v1/nodes
```

###Run example task
Try your configuration with example task on one of the server nodes:
```
docker exec nomad nomad init
docker exec nomad nomad run -address=http://<node_ip>:4646 example.nomad
```
Check that redis container is launched on client nodes.

##Enjoy!
