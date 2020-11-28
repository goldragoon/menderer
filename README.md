# Menderer

**Menderer** is a OpenGL-based 3D triangle mesh renderer. The primary purpose of this project is batch rendering a 3D triangle mesh into a list of specified camera poses. This repository is modification of [Dr.Robert Maier's](https://github.com/robmaier) [Menderer](https://github.com/robmaier/menderer), for my own purpose. 
Menderer supports meshes in [PLY (ascii or binary)](https://en.wikipedia.org/wiki/PLY_(file_format)) format.
While various shadings (e.g. [Phong shading](https://en.wikipedia.org/wiki/Phong_shading)) are implemented, there is currently no support for texture/normal maps.
The tool is written in C++ and can be executed as a standalone command line application.

## Installation
Note that, currently the runtime environments that we are targeted & tested are, x86-64 based Windows and Linux systems.

### Dependencies
Menderer requires
[CMake >= 3.1.0](https://cmake.org/download/),
[OpenCV >= 4.3.0](https://opencv.org/releases.html),
[Eigen](http://eigen.tuxfamily.org/),
[CLI11](https://github.com/CLIUtils/CLI11/),
[GLAD >= 0.1.34](https://github.com/Dav1dde/glad/tags),
[GLFW3](https://www.glfw.org/),
[happly](https://github.com/nmwsharp/happly)
as third-party dependencies.
While Eigen, CLI11 and GLFW3 are already contained in the ```thirdparty``` folder, the other dependencies can be installed directly from the default Ubuntu repositories:
```
sudo apt install cmake libopencv-dev
```
If you are working on windows, consider to use [vcpkg](https://github.com/microsoft/vcpkg) as package manager.
### Build Menderer
To compile the Menderer application, use the standard CMake approach:
```
git clone https://github.com/goldragoon/menderer.git
mkdir build
cd build/
cmake ..
make
```

## Run Menderer
In the following, we provide a simple usage example for the data in folder ```data/lion/```.
First, an output folder ```output/``` is created:
```
# go into data folder
cd ../data/lion/
# create output folder
mkdir output/
```

### Simple example
We render the mesh ```mesh.ply``` into a virtual camera, which is specified by the pinhole camera parameters in ```intrinsics.txt``` and the target pose in ```pose.txt```:
```
../../build/bin/Menderer -c intrinsics.txt -t pose.txt -m mesh.ply -o output/
```
Please check the rendered output image ```output/render_000000-color.png```.

Instead of having only a single target camera pose, it is also possible to render the mesh into a list of camera poses ```trajectory.txt``` ([TUM RGB-D Benchmark trajectory format](https://vision.in.tum.de/data/datasets/rgbd-dataset/file_formats)):
```
../../build/bin/Menderer -c intrinsics.txt -t trajectory.txt -m mesh.ply -o output/
```
Again, the output images ```render_xxxxxx-color.png``` are generated in the ```output/``` subfolder.


### Intrinsic3D format
In addition to the format above (```intrinsics.txt``` and ```pose.txt```/```trajectory.txt```) , we also support [Intrinsic3D dataset folders](https://vision.in.tum.de/data/datasets/intrinsic3d).
We show an example for downloading and rendering the *Intrinsic3D Tomb Statuary* data in the following:

```
# go into data/ folder again
cd ../
# create folder for dataset
mkdir tomb
cd tomb/
# create output/ folder
mkdir output/

# download, extract and rename mesh
wget https://vision.in.tum.de/_media/data/datasets/intrinsic3d/tomb-statuary-intrinsic3d.zip
unzip tomb-statuary-intrinsic3d.zip
mv tomb-statuary-intrinsic3d.ply mesh.ply

# download, extract and rename rgbd (with intrinsics and poses)
wget https://vision.in.tum.de/_media/data/datasets/intrinsic3d/tomb-statuary-rgbd.zip
unzip tomb-statuary-rgbd.zip
mv tomb-statuary-rgbd rgbd
```

After preparing the dataset, we can batch-render the mesh into the provided input camera poses:
```
../../build/bin/Menderer -d rgbd/ -m mesh.ply -o output/
```

### Mesh format
Since Menderer can only load meshes from .ply files, we recommend [Meshlab](http://www.meshlab.net/) for converting meshes in other formats (e.g. .obj, .wrl, etc.) to the .ply format.


### Command line arguments
There are various command line options for the ```Menderer``` application in order to adjust the renderings and output options.
```
Input parameters (mandatory):
-c,--camera             Camera intrinsics filename (file must exist).
-t,--trajectory         Camera trajectory file in TUM RGB-D benchmark format (file must exist).
-d,--dataset"           Dataset folder in Intrinsic3D format (folder must exist).
                        Either both options -c and -t or just option -d must be specified.
-m,--mesh"              Input mesh file (file must exist).

Output parameters (optional):
-o,--output             Output folder (folder must exist and must be empty).
Output flags (optional, without arguments):
--save_depth_png        Save rendered depth (.png files) in output folder.
                        (Divide by scale factor 5000.0 to get metric depth)
--save_depth_binary     Save rendered depth (.bin files) in output folder.
--save_mesh             Triangulate rendered depth and save generated mesh
                        as .ply file in output folder.

GUI flags (optional, without arguments):
--gui                   Show GUI for rendered color
--pause                 Pause after each frame (continue with any button/space)

Renderer parameters (optional):
--shader                OpenGL rendering shader (options: "none", 
                        "normals_phong" (default), "phong", "normals").
--color_r               Mesh color (red channel).
--color_b               Mesh color (blue channel).
--color_g               Mesh color (green channel).
--bg_r                  Rendering background color (red channel).
--bg_b                  Rendering background color (blue channel).
--bg_g                  Rendering background color (green channel).

Renderer flags (optional, without arguments):
--lighting      Enable lighting (default false).
--colored       Enable mesh colors (default false).
--flat          Enable flat rendering (default false, i.e. smooth).
```

### Example rendering modes
The following command line options demonstrate the use of implemented rendering modes:
```
1) Phong shading with normals as surface colors (default):
--shader phong_normals

2) Phong shading with uniform (gray) surface colors:
--shader phong --color_r 0.5 --color_g 0.5 --color_b 0.5

3) Render vertex colors without geometry/lighting:
--shader none --colored

4) Render normals (without Phong shading and geometry/lighting):
--shader normals
```


### Load exported depth in Matlab
When the ```--save_depth_binary``` flag is specified, the binary depth images ```render_xxxxxx-depth.bin``` are additionally generated in the ```output/``` subfolder.
The following code snippet shows how to load a binary depth map in Matlab (assumed the rendering resolution is 640x480).
```
% read file into matrix A
fileID = fopen('out_0.bin');
A = fread(fileID,[640, 480],'float');
fclose(fileID);
% permute dimensions (account for row-major/column-major)
A = permute(A,[2 1]);
% convert A to grayscale image
I = mat2gray(A);
% show image
imshow(I);
```


## License
The Menderer source code is licensed under the [GNU General Public License Version 3 (GPLv3)](http://www.gnu.org/licenses/gpl.html), please see the [LICENSE](LICENSE) file for details.


## Contact
If you have any questions, please contact [Gyu Jin Choi &lt;paganinist@gmail.com>](paganinist@gmail.com).
