# KHR_lights_imageBased

## Contributors

* Gary Hsu, Microsoft
* Mike Bond, Adobe
* Norbert Nopper, UX3D
* Alexey Knyazev
* `Please add or remove!`

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec, requires `KHR_texture_cttf` extension, interacts with `EXT_texture_bc6h` extension.

## Overview

This extension provides the ability to define image-based lights in a glTF scene. Image-based lights consist of an environment map that represents specular radiance for the scene as well as irradiance information.

Many 3D tools and engines support image-based global illumination but the exact technique and data formats employed vary. Using this extension, tools can export and engines can import image-based lights and the result should be highly consistent. 

This extension specifies exactly one way to format and reference the environment map to be used. The goals of this are two-fold. First, it makes implementing support for this extension easier. Secondly, it ensures that rendering of the image-based lighting is consistent across runtimes.

A conforming implementation of this extension must be able to load the image-based environment data and render the PBR materials using this lighting.

## Defining an Image-Based Light

The `KHR_lights_imageBased` extension defines a array of image-based lights at the root of the glTF and then each scene can reference one. Each image-based light definition consists of a single cubemap that describes the specular radiance of the scene, the l=2 spherical harmonics coefficients for diffuse irradiance and rotation, brighness factor and offset values.

```json
"extensions": {
    "KHR_lights_imageBased" : {
        "imageBasedLights": [
            {
                "rotation": [0, 0, 0, 1],
                "brightnessFactor": 1.0,
                "brightnessOffset": 0.0,
                "specularEnvironmentTexture": 0,
                "diffuseSphericalHarmonics": [
                    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
                    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
                    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
                ]
            }
        ]
    }
}
```

## Specular BRDF integration and Irradiance Coefficients

This extension uses a prefiltered environment map to define the specular lighting utilizing the split sum approximation. More information:
- [Specular BRDF integration in Google Filament](https://google.github.io/filament/Filament.md.html#lighting/imagebasedlights/processinglightprobes)
- [Pre-Filtered Environment Map in Unreal Engine 4](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)

This extension uses spherical harmonic coefficients to define irradiance used for diffuse lighting. Coefficients are calculated for the first 3 SH bands (l=2) and take the form of a 9x3 array. More information:
- [Realtime Image Based Lighting using Spherical Harmonics](https://metashapes.com/blog/realtime-image-based-lighting-using-spherical-harmonics/)
- [An Efficient Representation for Irradiance Environment Maps](http://graphics.stanford.edu/papers/envmap/)

## Adding Light Instances to Scenes

Each scene can have a single IBL light attached to it by defining the `extensions` and `KHR_lights_imageBased` property and, within that, an index into the `imageBasedLights` array using the `imageBasedLight` property.

```json
"scenes" : [
    {
        "extensions" : {
            "KHR_lights_imageBased" : {
                "imageBasedLight" : 0
            }
        }
    }            
]
```

### Validation and implementation notes

- `specularEnvironmentTexture` must refer to a KTX2 image of **Cubemap** type as defined in KTX2, Section 4.3. Namely:
  - `pixelHeight` must be greater than 0.
  - `pixelDepth` must be 0.
  - `numberOfArrayElements` must be 0.
  - `numberOfFaces` must be 6.
- The image must contain mip levels.
- The color is encoded in linear space for all image formats.
- Formula in shader code:

```
finalSampledColor = sampledColor * brightnessFactor + brightnessOffset;
```

- Shader code does not need a change, when further formats are supported
  - Values larger than 1.0 possible
  - Negative values possible

### Encoding

#### Normal Quality

Data must be stored as a CTTF image with alpha channel conforming to RGBD (or RGBE, **TBD**) encoding.

#### High Quality (Optional)

Data must be stored as a KTX2 image with BC6H payload and linked via `EXT_texture_bc6h` extension.

#### Example with Two Options for the Same Texture

```json
"extensionsUsed": [
    "KHR_lights_imageBased", "KHR_texture_cttf", "KHR_image_ktx2", "EXT_texture_bc6h"
],
"extensionsRequired": [
    "KHR_lights_imageBased", "KHR_texture_cttf", "KHR_image_ktx2"
],
"textures": [
    {
        "name": "Specular Environment Texture",
        "extensions": {
            "KHR_texture_cttf": {
                "source": 0
            },
            "EXT_texture_bc6h": {
                "source": 1
            }

        }
    }
],
"images": [
    {
        "uri": "image_lo.ktx2",
        "extensions": {
            "KHR_image_ktx2": {
                "supercompressionScheme": -1, // TBD, CTTF
                "pixelWidth": 512,
                "pixelHeight": 512,
                "faces": 6,
                "levels": [
                    {
                        "offset": 1024,
                        "bytesOfImages": 10485,
                    }
                ]
            }
        }
    },
    {
        "uri": "image_hi.ktx2",
        "extensions": {
            "KHR_image_ktx2": {
                "vkFormat": 144, // VK_FORMAT_BC6H_SFLOAT_BLOCK
                "pixelWidth": 512,
                "pixelHeight": 512,
                "faces": 6,
                "levels": [
                    {
                        "offset": 1024,
                        "bytesOfImages": 10485,
                    }
                ]
            }
        }
    }
]
```

## glTF Schema Updates

* **JSON schema**: 
- [glTF.KHR_lights_imageBased.schema.json](schema/glTF.KHR_lights_imageBased.schema.json)
- [imageBasedLight.schema.json](schema/imageBasedLight.schema.json)
- [scene.schema.json](schema/scene.KHR_lights_imageBased.schema.json)

## Known Implementations

None.