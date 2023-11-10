# kubernetes-backstage-pilot

En el ámbito de la gestión y despliegue de aplicaciones, Kubernetes ha emergido como una plataforma líder que facilita la orquestación eficiente de contenedores. Al mismo tiempo, herramientas como Backstage han ganado popularidad al proporcionar un entorno centralizado para la gestión del ciclo de vida de las aplicaciones. Este proyecto de investigación y prueba se enfoca en la sinergia entre Kubernetes y Backstage, explorando cómo su integración puede mejorar la eficiencia en el desarrollo, implementación y mantenimiento de aplicaciones en entornos modernos.

## Contexto:

Kubernetes, desarrollado por Google, ha revolucionado la gestión de contenedores al proporcionar una plataforma robusta y escalable para automatizar la implementación, escalado y operaciones de aplicaciones en contenedores. Por otro lado, Backstage, una iniciativa de Spotify, se ha destacado como una plataforma de código abierto que simplifica la administración de servicios y aplicaciones a lo largo de su ciclo de vida.

## Objetivos de la Investigación:

- Explorar la Arquitectura de Kubernetes: Analizar los componentes clave de Kubernetes y comprender cómo facilita la gestión de contenedores a escala.

- Investigar las Funcionalidades de Backstage: Examinar las características de Backstage que permiten la gestión eficiente de servicios, desde el desarrollo hasta la implementación y el monitoreo.

- Integración entre Kubernetes y Backstage: Investigar cómo la integración de Kubernetes y Backstage puede mejorar la visibilidad y la gestión de aplicaciones en un entorno de contenedores.

## Metodología de Prueba:

- Configuración del Entorno: Establecer un entorno de prueba con Kubernetes como orquestador de contenedores y Backstage como plataforma de gestión.

- Despliegue de Aplicaciones de Muestra: Implementar aplicaciones de muestra utilizando Kubernetes y gestionar su ciclo de vida a través de Backstage.

- Monitoreo y Seguimiento: Evaluar las capacidades de monitoreo proporcionadas por Kubernetes y Backstage para garantizar la visibilidad y la resolución proactiva de problemas.

- Automatización de Tareas: Probar la automatización de tareas de implementación, escalado y actualización de aplicaciones utilizando la integración de Kubernetes y Backstage.

## MicroK8s

### Install MicroK8s on Linux
sudo snap install microk8s --classic

### Check the status while Kubernetes starts
microk8s status

### Turn on the services you want
microk8s enable dashboard dns registry metrics-server hostpath-storage ingress

### Check services enabled
microk8s status

###  Turn off a service
microk8s disable <name> 

### Start and stop Kubernetes
microk8s start 

microk8s stop 

### Check Kubernetes
microk8s kubectl get all --all-namespaces

### Access the Kubernetes dashboard
microk8s dashboard-proxy

## Backstage

### Install Backstage
See https://backstage.io/docs/getting-started/

### Config Backstage
See https://backstage.io/docs/getting-started/configuration

### Config GitHub 

#### Set GitHub Token 
export GITHUB_TOKEN=\<github-token> 

NOTE: Get token from Github.com

Or you can add a Github token to app-config.local.yaml file. If you do that, you don't need to export

#### Config Github in app-config.yaml
add:

```yaml
integrations:
  github:
    - host: github.com
      # This is a Personal Access Token or PAT from GitHub. You can find out how to generate this token, and more information
      # about setting up the GitHub integration here: https://backstage.io/docs/getting-started/configuration#setting-up-a-github-integration
      token: ${GITHUB_TOKEN}
    ### Example for how to add your GitHub Enterprise instance using the API:
    # - host: ghe.example.net
    #   apiBaseUrl: https://ghe.example.net/api/v3
    #   token: ${GHE_TOKEN}
```

### Config Postgress in app-config.yaml
add:

```yaml
backend:
 database:
    client: pg
    connection:
      host: localhost
      port: 5432
      user: postgres
      password: secret 
```

## Kubernetes Plugin

### Kubernetes control plane 
alias mkctl="microk8s kubectl"

mkctl cluster-info

Kubernetes control plane is running at https://127.0.0.1:16443

### Integration Kubernetes  
https://backstage.io/docs/features/kubernetes/

https://backstage.io/docs/features/kubernetes/configuration/

### Set Kubernetes token

microk8s kubectl create serviceaccount backstage -n default

microk8s kubectl create clusterrolebinding backstage-clusterrolebinding --clusterrole=cluster-admin --serviceaccount=default:backstage

export K8S_MINIKUBE_TOKEN_NAME=$(kubectl -n default get serviceaccount/backstage -o jsonpath='{.secrets[0].name}')

export K8S_MINIKUBE_TOKEN=$(kubectl -n default get secret $K8S_MINIKUBE_TOKEN_NAME -o jsonpath='{.data.token}' | base64 --decode)

NOTE: If token is empty, use: (example)

