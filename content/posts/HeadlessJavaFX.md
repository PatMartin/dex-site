+++
date = "2017-04-30T01:36:43-04:00"
title = "Headless JavaFX"
draft = true

[params]
  scripts = [
  "https://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.8.3/underscore-min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/raphael/2.2.7/raphael.min.js",
    "https://cdnjs.cloudflare.com/ajax/libs/js-sequence-diagrams/1.0.6/sequence-diagram-min.js"
  ]

+++

# Headless JavaFX

In this post, I will discuss my approach to running Dex headless, or without a GUI.
"Huh? What? Wait a minute! Why run without a GUI?" you might wonder.  After all,
isn't JavaFX all about the GUI?  When would you ever want to run a UI driven
application without its user interface?

As it turns out, there are many common situations where one might want to run without
a display which I will cover in this post; then address them specifically with respect
to my OSS JavaFX application: Dex.  I will discuss my approach to the problem
in enough detail so that others can use it as a roadmap for their own purposes.

## Motivations

So why no GUI?

### Automation

Automation is the primary driver.  There are a number of useful scenarios for this
type of automation which I will discuss.  Some are specifically applicable to Dex,
some are not.

#### Command Line Deployment

Automated environments tend to have no GUI capabilities.

Many times, I use Dex to create workflows which perform some general purpose ETL
(Extract Transformation Load).  This ETL process may need to be executed at some
cadence (monthly, weekly, daily or even hourly).  Even though this is a simple
task, it is still cumbersome if a human being is required to open the Dex GUI,
load the project and execute it.

For example purposes, let's say the flow looks like:

<div class="diagram">
Title: Human Driven Workflow
User -> Dex : Start Dex
User -> Dex : Load Project
User -> Dex : Execute Project
Dex -> Load_Data : Execute Tasks
Load_Data -> Trim_Whitespace: Data
Trim_Whitespace->Populate_Database: Trimmed Data
</div>

While developing this workflow goes much smoother with a GUI, why do I need the
GUI if I simply want to execute the flow?  Since the flow requires no human
intervention, I don't.  Furthermore, it is unlikely
that I will perform the task at exactly the same time everyday. It is unlikely
that if I were to perform this repetitive task, say hundreds of times,
that I would not eventually make a mistake.


<div class="diagram">
Title: Automated
Scheduler->DexCLI: Execute Project
DexCLI -> Load_Data : Execute Tasks
Load_Data -> Trim_Whitespace: Data
Trim_Whitespace->Populate_Database: Trimmed Data
</div>

Once I set the project execution using a scheduling tool such as cron or quartz, things are
fully automated and I am free to spend my time elsewhere (like on the beach).

#### Testing and CI/CD

In today's world of Agile methodologies and CI/CD (Continuous Integration/Continuous Deployment),
we need to trim out the need for human intervention in our testing pipeline wherever and
whenever possible.  This includes GUI applications as well.

