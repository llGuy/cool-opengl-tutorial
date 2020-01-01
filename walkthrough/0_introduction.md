# Introduction to OpenGL

![photo](/walkthrough/0_introduction/images/opengl_logo.png)

Hello!

I wanted to make this cool and simple little OpenGL tutorial to possible demystify this API through a simple walkthrough on how to draw a triangle and then so on...

## Introduction to the GPU 

OpenGL is a 3D graphics API, allowing the programmer to interface with graphics hardware (such as the GPU).

The great thing about the GPU for graphics (and  therefore making games), is that it allows us to do a lot of things in parallel - this will become very obvious in just a second.

To render (draw) a shape to the screen, we usually use triangles as this allows us to create a huge number of shapes (square, pentagon, any 3D model...)

![photo](/walkthrough/0_introduction/images/sphere.png)

The way triangles are sent to the GPU, is through a *graphics pipeline*. In OpenGL, this is called a *GPU program*.

This graphics pipeline consists of a list of instructions (stages) that need to be executed one by one in order to render something to the screen. There are many stages that are already configured by default by OpenGL that we might delve into later on, but there are two main ones that we need to know about for now that we will have to configure manually for OpenGL: the *vertex shader* and the *fragment shader*. These shaders are programs that are directly loaded onto the GPU and are run in the GPU.

These programs are similar to C programs in the way they are used / created.

We write these programs in human-readable code (the language is called GLSL - similar to C in syntax), then there is a compiler that turns this code into some binary format that the GPU understands.

Much like Assembly (which is an intermediate for CPU binary instructions), SPIR-V is the intermediate language for GPU binary instructions.



## Shaders and Rendering

The two main shaders that you need to know for now are: the *vertex* shader (responsible for setting the positions of the vertices) and the *fragment* shader (responsible for setting the colors of the pixels) - essential and necessary stages of the *graphics pipeline* which is represented by an OpenGL *GPU program*. 

> NOTE: there are other shaders which constitute different graphics pipeline stages like the geometry shader and tesselation shaders, however, we don't need to configure them right now - they will be configured by OpenGL by default.

We therefore need to provide OpenGL with the GLSL source code of these 2 shaders to create an OpenGL *GPU program*. 

In a nutshell, the way rendering works, is that you tell OpenGL to run the GPU program that you want. When this happens, the graphics pipeline stages get executed one by one. In the first stage of the *graphics pipeline* (vertex shader), OpenGL creates a sort of loop that runs for the specified vertex count (if we want 3 vertices for a triangle, we tell OpenGL to loop 3 times).

For each iteration of the loop, OpenGL is going to run the *vertex shader* which will set the position of the current vertex. Once all the iterations finish, all the vertices will be set, and we are ready to go to the next stage (fragment shader).

Now, OpenGL runs the fragment shader for every pixel inside the shape and sets the color of the pixel.

Here's an example:

![photo](/walkthrough/0_introduction/images/vertex_shader_visualisation.PNG)

Here, we tell OpenGL to run the vertex shader 3 times, and plot the 3 points for the triangle that will be drawn accordingly.

![photo](/walkthrough/0_introduction/images/fragment_shader_visualisation.PNG)

Next, the fragment shader gets run for every pixel and colors them in (in the way that the fragment shader was programmed to do).



## Translation to code

Here's how this translates to OpenGL code (I expect that you initialized the OpenGL context either with GLFW/GLEW or Win32, or other APIs).

First of all, the way a typical graphics program is setup, is with some sort of initialization function (which creates all the shaders, loads models, etc...), then the update function (which will run every frame until the game is over, and update the game and render to the screen). We will call these two functions accordingly:

```c
void initialize_game()
{
	// Initialization
}

void update()
{
	// Update and render
}
```

Right now, we are taking care of the initialization function.

### Initialization

To initialize the GPU program, we first need to obtain two strings, containing the vertex shader code in GLSL and the fragment shader code in GLSL respectively.

