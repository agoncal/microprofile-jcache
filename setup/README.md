# Setup

This document tells you how to setup the Raspberry PI Cluster which simulates 2 data centers and clients accessing 
those two data centers thanks to the router.

# Prerequisites

## Hardware

The cluster for this demo uses: 

* 12 x [Raspberry PIs V3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/). Each datacenter is made of 4
 Raspberry PIs, as well as 4 Raspberry PIs for the clients 
* 12 x [Raspberry PI protective case](https://www.amazon.com/Raspberry-Clear-Protective-Heatsinks-Screwdriver/dp/B071YM8QVK/ref=sr_1_25) in different colours, it's better 
* 12 x SD cards (8 Mb is enough)
* 3  x [5-Port Desktop Switch TL-SG105](http://www.tp-link.com/us/products/details/cat-42_TL-SG105.html) to create a 
network between the 2 data centers and the clients.
* 12 x [6 inch ethernet cables](https://www.amazon.com/iMBAPrice-Ethernet-Network-Patch-Cable/dp/B00A1UPGY4/ref=sr_1_1) to plug each Raspberry PI to the switch 
(different colours is better)
* 1  x [Edge Router](https://www.ubnt.com/edgemax/edgerouter-x/) to route clients between the 2 data centers
* 3  x [one foot ethernet cables](https://www.amazon.com/Cable-Matters-5-Pack-Snagless-Ethernet/dp/B00C4U030G/ref=sr_1_1) to plug each switch to the router  
* 3  x [Zookki 6 Port USB Charger](http://www.zookki.com/category/Zookki-USB-Charger) to electrically power the 
Raspberry PIs
* 12 x [Retractable USB to Micro cable](https://www.amazon.com/Vktech-Micro-Retractable-Charger-Cable/dp/B00DQMHM14/ref=sr_1_13) from the charger to the Raspberry PI
* 2  x [Power strip with 6 sockets](http://www.belkin.com/us/F5C048-2-Belkin/p/P-F5C048-2/)
* [Reusable fastening cable tie](https://www.amazon.com/Reusable-Fastening-Cable-Straps-Wisdompro/dp/B01M1L1YHO/ref=pd_rhf_se_p_img_2) because you want to tie 
everything so it doesn't look like a mess
* 1 x [USB to SD Card Reader](https://www.amazon.com/Cateck-USB3-0-4-Slot-Reader-Micro/dp/B01J5651NA) to flash the SD
 cards

## Software

To setup the Raspberry PI Cluster you need the following tools to be installed:

* [Docker](https://www.docker.com/)
* [Ansible](https://www.ansible.com/) ([how to install it](https://valdhaus.co/writings/ansible-mac-osx/)
* The Hypriot [flash utility](https://github.com/hypriot/flash)
* (optional) [Pipe Viewer](http://brewformulas.org/Pv) to monitor the progress of data (handy when flashing SD cards)
* (optional) [SSHPASS](https://gist.github.com/arunoda/7790979)

# Setup the Physical Raspberry PI Cluster 

For this demo, the cluster is setup as follow:

* Clients
  * 3 Raspberry PIs acting as clients browsing the 2 data centers (`pi-client-01`, `pi-client-02` and `pi-client-03`) 
  * 1 Load balancer for the clients (`pi-client-load-balancer`)
* Data center [Thrall](https://en.wikipedia.org/wiki/Characters_of_Warcraft#Thrall) 
  * 1 load balancer (`pi-thrall-load-balancer`) 
  * 2 servers (`pi-thrall-server-01` and `pi-thrall-server-02`) 
  * 1 database (`pi-thrall-database`) 
* Data center [Grom](https://en.wikipedia.org/wiki/Characters_of_Warcraft#Grommash_Hellscream) 
  * 1 load balancer (`pi-grom-load-balancer`) 
  * 2 servers (`pi-grom-server-01` and `pi-grom-server-02`) 
  * 1 database (`pi-grom-database`) 

## Setup the Raspberry PIs

You need to install [HypriotOS](http://blog.hypriot.com/about/#hypriotos) on each SD card (that's 12 of them). 
HypriotOS is a minimal Debian-based operating systems that is optimized to run Docker on Raspberry PIs. For that, you
 need to flash Hypriot using the [flash utility]()https://github.com/hypriot/flash).

Insert the SD card, and use the following commands to flash each cards (notice that `--hostname` sets the hostname for 
the SD image):
- `flash --hostname pi-client-01 https://github.com/hypriot/image-builder-rpi/releases/download/v1.5
.0/hypriotos-rpi-v1.5.0.img.zip` # downloads and then flashes the card 
- `flash --hostname pi-client-02 hypriotos-rpi-v1.5.0.img` # doesn't need to download the image anymore, just flashes
- `flash --hostname pi-client-03 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-client-load-balancer hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-load-balancer hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-server-01 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-server-02 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-database hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-load-balancer hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-server-01 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-server-02 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-database hypriotos-rpi-v1.5.0.img`

## Setup the Router

For this sample, we use the [Ubiquiti EdgeRouter X](https://www.ubnt.com/edgemax/edgerouter-x/). 
It's a cheap router with a lot of features in it. Any other router should work just fine, but the instructions detailed 
here are for the EdgeRouter X.

### Reset to Factory Settings

If you can't access the Router or if you see that it is not responding correctly, that might be because of previous setups. Better reset it. For that:

* Power on the Ubiquiti Router, place a paper clip or Pin into the hole on the back of the Router labeled Reset.
* Hold paper clip or pin down for 10 to 15 seconds and release.
* The Router will reboot on its own. Once the WLAN light stops blinking, the Router is reset.

### Connect to the Router

Before configuring the router, we need to connect to it. For that:

* Plug your computer into `eth0` on the router.
* Configure your computer to have the static IP (eg. `192.168.1.3`). The Router IP is `192.168.1.1`.

![Static IP on Mac](router-connect-01?raw=true)

* Open a browser and navigate to `https://192.168.1.1`. You should get the login screen to access the Router console. 
(You may need to trust the certificate first).
* Login with Username `ubnt` and Password `ubnt`.

### Setup DHCP Server

The Router's DHCP Server will provide the Raspberry PIs dynamic IP addresses, as well as an Internet Connection (by sharing a home Internet Connection). For that:

* Go to Wizards / WAN + 2LAN2 to set up DHCP Server. Use a non conventional range like `10.99.99.1` to avoid clashing with 
other networks. The DHCP will assign IP's from `10.99.99.1` to `10.99.99.255`.

![WAN + 2LAN2](router-setup-dhcp-01?raw=true)

* Click `Apply` (then `Apply Changes` and `Reboot`). The Router should restart with the new settings.
* Unplug your computer from the Router `eth0` and plug it on another port (eg. `eth4`) 
* Connect your Home Internet on `eth0`.
* Remove the static IP from your computer configuration. The router should now assign you a Dynamic IP in the 
`10.99.99.x` range (eg. `10.99.99.38`). On Mac OS X you can force it by renewing the DHCP lease
* The Router is now available in `https://10.99.99.1`. 
* Login with Username `ubnt` and Password `ubnt`.
* At this moment in time, if the Raspberry PIs are switched on, after a few minutes, you should get dynamic IPs for all of them (Go to Services / LAN / Actions / View Leases).

![Dynamic IP Leases](router-setup-dhcp-02?raw=true)

### Map a Static IP for your computer

For ease of use, your computer should have a static IP address (and you can leave dynamic IP addresses for the Raspberry PIs). For that:

* Go to Services / LAN / Actions / View Leases
* Look for your computer (eg. with the IP `10.99.99.38`) and click on `Map Static IP` (you can even give it a different, like here, `agoncal`), click save

![Tuen Mac to Static IP](router-setup-staticip-01?raw=true)

You should see it in the Static MAC/IP Mapping tab

![Static IPs for Mac](router-setup-staticip-02?raw=true)

* (optional) If you wish, you can assign static IP's to your PI's and do the same action for the 12 Raspberry PIs  (not required, but useful if you want to make sure that you 
access the same PI with the same IP every time).

### Map a Static IP for the docker-repo

Your computer has now a static IP address. The Docker Repo (`docker-repo`) is also hosted on your computer, but it's a good practice to allow it a different host-name (even if it's the same IP address). For that:

* This operation can only be done in the CLI console.
* Assuming that your computer static IP address is `10.99.99.38`, type the following

```
ssh ubnt@10.99.99.1
Welcome to EdgeOS

By logging in, accessing, or using the Ubiquiti product, you
acknowledge that you have read and understood the Ubiquiti
License Agreement (available in the Web UI at, by default,
http://192.168.1.1) and agree to be bound by its terms.

ubnt@10.99.99.1's password:
Linux ubnt 3.10.14-UBNT #1 SMP Mon Nov 2 16:45:25 PST 2015 mips
Welcome to EdgeOS
Last login: Sun Sep  3 21:09:22 2017 from radcortez
ubnt@ubnt:~$ configure
[edit]
ubnt@ubnt# set system static-host-mapping host-name docker-repo inet 10.99.99.38
[edit]
ubnt@ubnt# commit
[edit]
ubnt@ubnt# save
Saving configuration to '/config/config.boot'...
Done
e[edit]
ubnt@ubnt# exit
exit
ubnt@ubnt:~$ exit
logout
Connection to 10.99.99.1 closed.
```

* You can check the result on the Router console
* Go to Config Tree / system / static-host-mapping / host-name

![Docker repo static IP](router-setup-staticip-03.png?raw=true)

### Map hostnames into the Router's Hosts file

At this moment, you can ping all the Raspberry PIs using their IP addresses (eg. `ping 10.99.99.39`) but not their name (eg. `ping pi-thrall-server-01`). 
The Router has a hosts files to resolve DNS names. To be able to resolve hostnames we need to activate a configuration. For that:
 
* Go to Config Tree / service / dhcp-server / hostfile-update, set to "enable", click on preview and apply.

![Enable host file update](router-hostname-01.png?raw=true)

* For this to be taken into account, reboot the router (either, electrically, or using the System sub-menu)

![Reboot 1](router-hostname-02.png?raw=true)
![Reboot 2](router-hostname-03.png?raw=true)

### Access the PIs with hostname

Now you should ping the hostnames (and not just the physical IP addresses). For that:
 
* `ping pi-thrall-server-01`, `ping ppi-client-01`
* /!\ if pinging a hostname does not work, try to reboot all your Raspberry PIs
* Connect to the router (`ssh ubnt@10.99.99.1` pwd `ubnt`) and check the hosts file (`cat /etc/hosts`)
* You should get someting similar to that (notice that `#on-dhcp-event` is for dynamic IP, and `#vyatta` for static)

``` 
127.0.1.1        ubnt    #vyatta entry
10.99.99.38      docker-repo             #vyatta entry
10.99.99.38      agoncal                 #on-dhcp-event c:4d:e9:cb:82:db
10.99.99.47      pi-load-balancer        #on-dhcp-event b8:27:eb:e5:19:5e
10.99.99.44      pi-grom-server-01       #on-dhcp-event b8:27:eb:f5:88:47
10.99.99.50      pi-client-01            #on-dhcp-event b8:27:eb:67:2d:84
10.99.99.48      pi-thrall-server-01     #on-dhcp-event b8:27:eb:af:ec:f0
10.99.99.43      pi-grom-server-02       #on-dhcp-event b8:27:eb:d7:e2:b4
10.99.99.45      pi-thrall-load-balancer #on-dhcp-event b8:27:eb:44:e7:58
10.99.99.46      pi-client-03            #on-dhcp-event b8:27:eb:f6:77:aa
10.99.99.41      pi-thrall-server-02     #on-dhcp-event b8:27:eb:d5:51:a6
10.99.99.40      pi-grom-load-balancer   #on-dhcp-event b8:27:eb:38:b5:3a
10.99.99.39      pi-client-02            #on-dhcp-event b8:27:eb:ff:d1:a4
10.99.99.42      pi-grom-database        #on-dhcp-event b8:27:eb:cb:ba:69
10.99.99.49      pi-thrall-database      #on-dhcp-event b8:27:eb:ff:45:4a
```
* You can now log on into your PIs with `ssh pirate@pi-thrall-load-balancer` (password `hypriot`)
* To make sure all the Rasperry PIs are up and running, use the Ansible command (make sure you `cd` into the `setup/ansible` directory, where the `hosts` file is located): 
  * `ansible all -m ping -i hosts` 
  * if you get "to use the 'ssh' connection type with passwords, you must install the sshpass program" run `brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb` 
  * if it still does not work, use `ansible all -m ping -i hosts -c paramiko` you should be prompted :
  
``` 
paramiko: The authenticity of host 'pi-thrall-database' can't be established.
The ssh-ed25519 key fingerprint is 6678299330f009a422c3aeadf9929da1.
Are you sure you want to continue connecting (yes/no)?
yes
```  

### Troubleshooting

If you try to connect to the router through ssh and have the following error:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:ZgD1JUiD6OKEyu3TBRc7EEsf67vYkRsydyxyEF3X03g.
Please contact your system administrator.
Add correct host key in /Users/antoniombp/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/antoniombp/.ssh/known_hosts:40
RSA host key for 10.99.99.1 has changed and you have requested strict checking.
Host key verification failed.
```

Just open the file `/Users/antoniombp/.ssh/known_hosts:40` and delete the line (here, line `40`). Save the file, and relog in.

# Setup the application on the cluster 

## Setup Docker Repo Registry

To avoid using the [Docker Hub](https://hub.docker.com/) to pull/push Docker images, we setup a local Docker 
repository (on the Mac). A registry stores and lets you distribute Docker images. This explains how to setup a 
[local Docker Registry](https://docs.docker .com/registry/deploying/). We will call it `docker-repo`.

First, generate the certificates:
- `openssl req -newkey rsa:4096 -nodes -sha256 -keyout docker-repo.key -x509 -days 365 -out docker-repo.crt`
- This creates two files (`docker-repo.crt` and `docker-repo.key`) that are installed under the directory 
`ansible/docker-repo/certs` (used by the `REGISTRY_HTTP_TLS_CERTIFICATE` variable)

Run the local Docker Registry:
- docker run -d -p 5000:5000 --restart=always --name docker-repo -v `pwd`/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker-repo.crt -e REGISTRY_HTTP_TLS_KEY=/certs/docker-repo.key registry:2

- The self signed certificates need to be added to the PI's so Docker can downloads images from the local Docker 
Registry. They also need to be added to the local Docker Registry host. For Macs, the easiest way is to add the address 
(docker-repo:5000), in the Mac Docker Daemon / Preferences / Daemon / Insecure Registries.

## Setup PI's
Run the following Ansible playbooks in order:
- ansible-playbook docker-repo/install-repo-certs.yaml -i hosts -f 12
- reboot PI's (docker needs a restart to use the certificate)
- ansible-playbook load-balancers/load-balancers.yaml -i hosts -f 3
- ansible-playbook install-tomee.yaml -i hosts -f 8
- ansible-playbook databases/install-mysql.yaml -i hosts -f 2

## Add Applications (Client and Server)
- Build client and server with ```mvn clean install docker:build -DpushImage``` (this should build and upload the docker
images to the local registry)
- ansible-playbook servers/servers.yaml -i hosts -f 4
- ansible-playbook clients/clients.yaml -i hosts -f 3

## Useful links:
- https://docker-repo:5000/v2/_catalog (Local Docker Registry Catalog)
- http://pi-client-load-balancer:5000/stats (HA Proxy Stats)
- http://pi-grom-load-balancer:5000/stats (HA Proxy Stats)
- http://pi-thrall-load-balancer:5000/stats (HA Proxy Stats)

## Setup Metrics
- To run Metrics, use Docker Compose to start an ElasticSearch, Logstash, Kibana environment.
- Execute ```docker-compose up -d elk```. Then you can just stop / start the container with ```docker stop elk``` and ```docker start elk``` 
- Add the template.json into ElasticSearch: ```curl http://pi-elastic-01:9200/_template/tribe_template -d @template.json```
- Add the initial Kibana index pattern. Go to Kibana / Settings / Indices / Index name or pattern = tribe-metrics-* and Time-field name = @timestamp
- Import the dashboards. Use the file ```visualizations.json``` and go to Kibana / Settings / Objects / Import

# What it looks like

## Front of the cluster

![front](clusterFront.jpg)

## Back of the cluster

![back](clusterBack.jpg)