For JavaFX, there is an automated testing framework called [TestFX](https://github.com/TestFX/TestFX)
which allows the user to program user interface interactions using a Java.  If automated testing is
your primary reason for a headless mode for your JavaFX application, I recommend that you check it out.

### Embedded Devices

While not applicable to Dex, another common usecase for the headless environment is embedded
devices.  Since they have no graphics card, they have no GUI.

### Decoupled Architecture

Another reason one might wish to support a headless mode for applications would be to
decouple the frontend from the service layer.  With respect to Dex, this is causing me to
consider new tasks which might be suitable for deployment as servlets, RESTful APIs or
simply deployed to Docker.

## JavaFX Overview

Before digging into the headless setup, let's get a poor man's understanding of JavaFX and
it's basic architecture. JavaFX is the successor to Swing.  Originally developed by Chris
Oliver while working at SeeBeyond.  JavaFX, originally named F3 (Form Follows Function),
was intended to be a GUI scripting language which allowed deeper integration of people
into business processes.

Over time, this effort evolved into the JavaFX we know and love today.  Below is a diagram
of the JavaFX architecture:

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



## My Approach

### Monocle

First and foremost, we need to download Monocle.

Intially, I considered creating a new mode of operation for the existing Dex application, but
quickly discarded the idea because there was so much unnecessary setup of buttons, labels and
menus that the CLI application would never need.

```bash
-Dglass.platform=Monocle
-Dmonocle.platform=Headless
-Dprism.order=sw
-Dprism.text=t2k
-Dheadless.geometry="1600x1200-32"
```

```java
package com.dexvis.dex;

import java.io.File;
import java.util.List;
import java.util.concurrent.CompletionService;
import java.util.concurrent.ExecutorCompletionService;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

import javafx.application.Application;
import javafx.application.Platform;
import javafx.event.ActionEvent;
import javafx.scene.Scene;
import javafx.stage.Stage;

import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.CommandLineParser;
import org.apache.commons.cli.DefaultParser;
import org.apache.commons.cli.Options;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;
import org.tbee.javafx.scene.layout.MigPane;

import com.dexvis.dex.exception.DexException;
import com.dexvis.dex.wf.DexJob;
import com.dexvis.dex.wf.DexJobScheduler;
import com.dexvis.dex.wf.DexTaskState;
import com.dexvis.dex.wf.SerialJob;
import com.dexvis.javafx.scene.control.DexTaskItem;
import com.dexvis.util.ThreadUtil;

public class DexCLI extends Application
{
  // Thread factory for executing task serially.
  private final static BasicThreadFactory serialThreadFactory = new BasicThreadFactory.Builder()
      .namingPattern("Dex-Serial-Task-%d").daemon(true)
      .priority(Thread.MAX_PRIORITY).build();

  // Executor for executing task serially.
  public final static ExecutorService serialExecutor = Executors
      .newSingleThreadExecutor(serialThreadFactory);

  // Thread factory for concurrent task execution. Such task may not update the
  // UI.
  private final static BasicThreadFactory concurrentThreadFactory = new BasicThreadFactory.Builder()
      .namingPattern("Dex-Concurrent-Task-%d").daemon(true)
      .priority(Thread.MAX_PRIORITY).build();

  // Executor for parallel task execution.
  public final static ExecutorService concurrentExecutor = Executors
      .newFixedThreadPool(
          Math.max(1, Runtime.getRuntime().availableProcessors() - 1),
          concurrentThreadFactory);

  // Service for task completion notification.
  public final static CompletionService<Object> CCS = new ExecutorCompletionService(
      concurrentExecutor);

  // Main stage
  private Stage stage = null;
  // Main scene.
  private Scene scene;
  private static String[] arguments;

  private void init(Stage stage)
  {
    try
    {
      this.stage = stage;
      stage.setTitle("Data Explorer");
      MigPane rootLayout = new MigPane("", "[grow]", "[][grow]");
      scene = new Scene(rootLayout, 1600, 900);
      stage.setScene(scene);
    }
    catch(Exception ex)
    {
      ex.printStackTrace();
    }
  }

  public void start(Stage stage) throws Exception
  {
    init(stage);
    stage.show();
    Options options = new Options();
    options.addOption("p", "project", true, "The project to be run.");
    CommandLineParser parser = new DefaultParser();
    CommandLine cmd = parser.parse(options, arguments);
    System.out.println("Running: " + cmd.getOptionValue("project"));
    DexProject project = DexProject.readProject(stage,
        new File(cmd.getOptionValue("project")));
    List<DexTaskItem> tasks = project.getTaskItems();

    int taskNum = 1;
    DexTaskState state = new DexTaskState();
    long projectStartTime = System.currentTimeMillis();
    long taskStartTime;
    for (DexTaskItem task : tasks)
    {
      taskStartTime = System.currentTimeMillis();
      System.out.println("  TASK[" + taskNum + "]: '"
          + task.getName().getValue() + "'");
      //long startTime = System.currentTimeMillis();
      if (task.getActive().getValue())
      {
        state = task.getTask().getValue().execute(state);
      }
      System.out.println("    " + (System.currentTimeMillis() - taskStartTime) + " ms");
      taskNum++;
    }
    serialExecutor.shutdown();
    concurrentExecutor.shutdown();
    if (!serialExecutor.awaitTermination(3600, TimeUnit.SECONDS))
    {
      serialExecutor.shutdownNow();
    }
    if (!concurrentExecutor.awaitTermination(3600, TimeUnit.SECONDS))
    {
      concurrentExecutor.shutdownNow();
    }
    System.out.println("Excecution Completed In: " + (System.currentTimeMillis() - projectStartTime) + " ms");

    Platform.exit();
  }

  public void exit(ActionEvent evt)
  {
    System.exit(0);
  }

  public static void main(String[] args)
  {
    arguments = args;
    Platform.setImplicitExit(false);
    launch(args);
  }
}
```

<script>
$(".diagram").sequenceDiagram({theme: 'simple'});
</script>