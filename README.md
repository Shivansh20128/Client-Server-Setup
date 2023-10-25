# Client - Server Setup 
 This repository contains the process and steps I took to make a client server setup.

 The Client-Server setup intended must have the following properties.

 1. The Client and Server should be on a different hosts. For our case, since we are going to use Vegvisir tool, we also have a network shaper. Thus, the server and the shaper can be together on one host and the client can be an another host.

 2. Since the vegvisir tool uses docker containers for each of the client, the server and the network shaper, we are also going to use docker containers on our hosts.

 3. The client will have goDASH running on it.

 4. The motive behind this setup is to get transport layer logs at the application layer and then observe network parameters like congestion window, etc.

 
 # Using Docker Containers

 In this setup, we want to use docker containers at the client, server and network shaper on different hosts. Since in the original setup, all the three containers are running on the same machine, we need to make some changes.

 So the first thing we are going to do is understand how we can make a connection between containers when they are running on different hosts. For this purpose, we must first go into how networking happens in docker containers.
 