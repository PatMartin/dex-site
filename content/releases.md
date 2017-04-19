+++
date = "2017-04-08T13:37:53-05:00"
title = "releases"
[menu.main]
  identifier = "releases"
+++

# Releases

To save bandwidth and reduce my hosting costs, I have reduced the release size so that I can host the releases on
GitHub.  As a result, older releases are no longer available.

  * [dex-0.9.0.0.zip](https://github.com/PatMartin/DexReleases/blob/master/dex-0.9.0.0.zip?raw=true) - Latest release of Dex.
    * Release notes
      * Notable bug fixes.
        * When data is loaded from within the dex user directory, it will use a relative path.
        This will allow for demos to work as-is when the Dex is installed or moved to a
        different directory.
      * Examples, lots more examples and tutorials to get new users going.
      * Cleanup and optimization which reduced distribution size by 30 megs.
      * Refactored existing visualizations to use latest version dexjs where possible.