---
title: Calling an API
description: This tutorial demonstrates how to use angular2-jwt in Angular 2 applications to make authenticated API calls
budicon: 546
---

<%= include('../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-angular-samples',
  path: '05-Authorization',
  requirements: [
    'Angular 2+'
  ]
}) %>

The previous step covers how to protect the resources served by your API such that only users who have authenticated in your application are able to access them. Protecting your data resources in this way is sufficient if you want to restrict access in a catch-all fashion. However, by itself, it doesn't provide any means to make decisions about which resouces a _particular_ user should and should not have access to. **Authorization** in the context of this tutorial is about the way that these decisions about access for certain users is made and the mechanisms used to enforce them.

## Access Control in Single Page Apps

Distinguishing between different users and controlling what they do and do not have access to is typically known as access control. The way that access control is implemented depends on the needs of your application, but a common scenario is to introduce a set of groups. These groups might include something like administrators, paid users, and unpaid users, each with differing levels of access to data.

For single page applications which get their data from APIs, there are two major things that need to be considered when authorization and access control are introduced:

1. The particular data being requested from the API should only release data if the user making the request is authorized to access it
2. The user should only be able to access client-side routes if they have an appropriate access level to do so

