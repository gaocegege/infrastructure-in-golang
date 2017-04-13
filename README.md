# infrastructure-in-golang

The goal of this project is to make the easiest, fastest, and most painless way of building your infrastructures with tools written in golang.

## Pool Development Environment

In pool development environment, you don't have enough money to buy a server with public IP. You have to use some graceful hacks to build a devlopment environment.

```bash
$ cd dev; source ./env.sh
$ docker-compose up
```

### Gogs

```bash
# Get the IP from $(ifconfig)
GOGS_IP="docker0 IP"
```

Configuration in Gogs installation UI: 

```text
Database Settings

Host: mysql:3306
Password: gogsdafahao

Application General Settings

Domain: ${GOGS_IP}
SSH Port: 10022
HTTP Port: 10080
Application URL: http://${GOGS_IP}:10080/
```

URL:`http://${GOGS_IP}:10080/`

### Drone

```bash
DRONE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dev_drone-server_1)
```

URL: `http://${DRONE_IP}:8000/`

**Notice:** Use the URL above, not `http://localhost:8000/`, not `http://${GOGS_IP}:8000/`. Because drone will create a webhook in gogs, and the URL used in the hook is this URL. We starts drone server in a docker container, so URL should be visited by gogs, and only `http://${DRONE_IP}:8000/` meets the need. But if you are a rich man, public IP will solve this problem :)

### Distributed Tracing

TODO

## In Docker Swarm

### Gogs

TODO

### Drone

TODO

### Distributed Tracing

TODO

## In Kubernetes

### Gogs

TODO

### Drone

TODO

### Distributed Tracing

TODO
