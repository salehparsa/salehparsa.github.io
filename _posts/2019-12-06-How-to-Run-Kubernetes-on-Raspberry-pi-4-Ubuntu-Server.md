---
layout: post
title: How to Run Kubernetes on Raspberry Pi 4 - Ubuntu Server
categories: [Linux,kubernetes]
tags: [linux, kubernetes, raspberry_pi]
fullview: true
comments: true
---

In this post I just want to write or kind of document how I managed to run Kubernetes on Ubuntu Server image of Raspberry Pi. Just for a short background, I got two _<a href="https://www.raspberrypi.org/products/raspberry-pi-4-model-b/">Raspberry Pi 4 Model B 4GB</a>_ as a birthday Gift from my wife and since the first night I started to play with them.

### Finding an Image

As you may know, you have to find an image that suits you. In general, Raspbian is the official Operating System of Raspberry Pi but somehow it was not suitable for me since I was looking for something more like server operating system. Thus, I decided to go with <a href="https://ubuntu.com/download/raspberry-pi">Ubuntu Server Image</a>. Creating Installation Media was pretty neat for me since I just followed the documention _<a href="https://ubuntu.com/download/iot/installation-media">Raspberry Pi 4 Model B 4GB</a>_

Since I have two Raspberry Pi I have done the same thing twice!

### Home Network

In General you can have dedicated switch for your Raspberry Pi and connect them into this Switch. However, in my case, I just directly connected them into my router. My home network subnet is _192.168.0.50/32_. Take a note of your home address since you will need it for your configuration.

### First Boot

YaY! First boot and now I have both my devices up and running. But hey! Wait! I have no Monitor and due to some reason I have no direct access to my router to find the IP Address of the devices. Yes! Right! Too early for googling for a solution but I have no idea so I started to search and I figure out that I can use Address Resolution Protocol _aka arp_ to see the IP Addresses of my local network: 
{% highlight shell %}
$ arp -a
{% endhighlight %}

Running above told me that following IPs are my Raspberry Pi:

{% highlight shell %}
192.168.0.50
192.168.0.51
{% endhighlight %}

Now I can SSH into my machines and do the initial setup like a normal Ubuntu Server.

### Installing Docker

Docker installation is straight forward and you need to run following:

{% highlight shell %}
$ sudo apt install docker.io
$ sudo usermod -a -G docker $USER
{% endhighlight %}

### Enabling CGroup Memory

One of the biggest challenge for me was to enabling CGroup Memory in Ubuntu image of Raspberry Pi. Unlike other normal places for enabling it, in Raspberry Pi Ubuntu's image you need to append */boot/firmware/nobtcmd.txt* and add following:

{% highlight shell %}
ipv6.disable=1 cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
{% endhighlight %}

#### Note : 
I have disabled IPv6 as well to make the setup easier.


### Ensure iptables tooling does not use the nftables backend

First, enable iptables Filtering on Bridge Devices:

{% highlight shell %}
sudo sysctl net.bridge.bridge-nf-call-iptables=1
{% endhighlight %}

Apart from that, as per mentioned in <a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#ensure-iptables-tooling-does-not-use-the-nftables-backend">documentation of kubernetes</a> , you need to switch the iptables tooling to ‘legacy’ mode to avoid duplicated firewall rules and breaks kube-proxy.

{% highlight shell %}
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
{% endhighlight %}

### Installing Kubernetes

First lets add the Kubernetes package:

{% highlight shell %}
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
{% endhighlight %}

Now is the time to install _kubeadm_:

{% highlight shell %}
$ sudo apt-get install -y kubeadm
{% endhighlight %}

### Configuring _kubeadm_

First of all we have to pull image `k8s.gcr.io/kube-apiserver:v1.16.3` :

{% highlight shell %}
$ kubeadm config images pull
{% endhighlight %}

### When we need to configure second node?

You have to run all above steps on both Raspberry Pi.

### Node 1:

When you initializes a Kubernetes control-plane node, you need to consider assigning a `cidr` for your pod by `--pod-network-cidr`. It specify range of IP addresses for the pod network. If set, the control plane will automatically allocate CIDRs for every node.

{% highlight shell %}
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16
{% endhighlight %}

#### Note :

Do not use your home's subnet for `--pod-network-cidr` because later on when you setting up a flannel, you will end up with having a conflict between flannel IP Range and your home's network. In my case, I initially used home subnet and then I ended up with having flannel IP Address as `192.168.0.1` which is my Router IP Address. It seems flannel is taking a control of the network by picking the first IP Address from the range.

Once you finish initializing your first node you will get instructed to configure `$HOME/.kube/config` as a regular user:

{% highlight shell %}
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
{% endhighlight %}

As well as you will get the message like following to see how you can join this cluster:

{% highlight shell %}
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.50:6443 --token <token here> --discovery-token-ca-cert-hash sha256:<hash value here>
{% endhighlight %}

But hey! Wait we have one more step to do.

### Deploy a pod network to the cluster.

As you can see in the message after initialization, You should now deploy a pod network to the cluster:

{% highlight shell %}
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
{% endhighlight %}

Above gives you more options but I have already selected <a href="https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md">Flannel</a> . Thus, I applied it by following:

{% highlight shell %}
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
{% endhighlight %}

Above deployment will be deploy network configuration pods into your `kube-system` namespace and if you run following you will se the pods:

{% highlight shell %}
$ kubectl --namespace=kube-system get pods -o wide
{% endhighlight %}

### Node 2:

Now it's time to join the first node! You just need to copy and paste from the note after initialization on first node:

{% highlight shell %}
$ kubeadm join 192.168.0.50:6443 --token <token here> --discovery-token-ca-cert-hash sha256:<hash value here>
{% endhighlight %}

Now if you run following in your first node you will se the details of your both nodes:

{% highlight shell %}
kubectl get nodes -o wide
{% endhighlight %}

Hope this blog helps you to configure your cluster easier. I will try to write more.