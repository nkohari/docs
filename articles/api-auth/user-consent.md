---
title: User consent and third-party clients
---

# User consent and third-party clients

The [OIDC-conformant authentication pipeline](/api-auth/tutorials/adoption) supports defining [resource servers (i.e. APIs) as entities separate from clients](/api-auth/tutorials/adoption/api-tokens).
This lets you decouple APIs from the applications that consume them, and also lets you define third-party clients that you might not control or even fully trust.

## Types of clients

All Auth0 clients are either first-party or third-party.

**First-party** clients are those controlled by the same organization or person that owns the Auth0 domain.
For example, suppose you wanted to access the Contoso API; in this case, there would likely be a first-party client used for logging in at contoso.com.

**Third-party** clients are controlled by different people or organizations who most likely should not have administrative access to your Auth0 domain.
They enable external parties or partners to access protected resources at your API in a secure way.
A practical application of third-party clients is the creation of "developer centers", which allow users to obtain credentials in order to integrate their applications with your API.
Similar functionality is provided by well-known APIs such as Facebook, Twitter, GitHub, and many others.

## Creating a third-party client

All clients created from the [management dashboard](${manage_url}/#/clients) are assumed to be first-party by default.

At the time of writing, third-party clients cannot be created from the management dashboard.
They must be created through the management API, by setting `is_first_party: false`.

All clients created through [Dynamic Client Registration](/api-auth/dynamic-client-registration) will be third-party.

## Consent dialog

If a user is authenticating through a third-party client and is requesting authorization to access the user's information or perform some action at an API on their behalf, they will see a consent dialog.
For example:

<table>
    <tr>
        <td>
<pre><code>GET /authorize?
client_id=some_third_party_client
&redirect_uri=https://fabrikam.com/contoso_social
&response_type=token id_token
&<em>scope=openid profile email read:posts write:posts</em>
&<em>audience=https://social.contoso.com</em>
&nonce=...
&state=...
</code></pre>
        </td>
        <td>
        <img alt="Auth0 consent dialog - Fabrikam Client for Contoso is requesting access to your account" src="/media/articles/hosted-pages/consent-dialog.png">
        </td>
    </tr>
</table>

If the user chooses to allow the application, this will create a user grant which represents this user's consent to this combination of client, resource server and scopes.
The client application will then receive a successful authentication response from Auth0 as usual.

## Handling rejected permissions

If a user decides to reject consent to the application, they will be redirected to the `redirect_uri` specified in the request with an `access_denied` error:

```
HTTP/1.1 302 Found
Location: https://fabrikam.com/contoso_social#
    error=access_denied
    &state=...
```

## Skipping consent for first-party clients

Only first-party clients can skip the consent dialog, assuming the resource server they are trying to access on behalf of the user has the "Allow Skipping User Consent" option enabled.

Since third-party clients are assumed to be untrusted, they are not able to skip consent dialogs.

## Password-based flows

When performing a [Resource Owner Password Credentials exchange](/api-auth/grant/password), there is no consent dialog involved.
During a password exchange, the user provides their password to the client directly, which is equivalent to granting the client full access to the user's account.

### Forcing users to provide consent

When redirecting to /authorize, the `prompt=consent` parameter will force users to provide consent, even if they have an existing user grant for that client and requested scopes.

### Customizing the consent dialog

As of today the consent dialog UI cannot be customized or set to a custom domain.
We plan to implement this in future releases.
