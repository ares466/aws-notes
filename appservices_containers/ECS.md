# Containers

A `container` provides an isolated environment in which an application can run. Unlike a hypervisor, the container runs as a process on the host OS with the assistance of a container image.

![Containers](../static/images/containers.png)

A container `image` is simply a stopped container. In docker, images are defined using a `Dockerfile`. Images are created from a `base image` or from scratch. Each command in a Dockerfile adds a new layer to the environment using a differential architecture.

A container is simply a running image. The container is identical to the image with an added read/write layer.

Container images are stored in a `container registry`.

**Container Key Concepts**:
- Dockerfiles are used to build images
- Containers are portable and self-contained
- Containers are lightweight that run on a shared parent OS
- Containers only run the application and environments that are needed
- Containers provides isolation
- Ports must be exposed to the host and beyond
- Application stacks can contain many containers

# ECS


## ECS Concepts

A `container definition` defines container settings like which container to use and which ports are exposed. 

ECS `task definitions` contain one or more containers and include parameter values such as CPU & memory, networking mode, logging configuration, the task role, and cluster mode.

`Task roles` define which AWS permissions are available to the container.

An ECS `service definition` defines a `service`. Service definitions grant scalability and high availability to a set of tasks. You can place a load balancer in front of a service to distribute load across multiple tasks.

Tasks and services are deployed into an ECS `cluster`.

## ECS Cluster Types

ECS is a container orchestration engine for running containers at scale.

ECS supports two cluster types:
- [EC2 Mode](#ec2-mode)
- [Fargate](#fargate)

### EC2 Mode

In EC2 Mode, EC2 instances are used to host the containers. Load balancing is controlled by an ASG.

When using ECS in EC2 mode, the customer is responsible for managing the EC2 instances.

When running in EC2 mode, you pay for the entire EC2 instance regardless of whether its fully utilized.

![ECS - EC2 Mode](../static/images/ecs_ec2.png)

EC2 mode may be good for price-conscious customers that may benefit from spot instances.

### Fargate

In Fargate cluster mode, customers do not manage any EC2 instances. 

Fargate deploys containers to `shared infrastructure` and ENIs are injected into the customer's VPC. The containers can only be access via the ENIs.

When using Fargate, you only pay for the containers that are deployed.

Use Fargate for large workloads with lots of overhead, or small bursty workloads since you only pay for usage.

# EKS

