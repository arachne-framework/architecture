# Architecture Decision Record: Project Templates

## Context

When starting a new project, it isn't practical to start completely from scratch, every time. We would like to have a varity of "starting point" projects, for different purposes.

In the Clojure space, Leiningen Templates fill this purpose. These are sets of special string-interpolated files that are "rendered" into a working project using special tooling.

However, they have two major drawbacks:

- They only work when using Leiningen as a build tool.
- The template files are are not actually valid source files, which makes them difficult to maintain. Changes need to be manually copied over to the templates.

## Decision

To start with, Arachne templates will be standard git repositories containing an Arachne project. They will use no special syntax, and will be valid, runnable projects out of the box.

In order to allow users to create their own projects, these template projects will include a script to rename themselves. 

Therefore, the process to start a new Arachne project will be:

1. Choose an appropriate project template.
2. Clone its git repository from Github
3. Run the `rename` script to rename the project to whatever you wish
4. Start a repl, and begin editing.

## Status

PROPOSED

## Consequences

- It will take only a few moments for users to create new Arachne projects.
- It will be straightforward to build, curate, test and maintain multiple different types of template projects.
- The only code we will need to write to support templates is the "rename" script.
- The rename script will need to be capable of renaming all the code and files in the template, with awareness of the naming requirements and conventions for Clojure namespaces and code.




