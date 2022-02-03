# secrets-sync-operator
An operator to deploy [secrets-syn-agent](https://github.com/sumantripuraneni/secrets-sync-agent) in a namespace to create/update the OpenShift4 secrets from Hashi Vault.

This operator provides the functionality of creating :

* A namespace `hvault-secrets-sync-operator-system` for operator deployment 
* A cluster scoped custom resource deployment (CRD) named `hvaultsecretssync`
* A deployment that makes sure a Pod is running that contains the controller part of the operator
* A container image of the operator code
* A Controller code that watches for custom resource of type `hvaultsecretssync`in a cluster and deploys [hvault-ocp-secrets-sync](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) deployment



## How does it work?

### Design 

As a pre-requirement, two configmaps needs to be created in a namespace and a serviceaccount to authenticate with Hashi Vault using KubeAuth mechanism.


Once this operator is deployed, it will be watching for the custom resources (CR) in the cluster scope (all namespaces) and when a custom resource is created by an user in a namespace, operator will deploy [secrets-sync-agent](https://github.com/sumantripuraneni/secrets-sync-agent) in a corresponding namespace, which will create secrets as per the configmaps

![Alt text](images/secrets-sync-agnet-design1.jpeg?raw=true "Create secret in same namespace") 


Supported secret types are: 

*  ImagePull Secrets
*  TLS Secrets
*  SSH Auth Secrets 
*  Opaque Secrets
*  Opaque secrets based on a template (Jinja2)


#### Examples of custom resource(CR)


An example custom resource(CR) definition

```yaml
apiVersion: secretssync.redhat.com/v1alpha1                               ## API version 
kind: Hvaultsecretssync                                                   ## API kind 
metadata:
  name: hvaultsecretssync-sample-testing                                  ## Name of custom resource 
  namespace: suman-hvault-01                                              ## Namespace to create custom resource 
spec:
  serviceaccount: internal-app                                            ## Instructs service account to use to run "secrets-sync-agent". Default to "default" SA
  loglevel: DEBUG                                                         ## Instructs level of logging. Defaults to "INFO"
  secretsRefreshSeconds: 300                                              ## Instructs how frequent to check and refresh secrets. Defaults to "3600" seconds
  connectionInfo:                                                         ## vault connection details
    VAULT_ADDR: http://52.116.136.244:8200/                               ## 
    VAULT_LOGIN_ENDPOINT: v1/auth/kubernetes/login
    VAULT_ROLE: suman-test
    KUBE_SECRETS_MGMT_CREDS_PATH: v1/secrets/data/ocp/accounts/someaccount  ##vault secret path of user which wil be used to generate OpenShift session token to create/update/path/get/lust secrets.
    #VAULT_NAMESPACE: projects   For enterprise Hashi vault 
  secretsInfo:                                                            ## Information on secrets retrieval 
    KUBE_SECRETS:
      - VAULT_SECRET_PATH: v1/secret/data/appsecrets
        KUBERNETES_SECRET: demo-appsecrets
        SECRET_TYPE: opaque
      - VAULT_SECRET_PATH: v1/secret/data/certs
        KUBERNETES_SECRET: demo-appcerts
        SECRET_TYPE: tls
      - VAULT_SECRET_PATH: v1/secret/data/auth
        KUBERNETES_SECRET: demo-ssh-auth
        SECRET_TYPE: ssh-auth      
