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



## dvwa-namespace.yaml: Defines a dedicated Kubernetes namespace (dvwa) for better isolation.

'''
apiVersion: v1

kind: Namespace

metadata:

  name: dvwa 
              '''



#### dvwa-secret.yaml: Stores sensitive database credentials securely as a Kubernetes Secret.

#### dvwa-mysql-deployment.yaml: Defines the MySQL database deployment (a single pod running MySQL).

#### dvwa-mysql-service.yaml: Exposes the MySQL deployment internally within the cluster as a ClusterIP service.

#### dvwa-web-deployment.yaml: Defines the DVWA web application deployment.

#### dvwa-web-service.yaml: Exposes the DVWA web application outside the cluster using a NodePort.


### Deployment Commands and Verification:

The YAML files were applied in sequence:

.kubectl apply -f dvwa-namespace.yaml

.kubectl apply -f dvwa-secret.yaml

.kubectl apply -f dvwa-mysql-deployment.yaml

.kubectl apply -f dvwa-mysql-service.yaml

.kubectl apply -f dvwa-web-deployment.yaml

.kubectl apply -f dvwa-web-service.yaml


### ----------------->Outcome: All kubectl apply commands reported successful creation/configuration.

### Pods and Services were verified:

#### --------------------->kubectl get pods -n dvwa -l app=dvwa-mysql
#### --------------------->kubectl get pods -n dvwa -l app=dvwa-web
#### --------------------->kubectl get services -n dvwa -l app=dvwa-web


#### ----->Outcome: Both dvwa-mysql and dvwa-web pods showed 1/1 Running. The dvwa-web-service was of NodePort type, exposing the application on a high port (e.g., 39080).

3. The Persistent Network Connectivity Problem
Despite the Kubernetes deployment appearing successful, the DVWA application was initially inaccessible from the Windows host's web browser.

3.1 Initial Inaccessibility and Ping Failure
Access Attempt: Navigating to http://192.168.49.2:39080 (Minikube's IP and the DVWA NodePort) in a browser resulted in "This site cannot be reached. Took too long to respond."

#### Ping Test: A direct ping to the Minikube IP from the Windows Command Prompt confirmed the issue:

#### ping 192.168.49.2

### Output:

------> Pinging 192.168.49.2 with 32 bytes of data:
---->Request timed out.
---->Request timed out.
---->Request timed out.
---->Request timed out.

Ping statistics for 192.168.49.2:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

Diagnosis: This 100% loss indicated a fundamental network communication breakdown between the Windows host and the Minikube VM.



3.2 Diagnosis: The Subnet Mismatch
Further investigation using ipconfig /all revealed the root cause: a persistent subnet mismatch between Minikube's assigned IP and the Windows host's virtual adapter for WSL.

Minikube's Reported IP: Consistently 192.168.49.2. This implies Minikube's internal Docker network was operating in the 1992.168.49.0/24 range.

vEthernet (WSL) Adapter's IP: Consistently 192.168.16.1 with a 255.255.240.0 subnet mask (a /20 subnet).

Problem Summary: The Windows host's vEthernet (WSL) adapter, which acts as the bridge to the WSL2 environment where Docker and Minikube run, was in the 192.168.16.x subnet. Minikube's IP was in the 192.168.49.x subnet. These are distinct subnets, preventing direct IP communication between the host and the Minikube VM. The warning ! Failing to connect to https://registry.k8s.io/ from inside the minikube container was a direct symptom of Minikube's inability to route out to the internet through this misaligned bridge.



Re-downloaded and reinstalled the latest Docker Desktop, ensuring WSL2 integration was enabled (which is default for Windows 11 Home).

Launched Docker Desktop and waited for it to be fully running.

minikube delete followed by minikube start --driver=docker.

Outcome: Despite this comprehensive cleanup and reinstall, minikube ip was still 192.168.49.2, vEthernet (WSL) remained 192.168.16.1, and the ping continued to result in 100% loss.

daemon.json Configuration via Docker Desktop GUI:

Action: Attempted to explicitly force Docker's bridge IP by adding  "bip": "192.168.16.1/20" and "fixed-cidr-v6": "fd00::/80" to the daemon.json via Docker Desktop's Settings -> Docker Engine GUI, followed by "Apply & Restart".

Observation: The daemon.json content in the GUI correctly reflected the added lines.

Outcome: However, after this, minikube ip still returned 192.168.49.2, vEthernet (WSL) stayed at 192.168.16.1, and ping 192.168.49.2 continued to show 100% loss. This indicated that the bip setting was either not picked up or was overridden by the underlying WSL2 network configuration.

Conclusion of Troubleshooting: The persistent subnet mismatch, despite all efforts, pointed to an extremely stubborn, low-level network configuration issue specific to this Windows 11 system's interaction with WSL2 virtualization, beyond typical software fixes.


5. The Breakthrough: Accessing DVWA via Port Forwarding
Although the direct network communication between the host and Minikube's default IP remained unresolved, a standard Kubernetes feature provided a highly effective workaround: Port Forwarding.

### Command Used:

### -----> kubectl port-forward svc/dvwa-web 8080:80 -n dvwa


Explanation: This command creates a secure, temporary tunnel from a specific port on the local host machine (e.g., localhost:8080) directly to a specified port on a service within the Kubernetes cluster (dvwa-web service's port 80). This effectively bypasses the problematic network routing between the host's vEthernet (WSL) adapter and Minikube's internal IP.

Outcome: Executing this command successfully made the DVWA application accessible in the web browser via http://localhost:8080. This allowed for the successful completion of the assignment's demonstration requirements.

6. Demonstration of Attack Vectors
With the DVWA application successfully accessible, three common web vulnerabilities were demonstrated.

6.1 Attack 1: SQL Injection
Vulnerability: Allows an attacker to interfere with the queries that an application makes to its database.

#### Demonstration:

Navigated to the "SQL Injection" section in DVWA.

In the "User ID" input field, the following payload was entered:
### 1=1 --

Expected Result: The application, instead of querying for a specific user ID, interprets 1=1 as always true and  -- comments out the rest of the query, effectively returning all data (or the first entry, depending on the backend logic), including potentially sensitive information like other user details.



6.2 Attack 2: Command Injection
Vulnerability: Allows an attacker to execute arbitrary commands on the host operating system of the server.

Demonstration:

Navigated to the "Command Injection" section in DVWA.

### In the "Enter an IP address" input field, a command was appended to a valid ping command.
### 127.0.0.1 & whoami (on a Linux-based server, whoami reveals the current user)

Expected Result: The server executes both the ping and the whoami command, displaying the output of both.



6.3 Attack 3: Reflected Cross-Site Scripting (XSS)

Vulnerability: Occurs when a web application receives data in an HTTP request and includes that data within the immediate response in an unsafe way, allowing malicious scripts to be executed in the user's browser.

Demonstration:

Navigated to the "XSS (Reflected)" section in DVWA.

### In the "What's your name?" input field, the following script was entered:
### <script>alert('XSS')</script>

Expected Result: The application reflects the script back into the page without proper sanitization, causing the browser to execute the JavaScript alert function.



7. Conclusion
The assignment to deploy a local Kubernetes cluster with DVWA and demonstrate attack vectors was successfully completed. While the initial setup encountered a persistent and complex network routing issue between the Windows host and the Minikube VM (characterized by a subnet mismatch and confirmed by consistent ping failures and registry.k8s.io connection warnings), a highly effective workaround using kubectl port-forward allowed full access to the deployed DVWA application. This demonstrates the practical problem-solving skills required in real-world Kubernetes deployments and validated the functionality of the DVWA application for security testing purposes.