```c
const char *vertex_shader_code = "[...]"; // We will write these shaders later
const char *fragment_shader_code = "[...]";

unsigned int vertex_shader_code_length = strlen(vertex_shader_code);
unsigned int fragment_shader_code_length = strlen(fragment_shader_code);
```

We feed this code to OpenGL and create a GPU program object. This object, as mentioned before, ties together all the stages (vertex, fragment, ...) into one single object. This is so that we can execute the entire graphics pipeline in just one function call (it will execute the vertex shader, fragment shader, and other stages in order).

```c
// NOTE: OpenGL objects (shaders, programs, ...) are represented with IDs (numbers). To access and modify these objects, we have to use their corresponding IDs 

// Create the GPU program (glCreateProgram returns a number - ID, that corresponds to the actual program)
unsigned int gpu_program = glCreateProgram();

// Create the vertex shader stage and the fragment shader stage with the GLSL source code
// vsh and fsh are numbers corresponding to shader objects
unsigned int vsh = glCreateShader(GL_VERTEX_SHADER);
unsigned int fsh = glCreateShader(GL_FRAGMENT_SHADER);

// Provide OpenGL shaders with the source we wrote
glShaderSource(vsh, 1, &vertex_shader_code, &vertex_shader_code_length);
glShaderSource(fsh, 1, &fragment_shader_code, &fragment_shader_code_length);

// Compile the GLSL shaders into GPU binary format
glCompileShader(vsh);
glCompileShader(fsh);

// Provide these shaders to the GPU program (unsigned int gpu_program)
glAttachShader(gpu_program, vsh);
glAttachShader(gpu_program, fsh);

// Link the program (kind of like a C++ program)
glLinkProgram(gpu_program);

// Detach the shaders (don't need them anymore)
glDetachShader(gpu_program, vsh);
glDetachShader(gpu_program, fsh);
```

### Update function
In the `update()` function, we need to render (draw) our triangle.

The first thing that needs to happen when rendering is to clear the buffer of memory with the color values of the pixels of our window with `glClear(GL_COLOR_BUFFER_BIT)` - with GL_COLOR_BUFFER_BIT, we are telling OpenGL to specifically clear the color values of the screen (there will be others we will get to later like GL_DEPTH_BUFFER_BIT). The reason for why we would want to do this is so that we can render on a "blank canvas" so to speak - so that we are not rendering on an already drawn image.

Now we want to render the triangle.

To do this, we have to run the GPU program we created, which will kick off the vertex shader loop created by OpenGL, mentioned earlier, which runs the vertex shader for however many vertices we want, and will then execute the rest of the GPU program.

However, OpenGL needs to know which GPU program to use.

To tell OpenGL that we want to use the program we just created, write this:

```c
glUseProgram(gpu_program);
```

To run the graphics pipeline (and therefore start this vertex shader loop), we call `glDrawArrays`:

```c
glDrawArrays(GL_TRIANGLES /* GLenum primitive */, 0 /* unsigned int starting_index */, 3 /* unsigned int finishing_index */);
```

The way we can visualise this function is with this pseudo-code which we will be developing over the course of the tutorial:

```c
// Vertex shader stage:
for (unsigned int gl_VertexID = starting_index; gl_VertexID < finishing_index; ++gl_VertexID)
{
    call_currently_bound_vertex_shader(gl_VertexID);
}

// Fragment shader stage
foreach (pixel : pixels_to_be_shaded)
{
	call_currently_bound_fragment_shader_on_pixel(pixel);
}

// Next stage, etc...
```

After the vertex shader loop happens, the fragment shader gets run for every pixel and sets the pixel color.

The primitive parameter of `glDrawArrays()` basically just tells the GPU, how to determine which pixels to call the fragment shader on.

For example, `GL_TRIANGLES` will tell the GPU to call the fragment shader for all the pixels delimited by the vertices specified by the vertex shader. Here is what it looks like:

![photo](/walkthrough/0_introduction/images/gl_triangles.PNG)

Another example: `GL_LINE_LOOP` tells the GPU to call the fragment shader only the lines connecting the vertices. Here is what it looks like:

