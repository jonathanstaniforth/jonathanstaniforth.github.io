---
layout: post
title:  "Docker Service Routing with Nginx"
date:   2019-10-12 12:00:00 +0800
tags: [docker,nginx]
navigation: true
---
Only one container or service deployed using docker can use a port on the host machine at any given time, meaning, each container has to use a different port to be accessed. But, in certain situations, we may want several containers to use the same port, for example when using domain names to access containers. Additionally, this problem grows as the scale and complexity of docker services and swarms increases.

To solve this problem, we can use nginx as a proxy, listening to a particular port and sending traffic onwards to the appropraite container or service. This post discusses how to do this in both a single stack and multi-stack scenario, with nginx listening on port 80.

## Single stack scenario
In this scenario, a docker swarm is setup and two services are available, both using port 80 internally in the container. The docker compose file for this stack looks like the below:

```yaml
version: "3.7"
services:
    frontend:
        image: frontend:latest
        init: true
    
    api:
        image: api:latest
        init: true
```

Now, the nginx service is introduced into the stack, binding to port 80. The service will be added to the compose file as an additional service like follows:

```yaml
    nginx:
        image: nginx
        init: true
        volumes:
            - ./conf.d:/etc/nginx/conf.d
        ports:
            - 80:80
```

Since the nginx service is ready, we can create the nginx configuration file, called **example.conf**, and place it in the folder **./conf.d** on the host machine. The file contains two sections for the frontend and api services, which are accessable via a domain. In this example, the stack is named as **example** and, therefore, the services' domains are **http://example\_\<service_name\>:80**.

```
server {
    listen 80;

    server_name example.com;
    
    location / {
        proxy_pass http://example_frontend:80;
    }
}

server {
    listen 80;

    server_name api.example.com;

    location / {
        proxy_pass http://example_api:80;
    }
}
```

This file will be mounted into the container and read by nginx when the stack is deployed, as follows:

```bash
docker stack deploy -c docker-compose.yml example
```

This example showed how to setup an nginx service in a stack to route to services that both require port 80.

## Multi-stack scenario
A more complex situation may arise when the nginx service has to route to services that are separated into different stacks. Below discusses how to configure the nginx service to handle such a situation. In this example, we start with two stacks, each containing one service like so:

* Stack 1
  * frontend: http://example.com
* Stack 2
  * api: http://api.example.com

To allow the nginx service to access the service found in each stack, the overlay network for the stacks needs to be exposed. This can be achieved by making the overlay network attachable and connecting the service to that network, as shown below in the compose file for stack 1:

```yaml
version: "3.7"
services:
    frontend:
        networks:
            - overlay

networks:
    overlay:
        driver: overlay
        attachable: true
```

This Compose file is similarly repeat for Stack 2 as well, but contains the api service rather than the frontend service. The stacks are then deployed to the swarm like so:

```bash
docker stack deploy -c docker-compose-stack1.yml stack1

docker stack deploy -c docker-compose-stack2.yml stack2
```

Next, the nginx service is added to the swarm by deploying a separate stack with this service. This service then attaches to the overlay network of stack1 and stack2, allowing it to access the services found within.

```yaml
version: "3.7"
services:
    nginx:
        image: nginx
        init: true
        volumes:
            - ./conf.d:/etc/nginx/conf.d
        ports:
            - 80:80
        networks:
            - overlay
            - stack1_overlay
            - stack2_overlay

networks:
    overlay:
        driver: overlay
        attachable: true
    stack1_overlay:
        external: true
        name: stack1_overlay
    stack2_overlay:
        external: true
        name: stack2_overlay
```

The configuration file for nginx is shown below:

```
server {
    listen 80;

    server_name example.com;
    
    location / {
        proxy_pass http://stack1_frontend:80;
    }
}

server {
    listen 80;

    server_name api.example.com;

    location / {
        proxy_pass http://stack2_api:80;
    }
}
```

Finally, the nginx stack is deployed alongside the other two stacks as follows:

```bash
docker stack deploy -c docker-compose-nginx.yml nginx
```

This example showed how to expand upon the single stack example, deploying an nginx service that resides inside its own stack. This allows the nginx service to attach to stack networks to access the services inside and route traffic appropriately.