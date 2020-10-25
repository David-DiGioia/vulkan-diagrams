# vulkan-diagrams
Vulkan is a complicated API to learn, so having some good diagrams can go a long way :)

Please let me know if you spot any mistakes and I will be sure to correct them.
Also let me know if you have suggestions for diagrams and I will consider adding them.

<h2>Fence synchronization</h2>

In this diagram, time progresses from top to bottom. There names and setup are taken from [this chapter of Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Drawing/Rendering_and_presentation).

![fence_synchronization](fence_synchronization.png?raw=true "fence_synchronization")

<h2>Vertex buffer creation</h2>

![vertex_buffer](vertex_buffer.png?raw=true "vertex_buffer")

<h2>Render pass and swapchain</h2>

![render_pass_swapchain](render_pass_swapchain.png?raw=true "render_pass_swapchain")

<h2>Descriptor sets</h2>

![descriptor_sets](descriptor_sets.png?raw=true "descriptor_sets")

<h2>Pipeline barriers</h2>

This diagram shows the general use of pipeline barriers and how they create execution dependencies and memory dependencies. The specific example in the diagram shows the pipeline barrier which transfers the image's layout from VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL to VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL. This performed after copying the image data from a buffer into the image, to prepare the image to be read from shaders. This example was taken [from this chapter of Vulkan Tutorial](https://vulkan-tutorial.com/Texture_mapping/Images).

The set names and quotes are taken directly from [6.1.of the spec](https://www.khronos.org/registry/vulkan/specs/1.1-extensions/html/vkspec.html#synchronization-dependencies).

(ctrl+f search for PDF version of spec: <b>6.1. Execution and Memory Dependencies</b>)

![pipeline_barriers](barrier.png?raw=true "pipeline_barriers")
