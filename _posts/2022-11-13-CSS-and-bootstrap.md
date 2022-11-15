---
title: CSS and Bootstrap
---

* Including a stylesheet in an HTML file directly:

```html
<head>
	<link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
</head>
```

* Bootstrap is installable with npm using `npm install bootstrap`; puts it into `node_modules/bootstrap/dist/css/bootstrap.min.css`

## SASS

SASS stands for "syntactically awesome style sheets". It is a CSS extension language that includes more sophisticated language features than CSS has. Stylesheets are written in SASS, then pre-processed into CSS prior to being packaged into the deployable site.

SASS adds language features like imports from modules, functions, better indenting, variables.

CSS seems to be growing similar features natively, but they're still in incubation stage. SASS is the way to do this, for now.

## CSS @-rules

@-rules are a set of CSS directives that begin with @ and end with a semi-colon. Most of them have their own, unique syntax, although there are a set of them that are called "group rules" - they allow creation of a group of CSS rules that only apply in the case of the @-rule's condition evaluates to `true`. 

These @-rules can be nested inside other group rules.

### Media Queries

*@media* is a CSS rule used to apply other CSS rules only in the case of specific media attributes being present ("media" meaning characteristics of the output device).  In the below, `screen` is when the browser is rendering normally; `print` would be used for a print preview, for example.

```css
/* Apply the group only if the screen is at least 900px wide */
@media screen and (min-width: 900px) {

}
```

The condition in the `@media` rule is a `media query`: a set of attributes that target media types (`screen`, `print`) and some attributes of that media (view port size, in this case). Available attributes are [documented here](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries), but include things like if the device has a pointer, how large it is, its aspect ratio, if it supports scripting.

### Supports

*@supports* is used to apply a group of CSS rules only in the case that the browser supports a given feature. Eg. the following checks if a particular format of font is supported:

```css
@supports font-format(opentype) {
	/* Stuff */
}
```
## CSS variables

You can assign values to variables and use those later to avoid duplicating literals. Variables are assigned in rules, and are then in scope anywhere that rule applies:

```css
:root {
	--myvar: brown;
}
```

You use the variables with:

```css
p {
	background-color: var(--myvar);
}
```
## CSS selectors

* `div {background: blue}` - select by element type
* `div.myclass {display: inline}` - select by element type and class
* `div div {backgroun: blue}` - select a `div` inside a `div`, with arbitrary number of intermediate parents
* `div > div {background: blue}` - select a `div` whose immediate parent is a `div`

## CSS sizes

Sizes can be specified in:
* *Pixels:* Duh
* *em:* 1 `em` (short for "element") is equal to the font size of the element being styled, in pixels.
* *rem:* 1 `rem` (short for "root element") is equal to the font size of the root `html` element, in pixels.
* *percentage:* the given percentage of the parent element's size.

## Z-order

CSS has a `z-index` property used to put things on the z-axis. It's defined in the CSS spec as an integer, but not given a size ("one or more digits '0' - '9'"), so it can be arbitrarily large, and negative.

An item with explicit `z-index` (ie. non-`auto`) generates a new stacking context.

If two items in the same stacking context have the same z-order, they (and their sub-items, because they create a new sub-context) are painted in order of their position in the document; ie. the later one will be painted on top of the earlier.

The browser paints items in the following order:
* Backgrounds and borders of the element forming the stacking context
* Child contexts with negative `z-index`
* All in-flow (ie. non-positioned) child elements
* Child contexts with zero `z-index` and positioned children
* Child contexts with positive `z-index`.

Bootstrap has defined a number of standard `z-index` values to create layers: dropdown, sticky, fixed, modal-backdrop, modal, popover, tooltip.

## CSS Layout

### CSS Boxes

Every box has the following sections, from inner to outermost:
* Content
* Padding
* Border
* Margin

How these sections interact and render is determined by the box's display type.

Each box in CSS has an inner and outer display type, both settable via the `display` property. Outer determines how the box cooperates with its container (inline or block), and inner determines how the box lays out its children.

If the outer display type is block: 
* The box breaks to a new line
* width, height are respected
* padding, margin & border push other boxes away (but margins collapse)
* The box extends in inline direction to fill its container.  

`<p>` and `<h1>` are examples where outer display is `block` by default.

