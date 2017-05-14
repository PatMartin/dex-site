+++

date = "2017-05-05T12:22:18-05:00"
title = "Data Driven UI : JSON GUI"
type = "posts"
[menu.main]
  identifier = "posts"

[params]
  scripts = [
  "https://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.8.3/underscore-min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/raphael/2.2.7/raphael.min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/js-sequence-diagrams/1.0.6/sequence-diagram-min.js"
  ]
+++

# Data Driven UI

This article will describe the design and thought process around a
platform agnostic GUI specification language.  In this post I will:

  1. Introduce a rough draft of a JSON based GUI specification language and
  the thought process that has gone into it so far.
  2. Provide a couple of JSON GUI definitions with screenshots of their results.
  3. Demonstrate a renderer implementation written in Java using JavaFX
  and it's internal WebView. (Web/JQueryUI implementation to come later)
  4. Introduce a JavaFX component with a demo I am calling the JsonGuiPane.
  5. [Provide access to the code via github here.](https://github.com/PatMartin/json-gui).

Here is the basic flow of the JSON GUI demo:

<div class="diagram">
User -> JsonGuiApp : User Launches Demo App
JsonGuiApp -> JsonGuiPane : instantiate
JsonGuiApp -> WebView : instantiate
User -> JsonGuiApp : Load HTML
JsonGuiApp -> WebView : Load HTML
WebView -> JsonGuiApp : Document Loaded
JsonGuiApp -> WebView : getGuiDefinition()
WebView -> JsonGuiApp : JSON Gui Definition String
JsonGuiApp -> JsonGuiPane : JSON Gui Definition String
JsonGuiPane -> JsonGuiPane : instantiate gui
User -> JsonGuiPane : User interacts with GUI
JsonGuiPane -> JsonGuiApp : JsonGuiEvent event
JsonGuiPane -> WebView : setValue(event.target, event.value)
Note right of WebView : HTML chart is updated.
</div>

## Background

[Larry Wall](http://www.wall.org/~larry/)[^1], inventor of [Perl](https://www.perl.org/) once stated that the three great
virtues of a programmer are:

Laziness
: The quality that makes you go to great effort to reduce overall energy expenditure.
It makes you write labor-saving programs that other people will find useful and
document what you wrote so you don't have to answer so many questions about it.

Impatience
: The anger you feel when the computer is being lazy. This makes you write programs
that don't just react to your needs, but actually anticipate them. Or at least pretend to.

Hubris
: The quality that makes you write (and maintain) programs that other people
won't want to say bad things about.

I couldn't agree more.  We need more Larry Walls in this world.

So, after getting inspiration from a discussion with one of my forward
thinking colleagues (Pavel Vlasov) and with enough hubris to think it
wasn’t hubris, I tapped into my inner laziness and impatience and
started thinking about how great it would be if my GUI code could
simply generate itself automatically.

I started to wonder why I had to write configuration interfaces into my
visualizations over and over again. For each chart and variant, I end up
coding one interface to configure stuff from the web, and another
interface to configure the same chart dynamically from JavaFX.  Sure, it's
easy enough work, but so mundane and repetitive.  What's worse, the
configuration code is separate from the components being configured and
tend to get out of sync -- creating fragility.

Here’s an [example of what I am talking about](http://dexvis.net/vis/blog/2013/feb/mobilechord.html).
It’s functional, but ugly. I don’t have enough time to give single
components as much love as I would like.

Why can’t that lazy computer do all of this for me instead? Furthermore,
why can’t I specify the interface configuration within the component
itself using a loose format such as JSON and then provide this metadata
to different rendering engines such as JavaFX as well as Web driven GUIs?

Sure, I’d have to code both engines, but I would only have to code each
target engine once rather than every time a new visual component is
produced. The ultimate act of laziness!!! Better yet, the interfaces
would be cookie cutter consistent and this itself would lower the
learning curve for users.

[^1]: Warning!  His home page background color will burn your eyes.  It's just that hideous.

## So I got to thinking...

So I asked myself a few questions.

### What should the format be?

XML? JSON? Name Value Pair? Something else?

I quickly ruled out inventing my own format, and frankly couldn’t think
of any viable “something elses”, so it came down to JSON vs XML.

In spite of the rigor and rich ecosystem of XML[^2]; the world has embraced
JSON. While I may question the decision, it’s our current reality. XML
processing in Javascript is painful at best. JSON processing in Java is
easy enough and within Javascript understands JSON implicitly. So I
arrived at JSON being the specification format for this data driven UI.

[^2]: I am still reeling over the oscillations of the industry between spartan data representations such as associative arrays AKA "maps" over to a full featured, but verbose data representation such as XML, only to move back to a minimalistic data representation like JSON.  (Which to me is basically a map).  Such a fickle industry; but I go with the flow.

### Agonstic or opinionated?

Should the GUI spec remain agnostic, or provide a very tightly specified
GUI specification?

So I thought about the controls available within most web based GUI’s
as well as JavaFX, Swing, GTK and a few other toolkits I am familiar
with and came to the conclusion that a less opinionated specification
would be much easier to work with and give others the freedom to
innovate their own interpretations. The rendering engine, after all,
should be decoupled.  Besides, KISS[^3] is an often overlooked
design principle.

[^3]: **KISS** = Keep It Simple Stupid

### What should be represented?

So here I visited the various things I typically configure. During
“family time”, I mindmapped[^4] the following diagram while giving
perfunctory answers from my wife about the level of attention I was
paying to the show we were watching. Here’s what I was able to come
up with:

![JSON GUI Mindmap](/blog/2017/json-gui/json_gui_mindmap.png)

[^4]: Mindmapping with an iPad (I use SimpleMind) is a great design
exercise.  It keeps me from forgetting things and helps add robustness
to otherwise fragile and fleeting ideas.

#### Common Configuration

Each component's gui specification would support the following common
configuration options:

| OPTION | REQUIRED? | DESCRIPTION |
|--------|-----------|-------------|
| **name** | YES | A human readable name to be used in labels. |
| **description** | NO | An optional description for tooltips and such. |
| **target** | NO | The target of the value of the component. |
| **initialValue** | NO | The initial value. |

#### Primitives

The following components would offer additional relevant metadata.

##### Strings

| OPTION | REQUIRED | DESCRIPTION |
|--------|-----------|-------------|
| **minLength** | NO | The minimum length of the string. |
| **maxLength** | NO | The maximum length of the string. |

##### Passwords

This component is the same as a string, only considered confidential
and the renderer may choose to obscure the display to the screen.

| OPTION | REQUIRED | DESCRIPTION |
|--------|-----------|-------------|
| **minLength** | NO | The minimum length of the string. |
| **maxLength** | NO | The maximum length of the string. |

##### Integers

| OPTION | REQUIRED | DESCRIPTION |
|--------|-----------|-------------|
| **min** | NO | The minimum value of the integer. |
| **max** | NO | The maximum value of the integer. |
| **step** | NO | The increment for a component such as a slider. |

##### Floats

| OPTION | REQUIRED | DESCRIPTION |
|--------|-----------|-------------|
| **min** | NO | The minimum value of the floating point number. |
| **max** | NO | The maximum value of the floating point number. |
| **step** | NO | The increment for a component such as a slider. |

##### Boolean

There are no additional options for a boolean.

##### Color

There are no additional options for a color.

##### Choice

This represents an constrained choice from which the user may select one and
only one of the given choices.

| OPTION | REQUIRED | DESCRIPTION |
|--------|-----------|-------------|
| type   | NO | The type of the option.  Assumed to be categorical (string) if not supplied. |
| choices | YES | An array of choices from which the user may pick. |

##### Multiple Choice

This represents an constrained choice from which the user may select one
or more of the given choices.

| OPTION | REQUIRED | DESCRIPTION |
|--------|-----------|-------------|
| type   | NO | The type of the option.  Assumed to be categorical (string) if not supplied. |
| choices | YES | An array of choices from which the user may pick. |

#### Group

The group component is a suggested grouping for the interface components.  The
renderer may choose to represent this as a tab, accordion, whatever.

A group has no additional information.

#### Composites

Composites are cohesive groups of component definitions similar in nature
to a macro.  They simply help keep us from repetitively defining the same
tedious things over and over.

## JSON GUI Examples

So after turning thoughts into design, then design into implementation, I have
a concrete implementation.  Here are a few examples:

### A Simple Example

```javascript
var config = [
  {
    "type": "group",
    "name": "Chart Dimensions",
    "contents" :
    [
      {
        "name" : "Height",
        "description" : "The height of the chart.",
        "target" : "height",
        "type" : "int",
        "minValue" : 0,
        "maxValue" : 2000,
        "initialValue" : 600
      },
      {
        "name" : "Width",
        "description" : "The width of the chart.",
        "target" : "width",
        "type" : "int",
        "minValue" : 0,
        "maxValue" : 2000,
        "initialValue" : 800
	  }
	]
  }
]
```

Which yields the following configuration GUI:

![Configuration GUI](/blog/2017/json-gui/json_gui_app1.png)

When a user interacts with the sliders, the chord diagram depicted will
change dynamically.

### A More Complex Example

This screenshot shows the user dynamically selecting the stroke color of
a dendrogram's links.  Also note that this example has nested groups.

![Configuration GUI](/blog/2017/json-gui/json_gui_app2.png)

## JavaFX Code Overview

In this section, I will hit the highpoints of the JavaFX code.

### JsonGui

The JsonGuiPane is really just a souped up MigPane Layout with a few
extra routines tacked on.

Here we instantiate one:

```java
JsonGuiPane jsonGui = new JsonGuiPane("", "[grow]", "[grow]");
```

Here we get the gui definition from a web engined creatively named "we".
This occurs once the document is loaded, which is why it's located
within an load worker event listener.

We get the JSON string GUI definition by executing the function named
"getGuiDefinition();" then sending the results into the JsonGuiPane's
setGuiDefinition method.  This will dynamically parse the JSON and
create the specified GUI components.

```java
    we.getLoadWorker().stateProperty().addListener(new ChangeListener<State>()
    {
      public void changed(ObservableValue ov, State oldState, State newState)
      {
        if (newState == Worker.State.SUCCEEDED)
        {
          String guiDefinition = (String) we
              .executeScript("getGuiDefinition();");
          jsonGui.setGuiDefinition(guiDefinition);
        }
      }
    });
```

Lastly, here we catch all JsonGui change events and communicate the changes
back to the WebView via a javascript call to "setValue()".

```java
jsonGui.addEventHandler(
  JsonGuiEvent.CHANGE_EVENT,
  event -> {
    we.executeScript("setValue(\"" + event.getPayload().getTarget()
      + "\",\"" + event.getPayload().getValue() + "\");");
  });
```

# Wrapping Up

Anyway, I hope this component proves as useful to you as I believe it will
be for me.

I plan to flesh everything out a bit more and incorporate the JSON GUI
language into the [dex.js](https://dexjs.net) javascript components
as well as all the other Dex visuals.

I believe that this strategy of specifying the GUI within the component
itself will pay off in higher quality GUIs with greater consistency,
fewer bugs and greater robustness.

I guess we'll see...

<script>
$(".diagram").sequenceDiagram({theme: 'simple'});
</script>