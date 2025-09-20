# K8s GitOps Project with FluxCD

This project implements a complete GitOps workflow using k3s, FluxCD, WordPress, MySQL, and monitoring stack.

## Components
- k3s (Kubernetes)
- FluxCD (GitOps)
- MetalLB (Load Balancer)
- Ingress-NGINX with custom 404 redirect
- WordPress + MySQL (with SOPS encrypted secrets)
- Prometheus + Loki + Grafana (Monitoring)

## Setup Instructions

### 1. Install k3s
```bash
# Install k3s without traefik (we'll use nginx-ingress)
curl -sfL https://get.k3s.io | sh -s - --disable traefik

# Make kubectl accessible
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc

# Verify installation
kubectl get nodes



1.2 Install Flux CLI

# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Verify installation
flux --version




1.3 Install SOPS and age

# Install SOPS
curl -LO https://github.com/getsops/sops/releases/download/v3.8.1/sops-v3.8.1.linux.amd64
sudo mv sops-v3.8.1.linux.amd64 /usr/local/bin/sops
sudo chmod +x /usr/local/bin/sops

# Install age
sudo apt-get update && sudo apt-get install -y age
# OR download directly
curl -LO https://github.com/FiloSottile/age/releases/download/v1.1.1/age-v1.1.1-linux-amd64.tar.gz
tar -xf age-v1.1.1-linux-amd64.tar.gz
sudo mv age/age /usr/local/bin/
sudo mv age/age-keygen /usr/local/bin/



Step 2: Setup GitHub Repository

2.1 Create GitHub Repository

# Create a new repository on GitHub named 'k8s-gitops-flux'
# Clone it locally
git clone https://github.com/YOUR_USERNAME/k8s-gitops-flux.git
cd k8s-gitops-flux


2.2 Create Repository Structure

mkdir -p clusters/local
mkdir -p apps/{base,local}
mkdir -p infrastructure/{controllers,configs}
mkdir -p secrets



Step 3: Setup SOPS with age

3.1 Generate age Keys

# Generate age key pair
age-keygen -o age.key

# Store the public key
age-keygen -y age.key > age.pub

# Save the key securely (you'll need this for deliverables)
cp age.key ~/age-private.key
cp age.pub ~/age-public.key



3.2 Create SOPS Configuration
Create .sops.yaml in repository root:

creation_rules:
  - path_regex: .*\.yaml
    encrypted_regex: ^(data|stringData)$
    age: >-
      YOUR_AGE_PUBLIC_KEY_HERE




Step 4: Bootstrap FluxCD
4.1 Set Environment Variables

export GITHUB_TOKEN=your_github_token_here
export GITHUB_USER=your_github_username
export GITHUB_REPO=k8s-gitops-flux



4.2 Bootstrap Flux

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=./clusters/local \
  --personal
