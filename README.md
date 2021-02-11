
# fates-docs: Documentation Repository for the FATES Model

[![DOI](https://zenodo.org/badge/146812405.svg)](https://zenodo.org/badge/latestdoi/146812405)

FATES documentation is compiled using [sphinx](http://www.sphinx-doc.org/en/master/).  

The documentation source code found in the origin repository is automatically passed to Read The Docs, which compiles the source on server, and serves the documentation web-pages here:

https://fates-docs.readthedocs.io/en/latest/index.html

## Documentation Contributions

Contributors will likely need to work in three files where the meat of the documentation for fates is held:

1) Main Text: [docs/source/fates_tech_note.rst](https://github.com/rgknox/fates-docs/blob/master/docs/source/fates_tech_note.rst)

2) References: [docs/source/CLM50_Tech_Note_References.rst](https://github.com/rgknox/fates-docs/blob/master/docs/source/CLM50_Tech_Note_References.rst)

3) The Root File: [docs/source/index.rst](https://github.com/rgknox/fates-docs/blob/master/docs/source/index.rst)

Contributors looking to update the documentation will need to have a computer, prefferably a digital computer, with sphinx installed on their computer for testing.  Specifically the build Makefile will call "sphinx-build".

### Sphinx installation

An installation of `sphinx` and [`make`](https://www.gnu.org/software/make/) is necessary to compile contributions local so that user's may test that they are displaying correctly.  Sphinx is available for MacOS, Windows, and Linux distributions through a variety of package management tools, such as `conda`.  The installation instructions can be found here: https://www.sphinx-doc.org/en/master/usage/installation.html.  Installing via a `conda` environment is recommended so as be able to make sure that the necessary dependencies are appropriately installed.

### Testing contributions

To test contributions with sphinx, perform a compilation from the `fates-docs/docs/` folder. First remove any previous build output by calling via the command line:
```
$ make clean
Removing everything under 'build'...
```

Then compile the document as a local html file:
```
$ make html
```

The output from this command should look something like this:
```
$ make html
Running Sphinx v3.4.3
making output directory... done
building [mo]: targets for 0 po files that are out of date
building [html]: targets for 8 source files that are out of date
updating environment: [new config] 8 added, 0 changed, 0 removed
reading sources... [100%] parteh/turnover                                                                                                                                                                                                     
looking for now-outdated files... none found
pickling environment... done
checking consistency... done
preparing documents... done
writing output... [100%] parteh/turnover                                                                                                                                                                                                      
generating indices... genindex done
writing additional pages... search done
copying images... [100%] images/growth_allometry_p2.png                                                                                                                                                                                       
copying static files... done
copying extra files... done
dumping search index in English (code: en)... done
dumping object inventory... done
build succeeded.

The HTML pages are in build/html.
```

Then open the resulting local html code in your browser of choice, such as `google-chrome`, and review the contributes for correctness:

```
google-chrome docs/build/html/index.html
```

## Repository workflow for admins

The fates-doc are updated using a [fork & pull method](https://www.atlassian.com/git/tutorials/comparing-workflows/forking-workflow) in a manner similar to the [fates](https://github.com/NGEET/fates) repository.

1. Review and then merge pull request(s) from contributor(s)
2. Pull update into local repository
3. Create annotated version tag using the [versioning method](#versioning) described herein
    - `git tag -a v<x.x.x> -m "<update information>"`
    - Note: do **not** tag updates that consist only of meta-data.  For further information, see [here](#meta-data).  
4. Push tag to fates-docs master
    - This will update the `latest` version on readthedocs
5. If update is a major release, create a github release
    - This will create a updated DOI version number in Zenodo
    - This will update the `stable` version on readthedocs

### Versioning

This respository follows a modification of the Github recommended semantic versioning (https://semverdoc.org/) using a *major.minor.patch* version number format.  All updates to the documentation to be merged into master should be tagged with a new version number.  This version number should be incremented as follows:

- *Major*
    - New major feature descriptions added
    - New sections added
    - Major reorginzation of document
    - Removing content due to deprecated feature
- *Minor*
    - Adding or modifying content for clarity including but not limited to:
        - Language
        - Figures
        - Tables
        - References
    - Removing content for clarity
    - New minor features added
- *Patch*
    - Fixing typos and grammar

### Meta-data

Updates to documentation meta-data, that is files containing information for the github reposititory but not incorporated into the technical documentation itself, should not be tagged when updated seperate of the documentation itself.  Examples of these files include `LICENSE` and this README file.  This is to make sure only changes to the the documentation itself correspond to version updates; the readthedocs service that hosts the documentation triggers updates off of the creation of new tags.  See the section on [webhooks](#webhook-settings) for more information.

### Stable verion

Major versions are assumed to correspond to stable versions of the documentation and the corresponding code discussed therein.  This means that ideally the major documentation changes should only be merged into the github master branch once the corresponding code has been considered stable.  This is a somewhat qualitative distinction (currently), but at the least the corresponding code should have been reviewed, tested, and merged to the code repository master branch.  Major feature having been utilitized for (non-software development) research work by multiple individuals would be an additional ideal marker of stable code.

### Webhook settings

The repository is linked to two different software services, [readthedocs](https://docs.readthedocs.io/en/stable/index.html) and [zenodo](https://help.zenodo.org/).  The first is used to host both the lastest and stable versions of the documentation on the web.  The latter is used to archive major documentation releases and to mint a DOI.  The instructions for linking the repository to zenodo are found [here](https://guides.github.com/activities/citable-code/).  Instructions for linking readthedocs to the repository are found [here](https://docs.readthedocs.io/en/stable/webhooks.html).  The github webhooks are found in the [repository settings](https://github.com/NGEET/fates-docs/settings/hooks).

Webhooks are currently configured in the following manner:

- Readthedocs webhook is triggered during *tag creation* events
    - The `latest` version of the web-based documentation will corresponding to the latest tag.
    - The `stable` version of the web-based documentation will corresponding to the latest tagged release.
- Zenodo webhook is triggered during a *release creation* event
    - A new version DOI will be created for the latest release

This was setup such that only *major* updates, which get tagged releases, will update both the `stable` readthedocs version and get a new DOI version number will be created.  We do not want to create too many archived DOI versions for only minor updates.
