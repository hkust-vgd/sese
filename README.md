# Introduction
`sese` is a user interactive scene mesh annotation tool. It has been used to annotate more than 100 scenes in the SceneNN dataset. 
To download, please navigate to <https://github.com/scenenn/sese/releases>

We provide binary releases of the tool on Windows and Linux (Ubuntu 16.04 LTS). Please contact us for more details if you are interested in using the tool on other platforms.

Please read the disclaimer at the end of this document before using the tool. 

Please cite our technical report 
    
    A Robust 3D-2D Interactive Tool for Scene Segmentation and Annotation
    Duc Thanh Nguyen, Binh-Son Hua, Lap-Fai Yu, and Sai-Kit Yeung
    arXiv 2016
    
if you use this tool in your research. 

In the following sections, the data organization, shortcuts, and operation modes of this program are described. 

# Data organization
```
[data] 						Datasets, each stored in a separate folder. 
	[322]                   Data for scene <ID>
		[depth]             Images extracted from synchronized ONI video (optional)
		[image]
		trajectory.log      Camera pose
		322.ply	            Segmented triangle mesh
		322.xml             Text labels and bounding boxes.
[ini] 										
	sese.ini                Default parameter file for annotation tool.
	asus.ini                Default camera intrinsics for Asus Xtion.
	322.ini                 Custom parameter file for each scene (optional)
[ssao]					    Screen-space ambient occlusion shaders
[platforms]                 Qt dependencies
*.dll 						Other dependencies
sese.exe 					Main program
```

Each scene folder contains the annotated PLY, XML, and trajectory.log. This is the minimum data required to launch the annotation program. If depth and color images are extracted (from synchronized ONI files), please put them into the depth and images folder as shown in the folder tree above. 

We assume that color is stored at each vertex in the triangle mesh, which means that for accurate texture, the subdivision of the mesh has to be relatively fine. For mesh extracted by marching cubes from Kinect Fusion and its descendants, this subdivision is usually good enough. 

It is also possible to use this tool to annotate meshes reconstructed by multiple view geometry, i.e., from VisualSFM + CMPMVS. One just has to ensure that the mesh fits the above assumption to some good extents. Note that this feature is not quite carefully tested, but please feel free to experiment. 
	  
# Launch
To visual the scene mesh, simply go to sandbox folder and execute
```
sese.exe data/322 
```
where `322` is the scene ID, `data/` is the folder where the scenes are stored.

To start the program in annotation mode, execute 
```
sese.exe data/322 user 
```

An OpenGL window and an annotation window will pop up. If there are depth and color images, a window that shows 2D images will also be displayed. 

# Shortcuts

In the 3D editing window, one can refine the segmentation by drawing strokes. Typical keyboard and mouse shortcuts are:

Key | Description
--- | ---
`Alt` | switch to display small labels on the fly. 
`Shift + Left drag` | merge two labels. 
`Alt + Left drag` | extract a small label / group small labels into a big label.
`Ctrl + Left drag` | split a big label into several small labels.  In practice, avoid using this as its result could be messy.
`Left click` | show the bounding box of the current label.

A comprehensive list of supported keyboard shortcuts is as below.

Key | Description
--- | ---
 `A` | toggle displaying world axes 
 `R` | reset viewpoint
 `+`/`-` | adjust near plane clipping
 Space | switch to other operation mode
 `D` | switch to other display mode
 `L` | toggle lighting
 `Shift + L` | toggle between legacy and fancy rendering, i.e., screen-space ambient occlusion
`C` | toggle displaying camera trajectory
`S` | save current screen to image. The images are stored screenshots/ folder, named image000.png, image001.png, and so on. 
`P` | batch screenshot. Export screenshots for grey mesh, segmented mesh, bounding boxes. 
`Shift + S` | save (overwrite) current scene to PLY, XML, trajectory.log. 
`Shift + E` | save 2D labels (solid color). 
`Shift + B` | save 2D labels (blended with input color images).
 `Z` | undo
 `Q` | exit 

# Operation modes
The main operation mode is label interaction. This mode is designed for user to merge, extract, and split existing segments as described in the paper.

