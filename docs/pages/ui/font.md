# `font`

Handles font loading and font characters.

## FontCharacter (`struct`)

Stores information relating to a single font character loaded with `request`.

### Fields

`unicode: u32` - Unicode value of the character.

`size: c_uint` - Pixel size of the character.

`suballocation: TextureAtlas2D.TextureSuballocation` - Location of the character in the font's atlas.

`advance: u16` - Indicates how many pixels the next character should "advance" to after this one (used in text rendering).

`bearing_x: i16` - Indicates the pixel x-offset of the character (used in text rendering).

`bearing_y: u16` - Indicates the pixel y-offset of the character (used in text rendering).

## Font (`struct`)

Stores font data and performs font loading operations.

### Fields

`interface: *VkInterface` - Vulkan interface.

`vk_allocator: *VulkanAllocator` - Vulkan allocator.

`typeface: freetype.FT_Face` - FreeType typeface object.

`typesize: c_uint` - Current pixel size of any new requested characters.

`atlas: ?*TextureAtlas2D` - Texture atlas to load the font characters onto.

`characters: std.ArrayList(FontCharacter)` - List of all characters added to this font instance.

### Public Functions

**`deinit(self: *Font) void`**

Deinitializes the font class and releases all resources.

**`init(interface: *VkInterface, vk_allocator: *VulkanAllocator, path: []const u8, size: c_uint, atlas: ?*TextureAtlas2D) !Font`**

Initializes the font class. If `atlas` is null, it will need to be set before `request` is called.

**`request(self: *Font, unicode: u32, size: f32) !FontCharacter`**

Requests a unicode character to be added to the font. If the character is already available, then the associated `FontCharacter` is returned. Otherwise, a new one is allocated from the `atlas` and added to `characters` before being returned.

**`remove_all_size_characters(self: *Font, size: f32) !void`**

Removes all characters of a specific font size.

**`remove_character(self: *Font, unicode: u32, size: f32) !void`**

Removes a character from the font's character list and texture atlas.

**`set_font_size(self: *Font, size: c_uint) void`**

Updates the `typesize`. This change only applies to new requested characters, currently held ones will maintain their current sizes. You can request the same character(s) in multiple sizes.

### Private Functions

**`add_unicode_glyph(self: *Font, unicode: u32) !FontCharacter`**

Adds a unicode character directly to the atlas.

## Errors

### `FontError`

#### Values

**UIisUninitialized**: Ashbloom's UI is uninititalized (remember to call **`ash.ui.core.init`**).

**FailedToLoadFont**: Failure to load a font file.

**FailedToLoadGlyph**: Failure to load a character into a font.

**NoAtlas**: `atlas` is `null`.
