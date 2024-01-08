# DNS Sinkhole
Why dis?

DNS can really be a pain in the ass for IR. It's hard to control the communication of systems due to its nature and it is rarely monitored.

No wonder your favourite threat actor loves DNS tunneling more than having a long walk at the beach or pizza.

To quickly get control over DNS traffic you can use AdGuard to 

a) Whitelist traffic to prevent DNS Tunneling

b) Monitor the DNS traffic

**This is intended for quick deployment in incidents and not a recommendation for general integration in your production network!**

# Where to place the Sinkhole?
The sinkhole should be placed as close to the internet as possible. This will allow you to resolve internal services as usual while having control over internet communication.

![[Pasted image 20240108143610.png]]
# Quick Setup
I suggest taking a Ubuntu. Install Docker and Docker Compose on it.

## Disable DNSStubListener

create adguardhome.conf here:


    /etc/systemd/resolved.conf.d/adguardhome.conf


with this content:


    [Resolve]
    DNS=127.0.0.1
    DNSStubListener=no


save the corrent resolv.conf


    mv /etc/resolv.conf /etc/resolv.conf.backup


activate the new one


    ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf


restart resolved service


    systemctl reload-or-restart systemd-resolved


## set up AdGuard

get it running:


      git clone https://github.com/cgosec/DNS-Sinkhole
      cd DNS-Sinkhole
      docker-compose up -d
      docker-compose stop


*if you don't want to use the prepared config you can leave the container running and just configure the AdGuard as needed*

now copy the prepared config file 


      cp AdGuardHome.yaml data/confdir/
      docker-compose start


the predefined credentials in the config are:


      admin  
      This$$is"aN1cePw00rNuut?


access the dashboard at **http://your_ip:8083**
if you activate the wieguard vpn in the docker-compose file, I sugges you out comment the port mapping to 8083 and 4443 and access the dashboard only from the vpn. 


## Change DNS Upstream Server:

edit the AdGuardHome.yaml


      nano data/confdir/AdGuardHome.yaml


add your DNS upstream server here (you can remove the others):

    ...
    upstream_dns:
      - YOUR DESIRED DNS SERVER IP HERE
      - 8.8.4.4 
      - https://dns10.quad9.net/dns-query
    ...


## Adding urls to the whitelist

edit the AdGuardHome.yaml


      nano data/confdir/AdGuardHome.yaml


add you whitelisted urls over the implicid block **'/.*/'** regex
for a sub domain whitelist wildcard use '@@||YOUR_DOMAIN.com'

      ...
      user_rules: 
        - '@@||digicert.com'
        - '@@||lanner-lion.cloudsink.net'
        - '/.*/'
      ...

*This is preconfigured to allow communication to CrowdStrike Falcon EDR - change as you need*

**you can check in the logs and dashboard for blocked DNS request to identify malicious traffic or troubleshooting for services you want to check**
  
