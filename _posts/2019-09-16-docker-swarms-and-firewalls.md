---
layout: post
title:  "Docker Swarm & Firewalls"
date:   2019-09-16 19:00:00 +0800
tags: [docker]
navigation: true
---
Recently, I have been using [Docker](https://docs.docker.com/engine/swarm/) to deploy a set of web-based systems to a cluster of production servers. This is easy to do with [swarm mode](https://docs.docker.com/engine/swarm/), which allows a group of Docker Engines to be pulled together into a cluster, called a swarm. However, as I found out, swarm mode is not so great when faced with a firewall and communication between nodes can easily be blocked if the firewall is not configured correctly.

The Docker documentation states that a set of ports should be open to allow the swarm to function:

> The following ports must be available. On some systems, these ports are open by default.

> * TCP port 2377 for cluster management communications
> * TCP and UDP port 7946 for communication among nodes
> * UDP port 4789 for overlay network traffic
>
> [Docker Engine swarm tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)

Further reading about how to configure the firewall, such as this [DigitalOcean guide for Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-ubuntu-16-04), shows that these ports should be made open on the INPUT chain of iptables. However, when I configured the firewall to allow these ports and set the policy for INPUT as DROP, communication between the nodes stopped.

This was a serious problem as it prevented new nodes being added to the swarm and often caused the swarm to become destablised, requiring it to be rebuilt. Looking into why the nodes could not communicate, I found that Docker Engine was trying to connect to different ports than the ones stated in the docs. Using netstat, these ports all seemed to be in the range 30000 - 32000, and the closest information I could find in the Docker docs was relating to load balancing in the swarm:

> The swarm manager uses ingress load balancing to expose the services you want to make available externally to the swarm.
> The swarm manager can automatically assign the service a PublishedPort or you can configure a PublishedPort for the service.
> You can specify any unused port.
> If you do not specify a port, the swarm manager assigns the service a port in the 30000-32767 range.
>
> [Docker Engine swarm key concepts](https://docs.docker.com/engine/swarm/key-concepts/)

But, this does not explain why nodes could not join the swarm or communicate, and a quick Google search shows that a lot of others experienced similar issues with Docker Engine.

An easy solution would be to open the firewall, however, this would expose many services. Therefore, I had to find a way to allow communication between the nodes of the swarm to occur, while also limiting the number of externally open ports.

In this situation, the nodes in the swarm can be trusted and, as a result, communication between the nodes can be free. Therefore, the firewall should allow all inbound communication from the nodes and restrict other inbound traffic. To implement this, I used the following tools: iptables-persistent; ipset; and, iptables.

First, I installed 'iptables-persistent' to store the rules, and flushed any existing rules to prevent conflicts from occurring.

```bash
apt install iptables-persistent

netfilter-persistent flush
```

Next, I used ipset to create a list of IP addresses, which is used later on when creating the rules in iptables.

```bash
apt install ipset

# Create a set
ipset -N <ipset name> iphash

# Add an IP address to the set
ipset -A <ipset name> <ip address>
```

After that, I used iptables to create the rules for the set of IP addresses created above in the firewall. These rules allow all TCP and UDP traffic for those specific IP addresses, thus, allowing the nodes to communicate freely.

```bash
iptables -A INPUT -m set --match-set <ipset name> src -p tcp -j ACCEPT
iptables -A INPUT -m set --match-set <ipset name> src -p udp -j ACCEPT
```

In addition, a few extra ports are needed to allow communication with Docker Hub. Firstly, Port 53/UDP is needed so that domains can be resolved, while port 80/TCP and 443/TCP are needed so that the Docker images can be pulled to the nodes. Furthermore, port 22/TCP is required for SSH sessions and inbound communication for established connections is also needed.

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

> **Danger Zone!** Make sure inbound traffic to port 22 is allowed, otherwise communication with the server will be lost at the next step.

As the final step, I dropped all other inbound traffic to the node, saved the rules, and restarted Docker so that it can configure itself with the new rules.

```bash
iptables -P INPUT DROP

netfilter-persistent save

service docker restart
```
The firewall is now all set, and all nodes can communicate in the swarm, while external traffic is controlled.
