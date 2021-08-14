---
title: "How to embed static content into your go binaries"
description: "Simplify your deployment by including everything your app needs within the executable"
summary: "Simplify your deployment by including everything your app needs within the executable"
date: 2021-08-14T10:30:58-07:00
draft: false #remember to change to false before submitting PR
author: "wwsean08" # the name of your author file under data/authors minus the yaml file extension
---

Given that Go is a compiled language, compiling to a single binary, it can be easy to share your applications with the world, but sometimes there might be some other configuration or tooling that is necessary for your application to run.  One example of this, could be that you've trained a neural network, and wish to use the results in your go application, another might be if you are creating an API and front end, and wish to embed the HTML/CSS/JS within the binary and serve them up directly.  In both of these scenarios we can leverage go to embed the files into the application and retrieve them when necessary.  The one downside with this method is that these files are now tightly coupled with your application, so for example if you were to tweak the UI of a webapp, a new version of the binary produced by go would be required, there are some ways around this which will be outlined later, but first lets dive into the basic code.  All examples below are also available in full [here](https://gitlab.com/wwsean08/embed-examples).

First a simple hello world that leverages a text file in the same directory named `hello.txt` with the contents 'hello world'.
```go
package main

import (
	// note the _ which is requires since it's not being called 
	// directly anywhere in this case, and is just a build directive
	_ "embed"
	"fmt"
)

// note that hello can't be defined as a const which means it could 
// be overwritten by your application at runtime
//go:embed hello.txt
var hello string

func main() {
	fmt.Printf("%s\n", hello)
}
```

Running this would print out hello world, which is pretty slick, but not the most useful when you might have tens or even hundreds of files to embed.  In that case you can embed a file system which would be more useful:

```go
package main

import (
	"embed"
	"io/fs"
	"log"
	"net/http"
)

// This will embed anything under webapp in your project
//go:embed webapp/*
var files embed.FS

func main() {
	println("server listening on port 8080")
	webapp, err := fs.Sub(files, "webapp")
	if err != nil {
		log.Fatal(err.Error())
	}
	log.Fatal(http.ListenAndServe(":8080", http.FileServer(http.FS(webapp))))
}
```

In the above example (available [here](https://gitlab.com/wwsean08/embed-examples/-/tree/main/embed-fs)) all the files under webapp are embedded directly in the binary and served up and accessible via http://localhost:8080, allowing you to easily embed your webpage directly into your API that it is leveraging, or if you are using go templating, embedding the templates directly into the binary.  This does come with the downside outlined above that in order to make a change to a file, a new build is needed which is less flexible than reading it from the file system where your application is running.

With all that said you could (with a bit of work) have the best of both worlds, embed your files into the application, but allow for on the fly updates as well.  Note that this is fraught with danger and likely is a bad idea for most, but may be useful in some applications.

**_warning_** I do not advise anyone does this as most likely it will cause issues and not be worth the hassle and code complication.  Picking one method of dealing with ancillary files and sticking with it is by far a better option.

```go
package main

import (
	"embed"
	"fmt"
	"io/fs"
	"log"
	"net/http"
	"os"
)

// This will embed anything under webapp in your project
//go:embed webapp/*
var files embed.FS

const assetRoot = "./tmp"

func main() {
	err := updateFS()
	if err != nil {
		log.Fatal(err.Error())
	}
	webapp := os.DirFS(assetRoot)
	webapp, err = fs.Sub(files, "webapp")
	if err != nil {
		log.Fatal(err)
	}
	println("server listening on port 8080")
	log.Fatal(http.ListenAndServe(":8080", http.FileServer(http.FS(webapp))))
}

func updateFS() error {
	return fs.WalkDir(files, ".", walk)
}

func walk(path string, d fs.DirEntry, _ error) error {
	if path == "." {
		return nil
	}
	newPath := fmt.Sprintf("%s/%s", assetRoot, path)

	if d.IsDir() {
		// if the directory doesn't exist, create it
		if _, err := os.Stat(newPath); os.IsNotExist(err) {
			return os.MkdirAll(newPath, 0700)
		}
	} else {
		fInfo, err := os.Stat(newPath)
		// file doesn't exist, create it
		if os.IsNotExist(err) {
			file, err := files.Open(path)
			if err != nil {
				return nil
			}
			data, err := files.ReadFile(path)
			if err != nil {
				return err
			}
			fStat, err := file.Stat()
			if err != nil {
				return err
			}

			return os.WriteFile(newPath, data, fStat.Mode())
		} else {
			// file exists, update it if this version is newer (based on last modified time)
			file, err := files.Open(path)
			if err != nil {
				return nil
			}
			fStat, err := file.Stat()
			if err != nil {
				return err
			}
			if fStat.ModTime().After(fInfo.ModTime()) {
				data, err := files.ReadFile(path)
				if err != nil {
					return err
				}

				return os.WriteFile(newPath, data, fStat.Mode())
			}
		}
	}
	return nil
}
```

To see all the files associated with this example you can go [here](https://gitlab.com/wwsean08/embed-examples/-/tree/main/embed-and-extract).

With all that, you should now know how to embed files within your go applications for usage at runtime, whether you want to do something simple like embedding a single file like the model output from a neural network, or something more complex like a filesystem in the example of a webapp.