# How to enter a running pod's network namespace

A common problem in kubernetes is it can be difficult to debug problems related to pod networking from the pod's perspective.
This is particularly true if the pod is distroless and does not include busybox or coreutils.

To remidy this, we can use `nsenter` to drop into the pod's network namespace.

We can experiment with this using `kind`.

```sh
root@localhost:~$ kind create cluster
[truncated output]
root@localhost:~$ docker ps
983360964d6d   kindest/node:v1.21.1   "/usr/local/bin/entr…"   4 minutes ago   Up 4 minutes   127.0.0.1:35499->6443/tcp   kind-control-plane
root@localhost:~$ docker exec -it 983360964d6d /bin/bash
root@kind-control-plane:/# 
```

We are now in the `kind` host conntainer that is running our kubernetes infrastructure.

We can run a pod as follows:

```sh
kubectl run --image=nginx nginx
```

We now have a pod named nginx running.

We can inspect nginx container using `crictl`

```sh
root@kind-control-plane:/# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
248368db5ceb1       605c77e624ddb       7 minutes ago       Running             nginx                     0                   fe4e75bd97469
d612fb5c0b9f0       296a6d5035e2d       14 minutes ago      Running             coredns                   0                   97630d460c6b7
4bd43b049d5a9       296a6d5035e2d       14 minutes ago      Running             coredns                   0                   c231211191d22
a09e832a1d4ea       e422121c9c5f9       14 minutes ago      Running             local-path-provisioner    0                   c4624a3458a5f
c0ef9bb40d242       6de166512aa22       14 minutes ago      Running             kindnet-cni               0                   ba146b6e8f6fd
84d2442276ae7       0e124fb3c695b       14 minutes ago      Running             kube-proxy                0                   58cdae2bc568c
3e7470c4bd190       0369cf4303ffd       15 minutes ago      Running             etcd                      0                   c1951226c2fc0
00e2a711ca10e       96a295389d472       15 minutes ago      Running             kube-controller-manager   0                   52f6e5a717ed8
0a69eb6a4a9bb       94ffe308aeff9       15 minutes ago      Running             kube-apiserver            0                   7b0f818f4e6c6
77f662166de8d       1248d2d503d37       15 minutes ago      Running             kube-scheduler            0                   51e93e3da793f
```

We now have the container id. We can inspect the id and grab the PID.

```sh
root@kind-control-plane:/# crictl inspect 248368db5ceb1 | grep pid
    "pid": 2991,
            "pid": 1
            "type": "pid"
```

We now have the PID, we can enter the namespace with a new bash process.
Any applications started with this bash session will use the pod's networking stack.

```sh
root@kind-control-plane:/# nsenter -t 2991 -n /bin/bash
root@kind-control-plane:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether aa:b5:71:c4:9e:5c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.5/24 brd 10.244.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a8b5:71ff:fec4:9e5c/64 scope link 
       valid_lft forever preferred_lft forever
```

We can see we have an IP address of 10.224.0.5.

We can validate the pod's IP matches the current IP by checking the pod's IP in kubectl

```sh
root@kind-control-plane:/# kubectl get pods nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          13m   10.244.0.5   kind-control-plane   <none>           <none>
```

We can return back to the host networking stack by exiting the shell.

```sh
root@kind-control-plane:/# exit
```
