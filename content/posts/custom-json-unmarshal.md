---
title: "Unmarshal JSON data in non-standard ways"
description: ""
summary: "How to leverage the Unmarshaler interface to parse JSON data in ways that fit your needs"
date: 2021-08-06T07:57:00-07:00
draft: false #remember to change to false before submitting PR
author: "wwsean08" # the name of your author file under data/authors minus the yaml file extension
tags:
  - json
---

Reading and writing data is the basis of any program in every language, and go is no different.  While the default JSON unmarshaler is generally good enough to read in JSON data, there are times when you'll need to implement your own custom unmarshal.  The most common scenarios I've run into for this are that either a date/time isn't in RFC 3339 format, or the JSON data has more json data embedded in a string.  Below is an example to customize unmarshalling a time that is not in the RFC 3999 format.

```go
package main

import (
	"encoding/json"
	"time"
)

// Time wraps the standard time.Time to allow for custom parsing from json
type Time struct {
	time.Time
}

func main() {
	// temp structure to hold the data being parsed
	// in reality there would likely be a larger structure
	type temp struct {
		TempTime *Time `json:"time"`
	}
	// data to parse
	data := []byte("{\"time\": \"2021-05-22T18:31:59.588379+00:00\"}")
	parsedData := new(temp)

	err := json.Unmarshal(data, parsedData)
	if err != nil {
		println(err.Error())
		return
	}

	// print out the unix timestamp for the input time
	println(parsedData.TempTime.Unix())
}

// UnmarshalJSON implemented to allow for parsing this time object 
// with a different format
func (t *Time) UnmarshalJSON(b []byte) error {
	// the layout of the timestamp being parsed
	layout := "2006-01-02T15:04:05.999999999-07:00"
	var timeAsString string
	// grab the string value of the input
	err := json.Unmarshal(b, &timeAsString)
	
	// parse it using the time package, returning an error if it errors
	// otherwise storing the result
	timestamp, err := time.Parse(layout, timeAsString)
	if err != nil {
		return err
	}
	t.Time = timestamp
	return nil
}
```

If you were to run the above example code `1621708319` should print out from parsing the json data out of the byte slice, leveraging the custom unmarshaler that was written.

The reason this works is `UnmarshalJSON(b []byte) error` is defined in an interface, specifically the [Unmarshaler](https://pkg.go.dev/encoding/json#Unmarshaler) interface, similarly the `MarshalJSON` function is defined in the [Marshaler](https://pkg.go.dev/encoding/json#Marshaler) interface and could be overridden.