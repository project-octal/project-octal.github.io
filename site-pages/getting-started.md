---
layout: default
title: Project Octal
---

## Prerequisites
- A cloud based or on-prem Kubernetes cluster.
- Sufficient access to deploy resources to the target cluster.
- A working knowledge of Terraform workspace setup and configuration application.

---

## Getting Started

### Secrets stored in a workspaces `terraform.tfvars` file
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

### Contents of the workspaces `project-octal.tf` file
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

module "argocd" {
  source  = "project-octal/argocd/kubernetes"
  version = "0.0.4"

  argocd_url        = "argocd.turnbros.app"
  argocd_image_tag  = "v2.0.2"
  haproxy_image_tag = "2.0.4"
  redis_image_tag   = "6.2.1-alpine"

  namespace              = "kube-argocd"
  argocd_server_replicas = 2
  argocd_repo_replicas   = 2

  enable_dex      = false
  enable_ha_redis = false

  cluster_cert_issuer = module.cert_manager.cert_issuer
  ingress_class       = module.traefik.ingress_class

  # ArgoCD Server requests and limits.
  argocd_server_requests = {
    cpu    = "300m"
    memory = "256Mi"
  }
  argocd_server_limits = {
    cpu    = "600m"
    memory = "512Mi"
  }

  # ArgoCD Repo Server requests, limits, and other general configuration
  argocd_repo_requests = {
    cpu    = "300m"
    memory = "256Mi"
  }
  argocd_repo_limits = {
    cpu    = "600m"
    memory = "512Mi"
  }
  repo_server_exec_timeout = "300"
  argocd_repositories = [
    {
      name = "helm-public"
      type = "helm"
      url  = "https://charts.helm.sh/stable"
    }
  ]

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