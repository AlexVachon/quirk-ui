# ui

Widgets for Quirk's [`window`](https://github.com/AlexVachon/quirk-window)
package: labels, buttons, checkboxes, text fields and areas, sliders,
lists and dropdowns, laid out in rows, columns and scroll views, with
mouse, keyboard, Tab focus, the clipboard and tooltips handled for you.

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
| `ui.RadioGroup(options, selected=, on_change=)` | One of several; click or arrows; `on_change(index, item)` |
| `ui.TextInput(text=, placeholder=, on_change=, on_submit=, columns=)` | One-line field; Enter submits |
| `ui.TextArea(text=, rows=, columns=, on_change=, wrap=)` | Multi-line text; scrolls; Enter starts a line; `wrap=true` breaks long lines at spaces |
| `ui.Slider(min=, max=, value=, on_change=, step=)` | Drag or arrow keys |
| `ui.ProgressBar(value=, max=, show_text=)` | A filled bar; set `value` as work progresses |
| `ui.ListView(items, on_select=, rows=)` | Click / arrows / wheel; `on_select(index, item)` |
| `ui.Dropdown(options, selected=, on_change=)` | Pick one from a list that opens below; `on_change(index, item)` |
| `ui.TreeView(roots, on_select=, rows=)` | `ui.TreeNode(text, children, expanded=, data=)`s; click the arrow / double-click / Left / Right to expand and collapse |
| `ui.Table(columns, rows, on_select=, visible_rows=, editable=, on_edit=)` | Rows of values under headers; click a header to sort (again: reverse); `on_select(index, row)`; with `editable=true`, double-click / Enter / F2 edits a cell, `on_edit(index, column, value)` |
| `ui.DatePicker(year, month, day, on_change=)` | A date field with a month calendar; `text()` is "YYYY-MM-DD"; `ui.days_in_month`, `ui.weekday` |
| `ui.ColorPicker(color=, on_change=)` | Saturation / brightness square, hue strip, swatch with its hex code; `ui.hsv_to_color`, `ui.color_to_hsv` |
| `ui.ListItem(text, icon=, data=)` | An entry with an image icon, for a ListView or a Table cell; `.str()` is its text |
| `ui.Canvas(on_draw=, on_event=, width=, height=)` | Draw it yourself: `on_draw(win, x, y, w, h)` each frame (clipped), `on_event(e)` for its mouse / key events |
| `ui.ListView(..., reorderable=, on_reorder=)` | With `reorderable=true` rows move by dragging; `on_reorder(from, to)` |

## Menus

```quirk
file := ui.Menu([
    ui.MenuItem("Open", on_click=open_file, shortcut="Ctrl+O"),
    ui.MenuItem("-"),                                   // a separator
    ui.MenuItem("Quit", on_click=fn() { app.close() }),
])
app.root = ui.Column([ui.MenuBar(["File"], [file]), content])
app.on_key = fn(e) { if e.ctrl and e.key == "o" { open_file() } }
```

Click a title (or move over another while one is open); Up / Down /
Enter / Escape, Left / Right between menus. A `shortcut` ("Ctrl+S",
"Ctrl+Shift+N", "F5", "Delete"; Ctrl is also Cmd) runs its item from
anywhere in the window — menu bar items always, a right-click menu's
while its widget has the focus. `app.on_key` sees every key first. Any widget can have a right-click menu:
`widget.context_menu = ui.Menu([...])`; `menu.open_at(x, y)` opens one
from code.

## Fonts and animation

```quirk
app.theme.font = window.load_font("/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf", 15)
```

Every widget then lays out and draws in that font (proportional text,
any character it has — text fields place the cursor by real widths).
Without one, the built-in 8x16 font is used.

```quirk
app.animate(250, fn(t) { box.width = ui.lerp(40, 300, t) }, ease="out", on_done=fn() { print("done") })
```

`on_step(t)` gets the eased progress each frame (the last call is 1.0);
`ease` is "linear", "in", "out", "in_out" or "back". `animate` returns an
`Animation` with `cancel()`; `app.animating()` says whether one runs.

## Window layout

```quirk
app.root = ui.Column([
    ui.MenuBar(["File"], [file_menu]),
    ui.Toolbar([ui.ToolButton("Save", icon="icons/save.png", on_click=save, tooltip="Save"),
                ui.ToolButton("-"),
                ui.ToolButton("Run", on_click=run)]),
    ui.SplitView(file_list, editor, ratio=30),     // drag the divider
    ui.StatusBar(["Ready", "Ln 1, Col 1"]),
], spacing=0)
```

`ui.SplitView(a, b, vertical=false, ratio=50, min_size=40)`;
`status.set(1, "Ln 7, Col 3")`. A ToolButton's icon is any image file
(PNG, JPEG, BMP, GIF, TGA); a missing one draws as an empty box.

## Dialogs and drag and drop

```quirk
ui.FileDialog(mode="open", dir=".", filter="*.qk", on_pick=fn(path) { load(path) }).show()
```

`mode` is `"open"`, `"save"` (with a name field) or `"folder"`. While open
it takes all input: double-click / Enter opens a folder or picks a file,
Backspace goes up, Escape cancels (`on_cancel`).

Any widget can be dragged onto another: `card.on_drag = fn() => card_id`
makes it draggable (return `null` to refuse), `bin.on_drop = fn(data) {
... }` accepts drops. A label follows the mouse and the target under it is
outlined; `app.dragging()` says whether a drag is on.

Text fields and areas: Shift + arrows / Home / End or a mouse drag
select; double-click selects a word, triple-click a line; Ctrl+A / C /
X / V select all, copy, cut and paste through the system clipboard
(Cmd on macOS); Ctrl+Z / Ctrl+Y (or Ctrl+Shift+Z) undo and redo —
typing a word is one step; Ctrl + Left / Right / Backspace / Delete work
by word.
From code: `field.selected_text()`, `field.select(start, end)`,
`field.select_all()`, `field.copy()`, `cut()`, `paste()`.

Tooltips: set `widget.tooltip = "Hint"`, or inline:
`ui.tooltip(ui.Button("Save", on_click=save), "Write the file")`.
`app.tooltip_delay` (ms, default 500) is how long the mouse rests first.

## Layout

`ui.Column(children, padding=, spacing=)` stacks top to bottom,
`ui.Row(...)` side by side, `ui.Panel(...)` is a Column on a raised
surface, `ui.Spacer()` takes free space. `ui.ScrollView(content,
height=200)` shows `content` in an area that scrolls (wheel, scroll bar,
or Tab to a widget out of view) when the content is taller.
`ui.Tabs(["General", "Advanced"], [page1, page2])` shows one page under a
row of tabs (click, Left / Right, or Ctrl+Tab); `tabs.add(title, page)`. Any widget with `expand = true`
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

- The Quirk compiler 7.33.0+ and `window` 1.3.0+ (clipping, the clipboard, fonts).
- SDL2 for `window` (`sudo apt install libsdl2-2.0-0` / `brew install sdl2`).

Bundled with the compiler at `<QUIRK_HOME>/packages/ui/`, so `use ui`
works out of the box; this repo is its canonical home:

```bash
quirk install ui            # latest tag from this repo
quirk install ui@1.0.0      # pinned
```

## License

MIT (see [`LICENSE`](LICENSE)).
