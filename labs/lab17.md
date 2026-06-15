---
layout: default
title: "Lab 17: Multi Texturing"
---

While texture mapping is a nice way of simulating surface textures without additional geometry, we can further embellish the appearance by blending *multiple textures* together on the same surface using *multi-texturing*. In this lab we will see how to use multiple texture *units* to sample multiple textures in the fragment shader. 

## Getting Started

Download [CS370\_Lab17.zip](src/CS370_Lab17.zip), saving it into the **CS370\_Fa26** directory.

Double-click on **CS370\_Lab17.zip** and extract the contents of the archive into a subdirectory called **CS370\_Lab17**

Open CLion, select **CS370\_Fa26** from the main screen (you may need to close any open projects), and open the **CMakeLists.txt** file in this directory (**not** the one in the **CS370\_Lab17** subdirectory). Uncomment the line

```cpp
	add_subdirectory("CS370_Lab17" "CS370_Lab17/bin")
```

Finally, select **Reload changes** which should build the project and add it to the dropdown menu at the top of the IDE window.

#### Solution

Download [CS370\_Lab17\_Solution.zip](sol/CS370_Lab17_Solution.zip), saving it into the **CS370\_Fa26** directory.

Double-click on **CS370\_Lab17\_Solution.zip** and extract the contents of the archive into a subdirectory called **CS370\_Lab17\_Solution**

Open CLion, select **CS370\_Fa26** from the main screen (you may need to close any open projects), and open the **CMakeLists.txt** file in this directory (**not** the one in the **CS370\_Lab17\_Solution** subdirectory). Uncomment the line

```cpp
	add_subdirectory("CS370_Lab17_Solution" "CS370_Lab17_Solution/bin")
```

Finally, select **Reload changes** which should build the project and add it to the dropdown menu at the top of the IDE window.

## Texture Map references

Since we are using several textures, we will need to have several sampler shader variables. We do by declaring multiple **uniform sampler2D** variables in the *fragment shader*. Then we will associate these sampler variables with a application reference identifiers using

```cpp
GLint glGetUniformLocation(GLuint program, const GLchar *name);
```

where *program* is the shader program and *\*name* is a string with the name of the shader sampler variable.

### Tasks

- Add code in **shaders.h** in **build\_shaders()** to add a second reference assignment for **multi\_tex\_blend\_loc** for the **blendMap** shader variable which will be the sampler for the second (dirt) texture.

- Add code in **shaders.h** in **build\_shaders()** to add a reference assignment for **multi\_tex\_mix_loc** for the **mixFactor** shader variable which will control how much of each texture map is used. **Note:** This reference is also for a *uniform* location variable.

### Multiple Texture Units

In order to use multiple textures, we will need to utilize multiple *texture units* by defining which texture unit to associate with which shader sampler, and then *bind* the textures we wish to use to each unit. OpenGL supports a minimum of 16 texture units per pipeline stage which are referred to using the constants **GL_TEXTURE\#** where \# is the number of the texture unit, e.g. **GL_TEXTURE0**. Thus we will specify which texture unit to associate with a shader sampler using

```cpp
void glUniform1i(GLint location, GLint value);
```

where *location* is the reference for the shader sampler and *value* is the unit number we wish to associate with the sampler. Then we will make the texture unit *active* using 

```cpp
void glActiveTexture(GLenum tex_unit);
```

where *tex\_unit* is a symbolic constant of the form **GL\_TEXTURE***i* where *i* is the number of the texture unit we wish to make active, e.g. **GL\_TEXTURE0**.

Finally, we will bind the desired texture to the unit as usual using

```cpp
void glBindTexture(GLenum target, GLuint texture);
```

where *target* is a symbolic constant denoting the *type* of texture we are binding (e.g. for image data **GL\_TEXTURE\_2D**) and *texture* is the texture id for the texture we are binding to the currently active texture unit.