![photo](/walkthrough/0_introduction/images/gl_line_loop.PNG)

After rendering is finished, you need to swap buffers (with GLFW, it's `glfwSwapBuffers()`).

> NOTE: When rendering, we often use a method called double-buffering in which the computer actually allocates 2 buffers of memory containing pixel color values. While one is being displayed on the screen, we are rendering to the other one - once rendering is finished, we swap the buffers.

## Writing the vertex shader and fragment shader

### Vertex shader

The way that the vertex shader works, is similar to a C program in its syntax and entry point signature (when OpenGL runs a shader, it calls the main() function first):

```c
// We need to set the GLSL language version like so:
#version 330 core

void main()
{
    // Code
}
```

Just like a C program, the main function serves as the entry point into the shader.

However, before the vertex shader exits, there is a special global GLSL variable that needs to be set: `gl_Position`. This variable will basically set the position of the vertex. It is of type `vec4` - a 4 component vector. The reason for why it is a four component vector will make sense once we get to 3D graphics. For now, in 2D, just set the 4th coordinate (w) to 1, and the 3rd component (z) to 0.

```c
void main()
{
	// We will fill in x and y in a second
    gl_Position = vec4(x, y, 0, 1);
}
```

Now, how does the coordinate system of OpenGL work:

![photo](/walkthrough/0_introduction/images/opengl_coordinates.png)

No matter the window resolution, the x axis goes from -1 to +1 and the y axis goes also from -1 to +1.

So, here are the vertices I want to set:

```c
vec2 position0 = vec2(-0.5, -0.5);
vec2 position1 = vec2(0.0, 0.5);
vec2 position2 = vec2(0.5, -0.5);
```

Notice, that when I wrote the the pseudo-code (for the OpenGL loop), I wrote the the iterating variable was gl_VertexID (readonly). This is once again another global variable created by OpenGL. It will be set internally by OpenGL for every vertex shader before it runs and it corresponds to the index of the vertex currently being operated on. So, if in the vertex shader, we create an array of the vertices we want to set (in an array `vertices`), and set gl_Position's x and y component to the x and y components of `vertices[gl_VertexID]`, we would get a triangle!

Here's the code:

```c
#version 330 core

vec2 vertices[3] = vec2[3]( vec2(-0.5, -0.5), vec2(0.0, 0.5), vec2(0.5, -0.5) );

void main()
{
    gl_Position = vec4(vertices[gl_VertexID].x, vertices[gl_VertexID].y, 0, 1);
}
```

### Fragment shader

The fragment shader is similar to the vertex shader, in that before the program exits, a global variable (corresponding to the color of the pixel) needs to be set. However, this variable is not a global special GLSL variable (like `gl_Position`). It is an `out` variable. In GLSL, there are these special variables that can be created: `in` and `out`. An `out` variable (for example `out vec2 position;`) will be passed from one stage to the next (the next stage needs a corresponding `in` variable - for example: `in vec2 position;`). The reason why this is useful, is that often stages need to use data that has been calculated from the previous stage (color, normals, uv coordinates - which we will get to later).

The fragment shader needs to declare an `out` variable, corresponding to the color (also vec4 - r, g, b, a) and set it. This variable will be used by the next stage of the gpu program. This (final) stage is an internal stage, configured by default by OpenGL, that does some calculations on the `out` variable that you provided. For example, it might do some alpha blending (transparency stuff) - it's basically just some final calculations that need to be done to the color. 

This is what the code looks like:

```c
#version 330 core

out vec4 final_color; // Name can be whatever you want

void main()
{
    final_color = vec4(1, 0, 0, 1); // Red
}
```

Here, there is no `in` variable. This is because, the fragment shader here does not need to use any data that has been calculated by the previous stage (the vertex shader). In this case, we are using a constant color that does not depend on the vertex - so we don't need any `out` and `in` between the vertex and fragment shaders.

And voila! working triangle.

![photo](/walkthrough/0_introduction/images/gl_triangles.PNG)

