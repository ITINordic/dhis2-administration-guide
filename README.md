# DHIS2 Hosting Guide

**DHIS2 Hosting Guide**


## Table of Contents
- 1. Introduction..............................................................................................................
   - Background..........................................................................................................
   - System Architecture and Requirements.....................................................................
- 2. Setting Up and Securing the DHIS2 Server....................................................................
   - Overview.............................................................................................................
   - Creating Users......................................................................................................
- 3. Installing DHIS2......................................................................................................
   - Overview...........................................................................................................
   - Preparing to Set Up DHIS2...................................................................................
   - How to Set Up DHIS2.........................................................................................
      - Installing DHIS2 Instances............................................................................
- 4. DHIS2 Administration..............................................................................................
   - Overview...........................................................................................................
   - Useful LXC Commands........................................................................................
   - Access and Configuration of Containers..................................................................
      - Commands for Logging into Containers..........................................................
      - Optimisation of the Postgres Database Container...............................................
      - DHIS2 Instance Container Admin Tasks
      - Apache Proxy Container Configuration...........................................................
      - Munin Monitoring Container.........................................................................

**List of Figures**

```
1.1. DHIS2 Hosting Architecture Overview....................................................................... 4
3.1. Apache Proxy Home Page...................................................................................... 12
3.2. Selecting the DHIS2 Version to Install...................................................................... 14
```
```
iv
```

# Part I. The Hosting Environment


## Table of Contents


# Chapter 1. Introduction

## Background

