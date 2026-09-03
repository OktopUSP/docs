---
description: >-
  Let your users sign in to Oktopus through your existing identity provider
  (Keycloak, Auth0, Okta, Azure AD, and other OIDC-compliant providers) instead
  of managing separate Oktopus passwords.
cover: >-
  https://images.unsplash.com/photo-1633265486064-086b219458ec?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw1fHxTZWN1cml0eXxlbnwwfHx8fDE3ODg0NzcxNTF8MA&ixlib=rb-4.1.0&q=85
coverY: 0
layout:
  width: default
  cover:
    visible: true
    size: full
    mask: none
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
---

# Single Sign-On (SSO)

Single Sign-On lets every user on your Oktopus deployment authenticate through a single, external OpenID Connect (OIDC) identity provider — Keycloak, Auth0, Okta, Azure AD, or any other OIDC-compliant provider — instead of a separate Oktopus password. It's configured **once, platform-wide**: there's a single identity provider for the whole deployment, shared by every tenant, which fits an on-prem deployment where you already run one central identity provider for all your users. SAML is not supported — OIDC only.

## How It Works

{% stepper %}
{% step %}
#### Discovery

The login page asks Oktopus, on load, whether SSO is turned on and whether it's mandatory. This is what decides whether users see a "Continue with SSO" button, and whether the password fields are shown at all.
{% endstep %}

{% step %}
#### Redirect to Your Identity Provider

Clicking "Continue with SSO" sends the browser to your identity provider's login page. Oktopus generates a short-lived, signed request tying that specific browser session to the login attempt, so the flow can't be hijacked or replayed by another site.
{% endstep %}

{% step %}
#### Callback & Verification

After the user authenticates with your identity provider, it redirects back to Oktopus with an authorization code. Oktopus exchanges that code for tokens and cryptographically verifies the identity token — issuer, audience, signature, and replay protection — before trusting anything in it. A login is only accepted if the identity provider reports the user's email as **verified**.
{% endstep %}

{% step %}
#### Just-in-Time Provisioning

If the email already has an Oktopus account, that account signs in as-is. If it doesn't, Oktopus creates one automatically and adds it to the tenant and role you've configured as the SSO default — see [Configuring SSO](sso.md#configuring-sso) below.
{% endstep %}

{% step %}
#### Session Handoff

The user lands back in Oktopus fully signed in, with the exact same kind of session a password login produces — from this point on, an SSO session is indistinguishable from a password session anywhere else in the product.
{% endstep %}

{% step %}
#### Logout

Signing out of Oktopus also ends the session at your identity provider (RP-initiated logout), so a shared or public machine doesn't stay silently signed in after someone logs out — the next "Continue with SSO" click requires authenticating again, rather than silently reusing a lingering session.
{% endstep %}
{% endstepper %}

## Configuring the Environment

Three environment variables must be set on the controller for SSO to be available at all:

| Variable             | Purpose                                                                                                                                                                                                                                                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `PUBLIC_API_URL`     | The externally reachable base URL of your Oktopus controller (e.g. `https://api.yourdomain.com`). Used to build the callback URL that must be registered as an allowed redirect URI on your identity provider's client.                                                                                                                    |
| `FRONTEND_URL`       | The base URL of your Oktopus frontend (e.g. `https://app.yourdomain.com`). Used to build the post-login and post-logout redirects.                                                                                                                                                                                                         |
| `SSO_ENCRYPTION_KEY` | A long, random secret used to encrypt your identity provider's client secret at rest. **Generate this once and keep it stable** — rotating it invalidates the stored client secret, requiring you to re-enter it in the SSO settings. Losing or changing it accidentally is the most common cause of SSO suddenly failing after a restart. |

{% hint style="warning" %}
`SSO_ENCRYPTION_KEY` must stay the same across restarts. Generate it once (e.g. `openssl rand -base64 32`) and store it in your deployment's environment configuration — don't regenerate it on every restart.
{% endhint %}

## Configuring SSO

Once the environment variables above are set, an admin of your platform's **Root tenant** configures SSO from **Settings → Single Sign-On** — this page is only visible to Root-tenant admins, since the setting applies platform-wide.

| Field              | Notes                                                                                                                                                                                                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Enabled**        | Turns on the "Continue with SSO" button for every user on the platform.                                                                                                                                                                                                               |
| **Issuer URL**     | Your identity provider's OIDC issuer, e.g. `https://keycloak.example.com/realms/your-realm`. It must serve a standard `/.well-known/openid-configuration` discovery document — this is probed on save, so a misconfigured URL fails immediately instead of at the next login attempt. |
| **Client ID**      | The OIDC client/application ID registered on your identity provider.                                                                                                                                                                                                                  |
| **Client Secret**  | Encrypted at rest with `SSO_ENCRYPTION_KEY`. Leave it blank on a later update to keep the currently stored secret.                                                                                                                                                                    |
| **Default Tenant** | Required. Every brand-new SSO user is automatically added to this tenant — picked from a dropdown of every tenant on the platform.                                                                                                                                                    |
| **Default Role**   | Required. The role assigned to every new SSO user within the Default Tenant, picked from that tenant's own roles.                                                                                                                                                                     |
| **Enforced**       | Hides the password login fields on the login page by default, nudging everyone toward SSO.                                                                                                                                                                                            |

{% hint style="info" %}
**Enforced** hides the password fields by default, but native Oktopus users created through the software itself, can still switch back to password login from the login page if needed.
{% endhint %}
