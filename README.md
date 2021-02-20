# hvault-ocp-secrets-sync-operator
An operator to deploy [hvault-ocp-secrets-sync](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) in a namespace to create/update the OpenShift4 secrets from Hashi Vault.


This operator provides the functionality of creating :

* A namespace `hvault-secrets-sync-operator-system` for operator deployment 
* A cluster scoped custom resource deployment (CRD) named `hvaultsecretssync`
* A deployment that makes sure a Pod is running that contains the controller part of the operator
* A container image of the operator code
* A Controller code that watches for custom resource of type `hvaultsecretssync`in a cluster and deploys [hvault-ocp-secrets-sync](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) deployment



## How does it work?

Once this operator is deployed, it will be watching for the custom resources (CR) in the cluster and when a custom resource is created in a namespace, operator will deploy [hvault-ocp-secrets-sync agent](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) in a correspnding namespace. This `hvault-ocp-secrets-sync` agent will create secrets and it will be injected with a configmap from custom resources (CR).

Supported secret types are: 

*  ImagePull Secrets
*  TLS Secrets
*  SSH Auth Secrets 
*  Opaque Secrets
*  Opaque secrets based on a template (Jinja2)

#### Sample Custom Resource

An example custom resource(CR) definition

```yaml
apiVersion: secretssync.redhat.com/v1alpha1
kind: Hvaultsecretssync
metadata:
  name: hvaultsecretssync-sample
  namespace: suman-hvault-01
spec:
  configmap: 'vault-secrets-sync-agent'
  serviceaccount: 'internal-app'
```


Content of configmap: `vault-secrets-sync-agent`

```yaml
---
kube-secrets:
  - vault-secret-path: v1/secret/data/appsecrets
    kubernetes-secret: demo-appsecrets
    secret-type: opaque

  - vault-secret-path: v1/secret/data/nonprod-registry
    kubernetes-secret: demo-nonprod-registry
    secret-type: dockercfg

  - vault-secret-path: v1/secret/data/prod-registry
    kubernetes-secret: demo-nonprod-registry
    secret-type: dockercfg    

  - vault-secret-path: v1/secret/data/appcerts
    kubernetes-secret: demo-appcerts
    secret-type: tls

##########################
# Vault connection details
###########################

hashi-vault-url: http://10.24.0.1:8200/
vault-login-url-endpoint: v1/auth/suman-hvault-01/login
vault-secrets-refresh-seconds: 3000
vault-kube-auth-role-name: suman-hvault-01
```
