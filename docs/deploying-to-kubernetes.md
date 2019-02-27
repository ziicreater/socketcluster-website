---
id: deploying-to-kubernetes
title: Deploying to Kubernetes
sidebar_label: Deploying to Kubernetes
---

## Requirements

For the operating system, Linux or OSX is recommended. In addition to this, the following software is required:

- Node.js version 10 or higher. [Download Node.js](https://nodejs.org/en/).
- `docker` CLI. [Install Docker](https://docs.docker.com/install/).
- `kubectl` CLI. [Install Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

If using, GKE as your Kubernetes cluster, you will also need to install the gcloud command: https://cloud.google.com/sdk/install

!! If you want to setup your Kubernetes cluster from scratch, consider using [Rancher](https://rancher.com/) or [kops](https://github.com/kubernetes/kops).

## Install the asyngular CLI tool

```bash
npm install -g asyngular
```

!! If you get a permission issue, try `sudo npm install -g asyngular` instead.

Once installed, you can use this command to see a list of all available asyngular commands:

```bash
asyngular --help
```

## Create a new Asyngular project

```bash
asyngular create myapp
```

Once it finishes setting up and installing dependencies, navigate to your project directory with:

```bash
cd myapp
```

Note that all of the commands documented below assume that `myapp` is your working directory. If you want to run commands from a different directory, you will need to provide your project path explicitly to each command. See `asyngular --help` for details.

## Run the app inside a Docker container locally

During development, you may want to run your app locally inside a container (as an alternative to running it directly with the `node server` command). The `asyngular` CLI tool lets you do this with a single command (make sure that you are inside the `myapp` directory when you execute this command):

```bash
asyngular run
```

!! Essentially, this command launches an Asyngular container which binds to your local `myapp` directory and watches for code changes — This means that you don’t need to restart the container in order to test changes during development.

Once the app is running on your machine, you can access it from your browser at URL: http://localhost:8000

!! Note that when it comes to Docker and Kubernetes, the `asyngular` CLI commands are only shortcuts; they rely on the `docker` and `kubectl` commands behind the scenes. For more advanced scenarios, you may need to resort to using the `docker` or `kubectl` commands directly.

## View containerized app logs

```bash
asyngular logs -f
```

## Stop and remove an app container

```bash
asyngular stop
```

## Deploy the app to a Kubernetes cluster (initial deployment)

In this example, we will describe how to deploy to the [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) for simplicity.

By default, the Asyngular app is configured to be served over TLS/HTTPS/WSS so you will either need to disable TLS on the Ingress resource or you will need to provide the path to a private key and cert to use for TLS (the files need to be on your local machine; you may want to use a self-signed key and cert pair for maximum simplicity).

If you want to save yourself the hassle of generating a TLS key and cert pair, it may be easier to simply disable TLS; to do this, you just need to edit `myapp/kubernetes/agc-ingress.yaml` and remove the `tls` section under `spec` and the `annotations` section under `metadata`.

To deploy your app to your Kubernetes cluster, you will first need to add your cluster config to your `kubectl` `~/.kube/config file`. For GKE, you can just use this command:

```bash
gcloud container clusters get-credentials name-of-my-cluster
```

^ Replace `name-of-my-cluster` with the name of your cluster as displayed in your GKE control panel.

Now you can deploy using:

```bash
asyngular deploy
```

Just follow the prompts. The first deployment will prompt you for a lot of info. Once you’re done, a file called `asyngular-k8s.json` will be created in your project directory for convenience; this file will make subsequent deployments faster and easier. Do not commit this file to your repo though as it may contain encoded authentication details for your DockerHub account.

!! Note that for the initial deployment on GKE, it may take several minutes (sometimes 10 or more) for the ingress controller to launch. Once the ingress service warnings go away, you should be able to access your app over the public internet using the ingress IP address provided by GKE (e.g. `https://xxx.xxx.xxx.xxx`). **If using TLS, make sure that you use `https://` at the start of the URL because the GKE ingress does not offer HTTP to HTTPS redirection by default.**

!! Subsequent deployments using `asyngular deploy-update` should be much faster; often less than a minute.

!! On GKE, by default, the ingress load balancer has a connection timeout of `30` seconds; this is not ideal for WebSockets, so you may want to increase it to a bigger value like `86400` (one day). See [this page](https://cloud.google.com/load-balancing/docs/backend-service#timeout-setting) for steps.

## Deploy updates

Once your app is running on your Kubernetes cluster, you should deploy rollover updates using this command:

```bash
asyngular deploy-update
```

To try it out, you can change some of the back end code in `server.js` or front end code in `public/index.html` on your local machine and then deploy them to the cluster. This could take about a minute to roll out on GKE.

## Scale with kubectl

You can easily scale your Asyngular app using the `kubectl scale deployment` commands. Note that you should only need to scale your `agc-worker` and `agc-broker` deployments. The number of clients/users that your cluster can handle should scale linearly as you increase the number of worker and broker instances. An Asyngular cluster should be able to support thousands of worker and broker replicas.

To scale workers up to 2 replicas, you can use this command:

```bash
kubectl scale deployment agc-worker --replicas=2
```

To scale the brokers up to 2 replicas:

```bash
kubectl scale deployment agc-broker --replicas=2
```

## Undeploy the app

This will destroy all deployments and services from your Kubernetes cluster:

```bash
asyngular undeploy
```

## Customize Kubernetes .yaml files

As you build up your app, feel free to modify the default K8s `.yaml` files inside your `myapp/kubernetes/` directory. Once your app directory is created, the `asyngular` CLI will not make any further changes to them so you are free to modify them as you like.