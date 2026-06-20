---
layout: default
title: "Assignment 4: Walking Man"
---

**Written Questions Due: Tuesday, Nov 18th by 2:00 PM** (in class). Submit a **graded** pdf to Canvas by Thursday, Nov 20th.

**Program Due:**

**Milestone 1: Monday, Nov 10th by 11:59 PM** 

**Milestone 2: Wednesday, Nov 19th by 11:59 PM** Late assignments will be penalized 20 points per day.

## Getting Started

Download [CS370\_Assign04\_Fa26.zip](src/CS370_Assign04_Fa26.zip), saving it into the **CS370_Fa26** directory.

Double-click on **CS370\_Assign04\_Fa26.zip** and extract the contents of the archive into a subdirectory called **CS370\_Assign04\_Fa26**

Open CLion, select **CS370_Fa26** from the main screen (you may need to close any open projects), and open the **CMakeLists.txt** file in this directory (**not** the one in the **CS370\_Assign04\_Fa26** subdirectory). Uncomment the line

```cpp
	add_subdirectory("CS370_Assign04_Fa26" "CS370_Assign04_Fa26/bin")
```

Finally, select **Reload changes** which should build the project and add **WalkingMan** to the dropdown menu at the top of the IDE window.

## Written Questions

1.  Using the example robot from [lab 14](../labs/lab14.html) suppose we know the *position* where one of the upper arms is starting and have a specific final *location* we want the arm to have. Give a method to create the path the robot should follow to get from the starting point to the end point.

2.  Using the example scene graph diagram from [lab 14](../labs/lab14.html), sketch the scene graph for the programming portion of the assignment. **Note:** You only need to show the relationships between the nodes, i.e. it is not necessary to include transformations.

3.  Given the following texture map

    > <img src="images/assign04/texture.png" alt="Texture Map" height="300"/>
    >
    > with wrapping modes
    >
    >     glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP);
    >     glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    >
    > sketch the surrounding parts of the texture plane.

4.   Using the texture plane from question 3, sketch the textured figure given below using the provided texture coordinates

> <img src="images/assign04/textureObj.png" alt="Texture Object" height="300"/>

### Hints

> 1.  This problem is known as *reverse kinematics* and is a key area of robotics research. Representing the individual transformations symbolically as **T**(*dx*, *dy*, *dz*), **R**(*ang*), and **S**(*sx*, *sy*, *sz*), first determine the *net* transformation matrix for the upper arm in terms of the base angle θ, lower arm angle φ, and upper arm angle ψ along with any translations for each part. Then let the initial base and joint angles be θ<sub>i</sub>, φ<sub>i</sub>, ψ<sub>i</sub> and final base and joint angles be θ<sub>f</sub>, φ<sub>f</sub>, ψ<sub>f</sub>. These angles would be found by *solving* the transformation matrix equations for the angles (which you may assume can be done to compute all these angles). Using these sets of angles describing the location of the two points, suggest an interpolation scheme to *smoothly* transition from one to the other.
> 2.  Consider which nodes are relative to other nodes (*children*) and which ones are independent (*siblings*). 
> 3.  Note that the texture plane actually extends infinitely in both directions.
> 4.  Mark the texture coordinates on the texture plane, then "cut" this section out and "stretch" it to fit on the object. 

### Programming assignment

Write a program that draws a 3D scene of a walking player with articulated arms and legs along with a bouncing basketball into a translucent box sitting on a court. A sample executable is included in the **demo** directory as either **WalkingManSolWin.exe** or **WalkingManSolMac**. Keyboard controls are provided that allow an orthographic camera to be rotated using WASD. The scene should include:

-   A background image of your choosing (or you may use the included **ycp.png** file). **Note:** You will need to add texture coordinates for each vertex of the background quad for the *uvCoords* vector in **build\_background()**.
-   A *scene graph* to render the player, basketball, court, and box. Create materials to enhance the scene appearance.
-   The player should consist of

    > -   Rectangular torso with a shirt
    > -   Elliptical head with a face
    > -   Rectangular arms and legs with two segments that have lighting with different materials

-   Scale factors for all the player parts are included in **Player.h**.
-   All the parts of the player other than the torso and head should use materials and lighting.
-   The torso of the player should use the *TexCube* object (**not** the *Cube* object) which will be texture mapped using the **shirt\_z.png** texture. You will need to add texture coordinates for each face in the *uvCoords* vector in **build\_texture\_cube()**. Note that the texture map contains labelled pieces for the front, back, left, and right faces of the cube (you may use any parts of the texture map for the top/bottom faces). The pixel locations for the divisions between the segments is shown below:

    > <img src="images/assign04/shirt_tex_layout.png" alt="Shirt Texture Layout" height="350"/>

