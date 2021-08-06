---
title: "Are maps, slices, and channels thread safe"
description: "TL;DR; maps and slices are not thread safe, channels are."
summary: "TL;DR; maps and slices are not thread safe, channels are."
date: 2021-08-05T07:56:54-07:00
draft: false #remember to change to false before submitting PR
author: "wwsean08" # the name of your author file under data/authors minus the yaml file extension
tags:
  - concurrency
---

When it comes to concurrency, every language does something differently.  For go, most things are not thread safe, including maps and slices, however channels are thread safe.

# Making maps and slices thread safe
Maps and slices are safe for concurrent reads, however concurrently reading/writing is not supported, and concurrent writes are not supported.  A mutex can be used to make the objects safe for access from multiple threads as follows:

```go
package main

import "sync"

type DAO struct {
	data map[string]string
	mutex *sync.RWMutex
}

func main() {
	dao := &DAO{
		data: make(map[string]string),
		mutex: new(sync.RWMutex),
	}
	// concurrently read and write from DAO
}

func (d *DAO) Read(key string) (string, bool) {
	// Create a read lock which will allow concurrent reads but prevent any writes
	d.mutex.RLock()
	// Unlock when exiting the function
	defer d.mutex.RUnlock()
	val, exists := d.data[key]
	return val, exists
}

func (d *DAO) Write(key, value string) {
	// Creates a write lock which will prevent reads and  concurrent writes
	d.mutex.Lock()
	// Unlock when exiting the function
	defer d.mutex.Unlock()
	d.data[key] = value
}
```

This example would also work for slices as well by simply changing the data object to a slice, and tweaking the read/write functions.