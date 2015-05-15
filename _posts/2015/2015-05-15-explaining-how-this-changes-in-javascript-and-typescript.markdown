---
title: "Explaining How `this` Changes in JavaScript and TypeScript"
tags:
  - TypeScript
---

My girlfriend is learning [Meteor](https://www.meteor.com/) as she already has some knowledge in AngularJS and I personally think TypeScript would make her life easier (eventually) I suggested her, to follow the great [tutorial on AngularJS with Meteor](http://angular-meteor.com/tutorial), but implement it in TypeScript. Her code is on [GitHub](https://github.com/sigita42/angular-meteor-tutorial), in case you want to give it a try yourself and check how the solution could look like.

While it wasn't easy, in [chapter 14](http://angular-meteor.com/tutorial/step_14) it got really confusing for her, because of `this` being used in a function in JavaScript. There are over 19 Million (!) hits on my Google search for [*javascript this binding*](https://www.google.de/search?q=javascript+this+binding&oq=javascript+this+binding). And this issue gets bigger in TypeScript, which introduces a new way of defining function, with the arrow notation (`() => {}`).

So let's first look at the TypeScript's arrow function:

    ...
    var f = () => {
        console.log("I'm an arrow function in the context");
        console.log(this);
    }
    ...

That code will compile to:

    ...
    var _this = this;
    function f() {
        console.log("I'm an arrow function in the context");
        console.log(_this);
    }
    ...

You see, once you have an arrow function, `this` isn't going to change, ever. No [`bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) or [`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) will change it. TypeScript essentially does with the arrow function, what so many developers did before in plain JavaScript. It introduces a temporary variable which takes on the form of `this`, outside the function (usually called `_this`), and that variable is used instead of `this` inside the function.

Ok, so basically, what ever you know about `bind` and `apply` just isn't relevant, as long as you use arrow functions, awesome, TypeScript might have made your life actually easier. Now what if you actually need to retrieve the value of a modified `this` inside a function? Well, at first, don't define it as an arrow function, it won't work. Now, let's have a look at a practical example. Here is a Meteor method defined from chapter 14, the original JavaScript code:

      Meteor.methods({
        invite: function (partyId, userId) {
          check(partyId, String);
          check(userId, String);
          var party = Parties.findOne(partyId);
            if (!party)
              throw new Meteor.Error(404, "No such party");
            if (party.owner !== this.userId)
              throw new Meteor.Error(404, "No such party");
            if (party.public)
              throw new Meteor.Error(400,
                "That party is public. No need to invite people.");

          if (userId !== party.owner && ! _.contains(party.invited, userId)) {
            Parties.update(partyId, { $addToSet: { invited: userId } });

            var from = contactEmail(Meteor.users.findOne(this.userId));
            var to = contactEmail(Meteor.users.findOne(userId));

            if (Meteor.isServer && to) {
              // This code only runs on the server. If you didn't want clients
              // to be able to see it, you could move it to a separate file.
              Email.send({
                from: "noreply@socially.com",
                to: to,
                replyTo: from || undefined,
                subject: "PARTY: " + party.title,
                text:
                  "Hey, I just invited you to '" + party.title + "' on Socially." +
                  "\n\nCome check it out: " + Meteor.absoluteUrl() + "\n"
              });
            }
          }
        }
      });

Let's try to transform it into TypeScript. I won't be adding references as I assume they are in the head of the file. Here the new code, with comments on what changed:

      // first we need to create a new interface for the context `this`, in which the method will be executed
      interface IMeteorMethodThis {
        userId: string; // this is the only property I'm sure of and which I need below, so it's ok
      }

      Meteor.methods({
        invite: function (partyId: string, userId: string) { // adding type information, no arrow notation allowed :(
          check(partyId, String);
          check(userId, String);

          // we will be using `this` but in order to get better syntax support, we'll need to cast it. In these cases, I'm assigning it to a new variable called `me`
          // this GitHub issue is meant to deal with that problem: https://github.com/Microsoft/TypeScript/issues/229 - add comments there, to keep it alive
          var me = <IMeteorMethodThis>this;

          var party = Parties.findOne(partyId);
            if (!party)
              throw new Meteor.Error('404', "No such party"); // Meteor.Error actually expects 2 strings, so 404 needs to be in quotes
            if (party.owner !== me.userId) // we are using `me` here for type checking
              throw new Meteor.Error('404', "No such party"); // Meteor.Error actually expects 2 strings, so 404 needs to be in quotes
            if (party.public)
              throw new Meteor.Error('400', // Meteor.Error actually expects 2 strings, so 400 needs to be in quotes
                "That party is public. No need to invite people.");

          if (userId !== party.owner && ! _.contains(party.invited, userId)) {
            Parties.update(partyId, { $addToSet: { invited: userId } });

            var from = contactEmail(Meteor.users.findOne(me.userId)); // one more time `me`
            var to = contactEmail(Meteor.users.findOne(userId));
            // ... the rest doesn't change ...
          }
        }
      });

As you can see, the changes were actually minimal, but targeted. And for somebody without a concept of `bind` and `apply`, they might seem like magic. As it is written in [one of the first posts on the Google search](http://javascriptissexy.com/javascript-apply-call-and-bind-methods-are-essential-for-javascript-professionals/)

> JavaScript’s Apply, Call, and Bind Methods are Essential for JavaScript Professionals

But the tutorial isn't for professionals, still you need to understand something so complex, in order to translate it on the fly to TypeScript. Let's make it a bit more complicated to see the actual problem. Let's assume, I want to have a class with all my Meteor methods to structure the code better.

    class MyMethods {
        invite(partyId: string, userId: string) {
            ...
            this. ... // Question: what is the value of this?
            ...
        }
    }
    var m = new MyMethods();

Depending on how we will use it now, `this` inside our invite method will change:

    m.invite('asdf', 'qwer'); // `this` is `m`, as it's supposed to be

    Meteor.Methods({
        invite1: m.invite, // `this` is the special object, provided by Meteor with properties like the userId (IMeteorMethodThis from above)
        invite2: (partyId: string, userId: string) => { m.invite(partyId, userId); }, // `this` is again `m`
        invite3: m.invite.bind(m) // `this` is again `m`
    });

The problem here is, that you can only keep one `this`, either the instance of `MyMethods`, or the object provided by Meteor. The only solution, I've found to make code like above work (having a class with all the methods), would be having `invite2` as a normal function and than modifying either the object `m` or having another function, where the current `userId` (`this.userId`) could be passed on. Here an example:

    class MyMethods {
        invite(currentUserId: string, partyId: string, userId: string) {
            ...
        }
    }
    var m = new MyMethods();

    Meteor.Methods({
        invite: function (partyId: string, userId: string) {
            var me = <IMeteorMethodThis>this;
            m.invite(me.userId, partyId, userId);
        }
    });

Another option would be, creating the object only inside the `invite` function and passing `this.userId` as a parameter to the constructor:

    class MyMethods {
        constructor(private currentUserId: string)

        invite(currentUserId: string, partyId: string, userId: string) {
            ... // user this.currentUserId here
        }
    }

    Meteor.Methods({
        invite: function (partyId: string, userId: string) {
            var me = <IMeteorMethodThis>this;

            var m = new MyMethods(me.userId);
            m.invite(partyId, userId);
        }
    });

Which one of these methods is better and more appropriate will for sure depend on the implementation of `MyMethods`.

Additionally I've also created a small script, where you can see the confusing behavior of `this`, as I've been just describing it, in practice. You can find it on [GitHub](https://github.com/pgrm/typescript-this-explained). After downloading just run

    npm install
    npm start

to see it running. `npm start` compiles the `index.ts` and runs it immediately after. You can also see the generated JavaScript code in the compiled `index.js`. You can find the explanations on why, which line prints what as comments in the TypeScript code. Have fun with it, and hopefully it will save you some trouble.