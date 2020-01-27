# Deploying Docker containers on Catalyst 9300/IOS-XE

As we’ve discussed in the [past](https://github.com/sttrayno/Guestshell-Lab-Guide) when we're diving into network automation we often need an environment where we can package and run our software and scripts that we build. One of the most features of the Catalyst 9300 switch is the ability to run standard docker containers on the device.

The Catalyst 9000 switches have a x86 based CPU. One of the main drivers for including this was to allow for the running of applications on the switch. As of 16.12.1 the Catalyst 9300 supports a native docker engine allowing you to deploy docker applications straight onto the infrastructure which I plan on guiding you through the process of getting started with in this lab.

It should be noted that usecases for this feature aren’t just limited to network automation and include IOT, Security and performance monitoring.

## Prerequesites

Before we get started we'll need a test environment, one of the easiest test environments you'll find is on the Cisco DevNet Sandbox which has a dedicated sandbox for 9300 application hosting. This are completely free and can in some cases be accessed within seconds. https://developer.cisco.com/docs/sandbox/#!overview/all-networking-sandboxes

Please note you are free to use this with your own hardware or test environment. However the commands in this lab guide have been tested for the DevNet sandbox. 

If you're lucky enough to have a 9300 ready to run this please note: The device requires a the 120GB USB external storage installed on the back of the device, thats why for this lab we will be utilising the DevNet 9300 sandbox

## Method 1 - Deploying via the CLI

### Step 1 - Packaging and transferring the Docker container to the device



## Method 2 - Deploying via the GUI

For now our application isn't very interesting that we're running, they're are a few on the box already so I'd recommend deploying a couple to get yourself used to the process and trying out some of the different options. At somepoint we'll expand these exercises by building our own custom application packaged in a docker container, then look to deploy it in this hosting environement but for now I hope this lab has given you a brief taster of working with the Docker environment in the Catalyst 9300.
