# `vk_utils`

General or miscellaneous utilites related to Vulkan operations.

## Errors

### VulkanFormatError

#### Values

**FormatNotSupported**: The vulkan format is unsupported by the function or operation.

**FormatHasNoSize**: Specified format doesn't have a specific size.

**NoValidFormatsFound**: Format finder function couldn't find a useful or valid format.

## Public Functions

**`choose_best_depth_buffer_format(vk_interface: *ash.rendering.VkInterface) VulkanFormatError!vk.Format`**

Calls **`choose_image_format`** for a list of depth buffer attachments with optimal tiling and the depth-stencil feature.

**`choose_image_format(vk_interface: *ash.rendering.VkInterface, preferred_tiling: vk.ImageTiling, preferred_features: vk.FormatFeatureFlags, formats: []const vk.Format) VulkanFormatError!vk.Format`**

Picks the most well-suited from a list of image formats based on preferred image tiling and features.

**`get_vulkan_format_size(format: vk.Format) VulkanFormatError!u32`**

Returns the size, in bytes, of a complete pixel value in the given format.