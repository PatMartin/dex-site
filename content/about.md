+++
date = "2017-04-08T13:37:53-05:00"
title = "about"
[menu.main]
  identifier = "about"
+++

# About Dex

Dex is ultimately destined to be a machine learning platform.  However, I am taking the scenic
route via data transformation and visualization first.  It's been a long and fulfilling journey so
far.

In this page, I will go into my personal motivations, objectives and delve into my view of the
world of data analysis. If this is TLDR; for you, I understand.

## Why Dex?

I have benefitted from the work of too many people to list. I want to give something back.

I spend countless hours working on Open Source Software in the hopes that my ideas fuel ideas in
smarter folks than myself which result in something which helps the world in some small way.

### Principles of Invention

I found Bret Victor's presentation on [Inventing on Principle](https://www.youtube.com/watch?v=PUv66718DII)
to be quite inspiring and life changing.

Brett challenges the audience to figure out what core principles they want to incorporate into
their life and then to use these principles drive their day to day decisions.

## The History Of Dex

### The Early Years

Dex was a program born out of frustration.  I was drowning in data and lacked tools which
would allow me to quickly make sense of it all.  At the time, I was serving as a Java performance
expert in an organization with hundreds of individual Java applications running on thousands upon
thousands of hosts and virtual machines.  I was frustrated at the peanut gallery approach of
"try this, try that" with no scientific basis for the changes.  It was a lot of senseless activity
disguised as progress.

About 90% of the time, performance tuning consisted of optimizing memory usage.  I needed something which gave me
deeper insight into how memory was really being used than the tools available at the time.

As a result, I wrote an inital version of Dex (on my own time and as an OSS project) specifically with Java memory
analysis in mind; and it worked great.  Previous tools would report coarse grained memory; with Dex, I could see how
individual spaces were being used, why a Full GC was being invoked (and it's not always because of Old Generation
growth).

I could also add regression lines to VMs and **I could reliably predict subtle memory leaks days in advance of any real production
outage**.  That's a really powerful aspect -- ask any statistician.

With Dex, I could triage 10 or more applications per day and quickly determine if suggested fixes were
working or not.  Typically, I could turn around most analysis within 15-30 minutes of receiving telemetry data.

I had succeeded in unlocking the power of a particular dataset and **I was hooked on the idea of
reliable data feeding a reliable model producing knowledge which drove actionable corrective
decisions**.

## Dex Today

Dex, at present, is a fairly capable data manipulation and visualization platform.  It's
not perfect, but it's constantly getting better and there are enough extension points which
allow users to extend it's capabilities as they see fit.

I am always open to suggestions on how I might improve the platform.

## Dex Tomorrow

My focus over the next couple of years will be to:

  * Fill in functional gaps.
  * Improve the visualization capabilities.
  * Document everything thorougly.
  * Ease the learning curve through examples and tutorials.
  * Slowly start adding some basic machine learning capabilities.