# `misc`

Contains general and miscellaneous utilities.

## FontEntry (`struct`)

Contains data about an individual font file.

### Fields

`name: []u8` - Name of the font.

`path: []u8` - Path to the font.

### Public Functions

**`deinit(self: *FontEntry, allocator: *const std.mem.Allocator) void`**

Frees the allocated `name` and `path` fields.

## FontFamily (`struct`)

Contains data about a family of font files.

### Fields

`name: []u8` - Name of the font family.

`fonts: []FontEntry` - Fonts contained within the font family.

### Public Functions

**`deinit(self: *FontFamily, allocator: *const std.mem.Allocator) void`**

Frees all `fonts` and other allocated resources.

## Errors

### `FontSearchError`

#### Values

**MissingFont**: Font could not be found.

## Public Functions

**`memcpy_anonymous(dst: *anyopaque, src: *anyopaque, size: usize) void`**

Implementation of C's memcpy(). Takes two anonymous pointers and a given byte size.

**`wrapping_leftshift(comptime T: type, value: T, shift: usize) T`**

Performs a wrapping left-shift.

**`wrapping_rightshift(comptime T: type, value: T, shift: usize) T`**

Performs a wrapping right-shift.

**`get_exe_path() []u8`**

Returns the path of the emitted .exe file at runtime (implementation is OS-specific, only works on Windows & Linux for now).

**`enumerate_system_fonts(allocator: *const std.mem.Allocator) ![]FontEntry`**

Enumarates the available system fonts. Returns a dynamically-allocated array that must be freed if no errors are thrown.

**`search_font_entries(available_fonts: []FontEntry, names: []const []const u8) FontSearchError!FontEntry`**

Given a list of fonts, searches for any desired one, perferring names with lower indexes in the `names` slice.

## Private Functions

**`walk_font_directory(allocator: *const std.mem.Allocator, io: *std.Io.Threaded, dir: *std.Io.Dir, font_list: *std.ArrayList(FontEntry), prefix_str: []const u8) !void`**

Traverses a font directory and adds available font files to a list.