# Parser

Parser is an extensible parser that decodes multiple data formats and extracts
the results into a go map.

## Features

JSON, YAML, and TOML handlers are included.  You can easily add your own handler
by writing a function that implements `ParseFunc`. Then add your function by
calling: `Handle("my-type", MyTypeFunc)`. See the code below for an example.

## Install

```
go get github.com/delputnam/parser
```

## How to use

First instantiate a new parser: `p := parser.NewParser()`

Then call  `p.Parse(inputType, inputData)` where `inputType` is a string that
corresponds to the format of the string in `inputData`.

You can specify one of the built-in handlers with the following:

  * JSON with an `inputType` of `"json"`
  * YAML with an `inputType` of `"yaml"` or `"yml"`
  * TOML with an `inputType` of `"toml"` or `"tml"`

If the parser succeeds, `p.Parse()` returns a map where each named key in the input
data is the value of a key in the map `map[string]interface{}`.

To add your own parser, write a function that implements the `ParseFunc` interface and then register it using `p.Handle(inputType, HandlerFunc)`. After that, it will be available from `p.Parse()` just like the built-in parsers.

See the example code below for more info. 

```
package main

import (
	"fmt"
	"strings"
	"text/scanner"

	parser "github.com/delputnam/parser"
)

var json = `
{
  "json-string":"foo",
  "json-array":["bar","baz","qux"],
  "json-object":{
    "msv5":"corge",
    "msv6":"grault",
    "msv7":"garply"
  }
}
`
var yaml = `
yaml-scalar : foo
yaml-sequence :
  - bar
  - baz
  - qux
yaml-nested-sequences:
  - [cat, dog, mouse]
  - [goat, cow, horse]
  - [tina, louise, gene]
yaml-seq-of-maps:
  -
    name: archer
    job: field agent
  -
    name: barry
    job: evil cyborg
  -
    name: woodhouse
    job: itchy butler
`
var toml = `
key = "value"

[table]
pi = 3.141592654
e = 2.718281828
c = 299792458
`

var myType = `
  myKey ~ myValue
  key2 ~ value2
`

func main() {
	p := parser.NewParser()
	data, err := p.Parse("json", json)
	if err != nil {
		fmt.Printf("Err: %v\n", err)
	}
	fmt.Printf("json: %v\n", data)

	data, err = p.Parse("yaml", yaml)
	if err != nil {
		fmt.Printf("Err: %v\n", err)
	}
	fmt.Printf("yaml: %v\n", data)

	data, err = p.Parse("toml", toml)
	if err != nil {
		fmt.Printf("Err: %v\n", err)
	}
	fmt.Printf("toml: %v\n", data)

	p.Handle("myType", MyHandler)
	data, err = p.Parse("myType", myType)
	if err != nil {
		fmt.Printf("Err: %v\n", err)
	}
	fmt.Printf("myType: %v\n", data)
}

// MyHandler decodes "myType" input into a go map[string]interface{}
// "myType" is just a list of keys and values separated by a tilde (~)
// This is only to demonstrate the use of p.Handle(). Don't use this code.
func MyHandler(input string) (map[string]interface{}, error) {
	out := make(map[string]interface{})

	var s scanner.Scanner
	var tok rune
	var key, value string

	s.Init(strings.NewReader(input))
	for tok != scanner.EOF {
		tok = s.Scan()
		text := s.TokenText()
		if text == "~" {
			tok = s.Scan()
			value = s.TokenText()
			out[key] = value
		}
		key = s.TokenText()
	}

	return out, nil
}
```

## License

This code is licensed under the MIT license.  See [LICENSE](LICENSE) for the full license text.
