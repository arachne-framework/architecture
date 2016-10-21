# Architecture Decision Record: Project Templates

## Context

When starting a new project, it isn't practical to start completely from scratch, every time. We would like to have a varity of "starting point" projects, for different purposes.

### Lein templates
In the Clojure space, Leiningen Templates fill this purpose. These are sets of special string-interpolated files that are "rendered" into a working project using special tooling.

However, they have two major drawbacks:

- They only work when using Leiningen as a build tool.
- The template files are are not actually valid source files, which makes them difficult to maintain. Changes need to be manually copied over to the templates.

### Rails templates

Rails also provides a complete project templating solution. In Rails, the project template is a `template.rb` file which contains DSL forms that specify operations to perform on a fresh project. These operations include creating files, modifying a projects dependencies, adding Rake tasks, and running specific _generators_. 

Generators are particularly interesting, because the idea is that they can generate or modify stubs for files pertaining to a specific part of the application (e.g, a new model or a new controller), and they can be invoked _at any point_, not just initial project creation. 

## Decision

To start with, Arachne templates will be standard git repositories containing an Arachne project. They will use no special syntax, and will be valid, runnable projects out of the box.

In order to allow users to create their own projects, these template projects will include a `rename` script. The `rename` script will recursively rename an entire project directory to something that the user chooses, and will delete `.git` and re-run `git init`, 

Therefore, the process to start a new Arachne project will be:

1. Choose an appropriate project template.
2. Clone its git repository from Github
3. Run the `rename` script to rename the project to whatever you wish
4. Start a repl, and begin editing.

### Maven Distribution

There are certain development environments where there is not full access to the open internet (particularly in certain governmental applications.) Therefore, accessing GitHub can prove difficult. However, in order to support developers, these organizations often run their own Maven mirrors.

As a convenience to users in these situations, when it is necessary, we can build a wrapper that can compress and install a project directory as a Maven artifact. Then, using standard Maven command line tooling, it will be possible to download and decompress the artifact into a local filesystem directory, and proceed as normal.

## Status

PROPOSED

## Consequences

- It will take only a few moments for users to create new Arachne projects.
- It will be straightforward to build, curate, test and maintain multiple different types of template projects.
- The only code we will need to write to support templates is the "rename" script.
- The rename script will need to be capable of renaming all the code and files in the template, with awareness of the naming requirements and conventions for Clojure namespaces and code.
- Template projects themselves can be built continuously using CI

### Contrast with Rails

One way that this approach is inferior to Rails templates is that this approach is "atomic"; templating happens once, and it happens for the whole project. Rails templates can be composed of many different generators, and generators can be invoked at any point over a project's lifecycle to quickly stub out new functionality.

This also has implications for maintenance; because Rails generators are updated along with each Rails release, the template itself is more stable, wheras Arachne templates would need to be updated every single time Arachne itself changes. This imposes a maintenance burden on templates maintained by the core team, and risks poor user experience for users who find and try to use an out-of-date third-party template.

However, there is is mitigating difference between Arachne and Rails, which relates directly to the philosophy and approach of the two projects.

In Rails, the project *is* the source files, and the project directory layout. If you ask "where is a controller?", you can answer by pointing to the relevant `*.rb` file in the `app/controllers` directory. So in Rails, the task "create a new controller" _is equivalent to_ creating some number of new files in the appropriate places, containing the appropriate code. Hence the importance of generators.

In Arachne, by contrast, the project is not ultimately defined by its source files and directory structure; it is defined by the config. Of course there *are* source files and a directory structure, and there will be some conventions about how to organize them, but they are not the very definition of a project. Instead, a project's _Configuration_ is the canonical definition of what a project is and what it does. If you ask "where is a controller?" in Arachne, the only meaningful answer is to point to data in the configuration. And the task "create a controller" means inserting the appropriate data into the config (usually via the config DSL.) 

As a consequence, Arachne can focus less on code generation, and more on generating *config* data. Instead of providing a _code_ generator which writes source files to the project structure, Arachne can provide _config_ generators which users can invoke (with comparable effort) in their config scripts.

As such, Arachne templates will typically be very small. In Arachne, code generation is an antipattern. Instead of making it easy to generate code, Arachne focuses on building abstractions that let users specify their intent directly, in a terse manner.