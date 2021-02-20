# hvault-ocp-secrets-sync-operator
An operator to deploy [hvault-ocp-secrets-sync](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) in a namespace to create/update the OpenShift4 secrets from Hashi Vault.


This operator provides the functionality of creating :

* A namespace `hvault-secrets-sync-operator-system` for operator deployment 
* A cluster scoped custom resource deployment (CRD) named `hvaultsecretssync`
* A deployment that makes sure a Pod is running that contains the controller part of the operator
* A container image of the operator code
* A Controller code that watches for custom resource of type `hvaultsecretssync`in a cluster and deploys [hvault-ocp-secrets-sync](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) deployment



## How does it work?

Once this operator is deployed, it will be watching for the custom resources (CR) in the cluster and when a custom resource is created in a namespace, operator will deploy [hvault-ocp-secrets-sync agent](https://github.com/sumantripuraneni/hvault-ocp-secrets-sync) in a correspnding namespace. A `hvault-ocp-secrets-sync` agent based on configmap will create secrets.

Supported secret types are: 

*  ImagePull Secrets
*  TLS Secrets
*  SSH Auth Secrets 
*  Opaque Secrets
*  Opaque secrets based on a template (Jinja2)
