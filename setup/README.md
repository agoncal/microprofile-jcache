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

# Setup the Raspberry PI Cluster 

For this demo, the cluster is setup as follow:

* 3 Raspberry PIs acting as clients browsing the 2 data centers (`pi-client-01`, `pi-client-02` and `pi-client-03`) 
* 1 Load balancer for the clients (`pi-load-balancer`)
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
- `flash --hostname pi-load-balancer hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-load-balancer hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-server-01 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-server-02 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-grom-database hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-load-balancer hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-server-01 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-server-02 hypriotos-rpi-v1.5.0.img`
- `flash --hostname pi-thrall-database hypriotos-rpi-v1.5.0.img`

## Setup the Router

These instructions are for the [Edge Router X](https://www.ubnt.com/edgemax/edgerouter-x/). [Setup a static IP](https://www.youtube.com/watch?v=p2rQRvNutvY) to 
connect to the router (eg. [http://192.168.1.1]()). If you can't reach the router, give it a *reset*, sometimes it 
helps.

- Login into the router with user/pwd `ubnt` / `ubnt`.
- Setup the DHCP server: Go to Wizards / WAN + 2LAN2 / LAN Ports (eth1, eth2, eth3, and eth4) / (Address [10.99.99.1]() 
/ [255.255.255.0]()). This will reboot the router.
- Map PI's MAC Addresses to static IPs : Go to Services / LAN / View Details / Leases / Map Static IP
- Go to Configuration / dhcp-server / hostfile-update and set to "enable". (This will register the PI's hostnames in the hosts files and they will be reachable via DNS)
- Set manual hostname for local docker registry and elk (own box - using 10.99.99.11)
    - Login into the router with ssh and ubnt / ubnt and type the commands:
        - configure
        - set system static-host-mapping host-name docker-repo inet 10.99.99.11
        - set system static-host-mapping host-name pi-elastic-01 inet 10.99.99.11
        - commit
        - save
        - exit

* [Documentation and firmware](https://www.ubnt.com/download/edgemax/edgerouter-x/er-x)

## Setup Docker Repo Registry
Learn how to setup a local Docker Registry: https://docs.docker.com/registry/deploying/

To generate the certificates:
- openssl req -newkey rsa:4096 -nodes -sha256 -keyout docker-repo.key -x509 -days 365 -out docker-repo.crt

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
- http://pi-load-balancer:5000/stats (HA Proxy Stats)
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

![front](https://raw.githubusercontent.com/tomitribe/microprofile-jcache/master/setup/clusterFront.jpg)

## Back of the cluster

![back](https://raw.githubusercontent.com/tomitribe/microprofile-jcache/master/setup/clusterBack.jpg)

