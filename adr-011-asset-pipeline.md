# Architecture Decision Record: Asset Pipeline

## Context

In addition to handling arbitrary HTTP requests, we would like for Arachne to make it easy to serve up certain types of well-known resources, such as static HTML, images, CSS, and JavaScript.

These "static assets" can generally be served to users as files directly, without processing at the time they are served. However, it is extremely useful to provide *pre-processing*, to convert assets in one format to another format prior to serving them. Examples of such transformations include:

- SCSS/LESS to CSS
- CoffeeScript to JavaScript
- ClojureScript to JavaScript
- Full-size images to thumbnails
- Compress files using gzip

Additionally, in some cases, several such transformations might be required, on the same resource. For example, a file might need to be converted from CoffeeScript to JavaScript, then minified, then gzipped.

In this case, asset transformations form a logical pipeline, applying a set of transformations in a known order to resources that meet certain criteria.

Arachne needs a module that defines a way to specify what assets are, and what transformations ought to apply and in what order. Like everything else, this system needs to be open to extension by other modules, to provide custom processing steps.

### Development vs Production

Regardless of how the asset pipeline is implemented, it must provide a good development experience such that the developer can see their changes immediately. When the user modifies an asset file, it should be automatically reflected in the running application in near realtime. This keeps development cycle times low, and provides a fluid, low-friction development experience that allows developers to focus on their application.

Production usage, however, has a different set of priorities. Being able to reflect changes is less important; instead, minimizing processing cost and response time is paramount. In production, systems will generally want to do as much processing as they can ahead of time (during or before deployment), and then cache aggressively.

### Deployment & Distribution

For development and simple deployments, Arachne should be capable of serving assets itself. However, whatever technique it uses to implement the asset pipeline, it should also be capable of sending the final assets to a separate cache or CDN such that they can be served statically with optimal efficiency. This may be implemented as a separate module from the core asset pipeline, however.

### Entirely Static Sites

There is a large class of websites which actually do not require any dynamic behavior at all; they can be built entirely from static assets (and associated pre-processing.) Examples of frameworks that cater specifically to this type of "static site generation" include Jekyll, Middleman, Brunch, and many more.

By including the asset pipeline module, and *not* the HTTP or Pedestal modules, Arachne also ought to be able to function as a capable and extensible static site generator.

## Decision

Arachne will use Boot to provide an abstract asset pipeline. Boot has built-in support for immutable Filesets, temp directory management, and file watchers.

As with everything in Arachne, the pipeline will be specified as pure data in the configuration, specifying inputs, outputs, and transformations explicitly.

Modules that participate in the asset pipeline will develop against a well-defined API built around Boot Filesets.

## Status

PROPOSED

## Consequences

- The asset pipeline will be fully specified as data in the Arachne configuration.
- Adding Arachne support for an asset transformation will involve writing a relatively straightforward wrapper adapting the library to work on boot Filesets.
- We will need to program against some of Boot's internal APIs, although Alan and Micha have suggested they would be willing to factor out the Fileset support to a separate library.
