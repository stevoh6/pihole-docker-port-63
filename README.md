# pihole-docker-port-63
this is workeround how to separate Tenda nova devices (or any other periodically 'calling home' device) from normal pihole instance (pihole53) without touching pihole53.

We will create new instance of pihole (pihole63) listening on port 63 (web admin on port 81) and add some iptables rules in server (where is pihole53 running)

Lets asume that we ould like to separate DNS queries for following IPs (192.168.100.6,192.168.100.7) and server's network interface is eth0

## Step 1 ( create new pihole instance)
- create pihole63 folder (for example /var/docker-containers/pihole63 ) `mkdir –p /var/docker-contianers/pihole63`
- `cd /var/docker-contianers/pihole63`
- download docker-compose.yml `curl https://raw.githubusercontent.com/stevoh6/pihole-docker-port-63/master/docker-compose.yml -o docker-compose.yml`
- adjust docker-compose file by you needs (choose password, adjust web port, etc.)
- run `docker-compose up -d`

## Step 2 (add some rules)
- uncomment `sysctl -w net.ipv4.ip_forward=1` in sysctl `vi /etc/sysctl.conf`
- verify changes `sysctl -p`
- add following rule into iptables for ip 192.168.100.6 `iptables -t nat -D PREROUTING -i eth0 --src 192.168.100.6 -p udp --dport 53 -j REDIRECT --to-port 63` 
- add following rule into iptables for ip 192.168.100.7 `iptables -t nat -D PREROUTING -i eth0 --src 192.168.100.7 -p udp --dport 53 -j REDIRECT --to-port 63`
- verify changes `iptables -t nat -L -v`

## DONE. 
from now every DNS query comming from IPs () will be handled by pihole63.
For example you can block it or keep it alive and it will not break usefull statistics of normal pihole53 instance.


https://www.reddit.com/r/pihole/comments/cvon58/tenda_mw6_calling_home_very_often/

https://superuser.com/questions/1409745/tenda-mw6-mesh-is-talking-to-baidu-what-is-it-doing

https://www.amazon.co.uk/ask/questions/TxSL1F0KXZPPOV/ref=ask_ql_ql_al_hza

https://www.reddit.com/r/pihole/comments/dqau7q/pihole_has_detected_major_privacy_concern/

https://lokilabs.io/tenda-is-this-bad-design-or-a-backdoor/
