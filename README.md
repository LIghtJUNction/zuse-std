# Zuse standard library

Reusable Zuse functions for drawing, desktop setup, explicit text input, and timing.
The `std` package is source code: installation stores a pinned Git commit locally,
and `import` registers its functions. Desktop actions begin when a script calls them.

Version `0.1.0` requires Zuse **0.6.0 or later**, with `pkg`, window anchors,
`mouse.stroke`, and the `draw.*` primitives. Use `zuse --version` and `zuse schema`
to check the installed build.

## Install and inspect

Create a project directory, then add the tagged package:

```sh
mkdir my-drawing
cd my-drawing
zuse pkg init my-drawing
zuse pkg add https://github.com/LIghtJUNction/zuse-std.git --rev v0.1.0
zuse -c 'import(path="pkg:std",as="std"); functions(namespace="std")'
```

Commit the project's `zuse.json` and `zuse.lock.json`. The manifest records the
requested source revision; the lockfile records its full Git commit. Another
checkout restores the locked sources with:

```sh
zuse pkg install --locked
```

The package has no third-party package dependencies. Import uses installed files
and does not fetch code or run installation hooks. Git transport and the package
manager remain part of the installation process.

## Import a module

Import the whole library for `std.desktop`, `std.drawing`, `std.input`, and
`std.timing`. Import one export when a script needs a smaller namespace:

```zuse
import(path="pkg:std/drawing", as="art")
functions(namespace="art")
```

Function descriptions and parameter names are included in the catalog returned
by `functions`. Arguments are named and required; the table below lists every
public function. Source files are the authoritative implementation.

| Export | Function | Behavior |
| --- | --- | --- |
| [desktop](desktop.zuse) | `inspect()` | List windows and inspect held keyboard modifiers. |
| desktop | `focus(window_id)` | Focus the specified window and wait 200 ms. |
| desktop | `bind_canvas(name,window_id,x,y,width,height)` | Focus, borrow the system pointer, and create a window-local anchor. |
| desktop | `click(canvas,x,y)` | Click a local point with 30 ms input settling and 70 ms application settling. |
| desktop | `capture(window_id,path)` | Save a window screenshot at the requested path. |
| desktop | `release_canvas(name)` | Clear the named anchor and release the session's mouse handle. |
| [drawing](drawing.zuse) | `stroke(canvas,points,duration)` | Draw an open polyline with explicit timing. |
| drawing | `line(canvas,x1,y1,x2,y2)` | Draw a line in 180 ms. |
| drawing | `rectangle(canvas,x,y,width,height)` | Outline a rectangle in 240 ms. |
| drawing | `circle(canvas,x,y,radius)` | Outline a circle with 64 segments in 360 ms. |
| drawing | `polygon(canvas,points,duration)` | Close and outline a polygon with explicit timing. |
| drawing | `arrow(canvas,x1,y1,x2,y2,head)` | Draw a line and an explicitly supplied arrowhead polyline. |
| [input](input.zuse) | `type_in(window_id,text)` | Focus the specified window, wait 200 ms, then type into its current text target. |
| input | `key_in(window_id,key)` | Focus the specified window, wait 200 ms, then submit a key or chord. |
| input | `set_text(node_path,text)` | Focus an observed accessibility node and replace its text. |
| [timing](timing.zuse) | `start()`, `stop()`, `status()`, `reset()` | Forward to the session timer operations. |
| timing | `pause(ms)` | Wait within the script's configured wait budget. |

## Draw on an observed canvas

Observe the intended window and its canvas first. Replace `WINDOW_ID` and the
region below with those observations; the numbers are examples. Select the
application's brush, width, and color before executing drawing functions.

```zuse
import(path="pkg:std", as="std")
std.desktop.bind_canvas(name="board", window_id="WINDOW_ID", x=160, y=180, width=640, height=480)
std.timing.start()
std.drawing.rectangle(canvas="board", x=40, y=40, width=240, height=140)
std.drawing.circle(canvas="board", x=160, y=110, radius=45)
std.drawing.arrow(canvas="board", x1=310, y1=110, x2=430, y2=110, head="410,90;430,110;410,130")
std.timing.stop()
std.desktop.capture(window_id="WINDOW_ID", path="canvas.png")
std.desktop.release_canvas(name="board")
```

The drawing functions use the application's current brush and left mouse
button. They outline shapes; they do not set colors or fill areas. `points` and
`head` use `x,y;x,y` strings. The arrowhead is supplied explicitly because Zuse
function parameters currently support substitution, not coordinate arithmetic.

Coordinates are local to the named canvas anchor. The runtime rejects an anchor
when its window identity, focus, position, dimensions, or scale changes. It does
not track scrolling or zooming inside the page. Re-observe changed geometry
before binding again. A completed input receipt means input was submitted;
inspect the screenshot to confirm the application drew the intended result.

The fixed waits make these small compositions easier to reuse; they are not
application-readiness detection. For long workflows, use the runtime's bounded
observation and waiting primitives. Multiple scripts can retain functions and
anchors in `zuse shell`; its idle timeout controls cleanup of the session.

## Validate the package

These commands load declarations, inspect the catalog, and validate every
public function without sending desktop input:

```sh
zuse run examples/catalog.zuse
zuse check examples/validate-api.zuse
```

The validation fixture intentionally contains placeholder window and node IDs.
Use `check` for that file. To verify an installed package, run the safe catalog
example from the consuming project, or execute the inline catalog command in
the installation section.

## Publish a package

A package consists of a `zuse.json` manifest and ordinary `.zuse` source files.
This repository is a small example: [the manifest](zuse.json) names the package,
its version, its entry module, and its public exports. A package author can use
their own Git repository and publish a version tag; no central registry account
is required.

After committing reviewed source and configuring that repository's `origin`,
inspect and publish the current manifest version:

```sh
zuse pkg publish --dry-run
zuse pkg publish
gh repo edit OWNER/REPO --add-topic zuse --add-topic zuse-package
```

Add the `zuse` and `zuse-package` GitHub topics so other users can discover the
repository with `gh search repos --topic zuse`. The topics are discovery labels;
consumers should inspect the manifest and source before choosing a package.

Use a new version for changed source. Consumers select a source revision with
`pkg add` or `pkg update` and commit the resulting lockfile. See the Zuse
[package-management documentation](https://github.com/LIghtJUNction/zuse/blob/main/docs/packages.md)
for command behavior, dependency resolution, and the lockfile format.

## License

MIT. See [LICENSE](LICENSE).
