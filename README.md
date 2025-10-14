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

your-app/
├── Jenkinsfile              # CI pipeline
├── Dockerfile               # Container build
├── requirements.txt         # Python dependencies
├── k8s/                     # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── ingress.yaml
└── argocd/                  # GitOps (optional)
    └── application.yaml
