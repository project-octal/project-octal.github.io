---
layout: default
---

# Overview
An opinionated suite of open source projects and configurations aimed at simplifying the deployment and management of enterprise Kubernetes clusters.

# OCTAL Components

| Name                   | Version       | Link         |
|:-----------------------|:--------------|:-------------|
| Open Policy Agent      | v0.23.2       | https://     |
| Cert-Manager           | v1.0.2        | https://     |
| Traefik                | v2.3.1        | https://     |
| ArgoCD                 | v1.7.7        | https://     |
| Linkerd2               | stable-2.7.1  | https://     |
| Argo Rollouts          | v0.9.1        | https://     |

# Getting Started

```hcl
module "ocal_services" {
  source                 = "github.com/project-octal/terraform-kubernetes-octal"
  
  # These values would come from the existing Kubernetes cluster.
  cluster_endpoint       = module.kubernetes_cluster.cluster_endpoint
  cluster_token          = module.kubernetes_cluster.cluster_token
  cluster_ca_certificate = module.kubernetes_cluster.cluster_ca_certificate

  # This is where you would define one or more certificate issuers.
  cert_manager = {
    certificate_issuers = {
      letsencrypt = {
        name              = "letsencrypt-prod"
        server            = "https://acme-v02.api.letsencrypt.org/directory"
        email             = "email@domain.com"
        default_issuer    = true
        secret_base64_key = var.letsencrypt_secret_base64_key
      }
    }
  }

  # Define the parameters for Traefik
  traefik = {
    namespace                            = "kube-traefik"
    log_level                            = "INFO"
    replicas                             = 1
    rolling_update_max_surge             = 1
    rolling_update_max_unavailable       = 1
    pod_termination_grace_period_seconds = 1
  }
  
  # Define the parameters for ArgoCD. 
  # NOTE: The HA setting has only been partially tested.
  argocd = {
    # TODO: Not this. Gotta find another way.
    url                            = "argocd.brava.turnbros.app"
    namespace                      = "kube-argocd"
    server_replicas                = 1
    repo_replicas                  = 1
    enable_dex                     = false
    enable_ha_redis                = false
    oidc_name                      = var.argocd_oidc_name
    oidc_issuer                    = var.argocd_oidc_issuer
    oidc_client_id                 = var.argocd_oidc_client_id
    oidc_client_secret             = var.argocd_oidc_client_secret
    oidc_requested_scopes          = var.argocd_oidc_requested_scopes
    oidc_requested_id_token_claims = var.argocd_oidc_requested_id_token_claims
  }
}
```