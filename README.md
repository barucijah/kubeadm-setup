# Handbook for Kube setup

In this Handbook you will find informations about how to setup Kubernetes cluster (version 1.20 is working one for this setup) on Local nodes in your house. Additional to that, you will find out how to deploy Checklens apps and make Cluster ready for training models.

Articles for preparing Server for setting up new Kube cluster using Kubeadm:
- https://medium.com/@pratapagoutham/setting-up-kubernetes-to-use-nvidia-drivers-ed73b07741df

- https://dev.to/preethamsathyamurthy/set-up-a-kubernetes-master-slave-architecture-using-kubeadm-9b3

## Steps for setting up cluster

For properly starting up Kubernetes Cluster you have to install some tools. If you go through both of the articles you will find all necessary tools. Here is the list of the mandatory tools:

- docker.io
- kubelet
- kubeadm
- kubectl
- nvidia-docker2 (for gpu usage)

After installing tool you will have to add daemon.json file in /etc/docker/daemon.json which you can find in Kube-Setup folder on host server.

After executing kubeadm init you will have to install network plugin calico and nvidia plugin on kubernetes cluster.

- calico: `kubectl apply -f kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml`

- nvidia: `kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta4/nvidia-device-plugin.yml`

After this you should test check if nvidia gpu is ready for use. Check it out with those commands:

- `kubectl describe nodes` - You should see in the output of this command gpu parameters and how many gpus you have available
- `kubectl apply -f gpu-test.yaml` - Creating test pod which uses gpu, you will see in output after execution if this was successfull. This yaml you will find in Kube-Setup

Hint: In case gpus are not working, try restarting docker and kubelet.

## Steps for deploying Checklens apps

All necessary yamls you will be able to find in Kube-Setup folder on your host server. \
You will have to setup those apps: (Please go follow the list during installation)
 Please navigate to the Kube-Setup/pmodel-management-ap and do the `kubectl apply -f .`. Do same for all services.

- pmodel-management-api
- redis (follow instructions in redis-installation file in Kube-Setup/redis folder)
- pkube-api (before creating check pkube-api section bellow)
- pmodel-exporter (service)
- pmodel-exporter-worker


From this list you have to expose publicly `pmodel-management-api` (port 32500) and` pmodel-exporter` (port 31364).

After deploying apps, probably you will have to pay attention on credentials. Check in details are they correct.

`pkube-api` \
For sure you will have to add new `PKUBE_TOKEN` for `pkube-api` service. Please go to the `serviceaccounts` and select admin-user, check admin-user-token and past value of token in `pkube-api` in env variable `PKUBE_TOKEN`, then restart pod

Due to that we dont have ingress and we are exposing node ports, it is recommended to add nodeSelector for exposed services. It is already done for jumping-bull. 
``

## Add next node to the cluster

For joining next node to the cluster, you will have to install all necessary tools mentioned befero and prepare node as same as it is explained how to do for first one. \
On master node execute command `kubeadm token create --print-join-command`, as output you will get command which you will execute on the node you want to add in the cluster.

## Add Kubernetes Dashboard

Execute command `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml`

After execution, you will have to expose it publicly. Edit kubernetes-dashboard.Service of this. Change `spec.type` from `ClusterIp` to `NodePort` and add `nodePort: 32477` in the `spec.ports` block.

Then follow the guide how to add user and get creds on this link: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
