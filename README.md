<img src="https://devmounta.in/img/logowhiteblue.png" width="250" align="right">

CoderFriends
============

## Objective
Use a Node backend with Passport and Express to show a user's coder friends.

## Resources
* [passport-github2] (https://github.com/cfsghost/passport-github)
* [Github API Docs] (https://developer.github.com/v3/)
* [node-github] (https://github.com/mikedeboer/node-github)
* [passport-auth0] (https://www.npmjs.com/package/passport-auth0)

## Step 1: Create Skeleton of Angular App
To mix it up, let's create the file structure for the Angular app first.

* Create a `/public` folder
* Create the following files:
  * app.js
  * services/github-service.js
  * index.html
  * templates/home.html
  * templates/friend.html

Let's create routes for our app:

### /
The base route should display a "Login with Github" button that will redirect users to `/auth/github`. You may accomplish this by either 1) making a login.html templateUrl for this route, or 2) using an inline template in the route configuration (rather than a templateUrl).

### /home
The home route will display the current user's GitHub friends via the home.html template

### /friend/:github_username
This route will display a friend's information as well as what they're currently working on.

Create the server.js file and set it up to serve your static files.

## Step 2: Create the auth endpoints

* Install and require your dependencies
  * express
  * express-session
  * passport
  * passport-auth0
* [Create an Auth0 app ("client")](https://manage.auth0.com/#/) and then set up the Auth0 Strategy in your server.js with your associated `clientID` and `clientSecret`. Check the boxes to ask for permission to access the Github accounts our user is following. 
* Make sure you use the session, passport.initialize and passport.session middleware
* Set up your auth endpoints:

#### /auth/github
Use passport.authenticate with 'auth0' as the first parameter, then {connection: 'github'} as the second parameter. This will choose GitHub as the authentication service. 

Make sure to use 'auth/github/callback' as the callbackURL.

#### /auth/github/callback
Use passport.authenticate and upon successful auth, send the user to `/#/home`. 

## Step 3: Github following Endpoint
Let's link the Angular Github service to our server.js

#### GET `/api/github/followers`
In server.js, create the above endpoint and have it return the users who follow the currently logged in user. To do this, you will need to make an API call directly from your server.js file. You can one of two ways:
- Use an http request using the [request](https://www.npmjs.com/package/request#http-authentication) module. The url you will need to hit is 
```
https://api.github.com/user/followers
``` 
Make sure that you authenticate the request with the 'User-Agent' header as listed on the request docs. The value for 'User-Agent' should be the clientID received through auth0. 

- Or use the npm module [node-github](https://github.com/mikedeboer/node-github). The example on the page provides the needed information for your request.

Some hints:
* You'll want to make sure that whichever client that requests this endpoint is currently logged in. The best way to do this would be to write a middleware function that runs before the "get followers" logic so that you're sure that the current requesting user is logged in. Your middleware function could look like this:

```
var requireAuth = function(req, res, next) {
  if (!req.isAuthenticated()) {
    return res.status(403).end();
  }
  return next();
}
```

If the client gets a status of 403, it will know that it needs to redirect the user to the `/` page so the user can log in again. **Keep in mind, this will happen every time your server restarts.**

## Step 4: homeCtrl + Github Service
Now let's connect your Angular app to this setup.

* In GithubService, create a `getFollowing` method that returns the results from the API call we created in Step 3.
* Let's resolve the promise from `getFollowing` into a `friends` variable in the `/home` route before it loads.
* In your homeCtrl (create this file, or do an inline controller in the home route in `app.js`), let's throw friends into the scope and render them in the view (home.html).

## Step 5: NG un-authed auto-redirect
We need a way for Angular to detect an un-authed web request (403) so we can redirect them back to the login page. We can do that by injecting a service that acts as an interceptor in Angular's httpProvider. It works sort of like middleware in Node. Add this chunk of code to your `app.js` file.

```
  app.config(function($httpProvider) {
    $httpProvider.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
    $httpProvider.interceptors.push('myHttpInterceptor');
})

// register the interceptor as a service
  app.factory('myHttpInterceptor', function() {
    return {
        'responseError': function(rejection) {
            if (rejection.status == 403) {
              document.location = '/';
                return rejection;
            }
            return rejection;
        }
    };
});
```

## Step 5: Friend route
Make it so that when the user clicks on one of the selected friends, it loads in that user's latest activity.

#### GET /api/github/:username/activity
Create this endpoint in your server.js that grabs data for the given username. 
- If you are using the `request` module from Step 3, the url you will need to hit is:
```
https://api.github.com/users/<username>/events
```
This request does not need to be authenticated with any credentials. 

- Or, if you are using the `node-github` module from Step 3, you will need to use
```
github.activity.getEventsForUser
```

* Create a method in your Github service called `getFriendActivity` and make sure it's passed a username
* Have `eventData` be a resolved variable in the app's routing, then render each of the events in the `/friend/:github_username` route in `friend.html`.

## Contributions
If you see a problem or a typo, please fork, make the necessary changes, and create a pull request so we can review your changes and merge them into the master repo and branch.

## Copyright

© DevMountain LLC, 2015. Unauthorized use and/or duplication of this material without express and written permission from DevMountain, LLC is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to DevMountain with appropriate and specific direction to the original content.

<img src="https://devmounta.in/img/logowhiteblue.png" width="250">
