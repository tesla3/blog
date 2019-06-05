+++ 
draft = false
date = 2019-06-02T09:19:43-07:00
title = "Linux Container and Kubernetes Pod Demystified"
description = ""
slug = "" 
tags = ["Linux container", "Kubernetes", "pod", "namespace", "cgroup", "control group", "SELinux", "SECComp"]
categories = []
externalLink = ""
series = []
+++

# Intro
Linux container is in vogue now. We had an internal tech seminar @work on container and kubernetes. Many, your truly included, had difficulties to articulate and answer this question: what the heck, exactly, is a container? Inquisitive-minded me started to dig. This blog is an attemp to put in one place all the understanding I accumulated along the journey. Here I focus on Linux container.

# How come we are so confused

I was confused for quite a bit on container. Lots of others too! [Let's have a laugh first.](https://circleci.com/blog/getting-your-manager-to-say-yes-to-devops-tools/)

## [From the mouth of horse](https://www.docker.com/resources/what-container)

> A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. 

Here "container" should read "container image". Nice! Strike one.

## Is a container a process or a group of process?

I heard: a container is a process. Someone indeed claims to be so (note date: 2015 something:( )

{{< figure src="/images/container-is-a-process.png" caption="Container is a process" >}}

Not entirely satisifed, let's head to almighty Google and search "what is linux container". The top one is from "redhat.com" 

{{< figure src="/images/google-what-is-container.png" caption="Google search \"what is linux container\"" >}}

Hmmm, "a set of processes", not **a** process? Now, I am serious confused. Probably you too.

Going down the list, but still on the first page,  I clicked into an article titled "Everything You Need to Know about Linux Containers, Part I: Linux Control Groups and Process Isolation" from reputable source "linuxjournal.com":
 
{{< figure src="/images/everything-container.png" caption="Everything You Need to Know about Linux Container">}}

No quick and clear one sentence answer. It then goes right into advanced topic "Linux Control Group"...

No success yet!

Over many nights of "researching", the picture of container gets more and more clear. Someone finally wrote a very nice (read: long) blog [The Missing Introduction To Containerization] (https://medium.com/faun/the-missing-introduction-to-containerization-de1fbb73efc5). It talked about the history of "containing process" from chroot/jail, LinuxVserver, Solaries Zones, LXC, docker, lots of good stuff.

But, what is a container? Is it a process or a group process?

## Container is not "a real thing"
I do not blame these authors. We, humanoids, grok concrete stuff, better than abstract concept. We like quick and short anwser better than long explaination. Unfortunately, containers happen to be a complex "concept".

Overuse and overhype around container does not help, as acknowledged in [A sysadmin's guide to containers] (https://opensource.com/article/18/8/sysadmins-guide-containers):

> The term "containers" is heavily overused. Also, depending on the context, it can mean different things to different people.

In the same article, Dan described *container*: 

> Traditional Linux containers are really just ordinary processes on a Linux system. These groups of processes are isolated from other groups of processes using resource constraints (control groups [cgroups]), Linux security constraints (Unix permissions, capabilities, SELinux, AppArmor, seccomp, etc.), and namespaces (PID, network, mount, etc.).  **Do not confuse linux kernel namespace with k8s's namespace**

Althought it does not answer my question "is container a process or multiple processes?", it gives decent summary about containers.

The true is **Containers are not a real thing!**

{{< figure src="/images/container-is-not-a-real-thing.png" caption="Containers is not a real thing">}}

From [Jessie Frazelle's post from Mar 2017] (https://blog.jessfraz.com/post/containers-zones-jails-vms/):

>A “container” is just a term people use to describe a combination of Linux namespaces and cgroups. Linux namespaces and cgroups ARE first class objects. NOT containers.

Before we dive into, let's reflect for a moment on our good and old friend *process*?  Everyone knows what a process is in Linux. Processes are our good old friend. We inspect them us *ps* everyday. They have well defined and more importantly well understood boundaries: I cannot kill you process (at least not directly). They are permission controlled consistently with other part of OS like file system. Processes are "well organized" and form a tree rooted at PID 1, the *init* process. Tools like htop and pstree shows you the tree. The interaction between processes have been standardized long time ago as part of POSIX for various IPC mechanisms such as shared memory, message queue, mutux lock, semaphore, etc.

So, processes feel like concrete and "a real thing". But that kind of simple days are long gone. Dan explains in [A sysadmin's guide to containers] (https://opensource.com/article/18/8/sysadmins-guide-containers) (emphasis is mine)

> If you boot a modern Linux system and took a look at any process with cat /proc/PID/cgroup, you see that the process is in a cgroup. If you look at /proc/PID/status, you see capabilities. If you look at /proc/self/attr/current, you see SELinux labels. If you look at /proc/PID/ns, you see the list of namespaces the process is in. **So, if you define a container as a process with resource constraints, Linux security constraints, and namespaces, by definition every process on a Linux system is in a container**. This is why we often say [containers are Linux, Linux are container] (https://www.redhat.com/en/blog/containers-are-linux). 

Some confusion:
> **A Linux container is nothing more than a process that runs on Linux**. It shares a host kernel with other containerized processes. So what actually makes this process a "container"? 

> First, each containerized process is isolated from other processes running on the same Linux host, using kernel namespaces. Kernel namespaces provide a virtualized world for the container processes to run in. For example, the "PID" namespace causes a containerized process **to only see other processes inside of that container**, but not processes from other containers on the shared host. Additional security isolation is provided by kernel features like dropped capabilities, read-only mounts and seccomp. Additional file system security isolation is provided by SELinux in distributions like Red Hat Enterprise Linux. This isolation helps ensure that one container cannot exploit other containers or take down the underlying host.

Wait a minute! A container can have more than one or only process? This is one of the biggest confusion I had when I read about "container". If a container is a "contained" process, then we cannot put another process in that! The author in the above probably meant to say 

> ...For example, the "PID" namespace causes a containerized process to only see other processes ~~inside of that container~~ **sharing a same PID namespace**, but not processes ~~from other containers on the shared host~~ **not sharing a same PID namespace**. 

So, can we conclude **A containter can contain only one process**? But a container might share some namespace with other containers.

## Namespace Sharing

> Sharing Namespaces

> Since containers are made with specific building blocks of namespaces this allows for doing some super neat things like sharing namespaces.

> There are many different namespaces but I will give a couple examples.

> This specific example can be seen in a demo by Arnaud Porterie from our talk at Dockercon EU in 2015. You can have your application running in one container, then in a different container sharing a net namespace you can run wireshark and inspect the packets from the first container.

> You could also do the same with sharing a pid namespace, except instead of running wireshark you can run strace and debug your application from an entirely different container.

If sharing a network space means "network interface sharing", does sharing PID namespace mean "process re-entering"? 

### Sharing PID(Process) Namespace

It turns out in reality it is more complex due to different container launcher (run-time) implemented different. For example, in [The Curious Case of Pid Namespaces And How Containers Can Share Them](https://hackernoon.com/the-curious-case-of-pid-namespaces-1ce86b6bc900)

> Because kubernetes also supports pods, it illustrates a similar approach using docker. Due to some of the aforementioned issues with pid namespaces, kubernetes doesn’t yet share pid namespaces between containers in the same pod. This is unfortunate, because that means processes in the same pod cannot signal each other. In addition, each container in the pod has the aforementioned init problem: every container process will run as pid 1.

Looks this limitation was limited since k8s v1.10 according to [hidden-gems-1.10](https://blog.jetstack.io/blog/hidden-gems-1.10/). See more details in [latest k8s doc on this](https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/) 

> 1.10 adds alpha support for shared process namespaces in a pod. 

> Sharing the PID namespace inside a pod has a few effects. Most prominently, processes inside containers are visible to all other containers in the pod, and signals can be sent to processes across container boundaries. This makes sidecar containers more powerful (for example, sending a SIGHUP signal to reload configuration for an application running in a separate container is now possible).

While we are at it, let's check what namespaces are shared among containers in a same pod.

Official [k8s doc](https://kubernetes.io/docs/concepts/workloads/pods/pod/) says:

> Pods provide two kinds of shared resources for their constituent containers: networking and storage.

So, it did not mention **process namespace sharing** 

For [podman pod](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/), which is supposed to mimic k8s pod: Pods are a group of one or more containers sharing the same network, pid and ipc namespaces.

{{< figure src="/images/podman-pod-architecture.png" caption="Podman pod architecture" >}}

But, [Ian](https://www.ianlewis.org/en/what-are-kubernetes-pods-anyway) diagram for k8s pods shows slightly differently:
{{< figure src="/images/pods.png" caption="Pods: by Ian Lewis, what are k8s pods anyway">}}

shows containers in a pod shares ipc, network (tcp/ip stack and network interface), hostname, process/pid namespaces. I think they are also share mount namespace.



Note that resource contraints are always enforced on container level by k8s, according to [Doc: Managing Compute Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)

Note: empty pod (see podman run local pod for): a pod has only infra (pause) container. See [podman pod](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)

Still with me? Want to dive deep into pod, pid 1, zoombie process reaping, rkt vs runc, etc? Go straight to [Almighty pause container](https://www.ianlewis.org/en/almighty-pause-container).

Now, we start to see why "what is a container" is such a hard question to give simple answer. It is abstract (not a real thing). It is overused and overloaded (I will explain more later on). In addition container technology is quick evolving. The same term container means different construct in 2019 than how a container is contstuct and ran in 2015, especially many processes have been made in security side. 

Before we ~~open up the can of worm~~ dive into the topic of security, ~~let me share~~ [~~an pretty thorough presentation on "container"~~](https://boot2podman.github.io/assets/ContainersWithoutDocker.pdf) haven't found yet:( . But here is [a reasonably good but four years old (read: may obsolete and hence inaccurate) intro](https://www.youtube.com/watch?v=YsYzMPptB-k#t=18m10s). It (@18:00) explains the cgroup and namespace is a comprise between two solutions.. For more thorough treatment on containers and k8s, [Ian's blog](https://www.ianlewis.org/) seems to be best!

## Linux Container ~~Security~~ Lack of Security

Container is about process isolation. Security is a crucial aspect for isolation, don't we agree?

Talking about container security. Many smart people and deep pocket corps (read: redhat, google) are actively working on it. Jessie, nowdays, is working on ["hardening containers"](https://www.linkedin.com/in/jessie-frazelle/). As described by Dan above, container isolation goes beyond cgroup and namespace. Mechanism such as Seccomp, AppArmor, and SELinux are used to further isolate containers. Ready for a deep dive? Watch [this youtube from Daniel Walsh] (https://www.youtube.com/watch?v=ZE2nI1SwSVk). If you are a serious developer or want to call yourself "hacker" and not live under a rock, you should be pretty "familiar" with CVE discovered constantly on containers. Just couple of days ago, we got [Breaking Out of rkt - 3 New Unpatched CVEs](https://www.twistlock.com/labs-blog/breaking-out-of-coresos-rkt-3-new-cves/). Security is hard and complicated: Linux container is OS level virtualization. We haven't fully solved this OS level isolation problem yet. One of main source for container vulnarabilities is only until recently, containers have to be built and launched by root. [People are working towards "rootless" container] (https://www.slideshare.net/AkihiroSuda/rootless-containers).


Remember our good old "before-containerized" processes live happily in Linux user land. They are trusted by default. Proof: a normal user can do a fork bomb to exhaust process id limit. Or she can start a process and exhaust system memory by calling malloc like crazy. As a collateral damage, it may kill a few other processes, not neccessarily hers, by triggering kernel's OOM killer. User space is a shared space among all users. All processes are like living in a college dorm: roomates can be nice but some time noisy. 

{{< figure src="/images/process-in-same-machine-hostel.png" caption="Services (DB, Web Server, etc) on a Same Machine Without Isolation: hostel">}}

Container, process isolation on OS level, aims to estabish walls between processes. They are more like living in apartment buildings.

{{< figure src="/images/container-apartment.png" caption="Container, Process with OS Isolation: Apartment">}}

Cgroup and namespace are not strong enough isolation for security purpose. SELinux and other security mechnisms are often recommendated to be enforced for a sysem to run containers:
{{< figure src="/images/container-park.png" caption="Container Security with SELinux turned off">}}

For completness, again stolen from Daniel's presentation. Physical machine isolation is like living in single family houses, while virtual machine like Duplex:

{{< figure src="/images/physical-machine.png" caption="Physical Machine Isolation: Singhhe Family Hhouse">}}
{{< figure src="/images/virtual-machine-duplex.png" caption="Virutal Machine Isolation: Duplex">}}






