----Milestone 1----

Yin Tang: Procedural Terrain Generation

The main task I implemented is using noise function to procedurally generate terrain. Since noise function is only going to be used in procedural generation, I created a noisehelper to store all the noise functions that will be used. Later I separated the procedural generator into a different class to better use in later system features generation. The main function in procedural generator that the terrain class will use is the get height method, which will return a height value based on the x and z coordinates of the current block using noise functios. Terrain can then use the height to fill in the block type. 

The first thing I realized is that since we want to have the infinite expanding world map all randomly generated, the input fed into the noise function cannot be associated with any chunk, zone variable, which will then lead to repetition. Then the simplest method is to divide the x and z coordinates by a non-related constant, which can also serve to expand or compress the generated map into desired terrain. I used Perlin noise as the basis function for fbm to generate the mountain terrain height, and fbm noise as the basis function for worley noise, which factors in secondary cell distance, to generate grassland terrain height. The main difficulty of this part is choosing the right coefficients to tune the noise function to get to desire shape, e.g. the amplitude and persistence of fbm, the x and z coordinates divide constant, the random vector used in dot product etc. The last part is to blend different terrains together so that they can transition smoothly from one to another. I am having difficulty to understand which noise function to use and how to fine tune the noise function at the beginning, but ChatGPT is really helpful in giving me an example and I start from there. I first used Perlin noise as described in the description, but there were way too much mountain than grassland, so I used fbm again to scatter out the distribution.




Qichao Lin: Efficient Terrain Rendering and Chunking

I implemented creating chunk VBO data and I added a feature to create forward, backward, right, left chunks based on the player's position on the edge of the scene. 

Difficulties I encountered when coding the project:
After merging with Christine’s work, the position data seems not the same as chunk’s actual position. Then the destroy block function and collision function do not achieve the expected effect. With the discussion with Adam, I find the issue located inside my Chunk::createVBOdata() part. When I push back my position data, the calculation of my z value is not correct, so it affects all the errors. 

In addition, in the previous version, I needed to move to the edge of the current terrain to load new chunks, but it caused some issues. Therefore, I adjust the renderTerrain() part to make the judgment range bigger to prevent problems. 




Christine Kneer: Game Engine Tick Function and Player Physics

I implemented player movement, specifically:
W/A/S/D to move forward, left, backward, and right (with shift pressed to move faster)
Space to jump
F to toggle between flight mode and walk mode
E/Q to fly up and down in flight mode
Mouse movement to control the orientation of the player
These features are implemented by updating the InputBundle of the player in keypress/keyrelease events and mousemove events in MyGL. The InputBundle controls the acceleration of the player which is then used to compute the velocity and position of the player in ComputePhysics.

I also implemented collision detection and ground check for player (in walk mode). The player’s collision volume is assumed to be two Minecraft blocks stacked on top of one another. Collision detection is implemented through raymarching along each cardinal axis and then limiting the player’s movement on each axis based on the minimum distance to move for smoother gameplay. Ground check is achieved simply by checking the block beneath the player’s four corners to see whether they are empty. The player is only allowed to jump when they are grounded, and when they are not, gravity will apply so that the player will fall back to the ground. (Water physics has not been implemented yet.)

Lastly, I implemented adding and removing blocks. The block that the player is looking at (within 3 blocks distance) can be removed by left-clicking. The player can add a block adjacent to the block they are looking at (within 3 blocks distance) by right-clicking. So far, the added block is naively set to be of the same type as the block the player is looking at.

Adding a block was the first challenge I faced. Determining the correct position to add a block was not easy. I had the intuition to use the ray cast point’s position and compare it with the block position, but the exact math took me a while to figure out. I had to draw a 3D cube and manually figure out each case’s criteria. 

The main difficulty I faced was not the implementation itself but when merging with my teammates. As soon as we merged, things broke and it was hard to locate which branch caused the problem. But in the end, we narrowed down the problem by playtesting many times. For example, adding blocks broke after we merged. But through playtesting, we realized that the block actually exists (collision happens) but does not show up visually, so we realized that this might be a VBO update thing and we successfully fixed it.

Overall, a big thing that I learned is that in real-world coding, there are a lot of edge cases that are easily overlooked and are different than the theoretical formulas. For example, when using floating point, usually we need to allow some epsilon error when comparing values, although in formulas, everything is strictly equal. Also, we need to consider edge cases that break the code like division by 0 which happens when values become really small. 


----Milestone 2----

Yin Tang: Cave System

My implementation for milestone two can be divided into three parts: cave system, player physics in water and lava, and post-process shader.

