# blessed

A curses-like library for node.js.

Blessed was originally written to only support the xterm terminfo, but can
now parse and compile any terminfo to be completely portable accross all
terminals. See the `tput` example below.

I want this library to eventually become a high-level library for terminal
widgets.

## Example Usage

This will actually parse the xterm terminfo and compile every
string capability to a javascript function:

``` js
var Tput = require('blessed').Tput
  , tput = Tput('xterm');

console.log(tput.setaf(4) + 'hello' + tput.sgr0());
```

To play around with it on the command line, it works just like tput:

``` bash
$ tput.js setaf 2
$ tput.js sgr0
$ echo "$(tput.js setaf 2)hello world$(tput.js sgr0)"
```

The higher level functionality is exposed in the main `blessed` module:

``` js
var blessed = require('blessed')
  , program = blessed();

program.on('keypress', function(ch, key) {
  if (key.name === 'q') {
    program.clear();
    program.disableMouse();
    program.showCursor();
    program.normalBuffer();
    process.exit(0);
  }
});

program.on('mouse', function(data) {
  if (data.action === 'mouseup') return;
  program.move(1, program.rows);
  program.eraseInLine('right');
  if (data.action === 'wheelup') {
    program.write('Mouse wheel up at: ' + data.x + ', ' + data.y);
  } else if (data.action === 'wheeldown') {
    program.write('Mouse wheel down at: ' + data.x + ', ' + data.y);
  } else if (data.action === 'mousedown' && data.button === 'left') {
    program.write('Left button down at: ' + data.x + ', ' + data.y);
  } else if (data.action === 'mousedown' && data.button === 'right') {
    program.write('Right button down at: ' + data.x + ', ' + data.y);
  } else {
    program.write('Mouse at: ' + data.x + ', ' + data.y);
  }
  program.move(data.x, data.y);
  program.bg('red');
  program.write(' ');
  program.bg('!red');
});

program.on('focus', function() {
  program.move(1, program.rows);
  program.write('Gained focus.');
});

program.on('blur', function() {
  program.move(1, program.rows);
  program.write('Lost focus.');
});

program.alternateBuffer();
program.enableMouse();
program.hideCursor();
program.clear();

program.move(1, 1);
program.bg('black');
program.write('Hello world', 'blue fg');
program.setx((program.cols / 2 | 0) - 4);
program.down(5);
program.write('Hi again!');
program.bg('!black');
program.feed();

program.getCursor(function(err, data) {
  if (!err) {
    program.write('Cursor is at: ' + data.x + ', ' + data.y + '.');
    program.feed();
  }

  program.charset('SCLD');
  program.write('abcdefghijklmnopqrstuvwxyz0123456789');
  program.charset('US');
  program.setx(1);
});
```

## High-level Documentation

### Example

This will render a box with ascii borders containing the text 'Hello world!',
centered horizontally and vertically.

``` js
var blessed = require('blessed')
  , program = blessed()
  , screen = new blessed.Screen({ program: program });

screen.append(new blessed.Box({
  top: 'center',
  left: 'center',
  width: '50%',
  height: '50%',
  border: {
    type: 'ascii',
    fg: 1
  },
  fg: 3,
  bg: 5,
  content: 'Hello world!',
}));

screen.on('keypress', function(ch, key) {
  if (key.name === 'escape') {
    process.exit(0);
  }
});

screen.render();
```

### Widgets

Blessed comes with a number of high-level widgets so you can avoid all the
nasty low-level terminal stuff.

#### Node (from EventEmitter)

The base node which everything inherits from.

Options:

- screen - the screen to be associated with.
- parent - the desired parent.
- children - an arrray of children.

Properties:

- inherits all from EventEmitter.
- children - array of node's children.

Events:

- inherits all from EventEmitter.
- remove - received when node is removed from it's current parent.
- reparent - received when node gains a new parent.

Methods:

