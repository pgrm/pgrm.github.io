---
title: TypeScript Development in Practice
date: 2015-01-27 00:00:00 Z
tags:
- TypeScript
- NodeJS
- Gulp
---

I'm using [TypeScript](http://www.typescriptlang.org) already since 2012 for rich client development on top of AngularJS. As the server was written in ASP.Net, we were using Visual Studio for Server and Client. I didn't have almost any experience in JavaScript, or Angular. So TypeScript's strong typing and the type definitions (`.d.ts`) for AngularJS and other frameworks, we were using, helped me a lot getting to know the functionality.

By now I do feel confident enough to write plain JavaScript, but still choose to use TypeScript for the type safety as a simple first way to catch possible errors. However, I found it quite difficult, to set up a good working environment in Linux with TypeScript, where everything would be smooth.

I started with [WebStorm](https://www.jetbrains.com/webstorm/), which is an awesome IDE, but found it having a bug (I also encountered a similar behavior in Visual Studio) - from time to time, it would just show a pop-up that my `.ts` file has been modified outside of the IDE and if I want to reload it. Reloading it basically meant loosing my changes, so not a good idea. Also using all of WebStorm's features meant, that everyone else on my team would need to have WebStorm.

Second try was to use the TypeScript compiler directly from the command line (`tsc`). It has a `watch` option and is awesome for any small enough project, or JavaScript / TypeScript only code. However, it is difficult when you want to combine it with [Gulp](http://gulpjs.com/) or [Grunt](http://gruntjs.com/).

## The Build Systems

### 1. [typescript-require](https://github.com/eknkc/typescript-require)

I was wondering how people using [CoffeeScript](http://coffeescript.org/) were actually solving the problem, and I found [require-cs](https://github.com/requirejs/require-cs). So I thought, there has to be something similar for TypeScript. And there is - [typescript-require](https://github.com/eknkc/typescript-require). This is how it works:

0. You install it via `npm install typescript-require`
1. You need to have one entry file written in JavaScript
2. You put `require('typescript-require');` in the top of this file.
3. From here on you can simply require a TypeScript file, like if it were JavaScript. `typescript-require` will compile it on the fly.

**Awesome!** - However, it has some drawbacks. At the time of this writing, it has problems dealing with different directory hierarchies. So you want to keep all your files within one folder and no sub-folders (if your project is small enough this isn't too much of a limitation though). I was using it for [VimFika](http://vimfika.logtank.com/)([source code on GitHub](https://github.com/pgrm/vimfika)), but ended up having a problem. I had a compile error, which I didn't notice and typescript-require just failed without any exception. Then I switched to...

### 2. [gulp-typescript](https://www.npmjs.com/package/gulp-typescript)

Gulp with [gulp-typescript](https://www.npmjs.com/package/gulp-typescript) is what I'm using for server. It allows to plug the already useful and powerful `tsc` command into my gulp scripts (I've tried it just by executing the raw shell command before, but that didn't work too well). I use it for my sever-side code only, as I don't require there any optimization. When deploying my Rest API written in TypeScript, using ExpressJS for example, I use gulp-typescript to compile all ts files into a destination folder. You can run it from there with [nodemon](https://github.com/remy/nodemon) (or [gulp-nodemon](https://www.npmjs.com/package/gulp-nodemon) when using gulp). To deploy this (for instance to [heroku](http://heroku.com/)) you also need to copy the `package.json` file into the destination folder and deploy the destination folder, instead of your development folder.

### 3. [Yeoman](http://yeoman.io/)

Turns out there is an amazing generator for TypeScript working with [Yeoman](http://yeoman.io/). If you want to have a very sophisticated build system set up for client code, with minification, bower support, ... you have to try out the [gulp-angular generator](https://github.com/Swiip/generator-gulp-angular). It scaffolds **EVERYTHING** you could dream of. With one command and few configuration steps you are ready to start writing your rich client application.

Too bad there isn't something like that for the server.

## The IDE or Text Editor

I stopped using WebStorm because of the annoying little pop-up. I switched to [Sublime](http://www.sublimetext.com/) and am staying there for now. I'm using [T3S](https://github.com/Railk/T3S/tree/master) which is a great plugin to work with TypeScript. There is an [issue where the plugin might freeze](https://github.com/Railk/T3S/issues/72), but the latest dev branch works fine for me (yes, dev is actually behind master/ST3 branch, but that's what's working for me).

Other Editors also have TypeScript support in one form or another so maybe it is worth to look into [Atom](https://atom.io/) or Vim, but for now I'm happy that finally everything seems to work.