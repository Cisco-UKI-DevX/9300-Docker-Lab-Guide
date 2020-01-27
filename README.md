# Deploying Docker containers on Catalyst 9300/IOS-XE

As we’ve discussed in the past (link to guestshell lab). When we look at Network Automation we often need an environment where we can package and run our software and scripts that we build. One of the most features of the Catalyst 9300 switch is the ability to run standard docker containers on the device, The Catalyst 9000 switches have a x86 based CPU. One of the main drivers for including this was to allow for the running of applications on the switch. As of 16.12.1 the Catalyst 9300 supports a native docker engine allowing you to deploy docker applications straight onto the infrastructure.

Before we get started we'll need a test environment, one of the easiest test environments you'll find is on the Cisco DevNet Sandbox which has a dedicated sandbox for 9300 application hosting. This are completely free and can in some cases be accessed within seconds. https://developer.cisco.com/docs/sandbox/#!overview/all-networking-sandboxes

Please note you are free to use this with your own hardware or test environment. However the commands in this lab guide have been tested for the DevNet sandbox. 

If you're lucky enough to have a 9300 ready to run this please note: The 9300 requires a the 120GB USB external storage installed on the back of the device, thats why for this lab we will be utilising the DevNet 9300 sandbox

# Method 1 - Deploying via the CLI

# Method 3 - Deploying via the GUI