- inherits all from EventEmitter.
- prepend(node) - prepend a node to this node's children.
- append(node) - append a node to this node's children.
- remove(node) - remove child node from node.
- detach() - remove node from its parent.

#### Screen (from Node)

The screen on which every other node renders.

Options:

- program - the blessed Program to be associated with.

Properties:

- inherits all from Node.
- program - the blessed Program object.
- tput - the blessed Tput object.
- focused - top of the focus history stack.
- width - width of the screen (same as `program.cols`).
- height - height of the screen (same as `program.rows`).
- left - left offset, always zero.
- right - right offset, always zero.
- top - top offset, always zero.
- bottom - bottom offset, always zero.

Events:

- inherits all from Node.
- mouse - received on mouse events.
- keypress - received on key events.
- element [name] - global events received for all elements.

Methods:

- inherits all from Node.
- alloc() - allocate a new pending screen buffer and a new output screen buffer.
- draw(start, end) - draw the screen based on the contents of the screen buffer.
- render() - render all child elements, writing all data to the screen buffer and drawing the screen.
- clearRegion(x1, x2, y1, y2) - clear any region on the screen.
- fillRegion(attr, ch, x1, x2, y1, y2) - fill any region with a character of a certain attribute.
- focus(offset) - focus element by offset of focusable elements.
- focusPrev() - focus previous element in the index.
- focusNext() - focus next element in the index.
- focusPush(element) - push element on the focus stack (equivalent to `screen.focused = el`).
- focusPop()/focusLast() - pop element off the focus stack.

#### Element (from Node)

The base element.

Options:

- fg, bg, bold, underline - attributes.
- border - border object, see below.
- content - element's text content.
- clickable - element is clickable.
- input - element is focusable and can receive key input.
- hidden - whether the element is hidden.
- label - a simple text label for the element.

Properties:

- inherits all from Node.
- border - border object.
  - type - type of border (`ascii` or `bg`). `bg` by default.
  - ch - character to use if `bg` type, default is space.
  - bg, fg - border foreground and background, must be numbers (-1 for default).
  - bold, underline - border attributes.
- position - raw width, height, and offsets.
- content - text content.
- hidden - whether the element is hidden or not.
- fg, bg - foreground and background, must be numbers (-1 for default).
- bold, underline - attributes.
- width - calculated width.
- height - calculated height.
- left - calculated absolute left offset.
- right - calculated absolute right offset.
- top - calculated absolute top offset.
- bottom - calculated absolute bottom offset.
- rleft - calculated relative left offset.
- rright - calculated relative right offset.
- rtop - calculated relative top offset.
- rbottom - calculated relative bottom offset.

Events:

- inherits all from Node.
- mouse - received on mouse events for this element.
- keypress - received on key events for this element.
- move - received when the element is moved.
- resize - received when the element is resized.

Methods:

- inherits all from Node.
- render() - write content and children to the screen buffer.
- setContent(text) - set the content.
- hide() - hide element.
- show() - show element.
- toggle() - toggle hidden/shown.
- focus() - focus element.

#### Box (from Element)

A box element which draws a simple box containing `content` or other elements.

Inherits all options, properties, events, and methods from Box.

#### Text (from Element)

An element similar to Box, but geared towards rendering simple text elements.

Options:

- inherits all from Element.
- fill - fill the entire line with chosen bg until parent bg ends, even if
  there is not enough text to fill the entire width.

Inherits all options, properties, events, and methods from Element.

#### Line (from Box)

A simple line which can be `ascii` or `bg` styled.

Options:

- inherits all from Box.
- orientation - can be `vertical` or `horizontal`.
- type, bg, fg, ch - treated the same as a border object.

Inherits all options, properties, events, and methods from Box.

#### ScrollableBox (from Box)

A box with scrollable content.

Options:

- inherits all from Box.
- baseLimit - a limit to the childBase. default is `Infinity`.
- alwaysScroll - a option which causes the ignoring of `childOffset`. this in
  turn causes the childBase to change every time the element is scrolled.

