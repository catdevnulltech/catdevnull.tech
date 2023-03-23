---
title: "Podman Multiple Networks"
date: 2023-03-22T22:03:29-04:00
draft: false
---

With the release of podman 4.0 the networking backend has be rewritten in rust. Netavark and Ardavark are two tools that have been written to replace the CNI based networking stack in older versions of podman. 

One of the wonderful things that netavark supports is multiple networks. This means we can map multiple networks into a container. This is exterememly useful in telecommunications and data center infrastructure where the need to connect to multiple networks becomes a nessesity. 

Lets explore how this works. 

I have podman 4.4 installed in a RHEL 9 virtual machgine with two networks connected. 

```
[root@localhost ~] ip addr sho
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:32:dc:af brd ff:ff:ff:ff:ff:ff
    inet 10.10.100.50/24 brd 10.10.100.255 scope global dynamic noprefixroute enp1s0
       valid_lft 599sec preferred_lft 599sec
    inet6 fe80::5054:ff:fe32:dcaf/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:d1:b4:52 brd ff:ff:ff:ff:ff:ff
    inet 10.10.110.80/24 brd 10.10.110.255 scope global dynamic noprefixroute enp7s0
       valid_lft 572sec preferred_lft 572sec
    inet6 fe80::418:4b71:25f:9b90/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
5: veth2b2dc630@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman0 state UP group default qlen 1000
    link/ether ee:67:39:7a:ee:44 brd ff:ff:ff:ff:ff:ff link-netns netns-cb6c78e6-ae7d-0d5b-deca-ca98142c5342
    inet6 fe80::ec67:39ff:fe7a:ee44/64 scope link
       valid_lft forever preferred_lft forever
7: podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ee:67:39:7a:ee:44 brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::8a1:b6ff:fef7:6c16/64 scope link
       valid_lft forever preferred_lft forever

```

enp1s0 and enp7s0 are the interfaces connected to each different network. The podman0 network is a bridge that is installed by deafult when podman is installed. 

First lets see what networks podman can see. 

```
[root@localhost ~] podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge

```

Let's create two networks in podman and link them to the interfaces the server has.

```
podman network create -d macvlan -o parent=enp1s0 --subnet 10.10.100.0/24 netone
podman network create -d macvlan -o parent=enp7s0 --subnet 10.10.110.0/24 nettwo

```

We can see the networks have been created with podman network ls 
```
[root@localhost ~]# podman network ls
NETWORK ID    NAME        DRIVER
fdd5ce0ff2be  netone      macvlan
4deff6eacc13  nettwo      macvlan
2f259bab93aa  podman      bridge
```

So let's get a container up and running and connect the networks. 

```
[root@localhost ~] podman run -dt ubi9
[root@localhost ~] podman ps
CONTAINER ID  IMAGE                  COMMAND     CREATED        STATUS            PORTS       NAMES
0a05716b8f5c  localhost/ubi9:latest  /bin/bash   4 seconds ago  Up 3 seconds ago              hungry_chaplygin
```




```
[root@localhost ~] podman network connect --ip 10.10.110.81 nettwo hungry_chaplygin
[root@localhost ~] podman network connect --ip 10.10.100.81 netone hungry_chaplygin
```

If we inspect the conatiner we can see it has three network attachments. One for podman, one for netone, and one for nettwo ( I have trunkcated the output to show only the network componants. )

