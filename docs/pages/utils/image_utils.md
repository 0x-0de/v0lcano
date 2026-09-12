# `image_utils`

Contains various utilities related to handling images in Vulkan, including loading and storing textures and atlases.

## Texture2D (`struct`)

This struct holds a `VulkanImageAllocation`, `VkImageView`, and `VkSampler`, to be used for rendering textures.

### Fields

`interface: *VkInterface` - VkInterface handle.

`vk_allocator: *VulkanAllocator` - Vulkan allocator.

`image: VulkanAllocator.VulkanImageAllocation = undefined` - Image allocation.

`image_view: ?vk.ImageView` - VkImageView handle.

`sampler: ?vk.Sampler` - VkSampler handle.

`width: u32` - Width of the texture.

`height: u32` - Height of the texture.

### Public Functions

**`deinit(self: *Texture2D) void`**

Deinitializes the texture and frees all resources.

**`init_buffer(interface: *VkInterface, vk_allocator: *VulkanAllocator, comptime T: type, buffer: []T, width: u32, height: u32, format: vk.Format, usage: vk_memory.VulkanAllocatorUsage) !Texture2D`**

Initializes and allocates a texture using a provided slice of image pixel data (`buffer`). Requires a `format` and `usage` value to allocate correctly.

**`init(interface: *VkInterface, vk_allocator: *VulkanAllocator, path: [*:0]const u8, usage: vk_memory.VulkanAllocatorUsage) !Texture2D`**

Initializes and allocates a texture by loading a .bmp image at `path` directly. Automatically determines the image's format.

## TextureAtlas2D (`struct`)

A TextureAltas is a "dynamic" texture sheet, which can store and discard smaller textures.

### Fields

`interface: *VkInterface` - Vulkan interface.

`vk_allocator: *VulkanAllocator` - Vulkan allocator.

`image: VulkanAllocator.VulkanImageAllocation` - Image allocation.

`image_view: vk.ImageView` - VkImageView.

`sampler: vk.Sampler` - VkSampler.

`width: vk.DeviceSize` - Width of the atlas.

`height: vk.DeviceSize` - Height of the atlas.

`map_texels_used: [][]bool` - Contains a \[`width`x`height`\] map of all image pixels currently being occupied by a subtexture.

### Structures

#### TextureSuballocation (`struct`)

Contains the boundaries of a subtexture within the atlas.

##### Fields

`pos_x: f32` - X-offset of the subtexture.

`pos_y: f32` - Y-offset of the subtexture.

`scl_x: f32` - Width of the subtexture.

`scl_y: f32` - Height of the subtexture.

### Public Functions

**`add_texture(self: *TextureAtlas2D, texture: *Texture2D) !TextureSuballocation`**

Adds a texture to this atlas, returning its suballocation.

**`deinit(self: *TextureAtlas2D) void`**

Deinitializes the texture atlas, freeing all resources.

**`init(interface: *VkInterface, vk_allocator: *VulkanAllocator, width: u32, height: u32) !TextureAtlas2D`**

Initializes and allocates a texture atlas of size `width` x `height`.

**`remove_texture(self: *VkInterface, suballocation: TextureSuballocation) !void`**

Removes a suballocated texture from the atlas.

## Errors

### `ImageLoadError`

#### Values

**InvalidFileType**: File type isn't supported.

**MissingFormat**: Image file is missing important format data.

**InvalidData**: Image file data is invalid.

### `ImageTransitionError`

#### Values

**UnsupportedOldLayout**: Old layout to transition from isn't supported.

**UnsupportedNewLayout**: New layout to transition to isn't supported.

### `TextureAtlasError`

#### Values

**OutOfRange**: The provided suballocation is out of range of the texture atlas when trying to remove a texture.

**OutOfSpace**: There isn't enough space on the atlas to suballocate the texture.

## Public Functions

**`load_bmp_image(allocator: *const std.mem.Allocator, path: [*:0]const u8, image_width: *u32, image_height: *u32) ![]u8`**

Load a .bmp image, and return its pixel data as a slice. Also sets the values at `image_width` and `image_height`.

**`create_image_view_2d(interface: *VkInterface, image: vk.Image, format: vk.Format) !vk.ImageView`**

Creates a `VkImageView` using a 2D image.

**`create_sampler_2d_linear_repeat_no_mipmap(interface: *VkInterface) !vk.Sampler`**

Creates a `VkSampler` that samples a texture using bilinear interpolation, repeat wrapping, and no mipmapping.

**`transition_vulkan_image_layout(interface: *VkInterface, image: vk.Image, image_format: vk.Format, old_layout: vk.ImageLayout, new_layout: vk.ImageLayout, command_pool: vk.CommandPool, transfer_queue: vk.Queue) !void`**

Handles a small set of image transitions used by the Vulkan allocator.