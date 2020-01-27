*This lab is part of a series of guides from the [Network Automation and Tooling workshop series](https://github.com/sttrayno/Network-Automation-Tooling)*

# Deploying Docker containers on Catalyst 9300/IOS-XE

As we’ve discussed in the [past](https://github.com/sttrayno/Guestshell-Lab-Guide) when we dive into the world network automation we often need an environment where we can package and run our software and scripts that we build. One of the most features of the Catalyst 9300 switch is the ability to run standard docker containers on the device.

The Catalyst 9000 switches have a x86 based CPU. One of the main drivers for including this was to allow for the running of applications on the switch. As of 16.12.1 the Catalyst 9300 supports a native docker engine allowing you to deploy docker applications straight onto the infrastructure which I plan on guiding you through the process of getting started with in this lab.

It should be noted that usecases for this feature aren’t just limited to network automation and include IOT, Security and performance monitoring.

## Prerequiesites

Before we get started we'll need a test environment, one of the easiest test environments you'll find is on the Cisco DevNet Sandbox which has a dedicated sandbox for 9300 application hosting. This are completely free and can in some cases be accessed within seconds. https://developer.cisco.com/docs/sandbox/#!overview/all-networking-sandboxes

Please note you are free to use this with your own hardware or test environment. However the commands in this lab guide have been tested for the DevNet sandbox. 

If you're lucky enough to have a 9300 ready to run this please note: The device requires a the 120GB USB external storage installed on the back of the device, thats why for this lab we will be utilising the DevNet 9300 sandbox

In this guide we have 2 methods of running the containers, with the CLI or through the GUI. For completeness we'll cover both in this guide, your welcome to just do one (GUI is by far the easiest).

Within this guide we'll be focusing on how to run iperf3 to carry out very basic testing on the network. The container we're going to run can be found [here](https://hub.docker.com/r/mlabbe/iperf3)

### Step 1 - Packaging and transferring the Docker container to the device

First thing we need to do is get our docker container onto the device that we're going to deploy on. To do this we need to have at least a basic understanding of docker and docker containers. A great overview can be found in the docker documentation [here](https://docs.docker.com/engine/docker-overview/). In the context of the rest of this guide and if you're new to containers just think of them as a way in which we can build, ship, and run applications (dependancies and all).

Couple of brief things to cover that are important here.
 * Every docker container has a dockerfile which looks like the below to describe how the container should behave when deployed. For example see our iperfv3 containers dockerfile below.
 * Docker containers can be made available on Docker hub which acts like a library where anyone can [publish their container](https://hub.docker.com/r/mlabbe/iperf3).
 
 ```
 FROM alpine:latest
#FROM alpine:3.11.2

MAINTAINER Michel Labbe

# build intial apk binary cache and install iperf3
RUN apk add --no-cache iperf3 \
    && adduser -S iperf

USER iperf
    
# Expose the default iperf3 server ports
EXPOSE 5201/tcp 5201/udp

# entrypoint allows you to pass your arguments to the container at runtime
# very similar to a binary you would run. For example, in the following
# docker run -it <IMAGE> --help' is like running 'iperf --help'
ENTRYPOINT ["iperf3"]

# iperf3 -s = run in Server mode
CMD ["-s"]
```

Now we have a basic understanding of what we're going to deploy let's get started. To deploy our container on the Cat9K we need to build our dockerfile as a .tar package and transfer it over to the device. If you're using the sandbox like me this can be a little tricky as we dont have internet access to build our container and it's not a straight forward process of transfering a file via TFTP from your host. Luckily enough the sandbox has an image of iPerfV3 on the flash: already so we'll use that, but if you're doing this on your own box here's the process.

First off we must pull down the container we want to deploy from the docker hub. Remember you must have docker installed on your machine, for further details see the docker (documentation)[https://docs.docker.com/install/]

```
docker pull mlabbe/iperf3
```

Next we want to build a .tar package from the image we've pulled from docker hub, this can be done with the command:

```
docker save mlabbe/iperf3:latest -o iperf3.tar
```

As you can see from the below graphic, we then have our iperf3.tar file in our working directory ready for deployment.


![image](https://github.com/sttrayno/9300-Docker-Lab-Guide/blob/master/images/docker-image.gif?raw=true)

Alternatively, if you have a dockerfile like the one above you can build the image by using the command while in the same working directory as the file which must be named `Dockerfile`

```
docker build -t iperf3:1:0 .
```

You can then build the .tar package with the same save argument

```
docker save mlabbe/iperf3:latest -o iperf3.tar
```

![image](https://github.com/sttrayno/9300-Docker-Lab-Guide/blob/master/images/dockerfile.gif?raw=true)

Once you have your package you can transfer it to the device using TFTP, USB or any other method thats supported. Just ensure the .tar file is in the flash: directory as we'll need it later.

## Method 1 - Deploying via the CLI

Next up we need to do is configure our app-hosting parameters for iperf and assign a static IP address address/default gateway to the app.

Also take note of the runtime parameter required for the Docker container is configured under app-resource in run-opts which we need to configure here.

```
cat9k(config)#app-hosting appid iperf     
cat9k(config-app-hosting)# app-vnic AppGigEthernet vlan-access
cat9k(config-config-app-hosting-vlan-access)#  vlan 4000 guest-interface 0
cat9k(config-config-app-hosting-vlan-access-ip)# guest-ip address 10.10.20.101 netmask 255.255.255.0
cat9k(config-config-app-hosting-vlan-access-ip)# app-default-gateway 10.10.20.254 guest-interface 0
cat9k(config-app-hosting)# app-resource docker
cat9k(config-app-hosting-docker)#  run-opts "--restart=unless-stopped -p 5201:5201/tcp -p 5201:5201/udp"
```


![image](https://github.com/sttrayno/9300-Docker-Lab-Guide/blob/master/images/app-hosting-config.gif?raw=true)

Now we've configured our interfaces and app profile parameters its time to install the container, to do this simply runt the below command, as I mentioned earlier in the sandbox its tricky to transfer across our .tar package but convieniently a iperf3 package named 'iperf3nick.tar' can be used in place. If your using your own environment simply replace the path below with you own.

```
app-hosting install appid iperf package flash:iperf3nick.tar

```

Now that's done lets activate our iperf app

```
app-hosting activate appid iperf
```

Activating doesn't actually start the app so we need to do that too, give the command a couple of minutes to run.

```
app-hosting start appid iperf
```

When that's complete validate that our app is running with the below show command.

```
   show app-hosting list
   App id                                   State
   ---------------------------------------------------------
   iperf                                 RUNNING
```

Congratulations, your app has now been deployed. Keep reference these additional commands which can be used to stop, deactivate and uninstall our applcation as needed. You may now proceed to the 'Testing our application section'

```
   app-hosting stop appid iperf
   app-hosting deactivate appid iperf
   app-hosting uninstall appid iperf
```

## Method 2 - Deploying via the GUI


## Testing our application

## Final thoughts 

For now our application isn't very interesting that we're running and provides limited insight, but if you're using the sandbox they're are a few more apps already so I'd recommend deploying a couple to get yourself used to the process and trying out some of the different options. At somepoint we'll expand these exercises by building our own custom application packaged in a docker container, then look to deploy it in this hosting environment but for now I hope this lab has given you a brief taster of working with the Docker environment in the Catalyst 9300.
