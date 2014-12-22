# display: a browser-based graphics server

Originally forked from [gfx.js](https://github.com/clementfarabet/gfx.js/).

The initial goal was to remain compatible with the torch/python API of `gfx.js`,
but remove the term.js/tty.js/pty.js stuff which is served just fine by ssh.

Compared to `gfx.js`:

  - no terminal windows (no term.js)
  - [dygraphs](http://dygraphs.com/) instead of nvd3 (have built in zoom and are perfect for time-series plots)
  - plots resize when windows are resized
  - images support zoom and pan
  - image lists are rendered as one image to speed up loading
  - windows remember their positions
  - implementation not relying on the filesystem, supports remote clients (sources)

## Technical overview

The server is a trivial message forwarder:

    POST /events -> EventSource('/events')

The Lua client sends JSON commands directly to the server. The browser script
interprets these commands, e.g.

    { command: 'image', src: 'data:image/png;base64,....', title: 'lena' }

### Supported commands

Common parameters:
  - `id`: identifier of the window to be reused (pick a random one if you want a new window)
  - `title`: title for the window title bar

`image` creates a zoomable `<img>` element
  - `src`: URL for the `<img>` element
  - `width`: initial width in pixels
  - `labels`: array of 3-element arrays `[ x, y, text ]`, where `x`, `y` are the coordinates
    `(0, 0)` is top-left, `(1, 1)` is bottom-right; `text` is the label content

`plot` creates a Dygraph, all [Dygraph options](http://dygraphs.com/options.html) are supported
  - `file`: see [Dygraph data formats](http://dygraphs.com/data.html) for supported formats
  - `labels`: list of strings, first element is the X label

## Installation and usage

Install via:

    luarocks install https://raw.githubusercontent.com/szym/display/master/display-scm-0.rockspec

Launch the server:

    ~/.display/run.js &

See `example.lua` for sample usage.

    disp = require 'display'
    disp.image(image.lena())
    disp.plot(torch.cat(torch.linspace(0, 10, 10), torch.randn(10), 2))

![](https://raw.github.com/szym/display/master/example.png)

Note, there is no authentication and by default the server listens on external IP, so **don't use "as is"
for sensitive data**.
