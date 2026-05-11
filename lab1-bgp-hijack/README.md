## How the Hijack Works

1. AS100 legitimately announces `10.0.0.0/24` (256 addresses)
2. AS300 maliciously announces `10.0.0.0/25` (128 addresses)
3. BGP always prefers MORE SPECIFIC routes
4. So `/25` beats `/24` — traffic goes to AS300 instead of AS100!
5. This is exactly how the 2008 Pakistan Telecom hijacked YouTube! 🌍

## Setup

### Requirements
- Docker
- Containerlab

### Deploy

```bash
# Deploy the lab
sudo containerlab deploy -t bgp-hijack.yml

# Enable BGP on all containers
sudo docker exec clab-bgp-hijack-AS100 sed -i 's/bgpd=no/bgpd=yes/' /etc/frr/daemons
sudo docker exec clab-bgp-hijack-AS200 sed -i 's/bgpd=no/bgpd=yes/' /etc/frr/daemons
sudo docker exec clab-bgp-hijack-AS300 sed -i 's/bgpd=no/bgpd=yes/' /etc/frr/daemons

# Restart FRR
sudo docker exec clab-bgp-hijack-AS100 /usr/lib/frr/frrinit.sh restart
sudo docker exec clab-bgp-hijack-AS200 /usr/lib/frr/frrinit.sh restart
sudo docker exec clab-bgp-hijack-AS300 /usr/lib/frr/frrinit.sh restart
```

### Configure BGP

```bash
# AS100 - Legitimate Owner
sudo docker exec clab-bgp-hijack-AS100 vtysh -c "
configure terminal
router bgp 100
bgp router-id 1.1.1.1
no bgp ebgp-requires-policy
neighbor 172.20.20.3 remote-as 200
address-family ipv4 unicast
neighbor 172.20.20.3 activate
network 10.0.0.0/24
exit-address-family
end
write memory
"

# AS200 - Transit Router
sudo docker exec clab-bgp-hijack-AS200 vtysh -c "
configure terminal
router bgp 200
bgp router-id 2.2.2.2
no bgp ebgp-requires-policy
neighbor 172.20.20.2 remote-as 100
neighbor 172.20.20.4 remote-as 300
address-family ipv4 unicast
neighbor 172.20.20.2 activate
neighbor 172.20.20.4 activate
exit-address-family
end
write memory
"

# AS300 - Evil Hijacker
sudo docker exec clab-bgp-hijack-AS300 vtysh -c "
configure terminal
router bgp 300
bgp router-id 3.3.3.3
no bgp ebgp-requires-policy
neighbor 172.20.20.3 remote-as 200
address-family ipv4 unicast
neighbor 172.20.20.3 activate
network 10.0.0.0/25
exit-address-family
end
write memory
"
```

## Verify the Hijack

```bash
# Add networks to loopback interfaces
sudo docker exec clab-bgp-hijack-AS100 ip addr add 10.0.0.1/24 dev lo
sudo docker exec clab-bgp-hijack-AS300 ip addr add 10.0.0.1/25 dev lo

# Check BGP table on AS200
sudo docker exec clab-bgp-hijack-AS200 vtysh -c "show ip bgp"

# You should see:
# 10.0.0.0/24 via AS100 (legitimate)
# 10.0.0.0/25 via AS300 (hijack!)
```

## Key Learning

> BGP has NO built-in security!
> Any router can announce any prefix!
> This is why RPKI was invented — see Lab 2!
