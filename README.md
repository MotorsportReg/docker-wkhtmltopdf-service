# wkhtmltopdf as a web service

A dockerized webservice written in [Go](https://golang.org/) that uses [wkhtmltopdf](http://wkhtmltopdf.org/) to convert HTML into documents (images or pdf files).

## Build & Run

`docker build -t wkhtmltopdf . && docker run --rm -it -p 3000:3000 -v $(pwd)/app/ssl:/app/ssl wkhtmltopdf`

For security purposes, we do not embed the SSL certificates in the image. Instead, mount a volume containing a cert.pem and key.pem to `/app/ssl`

## Usage

The service listens on port 3000 for POST requests over SSL on the root path (`/`). Any other method returns a `405 not allowed` status. Any other path returns a `404 not found` status.

The body should contain a JSON-encoded object containing the following parameters:

One of:
- **content**: The HTML/CSS to be converted.
- **url**: The URL of the page to fetch and convert.

And: 
- **output**: The type of document to generate, can be either `jpg`, `png` or `pdf`. Defauts to `pdf` if not specified. Depending on the output type the appropriate binary is called.
- **options**: A list of key-value arguments that are passed on to the appropriate `wkhtmltopdf` binary. Boolean values are interpreted as flag arguments (e.g.: `--greyscale`).
- **cookies**: A list of key-value arguments that are passed on to the appropriate `wkhtmltopdf` binary as separate `cookie` arguments.

**Example:** posting the following JSON:

```
{
  "url": "http://www.google.com",
  "options": {
    "margin-bottom": "1cm",
    "orientation": "Landscape",
    "grayscale": true
  },
  "cookies": {
    "foo": "bar",
    "baz": "foo"
  }
}
```

will have the effect of the following command-line being executed on the server:

```
/usr/local/bin/wkhtmltopdf --margin-bottom 1cm --orientation Landscape --grayscale --cookie foo bar --cookie baz foo http://www.google.com -
```

The `-` at the end of the command-line is so that the document contents are redirected to stdout so we can in turn redirect it to the web server's response stream.

When using `jpg` or `png` output, the set of options you can pass are actually more limited. Please check [wkhtmltopdf usage docs](http://wkhtmltopdf.org/docs.html) or rather just use `wkhtmltopdf --help` or `wkhtmltoimage --help` to get help on the available command-line arguments.

You can also pass in the HTML/CSS directly like so:

```
{
  "content": "<html><body><h1>My Web Page<\/h1><p>The HTML that would have been fetched<\/p><\/body><\/html>",
  "options": {
    "margin-bottom": "1cm",
    "orientation": "Landscape",
    "grayscale": true
  },
  "cookies": {
    "foo": "bar",
    "baz": "foo"
  }
}
```

## Known limitations / TODOs

There are challenges with converting raw HTML when you have relative links. There are known workarounds that we haven't yet tried (see [this stack overflow question](http://stackoverflow.com/questions/21775572/wkhtmltopdf-relative-paths-in-html-with-redirected-in-out-streams-wont-work) for more information).

The server currently does not check if the arguments are valid, just passes them on to the `wkhtmltopdf` and `wkhtmltoimage` binaries. If you pass a wrong argument you don't currently get an error, just an empty image or pdf back.


## License

This code is released under the [MIT License](http://opensource.org/licenses/MIT).
