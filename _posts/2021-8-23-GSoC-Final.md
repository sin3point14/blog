---
layout: post
title: GSoC-Final Final Blog
date: 2021-7-30
---

This post is to summarize what all I managed to complete in my GSoC 2021 work period for my project: *glTF Model Loading*  

My projects contribtions can be broken down to 2 repositories: [gulkan](https://gitlab.freedesktop.org/xrdesktop/gulkan/) and [xrdesktop](https://gitlab.freedesktop.org/xrdesktop/xrdesktop/)

### gulkan

Gulkan is a a GLib library for Vulkan abstraction. It provides classes for handling Vulkan instances, devices, shaders and initialize textures GDK Pixbufs, Cairo surfaces and DMA buffers.

Over the course of first 5 weeks I added a normal mapped Cube example with a brick wall texture to Gulkan. I had experience with implementing normal mapping in DirectX 11 but it was my first time working with Vulkan and C hence it proved to by quite a challenge to get this done. A particularly notorious part of the project was to convert the cube data in the cube example, which is based on [VKCube](https://github.com/krh/vkcube), from `VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP` to `VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST` and manually compute all the tangents. This conversion work in itself took over 2 weeks to get in a somewhat presentable state. I also got to learn graphics debugging with Renderdoc and using the Vulkan API layers.

![Normal map Cube](https://media.discordapp.net/attachments/818922990715797515/863045554740789248/unknown.png)

### xrdesktop

xrdesktop is a library for XR interaction with classical desktop compositors.

In the remining course of 5 weeks I got MASSIVE progress here. Building upon my experience of working on gulkan I was able to load glTF models based in the `GthreeLoader` class in [Gthree](https://github.com/alexlarsson/gthree/). Gthree is a port of three.js to GObject and Gtk3. Since xrdesktop is also based on GObject the class structure could be used as it is. However Gthree is a complete rendering engine that provides a LOT of features and abstractions which are currently not supported xrdesktop, which has very minimal 3D rendering features. I ported a lot of classes, adapted Gthree's internal structs to be compatible with xrdesktop, removed features that were unnecessary. The resulting code follows a clean OOP paradigm which makes it pretty trivial to add new features as the base architecture is up.  

The features provided by my code are-  

- loading position, normals, texcoords for meshes
- multiple mesh loading
- scene tree
- primitive material with diffuse textures
- phong shading

![Suzanne in VR](https://media.discordapp.net/attachments/818922990715797515/876317283046281256/unknown.png)

### How to load a model?

You would need to build these custom branches:

- xrdesktop: [sin3point14:add-model2](https://gitlab.freedesktop.org/sin3point14/xrdesktop/-/tree/add-model2)
- gulkan: [sin3point14:vertex-buffers](https://gitlab.freedesktop.org/sin3point14/gulkan/-/tree/vertex-buffers)
- gxr: [lubosz:vertex-buffers](https://gitlab.freedesktop.org/lubosz/gxr/-/tree/vertex-buffers)

Set `MODEL_PATH` env var to the path of your gltf file(try [this Cube](https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0/Cube/glTF) or [this Suzanne](https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0/Suzanne/glTF)) and run any example in the `xrdesktop` repo after hooking up your VR HMD. Advice: run `environment.c` in case want no other widget other than the glTF model.

### Future Aspects

The 3D rendering can use a lot lot more features like, custom lights(possibly loaded from the glTF file itself), normal mapping, PBR materials, animations etc. The list won't end :p. An immediate applicaiton of my work would be to load HMD Controller models. A couple of hacks have been used here and there, like gulkan currently doesn't provide a proper way to draw indexed with synamic vertex budffers, so that has been done manually. Also the G3kMesh class can be further abstracted since all of its data members are public.

### Merge requests

- Merged: [https://gitlab.freedesktop.org/xrdesktop/gulkan/-/merge_requests/14](https://gitlab.freedesktop.org/xrdesktop/gulkan/-/merge_requests/14)
- Open: [https://gitlab.freedesktop.org/xrdesktop/xrdesktop/-/merge_requests/34](https://gitlab.freedesktop.org/xrdesktop/xrdesktop/-/merge_requests/34)

### Ending Note

I had a lot of fun programming on xrdesktop over this summer and had a blast with my mentors. Thanks!!!
