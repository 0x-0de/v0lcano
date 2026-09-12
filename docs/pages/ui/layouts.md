# `layouts`

Contains some simple UI layouts supported by `theme_basic`. Layouts are essentially a series of `AddChild`, `RemoveChild`, and sometimes `WindowResize` and `Rebuild` callback functions added to elements which automatically set their children's placements. Similar to `theme_basic`, private callback functions are not documented.

## LayoutAxis (`enum`)

### Values

**Horizontal**: X-axis

**Vertical**: Y-axis

## LinearLayoutDirection (`struct`)

Determines how elements are spaced out along an axis/direction on the linear layout.

### Fields

`spacing: f32` - How much spacing is between each element along this direction.

`margin: f32` - How much spacing is given between the borders of the parent element on this axis.

## LinearLayoutProperties (`struct`)

Sets the properties of the element's linear layout.

`alignment: ui.core.Alignment` - After the lines have been calculated, shifts the overall structure in a certain direction (should the lines "lean" left or right? what about top or bottom?).

`primary_axis: LayoutAxis` - The direction that elements will attempt to move along first.

`primary_direction: LinearLayoutDirection` - Determines the spacing and margin of elements along the `primary_axis`.

`secondary_direction: LinearLayoutDirection` - Determines the spacing and margin of elements along the other axis.

## SplitLayoutProperties (`struct`)

Sets the properties of the element's split layout.

### Fields

`primary_axis: LayoutAxis` - Determines the axis that elements will first attempt to split along.

`primary_limit: usize = 0` - Determines how many elements can be split along `primary_axis` before a split along the other axis occurs.

## Public Functions

**`set_layout_linear(e: *Element, properties: LinearLayoutProperties) !void`**

Adds linear layout callback functions to `e`, initializes its `layout_data` to `properties`.

**`set_layout_split(e: *Element, properties: SplitLayoutProperties) !void`**

Adds split layout callback functions to `e`, initializes its `layout_data` to `properties`.