This is a sample application deploy to Kubernetes.  It's a good first application as it has no other components, but has an environment variable you can use to practice secrets on.

# Prep Work

- Install minikube, rancher desktop, or docker desktop locally.  
- Open up a command prompt or terminal.  Change the current directory to `k8s/provision` folder in this repo.
- Run the following commands:
    - Create all the namespaces: `kubectl apply -f namespaces.yaml`
    - Create the service account for deployments: `kubectl apply -f service-account-and-token.yaml`
    - To get the token value run: `kubectl describe secret octopus-svc-account-token`.  Copy the token to a file for future usage.
    - Install the NGINX Ingress Controller: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml`
- Go to your hosts file (if on Windows and add the following entries)

```
127.0.0.1       randomquotes.local
127.0.0.1       randomquotesdev.local
127.0.0.1       randomquotestest.local
127.0.0.1       randomquotesstaging.local
127.0.0.1       randomquotesprod.local
```

# Basic Deployment 

In the first activity we will do a standard manifest file deployment to the default namespace in kubernetes using `kubectl apply`.

Instructions Lorem Ipsum

# Leveraging Kustomize

In the second activity we will deploy to each of the environment namespaces using kustomize and overlays.

Instructions Lorem Ipsum

# Using Octopus Deploy

In the final activity we will configure Octopus Deploy to deploy our application to k8s using the manifest files.

Instructions Lorem Ipsum