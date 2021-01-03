# Docker swarm: Built-In Orchestration

Docker swarm answers following questions

- How do we automate container life-cycle

- How can we easily scale out/in/up/down

- How can we ensure our containers are re-created if they fail?

- How can we replace containers without downtime (blue/green deploy) ?

- How can we control/track where containers get started ?

- How can we ensure only trusted servers run our containers ?

- How can we store secrets, keys, passwords and get them to the right container (and only that container) ?

  

Swarm mode is a server clustering solution built inside Docker. A swarm is a group of machines or VM that are running Docker and joined into a cluster. Docker  swarm is a tool for Container Orchestration.

Orchestration means automate container life-cycle management, scaling (out/in/up/down), Security & privacy (only trusted servers run the container,  store secrets, keys, password and get them to right containers)

- A swarm consists of multiple Docker hosts which run in **swarm mode** and act as managers (to manage membership and delegation) and workers (which run [swarm services](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks)). 
- A given Docker host can be a manager, a worker, or perform both roles. When you create a service, you define its optimal state (number of replicas, network and storage resources available to it, ports the service exposes to the outside world, and more). 
- Docker works to maintain that desired state. For instance, if a worker node becomes unavailable, Docker schedules that node’s tasks on other nodes. 
  - A *task* is a running container which is part of a swarm service and managed by a swarm manager, as opposed to a standalone container.

### Advantages of swarm mode over standalone containers

Blue/green deployment: swarm services can modify a service’s configuration, including the networks and volumes it is connected to, without the need to manually restart the service. Docker will update the configuration, stop the service tasks with the out of date configuration, and create new ones matching the desired configuration.

When Docker is running in swarm mode, you can still run standalone containers on any of the Docker hosts participating in the swarm, as well as swarm services. A key difference between standalone containers and swarm services is that only swarm managers can manage a swarm, while standalone containers can be started on any daemon. Docker daemons can participate in a swarm as managers, workers, or both.

### Entities and their role

##### Nodes

- A **node** is an instance of the Docker engine  participating in the swarm. You can also think of this as a Docker node. 
- You can run one or more nodes on a single physical computer or cloud  server, but production swarm deployments typically include Docker nodes  distributed across multiple physical and cloud machines.

##### Deploying application in a node 

- To deploy your application to a swarm, you submit a service definition to a **manager node**. The manager node dispatches units of work called [**tasks**](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks) to worker nodes.

##### Orchestration in swarm

- **Manager nodes** also perform the orchestration and cluster management functions required to maintain the desired state of the swarm. Manager nodes elect a single leader to conduct orchestration tasks.
- **Worker nodes** receive and execute tasks dispatched from manager nodes. By default manager nodes also run services as worker nodes, but you can configure them to run manager tasks exclusively and be manager-only nodes. 
- An **agent** **runs on each worker node** and reports on the tasks assigned to it. The worker node notifies the manager node of the current state of its assigned tasks so that the manager can maintain the desired state of each worker.

##### Services

- A **service** is the definition of the tasks to execute on the manager or worker nodes. It is the central structure of the swarm system and the primary root of user interaction with the swarm.
- When you create a service, you specify which container image to use and which commands to execute inside running containers.
- In the **replicated services** model, the swarm manager distributes a specific number of replica tasks among the nodes based upon the scale you set in the desired state.
- For **global services**, the swarm runs one task for the service on every available node in the cluster.

##### Tasks

- A **task** carries a Docker container and the commands to run inside the container. It is the atomic scheduling unit of swarm. 
- Manager nodes assign tasks to worker nodes according to the number of replicas set in the service scale. 
- **Once a task is assigned** to a node, **it cannot move to another node**. It can only run on the assigned node or fail.

##### Load balancing

- The swarm manager uses **ingress load balancing** to expose the services  to make available externally to the swarm. 
- [Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#ingress-v1-networking-k8s-io) exposes HTTP and HTTPS routes from outside the cluster to [services](https://kubernetes.io/docs/concepts/services-networking/service/) within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
- The swarm manager can automatically assign the service a **PublishedPort** or you can configure a PublishedPort for the service. You can specify any unused port. If you do not specify a port, the swarm manager assigns the service a port in the 30000-32767 range.
- **External components**, such as cloud load balancers, **can access the service** on the **PublishedPort of any node in the cluster** whether or not the node is currently running the task for the service. All nodes in the swarm route ingress connections to a running task instance.
- Swarm mode has an internal DNS component that automatically assigns **each service in the swarm a DNS entry**. The swarm manager uses **internal load balancing** to **distribute requests** among services within the cluster based upon the **DNS name of the service**.

### Swarm commands

There following types of docker command

- docker swarm <CMD>  : This is for swarm manager
- docker node <CMD>      : This is for worker node
- docker service <CMD>   : This is for a task
- docker stack <CMD>
- docker secret <CMD>

##### docker swarm <CMD>

These commands affects swarm manager



:octopus: To check if swarm feature is enabled or not

```bash
docker info
```

:bell: Check for swarm in the logs



 :octopus: Help in swarm

```bash
docker swarm --help

#logs
Usage:	docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

```



:octopus:  ​To enable swarm feature, 

```bash
docker swarm init
```

:bell: By default swarm mode is disabled

What happens during swarm init ?

- Lots of PKI ([Public Key Infrastructure](https://www.venafi.com/education-center/pki/how-does-pki-work)) and security automation
  - Root Signing Certificate created for the swarm
  - Certificate is issued for first manager node
  - Join tokens are created
- Creates Raft database to store root CA, configs and secret
  - Encryptes by default on disk
  - No need for another key/value system to hold orchestration/secrets
  - Replicates logs amongst Managers via mutual TLS in "control plane"

:octopus: To create a new service

```bash
docker service create <image-name> <option>

# example
docker service create alpine ping 8.8.8.8
```

:octopus: To create a new services with replicas

```bash
docker service create --replicas <count> <image-name> <option>

# example
docker service create --replicas 3 alpine ping 8.8.8.8 
```

:octopus: To check which services are running

```bash
docker service ls
```

:octopus: To check a container in a swarm

```bash
docker service ps <service-name>
```

:octopus: To scale services

```bash
docker service update <service-id/service-name> --replicas <count>

#Example
docker service update mqtt_client --replicas 3
```



:octopus: Getting the swarm managers token to be able for other nodes to join

- Type below command in swarm manager and then logs are displayed about join-token string

```bash
docker swarm join-manager manager
```



:octopus: Joining the swarm 

```bash
docker swarm join-token <token-string>
```

##### docker node <CMD>

:octopus:  List docker node both manager & workers

```bash
docker node ls
```

:octopus: Updating node role

```bash
docker node update --role <manager/worker> <node-name>

#Example
docker node update --role manager node2
```



