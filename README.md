# All Pi config

## Enable cgroups and IP
sudo nano /boot/firmware/cmdline.txt
ADD:
```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory ip=<ip name here>::<gateway here>:<subnet>:<node name>:eth0:off
```
Example:
```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory ip=192.168.2.203::192.168.2.254:255.255.255.0:rasnode02:eth0:off
```

## Enable ssh

ADD:
```
touch ssh
```

## Speed up the pi
sudo nano /boot/firmware/config.txt
ADD:
```
over_voltage=2
arm_freq=1750
```

## Enable nfs
NFS preps:
```
sudo apt-get install nfs-common -y
```

## Configure ip
IP configuration:

sudo nano /etc/netplan/00-installer-config.yaml
```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      dhcp6: no
      addresses: [192.168.XXX.YYY/24]
      gateway4: 192.168.XXX.YYY
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

## Configure iptables / networking
iptables setup:
```bash
sudo iptables -F
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo reboot
```
Configure hostname. 
```bash
sudo nano /etc/hostname
```



# Setup k8s


## Master node
```bash
curl -sfL https://get.k3s.io | sh -s - --flannel-backend=ipsec
```
Master node requires a flannel-backend of iptables, nftables does not work.
https://rancher.com/docs/k3s/latest/en/advanced/#enabling-legacy-iptables-on-raspbian-buster


## Node

Get output from __master__ node:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

Run command on slave-node:
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<master node>:6443 K3S_TOKEN=<token> sh -s -
```

# Finish cluster config remotely

Get the k3s kube config from the master node:
```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```
Save the output somewhere locally.  
Configure lens accordingly:
https://k8slens.dev/


LENS - metrics fix:
https://opensource.com/article/20/6/kubernetes-lens


# Personal setup.

Enable system upgrades
```bash
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/download/v0.7.3/system-upgrade-controller.yaml
kubectl apply -f upgrade.yaml
```
Add claims:
```bash
kubectl apply -f pv-influx.yaml
kubectl apply -f pv-config.yaml
kubectl apply -f pv-media.yaml
kubectl apply -f pv-maria.yaml
kubectl apply -f pv-postgres.yaml
```

# Add nfs
```shell
kubectl create namespace nfs
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -n nfs -f nfs-values.yaml
```

Any other host:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx/

helm upgrade ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx -f nginx-values.yaml --set controller.metrics.enabled=true --set controller.metrics.serviceMonitor.enabled=true --set controller.metrics.serviceMonitor.additionalLabels.release="prometheus"


helm upgrade esphome --namespace esphome ./ha-ESP/esphome-8.4.2.tgz -f ./ha-ESP/esp-values.yaml --install
helm upgrade sonarr --namespace download ./charts/helm/sonarr-16.3.2.tgz -f sonarr-values.yaml --install
helm upgrade radarr --namespace download ./charts/helm/radarr-16.3.2.tgz -f radarr-values.yaml --install
helm upgrade sabnzbd --namespace download ./charts/helm/sabnzbd-9.4.2.tgz --version 9.4.2 -f sabnzbd-values.yaml --install
helm upgrade plex ./charts/helm/plex-6.4.3.tgz -f plex-values.yaml --install
helm upgrade home ./charts/helm/home-assistant-13.4.2.tgz -f home-values.yaml --install
helm upgrade lancache ./charts/helm/lancache-0.6.2.tgz -f lancache/values.yaml -n lancache --install

helm upgrade pihole mojo2600/pihole --version 2.5.8 --namespace pihole -f pihole-values.yaml --install
helm upgrade --namespace monitoring --install kube-stack-prometheus prometheus-community/kube-prometheus-stack -f prometheus-values.yaml --install


```

# Custom setup for zwave
Use the default lite image

```bash
# zwave
kubectl label nodes rasnode02 kubernetes.io/role=worker
kubectl label nodes rasnode03 cluster.dissi.me/zwave=true
kubectl label nodes rasnode02 cluster.dissi.me/zigbee=true

```

# Custom setup for plex hardware encoding

```bash
kubectl apply -k https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/nfd
kubectl apply -k https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/gpu_plugin?ref=v0.23.0
# Daemon sets change "nfd-worker" pull from 
# k8s.gcr.io/nfd/node-feature-discovery:v0.10.1
# to 
# docker.io/raspbernetes/node-feature-discovery:v0.9.0-minimal
```

# (additional) Master setup
```bash
#!/usr/bin/env sh
main() {
  export K3S_KUBECONFIG_MODE="644"
  export K3S_URL="https://rasmaster.local:6443"
  # Get token from /var/lib/rancher/k3s/server/node-token
  export K3S_TOKEN=""

  curl -sfL https://get.k3s.io | sh -s - server --disable traefik --disable servicelb
  return 0
}
main

```

# Node setup

```bash
#!/usr/bin/env sh
main() {
  export K3S_KUBECONFIG_MODE="644"
  export K3S_URL="https://rasmaster.local:6443"
  # Get token from /var/lib/rancher/k3s/server/node-token
  export K3S_TOKEN=""

  curl -sfL https://get.k3s.io | sh -s - --disable traefik --disable servicelb
  return 0
}
main

```



# MetalB setup

Experimental
https://blog.xirion.net/posts/metallb-opnsense/

```shell
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --version 0.12.1 -n metallb-system

```


# Ingress setup


```shell
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm upgrade --install external-dns external-dns/external-dns --namespace external-dns -f dns/values.yaml
```