# Kubernetes-Installation
Steps to install kubernetes on Linux  || Aws Ec2 


## Step 1: Set the hostname

```bash
sudo hostnamectl set-hostname Name__00
```

Sets the hostname of the machine to `Name__00`.

## Step 2: Disable swap

```bash
sudo swapoff -a
```

Disables swap memory, which is required for Kubernetes to function correctly.

## Step 3: Load kernel modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Creates a config file to ensure the necessary kernel modules are loaded.

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Loads the `overlay` and `br_netfilter` modules immediately.

## Step 4: Configure sysctl settings

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Configures system settings required for Kubernetes networking.

```bash
sudo sysctl --system
```

Applies the new settings.

## Step 5: Verify kernel modules

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

Confirms the necessary kernel modules are loaded.

## Step 6: Update package lists

```bash
sudo apt-get update
```

Updates the list of available packages.

## Step 7: Install dependencies

```bash
sudo apt-get install -y ca-certificates curl
```

Installs essential packages for downloading and verifying software.

## Step 8: Add Docker’s official GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Creates a directory for storing trusted keys.

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

Downloads Docker's GPG key.

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Sets the correct permissions for the GPG key.

## Step 9: Add Docker's repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Adds Docker's official repository to the package sources.

## Step 10: Install container runtime

```bash
sudo apt-get update
sudo apt-get install -y containerd.io
```

Installs the container runtime (Containerd) required for Kubernetes.

## Step 11: Configure containerd

```bash
containerd config default | sed -e 's/SystemdCgroup = false/SystemdCgroup = true/' -e 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml
```

Configures containerd to use systemd as the cgroup driver and updates the sandbox image.

```bash
sudo systemctl restart containerd
sudo systemctl status containerd
```

Restarts and verifies the status of the containerd service.

## Step 12: Install Kubernetes packages

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Installs additional dependencies for adding Kubernetes repositories.

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Downloads and adds Kubernetes' GPG key.

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Adds Kubernetes' official repository.

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

Installs Kubernetes components: kubelet, kubeadm, and kubectl.

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Prevents automatic updates to Kubernetes components.

## Step 13: Initialize Kubernetes cluster

```bash
sudo kubeadm init
```

Initializes the Kubernetes cluster.

**If a cluster already exists:**

```bash
kubeadm token create --print-join-command
```

Generates a join command for worker nodes.

## Step 14: Configure kubectl

```bash
mkdir -p "$HOME"/.kube
```

Creates a kube config directory.

```bash
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
```

Copies the Kubernetes admin configuration.

```bash
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
```

Sets correct permissions for the kube config file.

## Step 15: Deploy a network add-on

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

Deploys Calico for pod networking.

## Step 16: Join worker nodes

```bash
kubeadm join 172.......39:6443 --token 3q2p................hb6 \
    --discovery-token-ca-cert-hash sha256:b6710716............................5575aa9038012af831
```

Adds worker nodes to the Kubernetes cluster (replace IP, token, and hash with your values).

## Step 17: Verify cluster nodes

```bash
kubectl get nodes
```

Lists all nodes in the Kubernetes cluster.

---

This guide provides step-by-step instructions to set up Kubernetes on your machine. Copy the commands directly to your terminal for a smooth setup.

---
