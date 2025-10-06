# CICD

# How Kaniko works
Reads your Dockerfile
Builds layers unprivileged (no Docker daemon needed)
Pushes directly to registry using credentials from Kubernetes secret
Uses layer caching to speed up rebuilds

The tradeoff: You need to create a Kubernetes secret with Docker credentials:

# Option 1: Create from Docker Hub token (recommended)
kubectl create secret docker-registry docker-config \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=DOCKER_USERNAME \
  --docker-password=YOUR_DOCKER_HUB_TOKEN \
  --namespace=jenkins

# Option 2: Create from existing Docker config
kubectl create secret generic docker-config \
  --from-file=config.json=$HOME/.docker/config.json \
  --namespace=jenkins