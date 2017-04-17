# infrastructure-in-golang

The goal of this project is to make the easiest, fastest, and most painless way of building your infrastructures with tools written in golang.

## TOC

   * [infrastructure-in-golang](#infrastructure-in-golang)
      * [TOC](#toc)
      * [Technical Stack](#technical-stack)
      * [Setup](#setup)
         * [Pool Development Environment(Native   Docker)](#pool-development-environmentnative--docker)
            * [Gogs](#gogs)
            * [Drone](#drone)
            * [OpenTracing](#opentracing)
         * [Rich Development Environment(Public IP   Docker)](#rich-development-environmentpublic-ip--docker)
            * [Gogs](#gogs-1)
            * [Drone](#drone-1)
            * [OpenTracing](#opentracing-1)
         * [Awesome Development Environment(Docker Machine   Docker)](#awesome-development-environmentdocker-machine--docker)
            * [Gogs](#gogs-2)
            * [Drone](#drone-2)
         * [Poor Docker Swarm mode(Docker Machine   Docker Swarm Mode)](#poor-docker-swarm-modedocker-machine--docker-swarm-mode)
            * [Gogs](#gogs-3)
            * [Drone](#drone-3)
            * [OpenTracing](#opentracing-2)
         * [Rich Docker Swarm mode(Docker Swarm Mode)](#rich-docker-swarm-modedocker-swarm-mode)
         * [Docker Swarm Standalone(Docker Swarm)](#docker-swarm-standalonedocker-swarm)
         * [Poor Kubernetes(minikube)](#poor-kubernetesminikube)
            * [Gogs](#gogs-4)
            * [Drone](#drone-4)
            * [Kubenetes Dashboard](#kubenetes-dashboard)
            * [Kubernetes Grafana](#kubernetes-grafana)
            * [OpenTracing](#opentracing-3)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Technical Stack

* Version Control System: [Git](https://git-scm.com/)
* Source Code Management Tool: [Gogs](https://gogs.io/)
* Continuous Integration and Deployment Tool: [Drone](https://github.com/drone/drone)
* Log Tracing Tool: [OpenTracing](https://github.com/opentracing/opentracing-go)

## Setup

### Pool Development Environment(Native + Docker)

In pool development environment, you don't have enough money to buy a server with public IP. You have to use some graceful hacks to build a devlopment environment.

```bash
$ source dev/env.sh && cd dev
$ docker-compose up
```

#### Gogs

```bash
# Get the IP from $(ifconfig).
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

URL: `http://${GOGS_IP}:10080/`

#### Drone

```bash
DRONE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dev_drone-server_1)
```

URL: `http://${DRONE_IP}:8000/`

**Notice:** Use the URL above, not `http://localhost:8000/`, not `http://${GOGS_IP}:8000/`. Because drone will create a webhook in gogs, and the URL used in the hook is this URL. We starts drone server in a docker container, so URL should be visited by gogs, and only `http://${DRONE_IP}:8000/` meets the need. But if you are a rich man, public IP will solve this problem :)

#### OpenTracing

TODO

### Rich Development Environment(Public IP + Docker)

You are not a poor man. You have money to buy a server with public IP, so things are easy to you.

```bash
# Get the IP from your cloud server vendor.
$ IP="Your Public IP"
$ source dev/env.sh && cd dev
$ docker-compose up
```

#### Gogs

Configuration in Gogs installation UI: 

```text
Database Settings

Host: mysql:3306
Password: gogsdafahao

Application General Settings

Domain: ${IP}
SSH Port: 10022
HTTP Port: 10080
Application URL: http://${IP}:10080/
```

URL: `http://${IP}:10080/`

#### Drone

URL: `http://${IP}:8000/`

#### OpenTracing

TODO

### Awesome Development Environment(Docker Machine + Docker)

You know there is a tool `docker-machine`, which could give you a fixed internal IP. You could use it as a mock public IP. Then you could build a awesome development although you are a poor man.

```bash
$ cd dev/
# You need to install a hypervisor vendor such as virtualbox or kvm before you use docker machine.
$ docker-machine create --driver=<hypervisor vendor> awesome-dev
$ docker-machine ssh awesome-dev
# Get the IP of the machine via `docker-machine ls | grep awesome-dev`, usually 192.168.99.10X.
$ IP="Your awesome-dev machine IP"
```

#### Gogs

Configuration in Gogs installation UI: 

```text
Database Settings

Host: mysql:3306
Password: gogsdafahao

Application General Settings

Domain: ${IP}
SSH Port: 10022
HTTP Port: 10080
Application URL: http://${IP}:10080/
```

URL: `http://${IP}:10080/`

#### Drone

URL: `http://${IP}:8000/`

### Poor Docker Swarm mode(Docker Machine + Docker Swarm Mode)

Again, you are a poor man but you want to use Docker Swarm mode to build your infrastructure in data center level, then you need `docker-machine`.

```bash
$ cd swarm-mode/
# You need to install a hypervisor vendor such as virtualbox or kvm before you use docker machine.
$ docker-machine create --driver=<hypervisor vendor> swarm-manager
$ docker-machine create --driver=<hypervisor vendor> swarm-worker-1
$ docker-machine ssh swarm-manager
# Get the IP of the machine via `docker-machine ls | grep swarm-manager`, usually 192.168.99.10X.
$ docker swarm init --advertise-addr <Manager IP>
```

Manager node will use port 2377 by default to communicate with worker nodes, then you need to configure the manager machine in virtualbox UI, to make sure port 2377 is exposed. You will get a bash statement if you successfully joins swarm as a manager, run the statement in worker machine:

```bash
$ docker-machine ssh swarm-worker-1
$ docker swarm join xxx <Manager IP>:2377
```

Congratulations, you have got a minor data center although you are poor :) Things goes easy now:

```bash
$ docker stack deploy --compose-file ./docker-compose.yml <a cool name>
```

#### Gogs

Configuration in Gogs installation UI: 

```text
Database Settings

Host: mysql:3306
Password: gogsdafahao

Application General Settings

Domain: ${Manager_IP}
SSH Port: 10022
HTTP Port: 10080
Application URL: http://${Manager_IP}:10080/
```

URL: `http://${Manager_IP}:10080/`

#### Drone

URL: `http://${Manager_IP}:8000/`

#### OpenTracing

TODO

### Rich Docker Swarm mode(Docker Swarm Mode)

TODO(I have no money to do)

### Docker Swarm Standalone(Docker Swarm)

TODO

### Poor Kubernetes(minikube)

It is me.

First, you need to [install the requirements for minikube.](https://kubernetes.io/docs/getting-started-guides/minikube).

Then minikube should be started:

```bash
$ cd kubernetes/
$ minikube start
$ MINIKUBE_IP=$(minikube ip)
$ kubectl create -f .
```

**NOTICE:** There are some performance bugs in this way: drone runs builds slowly. A simple "Hello, World" CI build spends drone 9mins. I don't know why now.

| NAMESPACE   |     NAME                |              URL            |
|-------------|-------------------------|-----------------------------|
| default     | drone-server            | http://${MINIKUBE_IP}:30800 |
| default     | gogs(ssh)               | http://${MINIKUBE_IP}:30022 |
| default     | gogs(http)              | http://${MINIKUBE_IP}:30080 |
| kube-system | heapster                | http://${MINIKUBE_IP}:30082 |
| kube-system | kubernetes-dashboard    | http://${MINIKUBE_IP}:30000 |
| kube-system | monitoring-grafana      | http://${MINIKUBE_IP}:30088 |

#### Gogs

Configuration in Gogs installation UI: 

```text
Database Settings

Host: mysql:3306
Password: gogsdafahao

Application General Settings

Domain: ${MINIKUBE_IP}
SSH Port: 30022
HTTP Port: 30080
Application URL: http://${MINIKUBE_IP}:30080/
```

URL: `http://${MINIKUBE_IP}:30080/`

#### Drone

URL: `http://${MINIKUBE_IP}:30800/`

#### Kubenetes Dashboard

URL: `http://${MINIKUBE_IP}:30000/`

#### Kubernetes Grafana

URL: `http://${MINIKUBE_IP}:30088/`

#### OpenTracing

TODO
