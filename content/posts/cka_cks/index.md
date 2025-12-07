---
title: "CKA and CKS: Into the Unknown"
date: 2025-12-02 # YYYY-MM-DD
description: "Outlining my journey through the Linux Foundation CKA and CKS certifications."
# weight: 1
# aliases: ["/first"]
draft: true
tags: ["kubernetes", "security", "certifications"]
categories: ["DevOps", "Kubernetes"]
showToc: true
TocOpen: false
hidemeta: false
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: images/ship.jpg
    # alt: "" # alt text
    # caption: "" # display caption under cover
    relative: true # when using page bundles set this to true
    hidden: false # only hide on current single page
    cover.responsiveImages: true
---

## 0x00: The Timeline

Dec 04, 2025: Neat sale, LFC!
Dec 05, 2025: Installing the kube

## 0x01: The Setup

Lemme just ignore all the advice that says to use Minikube to get used to it, surely that'll go well
smug ryan.jpg

Using Proxmox to set up VMs
Installing kubectl

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
sudo apt update && sudo apt install -yqq kubectl
```

Trying to add cluster...nope

Installing kubeadm
`kubeadm will not install or manage kubelet or kubectl for you, so you will need to ensure they match the version of the Kubernetes control plane you want kubeadm to install for you.`

Fail again - have to install container runtimes now? https://github.com/containerd/containerd/blob/main/docs/getting-started.md
```
wget https://github.com/containerd/containerd/releases/download/v2.2.0/containerd-2.2.0-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-2.2.0-linux-amd64.tar.gz 
```

Which means we need a `systemd` service now...
```
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

Service contents:
```
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

Runc:
```
wget https://github.com/opencontainers/runc/releases/download/v1.4.0/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

CNI plugins:
```
wget https://github.com/containernetworking/plugins/releases/download/v1.8.0/cni-plugins-linux-amd64-v1.8.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.8.0.tgz
```



Now that all that's done - integrate with kube time!
```
sudo mkdir -p /etc/containerd
sudo sh -c 'containerd config default > /etc/containerd/config.toml'
ctr --version
```
To make things easier, install `nerdctl` for interfacing with containerd

Now maybe we can `kubeadm init`? Nope, IPv4 forwarding needs to be enabled
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

Finally! `kubeadm init` is pulling images
```
sudo kubeadm config images pull
sudo kubeadm init
```

Well, we tried - now the kubelet isn't running
looks like there's an error - a file missing: `sudo journalctl -xeu kubelet`

`sudo swapoff -a`

Edit fstab entries, comment swap ones
Reboot
`kubeadm init` has a healthy kubelet!

Looks like init finished properly, let's enable `kubectl` functionality:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

networking time! adding a pod:
```

kubectl apply -f cilium-cni.yaml
```


time to join the worker to the mix: 
```
kkubeadm join k8scp:6443 --token v2rzox.o2nomc5gfs2e436l \
        --discovery-token-ca-cert-hash sha256:6efd9556dac334262d41bf829157e0a9696933493a99083f0c1778437a1ef088 --node-name=worker
```

Woo! Joined the node: `kubectl get nodes`



## 0x02: Companion Kube

