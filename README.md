# Ivy

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 7.1.1.

This is an initial test of server side rendering on top of Angular Ivy.

The base Ivy client project was generated by

`ng new ivy --experimental-ivy`

## Build and run the Universal application

`npm install`

1. `npm run build:ssr`
1. `npm run serve:ssr`

Navigate to http://localhost:4000

## What's this?

The example illustrates how Angular Ivy makes the following possible

1. "Automatic" code splitting at component and route level
1. Server-side Rendering with Angular Ivy
1. Selective client side rehydration based on user events / data binding changes

Coming soon:
1. Configurable load and prefetch strategies
1. Data fetching and state transfer
1. Server streaming?

## Code organization

All the things happening server side are at https://github.com/vikerman/ivy-universal/tree/master/server
All the user code are in https://github.com/vikerman/ivy-universal/tree/master/src/ except for https://github.com/vikerman/ivy-universal/tree/master/src/lib which contains all the supporting code which would eventually be part of some NPM node module.

The project itself relies on nightly Ivy builds with no modifications.

(Though [main.ts](https://github.com/vikerman/ivy-universal/blob/master/src/main.ts), [main.server.ts](https://github.com/vikerman/ivy-universal/blob/master/src/main.server.ts) and [routes.ts](https://github.com/vikerman/ivy-universal/blob/master/src/routes.ts) are currently hand written, they would eventually be automatically generated by Angular CLI using Angular architect APIs by parsing the decorator metadata for each component. This would be the part that would make the code splitting and progressive bootstrapping "automatic".)


## What's happening and how does it work?

The webpack magic comments in https://github.com/vikerman/ivy-universal/blob/master/src/main.ts ensures that each Angular Component in the components/ and pages/ folder gets a separate chunk. The default Angular CLI webpack settings ensures that common chunks are automatcially created for any code shared between these components.

On the server the page for the current route gets rendered(more on this later). The rendered HTML is sent to the client. On the client the only code that runs is a bootstrapping code. This code does the following:

- Setup a global event handler on the document in order to buffer events.
- Registers a shell Custom Element for every known component in the system.

At this point no Angular specific code has been loaded.

When a user initiates an event (say click a button) - The global event handler walks up the DOM tree from the event target till it find what appears to be a component root. It then uses webpack loader to load the chunk corresponding to the component.

The component on bootstrapping rehydrates the DOM node sent by the server using the [rehydration renderer](https://github.com/vikerman/ivy-universal/blob/master/src/lib/rehydration/rehydration_renderer.ts). This lets the component become active without having to load the code for its child components because as of now they didn't require to be re-rendered.

Let's say the currently active component changes its state based on the click handler and that resulted in chaging the data binding for the child - The shell Custom Element setup for the child component recognizes that its input properties have changed. In reaction to this the child Custom Element fetches the chunk coresponding to its component. The child component in turn boots up by rehydrating on top its server generated nodes and the process continues.

This way we can have a progressive bootstrapping strategy that loads components only on demand.

This is just an illustration of one possible strategy of how rehydration of SSR nodes is useful - but with the current organization of components as individually loadable units we can now tune the load strategy that makes sense for the application. In the future configurable prefetch and load strategies will be added to the project.
