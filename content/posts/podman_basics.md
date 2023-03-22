---
title: "Podman - A basic tutorial"
date: 2022-12-06T10:26:34-05:00
draft: false
---

## Podman basics

Podman is a container engine tool that provides a seamless and secure way to create, run, and manage containers and container images. Unlike other container engines, Podman does not require a daemon to be running in the background, which makes it a popular choice for devs and sys admins who want a lightweight and efficient container solution. 

Podman has several advantages over Docker that i will highlight here: 

- No daemon requirement: Unlike Docker, Podman does not require a background daemon to be running in order to manage containers. This makes it easier to install and use, and also eliminates potential security vulnerabilities.

- Rootless mode: Podman allows users to run containers as non-root users, which enhances security and reduces the risk of privilege escalation attacks.

- Better integration with system tools: Podman integrates better with system tools such as systemd and SELinux, which makes it easier to manage containers in production environments.

- Support for Kubernetes: Podman provides native support for Kubernetes, which enables users to seamlessly switch between Kubernetes and Podman without needing to change their workflows.

- Improved performance: Podman has been shown to have better performance than Docker in some benchmark tests, especially in situations where large numbers of containers are being managed.

Overall, Podman's focus on security, simplicity, and performance make it a compelling alternative to Docker for container management.

After that wonderful expliantion i am sure you are ready to dive in right!?! Great so how do i install and use it? Glad you asked!

Podman installation 
```bash
[root@localhost ~] sudo dnf install podman 
```

Once podman is installed we can run a podman ps if any pods are running. You will need to make sure you are root to run 

```bash
[root@localhost ~] podman ps 
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

```

Ok nothing is running. So let's create a container and run it. Podman has the ability to build containers from an image specfication called a Containerfile ( Dockerfile will also work ). 

So let's build a SIMPLE container image using a UBI image.

```bash

[root@localhost ~] vi Containerfile 
```

The contents of the file should look something like this;
```bash
FROM       ubi9/ubi:9.1
LABEL      description="Yet another ngnix container image"
MAINTAINER nulltech <nulltech@catdevnull.tech>
EXPOSE     80
CMD        ["/bin/bash"]

```

Once the Containerfile is created we can have podman build it. The -t switch is a tag so we can assign a name the image file. 

```bash
[root@localhost ~] podman build -t ubi9 .
```

Once the build completes one can see the image using this command. 
```bash
[root@localhost ~] podman images
REPOSITORY                           TAG         IMAGE ID      CREATED         SIZE
localhost/ubi9                       latest      f46297cc8522  47 seconds ago  219 MB

```

Now we simply need to run the image. The -dt allows the conatiner to run in the background and stay running. 

```bash
[root@localhost ~] podman run -dt ubi9
86358abbc3f805d3356b9d6da78bc28e8372f4a1caa47df49ab819889a28c0c9

```

Now if we check with ps we can see a running container image

```bash
[root@localhost ~] podman ps
CONTAINER ID  IMAGE                  COMMAND     CREATED        STATUS            PORTS       NAMES
86358abbc3f8  localhost/ubi9:latest  /bin/bash   9 seconds ago  Up 9 seconds ago              lucid_lederberg
```

We can get into this image and have a look around like this. Using the -i here will make the command interactive. 

```bash
[root@localhost ~] podman exec -it lucid_lederberg /bin/bash
[root@86358abbc3f8 /]#

```

Running commands in the container would work as normal as long as the utility you are running has been included in the container itself. 

```bash
[root@86358abbc3f8 /] ls
afs  bin  boot	dev  etc  home	lib  lib64  lost+found	media  mnt  opt  proc  root  run  sbin	srv  sys  tmp  usr  var
```

You can logout of the container by typing exit. 

```bash
[root@86358abbc3f8 /] exit
exit
[root@localhost ~]
```

In the next post I will be digging into more complicated container builds. 








