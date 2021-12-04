---
title: Azure Static Web Apps
weight: 20
description: >
  How Azure Static Web Apps is used to host the Bizzkit Platform documentation, and what it provides.
---

The Bizzkit Platform documentation website is hosted on Azure Static Web Apps.
This technology enables us to host a static website on Azure, as well as gain access to a number of features that are not available in Hugo itself:

- Route rules and custom fallback pages for various response codes.
- Authentication and Authorization with Azure Active Directory, and others.
- Geographical distribution of the static content on the website.
- TLS certificates.
- Custom domains.

Azure Static Web Apps even comes with [support for Hugo](https://docs.microsoft.com/en-us/azure/static-web-apps/publish-hugo) out of the box.
Furthermore, it enables us to directly integrate the website with APIs backed by Azure Functions, should the need ever arise.

## Route rules

Certain pages on the Bizzkit Platform documentation website should not be directly accessed by users.
This is not because those pages are secret, but rather because navigating to them does not make a lot of sense.

One such example is `https://docs.bizzk.it/docs`, which is the root directory of all versions of the Bizzkit documentation, which should be accessed via the version picker in the navigation bar instead.
To avoid presenting the meaningless docs page to users, we leverage [route rules](https://docs.microsoft.com/en-us/azure/static-web-apps/configuration#routes) to redirect users to the root of the _latest_ documentation:

```json
{
  "routes": [
    {
      "route": "/docs",
      "rewrite": "/docs/latest"
    }
  ]
}
```

## Response code overrides

If users attempt to access non-existent pages on the documentation website, which could occur when switching to a language that is not availbale for an archived version, we leverage [response code overrides](https://docs.microsoft.com/en-us/azure/static-web-apps/configuration#response-overrides) to present a helpful 404 fallback page to the user:

```json
{
  "responseOverrides": [
    "404": {
      "rewrite": "/404.html"
    }
  ]
}
```

This page includes a message to the user about the requested documentation not being available in the requested language for the specified version, as well as a link to navigate back to the landing page.

## Authentication

User authentication is not handled by the Bizzkit Platform documentation website itself, but is [delegated to Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/authentication-authorization?tabs=invitations).
The deployment automatically makes available a series of paths to which users can navigate to authenticate with their provider of choice:

- `/.auth/login/aad`
- `/.auth/login/github`
- `/.auth/login/twitter`

Should we ever need to, it is also possible to integrate with a [custom authentication provider](https://docs.microsoft.com/en-us/azure/static-web-apps/authentication-custom?tabs=aad).

For the time being, we wish to integrate only with Azure Active Directory (AAD), and for this reason we disable access to the other paths by use of route rules:

```json
{
  "routes": [
    {
      "route": "/.auth/login/github",
      "statusCode": 404
    },
    {
      "route": "/.auth/login/twitter",
      "statusCode": 404
    }
  ]
}
```

Furthermore, we rename the path for AAD authentication to `/login`, to reduce the number of dependencies on the authentication provider throughout the configuration:

```json
{
  "routes": [
    {
      "route": "/login",
      "rewrite": "/.auth/login/aad"
    }
  ]
}
```

## Authorization

The Bizzkit Platform documentation should be accessible only by authenticated users.
When a user visits the site without an active session, they belong to the `anonymous` role.
Using [secured routes](https://docs.microsoft.com/en-us/azure/static-web-apps/configuration#securing-routes-with-roles), we ensure that only user with the `authenticated` role may access the website.
This rule must be defined after the route rules for login and logout, to avoid restricting access to those paths:

```json
{
  "routes": [
    {
      "route": "*",
      "allowedRoles": ["authenticated"]
    }
  ]
}
```

When a user with the `anonymous` role attempts to access the page, they receive the 401 response code, for which an override rule ensure that they are redirected to the `/login` page of choice:

```json
{
  "responseOverrides": [
    "401": {
      "redirect": "/login",
      "statusCode": 302
    }
  ]
}
```
