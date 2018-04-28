<!-- The short-comings of other solutions are somewhat a continuation of the problem description -->

<!-- * things from the knowledge-base: redux, flux, angular, meteor, linked-data -->
<!--    * instead of in problem-description (more abstract there) -->
\chapter{State of the Art}

\section{Frameworks and Architecture}

<!-- TODO compare/synthesize with http://staltz.com/unidirectional-user-interface-architectures.html -->

\subsection{Model-View-Controller}\label{ref:mvc}

You probably already are familiar with the classical model-view-controller architecture, but for the sake of completeness a short overview will be given here. The pattern mainly consists of three types of building blocks (as can also be seen in figure \ref{fig:mvc}): <!--TODO {TODO sources}-->

\begin{description}
  \item[controllers] contain the lion's share of the business logic. User input gets handled by them and they get to query the model. Depending on these two information sources they decide what messages to send to the the model, i.e. the controller telling the model to change. Usually there's one controller per view and vice-versa.
  \item[models] hold the application's state and make sure it's consistent. If something in the data changes, it notifies views and controllers depending on it. These notifications can be parametrized, telling the dependants what changed.
  \item[views] are what the outside world/user's get to see. When the model changes, the view get's notified and---depending on the data passed along and what it reads from the model---updates accordingly.
  Especially in html-applications, views (and thereby controllers) tend to be nested (e.g. the entire screen -- a column -- a widget in it -- a button in the widget)
\end{description}

Note that there's a wide range of different instances/interpretations of the architectural patterns can organise models/views/controllers differently. Further down, in section \ref{ref:angular-mvc} you can find one of these (angular's MVC) described in more detail.

This is a reference ![image][someref] with multimarkdown attributes.

[someref]: figures/mvc.png "Image title" width=20px height=30px
       id=myId class="myClass1 myClass2"

