# Auth0 for Chrome Extensions

This package allows you to use Auth0 within a Chrome extension.

## Overview

This package provides a generic `PKCEClient.js` file which allows you to use the [Proof Key for Code Exchange](https://tools.ietf.org/html/rfc7636) spec, which is recommended for native applications.

With this package, you can set up your Chrome extension to use Auth0's hosted [Lock](https://auth0.com/lock) widget. It uses the `launchWebAuthFlow` from Chrome's identity API to retrieve tokens from Auth0.

## Integration

### Getting Started

If you haven't already done so, [sign up](https://auth0.com/signup) for your free Auth0 account and create an application in the dashboard. Find the **domain** and **client ID** from your app settings, as these will be required to integrate Auth0 in your Chrome extension.

When loading your application as an unpacked extension, a unique ID will be generated for it. You must whitelist your callback URL (the URL that Auth0 will return to once authentication is complete) and the allowed origin URL.

In the **Allowed Callback URLs** section, whitelist your callback URL.

```bash
https://<YOUR_APP_ID>.chromiumapp.org/auth0
```

In the **Allowed Origins** section, whitelist your chrome extension as an origin.

```bash
chrome-extension://<YOUR_APP_ID>
```

### Installation

Install the `auth0-chrome` package with npm.

```bash
npm install auth0-chrome
```

The `dist` folder contains a webpack bundle, including a minified version.

Configure your `manifest.json` file to run the `auth0chrome` script, along with an `env.js` and `main.js` script for your project. The `default_popup` should be set to an HTML file containing the content you would like to display.

```js
{
  ...
  "browser_action": {
    "default_title": "Auth0",
    "default_popup": "src/browser_action/browser_action.html"
  },
  "background": {
    "scripts": ["./env.js", "node_modules/auth0-chrome/dist/auth0chrome.min.js", "src/main.js"],
    "persistent": false
  },
  "permissions": [
    "identity",
    "notifications"
  ]
}
```

Add your Auth0 credentials in the `env.js` file.

```js
window.env = {
  AUTH0_DOMAIN: 'YOUR_AUTH0_DOMAIN',
  AUTH0_CLIENT_ID: 'YOUR_AUTH0_CLIENT_ID',
};
```

### Login

Somewhere in your browser action, create a **Log In** button and when it is clicked, emit an event that can be picked up to trigger the authentication flow. For example, listen for click events with jQuery and emit a message called `authenticate` with `chrome.runtime.sendMessage`.

```js
// ...
  $('.login-button').addEventListener('click', () => {
    $('.default').classList.add('hidden');
    $('.loading').classList.remove('hidden');
    chrome.runtime.sendMessage({
      type: "authenticate"
    });
  });
// ...
```

Your `main.js` file is where you should add the listener for the `authenticate` event. This is where you can instantiate `Auth0Chrome` and call the `authenticate` method to start the flow and save the authentication result when it comes back.

```js
// src/main.js

chrome.runtime.onMessage.addListener(function (event) {
  if (event.type === 'authenticate') {

    // scope
    //  - openid if you want an id_token returned
    //  - offline_access if you want a refresh_token returned
    // device
    //  - required if requesting the offline_access scope.
    let options = {
      scope: 'openid offline_access',
      device: 'chrome-extension'
    };

    new Auth0Chrome(env.AUTH0_DOMAIN, env.AUTH0_CLIENT_ID)
      .authenticate(options)
      .then(function (authResult) {
        localStorage.authResult = JSON.stringify(authResult);
        chrome.notifications.create({
          type: 'basic',
          iconUrl: 'icons/icon128.png',
          title: 'Login Successful',
          message: 'You can use the app now'
        });
      }).catch(function (err) {
      chrome.notifications.create({
        type: 'basic',
        title: 'Login Failed',
        message: err.message,
        iconUrl: 'icons/icon128.png'
      });
    });
  }
});
```

Auth0's hosted Lock widget will be displayed in a new window.

![auth0 lock](https://cdn.auth0.com/blog/auth0-chrome-lock.png)

## Using the Library


### `Auth0CLient(domain, clientId)`
The Library exposes Auth0Client which extends a generic PKCEClient. 

- `domain` : Your Auth0 Domain, to create one please visit https://auth0.com/
- `clientId`: The clientId for the chrome client, to create one 
   - Please visit https://manage.auth0.com/#/clients and click on  `+ Create Client`.
   - Select Native as Client Type.
   - In the Allowed Callback URLs Section please add `https://<yourchromeappid>.chromiumapps.org/auth0` as an allowed callback url
   - In the Allowed Origins section please add `chrome-extension://<yourchromeappid>`
   
### `Promise <Object> Auth0Client#authenticate(options, interactive)`

This will call the Authentication API, and will render the login ui should userinteraction is required. Upon completion this method will resolve an object which will contain the requested token and meta information related to the authentication process.

- `options` : options is an object which accepts all the parameters valid for [Auth0's Authentication API](https://auth0.com/docs/api/authentication/) except for `redirect_uri`, `response_type`, `code_challenge` & `code_challenge_method` as these are controlld by the library

- interactive: Interactive is a boolean, if set to false for advanced use-cases will result into chrome throwing an error if user-interaction is required during login. 


The `access_token` returned at the end of the authentication flow can then be used to make authenticated calls to your API. For more information on using access tokens, see the [full documentation](https://auth0.com/docs/api-auth).

## Contributing

Pull requests are welcome!

## Development

Install the dev dependencies.

```bash
npm install
```

When changes are made, run `npm run build` to produce new files for the `dist` folder.

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free Auth0 account

1. Go to [Auth0](https://auth0.com/signup) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE.txt) file for more info.
