---
layout: default
title: Project Octal
---

# What is Project Octal?
Project Octal is a set of open source projects and configurations collated into a set of Terraform modules aimed at simplifying and standardizing the deployment and operation of Kubernetes clusters in enterprise environments.

---

<img class="center" style="width: 30em" src="./assets/images/octal-components.svg">

---

# Core Module Implementation Status

## Current Capability

At this time 3 of the 5 core modules have been implemented. The implemented modules: terraform-kubernetes-argocd, terraform-kubernetes-cert-manager, and terraform-kubernetes-traefik are available via the Terraform public registry.

More information on the implemented core modules can be found here:
- [terraform-kubernetes-argocd](/site-pages/octal-core/argocd.html)
- [terraform-kubernetes-cert-manager](/site-pages/octal-core/cert-manager.html)
- [terraform-kubernetes-traefik](/site-pages/octal-core/traefik.html)

<img class="center" style="width: 30em" src="./assets/images/project-octal-current-capability.png">

## Planned Capability

The work of implementing the remaining 2 core modules for Linkerd and Open Policy Agent is underway. Once the modules are complete they will be made available via the Terraform public registry.

More information on the planned core modules can be found here:
- [terraform-kubernetes-linkerd](/site-pages/octal-core/linkerd.html)
- [terraform-kubernetes-opa](/site-pages/octal-core/open-policy-agent.html)

<img class="center" style="width: 30em" src="./assets/images/project-octal-planned-capability.png">