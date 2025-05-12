---
layout: page
title: "Learning Rust: Creating a Multi-threaded 4D Raymarcher"
description: "A project with the purpose of learning Rust by adapting some of the technologies from 4D Games"
github:
    public: true
    repo-link: https://github.com/JoeSubbi/Rust-4D-Raymarcher
---

<p style="text-align: justify">
Recently I have been wanting to have another go at Rust, having only touched it briefly during university. I figured a great place to start would be with some familiar technology; seeing how I could implement it in Rust. This was a great project for learning many of the core components of Rust as a language.
</p>

### Table of Contents:

- [Where To Start](#where-to-start)
  - [Exploring Traits with Vectors](#exploring-traits-with-vectors)
  - [Maths and Unit Tests: Bivectors and Rotors](#maths-and-unit-tests-bivectors-and-rotors)
- [Displaying Animated Renders to a Window](#displaying-animated-renders-to-a-window)
- [Speeding Things Up](#speeding-things-up)
  - [Async and Futures](#async-and-futures)
  - [Multithreading](#multithreading)

## Where To Start

<p style="text-align: justify">
I stumbled across a version of Ray Tracing In One Weekend, but for Rust: <a href="https://the-ray-tracing-road-to-rust.vercel.app/">The Ray Tracing Road To Rust</a>. Ray tracing isn't dissimilar to raymarching; both involve shooting rays from a camera to colour a pixel. That gave me the idea to adapt some stuff I had already written to see how concepts I am familiar with can translate to the Rust programming language. 
</p>

<img src="{{ '/assets/rust-raymarching/first_render.png' | absolute_url }} " alt="first render" style="max-width: 80%;margin-left: auto;margin-right: auto;"/>

### Exploring Traits with Vectors

<p style="text-align: justify">
One of the first steps in creating a ray tracer or raymarcher was to create a vector type. As I knew I wanted to make a 4D raymarcher I knew I would need vector types for both 3 and 4 dimensions. As a result I created structs <code>Float2</code>, <code>Float3</code>, and <code>Float4</code> as you might find in game engines or shader languages.
</p>

<p style="text-align: justify">
I almost exclusively work with object oriented programming, where typically you would use an interface or inheritance to abstract common functionality. This is useful when you want a function to take a generic parameter. For example the signed distance function for a sphere, which is the same in any dimension:
</p>

```rs
pub fn sdf_sphere<T: Vector>(p: T, centre: T, radius: f32) -> f32
{
    return (p-centre).length() - radius;
}
```

<p style="text-align: justify">
Rust is not an object oriented programming language; instead it uses Traits to abstract functionality between types.
</p>

<p style="text-align: justify">
Implementing Float3 first, I already had to make use of traits to derive operators like Add and Multiply, but vectors also have other operations such as the dot product, cross product, wedge product, and also the ability to normalize a vector. I created my own trait <code>Magnitude</code> which includes functions for normalizing a vector and getting its length as I knew this would also have to be derived by Rotors later on. I then created a new <code>Vector</code> trait which included the dot product, and also combined several other traits that the vector types were to implement, including <code>Magnitude</code>, <code>Add</code>, <code>Mul</code>, <code>MulAssign</code> etc. 
</p>

*Thoughts on Traits:*

<p style="text-align: justify">
Traits seem like a pretty tidy way to handle abstracted functionality. Any object can implement any number of traits; and traits can also combine with other traits to specify a wide range of included functionality. This can really tidy up some issues with many object oriented languages where you want to be able to inherit from multiple classes, but are restricted to one. Interfaces in OOPLs do get you most of the way there, but they sometimes are may not be able to wrap up all the functionality you would like in order to describe the object you are working with.
</p>

<p style="text-align: justify">
However, not allowing structs to inherit from other structs did become a limitation later on. When I started creating different cameras for 3 and 4 dimensions, I was very frustrated. The 2 cameras shared all the same variables (except vector and rotor types being different for 3D and 4D) and very similar or even identical functions. Traits didn't make a lot of sense here, as they don't declare variables. Being unable to create unique structs that inherit from the same parent led to duplicate code for near identical structs. I did consider working around this with a single generic struct and leave the types to the caller, but that felt quite untidy to me, and I much preferred distinct structs for simpler construction.
</p>

### Maths and Unit Tests: Bivectors and Rotors

<p style="text-align: justify">
Following the vector types I created Bivectors and Rotors for 3D and 4D rotation. I already had some Unit Tests from 4D games, and I figured I would use them as an opportunity to test out Rusts built in Unit Test functionality.
</p>

<p style="text-align: justify">
Having Unit Tests built into Rust is a fantastic feature, and they even came in use as I had made a mistake in translating some of the rotor code from C# to Rust, and the tests caught this. It was super simple to set up, and leaves very little excuse not to test your code.
</p>

<img src="{{ '/assets/rust-raymarching/unit_tests.png' | absolute_url }} " alt="unit tests" style="max-width: 80%;margin-left: auto;margin-right: auto;"/>

## Displaying Animated Renders to a Window

<p style="text-align: justify">
Up to this point, the project was just rendering an image to a file, as was shown in The Ray Tracing Road To Rust tutorial. My next goal was have this open in its own window and continuously render in order to display a rotating shape.
</p>

<p style="text-align: justify">
Whilst looking up how to do this I found a guide: <a href="https://sunjay.dev/learn-game-dev/intro.html">Game Development in Rust with SDL2</a>. This could come in handy if I ever wanted to take this project further, but it's main use for me was just an introduction to the SDL2 for Rust crate, and helping me quickly get a window up and running.
</p>

<p style="text-align: justify">
I spent a while trying to create a <code>Surface</code> and set the pixels directly to this type of texture, but I couldn't find many resources on for Rust, and whilst there is plenty of C++ examples, I ultimately decided on just setting the colour of pixels directly on the canvas rather than trying to handle <code>unsafe</code> code from the C++ examples.
</p>

<img src="{{ '/assets/rust-raymarching/render.png' | absolute_url }} " alt="window render" style="max-width: 80%;margin-left: auto;margin-right: auto;"/>

## Speeding Things Up

<p style="text-align: justify">
Now I had a continuously rendering window, but the performance was pretty awful. This is totally to be expected as I am not utilising any shaders, but it does provide an opportunity to learn more about Rust. I wanted to explore two types of concurrency: Async and Futures, and Multithreading.
</p>

### Async and Futures

<p style="text-align: justify">
Rusts version of asynchronous functions and futures is pretty similar to C#'s async tasks, which I had some experience with. Overall it was pretty straightforward to set up, and it helped me see how one might structure asynchronous code in Rust. I didn't see much of a performance benefit though, presumably because I made each pixel its own async call which probably just rendered more-or-less synchronously. This code is still visible in the <a href="
https://github.com/JoeSubbi/Rust-4D-Raymarcher/tree/async_rendering">async_rendering</a> branch on Github. Rather than trying to improve on this though I decided to branch again and take a look at multithreading.
</p>

### Multithreading

<p style="text-align: justify">
At first I had a bit of a struggle spawning threads. Rust was telling me the lifetimes of variables used by the threads may not necessarily live longer than the thread, as the lifetime of the thread is unknown. After some research I found <code>thread::scope</code> which is a neat piece of tech for telling the rust compiler how long the thread will live among other things.
</p>

<p style="text-align: justify">
Unfortunately after successfully spawning a new thread for every pixel, every frame, my performance absolutely tanked. spawning and joining threads of course adds additional overhead and as a result I was essentially finding a more expensive way to render each pixel.
</p>

<p style="text-align: justify">
I tried using <code>thread::available_parallelism()</code> which dropped the number of threads from several thousand to 16, but unsurprisingly this performance wasn't much better than using a single thread given the amount of pixels in the image.
</p>

<p style="text-align: justify">
In the end I just made each row of the image its own thread, and this significantly improved performance, hitting nearly 30fps on my AMD Ryzen 7 CPU.
</p>

<p style="text-align: justify">
Overall I am happy with where this project ended, and I learned a lot about many core features of Rust!
</p>

<img src="{{ '/assets/rust-raymarching/final_render.png' | absolute_url }} " alt="final render" style="max-width: 80%;margin-left: auto;margin-right: auto;"/>