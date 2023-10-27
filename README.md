## argo-k8s + AZURE
#### First you need to install Azure CLI on your OS and create two clusters of k8s. Keep in mind - it's not free!
#### In Azure I created two Resource groups: k8s-group1 and k8s-group2
![](/png/1.png "1100")
#### We create the clusters themselves:
~~~
$ az aks create --resource-group k8s-group1 --name k8s-az --node-count 3 --node-vm-size standard_b2s --kubernetes-version 1.26.6 --location canadacentral
$ az aks create --resource-group k8s-group2 --name k8s-az --node-count 3 --node-vm-size standard_b2s --kubernetes-version 1.26.6 --location canadaeast
~~~
#### Wait 5 minutes – creating
#### Then get a credentials and login in to 1 cluster
![](/png/2.png "1100")
#### We check our nodes:
~~~
$ kubectl get nodes
~~~
![](/png/3.png "1100")
#### I created a [repository](https://github.com/1100-git/argo-k8s) on GitHub while private - in order to show you how to connect a private repository in ArgoCD.
#### Install ArgoCD in first cluster:
~~~
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm repo update
$ kubectl create namespace argocd
$ helm install argocd argo/argo-cd -n argocd
~~~
![](/png/4.png "1100")
#### Get pass:
~~~
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
~~~
#### My pass from ArgoCD group1 CCZ0sLLm51VM7RAb. Port forward command:
~~~
$ kubectl port-forward service/argocd-server -n argocd 8080:443
~~~
![](/png/4.1.png "1100")
#### Logged in:
![](/png/5.png "1100")
#### Create a deploy, Apply this yml file:
![](/png/6.png "1100")
~~~
~/Documents/work/ArgoCD/argo-k8s/demo-dev$ kubectl apply -f root.yml 
application.argoproj.io/root created
~~~
#### Result:
![](/png/7.png "1100")
#### As you can see, it can't connect to git repo, let's connect, first create the ssh key, I do it all on Ubuntu 22.04:
~~~
$ ssh-keygen
~~~
![](/png/8.png "1100")
#### *.pub – to the git-hub repo secrets, other – to the ArgoCD
![](/png/9.png "1100")
![](/png/10.png "1100")
![](/png/11.png "1100")
#### Now let's add the manifest - files to the repository, and ArgoCD will automatically see and captivate, because it was spelled out in his debugging deploy. These manifest files build nginx and httpd images and create load-balancerS for each
![](/png/12.png "1100")
#### Now ArgoCD will monitor our repository and automatically make changes if they are in the process of development. Why did I create 2 clusters? - precisely because usually the development consists of a test environment and a working one - so if you look at the debugging - they show that the other ArgoCD follows another directory. Here's what we have now:
![](/png/13.png "1100")
![](/png/14.png "1100")
#### If we change the parameters of our manifest file, we will get an instant automatic result, ArgoCD will automatically block all changes after making changes to the repository within 3 minutes. Try to change replicaCount: 6 in the values_dev.yml file and push it to your repository - and you will see how your build changes automatically within 3 minutes - this is the task of ArgoCD.
#### Well, that's all for now - look at my repository, everything is clearly configured there. It was an instruction on how to set up ArgoCD on an openwork cluster
