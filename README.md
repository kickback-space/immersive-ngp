# Immersive Neural Graphics Primitives

In this project, we present immersive NGP, the first open-source VR NERF Unity package that brings high resolution, low-latency, 6-DOF NERF rendering to VR. This work is based on Nvidia's ground breaking [instant-ngp](https://github.com/NVlabs/instant-ngp) technique. Current version uses [this commit](https://github.com/NVlabs/instant-ngp/commit/54aba7cfbeaf6a60f29469a9938485bebeba24c3) of instant-ngp.

Pointers to non-VR version: 

The repo runs on top of instant-ngp through a Unity native rendering plugin we developed. This [manuscript](https://www.edit.fis.uni-hamburg.de/ws/files/29623932/2023_IEEEVR_NERF_Unity_Camera_Ready.pdf) documents how Unity native rendering works together with instant-ngp. It is actually quite simple, since instant-ngp uses OpenGL backend, we only need to pass texture handle pointers from Unity to instant-ngp for instant-ngp to render images to these Unity textures directly.

Q: "Could you give us some pointers on what would have to change to make this repo work in non-VR settings? "
A: To develop for non-VR settings, feel free to refer to the [StereoNerfRenderer.cpp](https://github.com/uhhhci/immersive-ngp/blob/main/stereo-nerf-unity/Assets/NERF_NativeRendering/Scripts/StereoNerfRenderer.cs) and the [unity.cu](https://github.com/uhhhci/immersive-ngp/blob/main/instant-ngp/src/unity.cu) scripts. For VR application, the renderer has two render buffers to create stereoscopic images. I think for desktop applications, you could remove one render buffer, and convert the camera transform in Unity to a camera view matrix like in the StereoNerfRenderer.cpp.

## Features

* Stereoscopic, 6-DOF, real-time, low-latency NERF VR rendering in Unity. 
        <img src=".\images\stereo-nerf-demo.gif"
        alt="plugin-files.PNG"
        style="float: center; margin-right: 10px; height:300px;" />

* DLSS Support for rendering at higher framerate.

* 6-DOF continuous locomotion in VR.

* Offline volume image slices rendering via [Unity Volume Rendering Toolkit](https://github.com/mlavik1/UnityVolumeRendering).

* Integration with [MRTK 2.8](https://github.com/microsoft/MixedRealityToolkit-Unity) for building mixed reality applications with NERF. 

## Dependencies

* Unity 2019.4.29 ( Use the legacy XR manager for compatibility with OpenVR)
* [instant-ngp](https://github.com/NVlabs/instant-ngp)
* Unity OpenVR desktop plugin && SteamVR
* Microsoft Mixed Reality Toolkit MRTK 2.8 (already included in the Unity project)
* OpenGL Graphics API
* Current version of the repository was tested on Windows 10, Windows 11, using a Oculus Quest 2. 

## Installation

1. Clone this repository: ```git clone --recursive https://github.com/uhhhci/immersive-ngp```

2. Make sure you have all the dependencies for [instant-ngp](https://github.com/NVlabs/instant-ngp) installed before proceed.

3. Update dependencies for submodules

    ```
    git submodule sync --recursive
    git submodule update --init --recursive
    ```

4. Build the instant-ngp project, similar to the build process for the original instant-ngp project.

    ```
    cmake . -B build
    cmake --build build --config RelWithDebInfo -j
    ```

5. After succesful build, copy the following plugin files from ```\instant-ngp\build\``` folder to the ```\stereo-nerf-unity\Assets\Plugins\x86_64``` folder.

    <img src=".\images\plugin-files.PNG"
    alt="plugin-files.PNG"
    style="float: center; margin-right: 10px; height:150px;" />

5. Now instant-ngp can be loaded as native plugins via Unity.

## Usage for Immersive NERF Rendering
1. For Oculus Quest 2, Lunch Oculus Rift, and connect the headset to the PC via Link Cable, or Air Link. 
2. Launch SteamVR, make sure that SteamVR detects your headset and controllers. 
3. For a quick demo train a model using the fox scene via:
   ```
   build\testbed.exe --scene ..\data\nerf\fox
   ```
   and safe a snapshot of the instant-ngp model through Instant-ngp > Snapshot > Save 
5. Open the stereo-nerf-unity Unity project with Unity 2019.4.29. 
6. For a quick VR test of your own NERF scene, go to the ```Assets\NERF_NativeRendering\Scenes\XRTest``` scene.
7. Copy the path to your nerf model, images folder, and transform.json file to the ``` Stereo Nerf Renderer``` in the ```Nerf path``` parameters, as ilustrated below.
   
    <img src=".\images\stereo-nerf-gameobj.PNG"
    alt=".\images\stereo-nerf-gameobj.PNG"
    style="float: center; margin-right: 10px; height:300px;" />

    (Note: please generate the nerf model using [this instant-ngp commit](https://github.com/NVlabs/instant-ngp/commit/54aba7cfbeaf6a60f29469a9938485bebeba24c3) and above, or just use the instant-ngp instance included in this repo).

6. Adjust DLSS settings, and image resolution as you like. 
7. Now you can run the scene in Editor :)
8. Use the joystick of the VR controllers for locomotion. 

## Common Questions & Troubleshoot

1. **How to reach good framerate and lower latency**

    Beside having a good GPU, it is highly recommended to turn on DLSS support in Unity, also when building the native plugin. 

    [The instant-ngp commit](https://github.com/NVlabs/instant-ngp/commit/54aba7cfbeaf6a60f29469a9938485bebeba24c3) we use also allows saving aabb cropping in the pre-trained model snapshot. If you adjust the aabb cropping when training the model, it will saved and be loaded in Unity as well. Reducing aabb cropping could reduce the render volume, thus save some computational power. 

2. **Locomotion doesn't work.**

    Make sure that SteamVR detects both of your controllers before starting the scenes in the Editors. 

3. **How to change the FoV the image frame.**
    
    You can customize the FoV or aspect ratio of the stereo NERF image plane by modifying the x and y scale of the ```NERFLeft Image``` and the ```NERFRight Image``` game object. (Make sure to adjust both at the same time).

    <img src=".\images\stereo-aspect-ratio-adjustment.PNG"
    alt=".\images\stereo-nerf-gameobj.PNG"
    style="float: center; margin-right: 10px; height:200px;" />

4. **Is VR headset required to run the demo?**
    
    Yes, the main focus of this repository target immersive applications with a VR headset + steamVR. Please feel free to fork this repo and build immersive desktop applications. 

## Updates

2023/01/05: Major refractoring and fixed the issue that editor needs to be restarted on second run.

## Roadmap

* <s>Fix Editor restart issue</s>
* Foveated NERF : under consturction, coming soon ...
* Automatic Fov adjutsment: under consturction, coming soon ...
* Various interaction techniques (egocentric manipulation, etc) : under consturction, coming soon ...
* Time-warp algorithm for latency compensation: welcome contribution ...
* Dynamics Resolution: welcome contribution
* Support for OpenXR: welcome contribution
* Support for higher Unity Version: welcome contribution
* Real-time SLAM capture for dynamic grow dataset: welcome contribution
* .... feature suggestions welcome!


## Contributions

We welcome community contributions to this repository. 

## Thanks

Many thanks to the authors of these open-source repositories:

1. [instant-ngp](https://github.com/NVlabs/instant-ngp)
2. [Unity Volume Rendering](https://github.com/mlavik1/UnityVolumeRendering)
3. [Mixed Reality Toolkit](https://github.com/microsoft/MixedRealityToolkit-Unity)
4. [Unity Native Tool](https://github.com/mcpiroman/UnityNativeTool)


## Citations

```bibtex
@misc{immersive-ngp,
      doi = {10.48550/ARXIV.2211.13494},
      url = {https://arxiv.org/abs/2211.13494},
      author = {*Li, Ke and *Rolff, Tim and Schmidt, Susanne and Bacher, Reinhard and Frintrop, Simone and Leemans, Wim and Steinicke, Frank},
      title = {Immersive Neural Graphics Primitives},
      publisher = {arXiv},
      year = {2022}}
```
**\*These authors contributed equally to the work.** 

Link to [Arxiv paper]( https://arxiv.org/pdf/2211.13494.pdf)

Contact: ke.li1@desy.de, tim.rolff@uni-hamburg.de
 
## Acknowledgment

This work was supported by DASHH (Data Science in Hamburg - HELMHOLTZ Graduate School for the Structure of Matter) with the Grant-No. HIDSS-0002, and the German Federal Ministry of Education and Research (BMBF).

## License

Please check [here](LICENSE.txt) to view a copy of Nvidia's license for instant-ngp and for this repository.


