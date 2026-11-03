# Kubernetes Packages

A beginner-friendly collection of Kubernetes learning resources, cluster setup guides, YAML manifests, and practical deployment examples using **Minikube, Kind, and kubeadm**.

This repository is organized so you can learn Kubernetes concepts, create a local cluster, deploy sample workloads, expose applications, and experiment with common Kubernetes resources.

## Repository Structure

```text
kubernetes-packages/
├── images/
│   ├── master.png
│   └── worker.png
├── kind/
│   ├── dashboard-admin-user.yml
│   ├── installer.sh
│   ├── kind-config.yml
│   └── README.md
├── kubeadm/
│   ├── deployment.yaml
│   ├── namespace.yaml
│   ├── pod.yaml
│   ├── README.md
│   └── service.yaml
├── manifests/
│   ├── cron-job.yml
│   ├── daemonsets.yml
│   ├── deployment.yml
│   ├── ingress.yml
│   ├── job.yml
│   ├── namespace.yml
│   ├── persistentVolume.yml
│   ├── persistentVolumeClaim.yml
│   ├── pod.yml
│   ├── replicasets.yml
│   └── service.yml
├── minikube/
│   ├── deploy-sample-application/
│   │   ├── 01-create-deployment.yaml
│   │   ├── 02-service-to-expose-deployment.yaml
│   │   ├── 03-update-or-rollout-deployment.yaml
│   │   └── README.md
│   └── minikube_installation.md
└── task-manager/
    ├── Dockerfile
    ├── index.html
    └── README.md
```

### What each directory contains

| Directory | Purpose |
|---|---|
| `images/` | Diagrams showing a Kubernetes control-plane (master) and worker-node layout. |
| `kind/` | Kind installation helper, multi-node cluster configuration, and Dashboard access resources. |
| `kubeadm/` | Example resources and instructions for deploying a cluster/application with kubeadm. |
| `manifests/` | Individual Kubernetes YAML examples for learning common resource types. |
| `minikube/` | Minikube installation instructions and a step-by-step sample application deployment. |
| `task-manager/` | A small static Task Manager application packaged into an Nginx Docker image. |

## 1. Prerequisites

For the local learning workflows, you should have:

- A Linux machine (the Kind installer in this repository is intended for Linux/Ubuntu).
- Docker installed and running.
- `kubectl` installed.
- Minikube or Kind, depending on the workflow you choose.
- Git to clone this repository.

For the kubeadm workflow, you need separate Linux machines or virtual machines for the control-plane and worker nodes, along with network connectivity and the required firewall rules.

> You do not need to install Minikube, Kind, and kubeadm all at once. Choose one cluster setup method first.

## 2. Clone the Repository

Run these commands in your terminal:

```bash
git clone https://github.com/tanahmeeedsx/kubernetes-packages.git
cd kubernetes-packages
```

Unless a section says otherwise, run the repository commands from the **repository root** (the `kubernetes-packages/` directory).

## 3. Choose Your Kubernetes Environment

This repository contains three different cluster setup approaches:

| Tool | What it does | Where to start |
|---|---|---|
| Minikube | Runs a Kubernetes cluster locally, commonly using Docker as its driver. | `minikube/minikube_installation.md` |
| Kind | Runs Kubernetes nodes as Docker containers; useful for multi-node local labs. | `kind/README.md` |
| kubeadm | Bootstraps a Kubernetes cluster on Linux machines or VMs. | `kubeadm/README.md` |

Do not apply Kind cluster configuration with `kubectl`. A Kind configuration file creates the cluster; Kubernetes resource manifests are applied to an existing cluster.

## 4. Minikube Setup and Sample Deployment

### 4.1 Install and start Minikube

Follow the installation guide:

`minikube/minikube_installation.md`

If Docker is installed and you want to use it as the driver, a typical start command is:

```bash
minikube start --driver=docker
```

Check the cluster:

```bash
minikube status
kubectl get nodes
kubectl cluster-info
```

### 4.2 Deploy the sample application

Open the sample application directory:

```bash
cd minikube/deploy-sample-application
```

The files are designed to be used in this order:

