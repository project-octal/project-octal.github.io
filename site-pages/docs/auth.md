---
layout: default
title: Auth
is_component: true
tagline: Auth
---

<!-- {% raw %} -->
<style>
   .auth-image {
      max-height: 20em;
      -webkit-box-shadow: 5px 5px 8px -2px #999999; 
      box-shadow: 5px 5px 8px -2px #999999;
      margin: 1em;
   }
</style>
<!-- {% endraw %}) -->

## Configuring OpenID Authentication

The purpose of this guide is to demonstrate the process of configuring an OpenID Application as an authentication and authorization source.

More information reguarding how OpenID Connect works can be found here:

- https://openid.net/connect/faq/
- https://auth0.com/docs/protocols/openid-connect-protocol
- https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc

## Prerequisites

- Either Keycloak has been deployed or an existing Auth0/Okta account exist.
  - Auth0 Signup: https://auth0.com/signup
  - Okta Signup: https://developer.okta.com/signup/
  - Getting started with Keycloak: https://www.keycloak.org/docs/latest/getting_started/
- A value needs generated for `openid-secret`. This value will be used to encrypt the OpenID Connect session token.
  - MacOS/Linux: `openssl rand -base64 25`
  - Windows: https://activedirectoryfaq.com/2017/08/creating-individual-random-passwords/

## Auth0 OpenID Connect Application Setup

### Creating The Application

1. Navigate to the Application management section of your Auth0 dashboard at `https://manage.auth0.com/dashboard/us/{{auth0_account_name}}/applications`
2. Click the **_Create Application_** button in the top right of the page.
3. Either provide a name for this application or use the default, then select **_Regular Web Application_** as the application type and click the blue **_Create_** button.  
<img class="auth-image"  src="/assets/images/openid-connect/auth0/create-application.png"/>


### Gather The Client ID

1. Make note of the `Client ID` value. This value will be used in the `openid-client-id` configuration variable.  
   <img class="auth-image"  src="/assets/images/openid-connect/auth0/application-client-id.png" />


### Update the application URLs

1. Update the **_Allowed Callback URL_** and **_Allowed Logout URLs_** fields so that they point to the correct endpoint.  
   <img class="auth-image"  src="/assets/images/openid-connect/auth0/update-application.png" />


### Example Auth0 Configuration

---

**NOTE:** The `openid-issuer-base-url` should be set to the value of the **_Domain_** field of the Okta application.

---

```yml
bind-addr: 127.0.0.1:8080
cert: false

auth: openid
openid-issuer-base-url: "https://example.auth0.com"
openid-client-id: "yg3IIz2alcikRqjqOsnRJABn8Uthfrtv"
openid-base-url: "http://localhost:8080"
openid-secret: "{prerequisites_openid-secret_value}"
```

## Okta OpenID Connect Application Setup

### Creating The Application

1. Navigate to the **_Add Application_** page of your Okta dashboard at `https://{{okta_account_name}}-admin.okta.com/admin/apps/add-app`
2. Click the **_Create New App_** button located in the upper right of the page.
3. Set the **_Platform_** to **_Web_** and **_Sign on method_** to **_OpenID Connect_**. Then click **_Create_**  
   <img class="auth-image"  src="/assets/images/openid-connect/okta/create-application.png" />

### Update The Application

1. Update the **_Application name_** field with the desired name for this application.
2. Update the **_Login redirect URIs_** and **_Logout redirect URIs (Optional)_** fields so that they point to the correct endpoint then click **_Save_**  
   <img class="auth-image"  src="/assets/images/openid-connect/okta/update-application.png"/>
3. To update the **_Allowed grant types_** start by navigating to the **_General Settings_** section and clicking the **_Edit_** link in the top right corner of the section card.
4. Once in the edit view, locate the **_Allowed grant types_** checkbox list and make sure the check the boxs for **_Implicit (Hybrid)_** and **_Allow ID Token with implicit grant type_** are checked, then scroll to the bottom of the page and click **_Save_**.  
   <img class="auth-image"  src="/assets/images/openid-connect/okta/update-application-grants.png"/>
5. Lastly, ensure a user is assigned to this application by navigating to the **_Assignments_** tab, clicking on **_Assign_**, then selecting **_Assign to People_**.  
   <img class="auth-image"  src="/assets/images/openid-connect/okta/update-application-assignments.png"/>

### Gather The Client ID

1. Make note of the `Client ID` value. This value will be used in the `openid-client-id` configuration variable.  
   <img class="auth-image"  src="/assets/images/openid-connect/okta/application-client-id.png" />

### Example Configuration

---

**NOTE:** The `openid-issuer-base-url` should be set to the value of the **_Okta domain_** field of the Okta application.

---

```yml
bind-addr: 127.0.0.1:8080
cert: false

auth: openid
openid-issuer-base-url: "https://example.okta.com"
openid-client-id: "0oal8azjyekLwJXan5d6"
openid-base-url: "http://localhost:8080"
openid-secret: "{prerequisites_openid-secret_value}"
```

## Keycloak OpenID Connect Client Setup

### Creating The Client

1. Once logged into the Keycloak instance, navigate to the **_Clients_** page by clicking the **_Clients_** link in the navigation menu on the left side of the page.
2. Begin the client creation process by clicking the **_Create_** button located in the top right corner of the clint list.
3. Fill out the client creation fields then click **_Save_** - **_Client ID_**: This is the value that will later be populated in `openid-client-id`. This value is entirely up the the user or process creating the client. - **_Client Protocol_**: This value should be set to **_openid-connect_** - **_Root URL_**: This field should be populated with the base url of the service it authenticates.  
   <img class="auth-image"  src="/assets/images/openid-connect/keycloak/create-client.png" />

### Update The Client Name

1. Once the **_Save_** button in the **_Add Client_** window has been clicked, the client will be created and the page will be redirected to the client settings view. From inside that view proceed to name the newly create client by populating the **_Name_** field.
2. Enable implicit flow by changing **_Implicit Flow Enabled_** from **_OFF_** to **_ON_**.
3. Scroll to the bottom of the page and click **_Save_**.  
   <img class="auth-image"  src="/assets/images/openid-connect/keycloak/update-client.png" />

### Example Configuration

---

**NOTE:** The `openid-issuer-base-url` will depend on the Keycloak realm. In the example below the realm is called `master`.

**NOTE:** The `openid-client-id` value should be set to the value given to the **_Client ID_** when the client was first created.

---

```hcl-terraform
# OIDC Configuration Argo will use for authentication and authorization
oidc_config = {
   name                      = var.argocd_oidc_name
   issuer                    = "https://keycloak.local/auth/realms/master/.well-known/openid-configuration"
   client_id                 = "example-for-argocd"
   client_secret             = "{prerequisites_openid-secret_value}"
   requested_scopes          = var.argocd_oidc_requested_scopes
   requested_id_token_claims = var.argocd_oidc_requested_id_token_claims
}
```
