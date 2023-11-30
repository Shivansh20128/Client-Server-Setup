# Next steps
The setup needs to establish a connection between the client, the shaper, and the server. All these services are running in separate docker containers of their own. 
The containers need an orchestration tool like docker swarm or Kubernetes to run from a single point. For our use case, we have chosen the docker swarm (for no particular reason).
Docker swarm allows multiple containers to work together. To use a docker swarm, we first need to initialize a docker swarm at a node. This node becomes our **leader node**. The command to do that is:
```
docker swarm init
```
From my experience, please add only Linux machines into your docker swarm as you may face problems if you add a Windows system into your docker swarm. In Windows, docker desktop is the running daemon for docker,
which is sometimes sleeping for Windows performance optimization, and hence sometimes exits or does not join the swarm.

This will return a command on the terminal for other nodes to join the swarm. Copy that command and run it in the node you want to add to your swarm. Note that the nodes should be a part of the same network, 
otherwise, the joining node will not be able to discover the swarm. After you have joined the docker swarm, enter the command ```docker node ls``` on the node where you initialized the swarm. 
You will be able to see a list of the nodes, including the leader node, that are a part of the docker swarm. You will also see that this system would be shown as a leader node, and the other nodes are worker nodes.
You will also see the ID of the nodes. We will use these IDs in the next steps.

Now that we have a docker swarm, we want to run containers on these nodes from our leader node. This can be done using a docker-compose file as shown below:
```
version: '3.8'

services:
  sim:
    image: alpine
    command: ["sleep", "infinity"]
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == worker2
    networks:
      - my_network

  client:
    image: alpine
    command: ["sleep", "infinity"]
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == worker2
    networks:
      - my_network

  server:
    image: alpine
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == worker1
    networks:
      - my_network

networks:
  my_network:
    driver: overlay

```

Do not try to remove one worker node from here and run that container on the leader node itself. That may give you errors in the future as the leader node is generally used for orchestration purposes
 and it is not advisable to run workloads on the leader node.
This docker-compose file is a basic file that uses an Alpine image. Notice that it places the shaper and the client on worker node 1 and the server on worker node 2. Thus, before running this file, we need to 
label our nodes as worker1 and worker2. You can do this using the command-
```
docker node update --label-add type=worker1 <ID for node 1>
docker node update --label-add type=worker2 <ID for node 2>
```

Now, we can run our containers on these nodes. We could have used ```docker compose up``` as used by vegvisir to run the containers. But that only works when all the containers reside on a single node. Since we
 have two different nodes, we need to use **docker stack**. 
```
docker stack deploy -c docker-compose.yml my_stack
```
Run the above command, and a service stack should be made running the three containers on the two nodes.

Now we have 3 containers running on 2 nodes with the Alpine docker image. We need to change this into the docker images used by vegvisir. These images can be found in this repo. The dockerfile for the client
resides in **cross-layer-implementation/**, and the dockerfile for the shaper resides in **tc-netem-shaper/**. You can build images for these dockerfiles by using the **docker run** command. 

To build the image for the client, go into the **cross-layer-implementation** folder, and run:
```
docker build -t godashcl .
```

To build the image for the shaper, go into the **tc-netem-shaper** folder, and run:
```
docker build -t tc-netem-cl .
```
You can check the formed images in your system by the command ```docker images```. Please note that you need to build docker images inside the worker nodes where you are going to run these images. For example, 
if you want to run the client on worker node1, then build the docker image on node1. There is no need to build these images on the other nodes and the leader node. 

For the server image, you can find the image in the folder **quic-go-interop**. In vegvisir, the docker image for the server is pulled directly from the docker hub, but since we may need to make some changes to these images
, I have added the source for those images here. For the server file, I want you to see the dockerfile first before building it. In the first line, it says **FROM quic-network-sim:latest**. It means it is building
 the server image upon a different image referred to here. So now, we need to see what is happening inside this referred image. So navigate to the **quic-network-simulator/endpoint** folder and see the dockerfile. 
 The dockerfile copies some content from the system and the web and then gives an entry point **run_endpoint.sh**. If we see that bash file, we will see a **/setup.sh** file called first, which is responsible for
 setting up routes for the simulation. This is the file that I believe is causing a problem, as this routing does not happen successfully. 

 If I comment out the setup.sh line from the entry-point file, the container runs and exits when it is done. 

 If you want to run and try testing the setup, all you need to do is build the images for server. First, navigate inside the **quic-network-simulator/endpoint** folder and run:
 ```
docker build -t quic-network-sim .
```
Then navigate inside the **quic-go-interop** folder, and run:
```
docker build -t server .
```
Please check the build images with ```docker images```. Again, build these images on the node where you need to run the server.

Now you can deploy the docker stack after changing the image names in the docker-compose file. You docker-compose.yml file should be like:
```
version: '3.8'

services:
  sim:
    image: tc-netem-cl
    command: ["sleep", "infinity"]
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == worker2
    networks:
      - my_network

  client:
    image: godashcl
    command: ["sleep", "infinity"]
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == worker2
    networks:
      - my_network

  server:
    image: server
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == worker1
    networks:
      - my_network

networks:
  my_network:
    driver: overlay
```

Happy debugging!
