---Milestone 3---
Anne

Procedural grass color
In fragment shader, I wrote code that blends 2 different shades of green with a fbm function. At random, the color will be changed to blue or red.  This color is then multiplied and then added to the texture. 
It was challenging to decide where to add the code. For example, whether to add it before or after light is applied, and whether to add it in frag or vert shader. At first I wrote all the code in vertex shader and that causes a smooth blend between colors, which is not what I wanted. I changed it to frag shader.

Distance fog
Within a certain distance(60), I blended the color with a blue color.Beyond that distance, a  version of that color where alpha is equal to 0 is used for blending.  This way, the terrain will look like they are glowing and blend into the sky. Determining the distance and what combination of smooth step and power to use is difficult.

Procedural buildings with shape grammars
I created buildings with bridges between them using l systems. Every time there is a ì+î, a building is generated and the turtle is turned.  When there is F, a bridge is drawn at a random height that is below the current building height. The bridges are drawn with the ray marching method. 


Catherine

Biomes
I decided to divide the terrain into 4 biomes, the salt flats, desert plateau, birch forest, and islands. They are divided based on two scales, height variation and moisture. Salt flats is low variation (<.5) low moisture (< .5), desert is high variation (>.5) low moisture (<.5), birch forest is low variation (<.5) high moisture >.5), and islands is high variation (>.5) high moisture >.5). The values for these two scales are generated using two separate noise functions, FBM and Perlin Noise. I used Perlin Noise for both initially, which created a terrain with more organic, bubbly divisions. However, these divisions did not seem very natural. With some experimentation, I discovered that using FBM for the height variation scale helped add a jagged naturalness to the landscape, and helped the biomes blend better. I also scaled the biomes to be larger by dividing the vec2 position by large multiples.

I modified the shape of the terrain according to the type. For each type of terrain, I wrote a generating function--saltflats(), plateau(), forest() and island(). Taking in the vec2 (x, z) position, they return the maximum y height for each terrain type. For example, saltflats() returns a flat sealevel height at 150 no matter the input, whereas plateau() uses smoothstep() between very close decimals to create an abrupt rise in elevation. The forest() and island() functions both use the smoother Perlin Noise, but with different height ranges, with the island function dipping below sea level down to 129 so that water could be filled up to sea level. The four different terrain types are then 2D interpolated based on bumpiness and moisture, using a smoothstep() function from 0.47 to 0.53 so that the two biomes are only blended at the edges.

I also added several new custom textures, including SALT, BIRCH, BIRCHLEAF, and PLATEAU, which were created using photoshop. Each texture is implemented and used differently. SALT and BIRCH is created and mapped onto the respective elements. While the sides of birch trees use the BIRCH texture, the top and bottom use the top and bottom of the wood texture. BIRCHLEAF uses the grey leaf texture, but puts a random function into mix() to blend between bright orange and bright yellow, and passes the color into vs_Col to blend with the texture using multiply. The PLATEAU texture is divided into separate top and side textures.

Procedurally Generated Assets
I added birch trees in the birch forest biome. I decided to alter Terrain::CreateTerrain() so that when the forest biome is generated, tree are directly generated as well. To space out the birch trees, I created the setTree() function, which places trees based on a grid using the mod function. Given a grid size x, the function leaves out a 1 block margin on the top and right side, and allows the tree to be randomly placed within the remaining (x-1) by (x-1) grid. This way, after the pattern is repeated down the grid, there is a 1 block margin between each (x-1) by (x-1) grid and the next, so no 2 trees will touch. The height of the trees is also based on a random function based on its (x, z) position, and leaves are placed in a 3 by 3 (x, z) grid with the tree at the center, only at a height y of 10 and up. Leaf placement is random based on x, z, and leaf height.

Another asset I randomly generated was the floating islands, at the end of Terrain::CreateTerrain(). They are generated everywhere but the first 3x3 terrains surrounding the original terrain. In every new terrain beyond, there is a 75% chance of generating a floating island, based on a random function taking in the terrainís x and z numbers. The islandís position, height, and radius varies randomly from terrain to terrain. Based on the given parameters, the islandís bottom is generated through a recursive function so that each level becomes smaller (as y decreases, radius decreases).The shape of each level is based on a circle of the predetermined radius, with randomness introduced through adding 1 to the radius with a 50% chance. Finally, when a radius of 1 is reached, the function places a single block at the center. Also, the number of waterfalls is random and varies between 1 to 4. They are placed at random x and z coordinates in a square of 2*radius by 2*radius, and only generated when they are within the actual radius of the circular island. To create the waterfall, the water continues downward until its below the island and until it reached a non-empty or non-leaf block. When the ground is hit, the water creates a 3x3 block pool of water at that level.