### Tasks

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to set the *multi\_tex\_base\_loc* to 0, i.e. we will bind the base texture to texture unit 0, using **glUniform1i()** (since this is an integer value).

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to make texture unit 0 active using **glActiveTexture()** with **GL_TEXTURE0**.

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to bind the *baseID* texture enum index variable parameter from the *TextureIDs[]* array to the active texture unit (texture unit 0).

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to set the *multi\_tex\_blend\_loc* to 1, i.e. we will bind the dirt texture to texture unit 1, using **glUniform1i()** (since this is an integer value).

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to make texture unit 1 active using **glActiveTexture()** with **GL_TEXTURE1**.

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to bind the *blendID* texture enum index variable parameter from the *TextureIDs[]* array to the active texture unit (texture unit 1).

**Note:** Since our shader will be combining colors from both textures, it is important that they each be associated with a texture unit containing an appropriate texture.

## Multiple Texture Coordinates

For this lab we will simply be using the same texture coordinates for both texture maps. However, if the model/loader supported multiple texture coordinates, we could store them in separate texture coordinate buffers and pass them through the vertex shader to the fragment shader.

## Combining Texture Colors

The last step to applying multiple textures is sampling both textures in the fragment shader and deciding how to *combine* the two colors together for the final fragment color. 

Several options can include simple addition of the two colors, simple multiplication of the two colors, using the **mix()** function to perform a linear interpolation (using an application variable), or some other combination of the two.

### Tasks

- Add code in **drawObjects.h** to **draw\_multi\_tex\_object()** to set the *multi\_tex\_mix\_loc* to *mix* using **glUniform1f()** (since this is a floating point value).

- Add code to **multiTex.frag** to sample *blendMap* at *texCoord* and store the result in *blendColor*

```cpp
    vec4 blendColor = texture(blendMap, texCoord);
```

- Add code to **multiTex.frag** to mix the two texture colors using **mix()** with the *mixFactor*

```cpp
    fragColor = mix(baseColor,blendColor,mixFactor);
```

**Note:** Try other ways of combining the two texture colors to see what effect it produces.

## Rendering Multi-texture Objects

Finally, to render an object with multi-texturing, we can use the **draw\_multi\_tex\_object()** function

```cpp
void draw_mulit_tex_object(GLuint obj, GLuint baseID, GLuint blendID, GLfloat mix);
```

where *obj* is the enum constant for the object to render, **baseID** is the base texture enum constant **blendID** is the second texture enum constant, and **mix** is a floating point value to determine the proportion of each texture to use (we are using the linear interpolation **mix()** shader function).

### Tasks

- Add code to **multiTexMesh.cpp** in **render\_scene()** to draw the **Sphere** using multi-texturing with the *Carpet* texture enum constant for the base texture and the **multiTexID** texture variable (which is assigned in the logic) for the second texture.

## Compiling and running the program

You should be able to build and run the program by selecting **multiTexMesh** from the dropdown menu and clicking the small green arrow towards the right of the top toolbar.

At this point you should see a torus and revolving sphere over a carpet that is mixed with either a blank or "dirt" texture (toggled using enter). Arrow up/down will control the amount of the second texture that is mixed and \<spacebar\> will toggle the animation.

> <img src="images/lab17/multiMesh.png" alt="MultiTexture Mesh Window" height="500"/>

To quit the program simply close the window.

Congratulations, you have now written an application incorporating multiple textures.

Next we will investigate how to apply multiple textures to accomplish bump mapping.

### Shadows with Textures

With multitexturing, it is possible to combining shadow mapping with textures (since the shadow map is stored in a texture). Here is a sample solution

[CS370\_Lab17\_ShadowTexture\_Solution.zip](sol/CS370_Lab17_ShadowTexture_Solution.zip)

You will need to uncomment the following line in your **CMakeLists.txt** file in the **CS370\_Fa26** directory (**not** the one in the **CS370\_Lab17\_ShadowTexture\_Solution** subdirectory).

```cpp
	add_subdirectory("CS370_Lab17_ShadowTexture_Solution" "CS370_Lab17_ShadowTexture_Solution/bin")
```

> <img src="images/lab17/shadowTextureMesh.png" alt="Shadow Texture Mesh Window" height="500"/>

