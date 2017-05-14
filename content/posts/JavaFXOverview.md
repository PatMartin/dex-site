+++
date = "2017-04-30T01:36:43-04:00"
title = "JavaFX Overview"
draft = true

[params]
  scripts = [
  "https://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.8.3/underscore-min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/raphael/2.2.7/raphael.min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/js-sequence-diagrams/1.0.6/sequence-diagram-min.js"
  ]

+++

# JavaFX Overview

I find that I often need to reference some aspect of JavaFX when discussing other aspects of
Dex, so I decided to put it all together (hopefully coherently) in one place.

## History

I am always fascinated by how things evolved.  Learning the history of a thing drives
a deeper understanding much like learning Latin helps one understand other languages
with greater depth and appreciation.

**Click Image For Live Version**

[![Java FX Timeline](/blog/2017/javafx_overview/javafx_timeline.png)](/blog/2017/javafx_overview/javafx_timeline.html)

### The Early Years

In the early years, JavaFX evolved organically as a project known as F3.

#### F3: Form Follows Function

JavaFX is rooted in a declarative Java scripting language called Form Follows Function, or
 F3 for short.  F3 was originally conceived and implemented in 2005 by Chris Oliver while
 working at SeeBeyone; a company developing business integration software.

Chris took a look at the state of rich UI frameworks such as Adobe Flash. Adobe Flash was based
around XML and a dialect of JavaScript known as ActionScript.

Chris had strong feelings against using XML should not be used as a programming language, but
borrowing concept from Flash; F3 was born.

F3 provided Netbeans and Eclipse plugins which provided features such as as-you-type
valiation, code-completion, syntax highlithing and hyperlink navigation.  It also supported Swing,
Java2D.  Additionally, F3 could translate most of the SVG specification
into itself.

F3 allowed for the GUI to be expressed as a script.  Here is an example:

```javascript
import f3.ui.\*;
import f3.ui.canvas.\*;
import f3.ui.filter.\*;

Canvas {
    content: Text {
        x: 20
        y: 20
        content: "Welcome to F3"
        font: Font { face: VERDANA, style: [ITALIC, BOLD], size: 80 }
        fill: LinearGradient {
            x1: 0, y1: 0, x2: 0, y2: 1
            stops:
            [Stop {
                offset: 0
                color: blue
            },
            Stop {
                offset: 0.5
                color: dodgerblue
            },
            Stop {
                offset: 1
                color: blue
            }]
        }
        filter: [Glow {amount: 0.1}, Noise {monochrome: true, distribution: 0}]
    }
}
```

Which would yield:

![Welcome to F3](/blog/2017/javafx_overview/f3_demo.png)

It's strangely reminiscent of JSON (JavaScript Object Notation).

#### SUN Microsystems Acquires SeeBeyond

SeeBeyond was acquired by SUN Microsystems in September 2005 for $387 million USD in
a bid to promote their business integration capabilities.

While initially SUN seemed uninterested, this acquisition eventually helped bring introduce F3 to
innovators such as James Gosling; the father of Java itself.

**On November 21, 2006 James writes:**

<div class="bs-callout bs-callout-info">
<i>"If you like scripting, you should take a look at Chris Oliver's weblog. He mostly discusses a scripting
 language that he's been working on called F3. It's mind-bendingly cool. It's a client-side scripting
  engine, rather than one of the usual server-side languages. It's designed for doing very graphically
  cool applications. He's got lots of demos in is blog, mostly directly runnable. In particular,
  you really must run through his interactive tutorial. The most mind-bendingly cool feature
 (and a little tricky to understand) is F3's dependency-based evaluation. Magic.
  Have some fun with it."</i>
</div>

With evangelists such as James Gosling, good things were to follow.

#### JavaFX 1.x

On December 4, 2008, JavaFX officially releases Java FX 1.0.2.  JavaFX retains it's scripting focus through
the last of the 1.x releases; JavaFX 1.3.1 released on August 21, 2010.

During this time, JavaFX expands it's target platform base, adds many built in controls, layouts and charts.  CSS
controls, mobile support and many performance improvments are also added.

#### JavaFX 2.x

Java FX 2.0 is released on October 10, 2011.  Chris Oliver leaves Oracle in June 2011.  JavaFX, however,
has a bright future.

Most notable changes in the 2.x focus are:

  * JavaFX scripting is dropped in favor of a traditional Java API interface.
  * JavaFX mobile is dropped.
  * Oracle announces it's intent to Open Source JavaFX.
  * The underlying architecture is altered to leverage native, platform specific capabilities.
  Giving JavaFX applications near-native GUI performance.
  * FXML is introduced.

Incremental improvements occurred in point releases up to JavaFX 2.2 released on August 14, 2012.

#### JavaFX 8

In this release, JavaFX developers such as myself breathe a sigh of relief as JavaFX released sa part of Java.
No longer are our users required to download separate packages to enable JavaFX.

## Architecture

Ok, now lets delve into JavaFX architecture.  The following diagram depicts the main components at
a very high level.

![JavaFX Architecture](/blog/2017/headless_javafx/javafx_architecture.png)

### Scene Graph

### Graphics System

  * **Prism** - Processes render jobs.  It's Java's abstraction on top of things like DirectX,
  OpenGL, Java2D, etc...  This native accelleration is one of the reasons that JavaFX gives a
  much better user experience than Swing applications.
  * **Quantum Toolkit** - Ties Prism and Glass Windowing toolkit together and manages threading
  rules related to rendering and event handling.

### Windowing Tookit

#### Glass Windowing Toolkit

  * **Glass** - Glass is the lowest level in the JavaFX graphic stack.  Glass leverages the native
  platform for windowing and provides it's own resources when the native platform falls short. While
  AWT manages it's own event queue, Glass leverages the native event queue to schedule event usage.

  * **Web Engine** - The Web Engine is a full webkit based browser exposed as a JavaFX UI control
  which allows JavaFX to provide full web browsing capability.


<script>
$(".diagram").sequenceDiagram({theme: 'simple'});
</script>