If the outer display type is inline:
* Box is rendered next to the previous item
* Width & height are ignored
* Vertical padding, margins & borders will still be drawn, but *don't* push other inline boxes away from the box
* Horizontal padding, margins & borders *do* push other inline boxes away from the box (and margins *don't* collapse!).

You can have a display of "inline-block", where it will behave like a block (ie. padding, border & margin push other boxes away) but won't break to a new line.

Inner display type's default is "normal flow". `span`, `i`, `strong` are examples of inline types.

The `display` attribute is set by keywords; there is no syntactical differentiation between the inner and outer display values. Instead, some keywords apply to outer, some apply to inner:
* `block` and `inline` apply to the outside display.
* The remainder refer to inside display. `flow` (the default), `flow-root`, `table`, `flex`, `grid` and `ruby`

### Normal flow

"Normal flow" is the default. Elements are displayed one after the other in the order they appear in the code.

In English:
* Inline elements are displayed next to each other: the "inline direction".
* Block elements are displayed one below the other: the "block direction".

This flips for languages with vertical writing modes, like Japanese.

### flex

One-dimensional layout model, designed to give alignment and distribution capabilities of items. It cvan be either in a row or a column format; not both.

A flexbox has a primary axis called the "flex direction" - it's either `column`, `row`, `column-reverse` or `row-reverse`.

It's left/right, up/down agnostic; its "start edge" flips depending on the language you're working in.

* Enable with `display: flex` on a container.
* The child `flex` attribute is a shorthand one that can mean multiple things:
	* One value, integer: Set the child's `flex-grow`. This specifies how much of the container's remaining space should go to the item. *Remaining space* is defined as the space remaining after all siblings' sizes have been allocated. The remaining space is allocated according to the ratio defined by all sibling's grow factors.
	* One value, size (em, rem, px, %), or [other keywords](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-basis): set the child's `flex-basis`, or initial size. If `flex-grow` and `flex-shrink` are specified, the item may grow or shink accordingly.
	* Two values: the first is `flex-grow`. If the second has no unit, then it becomes `flex-shrink`; `flex-basis` is set to 0. Otherwise, the second becomes `flex-basis`; `flex-shrink` is set to 1.
	* Three values: `flex-grow`, `flex-shrink`, `flex-basis` respectively.
	
* Set  the parent's `flex-direction` to set the direction of the main axis.
* Set `flex-wrap` on the parent to `wrap` to create new rows of items when you overflow in the flex-direction; only a problem with fixed-size subitems.
* Aligning:
	* `align-items` determines aligning on the cross axis: `centre`, `flex-start`, `flex-end`, `stretch` (default)
	* `justify-content` determines position & spacing on the flex-direction: `flex-start` (default), `flex-end`, `center`, `space-around`, `space-between`
* Ordering:
	* Setting `order` on a child in the flexbox changes the order of items. The higher the `order` value, the later in the sort order it's rendered.

### Grid

#### Size and number of tracks (rows/cols)

Turn any container into a grid by setting its display to `grid`. You can set up the number of columns and rows in the grid with shorthand attribs of `grid-template-columns` and `grid-template-rows`.

The template is a list of sizes for the number of columns/rows you want:
* Specify in pixels to hardcode
* Specify in `fr` to give proportional sizing.

eg. `grid-template-colums= 1fr 2fr 1fr`: create three columns; the first is quarter of the available space, the second is half, and the third is the last quarter.

You can mix them: any hardcoded values are taken away from the available space, and the remainder is divided proportionately. A shorthand to repeat column/row definitions is with `repeat`:

```css
/* 2fr, then Repeat 1fr 3 times, then a final 2fr */
gird-template-columns: 2fr repeat(3, 1fr) 2fr
```

You can repeat multiple tracks:

```css
/* 3 sets of the repeated pattern */
gird-template-columns: repeat(3, 20px 1fr 2fr)
```

If you overflow the number of templated cols and rows, the browser will just create more for you. You can specify how you want those to look with `grid-auto-rows` and `grid-auto-columns`.  They work exactly the same as the templated ones, and will be used as a repeating pattern to create new tracks.

You can use `minmax` instead of a size: `minmax(2fr, auto)` will create a track at least `2fr` big, but that will grow as far as needed to render the content.

#### Rendering boxes across multiple tracks

The default is to render boxes in the created cells, one after the other, going in reading order (left to right, then down for English) - like standard flow layout.

However, you can render stuff where you want by using line numbers. Each line in the grid is numbered from 1 at the start of the reading direction.

You then use `grid-column-start`, `grid-column-end`, `grid-row-start` and `grid-row-end` to specify where you want the box.  There is a shorthand `grid-row` and `grid-column` where you can use `<from>/<to>`:

```css
.mybox { 
	grid-row: 1/3;
	grid-column:3
}
```

If you explicitly position only some items in the gird, the remaining items will then do a flow-based positioning across all of the remaining unoccupied cells.

#### Gaps

You can specify the size of the gap between rows, columns or both with `row-gap` and `column-gap`.

`gap` can be used for shorthand:
* A single value sets both row and column
* Two values sets row gap then column gap

## Bootstrap CSS - layout

### Breakpoints

Media queries are used in Angular implement "breakpoints" - pre-defined viewport sizes that match particular device types, like mobiles, laptops, desktops. They enable different views of a design to be implemented to have a reactive UI that can display well on a variety of devices.

Bootstrap has implemented a number of breakpoints that correspond to small, medium, large and extra large devices; each breakpoint is characterised by a viewport width. Maximum and minimum width are specified in different cases in Bootstrap. 

The breakpoints get redefined according to the viewport size; a breakpoint's size will be its configured size (according to Bootstrap) or the appropriate size of the viewport, whichever is the larger. In other words, on a viewport that will fit a `lg` breakpoint, the `sm` and `md` breakpoints are redefined to be the same as `lg`.

### Bootstrap grid

Bootstrap grid is a way to lay out your page. You create rows with cells for your content; the reactive approach of bootstrap allows those rows to stack (ie. render each child at 100% of parent width, so you end up with one column) when the screen gets narrower than the viewport size you used when building.

Bootstrap `container` classes set the width appropriately, and apply some padding and margins. 

`.container` chooses the right breakpoint (and thus width of container) for your viewport. `.container-fluid` always gives you 100% width.

`row` is a flexbox wrapper; you fill it with column divs. Each column div needs a class indicating it's a column:
* `col` is a vanilla flavoured column with `flex` set to `1` (ie. proportional sizing of one).
* `col-x` is a set of classes used to specify how wide you want the column; x is an integer indicating how many 12ths of the row you want to occupy with that column.

#### Reactive sizing with bootstrap grids

Reactive sizing is achieved with the `col-<size>-<count>` classes, where `size` is one of `sm`, `md`, `lg`, `xl` and `xxl`. The `count` is the number of 12ths the column should occupy.

Using these classes, you can specify a different set of columns sizes for each screen size; for a small screen you may want a 50/50 split between two columns, when for larger screens you may want that to be 30/70.

The browser will use the design for the smallest screen size that fits; you can specify a small and medium design, and the medium will be used for everything from medium upwards.

An example with two specified designs:

```html
    <div class="container-fluid">
        <div class="row">
            <div class="col-sm-4 col-md-2">
                First
            </div>
            <div class="col-sm-4 col-md-5">
                One of three cols
            </div>
            <div class="col-sm-4 col-md-5">
                One of three cols
            </div>
        </div>
    </div>
```

The way this is achieved is that Bootstrap declares the various classes if their minimum screen size has been met; ie. for a screen size of 700px, the `col-sm-*` classes will be defined as they're declared within a `@media(min-width:576px)` media query.  

However, the `col-md-*` classes will not exist, as they're inside a query requiring 768px. So, for the above html, only `sm` classes will exist and will be applied.  If the screen is large enough, both the `sm` and `md` classes will exist and will be applied - but as the `md` class is specified later, it will overwrite the CSS attributes applied by the `sm` classes.

Breakpoint-specific classes with a size of `auto` are provided so that you can specify some columns as taking on their natural size for some breakpoints, but fixed for others. Similarly, a class of `col-<size>` is provided, which sets the size to 0% but a grow-factor of 1; specifying this for every column for that breakpoint will make them all of equal size.

Similarly to the above, responsive classes are provided for the alignment and spacing of flex items:
* `align-items-<size>-start|end|baseline|center`
* `justify-content-<size>-evenly|around|between|centre|start|end`
* `flex-<size>-wrap|nowrap`
* Ditto grow and shrink (to set grow factors)

...and in fact, you can specify totall different layouts based on sizes. Grid, flex, inline, block, order, margins, padding, etc.