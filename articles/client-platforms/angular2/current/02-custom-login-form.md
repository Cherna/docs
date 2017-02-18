---
title: Custom Login Form
description: This tutorial demonstrates how to add a custom login form to an Angular 2+ application with Auth0
budicon: 448
---

<%= include('../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-angular-samples',
  path: '02-Custom-Login-Form',
  requirements: [
    'Angular 2+'
  ]
}) %>

The [previous step](/quickstart/spa/angular2/01-basic-login) demonstratd how to add authentication to your app using the Lock widget. While Lock provides an easy to use, fully-featured, and customizable login interface, you may want to build your own UI with a custom design. To do this, use the [auth0.js library](https://github.com/auth0/auth0.js).

## Install auth0.js

The auth0.js library can either be retrieved from Auth0's CDN or from npm.

**CDN Link**

```html
<!-- index.html  -->

<script src="https://cdn.auth0.com/js/auth0/8.2/auth0.min.js"></script>
```

**npm**

```bash
npm install --save auth0-js
```

## Create a Login Component

Create a component to house the template that should be used for your custom UI. In this example, the logic for all authentication transactions will be handled from the `Auth` service, which means the `LoginComponent` class only needs to inject that service.

```js
// src/app/login/login.component.ts

import { Component } from '@angular/core';
import { Auth } from './auth.service';

@Component({
  selector: 'login',
  templateUrl: 'app/login.component.html'
})
export class LoginComponent {
  constructor(private auth: Auth) {}
}
```

## Implement the Login Screen

Create a template with a `form` which allows users to pass in their email and password. This example will not make full use of Angular's form features, but rather will simply use template variables on the username and password inputs. The values from those inputs will be passed into the methods called on the Log In and Sign Up button's `click` events. You may also supply a button to trigger social authentication.

```html
<!-- app/login.component.html -->

<form>
  <div class="form-group">
    <label for="name">Username</label>
    <input
      type="text"
      class="form-control"
      #username
      placeholder="Enter your email">
  </div>
  <div class="form-group">
    <label for="name">Password</label>
    <input
      type="password"
      class="form-control"
      #password
      placeholder="Enter your password">
  </div>
  <button
    type="submit"
    class="btn btn-default"
    (click)="auth.login(username.value, password.value)">
      Log In
  </button>
  <button
    type="submit"
    class="btn btn-default"
    (click)="auth.signup(username.value, password.value)">
      Sign Up
  </button>
  <button
    type="button"
    class="btn btn-default btn-primary"
    (click)="auth.loginWithGoogle()">
      Log In with Google
  </button>
</form>
```

## Create an Authentication Service

All authentication transactions should be handled from an injectable service. The service requires methods named `login`, `signup`, and `loginWithGoogle` which all make calls to the appropriate auth0.js methods to handle those actions. These methods are called from the `login` template above.

The auth0.js methods for making authentication requests come from the `WebAuth` object. Create an instance of `auth0.WebAuth` and provide the domain, client ID, and callback URL for your client. A `responseType` of `token id_token` should also be specified.

The `login` and `signup` methods should take the username and password input supplied by the user and pass it to the appropriate auth0.js methods. In the case of `login`, these values are passed to the `client.login` method. Since `client.login` is an XHR-based transaction, the authentication result is handled in a callback and the user's access token and ID token are saved into local storage if the transaction is successful.

The `signup` method is a redirect-based flow and the authentication result is handled by the `handleAuthentication` method. This method looks for an access token and ID token in the URL hash when the user is redirected back to the application. If those tokens are found, they are saved into local storage and the user is redirected to the home route.

```js
// src/app/auth/auth.service.ts

import { Injectable } from '@angular/core';
import { Router } from '@angular/router';

// Avoid name not found warnings
declare var auth0: any;

@Injectable()
export class AuthService {

  // Configure Auth0
  auth0 = new auth0.WebAuth({
    domain: AUTH_CONFIG.domain,
    clientID: AUTH_CONFIG.clientID,
    redirectUri: AUTH_CONFIG.callbackURL,
    audience: `https://${AUTH_CONFIG.domain}/userinfo`,
    responseType: 'token id_token'
  });

  constructor(private router: Router) { }

  public login(username: string, password: string): void {
    this.auth0.client.login({
      realm: 'Username-Password-Authentication',
      username,
      password
    }, (err, authResult) => {
      if (err) {
        alert(`Error: ${err.description}`);
        return;
      }
      if (authResult && authResult.accessToken && authResult.idToken) {
        this.setSession(authResult);
      }
      this.router.navigate(['/home']);
    });
  }

  public signup(email, password): void {
    this.auth0.redirect.signupAndLogin({
      connection: 'Username-Password-Authentication',
      email,
      password,
    }, function (err) {
      if (err) {
        alert(`Error: ${err.description}`);
      }
    });
  }

  public loginWithGoogle(): void {
    this.auth0.authorize({
      connection: 'google-oauth2',
    }, function (err) {
      if (err) {
        alert(`Error: ${err.description}`);
      }
    });
  }

  public handleAuthentication(): void {
    this.auth0.parseHash((err, authResult) => {
      if (authResult && authResult.accessToken && authResult.idToken) {
        window.location.hash = '';
        this.setSession(authResult);
        this.router.navigate(['/home']);
      } else if (authResult && authResult.error) {
        alert(`Error: ${authResult.error}`);
      }
    });
  }

  private setSession(authResult): void {
    // Set the time that the access token will expire at
    let expiresAt = JSON.stringify(
      (authResult.expiresIn * 1000) + new Date().getTime()
    );
    localStorage.setItem('access_token', authResult.accessToken);
    localStorage.setItem('id_token', authResult.idToken);
    localStorage.setItem('expires_at', expiresAt);
  }

  public logout(): void {
    // Remove tokens and expiry time from localStorage
    localStorage.removeItem('access_token');
    localStorage.removeItem('id_token');
    localStorage.removeItem('expires_at');
    // Go back to the home route
    this.router.navigate(['/home']);
  }

  public isAuthenticated(): boolean {
    // Check whether the current time is past the 
    // access token's expiry time
    let expiresAt = JSON.parse(localStorage.getItem('expires_at'));
    return new Date().getTime() < expiresAt;
  }

}

```

The service has several other utility methods that are necessary to complete authentication transactions.

<%= include('../_includes/_custom_login_method_description') %>

The `handleAuthentication` method needs to be called in the application's root component.

```js
// src/app/app.component.ts

// ...
export class AppComponent {
  constructor(private auth: Auth) {
    this.auth.handleAuthentication();
  }
}
```