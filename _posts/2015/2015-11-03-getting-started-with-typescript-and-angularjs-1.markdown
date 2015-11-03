---
title: "Getting Started With TypeScript and AngularJS (1)"
tags:
  - TypeScript
  - AngularJS
  - Gulp
---

> Almost a year ago, I wrote an introductory article about developing server and client applications in TypeScript - [TypeScript Development in Practice]({% post_url /2015/2015-01-27-typescript-development-in-practice %}). A lot of things have changed since then so this is a longer and much more thorough description of what is possible today:

It’s said, the first step is the hardest, but it doesn’t have to be. With the correct explanation it can be just as easy and as much fun, as any subsequent one. In this series of posts, I'll explain the very first step to get going with AngularJS and TypeScript in different development environments. At the end of this series, you’ll be able to make the first step with ease and get your own projects going. This is the first of the following 3 posts:

- **Getting started with Visual Studio Code, Atom or Sublime Text 3**
  - Understanding the Gulp build system
- Getting started with Visual Studio 2015

# Getting started with Visual Studio Code, Atom or Sublime Text 3

The simplest way to get started is actually just with a text editor. I will focus on Visual Studio Code and mention additionally Atom and Sublime Text 3, as these have the best support for TypeScript at the moment. The reason why just a text editor and command line is the easiest way to get started, is that they don’t do anything else, except providing you, the user, with syntax highlighting. Later in the post you’ll see why this is of advantage.

At first we’ll need to install the required software, if you have any of the tools mentioned here already installed, consider to still update them. Afterwards we’ll create the very first application and analyse its components. We’ll do this in Visual Studio Code and Sublime Text 3, to see how these editors support TypeScript. In the end, as promised, I’ll share with you, the best TypeScript plugins I know for other editors and how you can find them.

> When you will download git and nodejs, and run the graphical installer, make sure to always include the executable in the PATH variable, in order to be able to use the applications from the command prompt, or console, later on.


## ￼￼￼￼￼￼￼Setting up the tools

