# `vk_memory`

Contains the `VulkanAllocator` structure, as well as various utilities to handle Vulkan memory allocations.

## VulkanAllocatorUsage (`enum`)

Specifies how an allocation made with the `VulkanAllocator` will be used. Helps determine where and how the data is placed and formatted in memory.

### Values

**VertexBuffer**: Buffer allocation used for vertex buffers.

**IndexBuffer**: Buffer allocation used for index buffers.

**UniformBuffer**: Buffer allocation used for uniform buffers.

**Texture**: Image allocation used for textures.

**Subtexture**: Image allocation to be used for a subtexture.

**GenericAttachment**: Image allocation used for a generic render target.

**DepthAttachment**: Image allocation used for a depth render target.

**CPUTransferDst**: Buffer used to act as a CPU-visible place to transfer data from the GPU.

## VulkanAllocator

Allocates Vulkan buffer and image objects on GPU memory pages.

### Fields

`interface: *VkInterface` - Vulkan interface handle.

`cpu_allocator: *const std.mem.Allocator` - `std.mem.Allocator` object used for allocating `ArrayList`s on the CPU.

`staging_command_pool: *const vk.CommandPool` - Command pool used for allocating command buffers used for staging (or memory transfer) operations.

`staging_queue: *const vk.Queue` - Queue used for executing staging (or memory transfer) commands.

`memory_pages: std.ArrayList(VulkanMemoryPage)` - List of memory pages that the allocator has allocated.

`options: VulkanAllocatorOptions` - Options for the allocator. Set when initialized.

`staging_buffer: vk.Buffer = undefined` - Staging buffer for the allocator.

`staging_buffer_memory: vk.DeviceMemory = undefined` - Staging buffer memory.

### Structures

#### MemorySpace (`struct`)

Helper struct which stores information about a free space in a memory page (just its offset and length).

##### Fields

`offset: vk.DeviceSize` - Offset of the memory space in the page, in bytes.

`length: vk.DeviceSize` - Size of the memory space, in bytes.

#### VulkanMemoryPage (`struct`)

Structure containing all the information for a page of Vulkan memory, including the memory handle itself.

##### Fields

`memory: vk.DeviceMemory = undefined` - Handle to the page's device memory.

`freelist: std.ArrayList(MemorySpace)` - List of all available free spaces in the memory page.

`properties: vk.MemoryPropertyFlags` - Memory usage properties for the memory page.

`memory_type_index: u32` - Index for the memory type in the physical device that this page uses.

#### VulkanAllocatorOptions (`struct`)

Structure containing options used when intializing the VulkanAllocator.

##### Fields

`transfer_command_pool: *const vk.CommandPool` - Command pool used to allocate buffers for memory transfer commands.

`transfer_queue: *const vk.Queue` - Queue that memory transfer commands will be placed into.

`page_size: vk.DeviceSize` - Size of each allocated memory page.

`staging_size: vk.DeviceSize` - Size of the staging buffer.

#### MemoryLocation (`struct`)

Simple data type that points to a location in the allocator's memory. May contain the index of a freelist MemorySpace.

##### Fields

`page: usize` - Index of the memory page in the `VulkanAllocator`'s `memory_pages` list.

`offset: vk.DeviceSize` - Offset of the location within the memory page.

`memory_space_index: ?usize = null` - If the location is a free space, this field will be the index of said free space in `VulkanMemoryPage.freelist.items`.

#### VulkanBufferAllocation (`struct`)

Object representing a finished/successful buffer allocation. Members of this struct can be accessed, but should NOT be modified outside of this allocator.

##### Fields

`buffer: vk.Buffer` - `VkBuffer` handle.

`page: usize` - Page index, used during deallocation.

`offset: vk.DeviceSize` - Memory offset within the page, used during deallocation.

`size: vk.DeviceSize` - Size of the buffer.

#### VulkanImageAllocation (`struct`)

Object representing a finished/successful image allocation. Members of this struct can be accessed, but should NOT be modified outside of this allocator.

##### Fields

`image: vk.Image` - `VkImage` handle.

`page: usize` - Page index, using during deallocation.

`offset: vk.DeviceSize` - Memory offset within the page, used during deallocation.

`size: vk.DeviceSize` - Size of the image.

### Public Functions

**`alloc_buffer(self: *VulkanAllocator, comptime T: type, data: []T, share_mode: vk.SharingMode, usage: VulkanAllocatorUsage) !?VulkanBufferAllocation`**

Allocates a `VkBuffer` object of a size, share mode, and allocator usage. Returns an allocation object used to keep track of where the buffer is in GPU memory. **DO NOT** edit the contents of the allocation object.

**`alloc_buffer_empty(self: *VulkanAllocator, size: vk.DeviceSize, share_mode: vk.SharingMode, usage: VulkanAllocatorUsage) !?VulkanBufferAllocation`**

Allocates a `VkBuffer` with uninitialized memory. Returns an allocation object used to keep track of where the buffer is in GPU memory. **DO NOT** edit the contents of the allocation object.

