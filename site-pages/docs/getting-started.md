---
layout: default
title: Getting Started
tagline: Getting Started
---

## Getting Started
{:toc}

The following guide is meant to outline the bare-minium configuration required to stand up the Traefik, Cert-Manager, and ArgoCD modules. Please visit the module documentation which will contain detailed implementation and feature information.

---

### Prerequisites
{:toc}
- A cloud based or on-prem Kubernetes cluster.
- Sufficient access to deploy resources to the target cluster.
- A working knowledge of Terraform workspace setup and configuration application.
- A Provider for OAuth authentication
- Some DNS configuration

---

### Workspace Setup
{:toc}

These values might be stored in a `terraform.tfvars` file

```hcl-terraform

# The Let's Encrypt account key encoded in Base64
letsencrypt_secret_base64_key = "aGFoLCBuaWNlIHRyeSA6LVA="

# The OIDC provider parameters needed by ArgoCD to authenticate and authorize users.
argocd_oidc_name = "Auth0"
argocd_oidc_issuer = "https://homestead.us.auth0.com/"
argocd_oidc_client_id = "r20485SomHEXRp8jF0RTvymSbrrT622l"
argocd_oidc_client_secret = "dGhpcyBvbmUgaXNuJ3Qgc3VwcG9zZWQgdG8gYmUgYmFzZTY0IGVuY29kZWQ="
argocd_oidc_requested_scopes = [
  "openid"
  "profile"
  "email"
  "https://turnbros.app/claims/groups"
]
argocd_oidc_requested_id_token_claims = {
  "groups": {
    "essential": true
  }
}
```

### Configure Traefik
{:toc}
```hcl-terraform
module "traefik" {
  source  = "project-octal/traefik/kubernetes"
  version = "0.0.1"
  
  image_tag                            = "2.4.8"
  namespace                            = "kube-traefik"
  log_level                            = "INFO"
  replicas                             = 2
  rolling_update_max_surge             = 1
  rolling_update_max_unavailable       = 1
  pod_termination_grace_period_seconds = 1
  service_type                         = "LoadBalancer"
  preferred_node_selector              = []
}
```

### Configure Cert-Manager
{:toc}
```hcl-terraform
module "cert_manager" {
  source  = "project-octal/cert-manager/kubernetes"
  version = "0.0.3"

  # If necessary multiple cluster issuers can be defined as { issuer-name => issuer-configuration }
  certificate_issuers = {
    letsencrypt = {
      name              = "letsencrypt-prod"
      server            = "https://acme-v02.api.letsencrypt.org/directory"
      email             = "dylanturn@gmail.com"
      secret_base64_key = var.letsencrypt_secret_base64_key
      default_issuer : true
      ingress_class = module.traefik.ingress_class
    }
  }
}
```

### Configure ArgoCD
{:toc}
```hcl-terraform
module "argocd" {
  source  = "project-octal/argocd/kubernetes"
  version = "0.0.4"

  argocd_url        = "argocd.turnbros.app"

  namespace              = "kube-argocd"
  argocd_server_replicas = 2
  argocd_repo_replicas   = 2

  enable_dex      = false
  enable_ha_redis = false

  cluster_cert_issuer = module.cert_manager.cert_issuer
  ingress_class       = module.traefik.ingress_class

  # OIDC Configuration Argo will use for authentication and authorization
  oidc_config = {
    name                      = var.argocd_oidc_name
    issuer                    = var.argocd_oidc_issuer
    client_id                 = var.argocd_oidc_client_id
    client_secret             = var.argocd_oidc_client_secret
    requested_scopes          = var.argocd_oidc_requested_scopes
    requested_id_token_claims = var.argocd_oidc_requested_id_token_claims
  }
}
```