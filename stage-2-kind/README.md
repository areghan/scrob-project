Stage 2 — Kind Kubernetes Cluster

Goal

Create a local Kubernetes cluster using Kind that provides a realistic multi-node Kubernetes environment for the SCROB project.

The cluster is configured with:

1 control-plane node

2 worker nodes

An ingress-ready control-plane node

HTTP port mapping (80)

HTTPS port mapping (443)

Kubernetes API access

Local path storage

Architecture

                    scrob-cluster
                         |
             +-----------+-----------+
             |                       |
       Control Plane              Workers
             |                 +-----+-----+
             |                 |           |
             v                 v           v
    scrob-cluster-        scrob-cluster-  scrob-cluster-
    control-plane         worker          worker2
             |
       ingress-ready=true
             |
        +----+----+
        |         |
       :80       :443
        |         |
        +----+----+
             |
          Host OS

Directory Structure

stage-2-kind/
├── kind-cluster.yaml
└── README.md

Kind Configuration

The cluster configuration is stored in:

stage-2-kind/kind-cluster.yaml

The configuration creates:

name: scrob-cluster

with the following nodes:

Node

Role

scrob-cluster-control-plane

Control Plane

scrob-cluster-worker

Worker

scrob-cluster-worker2

Worker

Ingress-Ready Node

The control-plane node is labelled:

ingress-ready=true

This allows an ingress controller to be scheduled onto this node in a later stage.

The label is configured using:

kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"

Verify the label with:

kubectl get nodes --show-labels

Expected output includes:

ingress-ready=true

Port Mappings

The control-plane node maps HTTP and HTTPS traffic from the host into the Kind cluster.

HTTP

Host :80 → Control Plane :80

HTTPS

Host :443 → Control Plane :443

This is configured in kind-cluster.yaml using:

extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP

  - containerPort: 443
    hostPort: 443
    protocol: TCP

These mappings allow applications exposed through an ingress controller to eventually be accessed from the host machine.

Kubernetes API

The Kubernetes API server is exposed locally.

The current API mapping is:

127.0.0.1:44013 → Control Plane :6443

The host port may change when the cluster is recreated because Kind can assign an available host port.

Cluster Creation

The cluster was created using:

kind create cluster --config stage-2-kind/kind-cluster.yaml

Verify the cluster exists:

kind get clusters

Expected:

kind
mealie-cluster
scrob-cluster

kubectl Context

Verify the current Kubernetes context:

kubectl config current-context

Expected:

kind-scrob-cluster

Cluster Verification

Check cluster information

kubectl cluster-info --context kind-scrob-cluster

The Kubernetes control plane should be reported as running.

Check nodes

kubectl get nodes -o wide

Expected:

NAME                          STATUS   ROLES           VERSION
scrob-cluster-control-plane   Ready    control-plane   v1.37.0
scrob-cluster-worker          Ready   <none>          v1.37.0
scrob-cluster-worker2         Ready   <none>          v1.37.0

All three nodes should have:

STATUS = Ready

Check node labels

kubectl get nodes --show-labels

The control-plane node should contain:

ingress-ready=true

Check system pods

kubectl get pods -A

The following Kubernetes components should be running:

CoreDNS

etcd

kube-apiserver

kube-controller-manager

kube-scheduler

kube-proxy

kindnet

local-path-provisioner

All should report:

STATUS = Running

Check services

kubectl get svc -A

The default Kubernetes services should be present, including:

kubernetes
kube-dns

Verify Docker port mappings

docker port scrob-cluster-control-plane

Expected:

80/tcp -> 0.0.0.0:80
443/tcp -> 0.0.0.0:443
6443/tcp -> 127.0.0.1:<dynamic-port>

The Kubernetes API port is dynamic, while ports 80 and 443 are explicitly configured.

Current Environment

The cluster was verified with:

Kubernetes: v1.37.0
Kind: v0.33.0-alpha
Container Runtime: containerd 2.3.4
Operating System: Debian GNU/Linux 13 (Trixie)
Environment: WSL2
Architecture: amd64

Troubleshooting

Check cluster status

kind get clusters

Check current context

kubectl config current-context

Check nodes

kubectl get nodes

Check all pods

kubectl get pods -A

Check cluster information

kubectl cluster-info

Check Kind containers

docker ps

Check port mappings

docker port scrob-cluster-control-plane

Stage 2 Completion Criteria

Stage 2 is considered complete when:

Kind is installed

kind-cluster.yaml is created

scrob-cluster is created

One control-plane node exists

Two worker nodes exist

All nodes report Ready

Control-plane node has ingress-ready=true

Host port 80 maps to container port 80

Host port 443 maps to container port 443

Kubernetes system pods are running

kubectl is using kind-scrob-cluster

Cluster information has been verified

Next Stage

Stage 2 provides the Kubernetes foundation for the SCROB project.

The ingress-ready configuration and port mappings prepare the cluster for deploying an ingress controller and exposing applications in later stages.
