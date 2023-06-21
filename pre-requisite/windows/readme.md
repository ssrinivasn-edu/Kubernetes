

To install container environment on Windows 10/11, use these steps:

### Step1: Install Windows Subsystem for Linux (WSL)
- Prerequisites
  You must be running Windows 10 version 2004 and higher (Build 19041 and higher) or Windows 11 to use the commands below.
- Open PowerShell or Windows Command Prompt as an administrator
- Run command
```
  wsl –install
```
- Restart machine

## Step 2: Install Docker Desktop

- Download Docker Desktop for Windows from the official Docker website: https://www.docker.com/products/docker-desktop
- Install Docker Desktop by running the downloaded installer as an Administrator.
- Go To Start and search Docker Desktop. Click Docker Desktop app icon to start the Docker on the machine 

## Step 3: Install kubectl
kubectl is a command-line tool used to interact with Kubernetes clusters. You need to install kubectl separately.

- Download the kubectl binary for Windows from the official Kubernetes website: https://kubernetes.io/docs/tasks/tools/
- Extract the downloaded archive and move the `kubectl` binary to a directory in your system's PATH - c:\windows\system32. Move file using File Explorer as the Administrative previlidage required for it.
