# ui

Widgets for Quirk's [`window`](https://github.com/AlexVachon/quirk-window)
package: labels, buttons, checkboxes, text inputs, sliders and lists,
laid out in rows and columns, with mouse, keyboard and Tab focus handled
for you.

```quirk
use ui

app := ui.App("Greeter", 360, 220)
name := ui.TextInput(placeholder="Your name")
greeting := ui.Label("")
app.root = ui.Column([
    ui.Label("What's your name?"),
    ui.Row([name, ui.Button("Greet", on_click=fn() { greeting.text = "Hello, " + name.text + "!" })]),
    greeting,
], padding=16)
app.run()
```

## Widgets

| | |
|---|---|
| `ui.Label(text, color=, align=)` | Text (`"\n"` for more lines) |
| `ui.Button(text, on_click=)` | Click, or Enter / Space when focused |
| `ui.Checkbox(text, checked=, on_change=)` | `on_change(checked)` |
| `ui.TextInput(text=, placeholder=, on_change=, on_submit=, columns=)` | One-line field: cursor keys, Backspace / Delete, Enter submits |
| `ui.Slider(min=, max=, value=, on_change=, step=)` | Drag or arrow keys |
| `ui.ListView(items, on_select=, rows=)` | Click / arrows / wheel; `on_select(index, item)` |

## Layout

`ui.Column(children, padding=, spacing=)` stacks top to bottom,
`ui.Row(...)` side by side, `ui.Panel(...)` is a Column on a raised
surface, `ui.Spacer()` takes free space. Any widget with `expand = true`
gets a share of its container's free space; `visible = false` hides it,
`enabled = false` dims it.

## App

`ui.App(title, width, height, theme=, resizable=, headless=)` opens the
window. Set `root`, then `run(fps=60)`. `app.on_frame = fn() { ... }`
runs every frame; `app.step()` runs one frame and `app.dispatch(event)`
routes one event — for your own loop or headless tests. Themes:
`ui.dark()` (default), `ui.light()`, or your own `ui.Theme`.

Your own widgets: subclass `ui.Widget` and override `measure`, `draw`,
`handle` (and `children` for containers).

## Requirements

- The Quirk compiler 7.21.0+.
- SDL2 for `window` (`sudo apt install libsdl2-2.0-0` / `brew install sdl2`).

Bundled with the compiler at `<QUIRK_HOME>/packages/ui/`, so `use ui`
works out of the box; this repo is its canonical home:

```bash
quirk install ui            # latest tag from this repo
quirk install ui@1.0.0      # pinned
```

## License

MIT (see [`LICENSE`](LICENSE)).
