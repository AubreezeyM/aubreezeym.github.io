<!--
.. title: Staring at books and stackoverflow until things make sense.
.. slug: understanding-the-vao-vbo-and-ebo
.. date: 2025-06-14 16:53:22 UTC-00:00
.. tags: opengl,dev
.. category: opengl         
.. link: 
.. description: A look into my understanding of OpenGL's fundamentals.
.. type: text
-->

# Hi there.
It's been a couple days.  In that time, I've done a bit more research into what I've already read about.  And because of that that I've learned.... not a whole lot, but I think it will be enough to plan my project and take whoever is reading this with me!

If I'm wrong at any point, the best place to contact me would be my [BlueSky](bsky.app/profile/aubreezey.bsky.social). As a warning, I may have questions, lol.

# What are we looking at
My main area of focus was understanding the Vertex Buffer Object, the Vertex Array Object, the Element Buffer object, and the basic idea behind shaders.  While I don't think I have a *good* grasp on the concepts, I do want to share where I'm at.

## Understanding OpenGL's buffers.
I want to start with the VBO and EBO.  One thing I've heard countless times during my reading ~~plus youtube videos and stackoverflow, honestly~~ is that both of them are "stupid".  They don't care what you put in them.

That being said, the common use cases I've seen are;
* VBO's are typically used to hold vertex data such as position, color, and transparency.
* EBOs are typically used to store unique indices of each vertex to avoid duplication by OpenGL.  
* VAO's, on the other hand, store state about buffer bindings and attribute pointers.

For this example, I used a `float[]` to store my vertex position data, broken up into 9 chunks, like so.

```C++
float vertices[] = {
        -0.5f, -0.5f, 0.0f, //vertex 0
        0.5f, -0.5f, 0.0f, //vertex 1
        0.0f, 0.5f, 0.0f}; //vertex 2
```

In practice, the buffer allocation code would look something like this:

```C++
unsigned int VAO, VBO, EBO;

// genBuffers and genVertexArrays do basically what they say,
// taking a reference to the unsigned ints we defined a moment ago.
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glGenBuffers(1, s&EBO);

// Bind the Vertex Array, then provide it instructions on how to process our data.
glBindVertexArray(VAO);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);

// Attach our VBO to our VAO.
glBindBuffer(GL_ARRAY_BUFFER, VBO);

// Here we load the data into the VBO.  `vertices`, in this case,
// is a std::list<float>.  
glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(float), vertices.data(), GL_STATIC_DRAW);
glEnableVertexAttribArray(0);

while (true){
    // main loop...
}
```

I wanna take a moment to talk about `glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);`  This function in particular is very important to setup.  In provided order, the parameters are as follows;

    * `0` - The index of the vertex attribute we wanna start at.  This will later be defined in shader code.
    * `3` - The size of the vertex attribute
    * `GL_FLOAT` - specifies that we are working with floats
    * `GL_FALSE` - Disables integer normalization into floats.  OpenGLs screenspace goes from -1.0f to 1.0f on all axis (I believe).
    * '3 * sizeof(float)' - defines the stride, ie, the distance from the start of one object to another in the buffer.  Since each vertex consists of 3 floats in our example, we tell openGL that outright.
    * `(void *)0` - This calculates the offset of where the position data begins.  Since we're using a tightly packed array (meaning there's no space between our data points), we set that to 0 with a cast to `void *`.

This is required in modern OpenGL.  Without a properly setup VAO, OpenGL will refuse to initialize.

You probably noticed something missing, too.  Where's the EBO?  Well... for this project I didn't set one up yet.  Sorry!

Here's how we can set one up anyway;

```C++
glGenBuffers(1, &EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, <indices array size> * sizeof(<index datatype size>), <indices array data>, GL_STATIC_DRAW); 
```
We use `GL_ELEMENT_ARRAY_BUFFER` to signal to OpenGL that the provided buffer is meant to be an EBO.  If you remember, the VBO call used `GL_ARRAY_BUFFER`.  

We can add data to the EBO in much as the same way as the VBO, just replace any `GL_ARRAY_BUFFER` calls with `GL_ELEMENT_ARRAY_BUFFER`
## Shaders.  Once a black box now demystified
Before I share all my code there's one more topic I'd like to cover.  Shaders.

I think what finally connected the dots was understanding that in the Machine Learning/AI field, they *also* use shaders similarly to how I want to, only for a different result.

There are two main types of shaders, Vertex shaders and Fragment shaders.  I'll discuss the differences just ahead, but as for the similarities I've noticed...

