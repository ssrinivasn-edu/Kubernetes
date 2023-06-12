
To install Minikube on Windows 11, you can follow these steps:


## Step 1: Install Docker Desktop

- Download Docker Desktop for Windows from the official Docker website: https://www.docker.com/products/docker-desktop
- Install Docker Desktop by running the downloaded installer as an Administrator.
- Go To Start and serach Docker Desktop. Click Docker Desktop app icon to start the Docker on the machine 


## Step 2: Install kubectl
kubectl is a command-line tool used to interact with Kubernetes clusters. You need to install kubectl separately.

1. Download the kubectl binary for Windows from the official Kubernetes website: https://kubernetes.io/docs/tasks/tools/
2. Extract the downloaded archive and move the `kubectl` binary to a directory in your system's PATH - c:\windows\system32. Move file using File Explorer as the Administrative previlidage required for it.

Step 3: Install Minikube
1. Open PowerShell as an administrator.
2. Download the Minikube Windows installer by running the following command:
```
Invoke-WebRequest -Uri https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe -OutFile minikube-installer.exe
```
4. Re-open PowerShell as an administrator.
3. Install Minikube by running the following command:
```
.\minikube-installer.exe
```
4. Minikube will be installed in the default location (C:\Program Files\Minikube) by default. If you want to install it in a different location, use the `--install-dir` flag followed by the desired directory path.

Step 4: Start Minikube
1. Re-open PowerShell as an administrator.
2. Start Minikube by running the following command:
```
minikube start
```
3. Minikube will download the necessary Kubernetes components and create a virtual machine (VM) to host your cluster. This process may take a few minutes.
4. Once Minikube is started, you can use `kubectl` commands to interact with your Minikube cluster.

## Verify Minikube installation

 - Run command - kubectl get pod --all-namespaces
 - You will see out similar to following
 
 NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-787d4945fb-r7hvt           1/1     Running   0             81s
kube-system   etcd-minikube                      1/1     Running   0             94s
kube-system   kube-apiserver-minikube            1/1     Running   0             95s
kube-system   kube-controller-manager-minikube   1/1     Running   0             94s
kube-system   kube-proxy-dbm8w                   1/1     Running   0             82s
kube-system   kube-scheduler-minikube            1/1     Running   0             94s
kube-system   storage-provisioner                1/1     Running   1 (69s ago)   85s
