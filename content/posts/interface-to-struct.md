---
title: "Accessing the underlying struct of an interface"
description: "Converting from an interface to a structure is an important tool with a slightly obtuse syntax"
summary: "Converting from an interface to a structure is an important tool with a slightly obtuse syntax"
date: 2021-08-10T08:22:39-07:00
draft: false #remember to change to false before submitting PR
author: "wwsean08" # the name of your author file under data/authors minus the yaml file extension
---

Converting between an `interface` and a `struct` can be done leveraging type assertions in Go.  Let's take the following [example code](https://play.golang.org/p/EhsiuwZ89IH) and see how it would work:

```go
package main

import (
	"fmt"
	"math"
)

type shape interface {
	area() float64
}

type rectangle struct {
	length float64
	width  float64
}

type circle struct {
	radius float64
}

func (r rectangle) area() float64 {
	return r.length * r.width
}

func (c circle) area() float64 {
	return math.Pi * math.Pow(c.radius, 2)
}

func main() {
	// Create a slice of shapes to iterate over
	shapes := []shape{
		rectangle{length: 2, width: 2},
		circle{radius: 3},
	}

	for _, item := range shapes {
		if rect, ok := item.(rectangle); ok {
			// Check if the item is a rectangle, if it is, then enter this if block
			fmt.Println("I'm a rectangle")
			fmt.Printf("length: %f, width %f, area: %f\n", rect.length, rect.width, item.area())
		} else if circ, ok := item.(circle); ok {
			// Check if the item is a cirlce, if it is, then enter this if block
			fmt.Println("I'm a circle")
			fmt.Printf("radius: %f, area: %f\n", circ.radius, item.area())
		} else {
			// If it's anything else, we aren't sure what it is
			fmt.Println("I don't know what I am")
		}
	}
}
```

When running this code you should see the following output:
```text
I'm a rectangle
length: 2.000000, width 2.000000, area: 4.000000
I'm a circle
radius: 3.000000, area: 28.274334
```

As you can see in the above example you can leverage type assertions to get access to the underlying data of an object.  This can be extremely useful for things like custom errors, where you might have additional information, like what HTTP status code to return, or more detailed information on the error.