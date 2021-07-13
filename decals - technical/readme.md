### Requisites

An interest in graphics is required for this article. If you have no graphics background then it may be hard to understand. As an example, I assume that you familiar with these definitions:
 - Albedo texture
 - Normal texture
 - Specular texture
 - Vertex shader
 - Fragment shader

And in general how imperative programming languages such as C, C++ or C# operate. You can use these articles to read up on this knowledge:
 - [Learn OpenGL: general introduction](https://learnopengl.com/Getting-started/Hello-Triangle)
 - [Learn OpenGL: Shaders](https://learnopengl.com/Getting-started/Shaders)
 - [Learn OpenGL: Textures](https://learnopengl.com/Lighting/Lighting-maps)

### Parts

 - [Technical bits]()

### Introduction

This article will look at the logic that runs behind decals. In specific we'll look at the shader logic and how it turns textures into the colors that we see on screen. We'll look at:
 - Albedo decals
 - Specular decals
 - Normal decals

### Vertex shaders

The task of the vertex shader is to transform data (such as geometry or coordinates) to the correct position in world space. For a general idea about this read the _The global picture_ chapter at:
 - [Learn OpenGL: Coordinate systems](https://learnopengl.com/Getting-started/Coordinate-Systems)

We will not discuss the vertex shader directly as it is mathematical in nature. What we are interested in is the output of the vertex shader - the data that we can use as part of the pixel shader.

```c
struct VS_OUTPUT
{
    float4 mPos             : POSITION0;    // position in screen space
    float4 mTexWT           : TEXCOORD1;    // uv position based on map size 
    float4 mTexSS           : TEXCOORD2;    // position in screen space
    float4 mShadow          : TEXCOORD3;    // position in the shadow map
    float3 mViewDirection   : TEXCOORD4;    // view direction for this location
    float4 mTexDecal        : TEXCOORD5;    // uv position in local decal coordinates
};
```

This information is available in the pixel shader. There are two types of data present:
 - `POSITIONX`: This is position data that represents a location in some space, such as world or screen space.
 - `TEXTUREX`: This is coordinate data that represents a location on some texture.

### Fragment shaders

The task of the fragment shader is to transform the output of the vertex shader and combine it with various other input, such as texture data. The output is a color for a pixel on the final image.

#### Albedo / Specular 

```c
float4 DecalAlbedoXP( VS_OUTPUT inV, uniform bool inShadows) : COLOR
{
    // sampling
    float4 albedo = tex2Dproj(DecalAlbedoSampler,inV.mTexDecal);
    float  specularAmount = tex2Dproj(DecalSpecSampler,inV.mTexDecal).a;
    float  mask = tex2Dproj(DecalMaskSampler,inV.mTexDecal).a;
    float3 normal = normalize(2*SampleScreen(NormalSampler,inV.mTexSS).xyz-1);

    // specular computations
    float3 r = reflect(normalize(inV.mViewDirection), normal);
    float3 specular = pow(saturate(dot(r,SunDirection)), 80) * specularAmount * SpecularColor.a * SpecularColor.rgb;

    // light computations
    float dotSunNormal = dot(SunDirection, normal);
    float  shadow = tex2D(ShadowSampler, inV.mShadow.xy).g;
    float3 light = SunColor*saturate(dotSunNormal) * shadow + SunAmbience;
    light = LightingMultiplier * light + ShadowFillColor * (1-light);
    albedo.rgb = light * ( albedo.rgb + specular.rgb );

    // water ramp
    float waterDepth = tex2Dproj(UtilitySamplerC, inV.mTexWT*TerrainScale).g;
    float4 water = tex1D(WaterRampSampler, waterDepth);
    albedo.rgb = lerp(albedo.rgb, water.rgb, water.a);

    // output
    return float4(albedo.rgb,DecalAlpha*albedo.a*mask);
}
```

#### Normals

### Albedo / specular decals

There are two interpretations of the albedo decal. This depends on the terrain shader that is chosen. There are two shaders:
 - `TTerrainXP`
 - `Terrain`

The `TTerrainXP` is the default shader in the Ozone editor and supports up to eight stratum layers. We'll focus our discussion on this shader.

There are two different interpretations of the albedo decal depending on the shader that is used on the terrain. A shader is a piece of code that transforms the input (geometry, textures) into pixels. It performs this in two phases:
 - Vertex shader: Transforms the geometry to match our camera viewpoint
 - Pixel shader: Combines the output of the vertex shader with various textures to produce pixels

We will not discuss the vertex shader as those require a mathematical background. Instead, we'll just name the output of the vertex shader and then assume that the information provided is valid.

```c
struct VS_OUTPUT
{
    float4 mPos             : POSITION0;    // position in screen space
    float4 mTexWT           : TEXCOORD1;    // uv position based on map size 
    float4 mTexSS           : TEXCOORD2;    // position in screen space
    float4 mShadow          : TEXCOORD3;    // position in the shadow map
    float3 mViewDirection   : TEXCOORD4;    // view direction for this location
    float4 mTexDecal        : TEXCOORD5;    // uv position in local decal coordinates
};
```

#### TTerrainXP Shader

```
    // sampling
    float4 albedo = tex2Dproj(DecalAlbedoSampler,inV.mTexDecal);
    float  specularAmount = tex2Dproj(DecalSpecSampler,inV.mTexDecal).a;
    float  mask = tex2Dproj(DecalMaskSampler,inV.mTexDecal).a;
    float3 normal = normalize(2*SampleScreen(NormalSampler,inV.mTexSS).xyz-1);
```

A shader can utilise textures as input. In this case we have:
 - A pixel in decal-space of the albedo texture in question (`inV.mTexDecal`)
 - A pixel in decal-space the specular texture in question (`inV.mTexDeca`)
 - A pixel in decal-space of the common decal mask
 - A pixel in world-space of the normal of the terrain where the pixel in question is in world space



#### Terrain Shader


Throughout this article we'll use the map Mellow Shallows as an example. 

### TTerrainXP Shader

```c
float4 TerrainAlbedoXP( VS_OUTPUT pixel) : COLOR
{
    float4 position = TerrainScale*pixel.mTexWT;

    // sample mask information
    float4 mask0 = saturate(tex2Dproj(UtilitySamplerA,position)*2-1);
    float4 mask1 = saturate(tex2Dproj(UtilitySamplerB,position)*2-1);

    // sample albedo information
    float4 lowerAlbedo = tex2Dproj(LowerAlbedoSampler,position*LowerAlbedoTile);
    float4 stratum0Albedo = tex2Dproj(Stratum0AlbedoSampler,position*Stratum0AlbedoTile);
    float4 stratum1Albedo = tex2Dproj(Stratum1AlbedoSampler,position*Stratum1AlbedoTile);
    float4 stratum2Albedo = tex2Dproj(Stratum2AlbedoSampler,position*Stratum2AlbedoTile);
    float4 stratum3Albedo = tex2Dproj(Stratum3AlbedoSampler,position*Stratum3AlbedoTile);
    float4 stratum4Albedo = tex2Dproj(Stratum4AlbedoSampler,position*Stratum4AlbedoTile);
    float4 stratum5Albedo = tex2Dproj(Stratum5AlbedoSampler,position*Stratum5AlbedoTile);
    float4 stratum6Albedo = tex2Dproj(Stratum6AlbedoSampler,position*Stratum6AlbedoTile);
    float4 stratum7Albedo = tex2Dproj(Stratum7AlbedoSampler,position*Stratum7AlbedoTile);
    float4 upperAlbedo = tex2Dproj(UpperAlbedoSampler,position*UpperAlbedoTile);

    // blend the albedo information
    float4 albedo = lowerAlbedo;
    albedo = lerp(albedo,stratum0Albedo,mask0.x);
    albedo = lerp(albedo,stratum1Albedo,mask0.y);
    albedo = lerp(albedo,stratum2Albedo,mask0.z);
    albedo = lerp(albedo,stratum3Albedo,mask0.w);
    albedo = lerp(albedo,stratum4Albedo,mask1.x);
    albedo = lerp(albedo,stratum5Albedo,mask1.y);
    albedo = lerp(albedo,stratum6Albedo,mask1.z);
    albedo = lerp(albedo,stratum7Albedo,mask1.w);
    albedo.rgb = lerp(albedo.xyz,upperAlbedo.xyz,upperAlbedo.w);

    // sample the normal (deferred renderer)
    float3 normal = normalize(2*SampleScreen(NormalSampler,pixel.mTexSS).xyz-1);
    
    // determine specularity
    float3 r = reflect(normalize(pixel.mViewDirection),normal);
    float3 specular = pow(saturate(dot(r,SunDirection)),80)*albedo.aaa*SpecularColor.a*SpecularColor.rgb;

    // determine typical dot-product lighting
    float dotSunNormal = dot(SunDirection,normal);

    // take into account shadows from units, combine all the lighting
    float shadow = tex2D(ShadowSampler,pixel.mShadow.xy).g;
    float3 light = SunColor*saturate(dotSunNormal)*shadow + SunAmbience;
    light = LightingMultiplier*light + ShadowFillColor*(1-light);
    albedo.rgb = light * ( albedo.rgb + specular.rgb );

    // add in water-depth coloring
    float waterDepth = tex2Dproj(UtilitySamplerC,pixel.mTexWT*TerrainScale).g;
    float4 water = tex1D(WaterRampSampler,waterDepth);
    albedo.rgb = lerp(albedo.rgb,water.rgb,water.a);

    // return it in the expected format
    return float4(albedo.rgb, 0.01f);
}
```