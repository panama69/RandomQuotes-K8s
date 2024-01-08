This is a sample application deploy to Kubernetes.  It's a good first application as it has no other components, but has an environment variable you can use to practice secrets on.

The docker image is built using a GitHub action and it is pushed to Docker Hub.  You can find the docker repository here: https://hub.docker.com/r/octopussamples/randomquotes-k8s/tags

# Prep Work

The docker image, manifest files, and variables will be provided to you.  You need to provide a k8s cluster, octopus instance, and worker.

## 1. Install K8s
Install minikube, rancher desktop, or docker desktop locally.  

## 2. Configure K8s
Open up a command prompt or terminal.  Change the current directory in the terminal to the `k8s/provision` folder in this repo.
- Run the following commands:
    - Create all the namespaces: `kubectl apply -f namespaces.yaml`
    - Create the service account for deployments: `kubectl apply -f service-account-and-token.yaml`
    - To get the token value run: `kubectl describe secret octopus-svc-account-token`.  Copy the token to a file for future usage.
    - Install the NGINX Ingress Controller: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml`
    - Run `kubectl describe service kubernetes`.  Copy the endpoint, for example `172.18.135.254:6443` for later.

## 3. Pre-Configure Octopus
Using your cloud instance of choice do the following:

- Create a local worker pool.
- Install a polling tentacle locally and assign it to that local worker pool.  This will be used to deploy to k8s.
- Go to Infrastructure -> Accounts. Add the token from the earlier step.
- Go to Infrastructure -> Targets.  Add the kubernetes cluster.  
    - Use the endpoint IP address from earlier.  For example `https://172.18.135.254:6443`
    - Ensure the checkbox `Skip TLS Verification` is checked to make things easier.
    - Use the token account you created from earlier.
    - Use the local worker pool from earlier.
- Go to Library -> Git Credentials.
    - Add a new GitHub PAT token for your user.  
        - The PAT will need explict access to OctopusSamples.  
        - Create a fine-grained key
        - Set the resource owner to OctopusSamples
        - Select Public Repositories (read-only) as the option.
    - Username will be your username.
- Go to Library -> Feeds
    - Add a docker hub feed
    - Provide your username and PAT or a service account username and PAT otherwise you won't be able to create releases.

## 4. Configure your hosts file.
Go to your hosts file (if on Windows) and add the following entries.  The nginx ingress controller uses host headers for all routing.  Doing this will allow you to easily access the application running on your k8s cluster.

```
127.0.0.1       randomquotes.local
127.0.0.1       randomquotesdev.local
127.0.0.1       randomquotestest.local
127.0.0.1       randomquotesstaging.local
127.0.0.1       randomquotesprod.local
```

# 200 Session at RKO

All the activities below will be done at RKO.

## 1. First Activity - Basic Deployment 

In the first activity we will do a standard manifest file deployment to the default namespace in kubernetes using `kubectl apply`.

The goal of this activity is to expose you to a simple manifest file and what the experience is like for a bare-bones, no automation experience.

These instructions will deploy the following to the default namespace.  

- Secret
- Deployment (Image)
- ClusterIp Service
- Ingress Rule

To perform the deployment do the following:
- Go to https://hub.docker.com/r/octopussamples/randomquotes-k8s/tags and find the latest version tag (0.1.3 for example).  Update the `image`` entry in the randomquotes-deployment.yaml file.
- Open up a command prompt or terminal.  Change the current directory in the terminal to the `k8s/base` folder in this repo. 
- Run `kubectl apply -f randomquotes-secrets.yaml`
- Run `kubectl apply -f randomquotes-deployment.yaml`

It might take a moment for the deployment to finish.  I like to check the status of the pods.  Run `kubectl get pods` until the randomquotes pod shows up as healthy.

Once the deployment is finished go to http://randomquotes.local.

## 2. Second Activity - Leveraging Kustomize

In the second activity we will deploy to each of the environment namespaces using kustomize and overlays.

In the previous activity we deployed to the default namespace.  In the real-world, we'd want to do some environmental progression and testing before pushing up to production.  Rather than have a manifest file per environment, we will have a kustomize overlay per environment.  The following items are changed with each kustomize file:

- The image version
- The secret value
- The ingress rule

If we were using ArgoCD or some other similar tool we could use these kustomize overlays with no additional configuration changes.  

We are going to deploy to all four namespaces.  The instructions are the same, so repeat the following for each environment.

- Go to https://hub.docker.com/r/octopussamples/randomquotes-k8s/tags and find the latest version tag (0.1.3 for example).  Go to k8s/overlays/[environment]/kustomization.yaml file.  Update the newtag entry with the latest version.
- Open up a command prompt or terminal.  Change the current directory in the terminal to the `k8s/overlays/[environment]/` folder in this repo. 
- Run `kubectl apply -k ./`
- To check the pods run `kubectl get pods -n [environment namespace name]`
- Once the deployment is over go to http://randomquotes[environment name].local
- Assuming everything is working, repeat the following steps for each environment.

## 3. Third Activity - Using Octopus Deploy

In the final activity we will configure Octopus Deploy to deploy our application to k8s using the manifest files.

In this example, we will put the kustomize overlays aside and instead use Octopus' raw yaml steps + structured variable configuration.

- Create a new project called `Random Quotes Manifest Files`
- Add the following variables:
    - Name: spec:rules:0:host
        - Values: randomquotesdev.local Environment: Development
        - Values: randomquotestest.local Environment: Test
        - Values: randomquotesstaging.local Environment: Staging
        - Values: randomquotesprod.local Environment: Production
        - Type: Text
    - Name: spec:template:spec:containers:0:image
        - Value: octopussamples/randomquotes-k8s:#{Octopus.Action.Package[randomquotes-k8s].PackageVersion}
        - Type: Text
    - Name: stringData:homepageDisplay
        - Value: [Your Choice]
        - Type: Sensitive
- Go to the deployment process
    - Add a DEPLOY RAW KUBERNETES YAML
        - Name: Create Random Quotes Secret
        - Worker Pool: Use the local worker pool
        - Role: Use the role from your k8s cluster
        - YAML Source: Git Repository
        - Git Credentials: Use the git credentials from the library
        - Repository URL: https://github.com/OctopusSamples/RandomQuotes-K8s.git 
        - Branch Settings: main
        - Paths: k8s/base/randomquotes-secrets.yaml
        - Structured Configuration Valures: Check the `Enable Structured Configuration Variables` checkbox
        - Namespace: #{Octopus.Environment.Name | ToLower}
    - Add a DEPLOY RAW KUBERNETES YAML
        - Name: Deploy Random Quotes
        - Worker Pool: Use the local worker pool
        - Role: Use the role from your k8s cluster
        - YAML Source: Git Repository
        - Git Credentials: Use the git credentials from the library
        - Repository URL: https://github.com/OctopusSamples/RandomQuotes-K8s.git 
        - Branch Settings: main
        - Paths: k8s/base/randomquotes-deployment.yaml
        - Structured Configuration Valures: Check the `Enable Structured Configuration Variables` checkbox
        - Referenced Packages: 
            - Package Feed: DockerHub
            - PackageId: octopussamples/randomquotes-k8s
            - Name: randomquotes-k8s
        - Namespace: #{Octopus.Environment.Name | ToLower}
    - Save the deployment process
- Create a release and deploy it to dev.
- Test by going to randomquotesdev.local.
- Promote the release through each environment.  Test along the way.