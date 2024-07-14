## Docker Networking
This is a setup of small notes in terms of making docker networks. 
- The are 7 Network can be deployed with docker containers. 


#### Default Bridge Network
```sh ip address show

sudo apt update 
sudo apt install docker.io -y 

ip address show #- shows the ip information of the host machine

sudo docker network ls #- list docker network     

sudo run -itd --rm --name [name of the Container] [name container image]

sudo bridge link #- check the bridge links	
sudo docker ps  #- shows all the docker listed and running 

sudo docker inspect [network name]
sudo docker exec -it [name of the docker] [command to run]
sudo docker exec -it plmapp sh 

sudo docker stop [name of the container] #- this is to stop the docker container

#Running a Docker container to a mapped port from the host
sudo docker run -itd --rm -p 8001:80 --name [name of the Container] [Container Image Name]
```

#### User Define Bridge 
- Has DNS support for using the docker names that is part of this network 

```sh
sudo docker network create [network name to create]

sudo docker network ls #- list the network

#This addes a Container to a network that was created
sudo docker run -itd --rm --network [Name of the Network] --name [Name Container] [Name Image]

```

#### The Host Network 
- A Host network will place close to the host , with all features of the host.
- the Container runs as a Application

```sh
sudo docker run -itd --rm --network host --name [Name Container] [Name Image]
```

#### The MacVLAN network 
- Connecting Docker container directly to a local network
- they have there own ip address from the network

```sh 
sudo docker network create -d macvlan --subnet [Subnet to use] \
 --gateway [LAN Gateway] -o parent=[host NetInterface] \
 [Name of the Network]

# adding containers to the MacVlan Network
sudo docker run -itd --rm --network [the Network that was created] \
--ip [Ip to assign to the Container] \
--name [name of the container] [Name of Image]

```

