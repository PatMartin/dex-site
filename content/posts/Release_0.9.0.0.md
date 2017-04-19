+++
date = "2017-04-17T20:41:16-04:00"
title = "Dex Release 0.9"

+++

# Dex 0.9 Released

The [release for Dex 0.9 can be found here](https://github.com/PatMartin/DexReleases/blob/master/dex-0.9.0.0.zip?raw=true).
To reduce my personal bandwidth charges, I'll be posting the releases to github.  Starting with version 0.9, you will be able
to access the releases there, or by clicking the Release section on the top navbar of the screen.

## Release Nodes

I've been focused on shoring up the foundation project [dex.js](https://dexjs.net) for the past few months, so
its been awhile since I had a new release of Dex.  This release is mainly focused on synchronizing Dex with the
current capabilities of dex.js and refactoring some of the previous capabilities.

Some highlights include:

  * Integration with this new site.
  * Replacement of many old visuals with dex.js based ones.
  * Many new visuals such as:
    * Plot.ly World Chloropleth
    * Elegans scatterplot
    * Chord Multiples
    * Dendrogram Multiples
  * Added JRuby support
  * Reworked and streamlined workflow
  * Library refreshes
  * Selenium streamlined (reducing install footprint by about 30 megs)
  * New online help (still in progress, but progressing nicely.
  * New datasets
  * Support for interactive chart configuration from Java <-> Javascript
  * Debug support in WebView
  * Bug fixes
    * Relative path when loading CSV from user directory.
    * Process control (cancel, dismiss, background)
    * Fixed many views which would not display in WebView due to JS "use strict"
    * C3 resize bug fix
    * Streamlined save/load
    * Link updates to dexvis.net

## Dex Support

The old [Dex blog](https://dexvis.wordpress.com) will remain, however, new blog
entries will occur here.  I will probably not be porting content.

[Code for Dex](https://github.com/PatMartin/Dex) is on github.

The sister project [dex.js](https://dexjs.net) has it's own site and it's own blog.
