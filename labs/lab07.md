---
layout: default
title: "Lab 7: Graphics Pipeline and Basic Shaders"
---

All vertices passed into the graphics card are processed by a multi-stage *graphics pipeline*. The graphics pipeline is optimized to process *graphics* (as opposed to a more general purpose computing pipeline such as on the CPU) which provides a great deal of efficiency and program simplicity at the expense of a limited number of operations. As a graphics programmer, it will be your task to utilize the pipeline to do as much work as possible on the graphics card rather than on the CPU. The stages of this pipeline are shown in the figure below:

> <img src="images/lab07/RenderingPipeline.png" alt="Rendering Pipeline" height="200"/>

[https://www.khronos.org/opengl/wiki/Rendering\_Pipeline\_Overview](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview). **Note:** The stages shaded in blue are *programmable*, i.e. we can specify their behavior through shader code.

The two stages we will be considering in this course are:

-   **Vertex Shader** - performs coordinate transformations (including projection and camera transformations) and assigns vertex colors
-   **Fragment Shader** - applies pixel level effects such as texture mapping, per-pixel lighting, bump mapping, etc.

In this lab we will see how to load shaders from files and *use* them from within our programs by associating shader variables with references in our application.

## Getting Started

Download [CS370\_Lab07.zip](src/CS370_Lab07.zip), saving it into the **CS370\_Fa26** directory.

Double-click on **CS370\_Lab07.zip** and extract the contents of the archive into a subdirectory called **CS370\_Lab07**

Open CLion, select **CS370\_Fa26** from the main screen (you may need to close any open projects), and open the **CMakeLists.txt** file in this directory (**not** the one in the **CS370\_Lab07** subdirectory). Uncomment the line

```cpp
	add_subdirectory("CS370_Lab07" "CS370_Lab07/bin")
```

Finally, select **Reload changes** which should build the project and add it to the dropdown menu at the top of the IDE window.

#### Solution

Download [CS370\_Lab07\_Solution.zip](sol/CS370_Lab07_Solution.zip), saving it into the **CS370\_Fa26** directory.

Double-click on **CS370\_Lab07\_Solution.zip** and extract the contents of the archive into a subdirectory called **CS370\_Lab07\_Solution**

Open CLion, select **CS370\_Fa26** from the main screen (you may need to close any open projects), and open the **CMakeLists.txt** file in this directory (**not** the one in the **CS370\_Lab07\_Solution** subdirectory). Uncomment the line

```cpp
	add_subdirectory("CS370_Lab07_Solution" "CS370_Lab07_Solution/bin")
```

Finally, select **Reload changes** which should build the project and add it to the 

## Loading Shaders

Shaders, like other source code files, are simply text files with a syntax similar to C. In order to create them, the **shaderutils.cpp** code in the **common** directory contains a utility function we will use:

```cpp
GLuint LoadShaders(ShaderInfo* shaders);
```

where *shaders* is an array of type **ShaderInfo** which specifies which shader file to associate with which shader stage and returns a reference to a shader program built from these stages. For example, if we have two variables *color\_vertex\_shader* and *color\_frag\_shader* that contain the paths to shader source files, we can build a shader program using

```cpp
ShaderInfo default_shaders[] = { {GL_VERTEX_SHADER, color_vertex_shader},
                         {GL_FRAGMENT_SHADER, color_frag_shader},
                         {GL_NONE, NULL} };
default_program = LoadShaders(default_shaders);
```

**Note:** To create a shader *program* we need a vertex shader and *compatible* fragment shader.

### Creating Shader Programs

The **LoadShader()** function performs several steps using the shader files specified in the parameter. First, we need to read the shader text from the file and store it in a character array. **shaderutils.cpp** provides a **ReadShader()** function to perform this task. Next we will create various references to the program and the separate shaders using

```cpp
GLuint glCreateProgram();
GLUint glCreateShader(GLenum shaderType);
```

where *shaderType* is the type of shader stage we are creating, e.g. **GL\_VERTEX\_SHADER**.

Next we will associate the shader source with the shader reference using

```cpp
void glShaderSource(GLuint shader, GLsizei count, const GLchar **string, const GLint *length);
```

where *shader* is the reference returned from **glCreateShader()**, *count* is the number of shaders we are associating, *string* is an array of source strings, and *length* is an array containing the string lengths.

Once the source is associated with the reference, we will *compile* each shader stage using

```cpp
void glCompileShader(GLuint shader);
```

where *shader* is the shader reference. We can check whether or not the compilation was successful using

```cpp
void glGetShaderiv(GLuint shader, GLenum pname, GLint *params);
```

where *shader* is the shader reference for the shader we attempted to compile, *pname* is the parameter we wish to query (e.g. **GL\_COMPILE_STATUS**), and \**params* is a reference to the return variable for the resulting query. Thus we can verify that the shader stage was compiled correctly, or retrieve some error log information if not.

If compilation was successful, we can then attach the shader to our program using

```
void glAttachShader(GLuint program, GLuint shader);
```

where *program* is the shader program reference and *shader* is the shader stage reference.

After we have compiled and attached all our stages, the last step is to *link* the stages to create the shader program using

```cpp
void glLinkProgram(GLuint program);
```

where *program* is the shader program reference. Similar to compilation, we can query if linking was successful using

```cpp
void glGetProgramiv(GLuint program, GLenum pname, GLint *params);
```

where *program* is the program reference, *pname* is the parameter we wish to query (e.g. **GL\_LINK\_STATUS**, and \**params* is a reference to the return variable for the resulting query. Thus we can verify that the program was linked correctly or retrieve some error log information if not.

Thus in our application we can create multiple shader programs using different vertex and fragment shader files if we wish to use different effects on different objects in our scene.

### Tasks

**NOTE:** The global variables **color\_vertex\_shader**, **color\_frag\_shader**, and **dim\_vertex\_shader** defined in **shaders.h** contain the filenames for the shader files. We will be using the *same* fragment shader with two different vertex shaders to create two shader programs.

- Add code in **shaders.h** to **build\_shaders()** to create the shader program from the *color\_shaders[]* **ShaderInfo** array and store the resulting reference in *color\_program*. This shader will be used to render objects loaded from **.obj** files using a color buffer as in the previous lab.

- Add code in **shaders.h** to **build\_shaders()** to create the shader program from the *dim\_shaders[]* **ShaderInfo** array and store the resulting reference in *dim\_program*. This shader will take additional information to dynamically adjust (*dim*) the colors of the object.

You may wish to look at the code in **shaderutils.cpp** to see how the **ShaderInfo** array is used to compile and link the various shader parts into a shader program.

## Shader Variable References

Once we have created a shader program, we need to associate references in our application with the input shader variables to allow us to set the values of these shader variables during rendering. There are several types of shader variables:

- **uniform** - values that are set once *per object* (i.e. are the same for all vertices), for example a transformation matrix

- **attribute** - values that are set once *per vertex* (i.e. they may be different for each vertex), for example a color or lighting normals

Thus to associate a reference in the application we will use either

```cpp
GLint glGetUniformLocation(GLuint program, const GLchar *name);
GLint glGetAttribLocation(GLuint program, const GLchar *name);
```

where *program* is the shader program and \**name* is a string with the *name* of the shader variable in the shader source file.

### Tasks

**NOTE:** The shader variable are defined in **color.vert** (located in the **common/shaders** directory) and **dim.vert**. Even though many of the shader variables have the same names, they are separate variables since they will be in separate shader programs.

- Add code in **shaders.h** to **build\_shaders()** to set *color\_vPos* reference to the corresponding *vPosition* shader variable and *color\_vCol* reference to the corresponding *vColor* shader variable. **Note:** These variables are *attributes* from the *default\_program*.

- Add code in **shaders.h** to **build\_shaders()** to set the *color\_proj\_mat\_loc*, *color\_cam\_mat\_loc*, and *color\_model\_mat\_loc* references to the corresponding shader variables. **Note:** These variables are **uniforms** from the *default\_program*. **Hint:** Look in the **default.vert** for the names of the shader variables.

- Add code in **shaders.h** to **build\_shaders()** to set the *dim\_vPos* and *dim\_vCol* references to the corresponding *vPosition* and *vColor* shader variables. **Note:** These variables are *attributes* from the *dim\_program*.

- Add code in **shaders.h** to **build\_shaders()** to set the *dim\_proj\_mat\_loc*, *dim\_cam\_mat\_loc*, and *dim\_model\_mat\_loc* references to the corresponding shader variables. **Note:** These variables are **uniforms** from the *dim\_program*. **Hint:** Look in the **dim.vert** for the names of the shader variables.

- Add code in **shaders.h** to **build\_shaders()** to set the *dim\_factor\_loc* reference to the corresponding shader variable. **Note:** This variable is a **uniform** from the *dim\_program*. **Hint:** Look in the **dim.vert** for the name of the shader variable.

## Activating Shaders

In order to keep **render\_scene()** generic, we will select the appropriate shader in the various **draw\_\*** functions. This will allow us to render different types of objects in any order we desire.

Prior to rendering an object, we need to activate the shader we wish to use for rendering using

```cpp
void glUseProgram(GLuint program);
```

where *program* is the shader program to use.

### Tasks

- Add code in **drawObjects.h** to **draw\_color\_object()** to use the *color\_program*.

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to use the *dim\_program*.

## Passing Data to Shader Variables

Once we have references to the shader variables and selected the shader program we will be using, we can use the references for the program to pass the appropriate information from our application when we render the objects. 

### Attribute Variables

Since attribute variables are set *per vertex* they are passed via the buffer objects. Thus after we bind the proper vertex array object and vertex buffer, we need to define the layout of each attribute component using

```cpp
void glVertexAttribPointer(Gluint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const void *pointer);
```

where *index* is the shader variable reference, *size* is the number of coordinates in the attribute, *type* is the datatype for the elements, *normalized* indicates whether to normalize the values in the range [-1,1], *stride* is the offset between consecutive data in the buffer, and \**pointer* is the offset of the starting location for the data (despite its awkward signature).

We will enable these attributes using

```cpp
void glEnableVertexAttribArray(GLuint index);
```

where *index* is the reference to the corresponding shader variable location.

### Uniform Variables

We can set the value of *uniform* **vec4** *vector* shader variables from application variables using

```cpp
void glUniform4fv(GLint location, GLsizei count, const GLfloat *value);
```

where *location* is the shader variable reference, *count* is the number of shader variables we wish to set (1 for a single vector and more than 1 if the shader variable is an array of vectors), and \**value* is the application variable containing the vector data to be passed to the shader. **Note:** There are several other versions depending on the number of components to pass and the data type of the elements, see the [API](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glUniform.xhtml).

For **mat4** *matrix* shader variables, we can use

```cpp
void glUniformMatrix4fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value);
```

where *location* is the shader variable reference, *count* is the number of shader variables we wish to set (1 for a single matrix and more than 1 if the shader variable is an array of matrices), *transpose* is a flag specifying whether or not to transpose the matrix when it is passed, and \**value* is the application variable containing the matrix data to be passed to the shader. **Note:** There are several other versions depending on the size of the matrix to pass and the data type of the elements, see the [API](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glUniform.xhtml).

### Tasks

- Add code in **drawObjects.h** to **draw\_color\_object()** to pass *proj\_matrix* and *camera\_matrix* uniform matrices to the shader using their respective *color\_program* shader references. We are only passing 1 matrix to each shader variable and do not need to transpose them (i.e. **GL\_FALSE**).

- Add code in **drawObjects.h** to **draw\_color\_object()** to pass the *model\_matrix* uniform matrix to the shader using the respective *color\_program* shader reference. We are only passing 1 matrix to the shader variable and do not need to transpose it (i.e. **GL\_FALSE**).

- Add code in **drawObjects.h** to **draw\_color\_object()** to bind the *PosBuffer* element from the **ObjBuffers[]** array for the *obj* index.

- Add code in **drawObjects.h** to **draw\_color\_object()** to define and enable the vertex attribute *color\_vPos* after the *PosBuffer* has been bound. Use *posCoords* for the number of components in the position, **GL\_FLOAT** for the type, **GL\_FALSE** for normalization, 0 for the stride, and **NULL** for the offset.

- Add code in **drawObjects.h** to **draw\_color\_object()** to bind the *color* element from the **ColorBuffers[]** array.

- Add code in **drawObjects.h** to **draw\_color\_object()** to define and enable the vertex attribute *color\_vCol* after the *color* has been bound. Use *colCoords* for the number of components in the position, **GL\_FLOAT** for the type, **GL\_FALSE** for normalization, 0 for the stride, and **NULL** for the offset.

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to pass *proj\_matrix* and *camera\_matrix* uniform matrices to the shader using their respective *dim\_program* shader references. We are only passing 1 matrix to each shader variable and do not need to transpose them (i.e. **GL\_FALSE**).

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to pass the *model\_matrix* uniform matrix to the shader using the respective *dim\_program* shader reference. We are only passing 1 matrix to the shader variable and do not need to transpose it (i.e. **GL\_FALSE**).

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to bind the *PosBuffer* element from the **ObjBuffers[]** array for the *obj* index.

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to define and enable the vertex attribute *dim\_vPos* after the *PosBuffer* has been bound. Use *posCoords* for the number of components in the position, **GL\_FLOAT** for the type, **GL\_FALSE** for normalization, 0 for the stride, and **NULL** for the offset.

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to bind the *color* element from the **ColorBuffers[]** array.

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to define and enable the vertex attribute *dim\_vCol* after the *color* has been bound. Use *colCoords* for the number of components in the position, **GL\_FLOAT** for the type, **GL\_FALSE** for normalization, 0 for the stride, and **NULL** for the offset.

- Add code in **drawObjects.h** to **draw\_dim\_color\_object()** to pass the uniform *dim* parameter value using the *dim\_factor\_loc* reference as a single float value.

## Compiling and running the program

You should be able to build and run the program by selecting **shaderScene** from the dropdown menu and clicking the small green arrow towards the right of the top toolbar.

At this point you should see a gradient cube with a solid sphere on top whose color dims sinusoidally with time that can be adjusted with a spherical coordinate perspective camera.

> <img src="images/lab07/ShaderScene.png" alt="Shader Scene Window" height="500"/>

To quit the program simply close the window.

Congratulations, you have now used multiple shaders to pass different object data for rendering.

Next we will discuss the basics of how to *write* our own shaders.
