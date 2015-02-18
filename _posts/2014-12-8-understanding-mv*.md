---
layout: post
title: "Using AngularJS for an MVC Design Pattern"
date: 2014-12-7
---

Most NSS cohorts encounter the AngularJS framework to some degree. My cohort used it in our capstone projects for the front-end portion of the class after spending a couple weeks learning it. 

It took me a while to wrap my head around the MVC design pattern in general, but the good news is that Angular makes the whole process a little easier. MVC (Model-View-Controller) follows the principle of the separation of concerns - i.e. let&lsquo;s keep things focused on doing one job well, and integrating them with each other only out of necessity. So, we have the idea of a model - a representation of some type of information or data that we can work with. This could be the results of a query to a database or any other sort of stored information. A web user never (hopefully) interacts directly with a model, though - instead, a user sees a &lsquo;view&rsquo; which they can interact with. This can be a home page, a login page, a profile page, a shopping cart page, etc. Anytime the view/user side of things needs to interact with data from the model, the third component (a controller) mediates this. 

Coming from a habit of just writing one main.js file for a simple app, this seems like overkill and too complicated, but as soon as you need to write a larger app that processes a lot of different things, the one-big-file approach starts getting really messy. It&rsquo;s way better to start off with the right habits and to be prepared to work with decoupled and separated code for specific tasks. 

Fortunately, Angular does this quite nicely. It literally has components called models, views, and controllers (ng-model, ng-view, and .controller()). Assigning the directive ng-view to a div will load the view templates according to whatever route the user selects (e.g. home, profile, dashboard). Each view can be assigned a particular controller - so each view will only have access to functions and model data which that particular controller provides. 

I really like the way that models are implemented in Angular. The real magic of these models is that they employ two-way data binding, meaning that anything bound to the model will immediately reflect any changes made to the model. This is way cool, since you don&rsquo;t have to worry about consistency and updating data in different parts of your app - one change made in either the DOM or the model will immediately permeate to all the pieces bound to the model. 
