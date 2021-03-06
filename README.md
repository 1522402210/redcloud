# Redcloud
![](https://img.shields.io/badge/Python-3+-brightgreen.svg) [![](https://img.shields.io/badge/Usable_Templates-35-brightgreen.svg)](https://github.com/khast3x/redcloud/blob/master/nginx-templates/templates.yml) ![](https://img.shields.io/github/issues-raw/khast3x/redcloud.svg?style=social)

*Weather report. Cloudy with a chance of shells!*  

Early release. Follow me on [Twitter](https://twitter.com/kh4st3x) to stay updated on Redcloud's development :information_desk_person::cloud::shell::seedling:

___

## Introduction

Redcloud is a powerful and user-friendly toolbox for deploying a fully-featured **Red Team Infrastructure** using Docker. Use and manage it with its [polished web-interface](#screenshots).  

Ideal for your penetration tests, shooting range, red teaming and bug-bounties!  
  
  
Self-host your attack infrastructure painlessly, deploy your very own live, scalable and resilient offensive infrastructure in a matter of minutes.

___

**Briefly,**

`redcloud.py` deploys a [Portainer](https://www.portainer.io/) stack, **pre-loaded with many tool templates for your offensive engagements**, powered by Docker. Once deployed, use the [web-interface](#screenshots) to manage it. Easy remote deploy to your target server using system `ssh` or even `docker-machine` if that's your thing.
  

* :rocket: Ever wanted to spin up a Kali in a cloud with just a few clicks?  
* :package: Have clean silos between your tools, technics and stages?  
* :ambulance: Monitor the health of your scans and C2?  
* :fire: Skip those sysadmin tasks for setting up a phishing campaign and get pwning faster?  
* :smiling_imp: Curious how you would build *the* ideal attack infrastructure?


Use the Web UI to monitor, manage, and **interact with each container**. Use the snappy web terminal just as you would with yours. Create volumes, networks and port forwards using Portainer's simple UI.

Use all your favorite tools and technics with the power of data-center-grade internet.

___

* :book: **Table of content**
  - [Redcloud](#redcloud)
    - [Introduction](#introduction)
    - [Features](#features)
    - [Quick Start](#quick-start)
    - [Details](#details)
      - [Redcloud Architecture](#redcloud-architecture)
      - [Networks](#networks)
      - [Volumes](#volumes)
      - [Accessing files](#accessing-files)
      - [SSL Certificates](#ssl-certificates)
      - [Stopping Redcloud](#stopping-redcloud)
      - [Portainer App Templates](#portainer-app-templates)
      - [Redcloud security considerations](#redcloud-security-considerations)
    - [Tested deployment candidates](#tested-deployment-candidates)
    - [Troubleshooting](#troubleshooting)
    - [Use-cases](#use-cases)
    - [Screenshots](#screenshots)
    - [Contribution guideline](#contribution-guideline)
    - [Hosting Redcloud](#hosting-redcloud)
    - [Inspirations & Shout-outs](#inspirations--shout-outs)

___


## Features

* Deploy Redcloud locally and remotely using the builtin SSH functions, and even docker-machine
* Deploy Metasploit, Empire, GoPhish, a fully stacked Kali, and many other security tools using Portainer's sleek and responsive web-interface
* Comes with an NGINX reverse-proxy pre-configured for Metasploit and Empire reverse callbacks *(finishing implementation)*
* Monitor and manage your infrastructure with just a few clicks
* Use the cloud's full potential with Docker's underlying power. Easily manage a single server or a full swarm just the same. Distribute the load seamlessly
* Painless network management and volume sharing
* Deploy redirections, socks or Tor proxy for all your tools. Spawn your maze, Docker's internal network capabilities takes care of the complicated stuff
* User and password management
* Comes with training environments. Build your own shooting range
* Overall very comfy :hatching_chick:

___

## Quick Start


```bash
# If deploying using ssh
> cat ~/.ssh/id_rsa.pub | ssh root@your-deploy-target-ip 'cat >> .ssh/authorized_keys'

# If deploying using docker-machine, and using a machine named "default"
> eval (docker-machine env default)
```

```bash
> git clone https://github.com/khast3x/redcloud.git
> cd redcloud
> python --version
# Use python3 if default python version is 2.x
> python redcloud.py
```

The Redcloud menu offers 3 different deployment methods:
1. **Locally**
2. **Remotely, using ssh**. Requires having your public key in your target's `authorized_keys` file
3. **Remotely, using docker-machine**. Run the `eval (docker-machine env deploy_target)` line to pre-load your env with your docker-machine, and run `redcloud.py`. Redcloud should automatically detect your docker-machine, and highlight menu items relevant to a docker-machine deploy

**Redcloud deployment workflow is as follows:**
1. Clone/Download Redcloud repository
2. Launch `redcloud.py`
3. Choose deployment candidate from menu (local, ssh, docker-machine)
4. `redcloud.py` automatically:
   1.  checks for `docker` & `docker-compose` on target machine
   2.  installs `docker` & `docker-compose` if absent
   3.  deploys the web stack on target using `docker-compose`
5. Once deploy is complete, `redcloud.py` will output the URL. Head over to https://your-deploy-machine-ip/portainer.
6. Set username/password from the web-interface 
7. Select the endpoint (the only one on the list)
8. Access the templates using the "App Templates" menu item on the left :rocket:

**App Template deployment is as follows:**
1. Choose template
2. If you wish to add additional options, select "+ Show advanced options"
3. Add port mapping, networking options, and volume mapping as you see fit
4. Select "Deploy the container"
5. Portainer will launch the container. It may take a few minutes if it needs to fetch the image. If your server is in a data-center, this step will be very fast.
6. Container should be running :rocket:
7. Portainer will redirect you to the "Containers" page. From there, you can:
   1. View live container logs
   2. Inspect container details (`docker inspect`)
   3. View live container stats (memory/cpu/network/processes)
   4. Use a web shell to interact with your container
   5. Depending on the App Template, uses either `bash` or `sh`. Choose accordingly from the drop-down menu

___

## Details

### Redcloud Architecture

* `redcloud.py`: Starts/Stops the Web interface and App Templates, using Docker and Portainer
* `portainer-app`: The main container with the Portainer web-interface
* `portainer-proxy`: NGINX reverse-proxy container to the web-interface. Can proxy Metasploit and Empire reverse shells. Auto-generates an https certificate
* `nginx-templates`: NGINX server container that feeds the App Templates. Lives in a "inside" network
* `redcloud_cert_gen_1`: The [omgwtfssl](https://github.com/paulczar/omgwtfssl) container that generates the SSL certificates using best practices
* https://your-server-ip/portainer: Redcloud Web interface once deployed


### Networks

Redcloud makes it easy to play around with networks and containers.  
You can create additional networks with different drivers, and attach your containers as you see fit. Redcloud comes with 2 networks, `redcloud_default` and `redcloud_inside`

### Volumes

You can share data between containers by sharing volumes. Redcloud comes with 2 volumes:

* `certs`: Container with the certificates generated by [omgwtfssl](https://github.com/paulczar/omgwtfssl)
* `files`: Standard file sharing volume. For now, the files are available when browsing https://your-server-ip/, and are served by the NGINX reverse-proxy container directly from the `files` volume. A typical use-case is to attach the volume to a Metasploit container, generate your payload directly into the `files` volume. You can now serve your fresh payload directly through the NGINX file server.

### Accessing files

Please refer to the `files` volume.

### SSL Certificates

Redcloud generates a new *unsigned* SSL certificate when deploying.  
The certificate is generated by [omgwtfssl](https://github.com/paulczar/omgwtfssl), implementing most best practices.
Once generated:
> It will dump the certs it generated into /certs by default and will also output them to stdout in a standard YAML form making them easy to consume in Ansible or other tools that use YAML. 

Certificates are stored in a shared docker volume called `certs`. Your containers can access this volume if you indicate it in "+ Advanced Settings" when deploying it. The NGINX reverse-proxy container fetches the certificates directly from its configuration file. If you wish to replace these certificates with your own, simply replace them from this volume.

It also means you can share the generated certificates into other containers, such Empire or Metasploit for your reverse callbacks, or for a phishing campaign.
Most SSL related configurations can be found in the `docker-compose.yml` file or `nginx/config/portainer.conf`


### Stopping Redcloud

You can stop Redcloud directly from the menu.  
**Deployed App templates need to be stopped manually before stopping Redcloud.** You can stop them using the Portainer web-interface, or `docker rm -f container-name`.  
If you wish to force the Portainer containers running Redcloud to stop, simply run `docker-compose kill` inside the `redcloud/` folder.
The *local* and *docker-machine* stop option is the same, thus they are combined in the same option.


### Portainer App Templates

Redcloud uses Portainer to orchestrate and interface with the Docker engine. Portainer in itself is a fantastic project to manage Docker deployments remotely. Portainer also includes a very convenient [template system](https://portainer.readthedocs.io/en/stable/templates.html), which is the major component for our Redcloud deployment.  
Templates can be found in `./nginx-templates/templates.yml`. Portainer fetches the template file from a dedicated NGINX container (`nginx-templates`).


### Redcloud security considerations

Redcloud deploys with a self-signed https certificate, and proxies all interactions with the web console through it.  
However, your containers will expose ports. The default network exposes them to the outside world.

You can:
* Add custom `Location` blocks in the NGINX configuration
* Start a Ubuntu+noVNC (VNC through http) from template, add it to both a "inside" and "outside" network, and access exposed interfaces from inside
* Add .htaccess configurations. Some are planned in further Redcloud development

Additionally:  
* `docker` & `docker-machine` installation require root privileges. You can downgrade privilege requirements following [the official documentation](https://docs.docker.com/install/linux/linux-postinstall/)
* The install script is pulled directly from the official docker repositories
* `redcloud.py` fetches Redcloud's public IP address using icanhazip.com

___

## Tested deployment candidates

| Deploy Target |        Status       |
|:-------------:|:-------------------:|
| Ubuntu Bionic | :heavy_check_mark: |
| Ubuntu Xenial | :heavy_check_mark: |
| Debian Strech | :heavy_check_mark: |

___

## Troubleshooting

* Check your default python version with `python --version`. Redcloud needs python 3+
* Use `python3` instead of `python` if on an older system
* `redcloud.py` requires that deployment candidate have the public key in their `.ssh/authorized_keys`, and handles password-less authentication using the user's public key. This is the default configuration for most VPS workflows.
* docker-machine deployment requires that the user already have a running docker-machine on a cloud infrastructure (such as AWS, GCP, Linode and [many others](https://docs.docker.com/machine/drivers/)). Once deployed, simply run the `eval` command as illustrated above
* docker & docker-machine installation require root privileges. You can downgrade privilege requirements following [the official documentation](https://docs.docker.com/install/linux/linux-postinstall/)
* If you don't see the "App Templates" menu item right after deploying, refresh the web page and make sure you're not at the endpoint selection menu
* If you wish to create a new username/password combo, remove Portainer persistent data on deployment candidate: `rm -rf /opt/portainer/data`
* If you're running into python errors, you may need to install the `python3-distutils` package (use `apt-get install python3-distutils` on debian/ubuntu base)
* If you're getting an error when deploying an App Template saying the "container name already exists", it's probably because you're trying to deploy the same App Template without having removing a previously deployed one. Simply remove the old container with the same name, or change the name of your new container.
* If something seems wrong with your container, the standard procedure is to check the container's logs from the web-interface.


___

## Use-cases

* Create your personal pentest-lab, and practice your hacking skills with friends and colleagues 
* Deploying Metasploit or Empire, generate payload, serve with nginx files
* Launch Sniper, fetch logs using nginx files
* Use the reverse proxy to cover metasploit or empire
* Use an xss scanner on Juice shop
* Launch scans behind your own tor socks proxy
* View .onion site using Tor Socks + Ubuntu VNC
* Advanced OSINT with Spiderfoot and a Tor container as proxy
* Throw off the blue team by deploying honeypots. Can be one or one thousand honeypots thanks to containers!

___

## Screenshots

* Template List + `redcloud.py` deploy
![](https://i.imgur.com/j8YFjtw.png)

* Deploying a container
![](https://i.imgur.com/QCR1yHp.png)


* Using Metasploit's `msfconsole` through the web-interface

![](https://i.imgur.com/wUcFHbh.png)


___

## Contribution guideline

Any help is appreciated. This is a side project, so it's probably missing a few bolts and screws. Above all:
* Reporting or fixing Redcloud bugs & quirks
* Adding templates. Please keep it clean, and from the creator's docker hub repository if possible
* Adding documentation
* Detailing use-cases in blog articles. I'll add links to blog posts here, so please be sure to contact me if you make one! :v:
* Typos as issues *(no pull requests please)*

___

## Hosting Redcloud

You can host a Redcloud on any unix server that runs Docker.  
Redcloud is intended to be used in a cloud environment, such as a simple VPS with ssh, or even a AWS EC2, GCP etc...

A large range of cloud vendors offer **free credits** to get familiar with their services. Many lists and tutorials cover getting free hosting credits from major vendors. [This list is a good place to start](https://github.com/ripienaar/free-for-dev#iaas).

Regarding deployment method, I personally prefer working with [docker-machine](https://docs.docker.com/machine/overview/) as it becomes ridiculously easy to spawn new machines and manage them once you've got your cloud provider's [driver setup](https://docs.docker.com/machine/drivers/). If you prefer using `ssh`, be sure to take a look at evilsocket's [shellz](https://github.com/evilsocket/shellz) project to manage your keys and profils.

___

## Inspirations & Shout-outs

* [Red Team Infrastructure Wiki](https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki) - bluscreenofjeff
* [Automated Red Team Infrastructure Guide](https://rastamouse.me/2017/08/automated-red-team-infrastructure-deployment-with-terraform---part-1/) - rastamouse
* [Safe Red Team Infrastructure](https://medium.com/@malcomvetter/safe-red-team-infrastructure-c5d6a0f13fac) - Tim MalcomVetter
* [Red Baron](https://github.com/Coalfire-Research/Red-Baron) - Coalfire Research
* [Rapid Attack Infrastructure](https://github.com/obscuritylabs/RAI) - Obscurity Labs
* [Decker](https://github.com/stevenaldinger/decker) - Steven Aldinger
* [HideNSeek](https://github.com/rmikehodges/hideNsneak) - Mike Hodges

___

*Finally, I'd love to integrate Cobalt Strike. Unfortunately, I don't see myself having the funds to invest in a licence, so if you know someone who knows someone, I'm all ears* :innocent:
___

*If you wish to stay updated on this project:*

[![twitter](https://i.imgur.com/S79Nimd.png)](https://twitter.com/kh4st3x)