Cave system development is a more advanced application of Perlin noise that I used in milestone one, in which I extended the 2D Perlin noise to 3D Perlin noise and used a threshold to determine whether to keep the block. The result looked pretty nice, and I tuned the threshold a bit so that individual caves were not that far separated. Two things that I think I can improve in milestone three: first thing, I watched the Minecraft video presented by a Minecraft engineer posted on Canvas, and I am pretty interested in those two types of cave, the big chunk cheese cave and the narrow tunnel spaghetti cave. Currently, all the caves are cheese caves in shape, so I can add a separate noise function to create spaghetti caves and mix them so that caves get more interconnected. My second thought is that I can add texture to the surface area of the caves to make them look interesting.

The second part is the player physics in water and lava. Many thanks to Christine, who walked through the player code structure with me and pointed out some potential places to put in modifications so that I could finish it much faster. I added a new indicator variable into the player class to determine whether the player is in water or lava and used that variable to adjust the acceleration and velocity accordingly.

The third part, the post-process shader, took me the most time to debug. I finished the primary codec following HW4 ShaderFun's implementation, adding in framebuffer object, quat drawable and post-process shader into MyGL class, setting up the framebuffer and passing the texture accordingly. However, the output of the post-process shader looked like a magnified and shifted result of the texture. I spent a lot of time adjusting the viewport parameters and debugging the shader code, but no luck. Thanks to our TA Linda, who suggested that a resolution problem most likely caused the issue and that I should check the resize function. It turned out that after resizing the framebuffer, I also needed to destroy and create it again so that the textImage2D also had the correct size to take in the texture. 





Christine Kneer: Texturing and Texture Animation

I implemented texturing, specifically applied texture to GRASS, STONE, DIRT, SNOW, LAVA, WATER, BEDROCK using the minecraft texture file we are given. Each block type has different UVs for each face which are hardcoded in texturehelp.h. When creating VBO, the correct UV are read from texturehelp.h and pushed into the interleaved VBO. Note that in milestone1, the interleaved VBO consists of position, normal and color, but now color is replaced with uv. Since uv is a vec2 yet interleaved VBO are vec4s, the last two floats of uv are set to be 0.f just to remain the structure. I also optimized the interleaved VBO creation process. I realized that in milestone1, a separate vector is created for each attribute and then iterated through and combined at the very end. I got rid of them and have the interleaved VBO created directly.

In order for transparency to work, a separate transVBO needs to be created (transIdxVBO too). Opaque and Trans VBOs are simultaneously created and sent in Chunk::createVBOdata and Chunk::sendVBOdata. I also had to create a new draw function for transparent VBO in shaderprogram. This needed to be done in order to use the correct count (one for opaque blocks, one for transparent blocks). Meanwhile, in Terrain’s draw function, transparent blocks are drawn after opaque blocks for alpha blending.

I also implemented texture animation for WATER and LAVA blocks. This is achieved through displacing the UV based on an uniform time variable in the shader. Since only certain blocktypes need to be animated, an additional animation flag is passed to the shader. I used the uv portion of the interleaved VBO to achieve this. Specifically, since now there are 2 floats in uv that are wasted, I used the last float to serve as an animation flag for each block.

Overall, I thought that texturing was quite straightforward, but I encountered several small problems in the process. At first, all of my texture turned out to be black. I did not know what was wrong so I tried simply having the fragment shader output a certain color, but somehow nothing is showing up. I realized that the draw function actually had some specific logical error left from milestone1, but since I wasn’t in charge of that section it took me a while to figure out. Also, once the texture shows up some of the block faces are flipped, which also took me a while to debug until I realized that the createVBOdata is also somewhat incorrect. Milestone1 only used color, so orientation problems did not become apparent until texture was applied. Again, most of the difficulties arise from the collaborative nature of this project. Collaboration is efficient, but I learned that when working with other people, it would be very helpful to write meaningful comments and keep the code structured and clean along the process. Each part is dependent on one another, so it is important to always keep in mind that the goal is not only to make my part work, but also make it easy for other people to read/use/reuse/debug my code.





Qichao Lin: Multithreaded Terrain Generation
I implemented multithreading during milestone 2 through Qt library: QRunnable, QThreadPool, and QMutex classes. Every tick of Mygl, the program will check the player's current position. For every terrain generation zone based on the current position of the player that does not yet exist in Terrain's m_generatedTerrain, I spawn a thread to fill that zone's Chunks with BlockTypeWorkers. In addition, in terrain, I create struct ChunkVBOData to store opaque and transparent vertex and index data for the Chunk VBOs. 
More importantly, in Mygl:tick(), I used the BlockTypeWorker worker class to generate Block data in threads. It will store data in the m_chunksWithOnlyBlockData vector. Then I generate new threads to create VBO data for chunks using VBOWorker. In the final part, in the main thread, I send all VBO data into the GPU.