-   The box should be translucent such that the basketball can be seen inside it and the court/player can be seen through it.
-   The player should "walk" by moving arms in opposition to the legs. The steps should be time-based, i.e. should occur at a rate of *sps* (steps-per-second).
-   The player's head should rotate in sync with the arms/legs and basketball bounce.
-   The player should "dribble" the basketball in sync with the steps as well as rotate forward.
-   \<spacebar\> should toggle the animations
-   Use the part dimensions given in the file **Player.h** - **NO MAGIC NUMBERS**.
-   \<esc\> should quit the program.

**Note:** For Mac's running Sequoia, you may need to follow these [instructions](https://discussions.apple.com/thread/255759797?sortBy=rank) to be able to run the demo.

### Hints

> - When creating the node, the *base transform** should simply scale and locate the object in a convenient place, e.g. the bottom sitting on the x-z plane. Then the *update transform* should position it relative to its parent and apply any dynamic transformations.
>
> - Start with the court on the *x-z* plane, i.e. *y=0* and position the other nodes accordingly.
>
> - Consider the necessary rendering order for the objects in the scene graph in order for transparency to work properly.
>
> - Loading the included models will create vertex, normal, and texture coordinate buffers, thus they can be used for any type of node. Consider which elements of the scene should be **MatNode**s and which should be **TexNode**s to call the appropriate build functions.
>
> - Be sure to enable alpha blending and set appropriate blend factors for alpha blending effects. You will need to create "translucent" materials with non-unity alpha values and render translucent objects in the proper order within the scene graph.
>
> -  Set the appropriate parameter in **build\_lighting\_node()** to **true** for translucent objects.
>
> - Light sources have been provided.

## Grading Criteria

**The program MUST compile to receive any credit** (so develop incrementally).

**Milestone 1** - 30 points

-   Initialization (main): 3 points
-   Background image: 3 points
-   Player torso node (arbitrary texture coordinates): 4 points
-   Player head node (with proper texture map): 4 points
-   Player left upper arm node: 4 points
-   Player right upper arm node: 4 points
-   Player head animation (arbitrary rotation): 3 points
-   Upper limb material: 2 points
-   Upper limb animation (arbitrary rotation *about* shoulder): 3 points

> <img src="images/assign04/WalkingMan_MS1.png" alt="Walking Man MS1 Window" height="500"/>

**Milestone 2** - 70 points

-   Remaining player limb nodes: 15 points
-   Lower limb material: 3 points
-   Basketball node (with texture map): 5 points
-   Court node (with texture map): 5 points
-   Box node: 4 points
-   Translucent material: 3 points
-   Proper torso texture coordinates: 10 points
-   Robot arms/legs animation: 10 points
-   Robot head animation (proper rotation): 5 points
-   Basketball animation: 5 points
-   Creativity: 5 points

> <img src="images/assign04/WalkingMan.png" alt="Walking Man Window" height="500"/>

*Be creative!* For example, enhance the geometry of the scene and/or use additional animations. Remember that the program should still have reasonable performance on the lab machines.

## Compiling and running the program

You should be able to build and run the program by clicking the small green arrow towards the right of the top toolbar.

> <img src="images/assign04/WalkingMan.png" alt="Walking Man Window" height="500"/>

To quit the program simply close the window.

## Submitting to Marmoset

When you are done, submit the assignment to the Marmoset server using the Terminal window in CLion (click **Terminal** at the bottom left of the IDE). Navigate to the directory using

<pre>
$ <b>cd CS370_Assign04_Fa26</b>
CS370-Fa26/CS370_Assign04_Fa26
$ <b>make submit_ms1</b>
</pre>

or

<pre>
$ <b>cd CS370_Assign04_Fa26</b>
CS370-Fa26/CS370_Assign04_Fa26
$ <b>make submit_ms2</b>
</pre>

Enter your [Marmoset](https://cs.ycp.edu/marmoset) username and password, if successful you should see

<pre>
######################################################################
              >>>>>>>> Successful submission! <<<<<<<<<

Make sure that you log into the marmoset server to manually
check that the files you submitted are correct.

Details:

         Semester:   Fall 2025
         Course:     CS 370
         Assignment: assign04_ms1

######################################################################
</pre>

**You are responsible for making sure that your submission contains the correct file(s).**

