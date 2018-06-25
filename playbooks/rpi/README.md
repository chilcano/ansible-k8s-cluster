# Creating a Kubernetes Cluster on Raspberry Pi 3b+

## 1. Preparing the Raspberry Pis

Install Raspbian and enbale SSH:

```sh
$ touch /Volumes/boot/ssh
```

Now, we have to configure the networking for all Raspberry Pis.
- Setup the Networking (eth0 and wlan0): dhcp, static, set IP, etc.
- Setup the hostname
- Update the hosts file

And we are going to use separate Playbooks to do that.

```sh
$ git clone https://github.com/chilcano/ansible-k8s-cluster.git
$ cd ansible-k8s-cluster/playbooks/rpi/
```

If you have only 1 RPi, then just connect to your computer.

```sh
// ssh to your rpi
$ ssh pi@raspberrypi.local

// get wireless networks available
$ sudo iwlist wlan0 scan

// update the inventory in YAML format
$ cd networking/
$ vi inventory.yml

// check
$ ansible -i inventory.yml -m ping all -k

// config networking
$ ansible-playbook -i inventory.yml main.yml -k
```

Reference:
- https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md
- https://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip-address/37921#37921



// issues with update_cache=true



// rpi dhcp check:

$ cat /etc/dhcpcd.conf | egrep -v "^\s*(#|$)"


$ lsb_release -cs
stretch

// available packages (docker-ce is latest, no docker.io or docker-engine or docker)
$ apt-cache policy docker-ce
$ apt-cache madison docker-ce
$ apt-cache madison docker.io     (older!!!!)

$ apt-cache madison kubeadm
$ apt-cache madison kubernetes-cni

// installed packages
$ dpkg -l | grep kubeadm
$ dpkg -l | grep docker-ce


// install
$ sudo apt-get install docker-ce=18.05.0~ce~3-0~raspbian -yq
$ sudo apt-get install kubeadm=1.10.3-00 -yq


$ sudo journalctl -u kubelet

pi@raspberrypi:~ $ free -h
              total        used        free      shared  buff/cache   available
Mem:           927M         71M        136M         17M        719M        777M
Swap:            0B          0B          0B

master:
-------
$ sudo systemctl stop kubectl
$ sudo kubeadm reset
$ sudo kubeadm init --apiserver-advertise-address=192.168.0.44 --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.10.2

join:
----
$ sudo kubeadm join --token 70250d.80a3c4295584d015 192.168.1.32:6443 --discovery-token-ca-cert-hash sha256:be86d5e72307831efa0a598b6e8b42ccadc1a2f552d67ee0960da87751ebbd0d
$ sudo kubeadm join --token 8db24b.c91adc2663f36876 192.168.1.32:6443 --discovery-token-ca-cert-hash sha256:58f472ce788650a66292c5fd6adb9d474020de9717d923fedf939116f55b0a30
$ sudo kubeadm join --token 8db24b.c91adc2663f36876 192.168.1.32:6443 --discovery-token-unsafe-skip-ca-verification --ignore-preflight-errors all


test:
------------
$ nmap -p- 127.0.0.1 localhost 192.168.0.111

$ sudo journalctl -xeu kubelet
$ sudo systemctl daemon-reload
$ sudo systemctl status kubelet

$ sudo docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-scheduler-arm            v1.10.2             816c40ff51c0        5 weeks ago         43.6MB
k8s.gcr.io/kube-controller-manager-arm   v1.10.2             f67c023adb1b        5 weeks ago         129MB
k8s.gcr.io/kube-apiserver-arm            v1.10.2             c68f5521f86b        5 weeks ago         206MB
k8s.gcr.io/etcd-arm                      3.1.12              88c32b5960ff        3 months ago        178MB
k8s.gcr.io/pause-arm                     3.1                 e11a8cbeda86        5 months ago        374kB

$ sudo docker ps -a

----
Unfortunately, an error has occurred:
	timed out waiting for the condition

This error is likely caused by:
	- The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)
	- Either there is no internet connection, or imagePullPolicy is set to "Never",
	  so the kubelet cannot pull or find the following control plane images:
		- k8s.gcr.io/kube-apiserver-arm:v1.10.2
		- k8s.gcr.io/kube-controller-manager-arm:v1.10.2
		- k8s.gcr.io/kube-scheduler-arm:v1.10.2
		- k8s.gcr.io/etcd-arm:3.1.12 (only if no external etcd endpoints are configured)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
	- 'systemctl status kubelet'
	- 'journalctl -xeu kubelet'
couldn't initialize a Kubernetes cluster

----

using kubectl
--------------
$ sudo su -
$ export KUBECONFIG=/etc/kubernetes/admin.conf
$ kubectl get nodes

or:

$ unset KUBECONFIG
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ sudo kubectl get nodes


setting context:
pi@raspberrypi:~ $ sudo kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?

$ kubectl config use-context kubernetes-admin@kubernetes

pi@raspberrypi:~ $ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
weaveworks/weave-npc                                   2.3.0               e214242c20cf        7 weeks ago         44.5MB
weaveworks/weave-kube                                  2.3.0               10ead2ac9c17        7 weeks ago         88.8MB
gcr.io/google_containers/kube-apiserver-arm            v1.9.0              b1f2ee759790        5 months ago        192MB
gcr.io/google_containers/kube-controller-manager-arm   v1.9.0              77ec4abe2e26        5 months ago        120MB
gcr.io/google_containers/kube-scheduler-arm            v1.9.0              c3c5b959fa2b        5 months ago        54.2MB
gcr.io/google_containers/kube-proxy-arm                v1.9.0              782e6d4f59a1        5 months ago        97.6MB
gcr.io/google_containers/k8s-dns-sidecar-arm           1.14.7              a0cbaa87bdce        7 months ago        36.9MB
gcr.io/google_containers/k8s-dns-kube-dns-arm          1.14.7              81e624687528        7 months ago        44.2MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-arm     1.14.7              76c015d7978c        7 months ago        37.5MB
gcr.io/google_containers/etcd-arm                      3.1.10              020721ca9925        8 months ago        189MB
gcr.io/google_containers/pause-arm                     3.0                 b51c23e6a2ab        2 years ago         506kB
