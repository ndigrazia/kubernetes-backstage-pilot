# kubernetes-backstage-pilot
This is kubernetes-backstage


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


### Kubernetes control plane 
alias mkctl="microk8s kubectl"
mkctl cluster-info

Kubernetes control plane is running at https://127.0.0.1:16443

### Integration Kubernetes  
https://backstage.io/docs/features/kubernetes/

https://backstage.io/docs/features/kubernetes/configuration/

### Config Kubernetes token

microk8s kubectl create serviceaccount backstage -n default
microk8s kubectl create clusterrolebinding backstage-clusterrolebinding --clusterrole=cluster-admin --serviceaccount=default:backstage
export K8S_MINIKUBE_TOKEN_NAME=$(kubectl -n default get serviceaccount/backstage -o jsonpath='{.secrets[0].name}')
export K8S_MINIKUBE_TOKEN=$(kubectl -n default get secret $K8S_MINIKUBE_TOKEN_NAME -o jsonpath='{.data.token}' | base64 --decode)

NOTE: If token is empty use: (example)

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

### Config GitHub 

#### Set GitHub Token 
export GITHUB_TOKEN=<github-token> 
NOTE: Get token from Github.com

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

###  Start Backstage

####  Set Enviroment
See "Config Kubernetes token"
See "Set GitHub Token" 

#### Start docker postgress
ocker run --name postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=postgres-p 5432:5432 -d postgres

docker ps -a | grep postgres
docker start <container-id>

#### Start Backstage service - Dev
cd backstage
yarn dev