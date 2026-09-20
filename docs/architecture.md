# Architecture
## Project Architecture
![Image of project architecture including GitHub, Jenkins, and Argocd](https://github.com/KarenNgugi/CNE01-Independent_Project_5/blob/main/docs/CICD%20architecture.jpg)
## Kubernetes Architecture
![Image of Kubernetes workload involving a namespace, configmap, service, and deployment](https://github.com/KarenNgugi/CNE01-Independent_Project_5/blob/main/docs/K8s%20architecture.jpg)

# CI/CD Workflow

# Software versions
The following were the software versions of the software used at the time of this project:
| Software | Version |
| :--- | :--- |
| Docker | |
| Minikube | |
| kubectl | |
| Git | |
| Jenkins (Docker image) | 2.582 |
| NodeJS (in Jenkins) | 26.8.2 |
| ArgoCD | |

# Screenshots

# Troubleshhooting Guide
## Jenkins
## npm: not found
This error occurred because the environment does not yet have NodeJS installed. If you don't have the NodeJS plugin installed, go to **Manage Jenkins** >> **Plugins** >> **Available plugins** and select **NodeJS**. You will need to restart Jenkins for the change to take effect.

Once this is done, go to **Manage Jenkins** >> **Tools** >> **NodeJS Installations** and add a new tool called "NodeJS" (this is case sensitive).

### node: error while loading shared libraries: libatomic.so.1: cannot open shared object file: No such file or directory
This is an error encountered with the current Jenkins version ([2.582](https://hub.docker.com/layers/jenkins/jenkins/2.582/images/sha256-9baf323e2099be2201339225724d7ae1f26d633e8d89921d028dcc0a18d1687a)), and the only solution is to create a custom Jenkins file with the required OS package.

Create a new `jenkins.Dockerfile` and add the following:
```
FROM jenkins/jenkins:lts

USER root

RUN apt-get update \
    && apt-get install -y libatomic1 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

Then build the image using the following command:
```
docker build -t my_jenkins -f jenkins.Dockerfile .
```

Finally stop and remove the existing Jenkins container then run a new Jenkins instance using that specific image:
```
docker run -d
  --name my_jenkins
  --network jenkins
  -p 8080:8080
  -p 50000:50000
  -v jenkins_home:/var/jenkins_home
  my_jenkins
```

