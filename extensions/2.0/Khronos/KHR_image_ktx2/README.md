# KHR_image_ktx2

## Contributors

* Norbert Nopper, UX3D
* Alexey Knyazev
* Members of 3D Formats Working Group

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

This extension adds the ability to specify images using the [KTX2 file format](http://github.khronos.org/KTX-Specification/) (KTX2). An implementation of this extension can use the images provided in the KTX2 files as an alternative to the PNG or JPG images available in glTF 2.0. Furthermore, specifying images having advanced features such as mip map levels, cube map faces, or arrays is possible.

When the extension is used, it's allowed to use value `image/ktx2` for the `image.mimeType` property.

Also, a JSON form of the KTX2 header is provided in the extension object, so implementations can check compatibility and allocate GPU resources up front as well as directly pull needed sections of the KTX2 file instead of fetching them sequentially.

The following example shows an uncomressed RGBA8 2D image with swizzling metadata.

```json
"images": [
    {
        "uri": "texture.ktx2",
        "extensions": {
            "KHR_image_ktx2" : {
                "vkFormat": 37,
                "pixelWidth": 512,
                "pixelHeight": 512,
                "levels": [
                    {
                        "offset": 256,
                        "bytesOfImages": 1048576,
                    }
                ],
                "metadata": {
                    "KTXswizzle": "bgra"
                }
            }
        }
    }
]
```

### Validation and implementation notes

The information in the extension object must exactly match KTX2 header.

JSON type of each metadata entry depends on the type of the original value. Rules for KTX2-defined metadata keys are provided in the following table.

| Metadata Key | KTX2 Type | JSON Type |
|--------------|-----------|-----------|
| `KTXcubemapIncomplete` | `Uint8` | `integer` |
| `KTXorientation` | NUL-terminated string | `string` |
| `KTXglFormat` | `Uint32[3]` | `integer[3]` |
| `KTXdxgiFormat__` | `Uint32` | `integer` |
| `KTXmetalPixelFormat` | `Uint32` | `integer` |
| `KTXswizzle` | NUL-terminated string | `string` |
| `KTXwriter` | NUL-terminated string | `string` |
| `KTXastcDecodeRGB9E5` | no value | `true` |

This extension does not restrict texel formats or other features of the KTX2 images. Such limitations must be defined by other extensions referring to this extension.

> **Implementation Note:** The core glTF 2.0 spec allows only PNG and JPEG images, so references to KTX2 images can appear only in extension objects.


## glTF Schema Updates

* **JSON schema**: [image.KHR_image_ktx.schema.json](/schema/image.KHR_image_ktx.schema.json)

## Known Implementations

None.