Properties:

- inherits all from Box.
- childBase - the offset of the top of the scroll content.
- childOffset - the offset of the chosen item (if there is one).

Events:

- inherits all from Box.
- scroll - received when the element is scrolled.

Methods:

- scroll(offset) - scroll the content by an offset.

#### List (from ScrollableBox)

A scrollable list which can display selectable items.

Options:

- inherits all from ScrollableBox.
- selectFg, selectedBg - foreground and background for selected item, treated
  like fg and bg.
- selectedBold, selectedUnderline - character attributes for selected item,
  treated like bold and underline.
- mouse - whether to automatically enable mouse support for this list (allows
  clicking items).
- items - an array of strings which become the list's items.

Properties:

- inherits all from ScrollableBox.

Events:

- inherits all from ScrollableBox.
- select - received when an item is selected.

Methods:

- inherits all from ScrollableBox.
- add(text) - add an item based on a string.
- select(index) - select an index of an item.
- move(offset) - select item based on current offset.
- up(amount) - select item above selected.
- down(amount) - select item below selected.

#### ScrollableText (from ScrollableBox)

A scrollable text box which can display and scroll text, as well as handle pre-existing newlines and escape codes.

Options:

- inherits all from ScrollableBox.
- mouse - whether to enable automatic mouse support for this element.

Properties:

- inherits all from ScrollableBox.

Events:

- inherits all from ScrollableBox.

Methods:

- inherits all from ScrollableBox.

#### Input (from Box)

A form input.

#### Textbox (from Input)

A box which allows text input.

Options:

- inherits all from Input.

Properties:

- inherits all from Input.

Events:

- inherits all from Input.

Methods:

- inherits all from Input.
- setInput(callback) - grab key events and start reading text from the
  keyboard. takes a callback which receives the final value.
- setEditor(callback) - open text editor in $EDITOR, read the output from the
  resulting file. takes a callback which receives the final value.

#### ProgressBar (from Input)

A progress bar allowing various styles.

Options:

- inherits all from Input.
- barFg, barBg - (completed) bar foreground and background.
- ch - the character to fill the bar with (default is space).
- filled - the amount filled (0 - 100).

Properties:

- inherits all from Input.

Events:

- inherits all from Input.
- reset - bar was reset.
- complete - bar has completely filled.

Methods:

- inherits all from Input.
- progress(amount) - progress the bar by a fill amount.
- reset() - reset the bar.

### Positioning

Offsets may be a number, a percentage (e.g. `50%`), or a keyword (e.g.
`center`).

Dimensions may be a number, or a percentage (e.g. `50%`).

Positions are treated almost *exactly* the same as they are in CSS/CSSOM when
an element has the `position: absolute` CSS property.

When an element is created, it can be given coordinates in its constructor:

``` js
var box = new blessed.Box({
  left: 'center',
  top: 'center',
  bg: 3,
  width: '50%',
  height: '50%'
});
```

This tells blessed to create a box, perfectly centered **relative to its
parent**, 50% as wide and 50% as tall as its parent.

To access the calculated offsets, relative to the parent:

``` js
console.log(box.rleft);
console.log(box.rtop);
```

To access the calculated offsets, absolute (relative to the screen):

``` js
console.log(box.left);
console.log(box.top);
```

#### Overlapping offsets and dimensions greater than parents'

This still needs to be tested a bit, but it should work.

### Rendering

To actually render the screen buffer, you must call `render`.

``` js
box.setContent('Hello world.');
screen.render();
```

Elements are rendered with the lower elements in the children array being
painted first. In terms of the painter's algorithm, the lowest indicies in the
array are the furthest away, just like in the DOM.

### Testing

- For an interactive test, see `test/widget.js`.
- For a less interactive position testing, see `test/widget-pos.js`.

## License

Copyright (c) 2013, Christopher Jeffrey. (MIT License)

See LICENSE for more info.
