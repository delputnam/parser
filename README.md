# Parser

Parser is an extensible parser that decodes
multiple data formats and extracts the results
into a go map.

## Features

JSON, YAML, and TOML handlers are included.  You can easily add your own handler by writing a function that implements `ParseFunc`. Then add your function by calling: `Handle("my-type", MyTypeFunc)`. See the code below for an example.

## Install

```
go get github.com/delputnam/parser
```

## How to use
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

// MyHandler decodes my-type input into a go map[string]interface{}
// my-type is just a list of keys and values separated by a tilde (~)
// Warning: This function is just to demonstrate how to add your own
// parser handler and should not be used for anything else...ever.
// There is no error checking and it probably won't be useful for
// almost anything...ever.
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
