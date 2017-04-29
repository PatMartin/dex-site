+++
date = "2017-04-08T13:37:53-05:00"
title = "releases"
[menu.main]
  identifier = "releases"
+++

# Releases

To save bandwidth and reduce my hosting costs, I have reduced the release size so that I can host the releases on
GitHub.  As a result, older releases are no longer available.

  * [dex-0.9.0.0.zip](https://github.com/PatMartin/DexReleases/blob/master/dex-0.9.0.0.zip?raw=true)
    * Release notes
      * Notable bug fixes.
        * When data is loaded from within the dex user directory, it will use a relative path.
        This will allow for demos to work as-is when the Dex is installed or moved to a
        different directory.
      * Examples, lots more examples and tutorials to get new users going.
      * Cleanup and optimization which reduced distribution size by 30 megs.
      * Refactored existing visualizations to use latest version dexjs where possible.

## Development Version on Github

These changes are currently on GitHub and will be rolled into the next release.

  * UI CSS Styling changes
  * Refresh of NVD3
  * Refresh of dex.js
  * NVD3 Stacked Area Chart now supports categorical, date and numeric axis.
  * NVD3 Bubble Chart now support categorical, date and numeric axis.
  * Deprecated Javascript Script task
  * Deprecated JRuby Script task
  * Parallel Coordinates To Treemap
  * Parallel Coordinates To Sankey
  * Parallel Coordinates To Network
  * Parallel Coordinates To Sunburst
  * WriteCsv : Support for saving relative paths.
  * WriteCsv : Support for remembering last directory saved to.
  * New Multiples Visualizations
    * TreemapBarChart
    * Chord
    * Clustered Force
    * Dendrogram
    * Network
    * Orbital Layout
    * Parallel Coordinates
    * Sankey
    * Sunburst
    * Treemap