1. Create the Namespace, then deploy the application.
2. Expose the Deployment with a Service.
3. Update the Deployment and observe its rollout.

First, create the Namespace used by the sample resources:

```bash
kubectl create namespace sample-app
```

Apply the Deployment:

```bash
kubectl apply -f 01-create-deployment.yaml
```

Apply the Service:

```bash
kubectl apply -f 02-service-to-expose-deployment.yaml
```

Check the resources:

```bash
kubectl get deployments,pods,services -n sample-app
```

Get a URL for the Service:

```bash
minikube service sample-service -n sample-app --url
```

Apply the rollout/update example:

```bash
kubectl apply -f 03-update-or-rollout-deployment.yaml
kubectl rollout status deployment/sample-deployment -n sample-app
kubectl get pods -n sample-app
```

When you finish the lab, remove its resources:

```bash
kubectl delete namespace sample-app
```

You can also read `minikube/deploy-sample-application/README.md` for the instructions stored with this example.

## 5. Kind Setup and Multi-Node Cluster

Kind (Kubernetes in Docker) runs Kubernetes nodes as Docker containers.

### 5.1 Review the Kind files

| File | Purpose |
|---|---|
| `kind/installer.sh` | Helper script intended to install Docker (if needed), Kind, and kubectl on supported Linux systems. |
| `kind/kind-config.yml` | Defines a cluster with one control-plane node and three worker nodes. |
| `kind/dashboard-admin-user.yml` | Creates a Dashboard ServiceAccount and grants it cluster-admin permissions. |
| `kind/README.md` | Kind-specific setup notes. |

The Kind configuration maps host port **8080** to container port **80**, and host port **8443** to container port **443**, on the configured worker node.

### 5.2 Install the tools

Read the installer before running it:

```bash
less kind/installer.sh
```

If the script matches your operating system and CPU architecture, run it from the repository root:

```bash
chmod +x kind/installer.sh
./kind/installer.sh
```

The script downloads a Linux AMD64 kubectl binary. If your machine uses ARM64 or another architecture, use an architecture-matched installation method instead. If Docker group permissions were changed during installation, log out and back in before using Docker.

### 5.3 Create the Kind cluster

Run this from the repository root:

```bash
kind create cluster --config kind/kind-config.yml --name k8s-lab
```