Spencer - Day and Night Cycle & Post-Process Camera Overlay
Day and Night Cycle
For the day and night cycle I render the sky onto a quad that is located right in front of the far-clipping plane. The plane also fills the entire sky because the normalized display coordinates of the plane is scaled by a large number and sent from screen space to world space by multiplying by the inverse of the view projection matrix.
The time of day that is drawn is dependent on a time variable that added and is passed through a sinusoidal function to emulate a day-night cycle. The time of day is determined by the angle of the normalized height of the sun. If the sunís height is one then the sun is directly above the user, if it is zero then the sun is at the horizon, and if the height is less than zero, then it is below the horizon and the light source now switches over to be the moon.
To have day, dusk, sunset, and night time sky boxes, I make color palettes for each time of day and interpolate between these palettes at particular moments of the day so that the sky appears to be changing continuously.
The clouds are generated by worley fractal brownian noise function which also uses a 3d worley noise function which is basically the same as worley noise 2d except the points are scattered within a 3d grid where each cell of the grid contains a point. The value retrieved from this noise function is then projected onto the sky plane. The noise is also animated so that points kind of ìjiggleî in their cells to give the appearance of moving clouds.
Rings of light encircle the sun and moon by checking to see if the current fragment was within a certain radius from the light source and interpolating the light source color with the sky color. The interpolating factor t is inversely proportional to the distance the pixel is from the light source.
I also edited the fragment shader for lambert so that the light vec now changes with time such that th elight direction follows the sun's position. I also subtly colorshift the environment to blue-green at night and warmer near sunrise and sunset

Post-Process Camera Overlay
For this, I basically used the same post-procesing pipeline from the opengl fun homework and use a shader that has a blue transparency when in water and red when in lava. The transparency color is changed whenever the player is swimming.


My Contributions to the Project:
Terrain Chunking
Multithreaded Terrain Generation and Swimming Physics and Overlay
Day and Night Cycle
Post-Processing Camera Overlay




---Milestone 2---
Anne - L-system

New Class:L-system
New functions in terrain class: makeRiver(), makeDeltaRiver()

First I used a L-system visualizer online to find the rules that makes a desired river shaped. I made a L-system class that stores a terrain and functions to draw on the terrain with a given string. The class also has functions for expanding a string with the rules and functions that correspond to each character in the string such as storing turtle, rotating, draw line.  I implemented a data structure for the turtle, which stores position, rotation and width. The width is decreased when a turtle is stored and increased when turtle from stack is popped. Also, There are different rules for the main river and the delta river. 

The depth of the water is determined by the square of their distance to the shore.  The river is placed at 128 and the terrain above and near it are carved out.  I used a ray march function to carve out the land perpendicular to the river’s direction. 
Inside the terrain class, I made 2 functions for making the 2 rivers, which calls the functions in L-system. 

It was difficult to get the river’s shape to look good at first because we need to determine the length of the increment and the rotation angle.  I had to test many different numbers to understand how each parameter affects the look. 
Also, determining how to carve out the river so that it looks natural was difficult.  By using ray marching, I found all the blocks perpendicular to river’s direction and decreased their height. 



Catherine - Texturing and Texture Animation

Texture & normals
I used slot 0 for textures and slot 1 for normals, since using different slots would require images to be loaded only once. I also added the ìinî UV attribute to determine where within the normal map to read the UV. I had to change the interleaf VBO from storing vec4ís to storing floats in order to enter UV as a vec2, and later to store cosine power and the animateable flag. Based on the block type, I assigned different UVs, and also differentiated the top, bottom, and side UVs to create grass and wood blocks.  I also attempted to add normals by transforming the initial normal map, which represents the side with normals (0, 0, 1), to each side. But since the initial terrain is generated with the front having a normal of (0, 0, -1), and the back having (0, 0, 1), and assumes that the z axis goes backwards rather than forward, I had to make adjustments to the usual transformations to create a rendering that makes sense.