Let’s start by installing the two root dependencies, git and nodejs. You can get git online at [http://www.git-scm.com/download](http://www.git-scm.com/download) or via the command line. Nodejs for Windows and OS X can be also downloaded from [https://www.nodejs.org/](https://www.nodejs.org/), or if you prefer the command line, you can use chocolatey in Windows via

    choco install git
    choco install nodejs

or brew in OS X via

    brew install git
    brew install node

> Notice, in brew nodejs is only called node, the js-ending is missing.

If you use Linux, you will be certainly able to get both tools from your distributions package repository. In most cases nodejs is called, nodejs and you’ll need to install the tool npm additionally.

With nodejs also nodes package manager, npm, was installed and from here on it is the same for all operating systems. At first we have to verify whether node is installed properly and we can use it. To test this, open the console and run

    git --version
    node --version
    npm --version

The output should be the version numbers of those applications. Nodejs should be at least of version 0.12.0 and npm 2.7.0. If it didn’t work for you, try restarting the console, or for Windows users, it might be necessary to restart the computer.

Now that nodejs and npm are ready, we have to install TypeScript with a helper application, tsd, as well as some tools to help us get started and later on build the application. We can get all of this with one command:

    npm install –g typescript tsd bower gulp yo generator-gulp-angular

This installs the following additionally to TypeScript:

- the *TypeScript Definition manager for DefinitelyTyped* (tsd, we will use it later)
- a package manager for client-side JavaScript libraries (bower)
- a build system tool (gulp)
- the scaffolding tool Yeoman (yo),
- a generator for Yeoman, which will create our first TypeScript + AngularJS project in just a few steps.

To verify the installation was correct, run

    tsc --version
    tsd –version
    bower --version
    gulp --version
    yo --version
    yo

Verify, Yeoman offers you the *Gulp Angular* generator, and cancel it afterwards with `ctrl+C`. This should have shown you additionally the versions of TypeScript (tsc), which needs to be 1.5 or higher, as well as tsd, bower, gulp and yo, which are for me in the versions 0.6.0, 1.4.1, 3.8.11 and 1.4.6 respectively.

This is it, now we can go and finally create our first application which combines TypeScript and AngularJS.

## Creating the first application

Open the console and go to the new directory where you want to create the application and run:

    yo gulp-angular first-app

In the subsequent text-based assistant:

1. Choose the latest AngularJS version (no fun if it isn't the latest thing, or?)
2. Keep all modules selected (you can uninstall those unused once later on)
3. Choose a jQuery 2.x, as some Angular modules require jQuery (and we don't care about old IE browsers anymore).
4. Choose however you want to access the server. I prefer $http but ngResources, for instance, gives you a lot of additional features for free.
5. I prefer the UI Router, but the traditional angular router is just as good for simple applications and the new angular 2 router should be awesome.
6. Choose a style you prefer. I've used Material Design for new applications and Bootstrap for older ones. Both work great, but the other options are for sure also a good fit for different cases. In the end, you want your application to look a bit unique as well.
7. For demonstration purposes, I'd choose simple CSS, as the other options just add additional gulp-compilation code. But for real projects go ahead, I used to use LESS in the early days, switched to SASS and am using mostly the node version as it works seemlessly also in Windows (had problems with the ruby-based version before).
8. Choose TypeScript as the pre-processor (in case you forgot about the title)
9. Again, for the sake of this tutorial, I'll assume you didn't select a template engine, as those just add more stuff to you gulp scripts. In practice I'm still not selecting any, since they AngularJS is already providing me with one and I don't see quite the point, but go ahead, in case I missed it.

This is creating for you a simple AngularJS with TypeScript application in the current directory, which will have all necessary configurations prepared in order to be able to start development right away. After the command finishes you need to run

    gulp serve

This will compile the generated TypeScript application to JavaScript and open your default browser windows, where you’ll be able to have a look at it. The final output should look similar to mine:

    [BS] [BrowserSync SPA] Running...
    [BS] Access URLs:
     ---------------------------------------
          Local: http://localhost:3000/
       External: http://192.168.0.102:3000/
     ---------------------------------------
              UI: http://localhost:3001
     UI External: http://192.168.0.102:3001
     ---------------------------------------
    [BS] Serving files from: .tmp/serve
    [BS] Serving files from: src

The browser windows should have opened the page [http://localhost:3000](http://localhost:3000), if that isn’t the case, open it yourself. When you go to [http://localhost:3001](http://localhost:3001), you’ll be able to change configurations of the tool BrowserSync, which is used, in order to, not just automatically reload local changes in your browser, but also keep multiple different browsers in sync. You can see a short introduction video and explanation on [http://www.browsersync.io](http://www.browsersync.io). In the next steps we will use Visual Studio Code and Sublime Text 3, to look at the code which was generated, in order to find out, how to create an application without a generator later on.

## Setting up the editor

I will describe here the set up and configuration of the 3 editors, Visual Studio Code, Atom and Sublime Text 3. Visual Studio Code is the simplest editor to set up with the most advanced support for TypeScript and I strongly recommend choosing it. If you have preferences for any of the other two editors, you are of course free to choose them. In the end, you only need to install one editor, but can still try out all 3 to find out, which one will most suit you.

### Visual Studio Code

This is probably the best and most feature complete TypeScript editor out there. It doesn’t require any plugins or extensions. Just go to [https://code.visualstudio.com](https://code.visualstudio.com), download the latest version for your operating system and install it. After launching the editor you can choose to open the folder, in which you have created the first test application. That’s it. This is the editor I’ll use for the rest of this series and also other tutorials on this blog.

### Atom

Atom is a much more mature editor compared to Visual Studio Code, however TypeScript support is not built in. You can get the editor from [https://atom.io](https://atom.io) for any operating system. After installing and launching the editor, you’ll first need to install the TypeScript package. Search for atom-TypeScript in packages and press the install button. Now you can open the folder with the generated first test application, but before you can start doing something, we need to make sure, atom won’t try to compile the TypeScript files for us. For that we have other tools, which already do a great job at it.

So, open src/app/index.ts and you’ll see a small notification, that tsconfig.json has been created for you in src/app. Move this tsconfig.json into the main folder where you have src, gulp and other folders. Now open it and delete the values from “filesGlob”, so that it will stay an empty array. Save the file and “files” should be now also just an empty array. Now you are ready.

### Sublime Text 3

This is probably the most mature editor of these 3, but also has the least TypeScript support. You can download it form [http://www.sublimetext.com/3](http://www.sublimetext.com/3) for your operating system. After installation and the first launch, we’ll need to install the Package Control. The exact installation instructions can be found online: [https://packagecontrol.io/installation](https://packagecontrol.io/installation) and consist of pasting a piece of python code into Sublime Text’s Console. After that, the application needs to be restarted and now the required plugin can be installed.

For this,

1. First, press ctrl+shift+p (cmd+shift+p on OS X) and search Install Package.
2. Choose the first option and now search for the package, TypeScript. This is the full name of the package and should be one of the first in the list. Choose it, so that it gets installed. Now you can open the folder of the generated test application.

## Analysing the Generated Application

Now, that we have the application open, let’s finally look at the code. In src/app, let’s start with index.ts. There are only few lines different compared to a JavaScript file.

At the top, there are references. These are comparable to header files in C/C++ or using and import in C# and Java. When you’ll follow the path of the first reference file, you’ll see more references to files such as angular.d.ts or jquery.d.ts. These are TypeScript Definition Files. They are the bridge between existing JavaScript libraries and out TypeScript code. They describe the JavaScript library, with all its variables and types, like they would be written in a header file in C/C++. It’s important to know, that, no actual code can be inside a definition file (*.d.ts). The other references in our index.ts are pointing at two controllers, so a reference doesn’t need to point to a definition file, it can also point to an actual TypeScript file. The compiler will extract the necessary definitions out of there.

> ￼￼￼￼￼￼TypeScript Definition Files (*.d.ts) contain type definitions for JavaScript code and many existing JavaScript libraries. They allow you to use existing established libraries right from TypeScript, while keeping all the advantages of a strictly typed language, without actually changing those libraries.

The next line which doesn’t look like JavaScript contains module firstApp. TypeScript understands two different kinds of modules, internal and external ones. This is an internal module, and we will also only deal with internal modules during our development. External modules are automatically per file and mostly used for NodeJS development. Generally, modules are there, to provide more structure and easier separation of code in the application. This way, you can have the same class or function with the same name, in different modules. We will look at more of this, when looking at the controllers

In the end, there is the AngularJS configuration, which is written the following way:

    .config(function ($stateProvider: ng.ui.IStateProvider, $urlRouterProvider: ng.ui.IUrlRouterProvider) {

In JavaScript it would be much simpler, like this:

    .config(function ($stateProvider, $urlRouterProvider) {

The additional clutter in TypeScript, those are the statically defined types. Yes, types are defined, unlike in most other languages, after the parameter name. We can `ctrl+click` on the type, or *right-click* and ***Go to Definition***, in order to find out more about the type. In that case we’ll end up in `angular-ui-router.ts` and can explore all the different functions the variable offers us. Another was to do this, is just to write inside the .config- function

    $urlRouterProvider.

(Optionally, if no context menu opens up, press ctrl+space) and we see, what we can do with the Url-Router-Provider. This context aware and correct auto completion is one of the two great advantages of TypeScript. The information is now right in the auto complete, and there is no need to search through API documentations anymore. The second advantage is obviously, if you write a wrong method, or mistype the name, the compiler will catch the error, way before we’ll test our compile JavaScript code in the browser.

### Classes and Interfaces in TypeScript

Ok, now let’s have a look at our main.controller.ts, in src/app/main/main.controller.ts. This doesn’t look like JavaScript at all anymore. It starts with the same internal module declaration as index.ts, however, this follows by a class, and interface and an exported class in the end. Ok, let’s get through this new stuff.

TypeScript supports classes. They look and behave similar to Java and C# classes with few exceptions.

- The constructor is just called constructor, always.
- The arguments of a constructor can have private and public modifiers, which
will automatically make class properties out of it.
- To access class properties or functions, you always need to use this.

The class Thing has actually a lot of redundant code, let’s clean it up. This is how your class should look like:

    class Thing {
      public rank: number;
      public title: string;
      public url: string;

      public description: string;
      public logo: string;

      constructor(title: string, url: string, description: string, logo: string) {
        this.title = title;
        this.url = url;
        this.description = description;
        this.logo = logo;
        this.rank = Math.random();
      }
    }

By writing `public` ahead of the arguments in Thing’s constructor, we define the class properties and TypeScript will assign them automatically. Our new Thing class is much shorter and will compile to exactly the same JavaScript code:

    class Thing {
      public rank: number;

      constructor(public title: string, public url: string, public description: string, public logo: string) {
        this.rank = Math.random();
      }
    }

After saving these changes, you’ll notice, your command line has updated and there is text written, like

    main/main.controller.ts[7, 17]: 'title' cannot be declared in the constructor
    main/main.controller.ts[7, 39]: 'url' cannot be declared in the constructor
    main/main.controller.ts[8, 17]: 'description' cannot be declared in the constructor
    main/main.controller.ts[8, 45]: 'logo' cannot be declared in the constructor

These aren’t actual error messages. They are style errors, which are defined in the root folder of our project in `tslint.json`. We think, it’s perfectly ok to define variables in the constructor, so we’ll disable this warning. In `tslint.json`, search for “**no- constructor-vars**” and set it to `false`. Additionally we’ll disable the warning for trailing whitespaces. Again, search for “**no-trailing-whitespace**” and set it to `false`. Much better now.

Interfaces in TypeScript have, on one hand, similar functionality, like interfaces in Java and C#. On the other hand, objects can automatically implement an interface, as long as they contain all required variables. For instance an interface IThing like

    interface IThing {
      title: string;
      url?: string;
      description?: string;
      logo?: string;
      rank?: number;
    }

Would be implemented by the class Thing, but also by this simple JSON object:

    { title: ‘IThing object’, description: ‘I am a simple object’ }

> Names ending with a question mark (?) in TypeScript show that a property in an interface, or an argument in a function are optional.

### AngularJS Controllers in TypeScript

Controllers and also Services in AngularJS can be written as classes in TypeScript. This has many big advantages when you are thinking about structuring code. However, TypeScript also has disadvantages, when it comes to dealing with very dynamic objects, such as the `$scope` in AngularJS. It is common practice to create interfaces, such as the `IMainScope` interface, which extend AngularJS’s `ng.IScope`, in order to get better type support, when accessing the variables in the interface. Thankfully, this is obsolete now, and instead variables from the controller can be used directly in the view. So let’s see how we can modify the controller, to achieve this. The initial version looks like this:

    export class MainCtrl {
      /* @ngInject */
      constructor ($scope: IMainScope) {
        var awesomeThings = [...];

        $scope.awesomeThings = new Array<Thing>();

        awesomeThings.forEach(function(awesomeThing: Thing) {
          $scope.awesomeThings.push(awesomeThing);
        });
      }
    }

Let’s at first define `awesomeThings` as a public variable in the `MainCtrl` of type `Thing[]`. As a next step, let’s delete `$scope` from the constructor and replace it further down inside the constructor with a simple `this`. In the end, we’ll need to replace the traditional function definition inside `awesomeThings.forEach` with TypeScript’s alternative notation, with an arrow. This will allow us, to actually use `this` inside the function, without it taking on a different value. The new controller should look like this:

    export class MainCtrl {
      public awesomeThings: Thing[];
      /* @ngInject */
      constructor () {
        var awesomeThings = [...];

        this.awesomeThings = new Array<Thing>();

        awesomeThings.forEach((awesomeThing: Thing) => {
          this.awesomeThings.push(awesomeThing);
        });
      }
    }

However, in order to make this work, we’ll need to add in the state in `index.ts` a name for our MainCtrl. Let’s call it just `main`, and the name is added underneath `controller: ‘MainCtrl’` as `controllerAs: ‘main’`. The new state should look like the following:

    $stateProvider
      .state('home', {
        url: '/',
        templateUrl: 'app/main/main.html',
        controller: 'MainCtrl',
        controllerAs: 'main'
      });

And, in the end, everywhere, where `awesomeThings` are being accessed in `main.html`, we need to add `main` in front of it. Luckily, this happens only on one place, in the line:

    <md-card ng-repeat="awesomeThing in awesomeThings | orderBy:'rank'">

Add main in front of awesomeThings and everything works now:

    <md-card ng-repeat="awesomeThing in main.awesomeThings | orderBy:'rank'">
