# Architecture Decision Record: Error Reporting

## Context

Historically, error handling has not been Clojure's strong suit. For the most part, errors take the form of a JVM exception, with a long stack trace that includes a lot of Clojure's implementation as well as stack frames that pertain directly to user code.

Additionally, prior to the advent of `clojure.spec`, Clojure errors were often "deep": a very generic error (like a NullPointerException) would be thrown from far within a branch, rather than eagerly validating inputs.

There are Clojure libraries which make an attempt to improve the situation, but they typically do it by overriding Clojure's default exception printing functions across the board, and are sometimes "lossy", dropping information that could be desirable to a developer.

Spec provides an opportunity to improve the situation across the board, and with Arachne we want to be on the leading edge of providing helpful error messages that point straight to the problem, minimize time spent trying to figure out what's going on, and let developers get straight back to working on what matters to them.

Ideally, Arachne's error handling should exhibit the following qualities:

- Never hide possibly relevant information.
- Allow module developers to be as helpful as possible to people using their tools.
- Provide rich, colorful, multi-line detailed explanations of what went wrong (when applicable.)
- Be compatible with existing Clojure error-handling practices for errors thrown from libraries that Arachne doesn't control.
- Not violate expectations of experienced Clojure programmers.
- Be robust enough not to cause additional problems.
- Not break existing logging tools for production use.

## Decision

We will separate the problems of creating rich exceptions, and catching them and displaying them to the user.

### Creating Errors

Whenever a well-behaved Arachne module needs to report an error, it should throw an info-bearing exception. This exception should be formed such that it is handled gracefully by any JVM tooling; the message should be terse but communicative, containing key information with no newlines.

However, in the `ex-data`, the exception will also contain much more detailed information, that can be used (in the correct context) to provide much more detailed or verbose errors. Specifically, it may contain the following keys:

- `:arachne.error/message` - The short-form error message (the same as the Exception message.)
- `:arachne.error/explanation` - a long-form error message, complete with newlines and formatting.
- `:arachne.error/suggestions` - Zero or more suggestions on how the error might be fixed.
- `:arachne.error/type` - a namespaced keyword that uniquely identifies the type of error.
- `:arachne.error/spec` - The spec that failed (if applicable)
- `:arachne.error/failed-data` - The data that failed to match the spec (if applicable)
- `:arachne.error/explain-data` - An explain-data for the spec that failed (if applicable).
- `:arachne.error/env` - A map of the locals in the env at the time the error was thrown.

Exceptions may, of course, contain additional data; these are the common keys that tools can use to more effectively render errors.

There will be a suite of tools, provided with Arachne's core, for conveniently generating errors that match this pattern.

### Displaying Errors

We will use a pluggable "error handling system", where users can explicitly install an exception handler other than the default.

If the user does not install any exception handlers, errors will be handled the same way as they are by default (usually, dumped with the message and stack trace to  `System/err`.) This will not change.

However, Arachne will also provide a function that a user can invoke in their main process, prior to doing anything else. Invoking this function will install a set of default exception handlers that will handle errors in a richer, more Arachne-specific way. This includes printing out the long-form error, or even (eventually) popping open a graphical data browser/debugger (if applicable.)

## Status

PROPOSED

## Consequences

- Error handling will follow well-known JVM patterns.
- If users want, they can get much richer errors than baseline exception handling.
- The "enhanced" exception handling is optional and will not be present in production.
 