Additionally, when user clicks on a segment, the bounding box of the segment will be shown. If one clicks on the sides of the bounding box, its corresponding face will be highlighted. One can drag to adjust the size of the bounding boxes, which in turn discards all labels that are out of the bounding box from the selected segment. 

This program also comes with two other operation modes as follows.

- *Floor alignment mode*: usually the scene should have its floor aligned with the XZ axis, but the mesh from 3D reconstruction program might not have this property. Floor alignment mode is designed to address this issue. Press `G` to switch to this mode, and click on a segment on the floor to move the origin of the coordinate system there. 
Current implementation requires that the floor adjusted scene has to be saved, the program restarted before proceeding to other segmentation tasks. 

- *Object pose editing mode*: In this mode, the local XYZ axes for a segment is displayed. User can specify the front direction of the segment by aligning the local Z-axis accordingly. 

In this mode, press `X` to select the active axis (highlighted in bold), and `Shift + Left drag` to adjust the pose.

This is not the best UI for specifying object poses, but it's relatively simple to use. We are investigating more efficient approaches and will release when they are ready. Stay tuned!

# Display modes
The default display mode is segmentation, which shows the scene labels in colors. One can press `D` to switch to other display modes, namely, 

* Automatic segmentation using graphcut 
* Automatic segmentation using MRF.
* Grey mesh 
* Axis-aligned bounding boxes
* Oriented bounding boxes 

Press `S` or `P` to save the screenshots. 

# Recommended workflow

1. In the 2D window, check and see if 

* The 2D view and 3D view are the same when the slider is dragged.
* The labels are aligned well with the scene. If not, double check the camera intrinsics and trajectory.log as some parameters (e.g., focal length) might be set wrongly.
    You can also press `C` in the 3D window to display the locations of the camera poses. 

2. Merge labels based on objects first. This can be done in both the 2D and 3D window, but the 3D view is more easy to navigate.
Use `Shift + Left drag` and `Alt + Left drag` when necessary. Avoid using `Ctrl + Left drag` as much as possible as it can result in many segments that have to be merged again.

3. When done, switch to 2D window and scroll through the frames and double check.
 
4. Finally, in the 3D view, click on each object, switch to Annotation window and tag the segment with text.

5. Close the 3D window and save the results before quitting.
Or press `Shift + S` to save. 

# Tips

* Remember to observe the scenes at several views to make sure the occluded parts are segmented correctly. 

* The program has been quite stable in our use cases, but there is no guarantee that it won't crash. So please back up your annotated scenes and save regularly. 

* Be careful no not overwrite the _color.ply during save. Otherwise it has to be generated again which takes time.

* It is better not to merge labels and tag objects at the same time. We tried this workflow but found that it is not convenient with the current design of the program. Segment first. Annotate later.

* Please open new issue tickets on Github for questions and bug reports. Thanks!

# Loading Kinect v2 scenes 
By default, this program will attempt to look for `asus.ini` in the scene data folder. If this is not found, it will attempt to find `kinect2.ini`. When both files are not available, the program fallbacks to loading `ini/asus.ini` which is the default intrinsic parameters included with this program. 

Therefore, to load Kinect v2 scene, in the scene data folder, make sure that `kinect2.ini` is available. 
The structure of this file should be similar to `ini/asus.ini`, with image sizes set to resolution of Kinect v2. 
For current Kinect v2 scenes, we downsample all color images to the depth map resolution, which is to 512x424. 

# License 

Our annotation tool adopts various open-source projects including

* [RPly](http://w3.impa.br/~diego/software/rply/) for PLY read and write.
* [The VCG library](http://vcg.isti.cnr.it/vcglib/) for PLY read and write.
* SSAO shaders from [LearnOpenGL](http://learnopengl.com/#!Advanced-Lighting/SSAO) 
* [GLEW](http://glew.sourceforge.net/) for OpenGL extension.
* [Qt](https://www.qt.io/) for building UI. 

# Disclaimer 
Copyright (c) 2015-2017 Singapore University of Technology and Design

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
