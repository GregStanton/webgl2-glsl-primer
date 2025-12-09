# WebGL2 & GLSL primer: <br /> A zero-to-hero, spaced-repetition guide

**Status:** Complete  
**Author:** Greg Stanton

These notes take the reader from _zero_ (no knowledge of WebGL2 or GLSL) to _hero_ (confidence in everything from low-level state management to 3D graphics production). Brief explanations of prerequisite concepts are provided. Following the prerequisite material, the notes introduce the fundamentals of WebGL2 and GLSL in a natural order, chunking concepts and syntax into a Q&A format, suitable for spaced-repetition practice with software like [Anki](https://apps.ankiweb.net/). Projects are integrated throughout, with solution code, to provide practice applying ideas as soon as they’re introduced. At the end, recommendations on leveraging your new skills are provided, including an annotated list of links to high-quality projects and advanced resources.

# Background
This guide focuses on the _programmable geometry pipeline_—creating form and color through code, logic, and mathematics. It covers the irreducible minimum required to build a 3D engine from scratch. While it establishes the foundation for all graphics tasks, it does not cover external asset management (like texture image loading), focusing instead on the state machine and the vertex/fragment logic essential for procedural graphics and tools like the [RMF Engine](https://github.com/GregStanton/proposal-rmf-engine) created by this primer's author. Before diving into the programmable geometry pipeline, we'll make sure we have the prerequisite concepts and skills in place.

## Prerequisite topics
Prerequisites include both programming and math.

**Programming:** 
Knowledge of HTML and JavaScript is assumed.

**Mathematics:** 
Simple, concise explanations are provided for the topics below.

* 3D primitives, including triangle strips and triangle fans
* Matrix representations of geometric transformations
* Homogeneous coordinates in projective geometry
* Transforms in the standard 3D rendering pipeline

## Prerequisite explanations

This section explains the mathematical prerequisites at the level of detail we will need, with references for anyone desiring additional detail.

### 3D primitives (drawing modes)

The image below is sufficient for understanding WebGL drawing modes (shape “kinds” in p5.js):

<img 
  width="828" 
  height="517" 
  alt="A diagram illustrating the meaning of each drawing mode available in WebGL, including the following: `gl.POINTS`, `gl.LINES`, `gl.LINE_STRIP`, `gl.LINE_LOOP`, `gl.TRIANGLES`, `gl.TRIANGLE_STRIP`, `gl.TRIANGLE_FAN`."
  src="https://github.com/user-attachments/assets/3cd05534-3f2a-412c-a10e-e29ef8e6bd52" 
/>

*Attribution:* [“*Available WebGL shapes”*](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*HQHB5lCGqlOUiysy.jpg) *appears in [A Brief Introduction to WebGL](https://medium.com/trabe/a-brief-introduction-to-webgl-5b584db3d6d6), by Martín Lamas.*

### Matrix transformations and homogeneous coordinates
Below, we explain the two concepts from higher-level math that we'll need, at the level of detail that's required. Anyone who is unfamiliar with vectors, or who desires more detailed explanations of the two concepts explained here, may consult an [overview of the relevant math concepts](https://math.hws.edu/graphicsbook/c3/s5.html) in the online book _Introduction to Computer Graphics_, by David J. Eck.

1. **Matrix representations:** Although we won't be multiplying matrices manually, it will still be helpful to have a procedural understanding of matrix multiplication, which is covered in this [ten-minute YouTube video](https://youtu.be/2spTnAiQg4M?si=Qz-DjwWKN3D9wzQR). It will also be helpful for you to know that matrix multiplication represents geometric transformations. For example, if you want to rotate the point $(1, 1)$ around the origin in the plane, you can accomplish this by multiplying a vector with components `[1, 1]` by a _rotation matrix_. It's not necessary to know how to define matrices for rotations or other transformations; it's enough to know that multiplying by a matrix accomplishes a transformation. Specific APIs for programming matrix operations are not assumed.

2. **Homogeneous coordinates:** Homogeneous coordinates are to projective geometry what Cartesian coordinates are to Euclidean geometry. This is a very cool idea that allows us to represent not just _linear_ transformations (like scaling, rotating, shearing) via matrix multiplication, but also affine transformations (like translations), and even perspective projections (which make distant objects appear smaller). This is accomplished by including one extra coordinate, allowing us to represent both _points_ (which can be translated) and _directions_ (which cannot). It works as follows. When `w` is `0`, translation vectors are annihilated (multiplied by zero), so `[x, y, z, 0]` represents the direction `[x, y, z]`. When `w` is `1`, the translation vectors are preserved (multiplied by 1), so `[x, y, z, 1]` represents the point `[x, y, z]`. When `w` is a general nonzero value, the vector `[x, y, z, w]` represents the 3D point `[x / w, y / w, z / w]`.

### Overview of coordinate systems
It’s enough to understand the significance of each source and target space, from local to screen space, and to know the sequence of transformations between them. The diagram below contains the essentials.

<img 
  width="800" 
  height="394" 
  alt="A diagram showing the standard sequence of 3D graphics transforms, from local to world space (via the model matrix), from world to view space (via the view matrix), from view space to clip space (via the projection matrix), and from clip space to screen space (via the viewport transform)."
  src="https://github.com/user-attachments/assets/197931d8-81bc-4b73-ac91-34c7111fa18a" 
/>

*Attribution:* [*coordinate_systems.png*](https://learnopengl.com/img/getting-started/coordinate_systems.png) *by [Joey de Vries](https://x.com/JoeyDeVriez) appears in [Coordinate Systems](https://learnopengl.com/Getting-started/Coordinate-Systems) and is licensed under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/)*.

The basic role of each space is indicated by the diagram. An example will clarify this further, so let's imagine we are drawing a model car in 3D.

* **Local space**: It's easiest if we can design one tire, centered at the origin. This is _local space_.
* **World space**: Then we move to the space where the car is, so we can attach the wheel in four places. This is _world space_.
* **View space**: To view our car, we move into the viewer's space, with the viewer's eye (or camera) being the origin. This is _view space_.
* **Clip space**: To show what the viewer sees, we need to clip the space, leaving only what's in front of them. This is _clip space_.
* **Screen space**: Finally, we need to display what the viewer sees on an actual 2D screen (in a viewport). This is _screen space_.

While this explanation is sufficient for our purposes, additional details may be found in [Projection and viewing](https://math.hws.edu/graphicsbook/c3/s3.html) in Eck, or [Coordinate Systems](https://learnopengl.com/Getting-started/Coordinate-Systems) in the online book _Learn OpenGL_, by [Joey de Vries](https://joeydevries.com/#home).

### Normalized device coordinates
We'll be directly dealing with normalized device coordinates early on. WebGL automatically converts clip-space coordinates to normalized-device coordinates, prior to applying the viewport transform. By making coordinates range between -1 and 1, it becomes simple to stretch them to match the dimensions of the viewport.

<img 
  width="503" 
  height="440" 
  alt = "A cubic space, with a coordinate system whose origin is at the center of the cube. A horizontal axis points right, a vertical axis points up, and a depth axis points away. Values along each axis range between -1 and 1."
  src="https://github.com/user-attachments/assets/ea261f7e-18ed-4141-81fd-3e6de54513ce"
/>

*Attribution:* *Image of NDC space (referred to as “clipspace” in original source) appears in [WebGL model view projection - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/WebGL_model_view_projection) and is licensed under [CC BY SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.en).*

## Recommended experience

The following background experience is helpful but not necessary:

* Very basic familiarity with typed languages  
* Experience creating graphics with a high-level library like [p5.js](https://p5js.org/) (especially `beginShape([kind])`/`endShape([mode])`)

## Anki tip: Learning lists
For convenience, the Q&A _cards_ in these notes will sometimes have a full list as an answer. List answers tend to be more cognitively demanding, and therefore can disrupt mental flow unnecessarily. If you find this to be the case during your own spaced-repetition practice, you can customize one of the following methods according to your own background: 

* Accompany the answer with a hint containing a single **acronym or a mnemonic phrase** (e.g. in biology, "Do kings play chess on fine green silk?" is a mnemonic for domain, kingdom, phylum, class, order, family, genus, species)
* Accompany the answer with a hint explaining how to **chunk** a longer list into only 3–4 items
* Implement lists using **cloze deletion** in software like Anki (e.g. create a sequence of cards in which all list items are revealed except for one)
* Create cards explaining how each list item connects conceptually to its neighbors (a form of **elaborative encoding**)

# Introduction
As with all sections of this primer, the current introductory section is self contained. Since the concepts covered here are foundational, sources are provided. Anyone hungry for additional context on subsequent sections will be well served by the [MDN WebGL Reference](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API), the [OpenGL ES 3.0 Specification](https://registry.khronos.org/OpenGL/specs/es/3.0/es_spec_3.0.pdf), and [The OpenGL ES® Shading Language 3.00.6](https://registry.khronos.org/OpenGL/specs/es/3.0/GLSL_ES_Specification_3.00.pdf).

## Shaders
We begin by defining the core software units that process our geometry.

<details>
<summary>
  <strong>Q:</strong> 
  What are the geometric primitives in WebGL?
</summary>
<p>
  <strong>A:</strong>
  Points, lines, and triangles. (Typically, it’s all triangles.)
</p>
<p>
  <strong>Source:</strong> 
  <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a>, <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-points-lines-triangles.html">WebGL2 Points, Lines, and Triangles</a>, <a href="https://en.wikipedia.org/wiki/Geometric_primitive">Geometric primitive - Wikipedia</a>
</p>
</details>

<details>
<summary><strong>Q:</strong> What mathematical concept unifies points (0D), lines (1D), and triangles (2D)?</summary>
<p><strong>A:</strong> Each of these is a _simplex_, the simplest n-dimensional shape in its respective dimension.</p>
<p><strong>Note:</strong> This term is not commonly used in graphics, but it's fundamental in mathematics, which is excellent at unifying seemingly disparate ideas.</p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/Simplex">Simplex - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> In computer graphics, what is a vertex?</summary>
<p><strong>A:</strong> As in geometry, a vertex is one of a set of points that defines a shape (e.g. the three corners of a triangle). A vertex may have additional attributes for rendering (drawing), such as a color.</p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/Vertex_(computer_graphics)">Vertex (computer graphics) - Wikipedia</a>, <a href="https://en.wikipedia.org/wiki/Vertex_(geometry)">Vertex (geometry) - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> In computer graphics, what is a pixel?</summary>
<p><strong>A:</strong> It’s the smallest visual element on a screen. (It’s also known as a “picture element,” analogous to a chemical element in the periodic table). It’s usually a tiny square.</p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/Pixel">Pixel - Wikipedia</a>, <a href="https://en.wikipedia.org/wiki/Chemical_element">Chemical element - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> In computer graphics, what is a fragment?</summary>
<p><strong>A:</strong> It's a potential pixel. (For example, if part of a line is behind an opaque triangle, the obscured _fragments_ of that line will be discarded, and won't end up coloring a pixel on the screen.) </p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/Fragment_(computer_graphics)">Fragment (computer graphics) - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> What two pieces of code comprise a WebGL program? Name them.</summary>
<p><strong>A:</strong> A <em>vertex shader</em> and a <em>fragment shader</em>.</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a></p>
</details>

<details>
<summary><strong>Q:</strong> What does a vertex shader do?</summary>
<p><strong>A:</strong> It computes vertex positions. (These determine where geometric primitives are rendered on the screen.)</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a></p>
</details>

<details>
<summary><strong>Q:</strong> What does a fragment shader do?</summary>
<p><strong>A:</strong> It computes the color of a fragment. (It does this for each fragment in the primitive being drawn.)</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a></p>
</details>

<details>
<summary><strong>Q:</strong> Vertex shaders and fragment shaders are code units of what type? (Are they modules, objects, functions, or something else?)</summary>
<p><strong>A:</strong> They’re functions.</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a></p>
</details>

## Software and hardware
Now we zoom out, to understand the context in which our shaders are situated.

<details>
<summary><strong>Q:</strong> Is WebGL a language or an API?</summary>
<p><strong>A:</strong> It's an API. (It allows certain graphics features to be accessed through JavaScript.) </p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/WebGL">WebGL - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> What software design pattern best describes the behavior of WebGL?</summary>
<p><strong>A:</strong> A state machine. (You set a state, and it persists until changed).</p>
</details>

<details>
<summary><strong>Q:</strong> What is a state machine?</summary>
<p><strong>A:</strong> A mathematical model of computation defined by a list of states, initial values for those states, and the inputs that trigger each transition.</p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/Finite-state_machine">Finite-state machine - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what language is used specifically to code vertex shaders and fragment shaders?</summary>
<p><strong>A:</strong> GLSL (OpenGL Shading Language)</p>
<p><strong>Note:</strong> More precisely, WebGL uses GLSL ES, which is a bit different.</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a>, <a href="https://en.wikipedia.org/wiki/OpenGL_Shading_Language">OpenGL Shading Language - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> In both WebGL and OpenGL, what does "GL" stand for?</summary>
<p><strong>A:</strong> Graphics Library</p>
<p><strong>Source:</strong> <a href="https://en.wikipedia.org/wiki/WebGL">WebGL - Wikipedia</a>, <a href="https://en.wikipedia.org/wiki/OpenGL">OpenGL - Wikipedia</a> </p>
</details>

<details>
<summary><strong>Q:</strong> Must variable and function declarations have a declared type in GLSL?</summary>
<p><strong>A:</strong> Yes.</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a>, <a href="https://en.wikipedia.org/wiki/Type_system">Type system - Wikipedia</a>, <a href="https://registry.khronos.org/OpenGL/specs/es/3.0/GLSL_ES_Specification_3.00.pdf">The OpenGL ES®
 Shading Language 3.00.6</a> (page 22)</p>
</details>

<details>
<summary><strong>Q:</strong> What general-purpose language is the syntax of GLSL patterned after?</summary>
<p><strong>A:</strong> C</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a>, <a href="https://en.wikipedia.org/wiki/OpenGL_Shading_Language#:~:text=OpenGL%20Shading%20Language%20\(GLSL\)%20is,on%20the%20C%20programming%20language">OpenGL Shading Language - Wikipedia</a></p>
</details>

<details>
<summary><strong>Q:</strong> What hardware component does WebGL run on?</summary>
<p><strong>A:</strong> The GPU (graphics processing unit)</p>
<p><strong>Source:</strong> <a href="https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html">WebGL2 Fundamentals</a>, <a href="https://en.wikipedia.org/wiki/Graphics_processing_unit">Graphics processing unit - Wikipedia</a></p>
</details>

## Pipelines
Now that we understand the most basic concepts of shaders, and the software and hardware that power them, we consider how these two engines organize their execution.

<details>
<summary><strong>Q:</strong> The graphics pipeline can be understood through which two complementary perspectives?</summary>
<p><strong>A:</strong> The coordinate pipeline (mathematical spaces) and the execution pipeline (hardware stages).</p>
</details>

<details> <summary><strong>Q:</strong> In 3D graphics, what is the standard sequence of stages in the coordinate pipeline? (List them in order.)</summary> <p><strong>A:</strong></p> <ol> <li><strong>Local space</strong> (coordinates relative to an object's origin) </li> <li><strong>World space</strong> (coordinates relative to the origin of the world in which objects are placed) </li> <li><strong>View space</strong> (coordinates relative to the camera/eye) </li> <li><strong>Clip space</strong> (coordinates accounting for the eye's field of vision) </li> <li><strong>Screen space</strong> (coordinates for the physical viewport)</li></ol></details>

<details> <summary><strong>Q:</strong> What is rasterization?</summary> <p><strong>A:</strong> The process of converting vector geometry (points, lines, triangles) into fragments.</p> </details>

<details> <summary><strong>Q:</strong> In WebGL, what are the main stages of the execution pipeline? (List them in order.)</summary> <p><strong>A:</strong></p> <ol> <li><strong>Vertex shader</strong> (positions the geometry) </li> <li><strong>Rasterization</strong> (converts vector geometry into fragments)</li> <li><strong>Fragment shader</strong> (computes the color of each fragment) </li> <li><strong>Fragment processing</strong> (determines how fragments translate into pixels)</li> </ol> </details>

## Access syntax
Now we learn how to access the world we just described.

<details>
<summary><strong>Q:</strong> In the DOM, what HTML element provides the drawing surface for WebGL?</summary>
<p><strong>A:</strong> The <code>&lt;canvas&gt;</code> element.</p>
</details>

<details>
<summary><strong>Q:</strong> How do we access the WebGL2 API?</summary>
<p><strong>A:</strong> <code>canvas.getContext('webgl2')</code></p>
</details>

<details>
<summary><strong>Q:</strong> What does <code>canvas.getContext('webgl2')</code> return?</summary>
<p><strong>A:</strong> A <code>WebGL2RenderingContext</code></p>
</details>

<details>
<summary><strong>Q:</strong> A <code>WebGL2RenderingContext</code> is often given what abbreviated name in code?</summary>
<p><strong>A:</strong> <code>gl</code></p>
</details>

<details>
<summary><strong>Q:</strong> What is the 2D version of <code>WebGL2RenderingContext</code>?</summary>
<p><strong>A:</strong> <code>CanvasRenderingContext2D</code>.</p>
</details>

# Hello canvas
It’s time to make our first project! We just need to learn a few additional concepts.

## Colors and buffers

<details>
<summary><strong>Q:</strong> What color space is used by the WebGL context?</summary>
<p><strong>A:</strong> RGBA (red, green, blue, alpha)</p>
</details>

<details>
<summary><strong>Q:</strong> What is the valid range for color values in WebGL (red, green, blue, and alpha)?</summary>
<p><strong>A:</strong> <code>0.0</code> to <code>1.0</code> (floating point numbers).</p>
</details>

<details>
<summary><strong>Q:</strong> In a WebGL RGBA color, what value of A (alpha) indicates full opacity?</summary>
<p><strong>A:</strong> <code>1.0</code></p>
</details>

<details>
<summary><strong>Q:</strong> In a WebGL context, what function sets the canvas color? Include any parameters.</summary>
<p><strong>A:</strong> <code>gl.clearColor(r, g, b, a)</code></p>
</details>

<details>
<summary><strong>Q:</strong> What does <code>gl.clearColor(r, g, b, a)</code> do?</summary>
<p><strong>A:</strong> It sets the "clear color" state but does <em>not</em> change the colors on the screen.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what basic function erases buffers and assigns them preset values? Include any parameters.</summary>
<p><strong>A:</strong> <code>gl.clear(mask)</code></p>
</details>

<details>
<summary><strong>Q:</strong> What are the standard buffers that <code>gl.clear()</code> can affect?</summary>
<p><strong>A:</strong> Color, Depth, Stencil</p>
</details>

<details>
<summary><strong>Q:</strong> When calling <code>gl.clear()</code>, what constant do we pass to clear the color buffer?</summary>
<p><strong>A:</strong> <code>gl.COLOR_BUFFER_BIT</code></p>
</details>

<details>
<summary><strong>Q:</strong> When calling <code>gl.clear()</code>, what constant do we pass to clear the depth buffer?</summary>
<p><strong>A:</strong> <code>gl.DEPTH_BUFFER_BIT</code></p>
</details>

<details>
<summary><strong>Q:</strong> Why does <code>gl.clear()</code> accept a bitmask as its parameter?</summary>
<p><strong>A:</strong> To allow multiple buffers to be cleared simultaneously via a bitwise OR operation. (This is an optimization that eliminates the need for multiple function calls.)</p>
</details>

<details>
<summary><strong>Q:</strong> To clear multiple buffers <code>buffer1</code> and <code>buffer2</code> simultaneously with <code>gl.clear()</code>, what syntax is used?</summary>
<p><strong>A:</strong> Syntax: <code>gl.clear(buffer1 | buffer2)</code>. This uses the bitwise OR operator: <code>|</code> (a single pipe character).</p>
</details>

## Project 1: Colored canvas
<img width="250" height="250" alt="yellow canvas" src="https://github.com/user-attachments/assets/0498abe0-4899-4204-9549-36e62a7644fa" />
<p>
  <strong>Problem:</strong>
  Set up an <code>index.html</code> file and a JavaScript file. Make a canvas, get the WebGL context, and use it to set the canvas to a color of your choosing.
</p>
<details>
  <summary><strong>Solution:</strong></summary>

```html  
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Yellow canvas</title>  
</head>
<body>  
    <canvas id="yellow-canvas" width="400" height="400">  
      A canvas painted yellow.  
    </canvas>
    <script src="yellow-canvas.js"></script>  
</body>  
</html>  
```

```javascript  
const canvas = document.getElementById('yellow-canvas');  
const gl = canvas.getContext('webgl2');  
const yellow = [243 / 255, 208 / 255, 62 / 255, 1];

gl.clearColor(...yellow);  
gl.clear(gl.COLOR_BUFFER_BIT);  
```
</details>

# Hello triangle
Now we'll work toward getting a triangle on the screen. This will take some effort, since we're going to make sure we understand all the low-level boilerplate.

## Starting the data bridge (getting CPU data onto the GPU)

<details>
<summary><strong>Q:</strong> In WebGL, what does VBO stand for?</summary>
<p><strong>A:</strong> Vertex Buffer Object</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what is the general purpose of a VBO?</summary>
<p><strong>A:</strong> To store data in the GPU's memory.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what sorts of data are commonly stored in a VBO?</summary>
<p><strong>A:</strong> Vertex attribute data like positions, normals, colors, and texture coordinates.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, data in a VBO is stored in what data format?</summary>
<p><strong>A:</strong> Binary</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what does VAO stand for?</summary>
<p><strong>A:</strong> Vertex Array Object</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what is the purpose of a VAO?</summary>
<p><strong>A:</strong> Recording how to read data from the VBOs. (This is typically vertex-attribute data, hence the name.)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what does the term <em>binding</em> mean?</summary>
<p><strong>A:</strong> Setting an object (e.g. a VBO) as the "active" value for a particular state in the WebGL state machine.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what is the order of operations for creating and configuring the VAO and VBO?</summary>
<p><strong>A:</strong></p>
<ol>
<li>Create and Bind the VAO.</li>
<li>Create and Bind the VBO.</li>
<li>Upload buffer data.</li>
<li>Configure attributes (tell VAO how to read the VBO).</li>
</ol>
<p><strong>Hint:</strong> Since the WebGL context is a state machine, it needs a place to define state (VAO) <em>before</em> it can organize the data values (VBO). Once these are in place, we need to upload data, then tell the VAO how to read it.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what syntax creates a Vertex Array Object?</summary>
<p><strong>A:</strong> <code>gl.createVertexArray()</code> (this function does not take parameters)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what syntax binds a VAO?</summary>
<p><strong>A:</strong> <code>gl.bindVertexArray(vao)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what syntax creates a buffer (VBO)?</summary>
<p><strong>A:</strong> <code>gl.createBuffer()</code> (this function does not take parameters)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what syntax binds a buffer?</summary>
<p><strong>A:</strong> <code>gl.bindBuffer(target, buffer)</code></p>
</details>

<details>
<summary><strong>Q:</strong> What are the two most common targets for <code>gl.bindBuffer</code>?</summary>
<p><strong>A:</strong> <code>gl.ARRAY_BUFFER</code>, <code>gl.ELEMENT_ARRAY_BUFFER</code></p>
</details>

<details>
<summary><strong>Q:</strong> What kind of data is usually bound to <code>gl.ARRAY_BUFFER</code>?</summary>
<p><strong>A:</strong> Vertex attribute data (e.g. position, normal, color, texture data)</p>
</details>

<details>
<summary><strong>Q:</strong> What kind of data is usually bound to <code>gl.ELEMENT_ARRAY_BUFFER</code>?</summary>
<p><strong>A:</strong> Index data (indicating which vertices to connect)</p>
</details>

<details>
<summary><strong>Q:</strong> Can you give a concrete example to indicate the purpose of <code>gl.ELEMENT_ARRAY_BUFFER</code>?</summary>
<p><strong>A:</strong> Suppose you want to create a quadrilateral from four vertices. This needs to be created out of triangles, and there are two ways to triangulate a quadrilateral. The element array buffer can be used to specify the triangulation, by indicating which vertices should be connected.</p>
</details>

<details>
<summary><strong>Q:</strong> What syntax sends data to the currently bound buffer?</summary>
<p><strong>A:</strong> <code>gl.bufferData(target, data, usage)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.bufferData(target, data, usage)</code>, the <code>data</code> argument usually has what type?</summary>
<p><strong>A:</strong> <code>Float32Array</code></p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.bufferData</code>, if the geometry will not change after it is uploaded, what usage constant should be passed?</summary>
<p><strong>A:</strong> <code>gl.STATIC_DRAW</code></p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, an attribute will not be used unless you explicitly turn it on, using what function? Name the function (don’t specify any parameters).</summary>
<p><strong>A:</strong> <code>gl.enableVertexAttribArray()</code></p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what function tells the VAO how to interpret the data in the currently bound VBO? Name the function (don’t specify any parameters).</summary>
<p><strong>A:</strong> <code>gl.vertexAttribPointer()</code></p>
</details>

## Coordinates expected by WebGL

<details>
<summary><strong>Q:</strong> In WebGL, vertex coordinates in a VBO are expected to be in what space?</summary>
<p><strong>A:</strong> Local space. (Also known as <em>model space</em>.)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, vertex coordinates output by a vertex shader are expected to be in what space?</summary>
<p><strong>A:</strong> Clip space.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, vertex coordinates in clip space are automatically converted to what space, after the vertex shader runs?</summary>
<p><strong>A:</strong> NDC space (normalized device coordinates).</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what operation sends clip space to NDC space?</summary>
<p><strong>A:</strong> Perspective division: The point (x, y, z, w) is divided by w.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what is the range of x, y, and z in NDC space?</summary>
<p><strong>A:</strong> -1.0 to 1.0.</p>
</details>

<details>
<summary><strong>Q:</strong> In a WebGL vertex shader, if you define an attribute as a <code>vec4</code> but only provide (x, y) data from the buffer, what values are automatically assigned to z and w?</summary>
<p><strong>A:</strong> <code>z</code> = 0.0, <code>w</code> = 1.0. (Hint: Recall that a w-coordinate of 1 makes the vertex a point, rather than a direction, i.e. it is affected by translations.)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL Normalized Device Coordinates (NDC), where is the origin (0,0)?</summary>
<p><strong>A:</strong> The center.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL Normalized Device Coordinates (NDC), in which direction do the x, y, and z axes point?</summary>
<p><strong>A:</strong> Directions: x points right, y points up, and z points away (directionally, into the screen). (Hint: The xy-plane follows mathematical conventions: x points right and y points up. However, it’s a left handed system.)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, the depth buffer typically contains values in what range?</summary>
<p><strong>A:</strong> 0.0 to 1.0 (Hint: These are non-negative, as we’d expect of “depth” values, since “depth” implies a distance measured in only one direction. The depth range may be customized, but this is rarely necessary.)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, the values in the depth buffer are determined by what transformation? Name it.</summary>
<p><strong>A:</strong> The viewport transform.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what are the source and target spaces of the viewport transform?</summary>
<p><strong>A:</strong> NDC $\to$ screen space.</p>
</details>

<details>
<summary><strong>Q:</strong> Conceptually, what does the viewport transform do in WebGL?</summary>
<p><strong>A:</strong> It stretches NDC space so that its x and y dimensions match those of the canvas, and it converts z-values from NDC space (in $[-1, 1]$) to depth values (in $[0, 1]$ by default).</p>
</details>

<details> <summary><strong>Q:</strong> What syntax explicitly configures the viewport transform?</summary> <p><strong>A:</strong> <code>gl.viewport(x, y, width, height)</code>, where the <code>x</code> and <code>y</code> parameters are the lower-left corner of the rendering area relative to the canvas, and the other parameters are the rendering area's dimensions.</p> </details>

## Project 2: Create and bind VBO and VAO, supply triangle data

This project is continued from [Project 1](#project-1-colored-canvas).

<p>
  <strong>Problem:</strong>
  Extend your `yellow-canvas.js` program so that it defines a triangle as a flat array of three $(x, y)$ vertices. Create and bind a VAO and VBO, and upload the triangle data to the VBO. Assume the triangle data will not change after it’s uploaded. (We won't render the triangle yet. We'll do that in the next project.)
</p>

<details>
<summary><strong>Solution:</strong></summary>

```javascript  
// CANVAS
const canvas = document.getElementById('yellow-canvas');
const gl = canvas.getContext('webgl2');
const yellow = [243 / 255, 208 / 255, 62 / 255, 1];

gl.clearColor(...yellow);
gl.clear(gl.COLOR_BUFFER_BIT);

// TRIANGLE
const TAU = 2 * Math.PI;
const r = 2 / 3;
const t0 = TAU / 4;
const dt = TAU / 3;

const triangleVertices = new Float32Array([
  r * Math.cos(t0), r * Math.sin(t0), 
  r * Math.cos(t0 + dt), r * Math.sin(t0 + dt), 
  r * Math.cos(t0 + 2 * dt), r * Math.sin(t0 + 2 * dt),
]);

// STATE MANAGEMENT: VAO AND VBO
const vao = gl.createVertexArray();
gl.bindVertexArray(vao);
const vbo = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
gl.bufferData(gl.ARRAY_BUFFER, triangleVertices, gl.STATIC_DRAW);
```
</details>

## GLSL ES 3.00 syntax

<details>
<summary><strong>Q:</strong> In GLSL, what is the syntax to declare a floating-point variable named <code>alpha</code> and initialize it to 1.0?</summary>
<p><strong>A:</strong> <code>float alpha = 1.0;</code></p>
<p><strong>Hint:</strong> It is very similar to C or Java. Semicolons are required.</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL, <code>vec2</code>, <code>vec3</code>, and <code>vec4</code> refer to vectors whose components have what data type?</summary>
<p><strong>A:</strong> float (floating point number)</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL, a type (such as <code>vec4</code>) is also a ____________.</summary>
<p><strong>A:</strong> constructor</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL, how can you create a <code>vec4</code> with components $(0.1, 0.2, 0.3, 0.4)$?</summary>
<p><strong>A:</strong> <code>vec4(0.1, 0.2, 0.3, 0.4)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL, if <code>pos</code> is a variable of type <code>vec2</code>, how do you create a <code>vec4</code> using <code>pos</code> for the first two components, <code>0.0</code> for <code>z</code>, and <code>1.0</code> for <code>w</code>?</summary>
<p><strong>A:</strong> <code>vec4(pos, 0.0, 1.0)</code></p>
</details>

## Shader syntax

<details>
<summary><strong>Q:</strong> In a WebGL2 shader source string, what must the very first line be?</summary>
<p><strong>A:</strong> <code>#version 300 es</code></p>
<p><strong>Hint:</strong> This refers to GLSL ES 3.00.</p>
</details>

<details>
<summary>
  <strong>Q:</strong> 
  In a WebGL2 shader, what is wrong with the following code?
  
  ```javascript
  const shader = `
  #version 300 es
  // more code here...
  `;
```

</summary>
<p><strong>A:</strong> There is a newline after the backtick, creating a blank line above the version specification. It should look like this instead:</p>

```javascript
const shader = `#version 300 es
// more code here...
`;
```

</details>

<details>
<summary><strong>Q:</strong> In GLSL, which shader stage requires an explicit precision declaration?</summary>
<p><strong>A:</strong> The fragment shader. (Hint: If vertices aren’t in the right place, things go wrong, so high precision is mandated for vertex shaders, but lower precision is allowed for fragment shaders, e.g. to avoid draining battery on older mobile devices.)</p>
</details>

<details>
<summary><strong>Q:</strong> In a GLSL shader, what’s the syntax to declare that floats should have high precision?</summary>
<p><strong>A:</strong> <code>precision highp float;</code></p>
</details>

<details>
<summary><strong>Q:</strong> In a GLSL shader, where does the line of code go that sets the precision of floats?</summary>
<p><strong>A:</strong> The standard location is at the very top (underneath the line that specifies the GLSL version).</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, a shader begins with the execution of what function? What’s the syntax for it?</summary>
<p><strong>A:</strong> <code>void main() {/* code goes here */}</code> (as the syntax indicates, this function takes no parameters and returns no value)</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL, what’s the general syntactical term for the <code>in</code> and <code>out</code> keywords?</summary>
<p><strong>A:</strong> They are <em>storage qualifiers</em>.</p>
</details>

<details>
<summary><strong>Q:</strong> What do <code>in</code> and <code>out</code> storage qualifiers indicate in a vertex shader?</summary>
<p><strong>A:</strong> <code>in</code> = vertex attribute. <code>out</code> = data to be interpolated (previously a <code>varying</code>).</p>
</details>

<details>
<summary><strong>Q:</strong> What do <code>in</code> and <code>out</code> storage qualifiers indicate in a fragment shader?</summary>
<p><strong>A:</strong> <code>in</code> = interpolated data (previously a <code>varying</code>). <code>out</code> = final fragment color.</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL ES 3.00, what is the syntax for setting up a “port” for a shader to receive data?</summary>
<p><strong>A:</strong> <code>layout(location = 0) in &lt;type&gt; &lt;variableName&gt;</code></p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL ES 3.00, what is the meaning of <code>layout(location = 0)</code>, in the line <code>layout(location = 0) in &lt;type&gt; &lt;variableName&gt;</code>?</summary>
<p><strong>A:</strong> “Receive data at location 0.”</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL ES 3.00, what is the meaning of <code>&lt;variableName&gt;</code>, in the line <code>layout(location = 0) in &lt;type&gt; &lt;variableName&gt;</code>?</summary>
<p><strong>A:</strong> This is the name of the variable that contains the data received at location 0.</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL ES 3.00, where does the line <code>layout(location = 0) in &lt;type&gt; &lt;variableName&gt;</code> go inside a shader?</summary>
<p><strong>A:</strong> It goes in the global scope of the shader, before <code>main()</code>.</p>
</details>

<details>
<summary><strong>Q:</strong> What special built-in variable must the Vertex Shader write to?</summary>
<p><strong>A:</strong> <code>gl_Position</code></p>
</details>

<details>
<summary><strong>Q:</strong> In a GLSL vertex shader, what is the data type of the built-in variable <code>gl_Position</code>?</summary>
<p><strong>A:</strong> <code>vec4</code></p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL ES 3.00, what is the syntax to define the output color variable in a fragment shader?</summary>
<p><strong>A:</strong> <code>out vec4 fragColor;</code> (You can name the variable whatever you want, but <code>fragColor</code> is conventional.)</p>
</details>

<details>
<summary><strong>Q:</strong> In GLSL ES 3.00, where does the code defining the output color variable in a fragment shader go?</summary>
<p><strong>A:</strong> It goes in the global scope of the shader, before <code>main()</code>.</p>
</details>

## Finishing the data bridge (enabling and configuring attributes)

<details>
<summary><strong>Q:</strong> An attribute will not be used unless it’s explicitly turned on using what syntax?</summary>
<p><strong>A:</strong> <code>gl.enableVertexAttribArray(index)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.enableVertexAttribArray(index)</code>, what does <code>index</code> represent?</summary>
<p><strong>A:</strong> The <code>location</code> of the attribute that will receive the data in the shader (set by <code>layout(location = index)</code>).</p>
</details>

<details>
<summary><strong>Q:</strong> What’s the signature of <code>gl.vertexAttribPointer()</code>? (Parameter list and return value.)</summary>
<p><strong>A:</strong><br />
<code>gl.vertexAttribPointer(index, size, type, normalized, stride, offset)</code><br />
<em>Return value:</em> None ( <code>undefined</code>).</p>
<p><strong>Hint:</strong> Mentally chunk the parameters into three pairs—<code>index</code>, <code>size</code>; <code>type</code>, <code>normalized</code>; <code>stride</code>, <code>offset</code>.</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what does the <code>index</code> parameter represent?</summary>
<p><strong>A:</strong> The <code>location</code> of the attribute that will receive the data in the shader (set by <code>layout(location = index)</code>).</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what does the <code>size</code> parameter represent?</summary>
<p><strong>A:</strong> The number of components per vertex (e.g., <code>2</code> for a <code>vec2</code>, <code>3</code> for a <code>vec3</code>).</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what does the <code>type</code> parameter represent?</summary>
<p><strong>A:</strong> The data type of the array components.</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, the <code>type</code> parameter is typically set to what value?</summary>
<p><strong>A:</strong> <code>gl.FLOAT</code></p>
</details>

<details>
<summary><strong>Q:</strong> 
  What precise data type is indicated by <code>gl.FLOAT</code>?
</summary>
<p><strong>A:</strong> 
  A 32-bit (IEEE) floating point number.
</p>
</details>

<details>
<summary>
  <strong>Q:</strong>
  A <code>gl.FLOAT</code> consists of how many <em>bytes</em>?
</summary>
<p>
  <strong>A:</strong> 
  4 bytes
</p>
<p>
  <strong>Hint:</strong> 
  A <code>gl.FLOAT</code> is a 32-bit floating-point data type. A byte consists of 8 bits.
</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what does the <code>normalized</code> parameter represent?</summary>
<p><strong>A:</strong> A boolean value indicating whether integer data should be normalized to $[-1, 1]$ or $[0, 1]$ when converted to a float (has no effect for floats, so it's typically set to <code>false</code> in that case, as enabling normalization would have no effect).</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what is a basic use case for the <code>normalized</code> parameter?</summary>
<p><strong>A:</strong> If RGB values for a color are provided in the range $[0, 255]$ (with a <code>type</code> of <code>gl.UNSIGNED_BYTE</code>), setting the <code>normalized</code> parameter to true will automatically convert that data to floats in the required $[0.0, 1.0]$ range for color data.</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what does the <code>stride</code> parameter represent?</summary>
<p><strong>A:</strong> Byte offset (distance in bytes) between the start of one vertex attribute and the next one of the same type. (Equivalently, the number of bytes used to store attributes corresponding to one vertex</p>
<p><strong>Hint:</strong> Imagine attributes are stored like <code>x0, y0, u0, v0, x1, y1, u1, v1…</code> The stride tells WebGL that the memory occupied by <code>x0, y0, u0, v0</code> corresponds to one vertex.</p>
</details>

<details>
<summary><strong>Q:</strong> What term do we use to describe attribute data like <code>x0, y0, u0, v0, x1, y1, u1, v1…</code> in which attributes of different kinds are stored together in the same buffer?</summary>
<p><strong>A:</strong> <em>Interleaved</em></p>
</details>

<details>
<summary><strong>Q:</strong> What term do we use to describe attribute data like <code>x0, y0, x1, y1,…</code> in which attributes in a buffer all have the same kind (e.g. they’re all positions)?</summary>
<p><strong>A:</strong> <em>Tightly packed</em></p>
<p><strong>Hint:</strong> If only positions are represented, then that means there’s zero space between positions (e.g. we don’t have position data, then color data, then position data, etc.).</p>
</details>

<details>
<summary><strong>Q:</strong> What value do we give <code>stride</code> when calling <code>gl.vertexAttribPointer()</code>, if we want data to be tightly packed?</summary>
<p><strong>A:</strong> <code>0</code></p>
<p><strong>Hint:</strong> This is a special case: if <code>0</code> were the byte offset from the start of one position to the start of the next (for example), that’d mean there’s no position data. So WebGL interprets zero to mean “tightly packed,” (e.g. zero bytes between the <em>end</em> of one position and the <em>start</em> of the next).</p>
</details>

<details>
<summary><strong>Q:</strong> If <code>stride</code> is set to zero when calling <code>gl.vertexAttribPointer()</code>, how can WebGL determine the byte offset to get from one attribute to the next?</summary>
<p><strong>A:</strong> WebGL interprets a <code>stride</code> of <code>0</code> to mean the data is tightly packed (e.g. all position data, with no color data in between). It then automatically calculates the correct byte offset based on the <code>size</code> and <code>type</code> parameters.</p>
</details>

<details>
<summary><strong>Q:</strong> When calling <code>gl.vertexAttribPointer()</code>, suppose <code>size</code> is set to <code>3</code>, <code>type</code> is set to <code>gl.FLOAT</code>, and <code>stride</code> is set to zero. WebGL will automatically calculate that the byte offset between attributes is equal to what value?</summary>
<p><strong>A:</strong> A stride of zero means the data is tightly packed, so we have attributes with three components packed right next to each other. A <code>gl.FLOAT</code> consists of four bytes. So, the byte offset is 3 components $\times$ 4 bytes / component = 12 bytes.</p>
</details>

<details>
<summary><strong>Q:</strong> Roughly, when might it be useful to use tightly packed attributes in a WebGL array buffer?</summary>
<p><strong>A:</strong> Using tightly packed attributes means that all positions would go into one array buffer, all colors would go into another, etc. This can be useful for <strong>dynamic geometry</strong>, e.g. when positions need to be updated but not colors.</p>
</details>

<details>
<summary><strong>Q:</strong> Roughly, when might it be useful to use interleaved attributes in a WebGL array buffer?</summary>
<p><strong>A:</strong> This keeps all data for a single vertex close together in memory, which can be more efficient for <strong>static geometry</strong>, e.g. where it’s not necessary to update positions but keep colors the same. (Interleaved attributes also make it possible to deal with just a single buffer.)</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.vertexAttribPointer()</code>, what does the <code>offset</code> parameter represent?</summary>
<p><strong>A:</strong> The byte offset from the start of the buffer to the first component of the first vertex attribute.</p>
</details>

<details>
<summary><strong>Q:</strong> When <code>gl.vertexAttribPointer()</code> is called, how does WebGL know which VBO to read data from?</summary>
<p><strong>A:</strong> It uses whichever buffer is bound to the <code>gl.ARRAY_BUFFER</code> when <code>gl.vertexAttribPointer()</code> is called.</p>
</details>

<details>
<summary><strong>Q:</strong> What object stores the configuration set by <code>gl.vertexAttribPointer()</code> and <code>gl.enableVertexAttribArray()</code>?</summary>
<p><strong>A:</strong> The Vertex Array Object (VAO).</p>
</details>

<details>
<summary><strong>Q:</strong> Why do we use VAOs instead of just binding VBOs and setting pointers every frame?</summary>
<p><strong>A:</strong> A VAO allows us to store complex attribute setups once and restore them with a single bind call.</p>
</details>

## WebGL2 shader compilation

<details>
  <summary>
    <strong>Q:</strong> 
    In WebGL, what are the high-level steps to setting up a shader object? Answer in words.
  </summary>
<p>
  <strong>A:</strong>
    <ol>
      <li><em>Create</em></li>
      <li><em>Upload</em> (the GLSL source code)</li>
      <li><em>Compile</em></li>
      <li><em>Check</em> (the compile status)</li>
      <li>If compiling failed, <em>Handle</em> it (by logging or throwing the error and deleting the shader).</li>
    </ol>
</p>
<p>
  <strong>Hint:</strong>
  This list can be chunked into two stages. 
  <ol>
    <li> 
      As with setting up a VBO, we need to <em>Create</em> the object before we <em>Upload</em> to it.
    </li>
    <li>
      Since this is a program, we then need to <em>Compile</em> it, <em>Check</em> for errors, and then <em>Handle</em> errors if needed.
    </li>
  </ol>
</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what’s the syntax for creating a shader?</summary>
<p><strong>A:</strong> <code>gl.createShader(type)</code></p>
</details>

<details>
<summary><strong>Q:</strong> What are the two types of shaders passed to <code>gl.createShader()</code>?</summary>
<p><strong>A:</strong> <code>gl.VERTEX_SHADER</code> and <code>gl.FRAGMENT_SHADER</code>.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what’s the syntax to upload the GLSL source code string to a shader object?</summary>
<p><strong>A:</strong> <code>gl.shaderSource(shader, sourceString)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what’s the syntax to compile a shader?</summary>
<p><strong>A:</strong> <code>gl.compileShader(shader)</code></p>
</details>

<details>
<summary><strong>Q:</strong> After compiling a shader, what syntax checks if it succeeded?</summary>
<p><strong>A:</strong> <code>gl.getShaderParameter(shader, gl.COMPILE_STATUS)</code></p>
</details>

<details>
<summary><strong>Q:</strong> What is the return type of <code>gl.getShaderParameter(shader, gl.COMPILE_STATUS)</code>?</summary>
<p><strong>A:</strong> <code>Boolean</code></p>
</details>

<details>
<summary><strong>Q:</strong> If <code>gl.getShaderParameter(shader, gl.COMPILE_STATUS)</code> indicates an error has occurred, what syntax gets a string with information about the error?</summary>
<p><strong>A:</strong> Use <code>gl.getShaderInfoLog(shader)</code>.</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, why should you delete a shader object after it fails to compile?</summary>
<p><strong>A:</strong> If it fails to compile, it’s garbage (useless). Deleting it prevents memory leaks (accumulation of useless memory).</p>
</details>

<details>
<summary><strong>Q:</strong> What function removes a shader object from GPU memory?</summary>
<p><strong>A:</strong> <code>gl.deleteShader(shader)</code></p>
</details>

## WebGL2 program linking

<details>
<summary><strong>Q:</strong> In WebGL, what does it mean to “link” a program?</summary>
<p><strong>A:</strong> Linking a program connects it to dependencies, resulting in a program that’s executable.</p>
</details>

<details>
<summary><strong>Q:</strong> Regarding executability, what is the difference between a shader object and a program object in WebGL?</summary>
<p><strong>A:</strong> A <em>shader</em> is an intermediate (unlinked) compiled stage. A <em>program</em> is linked and ready to run (like an <code>.exe</code>).</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what are the high-level steps to setting up a program object? Answer in words.</summary>
<p><strong>A:</strong></p>
<ol>
<li><em>Create</em></li>
<li><em>Attach</em> (shaders)</li>
<li><em>Link</em> (program)</li>
<li><em>Check</em> (the link status)</li>
<li>If linking failed, <em>Log</em> (error) and <em>Delete</em> (the program).</li>
</ol>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what’s the syntax for creating a program object?</summary>
<p><strong>A:</strong> <code>gl.createProgram()</code> (no parameters)</p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, what’s the syntax for attaching a shader to a program object?</summary>
<p><strong>A:</strong> <code>gl.attachShader(program, shader)</code> (attaches a vertex or a fragment shader)</p>
</details>

<details>
<summary><strong>Q:</strong> After attaching shaders to a program, what syntax connects them into a usable executable?</summary>
<p><strong>A:</strong> <code>gl.linkProgram(program)</code></p>
</details>

<details>
<summary><strong>Q:</strong> After linking a WebGL program, how do you check if it succeeded?</summary>
<p><strong>A:</strong> <code>gl.getProgramParameter(program, gl.LINK_STATUS)</code></p>
</details>

<details>
<summary><strong>Q:</strong> What is the return type of <code>gl.getProgramParameter(program, gl.LINK_STATUS)</code>?</summary>
<p><strong>A:</strong> <code>Boolean</code></p>
</details>

<details>
<summary><strong>Q:</strong> If <code>gl.getProgramParameter(program, gl.LINK_STATUS)</code> indicates an error has occurred, what syntax gets a string with information about the error?</summary>
<p><strong>A:</strong> <code>gl.getProgramInfoLog(program)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In WebGL, why should you delete a program object after it fails to link?</summary>
<p><strong>A:</strong> If it fails to link, it’s garbage (useless). Deleting it prevents memory leaks (accumulation of useless memory).</p>
</details>

<details>
<summary><strong>Q:</strong> What function removes a program object from GPU memory?</summary>
<p><strong>A:</strong> <code>gl.deleteProgram(program)</code></p>
</details>

## Drawing

<details>
<summary><strong>Q:</strong> In the WebGL2 API, what syntax renders primitives?</summary>
<p><strong>A:</strong> <code>gl.drawArrays(mode, first, count)</code></p>
</details>

<details>
<summary><strong>Q:</strong> In the WebGL2 API, what does "arrays" refer to in <code>gl.drawArrays()</code>?</summary>
<p><strong>A:</strong> As it carries out the drawing task, this function aggregates vertex attributes from multiple arrays (e.g. position, color, and texture arrays), assembling all data for vertex 1, then for vertex 2, and so on.</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.drawArrays(mode, first, count)</code>, what are the possible values of the <code>mode</code> parameter? Answer with conceptual descriptions.</summary>
<p><strong>A:</strong> Disconnected points; disconnected lines, an open polyline, or a closed polyline; disconnected triangles, a triangle strip, or a triangle fan.</p>
<p><strong>Hint:</strong> The <code>mode</code> parameter is analogous to the <code>kind</code> parameter in p5's <code>beginShape(kind)</code>.</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.drawArrays(mode, first, count)</code>, what are the possible values of the <code>mode</code> parameter? Answer with variable names.</summary>
<p><strong>A:</strong> <code>gl.POINTS</code>; <code>gl.LINES</code>, <code>gl.LINE_STRIP</code>, <code>gl.LINE_LOOP</code>; <code>gl.TRIANGLES</code>, <code>gl.TRIANGLE_STRIP</code>, <code>gl.TRIANGLE_FAN</code>.</p>
<p><strong>Hint:</strong> The <code>mode</code> parameter is analogous to the <code>kind</code> parameter in p5's <code>beginShape(kind)</code>.</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.drawArrays(mode, first, count)</code>, what does the <code>first</code> parameter represent?</summary>
<p><strong>A:</strong> The starting index to read from, in the arrays of vertex attributes. (Usually 0.)</p>
</details>

<details>
<summary><strong>Q:</strong> In <code>gl.drawArrays(mode, first, count)</code>, what does the <code>count</code> parameter represent?</summary>
<p><strong>A:</strong> The number of vertices to be processed (rendered).</p>
</details>

<details>
<summary><strong>Q:</strong> Before issuing a <code>gl.drawArrays</code> command, what must you tell the GPU, conceptually?</summary>
<p><strong>A:</strong> You must tell it which shader program to use.</p>
</details>

<details>
<summary><strong>Q:</strong> What syntax tells <code>gl.drawArrays</code> which shader program to execute?</summary>
<p><strong>A:</strong> <code>gl.useProgram(program)</code></p>
<p><strong>Hint:</strong> This tells the WebGL state machine: "For all subsequent draw calls, use this specific compiled executable."</p>
</details>

## Project 3: Make boilerplate helper and draw triangle
<img 
  width="250" 
  height="250" 
  alt="yellow canvas with an orange triangle in the center" 
  src="https://github.com/user-attachments/assets/47448258-5800-45aa-8e61-a172ed90d46a" 
/>

<p>
<strong>Goal:</strong> Update <code>yellow-canvas.js</code> to render your existing triangle geometry in orange, on top of the yellow background.
</p>

<p>
<strong>Context:</strong> From <a href="#project-2-set-up-vbo-and-vao-supply-triangle-data">Project 2</a>, you already have the geometry (a <code>Float32Array</code> of 2D coordinates) in a VBO, and a VAO that is currently bound. Now you need to build the program to process that data.
</p>

<strong>Project Specifications:</strong>

1.  <strong>Helper Function:</strong> Create a function <code>createProgram(gl, vsSource, fsSource)</code> at the bottom of your file.
    * It must create two shaders and one program.
    * It must compile the shaders and check their compile status.
    * It must link the program and check its link status.
    * <strong>Constraint:</strong> If any check fails, log the error and <strong>delete</strong> the faulty object to avoid memory leaks. Return <code>null</code> if it fails, or the <code>program</code> if it succeeds.
2.  <strong>Shader Source Code:</strong> Define two template strings, <code>vsSource</code> and <code>fsSource</code>.
    * <strong>Vertex Shader:</strong>
        * Accept an attribute <code>position</code> at location 0. Note that your buffer has 2 numbers per vertex, so this should be a <code>vec2</code>.
        * Output a <code>gl_Position</code>. (Hint: You will need to convert your <code>vec2</code> input.)
    * <strong>Fragment Shader:</strong>
        * Declare the variable to output.
        * Output the color orange: <code>vec4(1.0, 0.4, 0.0, 1.0)</code>.
3.  <strong>Execution:</strong>
    * Call your helper to create the program.
    * Tell WebGL to use this program.
    * Draw the triangle.

<details>
  <summary><strong>Solution:</strong></summary>

```javascript
// CANVAS
const canvas = document.getElementById('yellow-canvas');
const gl = canvas.getContext('webgl2');
const yellow = [243 / 255, 208 / 255, 62 / 255, 1];

gl.clearColor(...yellow);
gl.clear(gl.COLOR_BUFFER_BIT);

// TRIANGLE
const TAU = 2 * Math.PI;
const r = 2 / 3;
const t0 = TAU / 4;
const dt = TAU / 3;

const triangleVertices = new Float32Array([
  r * Math.cos(t0), r * Math.sin(t0), 
  r * Math.cos(t0 + dt), r * Math.sin(t0 + dt), 
  r * Math.cos(t0 + 2 * dt), r * Math.sin(t0 + 2 * dt),
]);

// SHADER SOURCE
const vsSource = `#version 300 es
layout(location = 0) in vec2 position;

void main() {
  gl_Position = vec4(position, 0.0, 1.0);
}
`;

const fsSource = `#version 300 es
precision highp float;
out vec4 fragColor;

void main() {
  fragColor = vec4(1.0, 0.4, 0.0, 1.0);
}
`;

// STATE MANAGEMENT: VAO AND VBO
const vao = gl.createVertexArray();
gl.bindVertexArray(vao);
const vbo = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
gl.bufferData(gl.ARRAY_BUFFER, triangleVertices, gl.STATIC_DRAW);
gl.enableVertexAttribArray(0);
gl.vertexAttribPointer(0, 2, gl.FLOAT, false, 0, 0);

// CREATE AND USE PROGRAM TO DRAW
const program = createProgram(gl, vsSource, fsSource);
gl.useProgram(program);
gl.drawArrays(gl.TRIANGLES, 0, 3);

// CREATION UTILITIES: SHADERS AND PROGRAM 
function createShader(gl, type, source) {
  const shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  
  // Check success
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    const shaderInfoLog = gl.getShaderInfoLog(shader);
    gl.deleteShader(shader);
    throw new Error(`Could not compile shader: ${shaderInfoLog}`);
  }
  return shader;
}

function createProgram(gl, vsSource, fsSource) {
  const vs = createShader(gl, gl.VERTEX_SHADER, vsSource);
  const fs = createShader(gl, gl.FRAGMENT_SHADER, fsSource);
  
  const program = gl.createProgram();
  gl.attachShader(program, vs);
  gl.attachShader(program, fs);
  gl.linkProgram(program);
  
  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
    const programInfoLog = gl.getProgramInfoLog(program);
    gl.deleteProgram(program);
    throw new Error(`Could not link program: ${programInfoLog}`);
  }
  return program;
}
```
</details>

# Hello spinning cube
Time for some 3D action!

## Uniforms and matrices in GLSL

<details><summary><strong>Q:</strong> In GLSL, what storage qualifier is used for variables that remain constant for all vertices in a single draw call (e.g., a transformation matrix)?</summary> <p><strong>A:</strong> <code>uniform</code></p></details>

<details> <summary><strong>Q:</strong> In WebGL, before you can set the value of a uniform, you must look up its address using what syntax?</summary> <p><strong>A:</strong> <code>gl.getUniformLocation(program, name)</code></p> <p><strong>Hint:</strong> Just as attributes have integer locations, uniforms have location objects.</p></details>

<details> <summary><strong>Q:</strong> In <code>gl.getUniformLocation(program, name)</code>, you must be sure that <code>name</code> has what data type?</summary> <p><strong>A:</strong> A string. (Be sure to wrap the name of the uniform in single quotes, so that it's just a variable <em>name</em>, not an actual variable.)</p></details>

<details><summary><strong>Q:</strong> What do "row-major" and "column-major" mean when specifying matrices?</summary> <p><strong>A:</strong> Imagine reading a matrix aloud to someone else. You'll either read it to them row by row (<em>row-major</em> order) or column by column (<em>column-major</em> order). (The same concept applies when storing a matrix in a flat array.)</p> </details>

<details><summary><strong>Q:</strong> In GLSL, what data type represents a $4\times4$ matrix?</summary> <p><strong>A:</strong> <code>mat4</code> (a common shorthand for <code>mat4x4</code>)</p> </details>

<details><summary><strong>Q:</strong> In WebGL and GLSL, what is the convention regarding matrix order (column-major or row-major), if any? </summary> <p><strong>A:</strong> Always column major.</p> <p><strong>Hint:</strong> This is consistent with mathematical conventions, whereby the matrix of a transformation is built from vectors that are represented as columns.</p></details>

<details><summary><strong>Q:</strong> What is the order of multiplication when multiplying a matrix $M$ by a vector $v$, when following a column-major convention? </summary> <p><strong>A:</strong> $Mv$ </p> <p><strong>Hint:</strong> Since $v$ is a column, it must go on the right for the matrix product to be defined, in general.</p></details>

<details><summary><strong>Q:</strong> What is the order of multiplication when multiplying a matrix $M$ by a vector $v$, when following a row-major convention? </summary> <p><strong>A:</strong> $vM$ </p> <p><strong>Hint:</strong> Since $v$ is a row, it must go on the left for the matrix product to be defined, in general.</p></details>

<details> <summary><strong>Q:</strong> In WebGL, if you want to set the value of a uniform variable declared as a <code>mat4</code> in GLSL, what syntax do you use?</summary> <p><strong>A:</strong> <code>gl.uniformMatrix4fv(location, transpose, data)</code></p> <p><strong>Hint:</strong> It stands for "uniform Matrix 4 float vector," where float vector refers to the type of <code>data</code> used to specify the matrix.</p> </details>

<details> <summary><strong>Q:</strong> When calling <code>gl.uniformMatrix4fv()</code> in WebGL, what must the <code>transpose</code> argument always be?</summary> <p><strong>A:</strong> <code>false</code></p> <p><strong>Hint:</strong> The <code>transpose</code> parameter is kept for consistency with OpenGL, but WebGL requires this to be <code>false</code>, so that matrices are always in column-major order.</p> </details>

<details> <summary><strong>Q:</strong> When calling <code>gl.uniformMatrix4fv()</code> in WebGL, how is the <code>data</code> parameter typically specified?</summary> <p><strong>A:</strong> It's typically specified as a <code>Float32Array</code> (it could also be a sequence of separate 32-bit floats).</p></details>

<details> <summary><strong>Q:</strong> What function must you call before setting any uniforms? </summary> <p><strong>A:</strong> <code>gl.useProgram()</code></p></details>

<details> <summary><strong>Q:</strong> Why must a program be in use before setting any uniforms? </summary> <p><strong>A:</strong> Uniforms are global to the <em>program object</em> and are stored in that object, not in the global WebGL state. (WebGL needs to know <em>which</em> program's memory you intend to update.)</p> </details>

<details>
<summary><strong>Q:</strong> WebGL clip space is left-handed ($+z$ into screen). However, the popular <code>glMatrix</code> matrix library uses a right-handed system ($+z$ towards viewer), which aligns with standard mathematical conventions. Which matrix in <code>glMatrix</code> handles the conversion between them?</summary>
<p><strong>A:</strong> The projection matrix (it flips the $z$-axis).</p>
</details>

## GLSL matrix multiplication
<details> <summary><strong>Q:</strong> In GLSL, how do we compute the matrix-vector product $Mv$, where $M$ is a <code>mat4</code> and $v$ is a <code>vec4</code>?</summary> <p><strong>A:</strong> <code>M * v</code></details>

<details> <summary><strong>Q:</strong> In GLSL, how do we compute the matrix product $AB$, where $A$ is a <code>mat4</code> and $B$ is a <code>mat4</code>? (Here, $AB$ refers to the standard matrix product, not an entrywise/Hadamard product.) </summary> <p><strong>A:</strong> <code>A * B</code></p></details>

<details> <summary><strong>Q:</strong> In GLSL, what happens if you try to multiply <code>vector * matrix</code> (vector on the left)?</summary> <p><strong>A:</strong> It treats the vector as a row vector.</p> <p><strong>Hint:</strong> This is valid syntax but usually not what we want, since WebGL adheres to column-major order.</p> </details>

## 3D-state management (depth and culling)

<details> <summary><strong>Q:</strong> In WebGL, what feature must be enabled to prevent background triangles from drawing on top of foreground triangles? Answer in words. </summary> <p><strong>A:</strong> The <em>depth test</em>.</p> </details>

<details> <summary><strong>Q:</strong> What syntax enables the depth test in WebGL?</summary> <p><strong>A:</strong> <code>gl.enable(gl.DEPTH_TEST)</code></p> </details>

<details> <summary><strong>Q:</strong> Does setting <code>gl.enable(gl.DEPTH_TEST)</code> require an active program?</summary> <p><strong>A:</strong> No.</p> <p><strong>Hint:</strong> Depth testing is part of the <em>global context state</em>, not the program state.</p> </details>

<details> <summary><strong>Q:</strong> When the depth test is enabled in WebGL, what update must you be sure to make every frame? Answer in words. </summary> <p><strong>A:</strong> Clear the depth buffer. (This ensures that old data doesn't persist.)</p> </details>

<details> <summary><strong>Q:</strong> In computer graphics, what is "face culling"?</summary> <p><strong>A:</strong> It's an optimization that avoids drawing faces that wouldn't be visible anyway (e.g. the back face of a cube).</p> </details>

<details> <summary><strong>Q:</strong> In WebGL, face culling is applied to triangles if they have what spatial relation to the camera?</summary> <p><strong>A:</strong> The triangles are culled if they are facing away from the camera.</p> <p><strong>Hint:</strong> Imagine that you color a paper triangle red, but if someone flips it over, they'll see it's still white on the other side. That's the back face. WebGL also has a way of determining which face of a triangle is the front and which is the back.</p></details>

<details> <summary><strong>Q:</strong> By default, WebGL determines a triangle is "front-facing" if its vertices are defined in what winding order?</summary> <p><strong>A:</strong> Counter-Clockwise (CCW).</p> <p><strong>Hint:</strong> This is the positive orientation in the xy-plane: starting from the positive x-axis, this direction moves us through Quadrant I first.</p></details>

<details> <summary><strong>Q:</strong> What syntax enables face culling in WebGL?</summary> <p><strong>A:</strong> <code>gl.enable(gl.CULL_FACE)</code></p> </details>

## 3D geometry definition (winding order)
<details> <summary><strong>Q:</strong> In WebGL, when defining the vertices of a 3D mesh (like a cube), in what winding order should you list the vertices for every face?</summary> <p><strong>A:</strong> Counter-Clockwise (CCW).</p> </details>

<details> <summary><strong>Q:</strong> In WebGL, when determining the CCW winding order for a specific face of a 3D object, where should you imagine yourself standing?</summary> <p><strong>A:</strong> Outside the object, looking directly at the face.</p></details>

<details><summary><strong>Q:</strong> Suppose you defined a face with CCW winding. How does WebGL know when it's hidden from view and can therefore be culled? </summary> <p><strong>A:</strong> If the camera moves behind the face you defined, then the winding order appears to be clockwise on the 2D screen. So, WebGL calculates the 2D winding on the screen; if it flips to CW, it knows you're looking at the back.</p> </details>

## The animation loop
Here, we learn a general Web API for animations that is exposed to JavaScript. It can be used for many things. We will use it to create an animation with WebGL2.

<details> <summary><strong>Q:</strong> In the browser, what API is the standard for creating smooth animations? Answer with the precise syntax.</summary> <p><strong>A:</strong> <code>requestAnimationFrame(callback)</code></p> </details>

<details> <summary><strong>Q:</strong> Why is <code>requestAnimationFrame()</code> so named? </summary> <p><strong>A:</strong> It requests the browser to call the provided callback function, which determines the next frame in the animation.</p> </details>

<details> <summary><strong>Q:</strong> How does <code>requestAnimationFrame()</code> behave when the browser tab is inactive (not visible)?</summary> <p><strong>A:</strong> It pauses (or slows down significantly) to save battery and CPU cycles.</p> </details>

<details> <summary><strong>Q:</strong> <code>requestAnimationFrame()</code> runs its callback exactly once. How do you create a continuous loop?</summary> <p><strong>A:</strong> Call <code>requestAnimationFrame</code> recursively inside the callback function.</p> </details>

<details> <summary><strong>Q:</strong> What argument does <code>requestAnimationFrame()</code> automatically pass to its callback function?</summary> <p><strong>A:</strong> A <code>timestamp</code> argument (a <code>DOMHighResTimeStamp</code> type, indicating when the frame starts).</p> </details>

<details> 
  <summary>
    <strong>Q:</strong> 
    What is the minimal code structure for a continuous animation loop created with <code>requestAnimationFrame()</code>? (Assume the callback is named <code>draw</code>).
  </summary> 
  <p><strong>A:</strong></p>

  ```javascript
  function draw(timestamp) {
    // 1. Update state and render...
    // 2. Schedule next frame
    requestAnimationFrame(draw);
  }

  // 3. Start the loop
  requestAnimationFrame(draw);
```

</details>

<details><summary><strong>Q:</strong> The <code>timestamp</code> passed to the <code>requestAnimationFrame()</code> callback represents time in what unit?</summary><p><strong>A:</strong> Milliseconds.</p></details>

<details><summary><strong>Q:</strong> The <code>timestamp</code> passed to the <code>requestAnimationFrame()</code> callback measures time elapsed since what event? (Be general).</summary><p><strong>A:</strong> Since the time origin (usually when the page loaded).</p></details>

<details><summary><strong>Q:</strong> In the context of an animation created with <code>requestAnimationFrame()</code>, what does it mean to calculate a <em>zeroed</em> time?</summary><p><strong>A:</strong> It refers to calculating an elapsed time starting when the animation logic begins, rather than when the page loaded.</p></details>

<details>
  <summary>
    <strong>Q:</strong> 
    What's a simple way to calculate a zeroed time for an animation made with <code>requestAnimationFrame(callback)</code>? Sketch your answer in code, using a callback function named <code>draw</code>.
  </summary>
  <p><strong>A:</strong></p>

```javaScript
let startTime;

function draw(timestamp) {
  if (!startTime) {
    startTime = timestamp;
  }
  const elapsed = timestamp - startTime;
  
  // draw logic...
  
  requestAnimationFrame(draw);
}
```
  
</details>
<details> <summary><strong>Q:</strong> What syntax stops a scheduled animation frame request made with <code>requestAnimationFrame()</code>?</summary> <p><strong>A:</strong> <code>cancelAnimationFrame(requestID)</code></p> </details>

<details> <summary><strong>Q:</strong> Where do you get the <code>requestID</code> needed to cancel an animation frame request made with <code>requestAnimationFrame()</code>?</summary> <p><strong>A:</strong> It is the return value of the <code>requestAnimationFrame()</code> call.</p> </details>

## Project 4: The Spinning Cube

<img width="250" height="250" alt="spinning multicolored cube" src="https://github.com/user-attachments/assets/57bca82d-cfbf-48d0-9623-bc764039f39b" />

**Goal:** Render a multicolored unit cube, centered at the origin, that rotates in 3D space. You may reuse logic from Project 3 as appropriate.

**Allowed linear-algebra dependency:** You may use [glMatrix](https://glmatrix.net/) for the matrix transformations, by downloading [`gl-matrix-min.js`](https://github.com/toji/gl-matrix/blob/master/dist/gl-matrix-min.js) from the GitHub repo, putting it into a folder called `libs` in your project directory, and then including it in `index.html` with a `<script>` element above the line where you include your own script. You may also use the [`glMatrix` documentation](https://glmatrix.net/docs/module-glMatrix.html) as a reference if needed. Note that the library exposes a global `glMatrix` object: you'll typically access functions via `glMatrix.mat4.create()`, `glMatrix.vec3.fromValues()`, etc. Also note that vectors and matrices (e.g. `mat4` and `vec3`) are all `Float32Array` instances.

**Approach:** To allow distinct colors for each face, you may duplicate vertices. There will then be 36 vertices total: 6 faces $\times$ 2 triangles $\times$ 3 vertices.

Specifications:

1. **State Management:**
   * Enable the depth test and face culling.
   * Create a VAO.
   * Create two VBOs: one for `positions`, one for `colors` (Note: You can use two <code>bufferData</code> calls and two <code>vertexAttribPointer</code> calls attached to the same VAO).
   * Configure `position` (attribute location 0) and `color` (attribute location 1).
2. **Shaders:**
   * **Vertex Shader:**
     * Attributes: `in vec3 position`, `in vec3 color`.
     * Uniforms: `uniform mat4 uModel`, `uniform mat4 uView`, `uniform mat4 uProjection`.
     * Output: `out vec3 vColor` ("v" is conventional and stands for "varying," just as "u" stands for "uniform" in `uModel`).
     * Main: Set `gl_Position = uProjection * uView * uModel * vec4(position, 1.0);`. Pass `color` to `vColor`.
   * **Fragment Shader:**
     * Input: `in vec3 vColor`.
     * Output: `fragColor` using the interpolated input color (alpha 1.0).
3. **Matrix Logic (`glMatrix`):** Create model, view, and projection matrices. Upload view and projection matrices via `gl.uniformMatrix4fv`.
    * **Model:** Use `mat4.create()`.
    * **View:** Use `mat4.lookAt`. (Eye: `[0, 0, 4]`, Center: `[0, 0, 0]`, Up: `[0, 1, 0]`).
    * **Projection:** Use `mat4.perspective`. (FOV: $\frac{\pi}{4}$ radians, Aspect: canvas width/height, Near: 0.1, Far: 100.0).
4. **Render Loop:**
    * Use `requestAnimationFrame`.
    * Clear both color and depth buffers.
    * Update the model matrix (rotate it slightly every frame with `mat4.rotate`).
    * Upload the model matrix via `gl.uniformMatrix4fv`.
    * Draw 36 vertices using `gl.TRIANGLES`.

<details>
  <summary><strong>Solution:</strong></summary>

The solution below moves the WebGL utility functions `createShader()` and `createProgram()` into their own file.

<strong><code>index.html</code>:</strong>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Spinning cube</title>
</head>
<body>
    <canvas id="spinning-cube-canvas" width="400" height="400">
      A canvas with a spinning cube.
    </canvas>
    <script src="libs/gl-matrix-min.js"></script>
    <script src="webgl-utilities.js"></script>
    <script src="spinning-cube.js"></script>
</body>
</html>
```

<strong><code>spinning-cube.js</code>:</strong>
```js
// GET CONTEXT
const canvas = document.getElementById('spinning-cube-canvas');
const gl = canvas.getContext('webgl2');

// SET CANVAS BACKGROUND COLOR
const lightGray = [220 / 255, 220 / 255, 220 / 255, 1];
gl.clearColor(...lightGray);

// LOAD CUBE DATA
const positions = getCubePositions();
const colors = getCubeColors();

// CREATE MATRICES
const uModel = glMatrix.mat4.create();

const uView = glMatrix.mat4.create();
const eye = glMatrix.vec3.fromValues(0, 0, 4);
const center = glMatrix.vec3.fromValues(0, 0, 0);
const up = glMatrix.vec3.fromValues(0, 1, 0);
glMatrix.mat4.lookAt(uView, eye, center, up);

const uProjection = glMatrix.mat4.create();
const fovy = Math.PI / 4;
const aspect = canvas.width / canvas.height;
const near = 0.1;
const far = 100.0;
glMatrix.mat4.perspective(uProjection, fovy, aspect, near, far);

// CREATE SHADER SOURCE
const vsSource = `#version 300 es
layout(location = 0) in vec3 position;
layout(location = 1) in vec3 color;
uniform mat4 uModel;
uniform mat4 uView;
uniform mat4 uProjection;
out vec3 vColor;

void main() {
  gl_Position = uProjection * uView * uModel * vec4(position, 1.0);
  vColor = color;
}
`;

const fsSource = `#version 300 es
precision highp float;
in vec3 vColor;
out vec4 fragColor;

void main() {
  fragColor = vec4(vColor, 1.0);
}
`;

// MANAGE STATE: VAO AND VBO
const vao = gl.createVertexArray();
gl.bindVertexArray(vao);

const positionVBO = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, positionVBO);
gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW);
gl.enableVertexAttribArray(0);
gl.vertexAttribPointer(0, 3, gl.FLOAT, false, 0, 0);

const colorVBO = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, colorVBO);
gl.bufferData(gl.ARRAY_BUFFER, colors, gl.STATIC_DRAW);
gl.enableVertexAttribArray(1);
gl.vertexAttribPointer(1, 3, gl.FLOAT, false, 0, 0);

// CREATE AND ACTIVATE PROGRAM
const program = createProgram(gl, vsSource, fsSource);
gl.useProgram(program);

// ACTIVATE 3D OPERATIONS
gl.enable(gl.DEPTH_TEST);
gl.enable(gl.CULL_FACE);

// GET MATRIX LOCATIONS
const uModelLocation = gl.getUniformLocation(program, 'uModel');
const uViewLocation = gl.getUniformLocation(program, 'uView');
const uProjectionLocation = gl.getUniformLocation(program, 'uProjection');

// SET STATIC MATRICES
gl.uniformMatrix4fv(uViewLocation, false, uView);
gl.uniformMatrix4fv(uProjectionLocation, false, uProjection);

// CREATE AXIS OF ROTATION (a unit vector indicates the direction of the axis)
const axis = glMatrix.vec3.fromValues(1, 1, 0);
glMatrix.vec3.normalize(axis, axis);

let startTime;

// Tell WebGL how to map the Normalized Device Coordinates (NDC) 
// of the clip space (-1 to +1) to pixel coordinates on the screen.
// Required if the canvas is resized after the context is created.
gl.viewport(0, 0, canvas.width, canvas.height);

// ANIMATE
function draw(timestamp) {
  if (!startTime) {
    startTime = timestamp;
  }
  
  const elapsedTime = timestamp - startTime;

  gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

  // Rotate based on elapsed time (0.001 converts ms to seconds)
  glMatrix.mat4.identity(uModel); 
  glMatrix.mat4.rotate(uModel, uModel, elapsedTime * 0.001, axis)
  gl.uniformMatrix4fv(uModelLocation, false, uModel);

  gl.drawArrays(gl.TRIANGLES, 0, 36);

  requestAnimationFrame(draw);
}

requestAnimationFrame(draw);

// CUBE LOADERS
function getCubePositions() {
  // 36 vertices (x, y, z)
  return new Float32Array([
      // Front face
      -0.5, -0.5,  0.5,
      0.5, -0.5,  0.5,
      0.5,  0.5,  0.5,
      -0.5, -0.5,  0.5,
      0.5,  0.5,  0.5,
      -0.5,  0.5,  0.5,

      // Back face
      -0.5, -0.5, -0.5,
      -0.5,  0.5, -0.5,
      0.5,  0.5, -0.5,
      -0.5, -0.5, -0.5,
      0.5,  0.5, -0.5,
      0.5, -0.5, -0.5,

      // Top face
      -0.5,  0.5, -0.5,
      -0.5,  0.5,  0.5,
      0.5,  0.5,  0.5,
      -0.5,  0.5, -0.5,
      0.5,  0.5,  0.5,
      0.5,  0.5, -0.5,

      // Bottom face
      -0.5, -0.5, -0.5,
      0.5, -0.5, -0.5,
      0.5, -0.5,  0.5,
      -0.5, -0.5, -0.5,
      0.5, -0.5,  0.5,
      -0.5, -0.5,  0.5,

      // Right face
      0.5, -0.5, -0.5,
      0.5,  0.5, -0.5,
      0.5,  0.5,  0.5,
      0.5, -0.5, -0.5,
      0.5,  0.5,  0.5,
      0.5, -0.5,  0.5,

      // Left face
      -0.5, -0.5, -0.5,
      -0.5, -0.5,  0.5,
      -0.5,  0.5,  0.5,
      -0.5, -0.5, -0.5,
      -0.5,  0.5,  0.5,
      -0.5,  0.5, -0.5,
  ]);
}

function getCubeColors() {
  // 36 colors (r, g, b)
  // Colors match faces (e.g., first 6 vertices are red, next 6 are green, etc.)
  return new Float32Array([
      // Front: Red
      1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0,
      // Back: Green
      0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0,
      // Top: Blue
      0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1,
      // Bottom: Yellow
      1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0,
      // Right: Purple
      1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1,
      // Left: Cyan
      0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1,
  ]);
}
```

<strong><code>webgl-utilities.js</code>:</strong>
```javascript  
// CREATION UTILITIES: SHADERS AND PROGRAM 
function createShader(gl, type, source) {
  const shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  
  // Check success
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    const shaderInfoLog = gl.getShaderInfoLog(shader);
    gl.deleteShader(shader);
    throw new Error(`Could not compile shader: ${shaderInfoLog}`);
  }
  return shader;
}

function createProgram(gl, vsSource, fsSource) {
  const vs = createShader(gl, gl.VERTEX_SHADER, vsSource);
  const fs = createShader(gl, gl.FRAGMENT_SHADER, fsSource);
  
  const program = gl.createProgram();
  gl.attachShader(program, vs);
  gl.attachShader(program, fs);
  gl.linkProgram(program);
  
  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
    const programInfoLog = gl.getProgramInfoLog(program);
    gl.deleteProgram(program);
    throw new Error(`Could not link program: ${programInfoLog}`);
  }
  return program;
}
```

</details>

# Next steps

Now that you have the irreducible minimum of the programmable geometry pipeline memorized, you are ready to build. You can use the following resources to apply your skills to advanced procedural generation, to deepen your theoretical understanding, or to solve specific implementation problems.



* **[RMF Engine](https://github.com/GregStanton/proposal-rmf-engine):**

  * _What it is:_ A high-fidelity computational engine and API for creative coding, supporting sweep geometries (brushes, ribbons, tubes) and motion rails (camera trajectories, write-on effects, choreographed motion) .

  * _How to use it:_ This is the perfect capstone project to test your new skills, with significant real-world benefits. You can help flesh out the proof of concept, or implement the spec in your favorite creative-coding library. This path is for the bold, as you will need to develop the necessary mathematical skills if you don't yet have them.

* **[WebGL2 Fundamentals](https://webgl2fundamentals.org/):**

  * _What it is:_ The definitive encyclopedia and cookbook for WebGL2.

  * _How to use it:_ Use this as a reference. When you need to implement a specific feature not covered in the current primer (like textures, instanced drawing, or shadow maps), look it up here.

* **[The Book of Shaders](https://thebookofshaders.com/):**

  * _What it is_: A legendary guide to procedural pixel art.

  * _How to use it:_ Use this to master the fragment shader. While this primer focused largely on the vertex pipeline and state machine, the Book of Shaders will teach you how to use math to paint beautiful images on the geometry you create.

* **[Learn OpenGL](https://learnopengl.com/):**

  * _What it is:_ A deep dive into modern graphics theory (written for C++).

  * _How to use it:_ Use this for theory. If you want to understand the physics behind Physically Based Rendering (PBR) or the complex mathematics of lighting models, this is the industry standard.

# Citation & license

[![License: CC BY 4.0](https://licensebuttons.net/l/by/4.0/88x31.png)](https://creativecommons.org/licenses/by/4.0/)

If you found this guide helpful and wish to reference it, please include the following citation:

> [*WebGL2 & GLSL primer: A zero-to-hero, spaced-repetition guide*](https://github.com/GregStanton/proposal-rmf-engine/blob/main/docs/webgl2-and-glsl-primer.md) by Greg Stanton (2025), licensed under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/)
