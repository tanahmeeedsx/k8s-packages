# Kubernetes Packages

A collection of Kubernetes learning resources, configuration files, installation guides, and practical examples for working with Kubernetes clusters.

## Repository Structure

| Directory | Contents |
|---|---|
| `images/` | Kubernetes cluster node diagrams |
| `kind/` | Kind installation, cluster configuration, and dashboard access resources |
| `kubeadm/` | kubeadm-based cluster resources and Kubernetes YAML examples |
| `manifests/` | YAML manifests for Pods, Deployments, Services, Ingress, Jobs, CronJobs, DaemonSets, storage, and more |
| `minikube/` | Minikube installation guide and sample application deployment |
| `task-manager/` | Containerized Task Manager sample application |

## Tools & Technologies

- Kubernetes
- kubectl
- kubeadm
- Minikube
- Kind
- Docker
- YAML

## Getting Started

1. Choose the Kubernetes environment you want to work with: Minikube, Kind, or kubeadm.
2. Follow the setup guide in the relevant directory.
3. Review the YAML examples in `manifests/`.
4. Apply a manifest to your cluster with:

```bash
kubectl apply -f <manifest-file>.yaml
```

Check your cluster resources with:

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

> Make sure you have a running Kubernetes cluster and `kubectl` configured before applying manifests.

## Purpose

This repository serves as a practical reference for learning Kubernetes fundamentals, setting up local clusters, and experimenting with common Kubernetes resources and deployments.
