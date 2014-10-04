# FRP Zoo

Interested in trying FRP, but overwhelmed by the number of FRP libraries to choose from? To help you with this choice, this repository contains several implementations of the same small program, using various implementations of FRP.

![A window with 6 buttons labelled "0", "5", "10", "toggle", "toggle", and "toggle".](toy-app-thumbnail.png)

library | classification | example app
:---:|:---:|:---:
— | no FRP, callbacks | [code](callback-example/Main.hs)
— | no FRP, pure functions | [code](gloss-example/Main.hs)
[ordrea](https://hackage.haskell.org/package/ordrea) | scenario 0, high-order, behaviours and events | [code](ordrea-example/Main.hs)
[reactive-banana](https://hackage.haskell.org/package/reactive-banana) | scenario 0, high-order, behaviours and events | [code](reactive-banana-example/Main.hs)
[sodium](https://hackage.haskell.org/package/sodium) | all three scenarios, high-order, behaviours and events | [code](sodium-example/Main.hs)
[Animas](https://hackage.haskell.org/package/Animas) | untested | —
[artery](https://hackage.haskell.org/package/artery) | untested | —
[bot](https://hackage.haskell.org/package/bot) | untested | —
[buster](https://hackage.haskell.org/package/buster) | untested | —
[definitive-reactive](https://hackage.haskell.org/package/definitive-reactive) | untested | —
[Dflow](https://hackage.haskell.org/package/Dflow) | untested | -
[drClickOn](https://hackage.haskell.org/package/drClickOn) | untested | —
[DysFRP](https://hackage.haskell.org/package/DysFRP) | untested | —
[elerea](https://hackage.haskell.org/package/elerea) | untested | —
[grapefruit](https://hackage.haskell.org/package/grapefruit-frp) | untested | —
[helm](https://hackage.haskell.org/package/helm) | untested | —
[io-reactive](https://hackage.haskell.org/package/io-reactive) | untested | —
[Moe](https://hackage.haskell.org/package/Moe) | untested | —
[netwire](https://hackage.haskell.org/package/netwire) | untested | —
[peakachu](https://hackage.haskell.org/package/peakachu) | untested | —
[reaction-logic](https://hackage.haskell.org/package/reaction-logic) | untested | —
[reactive](https://hackage.haskell.org/package/reactive) | untested | —
[reactive-bacon](https://hackage.haskell.org/package/reactive-bacon) | untested | —
[reactive-haskell](https://hackage.haskell.org/package/reactive-haskell) | untested | —
[reactive-thread](https://hackage.haskell.org/package/reactive-thread) | untested | —
[reenact](https://hackage.haskell.org/package/reenact) | untested | —
[rsagl-frp](https://hackage.haskell.org/package/rsagl-frp) | untested | —
[RxHaskell](https://hackage.haskell.org/package/RxHaskell) | untested | —
[spice](https://hackage.haskell.org/package/spice) | untested | —
[wxFruit](https://hackage.haskell.org/package/wxFruit) | untested | —
[Yampa](https://hackage.haskell.org/package/Yampa) | untested | —


## The TodoMVC of FRP libraries

If you haven't heard of [TodoMVC](http://todomvc.com/), it's a website which hosts several implementation of the same simple Todo app, implemented using various Javascript frameworks and languages. This repository attempts to do the same thing, but with Haskell FRP libraries.

Instead of a Todo app, we use a small task which highlights the area in which FRP libraries differ the most: support for dynamic signal graphs.

### Three scenarios

Evan Czaplicki gave an excellent [presentation about the different categories of FRP libraries](https://www.youtube.com/watch?v=Agu6jipKfYw), in which he categorized FRP libraries according to the choices they make in their attempt to support dynamic graphs. He described the following example:

1. Start with a graph which counts the number of clicks.
1. Click 5 times.
1. Change the graph so that it ignores the clicks.
1. Click 5 more times.
1. Change the graph back to the original graph.

Which number is displayed now? 0, because the new click counter hasn't received any click yet? 5, because the clicks did not reach the click counter while it was outside the graph? 10, because equational reasoning somehow dictates it?

For our toy program, we implement all three scenarios.

## Specification

A [gloss](gloss.ouroborus.net) window shall display six buttons, organized as three columns of two buttons. Each column implements one of the above scenarios: the first column chooses 0, the second column chooses 5, and the third column chooses 10. In each column, there is a button whose clicks are counted, and a toggle button to turn the click-counting on and off, starting with on. When off, the counter displays -1.

To be completely precise about the three scenarios:

* The counter for scenario 0 displays the number of clicks since the launch of the app, or since the corresponding toggle button was last toggled on, whichever is most recent.
* The counter for scenario 5 displays the total number of clicks which were received while the corresponding toggle was on.
* The counter for scenario 10 displays the total number of clicks since the launch of the app.

### Static implementation

In order to compare equivalent features in each library, the first half of the implementation should implement the required behaviour using only the core FRP features which are common to all FRP libraries: filtering events, combining current values, and accumulating changes to a local state.

To be completely precise about the expected static graph:

* Each toggle button accumulates `not` operations on a boolean, to establish whether the mode is currently on or off.
* This boolean is used to filter the click events which reach the counter for scenario 5.
* The counters for scenarios 5 and 10 accumulate `(+1)` operations on an integer.
* The counter for scenario 0 accumulates `const 0` and `(+1)` operations, depending on whether the incoming event is a toggle or a click.
* The current counters and modes are combined so that `-1` is used when the mode is off.

### Dynamic implementation

In order to compare the parts of the libraries which differ from each other, this second half of the implementation should reimplement some of the scenarios from part one using dynamic graph modifications, when appropriate. Libraries which only support static graphs should skip this section (I hasten to point out that focusing on static graphs is a perfectly valid point in the design space, as explained in Evan's presentation).

For libraries which do support graph modifications, it is likely that only one of the scenarios can be implemented: temporarily remove the counter from the graph while the toggle is off, then try out the application to see which scenario occurs. In the unlikely case of a library which supports more than one way to modify its graph, more than one scenario might be implementable.

Due to the variability, I cannot give a more precise description of the task, but I do want to point out a common trap: don't reimplement first-order primitives using the high-order ones. In particular:

* For scenario 5, it is tempting to switch between a graph which generates click events and a graph which doesn't, but this is simply a filter reimplementation. Instead, demonstrate that the events don't reach the counter when it is outside the graph, by switching between two counters for example.
* For scenario 10, it is tempting to separately define a counter which counts all clicks and then to switch between this counter and -1, but this is simply combining the current values of existing signals. Instead, demonstrate that events are recorded and replayed, by hiding the definition of your counter inside a conditional for example.

### Gloss integration

Since the goal is to compare FRP implementations, not GUI systems, a simple implementation of the 6 buttons GUI is provided, along with methods to determine whether the current event is a click or a toggle. Gloss supports both IO-based and state-based APIs, which should make it easy to hook up any FRP library. See the existing implementations for details.

## Classification

Evan's presentation classifies FRP libraries into four categories according to the choices they make regarding dynamic graphs. In our list of implementations at the top of this page, we tag each library with the category it belongs to, as well as the scenarios it can implement via dynamic graph changes. There are also other important distinctions between libraries which have nothing to do with dynamic graphs, whose corresponding tags are described in this section.

* First-order FRP: from Evan's classification, an FRP library which only supports static graphs.
* High-order FRP: from Evan's classification, an FRP library in which event streams are infinite and the graphs can be changed in a way which typically matches scenario 10.
* Asynchronous data flow: from Evan's classification, an FRP library in which fast event-processing nodes may receive more recent events than their slower neighbours. Some versions of this model support "cold" signals, in which the event processing is skipped if nobody is listening for the results.
* Arrowized FRP: from Evan's classification, an FRP library in which graph nodes are automatons which may or may not tick each frame, depending on whether or not they are currently part of the graph. Best for scenario 5.
* Signals: an FRP library in which there is only one kind or reactive object: signals.
* Events and behaviours: an FRP library in which there are two kinds of reactive objects: events and behaviours.
* Continuous: an FRP library in the style of Conal Elliott, meaning that signals are functions from time to values. This built-in notion of time allows interpolation between values, and other time-based transformations.


## Contributing

If the example app for your favourite FRP library is missing or non-idiomatic, or if there are other axes of comparison which you think should also be considered, feel free to open an issue or to send a pull request!
