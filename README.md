# Bird Application Deployment using Helm on Kind Kubernetes Cluster

This guide provides step-by-step instructions to deploy a Python application using Docker, Helm, and Kind.

## Prerequisites

Ensure you have the following tools installed:
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation): To create a local Kubernetes cluster.
- [Helm](https://helm.sh/docs/intro/install/): To manage Kubernetes applications using Helm charts.
- [Docker](https://docs.docker.com/get-docker/): To containerize the Python application.
- [Kubectl](https://kubernetes.io/docs/tasks/tools/): To interact with the Kubernetes cluster.

## Steps to Deploy

### 1. Create a Kind Kubernetes Cluster

Create a Kubernetes cluster using Kind:

```bash
kind create cluster --name my-cluster
```

### 2. Create a Docker Image for the Python Application

Create a `Dockerfile` for your Python application:

```Dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Build the Docker image:

```bash
docker build -t birdapp:latest .
```

Since Kind runs in Docker, load the Docker image directly into the Kind cluster:

```bash
kind load docker-image birdapp:latest --name my-cluster
```

### 3. Create a Helm Chart for the Application

Navigate to your Helm chart directory (assuming you have a Helm chart named `birds`):

```bash
cd helm/birds
```

Update the `values.yaml` file with the Docker image information:

```yaml
# values.yaml
replicaCount: 1

image:
  repository: birdapp  # Name of the Docker image
  pullPolicy: IfNotPresent
  tag: "latest"  # Tag of the Docker image

service:
  type: ClusterIP
  port: 80

resources: {}

nodeSelector: {}

tolerations: []

affinity: {}
```

### 4. Install the Helm Chart

Install the Helm chart in your Kubernetes cluster:

```bash
helm install birds .
```

### 5. Verify the Deployment

Check the status of your deployment and services:

```bash
kubectl get deployments
kubectl get services
```

### 6. List Helm Releases

List the Helm releases to ensure your application is deployed:

```bash
helm list
```

Expected output:

```bash
NAME    NAMESPACE    REVISION    UPDATED                                 STATUS      CHART           APP VERSION
birds   default      1           2023-06-02 12:34:56.123456 -0700 PDT    deployed    birds-0.1.0     1.0
```

## Additional Customization

1. **Custom Resource Definitions (CRDs)**: Include CRDs in your Helm chart if required by your application.
2. **Configuration Management**: Use ConfigMaps and Secrets to manage application configuration securely.
3. **Helm Hooks**: Implement Helm hooks for pre-install, post-install, pre-upgrade, and post-upgrade actions if necessary.
4. **CI/CD Integration**: Integrate Helm chart deployment into your CI/CD pipeline for automated deployments.

## Example Helm Chart Structure

Ensure your Helm chart directory structure looks like this:

```
birds/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
```

### Chart.yaml Example

```yaml
apiVersion: v2
name: birds
description: A Helm chart for the Bird application
version: 0.1.0
appVersion: "1.0"
```

### Deployment.yaml Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "birds.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "birds.name" . }}
      release: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ include "birds.name" . }}
        release: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 80
```

### Service.yaml Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "birds.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
  selector:
    app: {{ include "birds.name" . }}
    release: {{ .Release.Name }}
```

By following these steps, you should have a well-structured Helm chart and a successfully deployed Python application in your Kind Kubernetes cluster.