Transparency
I created two VBOs, one to hold non-transparent blocks and one to hold transparent blocks. Based on the block type, they are either added into the opaque VBO or the transparent VBO. In the same draw function in ShaderProgram, I rendered the opaque blocks first, then the transparent blocks.

Texture Animation
I added an animateable flag to the end of the interleaf vector as the 16th element. For water and lava, animateable is set to 1, and for others it is set to 0. I also added a uniform Time variable, set by timerUpdate based on the number of milliseconds since Epoch. In frag.glsl, when animateable = 1, the time elapsed from the start of the program is scaled to be 1/10,000th of the original integer, or 1/10th of a second. Then it mods 1/16, which maps the number to a value between 0 and 1/16 to set as offset for x. This way, each fragment in the river would move in the x direction from the given UVís x to x+1/16 based on time.

Blinn Phong
Blinn Phong shading uses cosine power, the light direction, and the view vector.  Light direction is constant. The cosine power is added to the interleaf vector as the 15th element. For shinier elements like water and ice, I assigned a cosine power of 15, while for duller elements like dirt and grass, I assigned a power of 1.5. To calculate the view vector, I added a uniform variable CameraPosition to vert.glsl, which is set every time the user moves and the camera position is recomputed. Then CameraPosition is used to compute the view vector of the current vertex by doing CameraPosition - u_Model * vs_Pos, which would be the world camera position subtracted by the world vertex position.


Spencer - Swimming and Multithreaded Terrain Generation

Swimming Physics
This was the easiest part. After reading my partner's player physics code multiple times, I finally understood their logic so that I could edit it such that velocity and acceleration were two-thirds of what it usually is. In my partner's velocity and acceleration update functions, I simply multiply the velocity and acceleration by 2/3, but also reset the acceleration to what it was orginally, otherwise the acceleration would consantly decrease. This is problematic because then the acceleration would approach zero rendering the user completely motionless.

Swimming Color Distortion
In order to achieve thi effect, I first attempted to use a post-processing render pass like how we did in OpenGLFun since I thought this was mandatory. However, after many attempts at fixing my post-processing render pipeline so that it would render something instead of nothing I found a Piazza post saying that we don't have to use post-processing. So, instead I followed advice in the Piazza post and created a colored, semi-transparent quad that was set to a particular color depending on whether the player was in water, lava, or empty blocks. I didn't apply a model matrix to the quad since it should always be directly in front of the user at all times.

Multithreading and Mutexes
This gave me more trouble than I expected. It was crashing for unknown reasons and would sometimes throw errors (P.S. if it crashes immediately after attempting to run, then just try to run it again; this usually works). I created a thread object in the terrain class. Each thread was responsible for computing the fbm function in parallel for each surface block within 4 chunks of the 16 chunks. This was working fine but then I was having trouble with the initially generated terrain objects. Specifically with how the river is generated on top of these terrain objects. I think the generate river function and the fbm function were editting the terrain at the same time, which caused incorrect assigning of blocks. But I tried to fix this by having these initial terrain objects generate fbm data sequentially since this won't really affect gameplay since the game hasn't really started, but now my game crashes whenever I attempt to cross a boundary.


---Milestone 1---

Anne - Procedural Terrain
Terrain Generation (fbm)
I used the fbm method from lecture slides. The maximum highest is 140 because the rock goes to y = 128,  maxY = 140 allows  enough flat space for player to walk on. The terrains contains variable that stores the x and z position. This is necessary so that when drawing the terrain,the model matrix can be set to a translation matrix to shift the terrain. 

All the terrains are stored in a std map that maps a int pair(x and z position) to a terrain unique pointer. This way, we can check whether a terrain exist and decide if we have to create it. Also, we can find the block at any world coordinate by first determining the terrain a coordinate is in by using divide by 64 and floor. Then use modulo to decide local position within terrain.

New Terrain on playerís border:
I test both x and z direction to see if any side is close to the border, and generate the terrain close to that border. Additionally, I test if both x and z are near the border, which generates terrain in the diagonal.


Raycast
Method: distToBlock
Returns: Distance to first non empty block. -1 if none found.
It was written in MyGL because we need to access all the blocks in the world. 

