---
title: How to write your own TypeScript Definition Files
date: 2015-05-25 00:00:00 Z
tags:
- TypeScript
---

TypeScript is an amazing language. It makes working with JavaScript just so much easier, at least for me. To be honest, I never really understood or even liked JavaScript. It was a mystery for me, how you can use it to write large applications. But when TypeScript came out - wow, so amazing, almost like C# or Java. During the years, I've of course came to terms with JavaScript, and a solid understanding is necessary, to understand the behavior of TypeScript, just like understanding assembly helps you write better C code.

For me, the biggest advantage of TypeScript and its type system comes in play, when you need to use a new library. With JavaScript you'd have to sit there with the documentation opened and always check, what kind of methods do exist on the object, how can you call them, ... With TypeScript, it's all in your code. You can explore the library right from your code, with the help of context aware auto completion. This works also with JavaScript libraries, thanks to TypeScript Definition Files and [tsd](http://definitelytyped.org/tsd/). You can find definition files for around thousand different libraries already on [GitHub](https://github.com/borisyankov/DefinitelyTyped) and install them via `tsd`.

But what if the library, you are looking for, doesn't have a definition file yet? Well you can report a request and hope that somebody will do it for you, or - you could just do it yourself. OpenSource doesn't just mean to use everybody's tools for free. It also means to contribute to them and make them better for yourself as well as the community. *But it's scary.* - The first time I thought about writing a definition file myself, it really looked scary and it probably still is for large frameworks. Luckily, most of the large frameworks already have definition files. It's the small extensions and plugins, which are missing it. And writing those is actually really simple. Today, we will write together the definition file for a plugin for [leaflet](http://leafletjs.com/), a web mapping library, called [leaflet-draw](https://github.com/Leaflet/Leaflet.draw). Leaflet already has [definition files](https://github.com/borisyankov/DefinitelyTyped/tree/master/leaflet), which will make our task easier, as we can always check, how things are defined in the base library.

Before we start, there are few things, which are TypeScript specific, and you should know.

1. You can have multiple interfaces and/or module with the same name, and TypeScript will merge them together, even if they are in different files. We can use this feature to extend the original Leaflet definition files.
2. TypeScript supports at the moment 3 types of modules: internal, external and es6 modules. Generally for everything loaded via npm, you want to use the external module. This is also valid for [browserify](http://browserify.org/) and similar tools. If you just concatenate your JavaScript files, for browser use, or are using meteor, you need to define internal modules.

# Write your own TypeScript Definition File for [leaflet-draw](https://github.com/Leaflet/Leaflet.draw)

When you are planning on contributing the files back to [DefinitelyTyped](https://github.com/borisyankov/DefinitelyTyped), as I hope you are, you should start by reading their [contribution guide](http://definitelytyped.org/guides/contributing.html). It's very short and will make it easier for you, to get your pull request merged.

## Get to know the library

The first thing we need to do, is getting to know the library. Hopefully the library for which you want to create a definition file has a comprehensive API documentation or anything similar. The documentation for leaflet-draw can be found directly in the [README.md](https://github.com/Leaflet/Leaflet.draw/blob/master/README.md). In the first section, [Using the plugin](https://github.com/Leaflet/Leaflet.draw/blob/master/README.md#using), we see a practical example. Let's first go back to the [leaflet definition file](https://github.com/borisyankov/DefinitelyTyped/blob/master/leaflet/leaflet.d.ts), to see, which parts are already defined there, and which parts need to be defined by us.

Surprisingly, the sample code

    // create a map in the "map" div, set the view to a given place and zoom
    var map = L.map('map', {drawControl: true}).setView([51.505, -0.09], 13);

    // add an OpenStreetMap tile layer
    L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

seems to work without problems. But if we rewrite it slightly, we get to see the first error:

    // create a map in the "map" div, set the view to a given place and zoom
    var mapOptions: L.MapOptions = {};

    mapOptions.drawControl = true; // <-- error here, drawControl doesn't exist in MapOptions

    var map = L.map('map', {drawControl: true}).setView([51.505, -0.09], 13);

    // add an OpenStreetMap tile layer
    L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

In case your sample code didn't compile properly either, don't forget to add the reference tag to point at `leaflet.d.ts` or `typings/tsd.d.ts`;

## Fixing the first error, getting started

Let's create a file `typings/leaflet-draw/leaflet-draw.d.ts` and don't forget to have `typings/leaflet/leaflet.d.ts`. As our plugin depends on leaflet, we'll start `leaflet-draw.d.ts` with

    /// <reference path="../leaflet/leaflet.d.ts" />

The next part is to create a matching module `L`, so TypeScript would merge them together, like this

    declare module L {
        // all our code will end up here
    }

If you wonder, why `declare`, well TypeScript complains otherwise:

> A declare modifier is required for a top level declaration in a .d.ts file.

Inside the module, we can now define the interface, with the additional property `drawControl`, make sure the property is optional.

    export interface MapOptions {
        drawControl?: boolean;
    }

If you are referencing `leaflet-draw.d.ts` inside your test file, the modified example should now work. Let's try the next example from the page.

## 2nd example, what else is missing?

The second example already reports errors right away. In case you didn't replace the old code, remove `var` from `var map`, to not declare it a second time.

    // create a map in the "map" div, set the view to a given place and zoom
    var map = L.map('map').setView([51.505, -0.09], 13); // if you have the old example above, remove `var` here

    // add an OpenStreetMap tile layer
    L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    // Initialise the FeatureGroup to store editable layers
    var drawnItems = new L.FeatureGroup();
    map.addLayer(drawnItems);

    // Initialise the draw control and pass it the FeatureGroup of editable layers
    var drawControl = new L.Control.Draw({ // <-- error here, Draw is missing
        edit: {
            featureGroup: drawnItems
        }
    });
    map.addControl(drawControl);

Ok, now we have to check, how `L.Control` is defined in `leaflet.d.ts`.

You see, the name `Control` is used for 3 different things:

1. A variable: `export var Control: ControlStatic;`
2. An interface: `export interface Control extends IControl {...}`
3. A module: `module Control {...}`

Our definition of `L.Control.Draw` will consist of 2 parts.

1. We'll extend `ControlStatic` to contain the property `Draw`.
2. We'll define an interface for the property `Draw` inside the module `Control`, like others did it.

Back to our `declare module L` in `leaflet-draw.d.ts`, add the following code at the bottom:

    export interface ControlStatic {
        Draw: Control.DrawStatic;
    }

    module Control {
        export interface DrawStatic {

        }
    }

This looks worse than before. I don't know about your editor, but mine is showing now an even bigger error (judging from all the red). The reason is the missing constructor. Wait, constructor on an interface? - Yes, constructor on an interface, welcome to TypeScript. First we need to look at the definition of the constructor in the documentation. Go to [Advanced options](https://github.com/Leaflet/Leaflet.draw/blob/master/README.md#options) and you'll see all the properties and types we'll need to define in TypeScript. Starting with the constructor, add this inside the `DrawStatic` interface

    new(options?: IDrawConstructorOptions): Draw;

Yeah, unlike the other option types in leaflet's `Control` module, ours isn't just called `DrawOptions`, because you see, the term is reserved by the library. Anyway, we still need to define those interfaces inside our `Control` module:

    export interface IDrawConstructorOptions {

    }

    export interface Draw {

    }

## It's getting serious

What about the current error? `map.addControl` is complaining, that `Draw` isn't of type `IControl`. That one can be fixed easily, just modify the interface definition to extend `IControl`

    export interface Draw extends IControl {...}

great, we are without errors, but if we'd rewrite the code similarly, like we did with the first example, it would complain again, that `IDrawConstructorOptions` is missing `edit`. Let's not do it, let's instead create all the necessary properties and types based on the documentation. First we have to define `IDrawConstructorOptions`, based on the documentation and don't hesitate copying the description into comments and generally providing as much information to people who will be using your definition file, as possible, in order to make their life easier.

Add this to `IDrawConstructorOptions`

    /**
     * The initial position of the control (one of the map corners).
     * Default value: 'topleft'
     */
    position?: string;

    /**
     * The options used to configure the draw toolbar.
     */
    draw?: DrawOptions;

    /**
     * The options used to configure the edit toolbar.
     */
    edit: EditOptions;

`edit` is slightly strange, as it says *default is `false`*. What is actually meant, is that the field is required. You'll understand it when reading through the documentation, `edit` has a property itself which is mandatory and must be defined in order for the plugin to work. The property `featureGroup` in [EditOptions](https://github.com/Leaflet/Leaflet.draw/blob/master/README.md#editoptions):

> This is the FeatureGroup that stores all editable shapes. **THIS IS REQUIRED FOR THE EDIT TOOLBAR TO WORK**

Now `DrawOptions` and `EditOptions` are missing. Those are just additional interfaces in our module. For the sake of keeping this post short, I won't add comments to all those properties, but always keep in mind, you'd love to read those comments directly inside the IDE and not search for them somewhere online, so give yourself a treat and put them there, it's just a copy & paste. Next two interfaces:

    export interface DrawOptions {
        polyline?: PolylineOptions;
        polygon?: PolygonOptions;
        rectangle?: RectangleOptions;
        circle?: CircleOptions;
        marker?: MarkerOptions;
    }

    export interface EditOptions {
        featureGroup: FeatureGroup;
        edit?: EditHandlerOptions;
        remove?: DeleteHandlerOptions;
    }

As you can see, some of the types already exist. `PolylineOptions` and `MarkerOptions` are already defined inside the module `L`, but with different properties, as we'd want them to be. As I see it, we have 2 options.

1. Change the name of the interface - this would provide a lot of confusion for people who want to understand our interface, and combine it together with the documentation.
2. Put it inside an additional module, `Draw`.

Generally you should be careful with modules, when it comes to functions or variables, as those need to have exactly the same path from outside, as they have in the JavaScript library, but for interfaces it matters less, so I vote for an extra module. Let's create the module `DrawOptions` inside the module `L` at the bottom of this file. This is the module where we'll define `PolylineOptions`, `PolygonOptions` and all the other types, which are missing. Prepend all types of properties, in the previously defined interfaces with `DrawOptions.`. All, except `FeatureGroup` of course, as that one clearly should be referencing the [Leaflet FeatureGroup](http://leafletjs.com/reference.html#featuregroup). But also with `FeatureGroup` something is wrong, we need to define a generic type. As we don't want to restrict it too much, let's just take the already restricted generic type, `ILayer`. The new interfaces should look like this:

    export interface DrawOptions {
        polyline?: DrawOptions.PolylineOptions;
        polygon?: DrawOptions.PolygonOptions;
        rectangle?: DrawOptions.RectangleOptions;
        circle?: DrawOptions.CircleOptions;
        marker?: DrawOptions.MarkerOptions;
    }

    export interface EditOptions {
        featureGroup: FeatureGroup<ILayer>;
        edit?: DrawOptions.EditHandlerOptions;
        remove?: DrawOptions.DeleteHandlerOptions;
    }

As you can see, the tiring thing about writing those definition files, is that you are always missing more and more types, until it tips. I promise, this will end soon ;)

## More types

Let's define the next types within our new module `DrawOptions`. Again, simply from top to bottom, based on the documentation, and don't forget to add comments.

    export interface PolylineOptions {
        allowIntersection?: boolean;
        drawError?: any;
        guidelineDistance?: number;
        shapeOptions?: L.PolylineOptions;
        metric?: boolean;
        zIndexOffset?: number;
        repeatMode?: boolean;
    }

Let's have a short break here, with few comments from my side:

- Why is `drawError` `any` and not an `Object`? - you can see `any` as going back to classical JavaScript. You can call any property or function you want on a variable of type `any`, without the TypeScript compiler complaining. Object can contain any possible value, like `any`, but the compiler will complain, if you call methods which don't belong to the default object.
- `shapeOptions` are already defined in `L`. Those are btw. the same options with which we had a clash earlier.
- For the upcoming `PolygonOptions`, you can read the documentation, which essentially means, that the interface extends `PolylineOptions`.

> Polygon options include all of the Polyline options plus the option to show the approximate area.

Good, more types to define:

    export interface PolygonOptions extends PolylineOptions {
        showArea?: boolean;
    }

    export interface RectangleOptions {
        shapeOptions?: L.PathOptions;
        repeatMode?: boolean;
    }

    export interface CircleOptions {
        shapeOptions?: L.PathOptions;
        repeatMode?: boolean;
    }

    export interface MarkerOptions {
        icon?: L.Icon;
        zIndexOffset?: number;
        repeatMode?: boolean;
    }

Great, we're done with the interface `DrawOptions`. Before we move on to `EditOptions`, you might have noticed, `RectangleOptions` and `CircleOptions` share both exactly the same properties, so yes, we could create an interfaces `ShapeOptions` with the two properties and afterwards extend this interface. Maybe it's a good idea, maybe not, one would need to know the API and other modules which interact with it, to understand it better. For now I'll let it be, as it is, since this is also how it is defined within the documentation. But for future changes, or other libraries, you definitely can create base interfaces in order to not copy & paste code, but use OO patterns instead.

## Almost done, `EditOptions` is missing

As you see, we didn't get any additional interfaces to define, so we are almost there. Now let's deal with the `EditOptions`

    export interface EditHandlerOptions {
        selectedPathOptions?: L.PathOptions;
    }

    export interface DeleteHandlerOptions {

    }

And this is where we come to a slight bummer - `DeleteHandlerOptions` is not defined in the documentation. At such situations you have some possible choices:

- Ignore it - seriously, you've done already a great job, if you don't need it, just let it be.
- Go through the code, to figure out the properties - you are a brave hero, trying this. And depending on the code it actually might be easy, or at least feasible.
- Try to figure out the types, based on existing examples. I think this is the best solution so far, I usually go with it.
- Write an issue on [GitHub](https://github.com/Leaflet/Leaflet.draw/issues/408) to check, if they can clarify the type in question.

## Last test

Great, seems like we are finished. Let's test it out with the large sample code at the [bottom of the page](https://github.com/Leaflet/Leaflet.draw/blob/master/README.md#commontasks). I've added some TypeScript types to it, to make sure our definition files work properly, and here we go:

    /// <reference path="tsd.d.ts" />
    var cloudmadeUrl = 'http://{s}.tile.cloudmade.com/BC9A493B41014CAABB98F0471D759707/997/256/{z}/{x}/{y}.png',
       cloudmade = new L.TileLayer(cloudmadeUrl, {maxZoom: 18}),
       map = new L.Map('map', {layers: [cloudmade], center: new L.LatLng(-37.7772, 175.2756), zoom: 15 });

    var editableLayers = new L.FeatureGroup();
    map.addLayer(editableLayers);

    var MyCustomMarker = L.Icon.extend({
       options: {
           shadowUrl: null,
           iconAnchor: new L.Point(12, 12),
           iconSize: new L.Point(24, 24),
           iconUrl: 'link/to/image.png'
       }
    });

    var options: L.Control.IDrawConstructorOptions = {
       position: 'topright',
       draw: {
           polyline: {
               shapeOptions: {
                   color: '#f357a1',
                   weight: 10
               }
           },
           polygon: {
               allowIntersection: false, // Restricts shapes to simple polygons
               drawError: {
                   color: '#e1e100', // Color the shape will turn when intersects
                   message: '<strong>Oh snap!<strong> you can\'t draw that!' // Message that will show when intersect
               },
               shapeOptions: {
                   color: '#bada55'
               }
           },
           circle: false, // Turns off this drawing tool
           rectangle: {
               shapeOptions: {
                   clickable: false
               }
           },
           marker: {
               icon: new MyCustomMarker()
           }
       },
       edit: {
           featureGroup: editableLayers, //REQUIRED!!
           remove: false
       }
    };
    var drawnItems = new L.FeatureGroup();
    map.addLayer(drawnItems);

    var drawControl = new L.Control.Draw(options);
    map.addControl(drawControl);

    map.on('draw:created', function (e) {
       var type = e.layerType,
           layer = e.layer;

       if (type === 'marker') {
           layer.bindPopup('A popup!');
       }

       drawnItems.addLayer(layer);
    });

As you can see, everything works, except the function parameter `e`. Now we could define `e` as `any` and it the compiler would stop complaining, but we've come so far, let's go the last steps and help everybody, who's going to use this library in TypeScript later on.

# The extra mile, defining parameter types for event functions

The plugin, for which we are writing the definition file, [leaflet-draw](https://github.com/Leaflet/Leaflet.draw) is also throwing [events](https://github.com/Leaflet/Leaflet.draw/blob/master/README.md#events). Events are registered with a simple string, so it's not possible to know for us, which event will call which function signature. But we can provide parameter types for all the events, and let the developer use them later on. For this, we can add a new module inside Leaflet's `L` module, let's call it `DrawEvents`. In this module we can define the interfaces for all different event parameters. As before, I'm skipping the comments, but you should add them.

    module DrawEvents {
        export interface Created {
            layer: ILayer;
            layerType: string;
        }

        export interface Edited {
            layers: LayerGroup<ILayer>;
        }

        export interface Deleted {
            layers: LayerGroup<ILayer>;
        }

        export interface DrawStart {
            layerType: string;
        }

        export interface DrawStop {
            layerType: string;
        }

        export interface EditStart {
            handler: string;
        }

        export interface EditStop {
            handler: string;
        }

        export interface DeleteStart {
            handler: string;
        }

        export interface DeleteStop {
            handler: string;
        }
    }

And now we update the previous example code, to use the newly created interface. Out of

    map.on('draw:created', function (e) {
       var type = e.layerType,
           layer = e.layer;

       if (type === 'marker') {
           layer.bindPopup('A popup!');
       }

       drawnItems.addLayer(layer);
    });

we'll make

    map.on('draw:created', (e: L.DrawEvents.Created) => {
       var type = e.layerType,
           layer = e.layer;

       if (type === 'marker') {
           (<L.Marker>layer).bindPopup('A popup!');
       }

       drawnItems.addLayer(layer);
    });

We still need to check strings and cast variables, but I think it is better to understand the library now. You can see the final code, to compare, here: https://gist.github.com/pgrm/2b9e25ebe3a53188135d#file-leaflet-draw-d-ts

# Final words

We've now created a definition file for [leaflet-draw](https://github.com/Leaflet/Leaflet.draw). It might have not been very easy, but in my experience, having a definition file is worth the time, in order to have a clearer structure and better checks later on in the code. The only time, I don't write definition files is for [express](http://expressjs.com/) middlewares, which only contain few options and are defined only once in my whole application. Everything else gets messy, confusing and when you have co-workers, which don't know the library in question by heart, like you might do, they'll thank you a lot for a definition file.

So now, follow up the [contribution guide](http://definitelytyped.org/guides/contributing.html) and send a pull request, to share your definition file, with everyone else.