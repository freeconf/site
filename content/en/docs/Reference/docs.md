---
title: Doc generation
weight: 1000
description: >
    Generating documentation from your yang files.
---

FreeCONF comes with a utility to help generate documentation.  Your options include
1. HTML format
2. Markdown - Useful for storing in github.com, gitlab.com
3. Graphviz dot that can be then turned into SVG diagram that then can be added to top of HTML page
4. Raw JSON which can then be used as input to your own template script
5. Export template you can edit and pass back in


## Usage

```
$ go run github.com/freeconf/yang/cmd/fc-yang doc -help

Usage of fc-yang:
  -f string
    	output format. available formats include html, md, json or dot. (default "none")
  -img-link string
    	Link to image for HTML templates. Default is (module-name).svg.
  -module string
    	Module to be documented.
  -off value
    	disable this feature.  You can specify -off multiple times to disable multiple features. You cannot specify both on and off however.
  -on value
    	enable this feature.  You can specify -on multiple times to enable multiple features. You cannot specify both on and off however.
  -t string
    	Use the template instead of the builtin template.
  -title string
    	Title. (default "RESTful API")
  -x	export the builting template to stdout. You can then edit template and pass it back in using -t option.  Be sure to pick correct format.
  -ypath string
    	Path to YANG files
```

## Optional Graphviz

For the SVG, you will need to [install Graphviz](https://graphviz.org/download/).  

Example:
```bash
sudo apt install graphviz
```

## Example commands

**Generate HTML with SVG image at top**
```
fc-yang doc -f dot -module fc-restconf -ypath yang > fc-restconf.dot
dot -Tsvg fc-restconf.dot -o fc-restconf.svg
fc-yang doc -f html -module fc-restconf -title "FreeCONF RESTCONF" -ypath yang > fc-restconf.html
```

## Example output - HTML with SVG

![fc-restconf API](/docs-example-html.png)


## Customize by tweaking the template

```
fc-yang doc -f html -module my-mod -x > doc.template
# Edit doc.template
fc-yang doc -f html -t doc.template -title "My API" -module my-mod -ypath yang > my-api.html
```
