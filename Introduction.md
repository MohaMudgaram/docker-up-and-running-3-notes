Docker was first introduced to the world—with no pre-announcement and little fanfare—by Solomon Hykes, founder and CEO of a company then called dotCloud, in a five-minute [lightning talk](http://youtu.be/wW9CAH9nSLs) at the [Python Developers Conference](https://us.pycon.org/) in Santa Clara, California on March 15, 2013.
The project was quickly open-sourced and made publicly available on [GitHub](https://github.com/moby/moby), where anyone could download and contribute to the project.

The project was quickly open-sourced and made publicly available on [GitHub](https://github.com/moby/moby), where anyone could download and contribute to the project.

Docker is a tool that promises to easily encapsulate the process of creating a distributable artifact for any application, deploying it at scale into any environment, and streamlining the workflow and responsiveness of agile software organizations.

## The Promise of Docker
Initially, many people who were unfamiliar with Docker viewed it as some sort of virtualization platform, but in reality, it was the first widely accessible tool to build on top of a much newer technology called containerization.

Docker and Linux containers have had a significant impact on a wide range of industry segments that include tools and technologies like Vagrant, KVM, OpenStack, Mesos, Capistrano, Ansible, Chef, Puppet, and so on. There is something very telling about the list of products that Docker competes with, and maybe you’ve spotted it already. Looking over this list most engineers would recognize that these tools span a lot of different use-cases, yet all of these workflows have been forever changed by Docker.

## Benefits of the Docker Workflow
Here are some more of the benefits you get with Docker and Linux containers:
- Packaging software in a way that leverages the skills developers already have
- Bundling application software and required OS filesystems together in a single standardized image format
- Using packaged artifacts to test and deliver the exact same artifact to all systems in all environments
- Abstracting software applications from the hardware without sacrificing resources

## What Docker Isn’t

In the following list, we explore some of the tool categories that Docker doesn’t directly replace but that can often be used in conjunction to achieve great results:

- Enterprise virtualization platform (VMware, KVM, etc.)
	A container is not a virtual machine in the traditional sense. Virtual machines contain a complete operating system, running on top of a hypervisor that is managed by the underlying host operating system. Hypervisors create virtual hardware layers that make it possible to run additional operating systems on top of a single physical computer system. This makes it very easy to run many virtual machines with radically different operating systems on a single host. With containers, both the host and the containers share the same kernel. This means that containers utilize fewer system resources but must be based on the same underlying operating system (_e.g. Linux_).

- Cloud platform (OpenStack, CloudStack, etc.)
	Like enterprise virtualization, the container workflow shares a lot of similarities—​on the surface—​with more traditional cloud platforms. Both are traditionally leveraged to allow applications to be horizontally scaled in response to changing demand. Docker, however, is not a cloud platform. It only handles deploying, running, and managing containers on preexisting Docker hosts. It doesn’t allow you to create new host systems (instances), object stores, block storage, and the many other resources that are often managed with a cloud platform. That being said, as you start to expand your Docker tooling, you should start to experience more and more of the benefits that one traditionally associates with the cloud.

- Configuration management (Puppet, Chef, etc.)
	Although Docker can significantly improve an organization’s ability to manage applications and their dependencies, it does not directly replace more traditional configuration management. Dockerfiles are used to define how a container should look at build time, but they do not manage the container’s ongoing state, and cannot be used to manage the Docker host system. Docker can, however, significantly lessen the need for complex configuration management code. As more and more servers simply become Docker hosts, the configuration management codebase that a company uses can become much smaller, and Docker can be used to ship the more complex application requirements inside of standardized OCI images.

- Deployment framework (Capistrano, Fabric, etc.)
	Docker eases many aspects of deployment by creating container images that encapsulate all the dependencies of an application in a manner that can be deployed, in all environments, without changes. However, Docker can’t be used to automate a complex deployment process by itself. Other tools are usually still needed to stitch together the larger workflow. That being said, because Docker and other Linux container toolsets, like Kubernetes, provide a well-defined interface for deployment, the method required to deploy containers will be consistent on all hosts, and a single deployment workflow should suffice for most, if not all, of your Docker-based applications.

- Development environment (Vagrant, etc.)
	Vagrant is a virtual machine management tool for developers that is often used to simulate server stacks that closely resemble the production environment in which an application is destined to be deployed. Among other things, Vagrant makes it easy to run Linux software on macOS and Windows-based workstations. Virtual machines managed by tools like Vagrant, assist developers in trying to avoid the common “It worked on my machine” scenario that occurs when the software runs fine for the developer but does not run properly elsewhere. However, as with many of the previous examples, when you start to fully utilize Docker, there is a lot less need to mimic a wide variety of production systems in development, since most production systems will simply be Linux container servers, which can easily be reproduced locally.

- Workload management tool (Mesos, Kubernetes, Swarm, etc.)
	An orchestration layer (including the built-in Swarm mode) must be used to coordinate work intelligently across a pool of Linux container hosts, track the current state of all the hosts and their resources, and keep an inventory of running containers. These systems are designed to automate the regular tasks that are needed to keep a production cluster healthy, while also providing tools that help make the highly-dynamic nature of containerized workloads easier for human beings to interact with.

# Important Terminology

Here are a few terms that we will continue to use throughout the book and whose meanings you should become familiar with:

Docker client
	This is the `docker` command used to control most of the Docker workflow and talk to remote Docker servers.

Docker server
	This is the `dockerd` command that is used to start the Docker server process that builds and launches containers via a client.

Docker or OCI images
	Docker and OCI images consist of one or more filesystem layers and some important metadata that represent all the files required to run a containerized application. A single image can be copied to numerous hosts. An image typically has a repository address, a name, and a tag. The tag is generally used to identify a particular release of an image (_e.g. docker.io/superorbital/wordchain:v1.0.1_).

Linux container
	A container that has been instantiated from a Docker or OCI image. A specific container can exist only once; however, you can easily create multiple containers from the same image. The term Docker container is a misnomer since Docker simply leverages the operating system’s container functionality.

Atomic or immutable host
	An atomic or immutable host is a small, finely tuned OS image, like [Fedora CoreOS](https://getfedora.org/en/coreos), that supports container hosting and atomic OS upgrades.

[1](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch01.html#idm45865387871920-marker) Continuous integration and continuous delivery

[2](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch01.html#idm45865387802016-marker) Open Container Image

[3](https://learning.oreilly.com/library/view/docker-up/9781098131814/ch01.html#idm45865387606176-marker) command line interface