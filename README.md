# Client - Server Setup 
 This repository contains the process and steps I took to make a client server setup.

 The Client-Server setup intended must have the following properties.

 1. The Client and Server should be on a different hosts. For our case, since we are going to use Vegvisir tool, we also have a network shaper. Thus, the server and the shaper can be together on one host and the client can be an another host.

 2. Since the vegvisir tool uses docker containers for each of the client, the server and the network shaper, we are also going to use docker containers on our hosts.

 3. The client will have goDASH running on it.

 4. The motive behind this setup is to get transport layer logs at the application layer and then observe network parameters like congestion window, etc.

![Alt text](https://github.com/Shivansh20128/Client-Server-Setup/blob/7791c4c565b21b03a2889996daf7c692a49fe648/setup.png)
# Important links

Modified Vegvisir link: https://github.com/Shivansh20128/vegvisir

Modified Cross-layer-metrics-repo: https://github.com/Shivansh20128/cross-that-boundary-mmsys23-nossdav

 
 # Using Docker Containers

 In this setup, we want to use docker containers at the client, server and network shaper on different hosts. Since in the original setup, all the three containers are running on the same machine, we need to make some changes.

 So the first thing we are going to do is understand how we can make a connection between containers when they are running on different hosts. For this purpose, we must first go into how networking happens in docker containers.

 # Networking in docker containers

 Networking in Docker containers is crucial for communication between containers and with external systems. Docker offers various network modes, including bridge, host, and overlay networks. Bridge networks enable containers to communicate within a host, while overlay networks facilitate communication between containers on different hosts. Docker's networking features make it a powerful tool for building distributed and scalable applications.

 More information about networking docker can be found https://docs.docker.com/network/

 Since we are going to be running our containers on different hosts, we are going to use "Overlay Network".
 For that I am following this documentation: https://docs.docker.com/network/network-tutorial-overlay/#use-an-overlay-network-for-standalone-containers

An overlay network can span across hosts and containers can join this network using docker swarm. But I was unable to establish a connection between my laptop and a system in the general computing labs. I was able to make a docker swarm using "docker swarm init" command, which gave me a command " docker join" for other systems to join, but the 2nd container was unable to join the overlay network, as it could not discover it, and gave the error "docker: Error response from daemon: attaching to network failed, make sure your network options are correct and check manager logs: context deadline exceeded."


Then when I talked to a Professor, they said that it might be due to firewall restrictions at the campus gateway, and recommended using LAN cables directly to connect the two systems and make a network of their own, and then make the containers. I did that. A private network between the two hosts was established such that I was able to ping each system from the other system. But then when I made the overly network, I was again unable to connect them.


This week, I came home (having a home WIFI), and I tried doing the same thing with my 2 laptops, one windows 11 and other ubuntu 22. I was still getting the same error. I connected them with LAN cable and then tried doing it, still nothing.
Then I thought that maybe the OS on the two hosts should be the same. So I tried connecting my ubuntu laptop to other ubuntu machine inside college (by doing ssh into the machine from my home). Again, I could not connect them.

So then I thought that since my laptop is in a separate network, it might be causing some issues. So I tried connecting two ubuntu machines ,  both inside college through ssh. This time I was able to connect them. The machines I am talking about are xeon and ryzen02, and both of these machines belong to the Professor, and are highly used by students. So I thought of doing a similar thing with our machine, having IP 192.168.1.152. But for some reason, I was unable to ssh it.

Apart from this, I also thought of trying the same thing between two VMs running on the same host. So I made two Ubuntu20 VMs on my laptop and ran containers on each of them. And I was able to connect them. But I am not sure if this fits our usecase.

UPDATE:
I got two Ubuntu machines, both inside the lab. They share the same LAN network. 
I was able to establish a connection between them. So now, I am done with the first step, which was to connect containers on two hosts. Now, I have to use the three containers and establish a connection between them, where the server and the shaper will be on one host, and the client will be on a separate host.
