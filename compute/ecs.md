# Elastic Container Service (ECS)
ECS is a fully-managed service for running Docker applications on a scalable cluster.

# Concepts
## Containers & Images
A Docker *container* is a standardized unit of software development, containing everything that your software application needs to run: code, runtime, system tools, system libraries, etc. Containers are created from a read-only template called an *image*.

Images are typically built from a Dockerfile, a plain text file that specifies all of the components that are included in the container. These images are then stored in a registry from which they can be downloaded and run on your cluster.

### ECR (Elastic Container Registry)
Amazon ECR is the AWS Docker registry service. Devs can use the Docker CLI to push, pull, and manage images.

To push a Docker image to an ECR repository, run the following commands (can be copied from the AWS console):
1. Authenticate your Docker client to your registry.
`$(aws ecr get-login --no-include-email --region ap-southeast-2)`
2. Build the Docker image.
`docker build -t myimagerepo .`
3. Tag the image so it can be pushed to the repository.
`docker tag myimagerepo:latest 725350006743.dkr.ecr.ap-southeast-2.amazonaws.com/myimagerepo:latest`
4. Push the image to the repository.
`docker push 725350006743.dkr.ecr.ap-southeast-2.amazonaws.com/myimagerepo:latest`

### Using CodeBuild for building Docker images
Amazon CodeBuild can be used to build Docker images and push them to ECR. The commands are placed in **either** a `buildpsec.yml` file located in the source code root directory, or within the console form when launching a build.

## Task Definitions
A *task definition* is a JSON file that describes a logical grouping of 1-10 containers that form an application. E.g. for a custom blog engine, its task definition would specify a web server, an application server, and an in-memory cache, each in their own container. Task definitions specify various parameters for the application e.g. which containers to use, which launch type to use, which ports should be opened for your application, and what data volumes should be used with the containers in the task.

Example of a task definition containing a single container that runs an NGINX web server using the Fargate launch type:
```json
{
    "family": "webserver",
    "containerDefinitions": [
        {
            "name": "web",
            "image": "nginx",
            "memory": "100",
            "cpu": "99"
        },
    ],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "networkMode": "awsvpc",
    "memory": "512",
    "cpu": "256",
}
```

## Tasks & Scheduling
A *task* is an instantiation of a task definition within a cluster. 
![](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-service-fargate.png)

A *task placement strategy* is an algorithm for selecting instances for task placement or tasks for termination. Task placement strategies can be specified when either running a task or creating a new service.

Amazon ECS supports the following task placement strategies:

* `binpack` - Place tasks based on the least available amount of CPU or memory. This minimizes the number of instances in use.
```
"placementStrategy": [
    {
        "field": "memory",
        "type": "binpack"
    }
]
```
* `random` - Place tasks randomly.
```
"placementStrategy": [
    {
        "type": "random"
    }
]
```
* `spread` - Place tasks evenly based on the specified value. Accepted values are attribute key-value pairs, instanceId, or host. 
```
"placementStrategy": [
    {
        "field": "attribute:ecs.availability-zone",
        "type": "spread"
    }
]
```
```
"placementStrategy": [
    {
        "field": "instanceId",
        "type": "spread"
    }
]
```

### Cluster Query Language
The *Cluster Query Language* allows you to specify a group of containers on which ECS can run its tasks on.

Examples:
```
attribute:ecs.instance-type == t2.small
```
```
attribute:ecs.instance-type =~ g2.* and attribute:ecs.availability-zone != us-east-1d
```
```
registeredAt < 2018-02-13
```

## Clusters
A cluster is a **logical** grouping of resources. ECS downloads your container images from a specified registry, and runs those images within your cluster. The Amazon ECS *container agent* runs on each infrastructure resource within a cluster. It sends information about the resource's current running tasks and resource utilization to ECS, and starts and stops tasks whenever it receives a request from ECS.

![](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-containeragent-fargate.png)
