# Voting App

![Result App](https://github.com/sam-atkins/voting-app/workflows/PublishResultAppDockerImage/badge.svg)
![Vote App](https://github.com/sam-atkins/voting-app/workflows/PublishVoteAppDockerImage/badge.svg)
![Worker](https://github.com/sam-atkins/voting-app/workflows/PublishWorkerDockerImage/badge.svg)

- [Voting App](#voting-app)
  - [Context](#context)
  - [Local Development](#local-development)
    - [Prerequisites](#prerequisites)
    - [Create secret to pull image from Github](#create-secret-to-pull-image-from-github)
    - [Compose](#compose)
    - [K8s](#k8s)

## Context

This is a small project to help me learn some more about Kubernetes. It builds on the learning from my [helloworldapi](https://github.com/sam-atkins/helloworldapi) project. This includes multiple containers and as a result is a bit more complex.

The source code for the components `result`, `vote` and `worker` are taken (with some small modifications) from the Docker Samples repo which you can find [here](https://github.com/dockersamples/example-voting-app).

## Local Development

### Prerequisites

- [minikube](https://minikube.sigs.k8s.io/docs/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Skaffold](https://skaffold.dev)

### Create secret to pull image from Github

1. Create new Github Personal Access Token with read:packages scope at https://github.com/settings/tokens/new
2. Base-64 encode <your-github-username>:<TOKEN>

    ```bash
    $ echo -n <github-username>:<TOKEN> | base64
    <AUTH>
    ```

3. Manually create the secret:

    ```bash
    $ echo '{"auths":{"docker.pkg.github.com":{"auth":"<AUTH>"}}}' | kubectl create secret generic dockerconfigjson-github-com --type=kubernetes.io/dockerconfigjson --from-file=.dockerconfigjson=/dev/stdin
    ```

3. Now, you can reference the above secret from your pod's spec definition via imagePullSecrets field:

    ```yaml
    spec:
      containers:
      - name: your-container-name
        image: docker.pkg.github.com/<ORG>/<REPO>/<PKG>:<TAG>
      imagePullSecrets:
      - name: dockerconfigjson-github-com
    ```

### Compose

```bash
# from the project root run
docker-compose up --build
```

- The vote app runs in [http://localhost:5000](http://localhost:5000)
- The result app runs in [http://localhost:5001](http://localhost:5001)

### K8s

To run Kubernetes locally:

```bash
minikube start

# from the project root run
kubectl apply -f k8s

# run this command and leave it running
minikube service vote-loadbalancer-service -n <NAMESPACE>

# in a separate terminal run this command
minikube service result-loadbalancer-service -n <NAMESPACE>
```

When finished run:

```bash
# from the project root run
kubectl delete -f k8s

minikube stop
```
