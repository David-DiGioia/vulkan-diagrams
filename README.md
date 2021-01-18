# vulkan-diagrams

## Contents

- [Introduction](#introduction)
- [Fence synchronization](#fence-synchronization)
- [Vertex buffer creation](#vertex-buffer-creation)
- [Render pass and swapchain](#render-pass-and-swapchain)
- [Descriptor sets](#descriptor-sets)
- [Pipeline barriers](#pipeline-barriers)
- [Vertex input and multiple bindings](#vertex-input-and-multiple-bindings)

## Introduction

Vulkan Diagrams is a collection of diagrams which are designed to serve as a quick reference for various topics in Vulkan. The diagrams show the Vulkan objects needed to accomplish common tasks (e.g. creating a vertex buffer) and the relationships between these objects.

For clarity, some members of Vulkan objects are omitted and some names are slightly simplified (e.g. changing VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL to SHADER_READ_ONLY_OPTIMAL).

## Fence synchronization

In this diagram, time progresses from top to bottom. Their names and setup are taken from [the rendering and presentation chapter of Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Drawing/Rendering_and_presentation).

Note: `vkAcquireNextImageKHR` won't signal the semaphore/fence until the image is ready, and the image won't be ready until enough previously acquired images are released with `vkQueuePresentKHR`. `vkAcquireNextImageKHR` will return the code `VK_NOT_READY` to indicate that the semaphore/fence hasn't been signaled immediately, but it will signal later once an image is acquired.

If you have vsync enabled, `vkQueuePresentKHR` is the function that will block until the next vsync cycle.

![fence_synchronization](fence_synchronization.png?raw=true "fence_synchronization")

## Vertex buffer creation

NVIDIA has a [good article about memory management](https://developer.nvidia.com/vulkan-memory-management) which I recommend reading. The takeaway is that it's usually preferable to make big memory allocations with big buffers and to sub-allocate resources from those buffers. It includes the following descriptions of memory objects:

<blockquote>
<b>Heap</b> - Depending on the hardware and platform, the device will expose a fixed number of heaps, from which you can allocate certain amount of memory in total. Discrete GPUs with dedicated memory will be different to mobile or integrated solutions that share memory with the CPU. Heaps support different memory types which must be queried from the device.
  
<b>Memory type</b> - When creating a resource such as a buffer, Vulkan will provide information about which memory types are compatible with the resource. Depending on additional usage flags, the developer must pick the right type, and based on the type, the appropriate heap.

<b>Memory property flags</b> - These flags encode caching behavior and whether we can map the memory to the host (CPU), or if the GPU has fast access to the memory.

<b>Memory</b> - This object represents an allocation from a certain heap with a user-defined size.

<b>Resource (Buffer/Image)</b> - After querying for the memory requirements and picking a compatible allocation, the memory is associated with the resource at a certain offset. This offset must fulfill the provided alignment requirements. After this we can start using our resource for actual work.

<b>Sub-Resource (Offsets/View)</b> - It is not required to use a resource only in its full extent, just like in OpenGL we can bind ranges (e.g. varying the starting offset of a vertex-buffer) or make use of views (e.g. individual slice and mipmap of a texture array).
</blockquote>

Alternatively, [Vulkan Memory Allocator (VMA)](https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator) is a good library that handles memory allocations for you.

![vertex_buffer](vertex_buffer.png?raw=true "vertex_buffer")

## Render pass and swapchain

In this diagram, a single renderpass is used for each command buffer, and each renderpass has multiple subpasses.

You should use multiple subpasses instead of multiple render passes whenever possible. If a pass only needs to read from the one corresponding fragment in a previous pass, you can use a previous subpass as an input attachment and no additional render passes are needed. [Here is an example of how to do that](https://www.saschawillems.de/blog/2018/07/19/vulkan-input-attachments-and-sub-passes/). If you need random access to a previous pass (to implement a guassian blur, for example) then it would be appropriate to use multiple render passes.

In the diagram we see that one of the attachments in the frame buffer has an image which is owned by the swapchain, but this is not mandatory. For example, you could render to a texture by creating your own `VkImage` with the usage flags `VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT | VK_IMAGE_USAGE_SAMPLED_BIT` so it can be written into as a color attachment and then sampled from in a shader.

![render_pass_swapchain](render_pass_swapchain.png?raw=true "render_pass_swapchain")

## Descriptor sets

The best mental model description I've come across for some of the objects relating to descriptor sets are from a [comment in the Vulkan subreddit](https://www.reddit.com/r/vulkan/comments/b4uj52/visual_explanation_of_descriptor_sets_i_made_a/eja4bbj/):

<blockquote>
<b>DescriptorPool</b> - A big heap of available UBOs, textures, storage buffers, etc that can be used when instantiating DescriptorSets. This allows you to allocate a big heap of types ahead of time so that later on you don't have to ask the gpu to do expensive allocations.

<b>DescriptorSetLayout</b> - Defines the structure of a descriptor set, a template of sorts. Think of a `class` or `struct` in C or C++, it says "I am made out of, 3 UBOs, a texture sampler, etc". It's analogous to going
```
struct MyDesc {
  Buffer MyBuffer[3];
  Texture MyTex;
}

struct MyOtherDesc {
  Buffer MyBuffer;
}
```

<b>DescriptorSet</b> - An actual instance of a descriptor, as defined by a DescriptorSetLayout. Using the class/struct analogy, it's like going `MyDesc DescInstance();`

<b>PipelineLayout</b> - If you treat your entire shader as if it was just one big `void shader(arguments)` function then a PipelineLayout is like describing all the "arguments" passed into your shader such as `void shader(MyDesc desc, MyOtherDesc otherDesc)`. This generally maps up to statements like `layout(std140,set=0, binding = 0) uniform UBufferInfo{Blah MyBlah;}` and `layout(set=0, binding = 2, rgba32f) uniform image2D MyImage;` in your shader code.

<b>vkCmdBindDescriptorSet</b> - This is the mechanism to actually pass a DescriptorSet into a shader(aka pipeline). So basically passing the "arguments" like `shader(DescInstance,OtherDescInstance)`.
</blockquote>

Note that `vkUpdateDescriptorSets(...)` doesn't copy a buffer into the descriptor set, but rather gives the descriptor set a pointer to the buffer described by `VkDescriptorBufferInfo`. So then `vkUpdateDescriptorSets(...)` doesn't need to be called more than once for a descriptor set since modifying the buffer that a descriptor set points to will update what the descriptor set sees.

![descriptor_sets](descriptor_sets.png?raw=true "descriptor_sets")

## Pipeline barriers

This diagram shows the general use of pipeline barriers and how they create execution dependencies and memory dependencies. The specific example in the diagram shows the pipeline barrier which transfers the image's layout from `VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL` to `VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL`. This is performed after copying the image data from a buffer into the image, to prepare the image to be read from shaders. This example was taken [from the texture mapping chapter of Vulkan Tutorial](https://vulkan-tutorial.com/Texture_mapping/Images).

The set names and quotes are taken directly from [the spec](https://www.khronos.org/registry/vulkan/specs/1.1-extensions/html/vkspec.html#synchronization-dependencies).

(ctrl+f search for PDF version of spec: <b>Execution and Memory Dependencies</b>)

![pipeline_barriers](barrier.png?raw=true "pipeline_barriers")

## Vertex input and multiple bindings

`VkPipelineVertexInputStateCreateInfo` allows us to specify how our vertices are stored in memory. It is composed of an array of `VkVertexInputAttributeDescription`s and an array of `VkVertexInputBindingDescription`s.

As the name implies, we will have one binding description for each binding. In this example, we see that binding 0 has `VK_VERTEX_INPUT_RATE_VERTEX` as its input rate, which means it increments to the next set of data `stride` apart for every _vertex_. Binding 1, on the other hand, has `VK_VERTEX_INPUT_RATE_INSTANCE` as its input rate, so we increment to the next set of data `stride` apart only for every _instance_. We specify the number of instances and vertices we draw in `vkCmdDraw`.

We have one vertex attribute for each member of the struct associated with that binding. For example, binding 0 has 3 vertex attributes since the vertex buffer bound to binding 0 is a buffer of `Vertex` structs which has members `position`, `normal` and `texCoord`. Binding 1 has only 2 vertex attributes since `InstanceData` has only 2 members. The format of each vertex attribute is determined by the size and type of that attribute, so some common choices include:

```
float:	VK_FORMAT_R32_SFLOAT
vec2:	VK_FORMAT_R32G32_SFLOAT
vec3:	VK_FORMAT_R32G32B32_SFLOAT
vec4:	VK_FORMAT_R32G32B32A32_SFLOAT
ivec2:	VK_FORMAT_R32G32_SINT
uvec4:	VK_FORMAT_R32G32B32A32_UINT
double:	VK_FORMAT_R64_SFLOAT
etc.
```

In this example we make one call to `vkCmdBindVertexBuffer` and pass in arrays to bind both buffers at the same time, but note it would have been possible to make two separate calls so long as in one call we specify `firstBinding = 0` and in the other `firstBinding = 1`.

Lastly, notice that the vertex shader is completely blind to which binding each of its input variables are coming from. The vertex shader only specifies the locations, then the bound buffer the data comes from for each variable is determined by the corresponding `VkVertexInputAttributeDescription`.

![vertex_input](vertex_input.png?raw=true "vertex_input")
