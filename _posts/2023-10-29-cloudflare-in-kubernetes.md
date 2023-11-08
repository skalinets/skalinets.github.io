---
title: Setting up Cloudflare in Kubernetes Cluster
---

In modern cloud architectures, keeping services secure and fast is a top priority. Cloudflare Tunnel and Kubernetes can be combined to offer a robust, scalable, and secure solution. This blog post will guide you through the process of setting up a Cloudflare Tunnel for a Kubernetes cluster.

## Prerequisites

- A running Kubernetes cluster
- `kubectl` installed and configured
- Cloudflare account
- Cloudflared software installed

## Step 1: Install Cloudflared on Your Local Machine

Firstly, download and install the `cloudflared` CLI tool from the Cloudflare website. Below are the installation steps for different operating systems.

### For Linux:

```bash
wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.deb
sudo dpkg -i cloudflared-stable-linux-amd64.deb
```

### For macOS:

You can install `cloudflared` using Homebrew:

```bash
brew install cloudflare/cloudflare/cloudflared
```

Or manually download the package from the Cloudflare website and follow the installation instructions.

### For Windows:

1. Download the `cloudflared` executable for Windows from the Cloudflare website.
2. Place it in a directory (e.g., `C:\cloudflared`).
3. Add this directory to your system PATH variable.
4. Open Command Prompt as an administrator and run:

```bash
cloudflared --version
```

to ensure it's installed correctly.

Now you've installed `cloudflared` for your respective operating system and can proceed with the other steps.

## Step 2: Authenticate Cloudflared

Run the following command and follow the prompts to authenticate:

```bash
cloudflared tunnel login
```

## Step 3: Create a Tunnel

Create a new tunnel with a descriptive name.

```bash
cloudflared tunnel create my-k8s-tunnel
```

## Step 4: Configure Your Kubernetes Cluster

Now it's time to configure your Kubernetes cluster to direct traffic through the Cloudflare Tunnel. Create a Kubernetes ConfigMap for the cloudflared configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cloudflared-config
data:
  config.yml: |
    tunnel: my-k8s-tunnel
    credentials-file: /etc/cloudflared/credentials.json
```

Apply it using `kubectl`:

```bash
kubectl apply -f cloudflared-configmap.yml
```

## Step 5: Deploy Cloudflared to Kubernetes

Create a Kubernetes deployment for Cloudflared.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudflared
spec:
  replicas: 2
  template:
    ...
    volumes:
      - name: cloudflared-config
        configMap:
          name: cloudflared-config
```

Apply the deployment:

```bash
kubectl apply -f cloudflared-deployment.yml
```

## Step 6: Verify

To confirm the tunnel is working, you can describe the Cloudflared pods or check the Cloudflare dashboard.

```bash
kubectl describe pods -l app=cloudflared
```

## Conclusion

You've successfully set up a Cloudflare Tunnel for your Kubernetes cluster! This will help you secure and optimize traffic to your applications.

By following these steps, you can ensure better reliability and security for your Kubernetes-hosted services.
