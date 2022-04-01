# HW1: Set up PBRT and Experiment with Lighting Design

This assignment is designed to help you get pbrt, the physically-based rendering system you will use throughout this course, set up and running on your machine, as well as teach you a bit about lighting design. Along the way, you'll familiarize yourself with the software by rendering several test scenes and reading the first few chapters of the textbook. The pbrt system has been developed and tested extensively on Linux, MacOSX, and Windows; feel free to use whatever platform you're most comfortable with.

# Step 0: GitHub Setup
***

In order to submit your assignments, please create a private Github repo named `cs348b-2022-yoursunetid`, 
then duplicate the [pbrt-v3 github repo](https://github.com/mmp/pbrt-v3) by following [these 4 steps for mirroring a repository](https://help.github.com/articles/duplicating-a-repository/).  It is important to make sure your repo is private, as publicly posting solutions to assignments is a violation of the Stanford Honor Code. Note that this means you cannot use Github's fork feature as it does not allow making private forks of public repositories.

Once you have created your repository, invite the TAs using their Github usernames (posted on Ed) by following [these instructions](https://help.github.com/en/github/setting-up-and-managing-your-github-user-account/inviting-collaborators-to-a-personal-repository).

After you have created your duplicated repo, please checkout the repository locally and use the "master" remote branch:

      git clone --recursive url_for_your_repo pbrt-v3
      git checkout master

You will write each assignment in this class in a separate git branch to make submission easy. After cloning, change into your repository's root directory (pbrt-v3 in the prior command) and create the branch for this assignment:  

      git checkout -b assignment1

# Step 1: Building PBRT

* * *

Detailed instructions for setting up pbrt on various platforms are available in the main repository [README.md](https://github.com/mmp/pbrt-v3/blob/master/README.md#building-pbrt). As part of the instructions you will need to get [cmake](https://cmake.org/), which is a widely used cross-platform build tool. For most unix systems, the following commands will build PBRT in a a directory named `pbrt-build`:

    mkdir pbrt-build; cd pbrt-build
    cmake path_to_your_repo
    make -j #of cores on your machine

Once you have successfully built PBRT, you can run it explicitly from the command line by typing the path to
where the `pbrt` binary is located (for example `./pbrt-build/pbrt`). However, it is more convenient to add the path to the `pbrt` executable to the execution environment on your OS, which allows you to run the binary by only typing `pbrt`.

On OS X and linux this is as simple as adding 

    export PATH=$full_path_to_your_pbrt_build_directory:$PATH
    
to your [.bash\_profile file](http://superuser.com/questions/678113/how-to-add-a-line-to-bash-profile). On Windows, you can use the GUI to change your [PATH environment variable](http://superuser.com/questions/949560/how-do-i-set-system-environment-variables-in-windows-10).

For the remainder of this assignment, we'll assume that pbrt is correctly built and that the pbrt binaries are in your path.

# Step 2: Rendering Your First Image

* * *

Once you have successfully compiled pbrt, you can render your first image. Download the Assignment 1 starter files [here](http://graphics.stanford.edu/courses/cs348b-20-spring-content/uploads/assignment1.zip), and unzip them in the `scenes` subdirectory of your repository. You will be including these files in your submission, so please make sure they're in the right location. For example, a file name `lighting.pbrt` should now be located at: `path_to_your_repo/scenes/assignment1/lighting.pbrt`.


From within the `scenes/assignment1` directory, render an image with the command:

    pbrt lighting.pbrt

After pbrt is done rendering, it will print statistics of the rendering process to the screen and output the image file `lighting.exr`.

Congratulations, you've rendered your first image with pbrt! The EXR image stores linear radiance values, which can have a very large range; but your screen can only display ~8 bits of color information per RGB channel, so these values have to be mapped in some way.
We recommend converting to the standard .png format using the defaults in pbrt's own `imgtool` program (built automatically when you built pbrt).

    imgtool convert foo.exr foo.png

You can also view the EXR file directly using programs such as OS X's Preview, Adobe Photoshop, GIMP 2.9.2+, or OpenEXR's `exrdisplay`, but you must include the converted png files in your submission.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_2.jpg)

A correctly rendered image is shown at left in the figure above. In this scene, the camera is pointed directly at the model of a face. A single spotlight shines on the face from behind the camera and slightly off to the right, casting a hard shadow onto the wall behind the model. The location of the camera and spotlight are illustrated on the right.

Before you move on, please open `lighting.pbrt` and read all the comments carefully. Try to understand the positions of the camera, the spot light and the head model. Keep in mind pbrt uses a **left-handed** coordinate system. It is also highly recommended that you take some time to go over `Cameras`, `Lights`, and `Area Lights` in [the pbrt-v3 file format manual](http://www.pbrt.org/fileformat-v3.html) before you start doing the assignment.

# Step 3: Lighting the Scene

* * *

In this assignment you will manipulate the lighting in a simple test scene to create several images using pbrt. Try your best to imitate the provided renderings, using the illustrations as hints on where to position the lights.

## Configuration 1: Spotlight From Below

First, we will move the spotlight so it points up at the face from below. Open the `lighting.pbrt` scene file and find the description of the spotlight in the file. It should look like this:


  `LightSource "spot" "color I" [50 50 50] "point from" [-0.5 0 4.7] "point to" [0 0 0] "float coneangle" [60]`


By inspecting the `"point from"` field, we see that the light is located behind the camera and slightly off to the right. It is aimed at the center of the scene (the `"point to"` field) where the model is located. Now, move the light so that it is pointed at the model from the position and orientation shown below. A simple change in lighting completely changes the mood of the portrait!

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_3.jpg)

Save a copy of the modified `lighting.pbrt` file with your settings in `scenes/assignment1/config1.pbrt`. Save the rendered result in `scenes/assignment1/render1.png`.

## Configuration 2: Area Lighting From The Right

The light from a pbrt spotlight originates from a single point in space. Light originating from a single point in space results in hard shadows that photographers generally find visually objectionable. Area lights emit light from a large region in space, resulting in a softer lighting of the subject and the generation of shadows with penumbra. The pbrt scene file contains the following definition of an area light (commented out):

    AttributeBegin  
    AreaLightSource "area" "color L" [10 10 10]
    # use camera coordinate system [optional]  
    CoordSysTransform "camera"  
    # adjust light source position  
    Translate 0 0 -2  
    Rotate 90 1 0 0  
    # define the shape of the arealight to be a disk with radius 1.5  
    Shape "disk" "float radius" [1.5]  
    AttributeEnd

Light from this source is emitted from a disk of radius 1.5\\. The luminance (or energy) emitted by this light source is given by the value "L". Take out the spotlight from Configuration 1 and modify the area light definition above to produce an image similar to the one shown below. Save your settings in `scenes/assignment1/config2.pbrt`, and save the rendered result in `scenes/assignment1/render2.png`.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_4.jpg)

You will notice an increase in rendering time when you enable the area light in the scene. This brings up an important point: sometimes you'll want to speed up the rendering process so that you can tweak a scene's properties more efficiently. The simplest way to do this is to decrease the size and quality of the rendered image. Notice that `lighting.pbrt` is set up to render 300x300 images. To render smaller images, change `"xresolution"` and `"yresolution"` in the `Film` definition in `lighting.pbrt`.

    Film "image" "string filename" ["lighting.exr"] "integer xresolution" [300] "integer yresolution" [300]

Additionally, decreasing the number of samples (eye rays) that pbrt uses to compute the value of each output pixel will also speed up rendering. The `Sampler` definition in `lighting.pbrt` instructs pbrt to sample each output pixel using 4 rays.

    Sampler "halton" "integer pixelsamples" [4]

We ask you to use at least 4 samples for your final hand-in images (and at least 64 samples for configurations 5 and 6), though using more is fine and will lead to higher-quality images.

## Configuration 3: Two Light Sources

Notice how the left side of the face is in shadow in Configuration 2. In fact, it is so dark that no detail is visible at all. A photographer might use additional lights to illuminate the subject more evenly. In this setup, a spotlight shines on the model at an angle from the left, and a reflecting surface is placed to the right, in order to reflect light back onto the dark side of the face and produce more even illumination. This second source of lighting on the model is referred to as _fill lighting_. One way to approximate this effect in pbrt is to use two light sources. Illuminate the model using a spotlight from the position indicated in the illustration below. Then use a large area light on the right of the model to give the effect of fill lighting on the dark side of the face. Render the scene with and without the fill lighting to observe the visual difference.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_5.jpg)

Save your settings in `scenes/assignment1/config3.pbrt`.
Save the rendered results with the fill light in `scenes/assignment1/render3_fill.png` and without the fill light in `scenes/assignment1/render3_nofill.png`.

## Configuration 4: Many Light Sources

In practice, photographers use many lights to illuminate a subject so that the resulting image conforms precisely to their preferences. Try to adjust the lighting conditions to recreate the scene below, in which four light sources are used to create a visually pleasing effect. Notice that in this example you will also need to move the position of the camera to view the model from a slightly different angle. The LookAt directive specifies the position of the camera in world space, the location the camera is pointed at, and the "up" direction of the camera. Camera settings are defined by the following two lines in the scene file:

    LookAt 0 0 4.5 0 0 0 0 1 0 
    Camera "perspective" "float fov" [22]

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_6.jpg)

The following 4 images show the separate effect of the four light sources.

The _main light_ is on the side of the face away from the camera.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_7.jpg)

