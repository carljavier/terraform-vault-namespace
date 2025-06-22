# terraform-vault-namespace
Module to rinse and repeat creation of HashiCorp Vault Namespaces and enabling default secret engines. Used by my setup in terraform-vault-management repo for my own use cases. 


# Pre-req

- HashiCorp Enterprise Vault

# Background and use case

I have often had to manage HashiCorp Vault in the past and deploying various patterns to bootstrap setup cloud landing zones which usually entail setting up some sort of isolated or fenced off area in a cloud for an application to run its workload. Part of setting this up also means creating a space for the App/Project code in either Github or Azure DevOps. This example will concentrate on mapping an Azure DevOps project to a HashiCorp Vault namespace. 

The idea is to the App/Project code also is given a HashiCorp namespace to manage it's own secrets. 


# Namespace guidelines

- Only one level of namespace should be created. If an App/Project requires further child namespaces, this module setp will not cater for that. 
- Two roles for humans will be setup per namespace (Secrets User and Secrets Officer). These names match what Azure KeyVault definitions as well. 


# Namespace defaults:

The following is what each App/Project code will be provided by default and what inputs are required to setup the Namespace.



## Authenticated from Azure DevOps Pipeline

- A workload identity federated "identity" eg an app registration or managed identity from Azure DevOps/Entra is the auth method for any ADO pipeline to auth to vault for a vault batch token. 
- Oauth/OIDC should be used so passwordless config is setup. This does require Vault's OIDC well known common/ID/ Issuer URL needs  to be publicly available to Entra. 
- If the above is not possible, then client-secret with the root-cred rotation is to be scheduled/setup. 


## Authentications from Machine Identities (Eg Azure VMs)

- Any workload running on a VM or a VM Scale Set (VMSS) should be able to auth to Vault and receive a token to then gain access
- An SPN or user assigned identity is to be used and passed to this module so it is allowed to auth via to the Vault namespace
- Further restrictions such as subscription id, resource group id and/or VM/s network source ip range/s can also be bounded to further lock which machine can retrieve a vault token.


## Authenticated from user identity (eg Entra)

- App/Project users working on the app/project will gain access (via UI or CLI) to hashiCorp vault namespace by Entra groups/roles. 
- App/Project users will have the ability to manage and use the default secret engines. 
- App/Project will NOT have the ability to setup new auth/secret engines as this will be managed by Vault admins



## Static Secret KV2

- One static secret store will be provided. 
- To be used for secrets that are required by the App/Pipelines run time function and cannot used dynamic secrets. Some examples are API Keys to vendor apps, service accounts, certificats etc. 


## Further secret engines

- Further secret engines to be enabled on a case by case basis not part of this module. 
- Vault admins to be contacted about managing and setting up further secret engines (eg Dynamic DB credentials or Cloud credentials for a sepcific suscription/resource group)