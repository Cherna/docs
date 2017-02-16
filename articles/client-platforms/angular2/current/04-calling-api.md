---
title: Calling an API
description: This tutorial demonstrates how to use angular2-jwt in Angular 2 applications to make authenticated API calls
budicon: 546
---

<%= include('../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-angular-samples',
  path: '08-Calling-Api',
  requirements: [
    'Angular 2+'
  ]
}) %>

A common need for any client-side application is to access resources from a data API. Some of these data resources will likely need to be protected such that only the user who is authenticated in the client-side app can access them. This can be achieved by protecting your API's endpoints with your Auth0 JSON Web Key Set (JWKS) and sending the user's `access token` as an `Authorization` header when calling the API.

## Create an API in Auth0

The **Getting Started** step of this tutorial set covers how to create a new client in your Auth0 dashboard. The ID assigned to this client and the domain for your Auth0 account were used when initializing the Lock widget and/or Auth0 in the previous steps.

In a similar way, you must create a record for a resource server, or **API**, in your Auth0 dashboard. The identifier for this API will now be included as the `audience` parameter when you initialize Lock and/or auth0.js.

Navigate to the **APIs** section and choose "Create API". Give your API a name and identifier, and then choose **Create**.