The _fill light_ is close to the camera on the opposite side from the main light.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_8.jpg)

The _accent light_ shines toward the upper part from the back of the subject. Normally, it is used to highlight the texture of hair. In our case, we'll use it to accent our model's embarrassing _lack_ of hair.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_9.jpg)

The _background light_ separates the subject from the background. It is placed behind the subject and to one side, facing toward the backdrop.

![](http://graphics.stanford.edu/courses/cs348b-20-spring-content/article_images/4_10.jpg)

Save your final pbrt file with all four lights in `scenes/assignment1/config4.pbrt`.  
Save separate rendered results with each of the lights individually in `scenes/assignment1/render4_main.png`, `scenes/assignment1/render4_fill.png`, `scenes/assignment1/render4_accent.png`, and `scenes/assignment1/render4_background.png`.  
Save the combined render in `scenes/assignment1/render4_full.png`.

## Configuration 5: Realistic Lighting

You may have noticed that manually placing lights is a bit of a pain! A technique that has become common in the last few years is to use a HDR "environment map" that has captured light from all directions in a real-world environment, and use it as a light source for rendering. We provided one such map in the starter files, `textures/doge2_latlong.exr`: 

![](http://graphics.stanford.edu/courses/cs348b-17-spring-content/article_images/4_.jpg)

We already have the necessary configuration line in the `.pbrt` file to use this for lighting. Uncomment out the line 

    LightSource "infinite" "string mapname" ["textures/doge2_latlong.exr"]
    
and comment out all other lightsources. Generate an image like the one below:

![](http://graphics.stanford.edu/courses/cs348b-17-spring-content/article_images/4_1.jpg)
    
You will need to increase the `pixelsamples` of the `Sampler` significantly to get an image with a small amount of noise; please use at least 64 samples for this configuration. This brings up an interesting tradeoff that you can see everywhere: we were able to drastically simplify the amount of work that needs to be done by humans by making the computer work harder.

Save the configuration in `scenes/assignment1/config5.pbrt`, and save your rendered image in `scenes/assignment1/render5.png`. 

## Configuration 6: Realistic Model and Materials

Finally, lets try rendering a more realistic model. Comment out the "Head model" section of the `.pbrt` file, and uncomment the "More realistic head model" section, and rerun. You may want to increase the `pixelsamples` once again; you can also increase the `maxdepth` of the Integrator; this image was created using 4096 samples per pixel with a maxdepth of 5. That took quite a while on a laptop, so feel free to use more modest settings, but we request you use at least 64 samples per pixel for your final image.

![](http://graphics.stanford.edu/courses/cs348b-17-spring-content/article_images/4_2.jpg)

Save the configuration in `scenes/assignment1/config6.pbrt`, and save your rendered image in `scenes/assignment1/render6.png`. 

# Submission and Grading

* * *

This assignment will be graded on a **credit/no credit** basis. Credit will be given if the example renderings are reproduced to reasonable accuracy. Try your best to mimic the lighting configurations in the example renderings, but don't worry if your images aren't perfect matches. For example, for configuration 2 , you solution is acceptable as long as you put the light source to the left of the head model and manage to get highlight and shadow on the face. The important thing is that you become familiar with pbrt and manipulating properties of its scene files.

Once you are done configuring lighting and rendering, create a file named `Submission.md` in the root of your repository that contains a summary (in a few sentences) of the process you followed to develop the correct scene configurations. This summary might include problems you encountered with the pbrt build/installation process, steps you took to speed up the render/view/render process, other experimentation you performed, etc.


Add all of the files you created (`.pbrt` files and images) to your github repository, and commit them as follows:

      git add scenes/assignment1
                
      git commit -m "yourCommitMessage"

After you commit your changes, you can push to the remote branch using the command below to push to the remote branch "assignment1"

      git push -u origin assignment1


You only need to do this once, once you have successfully associated your local branch `assignment1` with the remote branch `assignment1`, you can push new changes to the remote branch with just:

      git push

When you are ready to submit, you can [issue a pull request](https://help.github.com/articles/creating-a-pull-request/) to your TAs. This means creating a pull request from your `assignment1` branch to *your* `master` branch, and adding the TAs as reviewers of the pull request. Note that you should *not* make a pull request to the public pbrt repository, as this would require making your solution public.

Once your pull request is created, review it to ensure it contains all the required files (please exclude extraneous files such as `.exr` files):

1. Images you rendered in the steps above in `.png` format (You should render all these images at a resolution of 300x300, with at least 4 samples per pixel for configurations 1-4, and 64 samples per pixel for configurations 5 and 6): 
 * one for configuration 1
 * one for configuration 2
 * two for configuration 3 (with and without the fill lighting)
 * five for configuration 4 (four for each single light source and one for all). 
 * one for configuration 5
 * one for configuration 6
2. `.pbrt` files (`config1-6.pbrt`)
3. Your Submission.md file
