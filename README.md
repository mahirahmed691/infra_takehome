# Steps taken to create helm chart using the template provided

## Prerequisites
### Kind: Installed Kind to create a local Kubernetes cluster.
### Helm: Installed Helm on your local machine.
### Docker: Python application will be containerized using Docker.
### Kubectl: Installed kubectl to interact with the Kubernetes cluster.

# Create Kind Kubernetes cluster

```kind create cluster --name my-cluster```

# Created a Docker Image for the Python Application

```
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

## Build the Docker image:


```
docker build -t birdapp:latest .
```

## Since Kind runs in Docker, Docker image directly into the Kind cluster:

``` 
kind load docker-image birdapp:latest --name my-cluster
```


# install helm chart

```cd helm/birds ```
```helm install birds . ```

# Verify Deployment
``` 
kubectl get deployments
kubectl get services
```


# Additional Information

``` 
# Default values for birds.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: birdApp # Changed this to Docker image name   <--- name was changed to use docker image
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "latest" # Changed this to the appropriate tag <--- tag was added to use latest image

```

```
helm list
```

`
NAME    NAMESPACE    REVISION    UPDATED                                 STATUS      CHART           APP VERSION
birds   default      1           2023-06-02 12:34:56.123456 -0700 PDT    deployed    birds-0.1.0     1.0
`
