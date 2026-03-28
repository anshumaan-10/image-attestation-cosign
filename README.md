# Keyless Image Signing with Cosign and Validation with Kyverno

This README provides an overview and instructions for setting up **keyless image signing** using **Cosign** and enforcing **image validation** with **Kyverno**. By leveraging Cosign for signing and Kyverno for enforcing policies, you can enhance container security and ensure that only properly signed and validated images are deployed in your Kubernetes clusters.

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setting Up Cosign for Keyless Image Signing](#setting-up-cosign-for-keyless-image-signing)
- [Setting Up Kyverno for Image Validation](#setting-up-kyverno-for-image-validation)
- [Enforcing Image Signing Policies](#enforcing-image-signing-policies)
- [Conclusion](#conclusion)

## Introduction

Container security is crucial in modern cloud-native environments. **Cosign** is a tool from the **Sigstore** project that allows you to sign container images with **keyless signing**, meaning you don't need to manage private keys. Instead, it uses public-key infrastructure and integrates with trusted certificate authorities.

**Kyverno** is a Kubernetes-native policy engine that can enforce various security policies. By using Kyverno, you can ensure that only signed container images are deployed to your Kubernetes cluster.

This guide walks you through how to integrate these two tools to strengthen your container security practices by ensuring that only signed images are deployed and are validated by policies in your Kubernetes environment.

## Prerequisites

Before proceeding, ensure you have the following:
- Kubernetes cluster (can be a local cluster using Minikube or a cloud-based one)
- kubectl configured to interact with your cluster
- Helm (for deploying Kyverno)
- Docker or another container runtime installed
- Cosign installed on your local machine (you can install it via the instructions provided on [Cosign GitHub](https://github.com/sigstore/cosign))

## Setting Up Cosign for Keyless Image Signing

### Step 1: Install Cosign

First, you need to install **Cosign** on your machine. Cosign is a command-line tool used to sign and verify container images.

```bash
# For macOS (with Homebrew)
brew install sigstore/tap/cosign

# For Linux
curl -LO https://github.com/sigstore/cosign/releases/download/v1.12.0/cosign-linux-amd64
chmod +x cosign-linux-amd64
mv cosign-linux-amd64 /usr/local/bin/cosign
```

### Step 2: Sign an Image with Cosign (Keyless)

Cosign supports **keyless image signing** using a public-key infrastructure and Google Cloud's Certificate Transparency log. To sign an image without needing to manage a private key, follow these steps:

```bash
# Log in to the Sigstore's Certificate Authority
cosign login

# Sign an image (replace <image-name> with your container image)
cosign sign <image-name>
```

This will create a signature for the image in the Sigstore database, which can be verified by others.

### Step 3: Verify the Image Signature

After signing an image, you can verify it using Cosign:

```bash
# Verify the signed image
cosign verify <image-name>
```

If the signature is valid, the verification will pass.

## Setting Up Kyverno for Image Validation

Kyverno is a Kubernetes-native policy engine that allows you to enforce policies on your Kubernetes resources. We will use Kyverno to ensure that only signed images are deployed into your cluster.

### Step 1: Install Kyverno

You can install Kyverno using Helm. Add the Kyverno Helm repository and install the Kyverno engine:

```bash
# Add the Kyverno Helm repository
helm repo add kyverno https://kyverno.github.io/kyverno/

# Install Kyverno in your Kubernetes cluster
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```

### Step 2: Create a Kyverno Policy to Validate Signed Images

Now, create a policy that ensures only signed images are deployed. Here’s an example policy that can be applied:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-signed-images
spec:
  validationFailureAction: enforce
  rules:
    - name: check-image-signature
      match:
        resources:
          kinds:
            - Pod
      validate:
        message: "The container image must be signed"
        pattern:
          spec:
            containers:
              - image: "?(*/*):*"
      actions:
        - name: cosign-signature-check
          type: cosign
          value: "true"
```

Apply this policy to your cluster:

```bash
kubectl apply -f require-signed-images-policy.yaml
```

### Step 3: Verify Kyverno Policy Enforcement

After applying the policy, deploy a pod with an unsigned image and observe the enforcement:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: unsigned-image-pod
spec:
  containers:
    - name: my-container
      image: <your-unsigned-image>
```

Kyverno will reject the deployment of this pod because the image is not signed. You will see an error message indicating the image validation failure.

## Enforcing Image Signing Policies

By combining Cosign and Kyverno, you enforce that only signed images are allowed to run in your cluster. This strengthens security by ensuring that only trusted container images are deployed, mitigating the risks of using unsigned or potentially compromised images.

### Best Practices:
- Regularly rotate and audit image signing certificates and keys.
- Set Kyverno to block the deployment of any pod with unsigned images using the `validationFailureAction: enforce` setting.
- Use automated pipelines to sign images before they are pushed to the container registry.

## Conclusion

By leveraging **Cosign** for keyless image signing and **Kyverno** for image validation, you can significantly improve the security posture of your Kubernetes environment. This integration ensures that only signed images are allowed to run, protecting against the deployment of unauthorized or compromised container images.

For more details, explore the following resources:
- [Cosign GitHub Repository](https://github.com/sigstore/cosign)
- [Kyverno GitHub Repository](https://github.com/kyverno/kyverno)

This approach is part of a broader **DevSecOps** strategy that emphasizes security throughout the container lifecycle, from image creation to deployment in the cluster.

--- 

This README outlines a secure and automated flow to ensure that your Kubernetes deployments are both secure and compliant with container image signing best practices.

## Overview

This repository contains project code and supporting assets. It is maintained actively with periodic updates.

## Getting Started

1. Clone this repository.
2. Install dependencies as documented in the project files.
3. Run/build using the project-specific commands.

## Repository Structure

Key source code, configuration, and documentation are organized by folders at the repository root.

## Contribution Guidelines

Please open an issue for major changes and submit focused pull requests with clear descriptions.
