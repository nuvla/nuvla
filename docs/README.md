# Nuvla Documentation

This subdirectory contains the sources for the high-level Nuvla
documentation intended for users, managers, and administrators.

This documentation appears on a [GitHub Pages
site](https://nuvla.github.io/nuvla) linked to this
repository. Commits to the master branch of this repository will
automatically regenerate the online documentation.

## Serving Documentation Locally

When writing documentation, it can be frustrating to run through the
commit, wait, fix with updates on the GitHub Page site. Fortunately,
this documentation can be served locally to shorten the editing
cycle.

First, you **must** have a full Ruby development environment setup on
your machine. For Mac OS, Ruby is available via
[Homebrew](https://brew.sh/). See the Ruby site for [installation
instructions](https://www.ruby-lang.org/en/documentation/installation/)
on other platforms.

Next, clone this repository and move to the `docs` subdirectory of
your local copy.

Install the necessary Ruby gems on your machine.  To avoid a global
installation, you can use the following command:

```sh
bundle install --path vendor/bundle
```

which will install all of the gems into the `vendor/bundle`
subdirectory.  The contents of this directory are ignored by git.

Finally, build and serve the documentation with the following command:

```sh
bundle exec jekyll serve --watch --baseurl "" --config _config_local.yml,_config.yml
```

The startup logging will indicate the server's URL; this is usually
http://127.0.0.1:4000/.

The double configuration files are required to allow the content to be
served locally and also on GitHub pages without build errors in either
place.

In principle, the `--watch` option could be replaced with
`--livereload`, which would then automatically reload the browser.
Unfortunately, this fails with a link error on Mac OS.  If you find a
solution to this issue, update the instructions here.
