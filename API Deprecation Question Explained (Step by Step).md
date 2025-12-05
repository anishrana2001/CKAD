# 1. Introduction
### In this video, you will learn how API deprecation works in Kubernetes and how CKAD tests this concept with practical tasks.​
### You will also see how to install and use kubectl convert to migrate old manifests to supported apiVersions using a realistic exam-style example.​

# 2. What is API Deprecation in Kubernetes?
### Kubernetes evolves quickly, so older API versions (like extensions/v1beta1 or apps/v1beta1) are marked as deprecated and later removed.​

### When a deprecated API is finally removed in newer cluster versions, manifests using that API will fail to apply, which can break workloads and deployments.​

# 3. How CKAD Tests API Deprecation
### In the CKAD exam, you may receive a YAML manifest using a deprecated apiVersion and be asked to migrate it to a supported version and create the resource successfully.​
### Typical examples include Deployments, Ingress, or CronJobs that use beta or legacy APIs that must be converted to their GA counterparts like apps/v1 or batch/v1.​

###  Common pattern in questions:

### “A manifest is stored in /opt/CKAD/deprecated/deploy.yaml using a deprecated API.
### Update it to use a supported API version and create the Deployment in the exam namespace.”

# 4. Installing kubectl and kubectl convert
### 4.1 Install kubectl
### Follow the official Kubernetes instructions to install kubectl for your OS.​

### Example for Linux (binary download):

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```
### The Kubernetes documentation provides equivalent commands for macOS and Windows, including package managers like brew, chocolatey, or winget.​

## 4.2 Install kubectl convert Plugin
### From Kubernetes 1.20+, kubectl convert is distributed as a separate plugin under the krew plugin ecosystem or as a standalone binary.​

### Option 1: Install via krew
### Install krew plugin manager (one-time setup) using the official script.​

### Then install the convert plugin:

```
kubectl krew install convert
```
### Verify:

```
kubectl convert --help
```
## Option 2: Download from Kubernetes Releases
### Download the kubectl-convert binary matching your OS and architecture from the official Kubernetes release artifacts and place it in your PATH.​

### Rename it to kubectl-convert (if needed) and ensure it is executable.​

## 5. CKAD-Style Example Scenario
### 5.1 Problem Statement (Exam-Style)
### “An existing application uses a Deployment manifest stored at /opt/CKAD/deprecated/deploy-nginx.yaml that relies on a deprecated API version.
### Update the manifest to use a supported API version and successfully create the Deployment in the ckad namespace.”

### This mirrors real exam scenarios where only the apiVersion and related fields need updates, not a full redesign of the resource.​

## 6. Step-by-Step Demo Using kubectl convert
### 6.1 Inspect the Original Manifest
```
cd /opt/CKAD/deprecated
cat deploy-nginx.yaml
```
### Explain on screen:

### Show an apiVersion like apps/v1beta1 or extensions/v1beta1.​

### Mention that this version is deprecated and removed in modern Kubernetes releases.​

## 6.2 Run kubectl convert
### Use the plugin to convert to a supported version, for example apps/v1:

```
kubectl convert -f deploy-nginx.yaml --output-version=apps/v1 -o yaml > deploy-nginx-v1.yaml
```
- -f deploy-nginx.yaml: Input file.
- --output-version=apps/v1: Target API version.
- -o yaml > deploy-nginx-v1.yaml: Save the converted YAML to a new file.

### The convert tool updates fields that have changed between versions, such as spec.selector or pod template labels for Deployments.​

## 6.3 Review the Converted Manifest
```
cat deploy-nginx-v1.yaml
```
### Explain what to check:

### apiVersion should now be apps/v1.

### Ensure spec.selector.matchLabels matches template.metadata.labels.​

### Emphasize that in CKAD, verifying labels and selectors is important to avoid subtle mistakes that make the Deployment not manage the Pods correctly.​

## 6.4 Create Namespace (If Required)
### If the namespace does not exist:

```
kubectl create namespace ckad
```
## 6.5 Apply the Converted Manifest
```
kubectl apply -f deploy-nginx-v1.yaml -n ckad
```
### Verify:

```
kubectl get deploy -n ckad
kubectl get pods -n ckad -o wide
```
### If everything is correct, the Deployment should be created and running using the new API version.​

## 7. Quick Commands Cheat Sheet (For Viewers)
### You can show this on screen or in the video description:

bash
# Inspect file
```
cat /path/to/deprecated.yaml
```
# Convert to supported API version
```
kubectl convert -f /path/to/deprecated.yaml --output-version=apps/v1 -o yaml > /path/to/new.yaml
```
# Create namespace (if needed)
```
kubectl create namespace ckad
```
# Apply converted manifest
```
kubectl apply -f /path/to/new.yaml -n ckad
```
# Verify resources
```
kubectl get all -n ckad
```

### These steps are directly useful for both CKAD practice and real cluster migrations.​

## 8. Tips for CKAD Exam
### Read the question carefully: Pay attention to file paths, namespaces, and resource names.​
### Prefer editing the existing file instead of creating a new one from scratch unless instructed.​
### If kubectl convert is not available in the exam environment, use the Kubernetes docs deprecated API migration tables to manually update apiVersion and required fields.​

