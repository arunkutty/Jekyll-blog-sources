---
layout: post
title:  Integrating loopback API Backend with an Angular application
date:   2016-02-20 17:53:00 +0530
categories: webapp
---

Couple of weeks back, as part of a personal project, I started reading up on loopback.io framework which sits tidily on top of ExpressJS.
 The framework provides users capability to create a REST API based backend on NodeJS by simply defining models for their data. Part of the interest,
 I should admit, was because Strongloop, the company that created loopback.io, was recently acquired by IBM.
 But it did deliver. For the relatively simple requirements that I had for my personal project, I didn't write a single line of code\!
 Think of loopback.io as an API-first-like-[Swagger](http://swagger.io/) way of development but specifically for NodeJS.
 Strongloop provides additional value to the enterprise by solutions to build, deploy and monitor APIs in production and some great support,
 but that's not what this post is about.

 In order to create the backend, I didn't have much to figure. All I did was follow the instructions on
 [this video.](https://vimeo.com/109596021) I may have had to refer loopback documentation a couple of times, but it was
 not hard at all. I also got the generated app talk to MongoDB pretty quick. The only part that I felt was not intuitive is where the
 persistence for the built-in models such as Users are defined. But that's also not the point of this post.

 The point of this post is to document the steps I used to create an [AngularJS](https://angularjs.org/) client and then get it
 speaking to the backend that I generated using loopback.io.

  So here goes...

 To begin with, I scaffolded an Angular app using Yeoman team's popular [Angular generator](https://github.com/yeoman/generator-angular#readme).

```shell
 npm install -g generator-angular
```
Once this completed, my app `ExpenseVoucher-WebClient` was generated by first creating a directory to hold the project contents and then
using the Angular generator which was installed just now.

```shell
mkdir my-new-project && cd $_
yo angular ExpenseVoucher-WebClient
```

My shiny new AngularJS client app was ready. After running an `npm install` followed by `bower install`, my app was ready to be served. But at this point
this app did pretty much nothing other than greeting with an enthusiastic *Allo\! Allo\!* from Yeoman. As I
was trying to integrate Loopback and AngularJS for the first time, I thought it was sufficient to create a simple view that lists out all of
a certain entity that was stored in the backend. (*Note:* I had used Strongloop's API Explorer to create stuff in the backend by this time.)
Based on this simple requirement, I used the Yeoman Angular generator once again and got the route entry for my *ExpenseVouchers* entity
added to the app along with controller and view files.

```shell
yo angular:route list-expense-vouchers
```

After hacking a bit on the view file to present *ExpenseVouchers*, I moved back to the backend project for integration.

The Loopback AngularJS SDK, which is installed along with Loopback, dynamically generates AngularJS Services that are compatible with ngResource.$resource.
It walks through the models in the backend project and creates a AngularJS factory for each model. The SDK provides 2 options for doing this,
one is based on `grunt` and the other a command-line tool `lb-ng`. I chose to use `lb-ng`. In my loopback generated project, I moved to the
`client` folder and ran the following command:

```shell
lb-ng ../server/server.js js/lb-services.js
```

This command, if all goes well as it did for me, would generate a `lb-services.js` file that contains all the AngularJS services that you need
to access the backend. I manually copied `lb-services.js` into the `app/scripts/services` path of my client app. The remaining steps were to be
done on the client app. So back to the client app...

So that it gets served to the client, I added the following line to the index.html file placing it next to where the list-expense-vouchers
controller was added by the angular generator.

```html
<script src="scripts/services/lb-services.js"></script>
```

The next step was to get the dependency to `lb-services.js` Services module added to the ExpenseVouchersClient app. In my app,
the ExpenseVouchersClient app module was declared in `app/scripts/app.js`. I added `lb-services.js` to the list of dependencies as you can see below:

 ```javascript
 angular.module('expenseVouchersClientApp', ['ngAnimate', <other dependencies>, 'lbServices'])
 ```

Now, I was all set to access the required resource on the view's controller. The controller's dependencies was updated to add ExpenseVouchers
resource which was defined in lbServices module loaded with lb-services.js file which we have already added to the app.
In `app/scripts/controllers/list-vouchers.js`

```javascript
angular.module('expenseVouchersClientApp')
  .controller('ListVoucherCtrl', function ($scope, User, ExpenseVoucher){
   $scope.vouchers = ExpenseVoucher.query(); //resource method that performs a GET on the model lying underneath
  }
```

Before I could get this whole thing working, there was a last piece remaining. How does the Angular app know where my backend server is running.
 The Angular app needs to be configured to let it know, for which I used methods exposed by `LoopBackResourceProvider` in the app's
 config block. Once again in the Angular module definition in `app/scripts/app.js`:

```javascript
 angular.module('expenseVouchersClientApp', ['ngAnimate', <other dependencies>, 'lbServices'])
 .config(function ($routeProvider, LoopBackResourceProvider) {
 <routing stuff...>
 // Use a custom auth header instead of the default 'Authorization'
 LoopBackResourceProvider.setAuthHeader('X-Access-Token');
 // Change the URL where to access the LoopBack REST API server
 LoopBackResourceProvider.setUrlBase('http://localhost:3000/api/');
```

The client and backend applications were integrated now. After launching the backend and client applications, I was able to view my
*ExpenseVouchers* by navigating to the list-vouchers client route. So that was it. Head on to the
[loopback AngularJS SDK docs](https://docs.strongloop.com/display/public/LB/AngularJS+JavaScript+SDK) to learn more.

Thanks for reading\!