**`alloc_image(self: *VulkanAllocator, image_info: vk.ImageCreateInfo, image_data: []u8, usage: VulkanAllocatorUsage) !VulkanImageAllocation`**

Allocates a `VkImage` with the data in `image_data`. Returns an allocation object used to keep track of where the image is in GPU memory. **DO NOT** edit the contents of the allocation object.

**`alloc_image_empty(self: *VulkanAllocator, image_info: vk.ImageCreateInfo, usage: VulkanAllocatorUsage) !VulkanImageAllocation`**

Allocates a `VkImage` with uninitialized memory. Returns an allocation object used to keep track of where the image is in GPU memory. **DO NOT** edit the contents of the allocation object.

**`alloc_image_2d(self: *VulkanAllocator, image_data: []u8, width: u32, height: u32, format: vk.Format, usage: VulkanAllocatorUsage) !VulkanImageAllocation`**

Allocates a `VkImage` with the data in `image_data`, and uses a set size and format in place of a `VkImageCreateInfo` struct. Assumes the image is 2D, using optimal tiling, and does not use mipmapping. Returns an allocation object used to keep track of where the image is in GPU memory. **DO NOT** edit the contents of the allocation object.

**`alloc_image_2d_empty(self: *VulkanAllocator, width: u32, height: u32, format: vk.Format, usage: VulkanAllocatorUsage) !VulkanImageAllocation`**

Allocates a `VkImage` with uninitialized memory, and uses a set size and format in place of a `VkImageCreateInfo` struct. Assumes the image is 2D, using optimal tiling, and does not use mipmapping. Returns an allocation object used to keep track of where the image is in GPU memory. **DO NOT** edit the contents of the allocation object.

**`debug_print_free_space(self: VulkanAllocator) void`**

Prints all available free spaces in currently used memory pages.

**`deinit(self: *VulkanAllocator) void`**

Deinitializes the allocator and frees all its allocated memory.

**`free_buffer(self: *VulkanAllocator, allocation: VulkanBufferAllocation) void`**

Frees a `VulkanBufferAllocation` from memory.

**`free_image(self: *VulkanAllocator, allocation: VulkanImageAllocation) void`**

Frees a `VulkanImageAllocation` from memory.

**`init(interface: *VkInterface, allocator: *const std.mem.Allocator, options: VulkanAllocatorOptions) !VulkanAllocator`**

Creates and returns a vulkan memory allocator.

**`map_data_to_buffer_subsection(self: *VulkanAllocator, comptime T: type, data: []T, allocation: VulkanBufferAllocation, buffer_offset: vk.DeviceSize) !void`**

For buffers that are host-visible and host-coherent (such as `VulkanAllocatorUsage.UniformBuffer`s), maps data directly to their memory. Using a non CPU-interactable memory type will result in a `VulkanMemoryError.InvalidMemoryType` being returned.

**`map_data_to_buffer(self: *VulkanAllocator, comptime T: type, data: []T, allocation: VulkanBufferAllocation) !void`**

Calls **`map_data_to_buffer_subsection`**, assumes `buffer_offset` is 0.

**`overwrite_buffer(self: *VulkanAllocator, allocation: VulkanBufferAllocation, comptime T: type, data: []T, offset: vk.DeviceSize) !void`**

Overwrites a buffer's existing memory at the `offset` with `data`.

**`pull_buffer_data(self: *VulkanAllocator, comptime T: type, buffer: VulkanAllocator.VulkanBufferAllocation) ![]T`**

Takes a CPU-visible and coherent buffer, copies its data to a slice of a given type `T`, and returns said slice.

### Private Functions

**`alloc_image_resource(self: *VulkanAllocator, image_info: vk.ImageCreateInfo) !VulkanImageAllocation`**

Handles image allocation within an existing memory page.

**`alloc_new_page(self: *VulkanAllocator, memory_type_index: u32, properties: vk.MemoryPropertyFlags) !void`**

Creates a new memory page with the memory type and properties given.

**`find_memory_space(self: VulkanAllocator, size: vk.DeviceSize, alignment: vk.DeviceSize, memory_type_index: u32, properties: vk.MemoryPropertyFlags) ?MemoryLocation`**

Browses the existing memory pages for one which matches the memory type and properties, and has a space at least as great as size. Returns null if no space in the existing memory pages was able to be found.

**`page_freelist_clear(self: *VulkanAllocator, page_index: usize, offset: vk.DeviceSize, size: vk.DeviceSize) void`**

Updates a page's freelist after a deallocation (or "free" operation).

**`page_freelist_fill(self: *VulkanAllocator, fill_location: MemoryLocation, fill_amount: usize) !void`**

Updates a page's freelist after an allocation.

**`stage_buffer(self: *VulkanAllocator, buffer: vk.Buffer, comptime T: type, data: []T) !void`**

Maps memory to the staging buffer, then transfers said memory to it's proper place in Vulkan memory.

**`setup_image_texture(self: *VulkanAllocator, image: vk.Image, extent: vk.Extent3D, format: vk.Format) !void`**

