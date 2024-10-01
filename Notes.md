For this to work on your machine you will need to make sure to start:
- docker desktop
- minikube (minikube start)
- in a terminal window start the tunnel (minikube tunnel) and remember to enter machine password

You will need the following
* dockerhub user: 
* dockerhub PAT (personal access token): 
* dockerhub repo: panama69/randomquotes-k8s
* Octo API key: 

---

The base environment isn't using an ingress point so you need to expose a port with a loadbalancer if you want to access the base image
* manually create deployment (https://minikube.sigs.k8s.io/docs/handbook/accessing/)
* kubectl create deployment randomquotes-k8s --image panama69/randomquotes-k8s:latest
* kubectl expose deployment randomquotes-k8s --type=LoadBalancer --port=8080

----------------------
minikube start (use delete if having issues connecting to it and then start)\
kubectl apply -f namespaces.yaml\
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml\

This will expose 8080 for app in default pod\
minikube tunnel\
kubectl create deployment randomquotes-k8s --image panama69/randomquotes-k8s:latest\
kubectl expose deployment randomquotes-k8s --type=LoadBalancer --port=8080\

Add or make sure the K8s Octo Agent is deployed\
	Deployment Targets -> Kubernetes Agent\
	
Delete pods & deployments https://stackoverflow.com/questions/33509194/command-to-delete-all-pods-in-all-kubernetes-namespaces\
kubectl delete --all deployments --namespace=default\
kubectl delete --all pods --namespace=foo\



IF YOU CAN NOT CONNECT CHECK THE TUNNEL IS UP (https://minikube.sigs.k8s.io/docs/handbook/accessing/)
minikube tunnel

additional info on ingress and tunnel
https://medium.com/@areesmoon/setting-up-minikube-and-accessing-minikube-dashboard-09b42fa25fb6#:~:text=Save%20the%20script%20using%20CTRL,later%20in%20the%20next%20article).
