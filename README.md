# Deploying Images to Kubernetes from Private Docker Repository

## Common Workflow

A common workflow to deploy your app to K8 entails;

* Commit to Git triggering Jenkins CI build
* Jenkins packages your app into a Docker image
* Docker image gets pushed to a Docker registry (Public/Private)
* Get docker image to K8s cluster (I'll demo a private repo)

To pull an image from a private registry eg Nexus, private Dockerhub registry, ECR, there are two main steps to follow.

1. Create **Secret** component in K8s
   * contains access tokens and credentials for Docker registry
2. Configure Deployment/Pod
   * To use Secret leveraging **imagePullSets**

I will be using a private docker repo hosted in AWS ECR and to ready the process I  build a NodeJS image locally with a Dockerfile, pushed it to Amazon ECR, named and tagged it as **my-app:1.0**.
![image-1](./images/image-1.png)

Locally I've readied my local development area and my Minikube cluster is empty. I've also authenticated my local environment with AWS by retrieving an authentication token from AWS and authenticating my Docker client to my registry.

Since Minikube has no access to ECR, I'll configure it with my AWS credentials to enable login directly. There are three ways of doing this. The first way is by configuring the AWS credentials inside Minikube (you have to SSH for this).

```shell
minikube ssh
```

![image-2](./images/image-2.png)

The second option is following a minikube addons configuration **[registry-creds](https://minikube.sigs.k8s.io/docs/tutorials/configuring_creds_for_aws_ecr/)** which is the easiest in my opinion.

![image-3](./images/image-3.png)

The third is by creating a secret docker registry key using **kubectl** and configure it with the yaml file.

```yml
apiVersion:v1
kind: Secret
metadata:
  name: my-registry-key
data:
  .dockerconfigjson:
type: kubernetes.io/dockerconfigjson
```

![image-4](./images/image-4.png)

Next thing to do is to configure the deployment yaml file accordingly with the ECR image endpoint and the registry key.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: my-app
        image: ********.dkr.ecr.eu-west-1.amazonaws.com/my-app:1.0
        imagePullPolicy: Always
        ports:
          - containerPort: 3000
```

![image-5](./images/image-5.png)

And by the way, if struggling to pass yaml files like me due to indentation errors  you can use a yaml lint like [this](http://www.yamllint.com/) one.

![image-6](./images/image-6.png)

In a nutshell, that's how you deploy images in K8's from a private Docker repository.