export K8S_MINIKUBE_TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IktSUEhyUmY4aUVSVGVMbllRcU1jTmZZTTR2SUZPY2NEbXh3SnU2emt1c3MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJtaWNyb2s4cy1kYXNoYm9hcmQtdG9rZW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImI1MjE1ODNmLTgxZjYtNDRmNy04MGFiLTI4Y2JlNzhkN2QzOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpkZWZhdWx0In0.QqbYb7GpZyHPSDCysZo9sMEm3AZA2HF_TWw6TBDofiakA0xad7i5INQ_7AiE_08GPBJJLM7u4ymZGP44oWZfDVBNSedzKGoo4dpU5nFWqBAjmglCtMxASYLZnlFnrS1SoiQYs5plTkJ0L8vwfYMIjTK8Do1nQ13qdpRXE5mDoy2857e4W-rlaE4grbwtknky2fgZA7HIDa3c-l9sNqoPSTEeo9oWkhn3Y-GFha4hcXW8NP4GuEWf8FliqA5-H7zddPS7sJ8Cr9UvpU7izSSIP40VC_vv5DngPtngbMf05JMoGQMABytHtffHLsB60DzDfJiHNZeOmkknGiYfOdNndA

### Config Kubernetes in app-config.yaml
add:

```yaml
kubernetes:
  serviceLocatorMethod:
    type: 'multiTenant'
  clusterLocatorMethods:
    - type: 'config'
      clusters:
        - url: https://127.0.0.1:16443 # Info: Kubernetes control plane 
          name: microk8s
          authProvider: 'serviceAccount'
          skipTLSVerify: true
          skipMetricsLookup: false
          serviceAccountToken: ${K8S_MINIKUBE_TOKEN}
          #caData: ${K8S_CONFIG_CA_DATA}
```

## Jenkins Plugin

### Integration Jenkins
https://github.com/backstage/backstage/tree/master/plugins/jenkins

https://www.npmjs.com/package/@backstage/plugin-jenkins-backend

## Deploy jenkins on Kubernetes
helm repo add jenkins https://charts.jenkins.io

helm repo update

helm search repo jenkins --devel --versions

helm install jenkins jenkins/jenkins	4.3.23 --namespace default

## Uninstall jenkins 
helm uninstall jenkins

### Config Jenkins in app-config.yaml
add:

```yaml
jenkins:
  baseUrl: http://localhost:8080
  username: admin
  apiKey: 1131dfee75059201cdd63633b784c25e15 #Get Token From Jenkins
  # optionally add extra headers
  # extraRequestHeaders:
  #   extra-header: my-value
```

### Get your 'admin' user password:
kubectl get pods -n default | grep jenkins

kubectl exec --namespace default -it <pod-id> -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo

### Connect to Jenkins as an administrator
kubectl get svc -n default | grep jenkins

kubectl --namespace default port-forward svc/\<service-id> 8080:8080

Example:  kubectl --namespace default port-forward svc/jenkins-1683125347 8080:8080

NOTE: Open localhost:8080 in a browser. Conect to jenkins with user Admin & password <See Get your 'admin' user password>

### Get Token From Jenkins
Log in to the Jenkins instance as an administrator

Click on “Manage Jenkins” in the Jenkins dashboard

Click on the “Manage Users“

Select the user we want to generate an API token for and click on their name to access their user configuration page

Generate a token using the “Add new token” section of the user configuration page

Click on the “Copy” button to copy the token to the clipboard

Save the configurations

### Add a pipeline to Jenkins
Log in to the Jenkins instance as an administrator

Create a Organization Folder item with backstage-git-scm name

Add a Project: Single repository

Assign a project name

Select a source name: Github

Select a Credential

Asign a Repository HTTPS URL (Github repository where is your jenkinsfile) - https://github.com/ndigrazia/kubernetes-backstage-pilot.git

### Kubernetes plugin to Jenkins

Install & configure Jenkins with Kubernetes

https://plugins.jenkins.io/kubernetes/

##  Catalog File
Create a catalog-info.yaml file in your project

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: "kubernetes-backstage"
  annotations:
    backstage.io/kubernetes-id: node-web-app # container name - See deployment.yaml file
    backstage.io/kubernetes-namespace: default # kubernetes namespace where is container deployed 
    backstage.io/techdocs-ref: dir:.
    jenkins.io/github-folder: 'backstage-git-scm/Github' #github-organization-project-name/job-name - See Add a pipeline to Jenkins
spec:
  type: service
  owner: user:guest
  lifecycle: experimental
```

##  Start Backstage

###  Set Enviroment
See "Set Kubernetes token"

See "Set GitHub Token" 

### Start microk8s
microk8s start

### Start docker postgress
docker run --name postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=postgres-p 5432:5432 -d postgres

Or:

docker ps -a | grep postgres

docker start \<container-id>

### Start Jenkins
kubectl get svc -n default | grep jenkins

kubectl --namespace default port-forward svc/\<service-id> 8080:8080

### Start Backstage service - Dev
cd backstage

yarn dev