All they really are are tiny bits of code written in GL Shader Language (GLSL) or something comperable.  For example, a very basic vertex shader (stolen from [Learn OpenGL](https://learnopengl.com/)) looks like this:

```GLSL
#version 400 core
layout (location = 0) in vec3 aPos;
void main()
{
 gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```
Already I'm feeling comfortable with this.

Shaders are meant to do one thing really well.  When we send them to the GPU for processing they get processed on the GPU cores.  Since typically you render well into the thousands of pixels, our GPU's small but numerous cores will handle this task much more efficiently than our CPU cores. (Especially mine, working on a quad core laptop from 2016).

### Vertex vs Fragment shaders
---
Shaders are still a topic I'm relatively new to, but my general understanding is this;

Vertex shaders determine vertex points.  Within a vertex shader you are able to essentially "add" more vertices by writing code that modifies or extends those vertices out.  The example given in the Learn OpenGL book ("the book" from now on) is drawing a second triangle while only defining one in the code.

Fragment shaders, on the other hand, usually contain data to help OpenGL determine the final pixel color.  Stuff like warmth, shadows, reflections of light, etc.  This is generally where advanced shader effects take place (according to *the book*).

My shaders currently are very basic and being stored as a C string `const char *vertexShaderSource = <shader code>`, so I don't want to share them as they're easy to find online. (Quite literally, copy and paste from any basic shader tutorial and we'd have the same shaders as of writing.)  As I start to work more on shaders, though, I'll extract them into their own files.

Now let's get into the... good part?

# The problems I'm running into
I'd like to go into a little bit of detail about this.  When I figure out the answers, I'll try to remember to come back and update this post.

Right now, my biggest hurdle is struggling to understand how to be efficient with buffer objects and vertex arrays.  The general consensus I've been seeing from the community is "Whatever's easiest for you is the right call."  Which... fair enough.  I guess I should keep studying until it's easy. 

Also, my math needs a lot of work if I'm going to be taking this seriously.  I peeked ahead and saw that I'm going to need to work with transformation matrices soon to get full 3D rendering, so my linear algebra needs some derusting.  I should also probably brush up on my geometry and trig.  I've heard that just a basic understanding of linear algebra is enough to get started, but don't have the experience to back it up.  Take that with a grain of salt. 

## Everything's coming together?  Nah, not quite.
As much as I'd love to end this point saying something like "I now understand everything about OpenGL's buffers", but the truth is I don't.  I can render a triangle as of now, multiple if I shove everything into a single `int main()`.

I'm still confused on one main thing; *How do I efficiently use and manage all these components?* 

My end goal of this is a custom built 2D game engine, a goal I've had since I started coding. But it seems my brain turns into mush whenever I try to manage GPU memory.  As such, I don't seem to get further than OpenGL initialization before I run into segfaults and crashes.  

Here's my entire `example.cpp` I promised earlier.  If it helps you, I'm glad.  As I mentioned earlier, If you notice anything funky/want to give advice, send me a message on [BlueSky](bsky.app/profile/aubreezey.bsky.social).

Thanks for reading!
-Bri

example.cpp
```C++
#define GLFW_INCLUDE_NONE
#include <GLFW/glfw3.h>
#include <glad/glad.h>

#include <stdio.h>

const char *fragmentShaderSource = "#version 330 core\n"
                                   "out vec4 FragColor;\n"
                                   "void main()\n"
                                   "{\n"
                                   "FragColor = vec4(0.11f, 0.937f, 0.969f, 1.0f);\n"
                                   "}\0";

const char *vertexShaderSource = "#version 330 core\n"
                                 "layout (location = 0) in vec3 aPos;\n"
                                 "void main()\n"
                                 "{\n"
                                 " gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
                                 "}\0";

void error_callback(int error, const char *description)
{
    fprintf(stderr, "!! OpenGL Error !! -  %s\n", description);
}

void framebuffer_size_callback(GLFWwindow *window, int width, int height)
{
    glViewport(0, 0, width, height);
}

void process_input(GLFWwindow *window)
{
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
    {
        glfwSetWindowShouldClose(window, true);
    }
}

int main()
{

    glfwSetErrorCallback(error_callback);
    if (!glfwInit())
    {
        fprintf(stderr,"OpenGl failed to init!");
        return 1;
    }

    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);
    GLFWwindow *main_window = glfwCreateWindow(640, 480, "The triangle", NULL, NULL);
    if (!main_window)
    {
        fprintf(stderr, "OpenGL Window failed to initialize!  Tried accessing a null window");
    }

    glfwMakeContextCurrent(main_window);
    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

    glViewport(0, 0, 640, 480);
    glfwSetFramebufferSizeCallback(main_window, framebuffer_size_callback);

    float triangle_vertices[] = {
        -0.5f, -0.5f, 0.0f,
        0.5f, -0.5f, 0.0f,
        0.0f, 0.5f, 0.0f};

    unsigned int VBO;
    glGenBuffers(1, &VBO);

    unsigned int VAO;
    glGenVertexArrays(1, &VAO);

    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(triangle_vertices), triangle_vertices, GL_STATIC_DRAW);

    // vertex shader
    unsigned int vertexShader;
    vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);

    int success = false;
    char vertexLogText[512];
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);

    if (!success)
    {
        glGetShaderInfoLog(vertexShader, 512, NULL, vertexLogText);
        fprintf(stderr, "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n");
    }
    else
    {
        fprintf(stdout, "Vertex Shader compiled normally!\n");
    }

    // fragment shader
    unsigned int fragmentShader;
    fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);

    char fragmentLogText[512];
    glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
    if (!success) //reuse int success from above
    {
        glGetShaderInfoLog(fragmentShader, 512, NULL, fragmentLogText);
        fprintf(stderr, "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\b");
        fprintf(stdout, fragmentLogText);
    }

    // Shader program
    unsigned int shaderProgram;
    shaderProgram = glCreateProgram();

    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);

    char programLinkLog[512];
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if (!success)
    {
        glGetProgramInfoLog(shaderProgram, 512, NULL, programLinkLog);
        fprintf(stderr, "ERROR::SHADER::PROGRAM::LINK_FAILED\n");
    }

    glUseProgram(shaderProgram);

    // Good practice to delete shaders after compiled and linked.
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    // Informing openGL how to interpret our float[] vertices
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);

    //main loop
    while (!glfwWindowShouldClose(main_window))
    {
        // process input first off, then poll events last.
        process_input(main_window);
        glClearColor(0.18f, 0.10f, 0.27f, 1.0f); 
        glClear(GL_COLOR_BUFFER_BIT); 

        // rendering code goes here
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);
        glfwSwapBuffers(main_window);
        glfwPollEvents();
    }

    // memory cleanup
    glfwDestroyWindow(main_window);
    glfwTerminate();
};
```