The District Health Information Software (DHIS2) is an open source Java-based web technology, which can be sustainably, and is recommended to be, installed with other free and open source tools. Many of the industry standard implementations of DHIS2 occur within the Linux operating system, even though it can be set-up in other environments. However, in order to benefit from the community, it is highly advised to work on Linux, particularly the newest Ubuntu LTS server editions. A number of tools have been developed to support administrators to install and maintain DHIS2 on Ubuntu. This guide will provide the information that is necessary, to install, and maintain DHIS2. It is important to know that since technology evolves, these instructions will need to be updated as new techniques and versions of DHIS2 emerge. In this vein, the authors shall provide the best practices as prevailing at the time of this publication, which can provide the necessary guidelines as to configure a the leading edge set up. Besides the pre-requisite that users of this guide should know how to use the Linux command-line, there are other technologies that are required in order to successfully host DHIS2. In particular, the system uses a Postgres database, is accessed through a web proxy technology such as Apache, and is more scalably hosted within virtualised container environments, of which LXD is used in this guide. It will be important for administrators to join the community which can be effectively accessed through enrolling onto the online forum that can be found here [ https://community.dhis2.org/]. It is on the forums found at this location that administrators will be able to get support on day to day challenges experienced while hosting the system, and in addition be able to provide support in turn to others who might be facing known issues.

## System Architecture and Requirements


As mentioned, DHIS2 is best run on Ubuntu LTS editions. The LTS means that the version of Ubuntu that is installed has long term support and will continue to receive patches and updates for a period, usually some years, without any need to upgrade. At the point of publishing this guide, the version which has been widely tested is Ubuntu 20.04 LTS. This is the version on which the examples provided in this guide will be tested.

As pertaining to hardware requirements, it is important to realise that there is no hard and fast rule as what is required to run DHIS2. As often retorted by Bob, who developed the techniques and tools used in this guide, "*asking for server specifications to reliably host your DHIS2 instance is like asking me how long a piece of string is and the answer is, 'I don't know!'*". You can understand the meaning of this statement once you realise the difference in scale for different DHIS2 set ups. Running DHIS2 configured for a hundred users, for instance, is not similar to an instance which supports 10 000 users. In addition, DHIS2 has both routine and vent and case-based modules, which determines the traffic and volume of data at different time-frames. It is important to remember that DHIS2 is run and configured on a number of Free and Open Source Software (FOSS) tools which have an impact on the minimum requirements. In particular, the Linux operating system itself can run on minimum hardware, yet the other components in combination, that is the virtualisation LXD containers, one for the proxy, preferably Apache, another container for the Postgres database, another for the DHIS2 application, and the last for system monitoring tools, will be the major determinants of the minimum hardware specifications. In addition, it is important to realise that the number of users of the system expected at any given time will form a variable scale on which the hardware required will be determined. In essence, it should always be possible to increase hardware allocated to the system as users and running instances increase, or performance bottlenecks are noted. A key element where performance issues can arise is on the database, since read and write operations from users do take time, and the number of permitted connections are limited in number. A number of parameters for optimisations, depending on hardware available should be tested until an optimal performance based on limited hardware is reached. For the sake of this guide, we will provide examples based on a server with 16 GB on RAM, 6 CPU Cores, and a 320 GB SSD Hard Disk. The system will be set up in accordance with the architecture in the figure below.

**Figure 1.1. DHIS2 Hosting Architecture Overview**


# Chapter 2. Setting Up and Securing

# the DHIS2 Server

## Overview


The first step to installing DHIS2, is to install the Linux Operating System (OS), currently the recom-
mended distribution is Ubuntu 20.04 LTS. As noted in the previous sections, for practical purposes,
the server used in this guide has 16 GB RAM and a 320 GB SSD hard disk. To limit the footprint of
the OS, the server edition of Ubuntu will only have a Command Line Interface after installation.

#### Creating Users


Once the server edition of Ubuntu has been installed, it will be time to create user accounts. By default,
Ubuntu will come with a ' root ' user account which will have administrative privileges. The password
for this user will have been set during the installation process. While the steps that follow assume that,
once installed, the server will be accessed remotely, it is possible that this can be performed on the
physical machine. If accessing the server remotely from a Windows computer, it is recommended to
use Cygwin, which is described on the site as a " a large collection of GNU and Open Source tools
which provide functionality similar to a Linux distribution on Windows ". If this is the case, it will
be important to choose OpenSSH server and client programs from the packages list. For access from
other Linux distributions, these tools come pre-installed in the ' Terminal ' client, or will be prompted
when needed by the OS.


If you do not have an SSH key pair, this will be the time to generate it. The SSH key enables its user
to access the server remotely without passing your username and password as plain text. There is a
risk that using a username password combination everytime you access your server will expose it to
attacks, such as via interception. To create an SSH key, you would run the command below in the
Terminal client:
```
ssh-keygen
```
When the above instruction is run, two files are created, a private key and a public key. Make sure
to keep both safe, and never to share your private key. The public key can be shared with others and
will be used in the steps that follow. In our training scenario, assuming our server is at IP address
' 139.162.170.134 '. As mentioned earlier, the ' root ' user is already on the machine. To access this server
from our ' Terminal ' client, we would type and run the command below:
```
ssh root@139.162.170.
```
After running the command, by pressing enter, the prompt below will appear, asking whether you
would like to connect to the server, at which point you answer ' yes' and continue, as shown:
```
Are you sure you want to continue connecting (yes/no)? yes
```
At this point, you will be asked to enter the password which you set for the root user, which you do and
press enter. If the password is correct you will succesfully log onto the server at which point you will
begin configuration. The first thing that must be done at this point is to create a new user for yourself,
it is not advisable to use the ' root ' user as it can potentially expose your server to malicious attacks.

- Create a new user, use your own name for example ' _bob_ '. You will be asked to enter details such
    as the full name, organisation and password

```
sudo adduser bob
```
- Add the created user to the administrators group

```
sudo usermod -aG sudo bob
```
- Change the user from ' _root_ ' to the newly created user, that is ' _bob_ ' in our example, and going into
    the user home directory by running the commands below.

```
sudo su bob
```
```
cd
```
- Create the directory which will hold the public SSH keys for the user, in this case ' _bob_ '

```
mkdir .ssh
```
- Open the file that will hold the public key for the user. You can use your favourite text editor program
    e.g ' _nano_ ', in this case we use ' _vi_ '. Once you open this file, using the command below, you should
    now paste the contents of your public key that you created above, and save. You must be familiar
    with the commands that enable you to edit and save the file in your editor program, or you can
    search online. Ensure you do not leave any trailing spaces

```
vi .ssh/authorized_keys
```
- A sample public key that you paste in the file above is as shown below. Remember to keep your private key private and not to put it on the server.

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC+7Q9mcuMOt8vGrgHrgVxOpHNjql
9qK05hT6Q6j1ULSjsQUYae/2tvPsGUf1vWyDC9We8fPrScFRkghPTYYvRtdUPjdkS
iNDbUoq7b5FY9dAiBYz6dTG5GgqOoHunaFyuQjFT+2TQBqmenjI/PcVEOR9T3jejSw
D4Dam4K4ALXWGwC3owfc8bNbQ9tEBeAXavs5TWS66jozJeN94DIhFWY6fBJWm1N4Ot
Cf1nfO6Q4KzvigtKdebIFsmTOOHXHGfEIvGq3RZ/41o2s1g+UMej+6/eSeOrA6xZXQ
JsDoJFg/fQ+lnM6JnHyzbCz+kDP9j86vALy+YQaj34xusUZpH5 bob@zambia
```
- After saving the public key file, you will need to set the right permissions for the key to be activated.
    Run the commands below

```
chmod 600 .ssh/authorized_keys
```
```
chmod 700 .ssh
```
- At this point it will be important to change some SSH settings, which can be done once. To do this, you would need to open the settings file using your favourite text editor such as using ' _vi_ ' as shown below:

```
sudo vi /etc/ssh/sshd_config
```
- There are a number of lines which you must change in this file to adequately secure the server. Find ChallengeResponseAuthentication and set to no

```
ChallengeResponseAuthentication no
```
- Find PasswordAuthentication set to no

```
PasswordAuthentication no
```
- Find UsePAM and set to no

```
UsePAM no
```
- Find PermitRootLogin and set to no: (This will ensure that the root user will no longer be able to log on to the server remotely)
```
PermitRootLogin no
```
- Fing the SSH port and change it to a preffered value from the default which is 22, for example uncomment and change to 877 as below:
```
Port 877
```
***Note: You should never set the sshd port to something over 1024.

- Find PubkeyAuthentication and set to yes

```
PubkeyAuthentication yes
```
- For the changes made above to take effect, you must restart SSH as shown below:
```
sudo systemctl restart ssh
```
At this point, if the instructions above have been accurately followed, it should be possible to now
log out and log back in using the SSH keys. Please note that, if there is a mistake, it is possible you
might lock yourself out of the server and will need to either physically access it if it is locally hosted,
or use some other support from the service provider to access the server to fix any issue. If however,
everything is as it should be it should be possible to log back in using SSH using a command such
as the one below. It is at this point that you would use the private key to access the server, and will
not be prompted for a password.
```
ssh -i /your_private_key_file_location bob@139.162.170.134 -p 877
```

# Part II. The DHIS2 Environment


# Chapter 3. Configuring Linux Containers

Install zfs

```
sudo apt-get install zfsutils-linux
```
Configure zpool

```
sudo zpool create tank /dev/sdc
```

Cheheck if zpool is created

```
zpool list
```
You should get output as below

```
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
tank   540G   118K   540G        -         -     0%     0%  1.00x    ONLINE  -
```
Set up file system in pool

```
sudo zfs create tank/lxd
```

For backups you can create additional compressed zfs location
```
sudo zfs create -o compression=lz4 tank/backups
```

Install lxd
```
sudo snap install lxd
```

Initialise lxd 
```
sudo lxd init
```

Configuration settings during set up
```
Would you like to use LXD clustering? (yes/no) [default=no]: no
Do you want to configure a new storage pool? (yes/no) [default=yes]: yes
Name of the new storage pool [default=default]: 
Name of the storage backend to use (ceph, btrfs, dir, lvm, zfs) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: no
Name of the existing ZFS pool or dataset: tank/lxd
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 192.168.0.1/24
Would you like LXD to NAT IPv4 traffic on your bridge? [default=yes]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like the LXD server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```
Add an Existing User Account e.g bob to LXD Group
```
sudo usermod -a -G lxd bob
```

To see the containers available run the command below
```
sudo lxc list
```


# Chapter 4. Installing DHIS

### Overview


As mentioned earlier, this guide uses DHIS2 developed tools, the latest, to assist administrators to set
up and manage a DHIS2 installation. A key approach which is utilised nowadays is that of containers.
While many can be familiar with other container technology such as Docker, in this case we utilise a
special Ubuntu container technology called LXD. A reference site states that " LXD is a next generation
system container manager. It offers a user experience similar to virtual machines but using Linux
containers instead ". Utilising LXD, we will create a system of containers as illustrated here. At this
point it will be important to log back into the server using the ssh keys created earlier, after which one
can proceed into the next section.

### Preparing to Set Up DHIS2


The first step in the set up is to configure the Firewall. This will limit incoming traffic to the server by allowing only certain types of connections through a finite number of ports. The Ubuntu server on which the installation occurs, already has a Firewall client which can be configured with only a few steps.

- Allow All Incoming HTTP connections
```
sudo ufw allow http
```
- DHIS2 is recommended to be run on a secure connection which uses HTTPS for encryption. To allow all such connections you should run the following instruction
```
sudo ufw allow https
```
- Earlier on, we changed the SSH connection port from 22 to 877, we should open this port as well so we can be able to connect to the server once the Firewall is enabled
```
sudo ufw allow 877
```
- It is now time to enable the Firewall with the following command
```
sudo ufw enable
```
- After agreeing to the prompt on enabling the Firewall, the next step is to check the status with the following command
```
sudo ufw status
```
Having confirmed that the Firewall is indeed enabled, it will be important to ensure that the IP address of the server is assigned a domain name. It will not be possible to set up a security certificate on the domain without the name. For this, you would need to contact the owner of your organisations domain so they can assign the IP address. The process can be immediate or, in some cases, take up to a few hours. For our case, we give the domain name a value such as ' dhis.moh.xxx.zz ' will be asigned to the IP address of the server, which is for our case, 139.162.170.134.

It will be important for the set up, to configure the time zone. In order to get the list of available timezones, there is need to run the instruction below:
```
timedatectl list-timezones
```
In our case, the timezone that is suitable for Zambia is:
```
Africa/Lusaka
```
Having done all the preceeding steps, the next part will be to download the latest DHIS2 installation tools. At present, these can be found at this link [ https://github.com/bobjolliffe/dhis2-tools-ng]. To get the latest tools onto your server ou would need to run.
```
git clone https://github.com/bobjolliffe/dhis2-tools-ng.git
```
The above command will download the tools into your home directory from where you can begin the final steps of setting up DHIS2. It will be important to keep up to date with the community, if these links should change by creating an account at this link [https://community.dhis2.org/].

### How to Set Up DHIS2

Once you have completed the steps above, the next thing is to navigate into the folder that was downloaded onto the server. Having located this folder, which should be in the same directory in which you ran the the ' _git clone_ ' command previously, run the following to navigate to the ' _setup_ ' directory:
```
cd dhis2-tools-ng/setup
```
Having navigated to this directory, it is important to have a sense of the files that are in it. The starting point of the setup will be to edit a settings file which will be used through the installation. The installation scripts that will be run will depend on this file. As highlighted earlier, it is expected that by this point, the Firewall has been activated, with the ' _http_ ' port, that is Port 80, and the ' _https_ ' port, that is port 443, open for traffic. A sample settings file named ' _containers.json.sample_ ' is located in the ' _configs_ ' folder found in the ' _setup_ ' directory to which you navigated to earlier. The next step is to copy the sample file and rename it so it can be found by the installation scripts by running the command below:
```
cp configs/containers.json.sample configs/containers.json
```
Having renamed the file, it will be important to edit some lines to be in accordance with your hosting environment: At this point, it is advisable t use a text editor of your choice, in which case we will use ' _vi_ ' to view and edit the file. We will therefore open the file using the command below:
```
sudo vim configs/containers.json
```
Once opened, the sample file will look as below:
```
{
    "fqdn":"xxx.xxx.xxx",
    "email": "xxx@xxxx",
    "environment": {
        "TZ": "Africa/Dar_es_Salaam"
    },
    "network": "192.168.0.1/24",
    "monitoring": "munin",
    "proxy": "apache2",
    "containers": [
        {
            "name": "proxy",
            "ip": "192.168.0.2",
            "type": "apache_proxy"
        },
        {
            "name": "postgres",
            "ip": "192.168.0.20",
            "type": "postgres"
        },
        {
            "name": "monitor",
            "ip": "192.168.0.30",
            "type": "munin_monitor"
        }
    ]
}
```
It will be important to edit the top 3 lines to reflect the configuration of your server. In our case, the testing values will now look as below:
```
"fqdn":"dhis.moh.xxx.zz",
"email": "dhisadmin@moh.xxx.zz",
"environment": {
    "ZM": "Africa/Lusaka"
},
```

Having edited the file, and saved it according to a preffered configuration, it will be time to start the
installation process by running the command below (This could take some time to run, and as Bob
Jolliffe, the author of the tools says, " _this could be a good time to make tea or coffee_ "):

If you set up the lxd containers manually, following the guide, you do not need the following step.
```
sudo ./lxd_setup.sh
```

Install the service scripts by running
```
sudo ./install_scripts.sh
```

In the case where you manually set up lxd, RESTART THE SERVER and then run the following command

```
sudo ./create_containers.sh
```

Once the instruction above runs to completion, a good way to test will be to navigate to the URL that
was set in the settings file, that is ' _dhis.moh.xxx.zz_ '. The page below should be seen:

**Figure 4.1. Apache Proxy Home Page**

At this point, all the containers necessary to install an instance of DHIS2 as shown in the architecture here, are now in place. To see the list of available containers, you can run the following command on your host server:
```
sudo lxc list
```
Having run this command, the following will be displayed:
```
+----------+---------+---------------------+------+------------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+----------+---------+---------------------+------+------------+-----------+
| monitor | RUNNING | 192.168.0.30 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| postgres | RUNNING | 192.168.0.20 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| proxy | RUNNING | 192.168.0.2 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
```
This shows the three LXD containers that have been created using the installation script run earlier. If this does not show, or there were errors seen in the installation earlier, you can run the following command to remove all the containers, and allow you to try the set up again - usually after resolving whatever issue would have affected the installation (remember that this command should be run from
the ' dhis2-tools-ng/setup ' directory :
```
#WARNING: Run the command below only if the installation was unsuccessful
#This will reset your server and would delete all containers
./delete_all.sh
```
If however, everything ran according to plan, there is no need to delete the containers as done above.
Once these steps above have been run successfully, you can now install individual instances of DHIS2.

**Configuring Apache**

log in to the proxy container using the following command
```
sudo lxc exec proxy bash
```
Configure Apache virtual hosts

Check which hosts are enabled
```
ls -la /etc/apache2/sites-enabled/
```
You get a list of existing host files, as shown below:
```
drwxr-xr-x 2 root root  3 Sep 23 11:14 .
drwxr-xr-x 9 root root 13 Sep 23 11:14 ..
lrwxrwxrwx 1 root root 35 Sep 23 11:14 000-default.conf -> ../sites-available/000-default.conf
```
As you can see, the DHIS2 apache configuration is not enabled. First we disable the current configuration file
```
sudo a2dissite 000-default.conf
```
Check available host configurations

```
sudo vi /etc/apache2/sites-available/
```
You should get an output as below, showing you the available configurations
```
000-default.conf   apache-dhis2.conf  default-ssl.conf
```

In this case, we know we want to activate the DHIS2 configuration file. So we run the following command
```
sudo a2ensite apache-dhis2.conf
```

Enable the updated configuration as below
```
systemctl reload apache2
```
Ensure all the existing SSL certficates are not enabled above and the Apache server has restarted with no errors using the following command
```
systemctl status apache2.service
```
If the server is running proceed to set up the security certificate.

**Installing SSL Certificates**

NB: If your organisation already has a certificate for your domain, you can continue to use that. However, you can easily install letsencrypt certificates which are free as shown below.

Get certbot:
```
sudo apt install certbot python3-certbot-apache
```
Install your certificate to the chosen domain
```
sudo certbot --apache -d your_fqdn_here[eg data.example.com]
```

You can reload your apache configuration after ensuring SSL is now on and the certificate location configured

**Installing DHIS2 Instances**

In our architecture, each instance of DHIS2 will run in its own container. These containers will need to know where they will be saving data, that is the container that is running the ' Postgres' database. In our case, we will create a DHIS2 intance called ' hmis '. To do this the command has the structure
```
'sudo dhis2-create-instance name_of_our_dhis2_instance IP_address_of_the_new_container
name_of_the_database_container '
```
In our case, to create an instance called ' hmis ' at the IP address ' 192.168.0.10 ' (note the IP address should be in the same range as the other containers above), we would run the following command:
```
sudo dhis2-create-instance hmis 192.168.0.10 postgres
```
To create another instance called ' tracker ' for example, we will need to run the following command (note how we have changed the IP address to an unused one):
```
sudo dhis2-create-instance tracker 192.168.0.11 postgres
```
Having created the two instances using the commands above, we can list the containers as we did in an example above, at which point we will see the following:

```
+----------+---------+---------------------+------+------------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+----------+---------+---------------------+------+------------+-----------+
| hmis | RUNNING | 192.168.0.10 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| monitor | RUNNING | 192.168.0.30 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| postgres | RUNNING | 192.168.0.20 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| proxy | RUNNING | 192.168.0.2 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| tracker | RUNNING | 192.168.0.11 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
```
As can be seen, two additional containers have been created to host the DHIS2 instances. remember that in our architecture, each instance runs in its own container. It is also important to realise, we have created DHIS2 instance containers, but we have not yet installed DHIS2 on any one of them. To do this, we would need to ' _deploy_ ' the version of DHIS2 that we would wish to use. The available versions can be seen at the DHIS2 website [https://www.dhis2.org/], or by going to the following link [https://www.dhis2.org/downloads]. To choose the version, one would have to right click on the corresponding
web archive, ' _WAR_ ' file as shown in the image below. In this case we are getting the URL to the latest
DHIS2 version at the time of developing this manual, which was 2.33.3.

**Figure 4.2. Selecting the DHIS2 Version to Install**

To deploy the WAR file selected in the above image to the 'hmis' container we would run the following
command:
```
dhis2-deploy-war -l https://releases.dhis2.org/2.33/2.33.3/dhis.war hmis
```
At this point, if the installation was successful, DHIS2 would be accessible at our URL, from the browser, which in our case would be ' _dhis.moh.xxx.zz/hmis_ '. The same can be done for our ' _tracker_ ' container by running the following:
```
dhis2-deploy-war -l https://releases.dhis2.org/2.33/2.33.3/dhis.war tracker
```
VOILA! You have now successfully installed DHIS2. However, installing DHIS2 is only the first part of hosting the system. In the next chapter, we will go through some of the ways in which this environment can be maintained in a production set up.


# Chapter 5. DHIS2 Administration

### Overview

This section is going to introduce us to a few tools that are necessary for the day to day administration of DHIS2. In particular, we will learn how to start and stop a running DHIS2 instance, where to find the logs and how to backup and restore a database. We will introduce the different locations as well where the DHIS2 configuration files can be found with some tips on basic troubleshooting. As a starting point, we will introduce a few commands which can be used to deal with containers, access each of the containers that has been created and work with Munin for monitoring.

### Useful LXC Commands

In the set up above, each DHIS2 instance has its own container. So by starting or stopping a container, you stop and start the Tomcat in which the system is running. Containers are virtual machines and can be seen as different machines. For a comprehensive coverage of the available LXC commands, you can refer to the reference manual online. In this guide we will deal with some of the basic instructions.

- To stop an instance, stop the container which hosts it. For example to stop the 'hmis' instance we
    can run the command below:
```
sudo lxc stop hmis
```
If in this case, we ran the previous command to list the LXC containers, the following would show:

```
+----------+---------+---------------------+------+------------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+----------+---------+---------------------+------+------------+-----------+
| hmis | STOPPED | | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| monitor | RUNNING | 192.168.0.30 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| postgres | RUNNING | 192.168.0.20 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| proxy | RUNNING | 192.168.0.2 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
| tracker | RUNNING | 192.168.0.11 (eth0) | | PERSISTENT | 0 |
+----------+---------+---------------------+------+------------+-----------+
```
As can be seen in this case, the ' hmis ' container has been stopped. At this point, if a user tried to access
the instance on a web browser, the DHIS2 system would be unavailable.

- To start the same instance up again, the following command would need to be run:
```
sudo lxc start hmis
```
- In some cases, there might be a need to create an additional container to host another system on the same server. For instance, if we needed to add an interoperability container which might be used for exachanging data with other systems. We would firstly create the container called ' _interop_ ' by running the following command:
```
sudo lxc init ubuntu: interop
```
- We would then attach the container to the network bridge by running

```
sudo lxc network attach lxdbr0 interop eth0 eth
```

- Then finally, we would assign an IP address to the container by running the following command:

```
sudo lxc config device set interop eth0 ipv4.address 192.168.0.
```
- At this point the created container will need to be started. However, in some cases it will be important to delete a container. To do this, for example to delete the ' _interop_ ' container you would first stop it, and run the following command:
```
sudo lxc delete interop
```
Of course, deleting containers should not be taken lightly at all, as mistakes can occur, which are
sometimes irreversible. To fully explore the options as regarding LXC commands, you can find nu-
merous resources online. All the containers will be running on the same physical hardware, so it is
important to consider the available resources before creating additional containers.

Also it is possible to manage memory and processor core allocations to containers

```
lxc config set container_name limits.memory 100MB
```

In the next section, we shall start to look at accessing individual containers and what can be done
inside them.

### Access and Configuration of Containers

As seen in the architecture document, there are at least 4 containers which are essential to the running
of the DHIS2 system,

1. The **_proxy container_** , which in our case runs the Apache Reverse Proxy
2. The **_database container_** on which we have installed Postgres
3. The **_system monitoring container_** , which has Munin installed
4. At least one **_DHIS2 container_** which runs the web application

**Commands for Logging into Containers**


There are many reasons why you might want to log directly into the container, such as retrieving log files when the system misbehaves or optimising performance. In these cases, it will be important to run commands inside the containers. We list here, the commands that will be needed to do just that.

- To log onto a container, for example the one that was named ' _hmis_ ', you would need to run the following command (note that you can subsititute ' _hmis_ ' with any name of a container that you would want to access):

```
sudo lxc exec hmis bash
```
- Once the task that you needed to perform on that server is completed, you simply type the command
    below to return to the host machine:

```
exit
```
- It is also possible to run commands directed at any container from the host machine. For example
    to run a query directly on the hmis database from the host we could run:

```
sudo lxc exec postgres -- psql -c 'select * from period limit 5' hmis
```
**Optimisation of the Postgres Database Container**

The container hosting the Postgres database, often needs to be tuned in order to make the best use of available resources. Tuning the database can noticeably improve the performance of the DHIS system. Postgres offers a number of configuration options which can be modified to improve performance. The available memory is the basis of some of the central configuration options. Generally, the more the memory assigned to the database, the better the system performs. It is also important to think about the other systems running on the host when allocating memory to Postgres. In our case, our host server has 16GB of RAM, and we have set up two instances of DHIS2 in two additional containers. We can start by assigning 8GB of data to the 'postgres' container, and could probably reduce it if we see a dip in perfomance. To assign a limited memory to the postgres container we would run the following command:
```
sudo lxc config set postgres limits.memory 8GB
```
The above command is very important as it will ensure the memory availbale to the Postgres database
is limited and will not interfere with other parts of the system during periods of high server usage.
Next we can log onto the 'postgres' container by running:
```
sudo lxc exec postgres bash
```
Once logged in, we can open the Postgres database configuration file, in order to fine tune the settings

- keeping in mind that the container only has 8GB of RAM available. In this case, you can use the text
editing tool of your choice. For the examples, we once again use 'vi' which is quite common. To open
the Postgres database configuration file for DHIS2 we would run the following command (note that
this assumes you are using Postgres version 10 - this can be different in future):
```
sudo vi /etc/postgresql/10/main/conf.d/dhispg.conf
```
This file would open in the terminal client and has the default settings below:
```
# Postgresql settings for DHIS2

# Adjust depending on number of DHIS2 instances and their pool size
# By default each instance requires up to 80 connections
# This might be different if you have set pool in dhis.conf
max_connections = 200

# Tune these according to your environment
# About 25% available RAM for postgres
# shared_buffers = 3GB

# Multiply by max_connections to know potentially how much RAM is required
# work_mem=20MB

# As much as you can reasonably afford. Helps with index generation
# during the analytics generation task
# maintenance_work_mem=512MB

# Approx 80% of (Available RAM - maintenance_work_mem - max_connections*work_mem)
# effective_cache_size=8GB

checkpoint_completion_target = 0.8
synchronous_commit = off
wal_writer_delay = 10000ms
random_page_cost = 1.1
log_min_duration_statement = 300s

# This is required for DHIS2.32+
max_locks_per_transaction = 1024
```
The 4 settings that you should uncomment and give values to are shared_buffers , work_mem , maintenance_work_mem and effective_cache_size. We are guided in setting values for these parameters by the limited memory of 8G that is allocated to the container. We can delve a little bit into these settings
so we know what each one is about:

- shared_buffers : This is the memory available for caching data, which is important when large data sets are accessed. So since we have 8GB available, you should uncomment this line set this value as below:
```
shared_buffers = 2GB
```
- work_mem : This parameter allows postgres to do tasks in memory, for example sorting. The problem here is that the memory is multiplied by the number of connections, so if it is left at 20MB as it is, 50 connections would mean 1GB of memory used. In our case, this is acceptable. However, with many users, it might be necessary to revise this downwards. For now, we can uncomment the line and keep it at 20MB as shown below:
```
work_mem=20MB
```
- maintenance_work_mem : This is the memory allocated to maintenance operations, for example creating indexes and tables such as occurs during the restoration of the database. Due to the memory limitation, we can uncomment and keep this at 512MB:
```
maintenance_work_mem=512MB
```
- effective_cache_size : This specifies how much memory is available to the database from the operating system. Keeping in mind that we assigned 8GB to the OS, and that other memory has been allocated above, we can assign this to 4GB:
```
effective_cache_size=4GB
```

Postgresql is an extremely configurable database with hundreds of configuration parameters. This brief installation guide only touches on the most important tunables. For consideration of other configuration options you can refer to the older DHIS2 set up guide in the ' PostgreSQL performance tuning ' section at this link [ https://docs.dhis2.org/2.31/en/implementer/html/install_server_setup.html]. For even more
advanced configuration, you can refer to the Postgres site. Having edited and saved the configuration file, as suggested above, you can log out of the database container back onto the host. The settings will be applied next time the database container is started. Before restarting the database server, it will be important to stop the containers running DHIS2. The sequence of commands is as follows in our case:
```
sudo lxc stop hmis  
sudo lxc stop tracker  
sudo lxc restart postgres  
sudo lxc start hmis  
sudo lxc start tracker
```
**DHIS2 Instance Container Admin Tasks**

The tools covered in this guide, created a container for each DHIS2 instance. The DHIS2 WAR file was installed in a Apache Tomcat (tomcat9) web server environment on the container.

**Important Logs**

One of the most important tasks when running DHIS2 is monitoring the logs. The logs are especially important when the system fails to load or suddenly crashes, where it will be important to see the cause of an error. In previous installations of DHIS2, the logs could be found in an Apache Tomcat file called catalina.out. However, with the new version that is used in this installation, that is tomcat9, the name and location of the main log file has changed. This file can be now be seen in real time by using the following command:
```
sudo tail -f /var/log/syslog
```
Having run this command, especially soon after the container is started, will enable you to see in real time the progress of the DHIS2 system in real time. If there are errors, these can also be found in this file. To see a fixed number of lines of the log file you could also run the following command, for
example to see the last 300 lines of the log
```
sudo tail -n 300 /var/log/syslog
```
You could therefore peruse the file for an errors. These logs are particularly useful when requesting for support from the community. It is often not enough to state that your system is not working, but rather by sharing the log files which could contain the error, experts in the community can provide remote support, especially those who might have faced similar issues in the past. In addition to this main log file, there are also other log files which were set up when DHIS2 was installed. A number of DHIS2 specific logs can be found by browsing to the location ' /opt/dhis2/logs/ '. You can list the contents of this file by simply by running:
```
sudo ls -la /opt/dhis2/logs/
```
It is possible that some issues that can be faced in running DHIS2 will require some additional information for diagnosis, and in this case one of the log files at that location could be useful.

**Additional Apache Tomcat Configuration**

As memory is generally an issue, it could be important to allocate the amount of memory that should be availed to any DHIS2 installation. In our case, we only have about 8GB available and are running 2 DHIS2 instances. We consequently might want to assign 4GB to our ' hmis ' instance. To do this, we would need to access the JAVA_OPTS variables. To do this, we would once again need to use our preferred text editor to open the file, as demonstrated in the command below:
```
sudo vi /etc/default/tomcat9
```
We would change the JAVA_OPTS variable to a value such as shown below:
```
JAVA_OPTS="-Djava.awt.headless=true -XX:+UseG1GC -Xmx4G -Xms4G -Djava.security.egd=file:/dev/./urandom"
```
In this case, it could be necessary to restart the Apache Tomcat server by running the command below:
```
sudo service tomcat9 restart
```
Having restarted the tomcat server this way, it is always a good idea to monitor the logs if any issue might arise. In our case, we would run the command shared earlier, that is:
```
sudo tail -f /var/log/syslog
```
If there is an error, it will be possible to see it in this log. Another file which could be of importance in Apache Tomcat is ' server.xml '. This file is generally fine as it is, but could need to be tweaked in certain environments. The file can be opened by running the command below:
```
sudo vi /etc/tomcat9/server.xml
```
Some settings which can be changed, include the server timeout or http pool size. However, this file should not be changed unless the situation requires it. For now, we can leave it as it is.

**DHIS2 Instance Configuration Options**

The DHIS2_HOME directory ca be found at the location '/ opt/dhis2 '. The most important file at this location is the ' dhis.conf ' file. This can be opened in our preferred text editor as follows:
```
sudo vi /opt/dhis2/dhis.conf
```
In general, the settings in this file are sensitive and should not be changed. However, DHIS2 comes with a number of options, for example the possibility to encrypt data. If this is needed, this is the file which will need to be edited. According to the reference manual, some of the settings that are immediately available for editing include the following:

1. _connection.pool.max_size_ - the default value of this is 80 which should be adequate for most systems. For a small test system consider reducing this to, say 10. In very rare instances, usually when there is some other problem in your database, you might need to increase this. You need to ensure as you modify this that you stay within the limits of max_connections in postgresql configuration.
2. _analytics.cache.expiration_ - if you uncomment this and keep the default setting of 3600, it will cache the results of SQL analytics queries for an hour. These can sometimes be a big load on your database so it is highly advisable to enable this. On large aggregate systems it can have a dramatic effect.
3. _system.session.timeout_ - this determines how long you can leave the application without being obliged to log back in again. The default setting (1 hour) is probably too long for sensitive applications in clinical settings. Something like 10 or 15 minutes might be more reasonable.

**Apache Proxy Container Configuration**

All incoming connections to your DHIS2 instances from outside, will come through the Apache Proxy. This was installed by deafult when the initial setup was done earlier. In our case, when using a web browser, we can access the HMIS instance of DHIS2 by going to our example URL of ' dhis.moh.xxx.zz/hmis ', or our Tracker instance by navigating to ' dhis.moh.xxx.zz/tracker ', or our monitoring software by navigating to ' dhis.moh.xxx.zz/munin '. These pages are accessed through the ' proxy' container, which uses Apache Reverse Proxy to also serve the results back to the browser. In addition, the security certificate that allows for encrypted traffic has also been set up on this container. In our case, we have used a free certificate from an open authority called ' Let's Encrypt '. Using this certificate, all traffic to and from the server will have a root of 'https', meaning it is safe for the transmission of user names and other secure information like passwords, which will be encrypted.

As it is, if a user types the example URL ' dhis.moh.xxx.zz ', they would arrive at the Apache landing page shown in the earlier figure. A decision must therefore be made on whether to redirect this URL to a default DHIS2 instance, or to create a new custom landing page. This is a simple HTML page, that can be created externally and then uploaded to the ' proxy ' container.

**Creating a Custom Landing Page**

If we use a desktop application to create a custom HTML landing page, for instance we call it ' customindex.html ', and it is in the current folder in our terminal client. We would need to ensure we are not on the host server, and are on the local machine. To copy the file to the server, we would use the secure copy client (SCP). The command to run for copying the index page to the host would be (where ' bob ' is our example user from before):
```
scp -i location_of_your_private_key_file -P 877 customindex.html bob@dhis.moh.xxx.zz:/home/bob/customindex.html
```
The command above would copy the created custom page onto the host. Using commands from before, you can now log back onto the host. Having logged back onto the host, it will be important to copy the created custom page onto the ' proxy ' container. To do this, the following LXC command can be run:

```
sudo lxc file push customindex.html proxy/var/lib/www/html/index.html
```
**Redirecting to a Default DHIS2 Instance**

On the other hand, instead of having a custom landing page, a decision could be made to redirect all requests made to the index page, for example when ' dhis.moh.xxx.zz ' is typed in the browser, the user would be automatically redirected to the main HMIS instance at ' dhis.moh.xxx.zz/hmis '. In this case,
there is a need to edit the main Apache Proxy virtual host file. To open the file, there are two options, option 1: log in to the proxy container from the host as shown below:
```
sudo lxc exec proxy bash
```
Next, open the virtual host file using your favourite text editor, such as ' vi ', as shown below:

```
sudo vi /etc/apache2/sites-enabled/apache-dhis2.conf
```

The second option is that you combine the two commands above while still in the host server into one as below:

```
sudo lxc exec proxy -- vi /etc/apache2/sites-enabled/apache-dhis2.conf
```
Having done this using either of the approaches above, it will now be possible to edit the host file to automatically redirect the traffic to the host onto to the default DHIS2 instance. The lines to be edited in the virtual host file are the ones shown below:
```
#===========================================================
# Rewrite requests for / to main dhis application
#===========================================================
```
```
# RewriteRule ^/$ /dhis/ [R]
```
The ' RewriteRule ' is the one that determines which page will be the default. In our case, since we would like the redirect to go the 'hmis' instance, we would edit the rule as follows (please remember
to uncomment first):
```
RewriteRule ^/$ /hmis/ [R]
```
Once this has been done, and the file saved, it will be important to reload the Apache server. To do this from the container itself you would run:
```
sudo service apache2 reload
```
Or from the host server:
```
sudo lxc exec proxy -- service apache2 reload
```
Over time, with experience, it will be possible to edit the host file for other tasks such as when a new container is added etc.

**Munin Monitoring Container**

It is important to setup a monitoring agent on your DHIS2 instance. If you have run the standard setup you will have installed a monitor called munin in a container called monitor.

To enable detailed monitoring of your DHIS2 tomcat application, you should run:
```
sudo dhis2-tomcat-munin <instance_name> proxy
```
For example, in our case, to moitor the ' hmis ' instance, we would run:
```
sudo dhis2-tomcat-munin hmis proxy
```
To access the console, you would need to go to our example URL at 'https://dhis.moh.xxx.zz/munin' or rather '<_server_name_>/_munin_' where the '_server_name_' is the home of your server.

# Part III. Backup and Disaster Recovery

# Chapter 6. Backups

***Database copy

First, enter the postgres container
```
lxc exec postgres bash
```

You perform the backup by running the following command, which also places the file in the ```tmp``` folder
```
pg_dump -O -Fp name_of_db -T aggregated* -T analytics_* -T completeness* | gzip > /tmp/name_of_output_file.gz
```
Exit the container, and pull the file (copy) using the command
```
lxc file pull postgres/tmp/name_of_output_file.gz /tmp/name_of_output_file.gz
```

Exit the host, and copy from the server to your local machine
```
scp -P port_number url_of_source_server:/tmp/name_of_output_file.gz local_file_location/name_of_output_file.gz
```
Copy from your local machine to the destination
```
scp -P port_number local_file_location/name_of_output_file.gz bob@url_of_destination_server:/tmp/name_of_output_file.gz
```
Access the machine where the databse was copied to, and stop the container running the instance whose databse you want to restore e.g.
```
lxc stop name_of_instance
```
Run the command from the dhis2 tools to restore the database:
```
sudo dhis2-restoredb /tmp/name_of_output_file.gz name_of_instance
```
Restart the container when it is done
```
lxc start name_of_instance
```

***LXD snapshots

To backup, you can take snapshots of the LXD containers. A snapshot is a 'picture' of the state of a container at the time it was taken, including its file systems. To take a snapshot of a container, you simply run the following command, for instance taking the snapshot of the *dhis* containter we would run:
```
lxc snapshot <container>
```
For example
```
lxc snapshot dhis
```
When this is run, a snapshot of the container is created with a given name of snapX where X is an incrementing number, e.g snap0.

Preferrably, you might want to give your snapshot a specific name, not the default *snapX* above. In that case you would need to specify the name of the snapshot as shown below:
```
lxc snapshot <container> <snapshot name>
```
For example to create a snapshot called *dhistest* I would run
```
lxc snapshot dhis dhistest
```
To restore a snapshot of the container you run
```
lxc restore <container> <snapshot name>
```
For example to restore the *dhis* container from the *snap0* snapshot you presumably created from a previous step you would run

```
lxc restore dhis snap0
```

***Scheduling auto snapshot creation and expiry

*TODO

***Exporting LXC containers 

Snapshots are useful in a backup strategy, however this  does not save you from something bad happening to your LXD host. If the host drive gets corrupted, then snapshots will not help you recover your containers. In this case, there is a need to export the container into a known file system,  save it on an external storage medium or ship it to another server, eg for remote/off-site backups. 

Shut down the container

```
lxc stop <container>
```

Export the container
```
lxc export <container> <storage name>
```
E.g
```
lxc export dhis /tank/backups/dhis0.tar.gz
```


