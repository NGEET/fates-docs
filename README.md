
# fates-docs: Documentation Repository for the FATES Model

FATES documentation is compiled using [sphinx](http://www.sphinx-doc.org/en/master/).  

The documentation source code found in the origin repository is automatically passed to Read The Docs, which compiles the source on server, and serves the documentation web-pages here:

https://fates-docs.readthedocs.io/en/latest/index.html


# Documentation Contributions

Contributors looking to update the documentation will need to have a computer, prefferably a digital computer, with sphinx installed on their computer for testing.  Specifically the build Makefile will call "sphinx-build".

Contributors will likely need to work in three files where the meat of the documentation for fates is held:

1) Main Text: [docs/source/fates_tech_note.rst](https://github.com/rgknox/fates-docs/blob/master/docs/source/fates_tech_note.rst)

2) References: [docs/source/CLM50_Tech_Note_References.rst](https://github.com/rgknox/fates-docs/blob/master/docs/source/CLM50_Tech_Note_References.rst)

3) The Root File: [docs/source/index.rst](https://github.com/rgknox/fates-docs/blob/master/docs/source/index.rst)



To test contributions, perform a compilation from the docs/ folder.

> make clean

> make html

Then open the resulting local html code in your browser of choice.

> google-chrome docs/build/html/index.html
