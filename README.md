# Accoknox-Task
This is just a practice repository

DVWA Deployment on Minikube Kubernetes Cluster: Setup, Troubleshooting, and Attack Demonstration
This report details the process of deploying the Damn Vulnerable Web Application (DVWA) on a local Kubernetes cluster using Minikube and Docker Desktop on a Windows 11 Home system. It outlines the initial setup, comprehensive troubleshooting steps for a persistent network connectivity issue, the eventual workaround employed, and the successful demonstration of three distinct attack surfaces.

1. Project Objective
The primary objective was to deploy a local Kubernetes (k8s) cluster using Minikube and then deploy the DVWA application onto this cluster. Following successful deployment and accessibility, the assignment required demonstrating and showcasing three specific attack surfaces as outlined in DVWA's documentation.

2. Prerequisites and Initial Setup
The chosen environment for this deployment was:

Operating System: Microsoft Windows 11 Home Single Language (Version 10.0.26100.4349)

Kubernetes Cluster Tool: Minikube (v1.36.0)

Container Runtime: Docker Desktop (utilizing the WSL2 backend)


2.1 Docker Desktop Installation & WSL2 Verification
1.Docker Desktop Installation: The latest stable version of Docker Desktop was downloaded from the official website and installed. For Windows 11 Home, Docker Desktop automatically defaults to and requires the WSL2 (Windows Subsystem for Linux 2) backend for its operation, as Hyper-V is not available in Home editions. No explicit WSL2 option was presented during installation, confirming the default behavior.

2.WSL2 Verification: To confirm the presence and version of WSL, the following command was executed in PowerShell:


----------------------------->    wsl --list --verbose

Output:

  NAME          STATE           VERSION
* Ubuntu        Stopped         2


Confirmation: This output confirmed that WSL was installed and the existing Ubuntu distribution was indeed running on WSL 2, which is essential for Docker Desktop's functionality on Windows Home.


2.2 Minikube Cluster Setup
1.Minikube Executable: The minikube.exe executable (v1.36.0) was ensured to be present in the system's PATH.

2.Initial Cluster Start: The Minikube cluster was initiated using the Docker driver:


----------------------------------->    minikube start --driver=docker


6.0 on Microsoft Windows 11 Home Single Language 10.0.26100.4349 Build 26100.4349
* Using the docker driver based on user configuration
* Using Docker Desktop driver with root privileges
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.47 ...
* Creating docker container (CPUs=2, Memory=2200MB) ...
! Failing to connect to https://registry.k8s.io/ from inside the minikube container
* To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
* Preparing Kubernetes v1.33.1 on Docker 28.1.1 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default


Observation: Despite the warning about failing to connect to registry.k8s.io, Minikube reported * Done!, indicating it believed the cluster was ready.

Minikube IP: minikube ip consistently returned 192.168.49.2.




## 2.3 DVWA Application Deployment on Kubernetes
The DVWA application requires a MySQL database and the PHP-based web application. Kubernetes YAML manifests were prepared and applied to deploy these components:

dvwa-namespace.yaml: Defines a dedicated Kubernetes namespace (dvwa) for better isolation.