```
[root@localhost ~]# podman inspect hungry_chaplygin
[
     {
          "Id": "0a05716b8f5c0b3e6959db1aa1f36aa7f12acb407199c1b1f9aba5e2184b078a",
          "Created": "2023-03-22T22:36:09.298901801-04:00",
          "Path": "/bin/bash",
          "Args": [
               "/bin/bash"
          ],
          "State": {
               "OciVersion": "1.0.2-dev",
               "Status": "running",
               "Running": true,
               "Paused": false,
               "Restarting": false,
               "OOMKilled": false,
               "Dead": false,
               "Pid": 4488,
               "ConmonPid": 4483,
               "ExitCode": 0,
               "Error": "",
               "StartedAt": "2023-03-22T22:36:10.040751789-04:00",
               "FinishedAt": "0001-01-01T00:00:00Z",
               "Health": {
                    "Status": "",
                    "FailingStreak": 0,
                    "Log": null
               },
               "CgroupPath": "/machine.slice/libpod-0a05716b8f5c0b3e6959db1aa1f36aa7f12acb407199c1b1f9aba5e2184b078a.scope",
               "CheckpointedAt": "0001-01-01T00:00:00Z",
               "RestoredAt": "0001-01-01T00:00:00Z"
               }
          },
          "Mounts": [],
          "Dependencies": [],
          "NetworkSettings": {
               "EndpointID": "",
               "Gateway": "",
               "IPAddress": "",
               "IPPrefixLen": 0,
               "IPv6Gateway": "",
               "GlobalIPv6Address": "",
               "GlobalIPv6PrefixLen": 0,
               "MacAddress": "",
               "Bridge": "",
               "SandboxID": "",
               "HairpinMode": false,
               "LinkLocalIPv6Address": "",
               "LinkLocalIPv6PrefixLen": 0,
               "Ports": {
                    "80/tcp": null
               },
               "SandboxKey": "/run/netns/netns-deaff2ea-53a0-013b-8749-d63d8947a135",
               "Networks": {
                    "netone": {
                         "EndpointID": "",
                         "Gateway": "10.10.100.1",
                         "IPAddress": "10.10.100.81",
                         "IPPrefixLen": 24,
                         "IPv6Gateway": "",
                         "GlobalIPv6Address": "",
                         "GlobalIPv6PrefixLen": 0,
                         "MacAddress": "12:b9:90:dd:5b:91",
                         "NetworkID": "netone",
                         "DriverOpts": null,
                         "IPAMConfig": null,
                         "Links": null,
                         "Aliases": [
                              "0a05716b8f5c"
                         ]
                    },
                    "newnet": {
                         "EndpointID": "",
                         "Gateway": "10.10.110.1",
                         "IPAddress": "10.10.110.81",
                         "IPPrefixLen": 24,
                         "IPv6Gateway": "",
                         "GlobalIPv6Address": "",
                         "GlobalIPv6PrefixLen": 0,
                         "MacAddress": "c2:85:6d:55:c5:d9",
                         "NetworkID": "newnet",
                         "DriverOpts": null,
                         "IPAMConfig": null,
                         "Links": null,
                         "Aliases": [
                              "0a05716b8f5c"
                         ]
                    },
                    "podman": {
                         "EndpointID": "",
                         "Gateway": "10.88.0.1",
                         "IPAddress": "10.88.0.3",
                         "IPPrefixLen": 16,
                         "IPv6Gateway": "",
                         "GlobalIPv6Address": "",
                         "GlobalIPv6PrefixLen": 0,
                         "MacAddress": "8a:69:f4:5d:61:57",
                         "NetworkID": "podman",
                         "DriverOpts": null,
                         "IPAMConfig": null,
                         "Links": null,
                         "Aliases": [
                              "0a05716b8f5c"
                         ]
                    }
               }
          },


``` 

So let's do some testing. 

```
[root@localhost ~]# podman exec -it hungry_chaplygin bash

```
I have installed iputils so I can use ping to test some network connectivity. 

```
[root@0a05716b8f5c /] dnf install iputils -y

```

```
[root@0a05716b8f5c /]# ping 10.10.100.1
PING 10.10.100.1 (10.10.100.1) 56(84) bytes of data.
64 bytes from 10.10.100.1: icmp_seq=1 ttl=64 time=0.872 ms
64 bytes from 10.10.100.1: icmp_seq=2 ttl=64 time=0.581 ms

[root@0a05716b8f5c /]# ping 10.10.110.1
PING 10.10.110.1 (10.10.110.1) 56(84) bytes of data.
64 bytes from 10.10.110.1: icmp_seq=1 ttl=64 time=0.575 ms

```
As you can see I can ping the gateway for both networks. 

One thing to note here is if you look at iptables you will see a rule in for netavark to allow traffic through it. This is implemented when you create the network in podman and so far it only works with iptables. 

```
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
NETAVARK_FORWARD  all  --  anywhere             anywhere             /* netavark firewall plugin rules */

```

So that is a pretty great feature that will help if you need to connect a container with podman to multiple networks. 
