## Process Simplification

Traditionally, the cycle of getting an application to production often looks something like the following (illustrated in Figure 2-1):

1.  Application developers request resources from operations engineers.
    
2.  Resources are provisioned and handed over to developers.
    
3.  Developers script and tool their deployment.
    
4.  Operations engineers and developers tweak the deployment repeatedly.
    
5.  Additional application dependencies are discovered by developers.
    
6.  Operations engineers work to install the additional requirements.
    
7.  Loop over steps 5 and 6 _n_ more times.
    
8.  The application is deployed.
    

![A Non-Docker deployment workflow](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131814/files/assets/dur3_0201.png)

###### Figure 2-1. A traditional deployment workflow (without Docker)

That’s not very productive, and even though DevOps practices work to alleviate many of the barriers, it often still requires a lot of effort and communication between teams of people. This process can be both technically challenging and expensive, but even worse, it can limit the kinds of innovation that development teams will undertake in the future.

Push-to-deploy systems like [Heroku](https://www.heroku.com/) have shown developers what the world can look like if you are in control of your application and a majority of your dependencies. You’ve probably heard complaints about how much slower your internal systems are compared with deploying on “push-button” solutions like Heroku, which are built on top of Linux container technology.

Some of the tooling and orchestrators that have been built on top of Docker (_e.g., Kubernetes, Docker Swarm mode, and Mesos_) aim to replicate the simplicity of systems like Heroku. But even though these platforms wrap more around Docker to provide a more capable and complex environment, a simple platform that uses only Docker still provides all of the core process benefits without the added complexity of a larger system.

As a company, Docker adopts an approach of “batteries included but removable.” This means that they want their tools to come with everything most people need to get the job done, while still being built from interchangeable parts that can easily be swapped in and out to support custom solutions.

By using an image repository as the hand-off point, Docker allows the responsibility of building the application image to be separated from the deployment and operation of the container. What this means in practice is that development teams can build their application with all of its dependencies, run it in development and test environments, and then just ship the exact same bundle of application and dependencies to production. Because those bundles all look the same from the outside, operations engineers can then build or install standard tooling to deploy and run the applications. The cycle described in Figure 2-1 then looks somewhat like this (illustrated in Figure 2-2):

1.  Developers build the Docker image and ship it to the registry.
    
2.  Operations engineers provide configuration details to the container and provision resources.
    
3.  Developers trigger deployment.
    

![A Docker deployment workflow](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131814/files/assets/dur3_0202.png)

###### Figure 2-2. A Docker deployment workflow

## Broad Support and Adoption

Docker is already well supported, with the majority of the large public clouds offering some direct support for it. For example, Docker and Linux containers has been used in AWS via multiple products like Elastic Container Service (ECS), Elastic Kubernetes Service (EKS), Fargate, and Elastic Beanstalk. Linux containers can also be used on Google AppEngine, Google Kubernetes Engine, Red Hat OpenShift, IBM Cloud, Microsoft Azure, and many more.

Docker’s image format for Linux containers has become the lingua franca between cloud providers, offering the potential for “write once, run anywhere” cloud applications. With most providers offering some form of Docker and Linux container orchestration as well as the container runtime itself, Docker is well supported for nearly any kind of workload in common production environments. If all of your tooling is built around Docker and Linux containers then your applications can be deployed in a cloud-agnostic manner, allowing for new flexibility that was not previously possible.

To gain some goodwill and support wider adoption in the marketplace, Docker, Inc., decided to help sponsor [the Open Container Initiative (OCI)](https://www.opencontainers.org/) in June of 2015. The first full specification from that effort was released in July 2017 and was based in large part on version 2 of the Docker image format. It is now possible to apply for OCI certification for both container images and container runtimes. Some OCI certified runtimes include:

-   [runC](https://github.com/opencontainers/runc), which is a core part of Docker and the opensource Moby project.
    
-   [crun](https://github.com/containers/crun), which is written in C, and designed to be fast and have a small memory footprint.
    
-   [Kata Containers](https://katacontainers.io/) from Intel, Hyper, and the OpenStack Foundation, is a virtualized runtime, that can run a mix of containers and virtual machines.
    
-   [gVisor](https://github.com/google/gvisor) from Google, is a sandboxed runtime, implemented entirely in user space.
    
-   and [Nabla Containers](https://nabla-containers.github.io/), provides another sandboxed runtime designed to significantly reduce the attack surface of Linux containers.

## Architecture

Docker is a powerful technology, and that often indicates something that comes with a high level of complexity. And, under the hood, Docker is fairly complex; however, its fundamental user-facing structure is indeed a simple client/server model. Several pieces are sitting behind the Docker API, including `containerd` and `runc`, but the basic system interaction is a client talking over an API to a server.

Underneath this simple exterior, Docker heavily leverages kernel mechanisms such as iptables, virtual bridging, cgroups[2](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387826736), namespaces[3](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387859424), Linux capabilities, secure computing mode, various filesystem drivers, and more.

### Client/Server Model

It’s easiest to think of Docker as consisting of two parts: the client and the server/daemon (see Figure 2-3). Optionally there is a third component called the registry, which stores Docker images and their metadata. The server does the ongoing work of building, running, and managing your containers, and you use the client to tell the server what to do. 

The Docker [daemon](http://bit.ly/1Bttd5s) can run on any number of servers in the infrastructure, and a single client can address any number of servers. Clients drive all of the communication, but Docker servers can talk directly to image registries when told to do so by the client. Clients are responsible for telling servers what to do, and servers focus on hosting and managing containerized applications.

![Docker Client/Server Model](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131814/files/assets/dur3_0203.png)

###### Figure 2-3. Docker client/server model

Docker is a little different in structure from some other client/server software. It has a `docker` client and a `dockerd` server, but rather than being entirely monolithic, the server then orchestrates a few other components behind the scenes on behalf of the client, including `containerd-shim-runc-v2`, which is used to interact with `runc` and `containerd`.

Docker cleanly hides any complexity behind the simple server API, though, so you can just think of it as a straight-forward client and server for most purposes. Each Docker host will normally have one Docker server running that can manage any number of containers. You can then use the `docker` command-line tool to talk to the server, either from the server itself or, if properly secured, from a remote client.

### Network Ports and Unix Sockets

The `docker` command-line tool and `dockerd` daemon can talk to each other over Unix sockets and network ports. Docker, Inc., has registered three ports with IANA for use by the Docker daemon and client: TCP ports 2375 for unencrypted traffic, port 2376 for encrypted SSL connections, and port 2377 for Docker Swarm mode. Using a different port is easily configurable for scenarios where you need to use different settings.

This is also easily configurable, but it is highly recommended that network ports are not used with Docker, due to the lack of user authentication and role-based-access controls within the Docker daemon. The Unix socket can be located in different paths on different operating systems, but in most cases, it can be found here: _/var/run/docker.sock_. If you have strong preferences for a different location, you can usually specify this at install time or simply change the server configuration afterward and restart the daemon. If you don’t, then the defaults will probably work for you.

### Robust Tooling

The tooling that Docker ships with supports building Docker images, basic deployment to individual Docker daemons, a distributed mode called Swarm mode, and all the functionality needed to manage a remote Docker server. Beyond the included Swarm mode, community efforts have focused on managing whole fleets (or clusters) of Docker servers and scheduling and orchestrating container deployments.

Docker has also launched its own orchestration toolset, including [Compose](https://github.com/docker/compose), [Docker Desktop](https://www.docker.com/products/docker-desktop/) and [Swarm mode](https://docs.docker.com/engine/swarm/), which creates a cohesive deployment story for developers. Docker’s offerings in the production orchestration space have been largely overshadowed by Google’s Kubernetes, although it should be noted that [Kubernetes relied heavily on Docker until v1.24 was released in early 2022](https://kubernetes.io/blog/2022/02/17/dockershim-faq/). But Docker’s orchestration tools remain useful, with Compose being particularly handy for local development.

Because Docker provides both a command-line tool and a remote REST API, it is easy to add further tooling in any language. The command-line tool lends itself well to shell scripting and anything the client can do can also be done programmatically via the REST API. The docker CLI[4](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387986496) is so well known that many other Linux container CLI tools, like [podman](https://podman.io/) and [nerdctl](https://github.com/containerd/nerdctl), mimic its arguments for compatibility and easy adoption.

### Docker Command-Line Tool

The command-line tool `docker` is the main interface that most people will have with Docker. The Docker client is a [Go program](https://golang.org/) that compiles and runs on all common architectures and operating systems. The command-line tool is available as part of the main Docker distribution on various platforms and also compiles directly from the Go source. Some of the things you can do with the Docker command-line tool include, but are not limited to:

-   Build a container image.
    
-   Pull images from a registry to a Docker daemon or push them up to a registry from the Docker daemon.
    
-   Start a container on a Docker server either in the foreground or background.
    
-   Retrieve the Docker logs from a remote server.
    
-   Start a command-line shell inside a running container on a remote server.
    
-   Monitor statistics about your container.
    
-   Get a process listing from your container.

You can probably see how these can be composed into a workflow for building, deploying, and observing applications. But the Docker command-line tool is not the only way to interact with Docker, and it’s not necessarily the most powerful.

### Docker Engine API

Like many other pieces of modern software, the Docker daemon has an application programming interface (API). This is in fact what the Docker command-line tool uses to communicate with the daemon. But because the API is documented and public, it’s quite common for external tooling to use the API directly. This enables all manners of tooling, from mapping deployed Linux containers to servers, to automated deployments, to distributed schedulers.

Extensive documentation for the [API](https://dockr.ly/2wxCHnx) is on the Docker site. As the ecosystem has matured, robust implementations of Docker API libraries have emerged for all popular languages. Docker maintains [SDKs for Python and Go](https://dockr.ly/2wxCHnx), there are additional libraries maintained by third parties that are worth considering. For example, over the years we have used these [Go](http://bit.ly/2C054AL) and [Ruby](http://bit.ly/2olBU5m) libraries, and have found them to be both robust and rapidly updated as new versions of Docker are released.

Two notable exceptions are the endpoints that require streaming or terminal access: running remote shells or executing the container in interactive mode. In these cases, it’s often easier to use one of these solid client libraries or the command-line tool.

### Container Networking

Even though Linux containers are largely made up of processes running on the host system itself, they usually behave quite differently from other processes at the network layer. Docker initially supported a single networking model, but now supports a robust assortment of configurations that handle most application requirements. Most people run their containers in the default configuration, called _bridge mode_.

To understand bridge mode, it’s easiest to think of each of your Linux containers as behaving like a host on a private network. The Docker server acts as a virtual bridge and the containers are clients behind it. A bridge is just a network device that repeats traffic from one side to another. So you can think of it like a mini–virtual network with each container acting like a host attached to that network.

The actual implementation, as shown in Figure 2-4, is that each container has a virtual Ethernet interface connected to the Docker bridge and an IP address allocated to the virtual interface. Docker lets you bind and expose individual or groups of ports on the host to the container so that the outside world can reach your container on those ports. The traffic is largely managed by the [vpnkit](https://github.com/moby/vpnkit) library.

Docker allocates the private subnet from an unused [RFC 1918](http://bit.ly/2C6F46C) private subnet block. It detects which network blocks are unused on the host and allocates one of those to the virtual network. That is bridged to the host’s local network through an interface on the server called `docker0`. This means that, by default, all of the containers are on a network together and can talk to each other directly. But to get to the host or the outside world, they go over the `docker0` virtual bridge interface.

![The network on a typical Docker server](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131814/files/assets/dur3_0204.png)

###### Figure 2-4. The network on a typical Docker server

There is a dizzying array of ways in which you can configure Docker’s network layer, from allocating your own network blocks to configuring your own custom bridge interface. People often run with the default mechanisms, but there are times when something more complex or specific to your application is required. You can find much more detail about Docker networking in the [documentation](http://dockr.ly/2otp461).

## Getting the Most from Docker

To begin with, Docker’s architecture is aimed squarely at applications that are either stateless or where the state is externalized into data stores like databases or caches. Those are the easiest to containerize.

Some good applications for beginning with Docker include web frontends, backend APIs, and short-running tasks like maintenance scripts that might normally be handled by cron.

### Containers Are Not Virtual Machines

A good way to start shaping your understanding of how to leverage Docker is to think of Linux containers not as virtual machines but as very lightweight wrappers around a single Unix process. During actual implementation, that process might spawn other processes, but on the other hand, one statically compiled binary could be all that’s inside your container. Containers are also ephemeral: they may come and go much more readily than a traditional virtual machine.

Virtual machines are by design a stand-in for real hardware that you might throw in a rack and leave there for a few years. Because a real server is what they’re abstracting, virtual machines are often long-lived in nature. Even in the cloud where companies often spin virtual machines up and down on demand, they usually have a running lifespan of days or more.

On the other hand, a particular container might exist for months, or it may be created, run a task for a minute, and then be destroyed. All of that is OK, but it’s a fundamentally different approach than the one virtual machines are typically used for.

To help drive this differentiation home, if you run Docker on a Mac or Windows system you are leveraging a Linux virtual machine to run `dockerd`, the Docker server. However, on Linux, `dockerd` can be run natively and therefore there is no need for a virtual machine to be run anywhere on the system (see [Figure 2-5](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#figure2-4)).

![Typical Docker Installations](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131814/files/assets/dur3_0205.png)

###### Figure 2-5. Typical Docker installations

## Limited Isolation

Containers are isolated from each other, but that isolation is probably more limited than you might expect. While you can put limits on their resources, the default container configuration just has them all sharing CPU and memory on the host system, much as you would expect from co-located Unix processes. This means that unless you constrain them, containers can compete for resources on your production machines. That might be fine for your use case, but it impacts your design decisions. Limits on CPU and memory use are encouraged through Docker, but in most cases, they are not the default like they would be with a virtual machine.

It’s often the case that many containers share one or more common filesystem layers. That’s one of the more powerful design decisions in Docker, but it also means that if you update a shared image, you may also need to rebuild and redeploy containers that are still utilizing the older image.

Containerized processes are just processes on the Docker server itself. They are running on the same instance of the Linux kernel as the host operating system. All container processes show up in the normal `ps` output on the Docker server. That is utterly different from a hypervisor, where the depth of process isolation usually includes running an entirely separate instance of the operating system kernel for each virtual machine.

This light containment can lead to the tempting option of exposing more resources from the host, such as shared filesystems to allow the storage of state. But you should think hard before further exposing resources from the host into the container unless they are used exclusively by the container.

### Containers Are Lightweight

Creating a new container can take up very little disk space. A quick test reveals that a newly created container from an existing image takes a whopping 12 kilobytes of disk space. That’s pretty lightweight. By default, there is no copy of the data allocated to the container. Containers are just processes on the existing system that may only need to read information from the disk, so there may not be a need to copy any data for the exclusive use of the container, until a time when it needs to write data that is unique to that container instance.

The lightness of containers means that you can use them for situations where creating another virtual machine would be too heavyweight or where you need something to be truly ephemeral. You probably wouldn’t, for instance, spin up an entire virtual machine to run a `curl` command to a website from a remote location, but you might spin up a new container for this purpose.

### Towards an Immutable Infrastructure

By deploying most of your applications within containers, you can start simplifying your configuration management story by moving toward an immutable infrastructure, where components are replaced entirely rather than being changed in place. The idea of an immutable infrastructure has gained popularity in response to how difficult it is, in reality, to maintain a truly idempotent configuration management codebase. As your configuration management codebase grows, it can become as unwieldy and unmaintainable as large, monolithic legacy applications.

With Docker, it is possible to deploy a very lightweight Docker server that needs almost no configuration management, or in many cases, none at all. You handle all of your application management simply by deploying and redeploying containers to the server. When the server needs an important update to something like the Docker daemon or the Linux kernel, you can simply bring up a new server with the changes, deploy your containers there, and then decommission or reinstall the old server.

Container-based Linux distributions like [Red Hat’s Fedora CoreOS](https://getfedora.org/en/coreos) are designed around this principle. But rather than requiring you to decommission the instance, Fedora CoreOS can entirely update itself and switch to the updated OS. Your configuration and workload largely remain in your containers and you don’t have to configure the OS very much at all.

Because of this clean separation between deployment and configuration of your servers, many container-based production systems are now using tools such as [HashiCorp’s Packer](https://www.packer.io/intro/index.html) to build cloud virtual server images and then leveraging Docker to nearly or entirely avoid configuration management systems.

### Stateless Applications

A good example of the kind of application that containerizes well is a web application that keeps its state in a database. Stateless applications are normally designed to immediately answer a single self-contained request and have no need to track information between requests from one or more clients. You might also run something like ephemeral [memcached](https://memcached.org/) instances in containers.

If you think about your web application, though, it probably has some local state that you rely on, like configuration files. That might not seem like a lot of state, but if you bake that configuration into your images, it means that you’ve limited the reusability of your image and made it more challenging to deploy into different environments, without maintaining multiple images for different deployment targets.

In many cases, the process of containerizing your application means that you move configuration state into environment variables that can be passed to your application at runtime. Rather than baking the configuration into the container, you apply the configuration to the container when it is deployed. This allows you to easily do things like use the same container to run in either production or staging environments. In most companies, those environments would require many different configuration settings like the connection URLs for various external services that the application utilizes.

With containers, you might also find that you are always decreasing the size of your containerized application as you optimize it down to the bare essentials required to run. We have found that thinking of anything that you need to run in a distributed way as a container can lead to some interesting design decisions. If, for example, you have a service that collects some data, processes it, and returns the result, you might configure containers on many servers to run the job and then aggregate the response on another container.

### Externalizing State

If Docker works best for stateless applications, how do you best store state when you need to? Configuration is typically passed by environment variables, for example. Docker supports environment variables natively, and they are stored in the metadata that makes up a container configuration. This means that restarting the container will ensure that the same configuration is passed to your application each time. It also makes the configuration of the container easily observable while it’s running, which can make debugging a lot easier, although there are some security concerns around exposing secrets in environment variables.

Databases are often where scaled applications store state and nothing in Docker interferes with doing that for containerized applications. Applications that need to store files, however, face some challenges. Storing things to the container’s filesystem is not performant, will be limited by space, and will not preserve state when a container is re-created. If you re-deploy a stateful service without utilizing storage external to the container, you will lose all of that state. Applications that need to store filesystem state should be considered carefully before you put them into Docker. If you decide that you can benefit from Linux containers in these cases, it’s best to design a solution where the state can be stored in a centralized location that could be accessed regardless of which host a container runs on. In certain cases, this might mean using a service like Amazon S3, OpenStack Swift, a local block store, or even mounting EBS volumes or iSCSI disks inside the container. [Docker volume plug-ins](https://docs.docker.com/engine/extend/plugins_volume/) provide some additional options.

## The Docker Workflow

Like many tools, Docker strongly encourages a particular workflow. If the workflow is implemented well, it can help realize the promise of reduced communication overhead between teams.

### Revision Control

The first thing that Docker gives you out of the box is two forms of revision control. One of them is used to track the filesystem layers that each Docker image is comprised of and the other is a tagging system for those images.

#### Filesystem layers

Linux containers are made up of stacked filesystem layers, each identified by a unique hash, where each new set of changes made during the build process is laid on top of the previous changes. That’s great because it means that when you do a new build, you only have to rebuild the layers that follow the change you’re deploying. This saves time and bandwidth because containers are shipped around as layers and you don’t have to ship layers that a server already has stored. If you’ve done deployments with many classic deployment tools, you know that you can end up shipping hundreds of megabytes of the same data to a server over and over, with each deployment. That’s incredibly inefficient, and worse, you can’t be sure exactly what changed between deployments. Because of the layering effect, and because Linux containers include all of the application dependencies, with Docker you can be more confident about the changes that you are shipping to production.

To simplify this a bit, remember that a Docker image contains everything required to run your application. If you change one line of code, you certainly don’t want to waste time rebuilding every dependency that your code requires into a new image. Instead, by leveraging the build cache, Docker can ensure that only the layers affected by the code change are rebuilt.

#### Image tags

The second kind of revision control offered by Docker makes it easy to answer an important question: what was the previous version of the application that was deployed? That’s not always easy to answer. There are a lot of solutions for non-containerized applications, from Git tags for each release, to deployment logs, to tagged builds for deployment, and many more.

Docker has a built-in mechanism: it provides image tagging a standard build step. You can easily leave multiple revisions of your application on the server so that performing a rollback is trivial. This is not rocket science, and it’s not functionality that is hard to find in other deployment tooling, but with container images, it can easily be made standard across all of your applications, and everyone can have the same expectations about how things will be tagged for all applications. This makes communication easier between teams and it makes tooling much simpler because there is one source of truth for application releases.

### Building

Building applications is a black art in many organizations, where a few people know all the levers to pull and knobs to turn to spit out a well-formed, shippable artifact. Part of the heavy cost of getting a new application deployed is getting the build just right. Docker doesn’t solve all of these problems, but it does provide a standardized tool configuration and toolset for builds. That makes it a lot easier for people to learn how to build your applications and to get new builds up and running.

The Docker command-line tool contains a `build` flag that will consume a _Dockerfile_ and produce a Docker image. Each command in a _Dockerfile_ generates a new layer in the image, so it’s easy to reason about what the build is going to do by looking at the _Dockerfile_ itself. The great part of all of this standardization is that any engineer who has worked with a _Dockerfile_ can dive right in and modify the build of any other application. Because the Docker image is a standardized artifact, all of the tooling behind the build will be the same regardless of the development language or base image that is being used or the number of layers needed. The _Dockerfile_ is usually checked into a revision control system, which also means tracking changes to the build is simplified. Modern multistage Docker builds also allow you to define the build environment separately from the final artifact image. This provides huge configure-ability for your build environment just like you’d have for a production container.

Many Docker builds are a single invocation of the `docker image build` command and generate a single artifact, the container image. Because it’s usually the case that most of the logic about the build is wholly contained in the _Dockerfile_, it’s easy to create standard build jobs for any team to use in build systems like [Jenkins](http://jenkins-ci.org/). As a further standardization of the build process, many companies—eBay, for example—have standardized Linux containers to do the image builds from a _Dockerfile_. SaaS build offerings like [Travis CI](https://travis-ci.com/) and [Codeship](https://codeship.com/) also have first-class support for Docker builds.

It is also possible to automate the creation of multiple images that support different underlying compute architectures, like x86 and ARM, by utilizing the newer [BuildKit](https://github.com/moby/buildkit) support in Docker.

### Testing

While Docker itself does not include a built-in framework for testing, the way containers are built lends some advantages to testing with Linux containers.

Docker facilitates better testing by guaranteeing that the artifact that passed testing will be the one that ships to production. This can be guaranteed because we can either use the Docker SHA for the container, or a custom tag to make sure we’re consistently shipping the same version of the application.

Since, by design, containers include all of their dependencies, tests run on containers are very reliable. If a unit test framework says tests were successful against a container image, you can be sure that you will not experience a problem with the versioning of an underlying library at deployment time, for example. That’s not easy with most other technologies, and even Java WAR (Web application ARchive) files, for example, don’t include testing of the application server itself. That same Java application deployed in a Linux container will generally also include an application server like Tomcat, and the whole stack can be smoke-tested before shipping to production.

A secondary benefit of shipping applications in Linux containers is that in places where there are multiple applications that talk to each other remotely via something like an API, developers of one application can easily develop against a version of the other service that is currently tagged for the environment they require, like production or staging. Developers on each team don’t have to be experts in how the other service works or is deployed just to do development on their own application. If you expand this to a service-oriented architecture with innumerable microservices, Linux containers can be a real lifeline to developers or QA engineers who need to wade into the swamp of inter-microservice API calls.

A common practice in organizations that run Linux containers in production is for automated integration tests to pull down a versioned set of Linux containers for different services, matching the current deployed versions. The new service can then be integration-tested against the very same versions it will be deployed alongside.

### Packaging

Docker builds produce an image that can be treated as a single build artifact, although technically they may consist of multiple filesystem layers. No matter which language your application is written in or which distribution of Linux you run it on, you get a layered Docker image as the result of your build. And it is all built and handled by the Docker tooling. That build image is the shipping container metaphor that Docker is named for: a single, transportable unit that universal tooling can handle, regardless of what it contains.

Applications that traditionally took a lot of custom configuration to deploy onto a new host or development system become very portable with Docker. Once a container is built, it can easily be deployed on any system with a running Docker server.

## Deploying

Deployments are handled by so many kinds of tools in different shops that it would be impossible to list them here. Some of these tools include shell scripting, [Capistrano](https://capistranorb.com/), [Fabric](https://www.fabfile.org/), [Ansible](https://www.ansible.com/), and in-house custom tooling.

The built-in tooling supports a simple, one-line deployment strategy to get a build onto a host and up and running. The standard Docker client handles deploying only to a single host at a time, but there are a large array of tools available that make it easy to deploy into a cluster of Docker or other compatible Linux container hosts. Because of the standardization Docker provides, your build can be deployed into any of these systems, with low complexity on the part of the development teams.

### The Docker Ecosystem

Over the years, a wide community has formed around Docker, driven by both developers and system administrators. Like the DevOps movement, this has facilitated better tools by applying code to operations problems. Where there are gaps in the tooling provided by Docker, other companies and individuals have stepped up to the plate. Many of these tools are also open source. That means they are expandable and can be modified by any other company to fit their needs.

#### Orchestration

The first important category of tools that adds functionality to the core Docker distribution and Linux container experience contains orchestration and mass deployment tools. Early mass deployment tools like [New Relic’s Centurion](https://github.com/newrelic/centurion), [Spotify’s Helios](https://github.com/spotify/helios), and the [Ansible Docker tooling](https://docs.ansible.com/ansible/latest/collections/community/docker/docsite/scenario_guide.html#ansible-collections-community-docker-docsite-scenario-guide) still work largely like traditional deployment tools but leverage the container as the distribution artifact. They take a fairly simple, easy-to-implement approach. You get a lot of the benefits of Docker without much complexity, but many of these tools have been replaced by more robust and flexible tools, like Kubernetes.

Fully automatic schedulers like [Kubernetes](https://kubernetes.io/) or [Apache Mesos](http://mesos.apache.org/) with the [Marathon scheduler](https://mesosphere.github.io/marathon) are more powerful options that take nearly complete control of a pool of hosts on your behalf. Other commercial entries are widely available, such as [HashiCorp’s Nomad](https://www.nomadproject.io/), [Mesosphere’s DCOS](https://dcos.io/), and [Rancher](https://rancher.com/).[6](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387960304) The ecosystems of both free and commercial options continue to grow rapidly.

#### Immutable Atomic Hosts

One additional idea that you can leverage to enhance your Docker experience is immutable atomic hosts. Traditionally, servers and virtual machines are systems that an organization will carefully assemble, configure, and maintain to provide a wide variety of functionality that supports a broad range of usage patterns. Updates must often be applied via non-atomic operations, and there are many ways in which host configurations can diverge and introduce unexpected behavior into the system. Most running systems are patched and updated in place in today’s world. Conversely, in the world of software deployments, most people deploy an entire copy of their application, rather than trying to apply patches to a running system. Part of the appeal of containers is that they help make applications even more atomic than traditional deployment models.

What if you could extend that core container pattern down into the operating system? Instead of relying on configuration management to try to update, patch, and coalesce changes to your OS components, what if you could simply pull down a new, thin OS image and reboot the server? And then if something breaks, easily roll back to the exact image you were previously using?

This is one of the core ideas behind Linux-based atomic host distributions, like [Red Hat’s Fedora CoreOS](https://getfedora.org/en/coreos), [Bottlerocket OS](https://github.com/bottlerocket-os/bottlerocket), and others. Not only should you be able to easily tear down and redeploy your applications, but the same philosophy should apply for the whole software stack. This pattern helps provide very high levels of consistency and resilience to the whole stack.

Some of the typical characteristics of an immutable or [atomic host](http://bit.ly/2BZr6U3) are a minimal footprint, a design focused on supporting Linux containers and Docker, and atomic OS updates and rollbacks that can easily be controlled via multi-host orchestration tools on both bare-metal and common virtualization platforms.

#### Additional tools

Docker is not just a standalone solution. It has a massive feature set, but there is always a case where someone needs more than it can deliver on its own. There is a wide ecosystem of tools to either improve or augment Docker’s functionality. Some good production tools leverage the Docker API, like [Prometheus](https://prometheus.io/) for monitoring and [Ansible](https://www.ansible.com/) for simple orchestration. Others leverage Docker’s plug-in architecture. Plug-ins are executable programs that conform to a specification for receiving and returning data to Docker. Some examples include Rancher’s [Convoy](https://github.com/rancher/convoy) plug-in for managing persistent volumes on Amazon EBS volumes or over NFS mounts and Weaveworks’ [Weave Net](http://bit.ly/2om9SXo) network overlay.

There are many more good tools that either talk to the API or run as plug-ins. Many of these have sprung up to make life with Docker easier on the various cloud providers. These help with seamless integration between Docker and the cloud. As the community continues to innovate, the ecosystem continues to grow. There are new solutions and tools available in this space on an ongoing basis. If you find you are struggling with something in your environment, look to the ecosystem!

[1](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387831520-marker) Software as a Service

[2](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387826736-marker) Linux Control Groups

[3](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387859424-marker) Linux Namepsaces

[4](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387986496-marker) Command Line Interface

[5](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387786704-marker) Security-Enhanced Linux

[6](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch02.html#idm45865387960304-marker) Some of these commercial offerings have free editions of their platforms.