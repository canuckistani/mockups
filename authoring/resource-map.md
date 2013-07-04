
# Resource Map

## TL;DR

Resourcemaps are like sourcemaps but targeting a set of resources like a
website, allowing authoring tools to save modified sources read from HTTP.

In the short term with Firefox we will probably only consider a single mapping
with a filesystem backend; just enough for the trivial static website case.
This document goes further to make sure we're headed in the right direction.

## What is the point?

We should have something like [sourcemaps](SM) but that handles the resources
in a website rather than just for a single source file.

The point is to allow a mapping between the resources presented by a web server
and writable versions of the same resources - probably on a local disk.

With sourcemaps, a transformer (like a compiler, transpiler, compressor, etc)
can point to a file called a sourcemap which connects the final source to the
original source.

With resourcemaps, a transformer (i.e. a webserver) can point to a file called
a resourcemap which connects the final resources to the original resources.

Sourcemap is a 1-to-many mapping system that converts URL→URL. Resourcemap is
1-to-1 that converts URL→Writable-resource. Sourcemaps should be applied before
resourcemaps.

The principle consumers of resourcemap files will be authoring tools that want
to allow altered web resources to be saved back to their original positions.

[SM]: https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1

## Requirements

These requirements are a work in progress. This should currently be seen as a
spec for the spec.

We should think about having a meeting to consider plans to start a committee
to consider ideas for a wider discussion of a panel at which we actually do
real work.

### Composition

We want to make resourcemap files easy to create, so a resourcemap file should:

* be in JSON, because, well, why not?
* be trivially composable by hand for the common case where the web server is
  simply serving a directory to a URL. i.e. it shouldn't require a syntax that
  is significantly more complex than a file that simply states
  ``http://example.com/ → /projects/example`` (obviously this isn't JSON, but
  we should add the minimum possible syntactic overhead).
* not require manual intervention for all but the trivial case, so if the
  resourcemap file is complex then users can expect tools to generate it
* be generatable:
  * by a web server
  * by a development environment that knows how a project is viewed on the web
  * by an 'add mapping' dialog in an authoring tool
* be easily excluded from version control (the contents of a resourcemap file
  are likely to be dependent on position in filesystem, which is beyond the
  scope of version control)

### Security

Alteration of resources by users isn't necessarily a security problem. But we
should avoid taking actions which make it hard to stay secure.

A resourcemap file:

* should be easy to hide from a live website
* will not address authentication - that's the job of the authoring tool which
  already has to address the issue of password storage
* may be an information disclosure risk

Parallels can be drawn between a resourcemap file and an HTTP "Server:"
header. Both may be useful, but may also present an [information disclosure
risk](IDR). However neither are in an of themselves a security risk.

Webmasters that are concerned about information disclosure should not publish
resourcemaps, but making a mistake should not cause a problem that didn't
already exist.

[IDR]: http://www.petefreitag.com/item/505.cfm

### Storage

Resourcemap files are all about storage of resources.

* Resources referred to by a resourcemaps file may exist:
  * on a local filesystem
  * on remote filesystems over ssh
  * directly in version control systems (e.g. git)
  * unknown/custom filesystems (e.g. using adb, etc)
* The resourcemap spec will need to define how these filesystem resources are
  specified
* Tools that consume resourcemap files should not be required to support any
  one storage system
* The information required for an authoring tool to 'save' a resource should be
  limited to just the contents of the resource

### Notification

The authoring tool has 2 ways to discover resourcemap files:

* By direction: The webserver could add ``X-ResourceMap`` headers to HTML files
  pointing to a resourcemap file.
* By convention: A file called ``resourcemap.json`` can be stored at the 'root'
  of the webserver (this allows resourcemaps to be used without altering web
  server configuration at all)

### Open Questions

Some open questions that I don't think we need to resolve right now:

* Should resourcemaps allow 'smart matching'? i.e. There is only one file called
  ``strangename.js`` under ``/project`` so we can be fairly sure that when the
  browser is referred to ``http://example.com/123456/strangename.js``, that
  it can correctly guess the original path. Should we allow/encourage this
  guessing?
* What about composing. When a web server resource is made up of a number of
  file system resources, does the resourcemap need to discuss this, or is
  sourcemap enough?
* What about transformed resources. How does that work?

## Syntax

The resourcemap file is a JSON format file that contains an array of mappings.
Each mapping is a URL-prefix to file-system prefix mapping. If there is only
one mapping, then the containing ``[]`` can be omitted.

## Examples

Except where stated otherwise, these examples use a website that is served from
``http://localhost:8000/``, and stored on disk at ``/projects/example``.

### Static project

When the webserver simply serves the files from the current directory (i.e.
``python -m SimpleHTTPServer``) then ``http://localhost:8000/resourcemap.json``
would read

    { urlPath: '/', dir: '/projects/example' }

This allows the authoring to know that ``http://localhost:8000/styles.css`` is
stored in ``/projects/example/styles.css``.

The same file will work if the browser reads from ``http://127.0.0.1:8000`` or
using a FQDN.

When on Windows, the file would look more like:

    { urlPath: '/', dir: 'C:\projects\example' }

And will require the authoring tool to map ``\`` to ``/``.

### Larger static project

Sometimes a static project may be arranged in a non-obvious way. Perhaps CSS
files are read from a directory with a different mapping.

    [
      { urlPath: '/styles', dir: '/projects/example/media/css' },
      { urlPath: '/', dir: '/projects/example' }
    ]

The authoring tool should read through the mappings finding the first matching
mapping and using the associated directory, moving onto subsequent matching
mappings if the resource is not found in the associated directory.

### Static project with SASS/Traceur/Closure

There is ``sass --watch``, but there are cases where the build step if fired
off manually.

    [
      { urlPath: '/', dir: '/projects/example' },
      { glob: '**/*.scss', post: 'sass $FILE `echo $FILE | sed s/scss^/css/`' }
    ]

The ``post`` action is a literal string which is passed to a shell
(configurable by the authoring tool) after the file has been saved and the
``$FILE` environment variable has been set.

It should be noted that this solution sucks planets out of orbit. Suggestions
for solutions which disrupt our solar system less would be appreciated.

### PHP project

Simples:

    [
      { urlPath: '/', dir: '/projects/example' },
      { glob: '**/*.php', deny: true }
    ]

Here we're saying "don't overwrite my nice PHP with your stinkin' HTML"

### Other Ideas

There are various other project files that we could consider:

Perhaps we could match using regular expressions and use matched subexpressions
in the ``dir`` clause.

    { regexp: '\(.*\.css\)^', dir: 'styles/\1' }

Perhaps we could have a standard set of filters that the authoring tool would
just know how to cope with (the trouble with this is keeping the authoring tool
up to date and correctly configured).

    { regexp: '\.css^', filter: sass, dir: '/projects/example' }

In order to save to a non-local filesystem we could do:

    { urlPath: '/', ssh: 'joe@example.com:/var/srv/www' }

Or:

    { urlPath: '/', adb: '/sdcard/i/have/no/idea' }

Or generically:

    { urlPath: '/', pipe: 'sed s/foo/bar/g | cat > $FILE' }

## Scope

Initially I propose that we only tackle the case of a single mapping from a
urlPath to a directory in the local filesystem. i.e this case:

    { urlPath: '/', dir: '/projects/example' }

I'd really like to use ``urlPathPrefix`` in place of ``urlPath`` but that's
quite verbose.
It's certainly not a full URL (which is more verbose and also fragile when the
same host can be reached in more than one way)