![MVC-architecture (Krasner and Pope, 1988)](figures/mvc.png){#id .class width=30 height=20px}
<!-- TODO include again
\begin{figure*}
\centering
\includegraphics[width=1.0\textwidth]{figures/mvc.png}
\\cption[MVC-architecture]{MVC-architecture (Krasner and Pope, 1988)}
\label{fig:mvc}
\end{figure*} -->
<!--TODO { TODO krasner and pope 1988: http://heaveneverywhere.com/stp/PostScript/mvc.pdf }-->

\subsection{Model-View-ViewModel}\label{ref:mvvm}

This architectural pattern, also known as "Model-View-Binder" is similar to MVC but puts more emphasis on the seperation between back-end and front-end. It's parts are as follows (and can be seen in fig. \ref{fig:mvvm}):<!--TODO {TODO sources}-->

\begin{description}
  \item[The model] is the back-end business-logic and state. It can be on a different machine entirely, e.g. a web-server.
  \item[The view-model] contains the front-end logic and state. It is a thin binding layer, that processes inputs and that manages and provides the data required by the view.
  \item[The view] is a stateless rendering of the data retrieved from the view-model; in the case of some frameworks, this happens via declarative statements in the view's templates, that automatically get updated when the data in the view-model changes. User-input events raised in the view get forwarded to the view-model.
\end{description}

<!-- TODO include again
\begin{figure*}
\centering
\includegraphics[height=8cm]{figures/mvvm.pdf}
\\cption[MVC-architecture]{ MVVM-architecture (diagram \href{https://en.wikipedia.org/wiki/File:MVVMPattern.png}{adapted from wikimedia}\footnotemark{})}
\label{fig:mvvm}}
\end{figure*}
\footnotetext{\url{https://en.wikipedia.org/wiki/File:MVVMPattern.png}}
-->

\subsection{Angular 1.x MVC}\label{ref:angular-mvc}

<!--<!--TODO  {Too much detail! Move a lot of these details to later chapters (e.g. "solution » ng best practices" or "solution » why we moved from ng to ng-redux") -->

Angular 1.x is a javascript-framework that roughly follows the MVC/MVVM architectures, but has a few conceptual variations and extensions.

On the View-side of things there's templates (see fig. \ref{fig:ng-template} for an example from the webofneeds-codebase). These are either specified in an html-file and then later linked with a controller or are a string in the declaration of something called a "directive" (which are custom html tags or properties). Every template has a scope object bound to it and can contain expressions---e.g. those in curly braces---that have access to that scope object. For the example in fig. \ref{fig:ng-template} this means, that---in the HTML that the user gets to see---the curly braces will have been replaced by the result of \texttt{self.post.getIn(['won:hasContent','won:hasTextDescription'])} (the \texttt{getIn} is there because \texttt{post} is an [immutable-js](https://facebook.github.io/immutable-js/) object). Practically every time the result of that expression changes, angular will update the displayed value. Basically every expression causes a "watch"to be created (this can also be done manually via \texttt{\$scope.watch}). On every "digest-cycle""hecks all of these watch-expressions for changes and then executes their callbacks, which in the case of the curly-braces causes the DOM-update.

Beyond the curly braces, angular also provides a handful of other template-utilities in the form of directives. For instance the property-directive \texttt{ng-repeat} allows iterating over a collection as follows:

```html
<div ng-repeat="el in collection">{{el.someVar}}</div>
```

Or, similiarly, \texttt{ng-show="someBoolVar"} conditionally displays content.

Note that these template-bindings are bi-directional, i.e. the code in the template can change the the values in the scope. Additionally, templates/directives can be nested within each other. By default, their scopes then use javascript's [prototypical inheritance](https://developer.mozilla.org/en/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) mechanism, i.e. if a value can't be found on the template's/directive's scope, angular will then go on to try to get it from the on the one wrapping it (and so on)
This allows writing small apps or components where all data-flows are represented and all code contained in the template. For medium-sized or large apps however, the combination of bi-directional binding and scope inheritance, can lead to hard-to-follow causality, thus hard-to-track-down bugs and thus poor maintainability. More on that later. <!-- in section X -->
<!--TODO {TODO add reference to that subsection}--> 
Also, using scope inheritance reduces reusability, as the respective components won't work in other contexts any more. <!--TODO {move critique of bi-dir binding and inheritance to later chapter}-->

<!--TODO { TODO get syntax-highlighting to work in figures (see comment in .tex) }--> 
<!-- \begin{lstlisting}[style=json]} -->
```html
...
<h2 class="post-info__heading"
  ng-show="self.post.getIn(['won:hasContent','won:hasTextDescription'])">
    Description
</h2>
<p class="post-info__details"
  ng-show="self.post.getIn(['won:hasContent','won:hasTextDescription'])">
    {{ self.post.getIn(['won:hasContent','won:hasTextDescription']) }}
</p>
...
```
<!-- TODO include again
\\cption{Excerpt of angular-template}
\label{fig:ng-template}
\end{figure*}
-->

For all but the very smallest views/components the UI-update logic will be contained in angular's controllers, however. They are connected with their corresponding templates via the routing-configuration (more on that later <!--TODO {ref to routing-subsection}-->) or by being part of the same directive <!--TODO {ref to directive-subsection}-->. Controllers have access to their template's scope and vis versa <!--(see section X -->
<!--TODO {ref to controllerAs discussion}-->
<!--) -->
Theoretically it's possible to pair up / reuse controllers with different templates, but this can lead to hard-to-track-down and I'd advise against doing that.
When nesting templates and thus they're associated controllers, actually it's the controllers that form the prototypical inheritance chain. Thus, if a variable isn't found on the controller respectively it's scope, the default is to check on it's parent('s), up to the root-scope. Note that scopes can be defined as isolated in the routing config <!--TODO {ref to routing/isolated-scope section}--> to avoid this behaviour, which I'd recommend for predicatability- and thus maintainability-reasons.

<!--TODO { can be reused with different template, but that rarely happens and tends to lead to hard-to-track-down bugs.}-->
<!--TODO {nesting templates (not directives?)---how does it work anyway?}-->

<!--TODO {move controllerAs advice to later chapter}-->
<!--
Controllers have access to their template's scope via the variable \texttt{scope} that they get in their factory-function/constructor. 
Alternatively, they can be bound e.g. as \texttt{self} to the scope by specifying \texttt{controllerAs: "self"} in the routing-/directive-config. This avoids the situation where you specify a variable on the wrong object and then the template-expression can't find it (e.g. if you miss that \texttt{this} in a bound function points to the controller-object instead of the scope) Generally speaking, using \texttt{controllerAs} makes mistakes/bugs less likely. 
-->

Now, with models/scopes, views/templates and controllers we would have a classical MVC-framework (see section \ref{ref:mvc}). However, angular also has the concept of services: Essentially, they are objects that controllers can access and that can provide utility functions, manage global application state or make http-requests to a web-server. Controllers can't gain access to each other---except for nesting / prototypical inheritance---but they can always request access to any service (via dependency injection; more on
that later<!--TODO {ref to subsection}-->). Examples of services are \texttt{\$scope} that, amongst others, allows registering custom watch-expressions with angular outside of templates (see fig. \ref{fig:ng-simple-ctrl} for an example). Another example for a service would be our custom \texttt{linkeddata-service.js} that can be used to load and cache RDF-data\footnote{see section \ref{data-on-won-nodes} for more on RDF}.


<!--TODO { TODO get syntax-highlighting to work in figures (see comment in .tex) }--> <!-- \begin{lstlisting}[style=javascript]} -->
```js
var myApp = angular.module('myApp', []);
myApp.controller('PostController', function ($scope) {
  $scope.post = { text: 'heio! :)' };
  $scope.$watch('post.text', function(currentText, prevText) {
    console.log('Text has been edited: ', currentText);
  });
});
```
<!-- TODO include again
\\cption{Example of a very simple controller and usage of the \texttt{\$scope}-service}
\label{fig:ng-simple-ctrl}
\end{figure*}
-->

With services added to the mix, we can also view the angular framework through the lense of MVVM (see section \ref{ref:mvvm}), with templates as views, scopes and controllers as view-models and services as models or as proxies for models on a web-server (as we did with the \texttt{linkeddata-service.js}).

<!-- <!--TODO { TODO get syntax-highlighting to work in figures (see comment in .tex) }--> 

```js
myApp.config(['$routeProvider',
  function($routeProvider) {
    $routeProvider.
      when('/landingpage?:focusSignup', {
        templateUrl: 'app/components/\
          landingpage/landingpage.html',
        controller: 'LandingpageController'
      }).
      when('/post/?postUri', {
        templateUrl: 'app/components/\
          post/post.html',
        controller: 'PostController'
      }).
      otherwise({
        redirectTo: '/landingpage'
      });
  }]);
```
<!-- TODO include again
\\cption{Example of routing-configuration in Angular 1.X}
\label{fig:ng-simple-routing}
\end{figure*}
-->

<!-- todo move this to later chapters (e.g. a section on module systems) -->

Note, that Angular 1.x uses it's own module system to manage directives, controllers and services. If you include all modules directly via `<script>`-tags in your `index.html`, this mechanism makes sure they're executed in the correct order. However, this also means, that if you want to combine all your scripts into one `bundle.js`[^
Bundling for instance helps to reduce the number of HTTP-requests on page-load and thus it's performance. It can be done by using a build-tool like browserify, webpack or jspm plus a module system like AMD, CommonJS or the recently standardized ES6-modules (see <http://www.ecma-international.org/ecma-262/6.0/#sec-imports>)]
}
you'll have to specify the same dependencies twice---once for your bundling module system and once for angular's, as can be seen in figure \ref{fig:ng-duplicate-dependencies}.<!--TODO {ref number doesn't match figure's}-->

```js
/* es6 imports for bundling */

import angular from 'angular'
import createNeedTitleBarModule from '../create-need-title-bar';
import posttypeSelectModule from '../posttype-select';

//...

class CreateNeedController { /* ... */ }

//...

/* angular module declaration */

export default angular.module(
  /* module's name: */
  'won.owner.components.createNeed', 
  [ /* module's dependencies: */
    createNeedTitleBarModule,
    posttypeSelectModule,
    // ...
  ])
  .controller(
    'CreateNeedController', 
    [
      '$q', '$ngRedux', '$scope', // services for ctrl
      CreateNeedController // controller factory/class
    ])
.name;
```
<!-- TODO include again
\\cption{Example of module- and controller-declaration in `create-need.js`}
\label{fig:ng-duplicate-dependencies}
-->

As you can see writing applications in angular requires quite a few concepts to get started (this section only contains the essentials, you can find a [full list in the angular documentation](https://docs.angularjs.org/guide/concepts)). Accordingly, the learning curve is rather steep, especially if you want to use the framework well and avoid a lot of the pitfalls for beginners, that otherwise result in hard to debug and unmaintainable code.

<!--TODO {TODO reference ng docu}-->

<!--
    * [ ] super long list of concepts https://docs.angularjs.org/guide/concepts

    * [ ] views - templates + controller
	* [ ] controllers / factory methods
        * [x] somewhere between MVVM-viewmodels and MVC-controllers
		* [ ] best practice: avoid putting too much code into these
	* [x] routing 
        * [x] how controllers+html-templates are brought together
    * [ ] directives (component style or attributes)
        * [ ] explain with example of modal?
        * [ ] preferable to view+controller as code for both + bundling of them is in one place (~react components)
    * [x] services ~MVVM-model
    * [x] arguably more mvvm than mvc
    * [x] rather steep learning curve. 
        * [x] especially to use it well. there's many pitfalls for people just starting out with it to produce a code-base that's hard to maintain later on. (personal experience with smartengine-code and own code)
        * [ ] TEXT?: as concrete usage example angular solves a wide range of different problems (routing, state managment, networking, display,...) and it does so with an equally wide range of mechanisms. Redux on the other hand, has one a very small number of mechanisms, that it uses to solve all these problems similiarly (e.g. routing-information is part of the redux-state)
        * [ ] TEXT: ... As you can see angular is rather complex in it's architecture and contains quite a few pitfalls for the unwary newcomer. As such, it has a rather steep learning curve.
    * [x] bi-directional binding
    * [ ] auto-injection into scope via things like ng-model
    * [x] scoping / hierarchy of controllers
    * [x] scope-inheritance and it's problems!
    * [ ] modules and dependency-injection
      * [x] need to include each and every javascript file (in the right order?). everything is loaded with quite a few http-requests. can be bundled though
        * make sure to use strict mode to allow bundling
      * [ ] no tree-shaking
      * [x] redundant to es6-module system
    * [x] services
      * [x] keep global state
      * [x] wrap access utilities to server-APIs
      * [x] access to utility functions that can be injected (instead of being attached to the window)
    * [ ] controllers
      * [x] both controllers and partly models (is angular actually mvvm?)
      * [x] can be reused with different template, but that rarely happens and tends to lead to hard-to-track-down bugs.
      * [ ] mostly used at view level. below that directives provide more atomic bundling of template and code.
    * [x] templates
    * [ ] filters
      * { probably not necessary to explain }
    * [x] directives
      * [x] bundle controller and template in one file
      * [x] register custom html-tag (unless used in attribute, like ng-show/-class/-..., or class mode)
    * [ ] routing
      * [ ] html-fragments
      * [ ] ui-router (?) (are we using or have we used it?)
-->

<!-- \subsection{Meteor} -->

<!-- TODO { dunno if necessary? }-->

\subsection{React}

React is a framework that specifically provides the view (and potentially view-model) of application architectures. It provides a mechanism to define custom components/HTML-tags (comparable to directives in Angular 1.X and webcomponents in general) as a means to achieve seperation of concerns and code reusability. These components are stateful and contain their own template code, usually specified in the form of inline-HTML (that's processed to calls to the React-libary---more on that
below). <!-- see $x / at the bottom of this section for an example of $y, were it written as React-component <! - - TODO take short directive from won-codebase and translate it to React -->
For all but the smallest applications---where the state can be fully contained in the components---you'll need some extra architecture additional to React, for instance to handle the application-state or manage HTTP-requests and websockets. This is usually where Flux (see section \ref{ref:flux}) and Redux (see section \ref{ref:redux}) come in.

In any way, to get to the bottom of what distinguishes React, one should first start by talking about the big problem of the Document Object Model: When there's a large number of nodes on the screen, manipulating several quickly one after each other can take quite a while, causing the whole interface to noticeably lag as every changed node causes a reflow of the layout and rerendering of the interface. React is the first of a row of libraries to use a light-weight copy of the DOM (called "Virtual DOM". The idea is to only directly manipulate the VDOM and then apply
the differential / cumulative change-set to the actual DOM in one go. This means a performance gain where multiple operations are applied to the same node or multiple nodes at the same time as React makes sure that the slow reflow and rerendering only happens once. From a development side of things, this diff'ing-process means, that there's no need to manage DOM-state-changes and intermediate states; the template-code in the components can be written, as if they were rendered completely new every cycle, i.e. only a direct
mapping from data to desired HTML needs to be provided and React handles the changes to get there.

As a notable difference to Angular, React's data-flow is unidirectional, meaning a component can read the data it gets via it's html-tag-properties, but it can't modify them. This is a useful guarantee, to avoid bugs like when you use a component, don't know it modifies it's parameter variables (intentionally or as a bug) and thus influences your unsuspecting parent component as a side-effect. Intended child-to-parent communication can be done explicitly via events published by the child (or via callback functions). 

<!--
class Square extends React.Component {
  constructor() {
    super();
    this.state = {
      value: null,
    };
  }
  render() {
    return (
      <div className="square">
		{this.props.myproperty}
		{this.state.value}
      </div>
    );
  }
}
-->

\subsection{Flux}\label{ref:flux}

<!-- TODO include again
\begin{figure*}
\centering
    \includegraphics[width=1.0\textwidth]{figures/flux_simple.png}
    \\cption[Flux-pipe]{Core pipeline of the Flux-architecture (via \url{https://facebook.github.io/flux/img/flux-simple-f8-diagram-1300w.png)}}
\label{fig:flux_simple}
\end{figure*}
-->

When you start reading about React you'll probably stumple across Flux (see fig. \ref{fig:flux_simple}) rather earlier than later. It is the architecture popularized alongside of React and akin to MVC in that it seperates handling input, updating the state and displaying the latter.

However, instead of having bi-directional data-flow between the architectural components, Flux' is uni-directional and puts most of it's business logic into the stores that manage the state. To give an example of a flow through this loop: Say, a user clicks on a map widget with the intend of picking a location. The widget's on-click method, would then create an object that's called an action that usually contains type-field like \texttt{"PICK\_LOCATION"} and any other data describing the
user-interaction like geo-coordinates. It then goes on to pass the action object to the globally available dispatcher, that broadcasts it to all stores. Every store then decides for itself in what way it wants to update the data it holds. For instance, a \texttt{locationStore} could updated the geo-coordinates it holds. The stores would then go on to notify all components that are listening to them in particular that their state has changed (but not in what way). The affected
components, e.g. the map and a text-label below it, poll the store for the data and render themselves anew (as if it was the first time they were doing this)---e.g. the map would place a singular marker on the coordinates it gets from the store and the label would write out the coordinates as numbers.

Because of the last point---the components rendering themselves "from scratch" every time, i.e. them being an (ideally) state-less mapping from app-state to HTML---this architecture pairs well with React's VDOM.

When there's preprocessing that needs to be done on the data required for the action-object---e.g. we want to resolve the geo-coordinates to a human-friendly address-string---action-creators are the usual method to do so (see fig. \ref{fig:flux_full}). These are functions that do preprocessing---including HTTP-requests for instance---and then produce the action-objects and dispach them. 

Though being an architecture, i.e. a software-pattern,  per se, usually one will use one of many ready made dispatchers and also a store-prototype to inherit from, that will reduce the amount of boilerplate code necessary to bootstrap a Flux-based application.

Stores can have dependencies amongst each other. These are specified with a function along the lines of \texttt{B.waitFor(A)}, meaning that the store B only starts processing the action once A has finished doing so. Managing these dependencies in a medium-sized to large application can be quite complex, which is where Redux (see below) tries to improve over Flux.

In general, using Flux profits from using immutable data-structures for the state (e.g. those of [immutable-js](https://facebook.github.io/immutable-js/)). Without these, components could accidentally modify the app-state by changing fields on objects they get from the stores, thus having the potential for hard-to-track-down bugs.

<!-- TODO include again
\begin{figure*}
\centering
\includegraphics[width=1.0\textwidth]{figures/flux.png}
    \\cption[Flux-architecture]{Full Flux-architecture incl. networking (via \url{https://facebook.github.io/react/img/blog/flux-diagram.png})}
\label{fig:flux_full}
\end{figure*}
-->
 
<!--

  * [ ] quite a learning-curve
  * [x] actions
  * [x] dispatcher
    * [x] there are many different implementations of these (the one by fb, the one by yahoo,...)
  * [x] stores
    * [x] waitFor
    * [x] dependencies between these can be hard to understand
    * [ ] lot of overhead
  * [x] views
    * [ ] most dispatchers / setups are geared to be used with redux
-->


\subsection{Redux}\label{ref:redux}

<!-- TODO include again
\begin{figure*}
    \centering
<!--    \includegraphics[height=8cm]{figures/redux.pdf} -- >
    \includegraphics[width=1.0\textwidth]{figures/redux.pdf}
    \\cption[MVC-architecture]{Redux-architecture}
    \label{fig:redux}}
\end{figure*}
-->

The developers/designers of Redux list the object-oriented Flux- (see above) and functional Elm-architecture (see below) as [prior art](http://redux.js.org/docs/introduction/PriorArt.html). It mainly differs from Flux in eschewing the set of stateful stores, for the Elm-like solution of having a single object as app-state, that a single reducer-function \texttt{(state, action) => state'} gets applied to for every new action, thus updating the state (see fig. \ref{fig:redux}). As such there's also formally no need for a
dispatcher, as there's only a single function that's updating the state. Seperation of concerns---that Flux achieves with it's larger number of stores---can be achieved in Redux by having the reducer function call other functions, e.g. one per subobject/-tree of the state. 

As the simplest implementation of this architecture consists of only a single function and a component that feeds actions into it, the learning curve is relatively shallow compared to Flux and almost flat compared to Angular's MVC.

Redux profits from immutable data-structures for the app-state almost even more than Flux. The reducer function is supposed to be stateless and side-effect free (i.e. pure). In this particular case this means that parts of the system, that still hold references to the previous state, shouldn't be influenced by the state-update. If they want the new state, they'll get notified through their subscription. Using immutable data guarantees this side-effect freeness to some extend (nothing can
prevent you from accessing the global \texttt{window}-scope in javascript though, so ideally don't do that). This property also means, that you should try to move as much busieness logic as possible to the reducer, as it's comparatively easy to reason about and thus debug. For all things that require side-effects (e.g. anything asynchronous like networking) action-creators are the go-to solution---same as in Flux.


<!--
    <!-- TODO graphic -- >
  * [x] http://redux.js.org/
  * [x] can be super-simple (give trivial example)
  * [x] easy to learn (it's only one event-bus/dispatcher, one reduction-function)
  * [x] ideally used with immutable data for model (to avoid bugs due to pass-by-reference and later modification)
  * [x] doesn't deal with side-effects by default (see ACs and actors later)
  * [x] (action-creators (for pre-processing))
  * [x] actions
  * [x] dispatcher
  * [x] seperation of concerns by having subfunctions for reducer
  * [x] reduction function
    * [x]  synchronous (can't do asynch side-effects here)
    * [x] (supposed to be) side-effect free. do as much as business logic as possible here.
 * [x] components
-->

\subsection{Ng-Redux}\label{ref:ng-redux}

[Ng-Redux](https://github.com/angular-redux/ng-redux) is framework that's based on the Redux-architecture and is geared to be used with Angular applications. The latter then handles the Components/Directives and their updates of the DOM, whereas Ng-Redux manages the application state. In this combination, the frameworks binds functions to the angular controllers to trigger any of the available actions. Even more importantly, it allows registering a \texttt{selectFromState}-function that gets run after
the app-state has been updated and which' result is then bound to the controller. It also provides a plugin/middleware-system for plugins that provide convienient use of asynchronicity in action-creators (through "thunk" or keeping the routing information as part of the application state (through the "ngUiRouterMiddleware""

<!--
    <!-- TODO example of use in a simple directive? -- >
  * [ ] good for migrating (why we chose it)
  * [x] duplicate imports if using es6 (though not ng-redux inherent)
  * [ ] explain binding into routing, setup of watches, importance of one way bindings
  * [ ] has a dispatcher (what does it do? it should be super minimal)
-->


\subsection{Elm-Architecture}

<!-- TODO diagram -->

[Elm](http://elm-lang.org/) is a functional language who's designers set out to create something as accessible to newcomers as Python or Javascript. It can be used to build front-end web application (browser-less execution in node is currently being worked on). The original Elm-architecture was based on functional reactive programming---i.e. using streams/observables like CycleJS' MVI (see below) that it inspired as well---but they have since been removed to make it more accessible to
newcomers. The [current
architecture](https://guide.elm-lang.org/architecture/), in it's basic form, requires one to define the following three functions and pass them to Elm's \texttt{Html.beginnerProgram} (that runs the app):


\begin{enumerate}
    \item \texttt{model : Model}, that initializes the app-state.
    \item \texttt{update : Msg -> Model -> Model}, which performs the same role as \texttt{reduce} in Redux, with \texttt{Msg}s in Elm being the equivalent to actions in Redux.
    \item And lastly, \texttt{view : Model -> Html Msg} to produce the HTML from the model.
\end{enumerate}

As Elm is a pure/side-effect free language, these can't handle asynchronity yet (e.g. HTTP-requests, websockets) or even produce random numbers. The full architecture, that handles these, looks as follows (and is run via \texttt{Html.program}):

\begin{enumerate}
    \item \texttt{init : (Model, Cmd Msg)} fulfills the same role as \texttt{model}, but also defines the first \texttt{Cmd}. These allow \textit{requesting} for side-effectful computations like asynchronous operations (e.g. HTTP-requests) or random number generation. The result of the \texttt{Cmd} is fed back as \texttt{Msg} to the next \texttt{update}.
    \item the function \texttt{update : Msg -> Model -> (Model, Cmd Msg)} now also returns a \texttt{Cmd} to allow triggering these depending on user input or the results of previous \texttt{Cmd}s. This allows keeping all of the business-logic in the \texttt{update}-function (as compared to Flux'/Redux' action-creators) but trades off the quality, that every user-input or websocket message can only trigger exactly one action and thus exactly one update (thus making endless-loops
        possible again)---arguably this is a rather neglible price.
    \item \texttt{subscriptions : Model -> Sub Msg} allows to set up additional sources for \texttt{Msg}s beside user-input, things that \textit{push}---if you so will---
e.g. listening on a websocket.
    \item \texttt{view : Model -> Html Msg} works the same as in the simple variant.
\end{enumerate}

<!--
    <!-- TODO snippet / pic of previous  -- >
    * [x] previous (at time of designing)
    * [x] current
-->

\subsection{CycleJS MVI}

<!-- TODO diagram -->

CycleJS is an 'functional reactive programming'-based framework, which Model-View-Intent architecture is structured similar to the Redux- and (original) Elm-architectures. 

But first: The framework itself is based on functional reactive programming (FRP) and uses observables/streams of messages for it's internal data-flows. Think of them as Promises that can trigger multiple times, or even more abstract, pipes that manipulate data that flows through them and that can be composed to form a larger system. The integral part developer's using the framework need to specify is a function \texttt{main(sources) => ({ DOM: htmlStream})} (see fig. \ref{fig:cyclejs}) that takes a driver "\texttt{sources}" like the DOM-driver that allows creating stream-sources (e.g. click events on a button). One would then apply any data-manipulations in the function and return a stream of virtual DOM. In the very simple example of fig. \ref{fig:cyclejs} for every input-event a piece of data/message would travel down the chained functions and end up as a virtual DOM object. This \texttt{main}-function is passed to \texttt{run} to start the app.

<!-- TODO instead rewrite one of our components as example here. -->
<!-- TODO syntax highlighting -->
```js
import {run} from '@cycle/xstream-run';
import {div, label, input, hr, h1, makeDOMDriver} from '@cycle/dom';

function main(sources) {
  const sinks = {
    DOM: sources.DOM.select('.field').events('input')
      .map(ev => ev.target.value) // get text from field
      .startWith(" // initial value / first stream-message
      .map(name =>
        div([
          label('Name:'),
          input('.field', {attrs: {type: 'text'}}),
          hr(),
          h1('Hello ' + name),
        ])
      )
  };
  return sinks;
}

run(main, { DOM: makeDOMDriver('#app-container') });
```
<!-- TODO include again
\\cption{CycleJS hello-world example from \url{https://cycle.js.org/}}
\label{fig:cyclejs}
\end{figure*}
-->

For more complex applications, an architecture similiar to Redux/Elm, called "Model-View-Intent" is recommended. For this, the stream in \texttt{main} is split into three consecutive sections: 

\begin{enumerate}
\item Intent-functions that set up the input streams from event-sources (e.g. DOM and websockets) and return "intents" that are equivalent to Flux'/Redux' actions and Elm's messages.
\item The model-stage is usually implemented as a function that is \texttt{reduce}'d over the model (equivalent to how Redux deals with state-updates)
\item And lastly the view-stage takes the entire model and produces VDOM-messages.
\end{enumerate}

Seperation of concerns happens by using sub-functions or splitting the stream at each stage (or starting with several sources in the first) and combining them at the end of it.

<!--
* [ ] driver's are similiar to actors?
-->