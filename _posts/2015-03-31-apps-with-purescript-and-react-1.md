---
layout:     post
title:      Building Apps with PureScript and React #1
date:       2015-03-31 23:45:00
summary:    Setting up for our first project
categories: tutorial
---

In this blog post series I will show you, step by step, how I build apps with ReactJS, RxJS and PureScript. It is an architecture I came up with while developing [FROST](https://github.com/kRITZCREEK/FROST-Frontend), so don't expect industry proven best practices.

I also didn't start coding and everything just fell in place. Instead it _evolved_ to solve the new problems that came along during the project and this flexibility is also what makes me confident that __you__ can build apps with this setup.

Before we get started, I want to encourage you to experiment and change what worked for _me_ to make it work for _you_!

## The Application

Enough with the arcane prophecies. Let's get to the meat! I was thinking about a good project for a capable functional programmer like you.

#### Your Boss hastily calls you in...

... to talk to you about your next assignment. An important customer is asking your company to build a shopping site. They want their customers to be able to browse the products, gather a bunch of items into a shopping cart and proceed to checkout. They also provided you with a mockup for the expected result.

![Mockup](/images/squirrelzon_mockup.png)

Pretty standard stuff right?

## The Plottwist

Your customer is a company run by crazy _squirrels_. 

_Squirrelzon_ has seen huge success lately because of the adrenaline kicks their customers get during the shopping process. __Every second__ the price for random products change their price. This could mean a raise by a certain amount, a percentual change or even bulk discounts.

All of your JQuery obsessed coworkers deemed the task impossible and you're the __last remaining hope__.

### PureScript and FRP

To demonstrate the benefits that PureScript and functional reactive programming bring to the browser we're going to consume the provided WebSocket update stream and build a resilient functional webfrontend for the little rodents.

<span class="absolute-center"> ![squirrel](/images/squirrel.svg) </span>

## Setting up our project

A good build process and infrastructure is going to set us up for success. I will guide you through the necessary installation steps on a Linux system. I expect these instructions to work on a Mac without too much hassle but they will probably need some tweaking to work on Windows.

### Prerequisites

PureScript integrates very well with the existing Javascript infrastructure so all you really need is a recent version of [node](https://nodejs.org/). I suggest you use [nvm](https://github.com/creationix/nvm) though, to reduce the amount of `sudo` required. Once you've set up nvm we're going to install a recent version of node.

``` bash
nvm install v0.12.1
nvm alias default v0.12.1
nvm use default
```
Make sure that node is installed correctly:

``` bash
node --version
# v0.12.1
which node
# /home/your_username/.nvm/versions/v0.12.1/bin/node
```
As our only global requirements we're going to install two tools.

- `gulp`, a taskrunner, which we will use to build our project 
- `bower` a dependency management tool that the PureScript community uses to manage their libraries.

``` bash
npm install -g gulp bower
```
_If you didn't use nvm, you will have to use sudo for this command._

### Configuration Files...

Instead of starting from scratch I will provide you with a basic configuration and walk you through it. 

You can get the project scaffold from [this repository](https://github.com/kRITZCREEK/squirrelzon).

#### Gulp Tasks

##### clean

  This just removes our destination folder, so that our build starts fresh

##### compile:purescript

  This task will compile all our PureScript sources aswell as our dependencies into the output folder using the excellent [gulp-purescript](https://github.com/purescript-contrib/gulp-purescript) plugin

##### build

  Here we take the compiled PureScript sources, our JavaScript files and bundle them up into one application using webpack (more on that later)

##### watch-build
  The only difference to the build task is, that this task will also reload the browser after building.

##### serve

  This task will first start a Webserver at `localhost:3000` and then watch our source files. Whenever we make a change to our source files it will trigger a rebuild.

These tasks are configured in `Gulpfile.js`

#### Webpack

We will use [Webpack](http://webpack.github.io/) to bundle our files. The main advantage this gives us, is that we can import our PureScript modules right in our JavaScript files. Using [Babel](https://babeljs.io/) will also allow us to use the much cleaner and nicer ES6 syntax for our JavaScript.

This is configured in `webpack.config.js`

## First Build

The last step before we can trigger the build is installing our dependencies.
In the project folder run:

```
npm install
```

If this succeeds we are through with the drudge work. Let's get to the fun part: __Running our Application__

In the project folder run:

```
gulp
```

This is the only step that you will have to run everytime you want to start hacking on Squirrelzon. Once the build is finished point your favourite browser at [http://localhost:3000](http://localhost:3000).

If you see a nice greeting you are up and running!

### Editor Setup

You are of course free to use whatever Text Editor you prefer.

I did include a `.eslintrc` file so that you can use [eslint](http://eslint.org/) with the [babel parser](https://github.com/babel/babel-eslint) to lint your Javascript files.

You can find a tutorial for how to set up Sublime Text to work with babel-eslint here: [https://medium.com/@dan_abramov/lint-like-it-s-2015-6987d44c5b48](https://medium.com/@dan_abramov/lint-like-it-s-2015-6987d44c5b48)

If you want to use Sublime Text as your editor you should also install the `purescript` Package with [Package Control](https://packagecontrol.io/).

## Conclusion

We got a lot done already. A solid configuration is the best basis for our project.  Next time we will see how the greeting gets put to the screen and start putting together our interface. 

Feel free to contact me by either filing an [issue](https://github.com/kRITZCREEK/squirrelzon/issues) on GitHub or messaging me on Twitter @kRITZCREEK.

I would love to get some feedback and suggestions from you since this is my first time writing a tutorial. I will also try to answer all your questions as good as I can.