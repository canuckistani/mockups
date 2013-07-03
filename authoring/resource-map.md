
# Resource Map

## TL;DR

We should have something like [sourcemaps](SM) but that handles the resources
in a website rather than just for a single source file.

The point is to allow a mapping between the resources presented by a web server
and writable versions of the same resources - probably on a local disk.

With sourcemaps, a transformer (like a compiler, transpiler, compressor, etc)
can point to a file called a sourcemap which connects the final source to the
original source using a special ``//# sourceMappingURL`` comment or an
``X-SourceMap`` header.

With resourcemaps, a transformer (i.e. a webserver) can point to a file called
a resourcemap which connects the final resources to the original resources
using a special ``X-ResourceMap`` header.

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

We want to make resourcemap files easy to create, so a resourcemap file must:

* be trivially composable by hand for the common case where the web server is
  simply serving a directory to a URL. i.e. it shouldn't require a syntax that
  is significantly more complex than the following example

      { 'http://localhost:8080/': '/home/me/projects/example' }

* not require manual intervention for all but the trivial case, so if the
  resourcemap file is complex then users can expect tools to generate it
* be generatable:
  * by a web server
  * by a development environment that knows how a project is viewed on the web
  * by an 'add mappping' dialog in an authoring tool
* be easily excluded from version control (the contents of a resourcemap file
  are likely to be dependent on position in filesystem, which is beyond the
  scope of version control)

I currently thinking that the resourcemap file format will be based on JSON
because, well, why not?

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

[IDR]: http://www.petefreitag.com/item/505.cfm

Webmasters that are concerned about information disclousure should not publish
resourcemaps, but making a mistake should not cause a problem that didn't
already exist.

### Storage

Resourcemap files are all about storage of resources.

* Resources referred to by a projectmap file may exist:
  * on a local filesystem
  * on remote filesystems over ssh
  * directly in version control systems (e.g. git)
  * unknown/custom filesystems (e.g. using adb, etc)
* The resourcemap spec will need to define how these filesystem resources are
  specified
* Tools that consume resourcemap files should not be required to support any
  one storage system
* The information required for an authoring tool to 'save' a resource should be
  limited to just the contents of the resouce

### Open Questions

TODO:

* Should resourcemaps allow 'smart matching'? i.e. There is only one file called
  ``strangename.js`` under ``/project`` so we can be fairly sure that when the
  browser is referred to ``http://example.com/123456/strangename.js``, that
  it can correctly guess the original path. Should we allow/encourage this
  guessing?
* What about composing. When a web server resource is made up of a number of
  file system resources, does the resourcemap need to discuss this, or is
  sourcemap enough?
* What about transformed resources. How does that work?
* What about lossily transformed resources. (PHP->HTML)

## Examples

WIP
