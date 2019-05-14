---
layout: post
title: "Struggling with Rust's References"
tags: rust
---

I've been working on my [raytracer](https://github.com/P1n3appl3/ray/) on and off for a few months now. Overall it's been fun and educational, but there's one problem I just cant seem to solve. I've essentially given up on it for now, but I want to write it down so I don't forget. Maybe in a few months or years when I become more familiar with rust I'll figure out a proper way to handle it, but for now, here's the issue:

So in my raytracer I can render 3d objects. If you're familiar with 3d graphics you might be aware that those 3d objects are actually constructed from many (thousands to millions) of tiny triangles. The triangles are defined something like this:

``` rust
struct Vec3 {
    x: f32,
    y: f32,
    z: f32,
}

struct Vertex {
    position: Vec3,
    normal: Vec3,               // don't worry about what
    texture_coordinates: Vec2,  // these fields mean
}

struct Triangle {
    v0: Vertex,
    v1: Vertex,
    v2: Vertex,
    material: Material,
}
```

As you can see, triangles contain 3 vertices and a material (what a material actually consists of is unimportant). This worked pretty well and I drew some pretty triangles. Then I started importing 3d models and converting them to a _lot_ of triangles. Those objects made of triangles are called "meshes" and they look like this:

``` rust
struct Mesh {
    faces: Vec<Triangle>,
}
```

![](https://upload.wikimedia.org/wikipedia/commons/f/fb/Dolphin_triangle_mesh.png)

This works fine, but is actually hugely inefficient in terms of memory usage. This is because each triangle owns all of its members even though they could usually be sharing them. Look at that dolphin above; many of the vertices are part of several triangles. I checked the ratio of triangles to vertices for a few models and it varies widely, but usually each vertex is shared between _at least_ 3 faces. In addition, usually many faces will have the same material, but in this model they have to have their own copy of it. Unfortunately a `Material` may contain a lot of data (like an image to use as a texture) and I quickly realized that this is disastrously wasteful.

Luckily rust has references which I can use to share those elements between the triangles. All I have to do is replace each member with a reference and annotate them with lifetimes. The lifetimes just express to the compiler that "all the references in this struct will point to valid things at least until this struct dies". This guarantees that the program cant try to dereference a bad pointer and SEGFAULT.

``` rust
struct Triangle<'a> {
    v0: &'a Vertex,
    v1: &'a Vertex,
    v2: &'a Vertex,
    material: &'a Material,
}
```

It may seem that I'm done, but now I have a new problem to worry about. Namely, _something_ still has to own the stuff each triangle has references to. At this point I need to get more specific about how these triangles are actually being passed around in the program. There's a `Scene` struct that contains pretty much all the information in the program like where the camera is, what the resolution should be, and all the objects to be rendered. The natural place to put all those `Material`s is in that Scene object as well, because then it's pretty easy to guarantee that they will live as long as everything else in there. The triangle vertices can go in the mesh that contains them and now I have owners for all the data.

``` rust
struct Mesh {
    faces: Vec<Triangle>,
    vertices: Vec<Vertex>,
}

struct Scene {
    camera: Camera,
    background: Texture,
    ...
    objects: Vec<Mesh>,
    materials: Vec<Material>,
}
```

At this point, having fixed the last few errors that rustc was yelling about, I was pretty much ready to declare victory. Unfortunately this code does not compile (as I was informed rather rudely, with dozens of long error messages about lifetimes and move after borrow). It turns out that while keeping all those structs in big immutable buffers is a good way to convince rustc that something is keeping track of them, actually _getting_ them into that arrangement is rather nontrivial. For example lets say that I try to construct the `Scene` like this:

``` rust
let materials: Vec<Material> = vec![define, some, materials]
let vertices: Vec<Vertex> = read_vertices_from_file("dolphin.obj");
let mesh: Mesh {
    faces: make_triangles_from_points_and_materials(&vertices, &materials),
    vertices,
}
let s = Scene {
    ...
    objects: vec![mesh],
    materials,
}
```

To understand why this is invalid you have to understand what "borrowing" is, but I think it's not too hard to grasp with this example. When I create a reference to a piece of data, rustc notes that I've borrowed it. This means that while that data is still owned by something else, I can use my reference to access it. Then when I'm done with it, rustc will realize that I've stopped using it and "drop" the borrow. One of the invariants inherent in this system is that while I'm borrowing the variable, it has to stay in the same place in memory. Otherwise the reference I had to it would be pointing to nondeterministic garbage and everything would break. Luckily rustc can make sure that never happens by making it illegal to move borrowed values. Unfortunately that's exactly what I'm doing in that code. `faces` is constructed by a function that borrows both the materials and the vertices, but then I move those two things into the mesh and scene while faces still had references to them.

This isn't actually the end of the world because it's possible to construct those structs in a very careful order such that nothing gets moved after it gets borrowed. It feels like jumping through hoops and requires a lot of mutability so I wont write it out, but I'm pretty sure it works. The reason I say "pretty sure" is that I never got to see

"solutions":
everything static
indicies
rc/arc
unsafe raw *
pin?
[r]()[e](manish)[a]()[l]() [w](whitequark)[i]()[z](jvns)[a]()[r]()[d]()[s](burntsusih)

should have read references and borrowing chapter and Pin first
