# This repo was created to deploy the Minio Helm chart from ArgoCD, to be used with the Grafana stack in a demo env with custom values utilizing multiple sources. 
To note: These repos were developed for rapid redeployment of the Grafana Enterprise stack (there will be one for each of LGTM )based on the repo maintainers knowledge and 
experience around Kubernetes, this is not official Grafana Documentation. These can also be a basis for the OSS version. 
A rapidly redployable stack should be present in any enterprise environment, these examples can be used as a foundation but should not be the be all end all of MTTR. 

## The Why
Efficiency, repeatability and accountability. Utilizing GitOps approaches, one can (theoretically) lock down a cluster with all actions that take place doing so only after a 
peer review and merge request has been completed and approved. I have worked with large enterprise customers where only a handful of people had direct cluster access, all 
changes took place vis GitOps; Branch, edit, peer review, merge request and move forward. The "oops" moments were not completely gone but were reduced changes (if not done 
with direct cluster/machine access) were auditable in a single place (Git). 

## End Goals
In the end, I would like to setup 3 demo paths, the stack, the datasources and the dashboards. As will all things OpenSource, there are many ways these can be done. The end 
objective is to be sure you learn and come away from this with more confidence and knowledge for your day to day computing experiences. 

The demos rely on 2 base ites:
- A working Kubernetes cluster of your choice.
- ArgoCD installed and working properly.
  
## For a minimal Minio functioning setup, you will need 4 manifests:

- A namespace for Minio (metrics used here)
- A secret for your root user.
- A secret for your minio user. 
- Your Helm overrides file
- An ArgoCD Application pointing to the Helm chart and your overrides file. 

Examples of each of these can be found below. [Secret storage in Kubernetes is not natively secure](https://kubernetes.io/docs/concepts/configuration/secret/
#:~:text=%23%20values%20are%20base64,level%20of%20confidentiality), secrets are merely base64 encoded and should never be stored in Git. 
[The ExternalSecretOperator](https://external-secrets.io/latest/) paired with a Vault back end (or other secure secrets manager) is recommended. There are multiple 
approaches to external secrets, [this](https://www.redhat.com/en/blog/external-secrets-with-hashicorp-vault) linked article is an aging but good foundational knowledge for 
this approach. For this base tutorial, secrets are created imperatively by the user but this brings me to another point; [Kubernetes is a declarative system](https://
kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/), imperative approaches can be used for testing and learning but ultimately declarative approaches are 
desired. The focus of these repos is not the ESO or Vault or imperative vs declarative approaches but it is important to call these out for general knowledge. 

### The root user secret
```bash
apiVersion: v1
data:
  rootPassword: <base64 encoded password>
  rootUser: <Baae64 root user>
kind: Secret
metadata:
  name: minio-root 
  namespace: metrics 
type: Opaque
```

### The Minio user secret
```bash
apiVersion: v1
data:
  password: <your base64 password>
kind: Secret
metadata:
  name: minio-user 
  namespace: metrics 
type: Opaque
```

### The ArgoCD Application
```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  labels:
    app.kubernetes.io/instance: ge-application
  name: minio
  namespace: argocd
spec:
  destination:
    namespace: metrics
    server: https://kubernetes.default.svc
  project: default
  sources:
  - chart: minio
    helm:
      valueFiles:
      - $values/manifests/gem-minio-overrides.yaml
    repoURL: https://charts.min.io/
    targetRevision: 5.3.0
  - ref: values
    repoURL: https://github.com/pbmoses/ge-minio.git
    targetRevision: HEAD

```
### The overrides file
[Values that you have determined are best for your environment](./manifests/gem-minio-overrides.yaml))

Deploy each of the secrets `kubectl create -f <manifest>`. You can either apply the ArgoCD application imperatively `kubectl create -f application.yaml`or also place this in 
a repo to allow ArgoCD to sync from it. Your values file should be present in the source that ArgoCD is pointing to. 
