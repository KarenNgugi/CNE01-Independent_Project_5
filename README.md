# CNE01 Independent Project 5 - CI/CD

## Project Overview
This project seeks to demonstrate Continuous Integration (CI) and Continuous Delivery (CD) using both Jenkins and ArgoCD respectively.

The following will be demonstrated upon completion of the project:
- designing a basic CI/CD architecture using Jenkins and ArgoCD
- keeping both CI and CD configuration in Git
- writing a functional Jenkinsfile with the required stages
- writing Kubernetes manifests that ArgoCD can sync
- documenting the setup and evidence clearly
- using clear, meaningful commits to show progress

## Architecture
![](https://github.com/KarenNgugi/CNE01-Independent_Project_5/blob/main/docs/screenshots/CICD%20architecture.jpg)

You can find more information on the architecture in [docs/architecture.md](https://github.com/KarenNgugi/CNE01-Independent_Project_5/blob/main/docs/architecture.md).

## Project Setup
Before you proceed, make sure you have the following:
- Git
- Docker
- Kubectl
- A Kubernetes cluster (e.g. Minikube)

### Jenkins
Due to issues I encountered with the official Jenkins Docker image, I created a custom Jenkins image with the required `libatomic1` OS package installed for the Node.js runtime.

First create a jenkins.Dockerfile:
```
FROM jenkins/jenkins:lts

USER root

RUN apt-get update \
    && apt-get install -y libatomic1 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

Then build the image:
```
docker build -t my_jenkins_image -f jenkins.Dockerfile .
```

Once the image is built, you need to create a volume that Jenkins can store its data at. This prevents loss of data otherwise you will need to set up Jenkins again every time you start up a Jenkins container:
```
docker volume create jenkins_home
```


Create a container and port map to 8080 so as to make the Jenkins container accessible via the browser. Attach the volume created in the previous step:
```
docker run -d \
    --name jenkins \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins_home:/var/jenkins_home \
    my_jenkins_image
```

Once the container is created, access Jenkins on your browser via `http://localhost:8080`. From there, select the option to install recommended plugins, and then provide the details to create the admin account.

Once that is done and you are logged in, you will see the dashboard. Go to Manage Jenkins on the top right (settings icon) and go to Plugins. Under "Available Plugins", select NodeJS to install it.

Once NodeJS is installed, go to Manage Jenkins >> Tools. Scroll down until you find "NodeJS Installations" and create a new NodeJS tool. Give it the name "NodeJS" since that's the name referenced in the [Jenkinsfile](https://github.com/KarenNgugi/CNE01-Independent_Project_5/blob/main/Jenkinsfile). This project used Node version 26.8.2.

Once you are done with this, go back to the main/dashboard page and click "New Item" on the left pane. Select the "Pipeline" option and give it any name you desire (e.g. "Karen's Demo CI/CD Pipeline"). Then scroll down to Pipeline section. Under Definition, select the "Pipeline Script from SCM" option. Then under Git, provide the link to the GitHub project (`https://github.com/KarenNgugi/CNE01-Independent_Project_5`). Ensure the branch specifier is set to `*/main` and the ScriptPath has selected the `Jenkinsfile` option, then click Save. In the next page, select "Build Now" on the left pane. Select the recently created build number in the Builds box, and go to Pipeline Overview to view the progress of the build.

### Kubernetes
Clone this project into a directory of your choice:
```
git clone https://github.com/KarenNgugi/CNE01-Independent_Project_5.git
```

Navigate into the `k8s` directory:
```
cd CNE01-Independent_Project_5/k8s
```

Create the namespace:
```
kubectl apply -f namespace.yaml
```

Set the new namespace as the default one:
```
kubectl config set-context --current --namespace=cicd-namespace
```

The namespace is created manually because the ArgoCD Application itself is deployed into the `argocd` namespace, while the application resources are deployed into `cicd-namespace`.

Apply the remaining manifests to create the resources:
```
kubectl apply -f configmap-site.yaml
kubectl apply -f service.yaml
kubectl apply -f deployment.yaml
```

To view the website being served by the deployment, you first obtain the IP address of your cluster. In my case, I'm using Minikube so I run `minikube service cicd-service -n cicd-namespace` to get the IP address then I access the site on `http://<minikube_ip_address>:30080`.
### ArgoCD
In a new terminal create a new namespace called `argocd` then run the following command to install ArgoCD resources:
```
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -n argocd
```

Wait until all pods are running:
```
kubectl get pods --watch -n argocd
```

Once all the pods are ready, map ArgoCD's port 443 to a port of your choice that you can access on your browser, e.g. 8082:
```
kubectl port-forward svc/argocd-server 8082:443 -n argocd
```

Open a new terminal and obtain the admin password via:
```
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
``` 

Then navigate to the `argocd` directory and apply the application manifest:
```
cd ../argocd
kubectl apply -f application.yaml
```

  
Then go to `http://localhost:8082` on your browser, input the admin password, then update it in the User Info menu so that it is easier for you to log in next time. Then go  to the Application page to see an overview of your ArgoCD applications. You can click on it to obtain further details.

## Troubleshooting
If experiencing issues, you can check [here]() for the troubleshooting guide.
## Authors
[KarenNgugi](https://github.com/KarenNgugi)
