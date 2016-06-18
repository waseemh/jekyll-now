---
layout: post
title: "Virtualized Web Automation Stack: Vagrant + Ubuntu + LXC + Headless FF/Chrome + Selenium Grid"
comments: true
permalink: virtualized-web-automation-stack
---

Creating a robust and scalable execution environment for automation tests is a very essential phase. After tests are written, they need to be executed on frequent basis over multiple environments. In automated tests for web, there is an additional need to cover different browsers and platforms. 
In this tutorial we are going to show, step by step, how to setup a fully functional web automation execution process in a virtualized environment.

__LXC__ will be used to bring up multiple light-weight virtualizated machines for executing Selenium WebDriver tests over different headless browsers. __Selenium Grid__ will manage the execution of these tests. __Vagrant__ will help us create a portable and self-contained execution environment, all stacked up in a single VM.

![Vagrant and LXC](/assets/vagrant_lxc.png)

## Setting up main station (Virtual Box + Vagrant with Ubuntu)

Vagrant is an open source tool for managing virtual machines. It supports different [providers](https://docs.vagrantup.com/v2/providers/) such as VirtualBox, VMWare, AWS and OpenStack.
It basically adds an additional layer to VM creation and management, making the whole process much easier and flexible.

Vagrant uses "boxes" to package a virtual machine along with its configuration. You can provide anyone with a Vagrant "box" so they can bring up an identical working environment, across different providers.

In this article, Virtual Box is used as a provider.

[Install VirtualBox](https://www.virtualbox.org/wiki/Downloads)

[Install Vagrant](http://www.vagrantup.com/downloads.html)

Once above are installed, we should install a new Vagrant box using a template (base box). This box can be used later saved to bring up multiple environments.

You can find [here](http://www.vagrantbox.es/) a list of various Vagrant base boxes to download.

We will download an Ubuntu14.04-32bit cloud base box with VirtualBox provider and name it "ubuntu-selenium".

	vagrant box add ubuntu-selenium https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-i386-vagrant-disk1.box
	
After process is completed, you should see following command line output:

	==> box: Successfully added box 'ubuntu-selenium' for 'virtualbox'!
	
Now we should create a new Vagrant environment using our "ubuntu-selenium" base box.

	vagrant init ubuntu-selenium
	
Finally, deploy the created environment

	vagrant up

Now you should have a new Ubuntu VM up and running. You can SSH to new VM using command:
	
	vagrant ssh

Note: This section is optional. If you already have an Ubuntu machine, you can use it to bring up all necessary environment in upcoming steps. However, using Vagrant has many benefits such as providing portable environments, source control of configurations and easy customizations of VMs.

## Installing LXC

[LXC](https://linuxcontainers.org/) (Linux Containers) is a virtualization method at OS-level. It allows to run multiple Linux virtualized systems (containers) on a single parent machine (host). LXC is a light-weight alternative to hypervisors virtualization such as VMWare ESXi or KVM.

We can setup multiple isolated Linux instances inside a single LXC host (which is the VM in our case) . Each instance (container) will have its own IP address, local file system, services, and memory/CPU allocation.

Let's start by SSHing to our Ubuntu VM and install LXC package. This package will install all LXC related files and configurations. 
	
	sudo apt-get install lxc

## Creating containers

After all required packages for LXC support are installed, we can start creating containers. Containers are created using "lxc-create" command.
Following call will install a basic Ubuntu container named "c1" inside VM (LXC host).

	sudo lxc-create -n c1 -t ubuntu

You can see the current status of installed containers using "lxc-ls"  command.

	sudo lxc-ls --fancy
 
	NAME  STATE    IPV4  IPV6  AUTOSTART
	------------------------------------
	c1    STOPPED  -     -     NO

Notice that initial status of container is to be stopped. We can use "lxc-start" command to start container:
	
	sudo lxc-start -n c1 -d

This command will bring up our "c1" container in the background. Let's take a look again at status of containers:
	
	sudo lxc-ls --fancy
	
	NAME  STATE    IPV4       IPV6  AUTOSTART
	-----------------------------------------
	c1    RUNNING  10.0.3.31  -     NO

Notice how container has been started and assigned a new private IP address (LXC host has a built in DHCP server using dnsmasq).

We can repeat above procedure to install more containers. 
In order to avoid repeating installation procedure on all containers, we can use lxc-clone command to create multiple containers from a pre-configured container. 

	sudo lxc-clone -o c1 -n c2
 
Each container comes with SSH access. You can SSH from LXC host to any container using lxc-console command or using an SSH client inside VM.

## Installing headless browsers

Since containers don't have a display (SSH access only), we will launch browsers in headless mode during automated tests. It means that browsers will run on a fake display, using xfvb as a display server. No worries, we are not running tests on fake browsers. We are only using an X-Server which doesn't produce any output.
 
Below installations should be done on all containers, since they will launch the browsers. 

Install xfvb server

	sudo apt-get install xvfb
 
Install Firefox

	sudo apt-get update 
	sudo apt-get install firefox
 
Install Chrome

	wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
	sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' 
	sudo apt-get update
	sudo apt-get install google-chrome-stable
 
Start xfvb server with aribtary display number (here I used 8)

	Xvfb :8 -ac 

Set the DISPLAY environment variable with the display number we used to start xfvb server

	export DISPLAY=:8

Launch Firefox browser headlessly

	firefox

## Installing Selenium Grid

[Selenium Grid](https://code.google.com/p/selenium/wiki/Grid2) is used to run Selenium/WebDriver based tests on different machines, browsers and operating systems in parallel. It makes the whole process of automation tests execution more scalable and available.
Selenium Grid operates in a hub-node mode. A hub is the main station which stores the automated tests and distribute their execution over the nodes. Each node may have different browsers and may run on different platforms.

In our environment, the LXC host will act as a hub and the containers will act as nodes. Each container node will use a different browser when executing tests from hub.

![Selenium  Grid](/assets/grid.png)

First, we need to download Selenium Server and install it on both hub and nodes. 
Since Selenium Server is a Java application, we also need to install JDK in all machines (you will also need it to build your Java tests and run them from hub).

Install latest JDK version

	sudo apt-get install default-jdk

Selenium Server can be downloaded using wget from Selenium website to any preferred location in LXC host and containers.

	wget http://selenium-release.storage.googleapis.com/2.43/selenium-server-standalone-2.43.0.jar
 
After we are done installing JDK and Selenium Grid, we can start bringing up the hubs and nodes.

Inside the LXC host we launch a hub

	java -jar selenium-server-standalone-2.43.0.jar -role hub
	
You should see following command line output:

	INFO - Launching a selenium grid server

Now you should be able to access following URL for monitoring Grid's nodes (4444 is the default port bind for Selenium Grid's HTTP service):
	
	http://localhost:4444/grid/console

![Selenium  Grid initial state](/assets/grid_init.png)

Few notes regarding Vagrant and VirtualBox networking:

- In order to access URL from host machine (outside VM), you need to use [port forwarding](https://docs.vagrantup.com/v2/networking/forwarded_ports.html) for Grid's port.

- IP address of LXC host (hub) is the internal network address in the VM's private network. VirtualBox uses IP addresses in range 10.0.0.0 - 10.255.255.255 (according to [RFC1918](http://tools.ietf.org/html/rfc1918) Section 3: "Private Address Space")
	
After our hub is ready, we should go over containers (using SSH) and launch the nodes. Notice that Selenium Server host should be the private IP address of LXC host.

	java -jar selenium-server-standalone-2.43.0.jar -role node -hub http://LXC-HOST-IP:4444/grid/register
	
You should see following command line output:

	INFO - Launching a selenium grid node
 
After we launch the nodes on all containers, we should see them in Grid's web interface. In this example, we only use one container (node).

![Selenium  Grid with node](/assets/grid_with_node.png)

## Running tests with Grid

Now that all nodes are ready, we can start running the tests on hub.

A minor modification of WebDriver initialization is required to run Selenium WebDriver tests over grid. Instead of creating a specific WebDriver browser instance such as FirefoxWebDriver or ChromeWebDriver, we should use RemoteWebDriver and DesiredCapabilites instead.

__RemoteWebDriver__ -  used to execute the tests remotely on nodes.

__DesiredCapabilities__ - used to define the browser, version and platform that node should match.

RemoteWebDriver instance is initialized with URL of hub and the DesiredCapability object. For example:

	DesiredCapabilities capability = DesiredCapabilities.firefox();
	WebDriver driver = new RemoteWebDriver(new URL("http://LXC-HOST-IP:4444/wd/hub"), capability);

A node that matches the criteria in DesiredCapabilities will be chosen by RemoteWebDriver to run the tests.

Below is a complete example of a simple WebDriver test that uses Selenium Grid. This simple test verifies Google's main page title and takes a screenshot.

{% gist cfc60640702a07639f64 %}

When running the test from hub, the container with matched browser type, version and platform will be chosen.

Tests are built and run according to used build tool ([Apache Ant](http://ant.apache.org/), [Apache Maven](http://maven.apache.org/), [Gradle](http://www.gradle.org/), etc...) and testing framework ([JUnit](http://junit.org/), [TestNG](http://testng.org/), etc...). These software should also be installed on LXC host prior to running tests.
 
##Exporting the environment

Using Vagrant, you can create a new base box (template) with all the changes we applied during previous sections. That way, you can setup multiple environments using the same configurations in less than a minute.
We can create a new base box of out of current Vagrant environment using vagrant package command:

	vagrant package --output ubuntu-selenium-new.box
	
Note that above command supports VirtualBox provider only. Exporting to other providers should be done manually (there should be enough documentation on the internet on how to package base boxes with different providers).

## Additional tips

A configuration management tool such as [Puppet](http://puppetlabs.com/) or [Chef](https://www.getchef.com/chef/) can be used to install and manage all needed packages for setting up the environment. Vagrant even has built-in provisioning support for such tools, which makes it easier to configure Vagrant boxes using Chef recipes or Puppet. 

Additionally, you may setup a [Jenkins](http://jenkins-ci.org/) build server on LXC host (hub) to run tests in an orderly manner using scheduled jobs for Continuous Integration.

![Jenkins, Puppet and Chef](/assets/jenkins_puppet_chef.png)

Now you should have a fully functional environment for running automated web tests over different machines and browsers in parallel. All these features are stacked inside a single portable VM which can be scaled up according to execution needs.