Verify the cluster and nodes:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-lab
kubectl get nodes
kubectl cluster-info
```

The configuration file is passed to `kind create cluster`; it is **not** applied using `kubectl apply`.

To delete this lab cluster:

```bash
kind delete cluster --name k8s-lab
```

### 5.4 Dashboard admin user

The file `kind/dashboard-admin-user.yml` creates an admin ServiceAccount and binds it to the built-in `cluster-admin` role.

Only use this in a disposable, trusted learning cluster:

```bash
kubectl apply -f kind/dashboard-admin-user.yml
```

Granting `cluster-admin` gives broad administrative access to the cluster. Do not use this configuration as-is for a production cluster or expose its credentials publicly.

## 6. Kubernetes Manifests

The `manifests/` directory contains individual examples of common Kubernetes resources.

| File | Resource | What it demonstrates |
|---|---|---|
| `namespace.yml` | Namespace | Creates the `nginx` namespace used by several examples. |
| `pod.yml` | Pod | Runs a standalone Nginx Pod. |
| `deployment.yml` | Deployment | Runs two Nginx replicas and references the `local-pvc` claim. |
| `replicasets.yml` | ReplicaSet | Maintains two matching Nginx Pod replicas. |
| `service.yml` | Service | Creates a ClusterIP Service selecting Pods labeled `app: nginx`. |
| `daemonsets.yml` | DaemonSet | Runs an Nginx Pod on eligible nodes. |
| `job.yml` | Job | Runs a one-time BusyBox task. |
| `cron-job.yml` | CronJob | Runs a scheduled BusyBox task every minute. |
| `persistentVolume.yml` | PersistentVolume | Defines a 1 Gi hostPath volume using `local-storage`. |
| `persistentVolumeClaim.yml` | PersistentVolumeClaim | Requests 1 Gi of storage from `local-storage`. |
| `ingress.yml` | Ingress | Demonstrates HTTP routing to backend Services. |

### 6.1 Confirm your active cluster

Before applying any manifest, check which cluster your kubectl context points to:

```bash
kubectl config current-context
kubectl get nodes
```

### 6.2 Apply the basic Nginx examples

Run these commands from the repository root.

Create the Namespace first:

```bash
kubectl apply -f manifests/namespace.yml
```

Then you can study and apply the individual examples:

```bash
kubectl apply -f manifests/pod.yml
kubectl apply -f manifests/deployment.yml
kubectl apply -f manifests/replicasets.yml
kubectl apply -f manifests/service.yml
kubectl apply -f manifests/daemonsets.yml
kubectl apply -f manifests/job.yml
kubectl apply -f manifests/cron-job.yml
```

These files are separate learning examples, not one application stack. Several controllers use the label `app: nginx`; applying all of them together can result in multiple controllers creating matching Pods, and the Service may select Pods from more than one controller. For clearer experiments, apply and inspect one controller example at a time.

Inspect the resources:

```bash
kubectl get all -n nginx
kubectl get jobs,cronjobs -n nginx
kubectl describe deployment -n nginx
kubectl get events -n nginx --sort-by=.metadata.creationTimestamp
```

### 6.3 Persistent Volume and Persistent Volume Claim

The storage example uses a 1 Gi hostPath PersistentVolume and a matching PVC.

Important before applying:

- A PersistentVolume is cluster-scoped. If `manifests/persistentVolume.yml` contains `metadata.namespace: nginx`, remove that line before applying the file.
- The hostPath directory must exist on the Kubernetes node and have suitable permissions.
- With Kind, Kubernetes nodes are Docker containers. A path inside a Kind node is not automatically the same as a path on your laptop.
- This hostPath setup is for learning, not a production storage design.

After preparing the PV manifest and creating the `nginx` Namespace, apply the PV first and the PVC second:

```bash
kubectl apply -f manifests/persistentVolume.yml
kubectl apply -f manifests/persistentVolumeClaim.yml
kubectl get pv
kubectl get pvc -n nginx
```

The Deployment example references the claim named `local-pvc`. Make sure the PVC exists and is Bound before applying that Deployment:

```bash
kubectl get pvc local-pvc -n nginx
kubectl apply -f manifests/deployment.yml
```

### 6.4 Ingress

The Ingress example references two backend Services: `nginx-service` and `notes-app-service`.

Before applying it:

1. Install and enable an Ingress controller for your cluster (for example, the Minikube Ingress addon when using Minikube).
2. Ensure that both referenced Services exist in the same Namespace as the Ingress.
3. Create the `notes-app-service` backend yourself if you want to use that route; it is not defined by the other Service manifest in this repository.

Then apply and inspect the Ingress:

```bash
kubectl apply -f manifests/ingress.yml
kubectl get ingress -n nginx
kubectl describe ingress -n nginx
```

## 7. Build and Run the Task Manager Docker Image

The `task-manager/` directory contains a small static HTML Task Manager served by Nginx. Its tasks are stored in the browser's localStorage; this example does not include a backend API or database.

Run the following commands from the repository root. Replace `YOUR_DOCKERHUB_USERNAME` with your Docker Hub username.

Build the image:

```bash
docker build -t YOUR_DOCKERHUB_USERNAME/task-manager:latest ./task-manager
```

Run it locally:

```bash
docker run -d --name task-manager-demo -p 8080:80 YOUR_DOCKERHUB_USERNAME/task-manager:latest
```

Open the application in your browser:

`http://localhost:8080`

Check the container:

```bash
docker ps
docker logs task-manager-demo
```

To publish the image to Docker Hub:

```bash
docker login
docker push YOUR_DOCKERHUB_USERNAME/task-manager:latest
```

To stop and remove the local demo container:

```bash
docker rm -f task-manager-demo
```

Read `task-manager/README.md` for application-specific notes.

## 8. Deploy the Task Manager on a kubeadm Cluster

The `kubeadm/` directory contains a Namespace, Pod, Deployment, and Service example for the Task Manager image.

