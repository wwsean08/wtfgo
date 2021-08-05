---
title: "How to conditionally build and test source code"
description: "Build tags and special naming conventions are useful tools in your go toolbox"
date: 2021-08-04T16:08:02-07:00
draft: false #remember to change to false before submitting PR
author: "wwsean08" # the name of your author file under data/authors minus the yaml file extension
---

Go allows you to conditionally build source code based on attributes like the Operating System, Architecture, and even just plaintext tags, below is a breakdown of how they work.

# File Naming
The easiest way to build for a specific OS/Architecture is to append that information to the end of your filename, some examples include:
* `main_windows.go`
* `main_darwin.go`
* `main_darwin_arm64.go` specific to ARM macs, like MacBook Pros with an M1 processor

The order of these names must be GOOS_GOARCH, the other way around will not work, and of course they still require a name beyond this suffix.

This method is extremely useful if you have some set of functions that only exist in one OS or OS/ARCH combination.  If you want something a little more granular, then you will need build tags.

# Build tags
Build tags are a special type of comment that go at the top of a code file to instruct the compiler when to conditionally compile the code.  They can similarly specify that the code can only be built for a specific platform, but even more powerfully, they can specify multiple platforms.  Below is an example of a file specifying that it can be built for Linux or Mac, but not for Windows.

```go
// +build linux darwin

package foo // note the blank line above, that is required
```

This could also be written similarly below (I realize there are more OSs than Windows/Mac/Linux in reality):

```go
// +build !windows

package foo
```

The rewritten form says that the file can be compiled for all operating systems that are not Windows.

This can also be used for tests, for example say you want to create a suite of integration tests for your code, you could simply create some test files with a build tag like this:

```go
// +build integration

package foo
```

During a regular `go test` command these files will not be executed, however if you run `go test --tags integration` they will.  The caveat is that your regular untagged tests will run in this case as well, in order to get only integration tests to run would be to add a `!integration` build tag to regular test files.

# Sources and useful links
* [Build Constraints Documentation](https://pkg.go.dev/cmd/go#hdr-Build_constraints)
* [DigitalOcean build customization using tags](https://www.digitalocean.com/community/tutorials/customizing-go-binaries-with-build-tags)