addBlock/deleteBlock functions
These functions calls distToBlock() which returns the t value. Using t, we can find the position of the block it hits. For delete, lerp 0.001 from block position in the direction of ray so that the ray is inside block and we know which one to delete.For add, lerp 0.001 from block position in the opposite direction of ray and that gives the position of the box to add.




Spencer - Efficient Terrain Rendering & Chunking
Chunk class
The chunk class is an inner class of the Terrain class since it is an internal data structure that should not be manipulated directly unless the programmer knew exactly how everything was indexed inside of the class.
I gave my group mates functions that gave them access to the underlying chunking data structure so that they would not have to directly interact with the chunking data structure inside of the terrain objects
The Chunk class has a data member of type BlockType array of size 65536. I used a 1D array instead of a 3D array to represent the contents of the array because this would provide somewhat faster access time .
Leveraged the error throwing properties of the at() method instead of the [] operator when accessing maps and arrays to ensure that we can catch any accidental out of bounds access. For even more debugging assistance, I wrapped these at methods inside try-catch blocks which gave a detailed error message whenever out-of-bounds exceptions occurred.

Edited and expanded the terrain class functionality so that all of its functions still work in the case when there are multiple terrain objects.

Computed which faces of the cubes comprise the hull of the terrain to drastically increase render time. I then populate the CPU vectors with the corresponding position, normal , and color attributes of each vertex defining a visible face. Instead of dividing each of these attributes into 3 VBOs on the GPU, I combined them into a single interleaved VBO where all of the vertex attributes of a single vertex are contiguous. This allows for somewhat faster render times.

Removed the m_blocks 3D array and replaced with a map of uint6_t4 to uPtr<Chunk>. Since tuples and ivec2s donít have hash functions for mapping, I used the x, z position of the chunk relative to the chunkís corresponding terrain. I combined these two 32-bit integer values into a single 64-bit integer so that I can get automatic hashing. I was concerned about this implementation because I was unsure of how bitwise operators would behave on data types of different sizes. Specifically, I was concerned if C++ uses sign-extension which could potentially mess up how I combine binary numbers. To ensure that the key that I was calculating was correct, I wrote a recursive function that prints the binary representation of numbers.

I then needed to figure out how to modify the ShaderProgram and Drawable classes. I used the existing code as a template to determine which functions to call and in which order. I also changed how the lambert vertex shader determined which color to draw boxes. Instead of having a uniform value color that is set inside of mygl.cpp based on the block type, the shader uses an in vec4 for color that I set with the interleaved VBO.



Catherine - Game Engine Update Function and Player Physics
Character Dimensions
The character is 1x2 and aligned with the world axis. The character height is set to 2.f, but the camera is at 1.8f so that the eyes are towards the top of the top block, but not at the exact top. This way, when the character collides at the top (in a cave or indoor environment), they can still look up and see what they are colliding with.

Camera Angles
The character position controls the camera position. I made sure to clamp the y camera angles to -89 degrees to 89 degrees to reduce jitteriness, and to ensure there is always a horizontal component that the character can walk towards.

Timer Update Using Elapsed Time
The acceleration changes according to time elapsed. It is scaled by elapsed_time*base_accelartion/16.f since the original velocity is created for an average update time of 16ms. The scaling multiplies the number of actual milliseconds elapsed with the velocity per millisecond. Then velocity is calculated based on the acceleration*time using velocity=velocity+acceleration*time.

Aerial State
I decided to keep track of whether the character is aerial, and store it in a boolean in Player. The aerial state would tell us when to set a negative velocity, and remove the necessity of raycasting downwards constantly. Also, having an aerial state assists with jump. When a character is aerial, that character cannot jump and gain positive velocity from it. To check whether a character is aerial, in timerUpdate, I checked the blocks beneath the four corners on the base of the character. This way, if either one of the characterís corners is still on a non-empty block, the character should not fall. This ensures compliance with collisions.

Collisions
We decided to collaboratively write one raycast function distToBlock() in MyGL that takes in the origin of the raycast, the direction of the ray, and the max distance that it casts for. This is because raycast is used in both adding and removing blocks, and in player physics. Using the global raycast function, I casted rays of length current velocity from all 12 points (base, middle plain, top) to detect collision with world blocks. The minimum collision distance from the 12 points was used as the travel distance.