Read `kubeadm/README.md` first and prepare your kubeadm cluster before applying these resources. The guide currently contains Kubernetes v1.29 package instructions; check the Kubernetes and network-plugin documentation for version compatibility before following older installation commands.

The example manifests use the image:

`bongodev/task-manager:latest`

If you build and push the image under a different Docker Hub account, update the image name in **both** `kubeadm/pod.yaml` and `kubeadm/deployment.yaml` before applying them.

From the repository root, apply the resources in this order:

```bash
kubectl apply -f kubeadm/namespace.yaml
kubectl apply -f kubeadm/deployment.yaml
kubectl apply -f kubeadm/service.yaml
```

You can also apply the standalone Pod example separately:

```bash
kubectl apply -f kubeadm/pod.yaml
```

The Service uses NodePort **30080**. Allow that port through the relevant host/cloud firewall rules if you need to access it from outside the node.

Check the deployment:

```bash
kubectl get all -n kubernetes-cluster
kubectl describe service -n kubernetes-cluster
```

## 9. Useful kubectl Commands

Use these commands while practicing:

```bash
# Cluster information
kubectl cluster-info
kubectl get nodes
kubectl config get-contexts
kubectl config current-context

# View resources
kubectl get namespaces
kubectl get pods -A
kubectl get deployments -A
kubectl get services -A

# Inspect a resource
kubectl describe pod POD_NAME -n NAMESPACE
kubectl logs POD_NAME -n NAMESPACE

# Apply or remove a manifest
kubectl apply -f FILE.yaml
kubectl delete -f FILE.yaml

# Watch resources update
kubectl get pods -n NAMESPACE -w
```

Replace placeholder values such as `POD_NAME`, `NAMESPACE`, and `FILE.yaml` with the actual names and paths in your cluster.

## 10. Troubleshooting Notes

- **kubectl cannot connect:** Check that your cluster is running and that the current context is correct.
- **ImagePullBackOff:** Check the image name, tag, registry access, and whether the image is public or requires authentication.
- **PVC remains Pending:** Check that a matching PersistentVolume or StorageClass exists and that the requested capacity and access mode match.
- **Ingress has no address or does not route:** Confirm that an Ingress controller is installed and that the referenced Services and ports exist.
- **NodePort is unreachable:** Check the Service, node address, and host/cloud firewall rules.
- **Kind port mapping does not work:** Confirm that the cluster was created using the repository's Kind configuration and that the application Service/Ingress is configured correctly.
- **Docker permission denied:** Confirm Docker is running and that your user has the required permissions; a new login session may be needed after group changes.

## 11. Clean Up Your Lab

Delete only the resources and clusters you created for your experiment. For example:

```bash
kubectl delete -f manifests/cron-job.yml
kubectl delete -f manifests/job.yml
kubectl delete -f manifests/daemonsets.yml
kubectl delete -f manifests/service.yml
kubectl delete -f manifests/replicasets.yml
kubectl delete -f manifests/deployment.yml
kubectl delete -f manifests/pod.yml
kubectl delete namespace nginx
```

Delete a Minikube cluster:

```bash
minikube delete
```

Delete the Kind cluster created in this guide:

```bash
kind delete cluster --name k8s-lab
```

Only run cleanup commands when you are sure you are targeting your learning resources. Deleting a Namespace also deletes the namespaced resources inside it.

## 12. Suggested Learning Path

If you are new to Kubernetes, follow this order:

1. Learn the basic cluster concepts and start with Minikube.
2. Practice Namespace, Pod, Deployment, ReplicaSet, and Service.
3. Explore Jobs, CronJobs, and DaemonSets.
4. Learn PersistentVolumes and PersistentVolumeClaims.
5. Deploy the sample application and practice rolling updates.
6. Explore Ingress and service routing.
7. Create a multi-node local cluster with Kind.
8. Move on to kubeadm and multi-machine cluster administration.

## Purpose

This repository is a hands-on reference for learning Kubernetes fundamentals, understanding common Kubernetes resources, setting up different types of clusters, and practicing application deployment.

**Learn by doing, inspect every resource, and experiment safely in a disposable lab cluster.**
