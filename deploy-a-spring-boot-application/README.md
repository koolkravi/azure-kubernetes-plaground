# Deploy Spring Boot Application to the Azure Kubernetes Service

# A. Prerequisites

## 1. [Azure Command-Line Interface (CLI)](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
The current version of the Azure CLI is 2.8.0 (as of 12/07/2020)

```
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
```

### useful commands
```
az --version
az login
```

## 2. A supported Java Development Kit (JDK) . see https://aka.ms/azure-jdks

## 3. A Docker client
```
https://docs.docker.com/toolbox/toolbox_install_windows/
DockerToolbox-19.03.1.exe
```

### useful commands
```
open Docker Quickstart Terminal from desktop
docker run hello-world
docker-machine ls
```

## 4. [The ACR Docker credential helper](https://github.com/Azure/acr-docker-credential-helper)
The ACR Docker Credential Helper allows users to sign-in to the Azure Container Registry service using their Azure Active Directory (AAD) credentials.

```
iex ([System.Text.Encoding]::UTF8.GetString((Invoke-WebRequest -Uri https://aka.ms/acr/installaad/win).Content))
```

### useful commands
```
az acr login -n <registry name>
```

# B. Create the Spring Boot on Docker Getting Started web app

## 5. Clone the Spring Boot on Docker Getting Started sample project into the directory

```
mkdir springboot
cd springboot
git clone https://github.com/spring-guides/gs-spring-boot-docker.git
cd gs-spring-boot-docker/complete/
```
## 6. Use Maven to build and run the sample app
```
mvn package spring-boot:run
```
## 7. Test the web app by browsing 
```
http://localhost:8080
```

# C. Create an Azure Container Registry using the Azure CLI

## 8. Log in to your Azure account
```
az login
az account set -s <YourSubscriptionID>
az group create --name=koolkravi-kubernetes --location=eastus
```

### useful commands
```
az account show
az account list
```

## 9. Create a private Azure container registry

```
az acr create --resource-group koolkravi-kubernetes --location eastus --name koolkravi --sku Basic
```

# D. Push your app to the container registry via Jib

## 10.  Log in to your Azure Container Registry from the Azure CLI
```
# set the default name for Azure Container Registry, otherwise you will need to specify the name in "az acr login"
az configure --defaults acr=koolkravi
az acr login
```

###
```
az acr login -n <your registry name> 
```

## 11.  update properties pom.xml file with the registry name for Azure Container Registry and the latest version of jib-maven-plugin.
```
cd springboot\gs-spring-boot-docker\complete

<properties>
   <docker.image.prefix>koolkravi.azurecr.io</docker.image.prefix>
   <jib-maven-plugin.version>2.4.0</jib-maven-plugin.version>
   <java.version>1.8</java.version>
</properties>
```

## 12. Update the <plugins> collection in the pom.xml file so that the <plugin> element contains an entry for the jib-maven-plugin

```
<plugin>
  <artifactId>jib-maven-plugin</artifactId>
  <groupId>com.google.cloud.tools</groupId>
  <version>${jib-maven-plugin.version}</version>
  <configuration>
     <from>
         <image>mcr.microsoft.com/java/jdk:8-zulu-alpine</image>
     </from>
     <to>
         <image>${docker.image.prefix}/${project.artifactId}</image>
     </to>
  </configuration>
</plugin>
```

- ps: Note that we are using a base image from the Microsoft Container Registry (MCR): mcr.microsoft.com/java/jdk:8-zulu-alpine, which contains an officially supported JDK for Azure

## 13. run the following command to build the image and push the image to the registry

```
cd springboot/gs-spring-boot-docker/complete
az acr login && mvn compile jib:build

$out
Built and pushed image as koolkravi.azurecr.io/gs-spring-boot-docker
```

# E. Create a Kubernetes Cluster on AKS using the Azure CLI

## 14. Create a Kubernetes cluster in Azure Kubernetes Service
```
az aks create --resource-group=koolkravi-kubernetes --name=koolkravi-akscluster --attach-acr koolkravi --dns-name-prefix=koolkravi-kubernetes --node-count=2  --generate-ssh-keys
```

## 15. Install kubectl using the Azure CLI
```
az aks install-cli
```

## 16. Download the cluster configuration information so you can manage your cluster from the Kubernetes web interface and kubectl
```
az aks get-credentials --resource-group=koolkravi-kubernetes --name=koolkravi-akscluster

#out
Merged "koolkravi-akscluster" as current context in C:\Users\ravikumar\.kube\config
```

# F. Deploy the image to your Kubernetes cluster - Deploy with kubectl

## 17. 
```
kubectl run gs-spring-boot-docker --image=koolkravi.azurecr.io/gs-spring-boot-docker:latest
```
- ps: gs-spring-boot-docker is container name

## 18. Expose your Kubernetes cluster externally by using the kubectl expose command. 
Specify your service name, the public-facing TCP port used to access the app, and the internal target port your app listens on

```
kubectl expose deployment gs-spring-boot-docker --type=LoadBalancer --port=80 --target-port=8080
```

## 19. Query the external IP address and open it in your web browser:
```
kubectl get services -o jsonpath={.items[*].status.loadBalancer.ingress[0].ip} --namespace=default
```

### useful commands
```
kubectl get services -o wide
#out
52.188.74.159
```

## 20.  open from browser
52.188.74.159

# G. Deploy with the Kubernetes web interface

## 21. open command promt and run below
#pen the configuration website for your Kubernetes cluster in your default browser:
```
az aks browse --resource-group=koolkravi-kubernetes --name=koolkravi-akscluster

#out 
# See step 16 to get .kube\config 
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/login
```

### note
- You can also integrate Azure Active Directory authentication to provide a more granular level of access.


### 21-1. select the link to deploy a containerized app
### 21-2. Select CREATE AN APP

# H. Clean up

Below command will delete all resources created
```
az group delete -n koolkravi-kubernetes
```







# Reference

## 1. Deploy Spring Boot Application to the Azure Kubernetes Service
```
https://docs.microsoft.com/en-us/azure/developer/java/spring-framework/deploy-spring-boot-java-app-on-kubernetes
```
## 2. Day 6-15 Azure Kubernetes Service core concepts
```
https://azure.microsoft.com/mediahandler/files/resourcefiles/kubernetes-learning-path/Kubernetes%20Learning%20Path%20version%201.0.pdf
https://docs.microsoft.com/en-us/azure/aks/concepts-clusters-workloads
```

## 3. CLI reference: 
```
https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest
https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest
```

## 4. Jib - Containerize your Maven project
```
https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin
```
##5. To find total cores available for a region
```
Azure Portal: All services => Subscriptions => Select your subscription => Usage + Quotas
```
