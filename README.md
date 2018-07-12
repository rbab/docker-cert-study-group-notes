## Meeting Notes

**TODO:**
- make list of exercises that will train on hands-on, prod bugs, etc

## Resources

* [Docker Certification site w/Study Guide](https://success.docker.com/certification)
* [Docker Certified Associate Exam Prep Guide - w/links from exam bullets](https://github.com/Evalle/DCA)
* [Sample test questions](https://djitz.com/certification/docker-certified-associate-test-review-questions-set-1-image-creation/)
* [Interactive Scenarios - Kubernetes, Docker, etc - Katakoda](https://www.katacoda.com/courses/kubernetes)
* [Play with Kubernetes](https://labs.play-with-k8s.com/)
* [Play with Docker Enterprise](https://medium.com/@marcosnils/60-seconds-away-from-docker-ee-13d7cf66713f)
* [Docker Swarm Workshop, J. Petazzo](https://github.com/jpetazzo/container.training)
* [Linux Container from Scratch, Joshua Hoffman](https://vimeo.com/115073286)
* [Wyn's Kubernetes Workshop](https://github.com/excellalabs/docker-workshop-2)

## Docker Deep Dive Notes

1. Containers from 30,000 feet
    * What are containers, etc
    * Windows vs Linux

1.  Docker
    * Kubernetes - leading orchestrator, needed for managing deployment and running apps
    * Docker Engine - plumbing that runs and orchestrates containers
    * EE vs CE
    * Moby - tools that combine to make Docker daemon. Breaks dDOcekr into modular components. Big infra contributors. Golang.
        * Batteries included but removable
    * Open Container Initiative (OCI) - standardized image format and container runtime

1. Installing Docker: 
    - Windows, Mac, Windows Server 2016, Ubuntu, Docker EE, etc
    - Upgrading Docker: 
        - make sure containers have restart policy, 
        - drain nodes if needed (using Swarm mode). 
        - Steps: 
            - Stop Docker daemon, remove old version, install new version, configure new version to auto start when system boots, ensure containers restarted.
    - Docker and storage drivers
        - Every Docker container gets an area of local storage where image layers are stacked and the container filesystem is mounted. By default this is where all container read/write ops occur, making it integral to the perf and stability of every container.
        - Local storage area has been managed by storage driver, which are sometiomes called the graph driver. Docker supports several different storage drivers, each which implements layering and copy-on-write in its own way - impacting perf and stability.
          - Some available drivers:
              - aufs (the original and oldest)
              - overlay2 (probably best choice for future)
              - devicemapper
              - btrfs
              - zfs
          - Windows only supports windowsfilter driver

1. The Big Picture
    - Ops perspective 
        - Docker client, 
        - daemon/engine, 
        - images, running containers
        - attaching to running containers
      - Dev perspective - containerize app

1. The Docker Engine

    Docker used to be made up of:

    - **Docker client**: a cli or lib that talks to the Docker daemon.

    - **Monolithic Docker daemon**: performed __***all***__ docker image workflow tasks and container creation tasks

    - **LXC**: a system-container solution tailored to the linux kernel

    This was not only a bad design, but there were some pretty terrible consequences (e.g. if you stop the docker daemon, all of your running containers are killed!). This lead to a major overhaul to break out the functionality into smaller components (using the UNIX philosophy).

    Docker is now made up of:

    - **Docker client** aka "docker": (unchanged)

    - **Docker daemon** aka "dockerd": responsible for the API, docker workflow tasks, security features, core networking, and volume management. (at least it's not "everything" like it used to be)

    - **containerd**: is a **container execution supervisor** which can manage a complete container lifecycle and enable runtime telemetry.

    - **containerd-shim**: a small process for invoking runc. This is used to keep the STDIO (and other FDs) open for the container incase containerd and/or docker both die. The Shim also reaps the container's exit status and reports the value back to containerd, all without having the be the actual parent of the container's process and do a wait4. This solves the "zombie" reaping problem that has plauged Docker for years.

    - **runc**: is lightweight universal container runtime. Runc is used by containerd for spawning and running containers according to OCI spec. It is essentially a CLI wrapper around libcontainer, the library that replaced LXC. Once the user process starts the runc process exits, allowing for a daemonless container configuration.


    How you start a container with this engine from the ground up:

    ```
    Client <==REST==> Daemon <==GRPC==> containerd > (fork+exec) > containerd-shim + runc > (exec) > your processes

    ```

    Want more? Go nuts: https://ops.tips/blog/run-docker-with-forked-runc/


8. [Dockerizing an app](README-08-containerizing-an-app.md)

9. Docker Compose

Docker-compose is a python application installed on top of the docker engine

A tool called Fig by Orchard. Awesome tool. Used fig command to start, stop and rebuild services,
and view the status of running services. Acquired by docker in 2014 and renamed as docker-compose

Uses a dockerfile

Dockerfile - how to build the image


Docker-compose.yml compose file to describe how Docker should deploy the app
	compose files can be yaml or JSON

Compose will deploy a container for each service

docker-compose.yml contains keys:

Start with version

Top level keys: 
   version - mandatory and always first line. Defines the version of the compose file format (API)
   services - where you define services to use (i.e. a front end service and a db service). Compose will deploy each in its own container
   networks - Tells compose to create new networks. By default, creates bridge networks that are single host (onlny connect to containers on same host) but can define them specifically using the driver property
   volumes - where you store and persist data

Docker-compose commands
-build - build a new image from the docker file
-command - used to run an app as the main app in the container (must exist in the image)
-ports - tells doctor to map port inside the container
-networks - tells which network to attach the derive's container to (should already exist or be defined in the file)
-volumes - tells docker to mount the counter-vol volume to /code(:target).


Book calls a multi container app a Compose app

-d runs in the background

docker-compose up & - to bring the terminal back up


Docker ps to show current state of the docker-compose app.
Docker-compose top to list the processes running inside each container
Docker-compose stop does not remove the application but just stops it
Delete a stopped container with docker-compose rm
Restart with docker-compose restart

Docker-compose down
-volumes aren't deleted - that info is meant to persist
-both stops and deletes it
-only the images, volumes and source code remain