Maps memory to the staging buffer, transfers said memory to the image, and then transitions the image's layout to be read by the shader.

**`setup_image_subtexture(self: *VulkanAllocator, image: vk.Image, extent: vk.Extent3D, format: vk.Format) !void`**

Maps memory to the staging buffer, transfers said memory to the image, and then transitions the image's layout to be used as a source in a copy operation.

## Public Functions

**`find_physical_device_memory_type(interface: *VkInterface, type_filter: u32, requested_properties: vk.MemoryPropertyFlags) VulkanMemoryError!u32`**

Helper function to find the correct memory type in the selected physical device, if one exists.

**`create_buffer(interface: *VkInterface, buffer_size: vk.DeviceSize, share_mode: vk.SharingMode, usage: vk.BufferUsageFlags) !vk.Buffer`**

Helper function to create a vk.Buffer with the buffer_size, share_mode, and usage flags.

**`allocate_vulkan_memory_from_buffer(interface: *VkInterface, buffer: vk.Buffer, requested_properties: vk.MemoryPropertyFlags) !vk.DeviceMemory`**

Helper function to allocate memory directly from Vulkan, and then bind the memory to the buffer. Should never be used outside of the allocator, unless you plan on implementing your own allocator.

**`map_data_to_memory(comptime T: type, interface: *VkInterface, memory: vk.DeviceMemory, data: []T) !void`**

Maps data directly to Vulkan memory. Will only work with data that can be accessed by the CPU. Assumes the offset of memory to map to is 0.

**`copy_buffer_with_offsets(interface: *VkInterface, size: vk.DeviceSize, src_buffer: vk.Buffer, dst_buffer: vk.Buffer, command_pool: vk.CommandPool, transfer_queue: vk.Queue, src_offset: vk.DeviceSize, dst_offset: vk.DeviceSize) !void`**

Perform a copy operation to copy data from one buffer to another, with offset values (often used for copying CPU-visible data to GPU-only buffers). Command pool and queue must both support transfer operations. **NOTE:** All Vulkan queues and command buffers that support graphics operations implicitly support transfer operations as well.

**`copy_buffer(interface: *VkInterface, size: vk.DeviceSize, src_buffer: vk.Buffer, dst_buffer: vk.Buffer, command_pool: vk.CommandPool, transfer_queue: vk.Queue) !void`**

Perform a copy operation to copy data from one buffer to another (often used for copying CPU-visible data to GPU-only buffers). Command pool and queue must both support transfer operations. **NOTE:** All Vulkan queues and command buffers that support graphics operations implicitly support transfer operations as well.

**`copy_image(interface: *VkInterface, src_image: vk.Image, dst_image: vk.Image, src_layout: vk.ImageLayout, dst_layout: vk.ImageLayout, command_pool: vk.CommandPool, transfer_queue: vk.Queue, src_offset: vk.Offset3D, dst_offset: vk.Offset3D, copy_extent: vk.Extent3D, src_subresource: vk.ImageSubresourceLayers, dst_subresource: vk.ImageSubresourceLayers) !void`**

Perform a copy operation to copy data from one image to another. Command pool and queue must both support transfer operations. **NOTE:** All Vulkan queues and command buffers that support graphics operations implicitly support transfer operations as well.

**`copy_buffer_to_image(interface: *VkInterface, src_buffer: vk.Buffer, dst_image: vk.Image, image_extent: vk.Extent3D, command_pool: vk.CommandPool, transfer_queue: vk.Queue) !void`**

Perform a copy operation to copy data from a buffer to an image. Command pool and queue must both support transfer operations. **NOTE:** All Vulkan queues and command buffers that support graphics operations implicitly support transfer operations as well.

**`copy_image_to_buffer(interface: *VkInterface, src_image: vk.Image, dst_buffer: vk.Buffer, image_offset: vk.Offset3D, image_extent: vk.Extent3D, command_pool: vk.CommandPool, transfer_queue: vk.Queue) !void`**

Perform a copy operation to copy data from an image to a buffer. Command pool and queue must both support transfer operations. NOTE: All Vulkan queues and command buffers that support graphics operations implicitly support transfer operations as well.

**`get_allocator_buffer_usage_flags(allocator_usage: VulkanAllocatorUsage) VulkanAllocatorError!vk.BufferUsageFlags`**

Based on the `allocator_usage` parameter, return the appropriate `VkBufferUsageFlags` or `VulkanAllocatorError.InvalidAllocatorUsage` if an image allocator usage value is provided.

**`get_allocator_image_usage_flags(allocator_usage: VulkanAllocatorUsage) VulkanAllocatorError!vk.ImageUsageFlags`**

Based on the `allocator_usage` parameter, return the appropriate `VkImageUsageFlags` or `VulkanAllocatorError.InvalidAllocatorUsage` if a buffer allocator usage value is provided.

**`get_allocator_usage_memory_properties(allocator_usage: VulkanAllocatorUsage) vk.MemoryPropertyFlags`**

Based on the `allocator_usage` parameter, return the appropriate `VkMemoryPropertyFlags` which helps determines where this allocation ought to be made in memory.