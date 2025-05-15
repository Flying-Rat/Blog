---
title: "Correct Motion Vectors on Cutout Shaders"
date: 2025-06-01T16:00:00+02:00
author: Tomáš Bubeníček
categories:
  - unity
tags:
  - unity
  - rendering
  - built-in render pipeline
  - post-processing
  - shaders
toc: true
---

## Introduction

Motion vectors are very important in modern rendering. Real-time rendering engines capture static moments in time, discarding the visual information between frames. This is different from physical cameras, which record light over a duration, and from display technology, which typically alsi shows each static frame continuously over a longer duration until the next one appears (except for technologies like backlight strobing). This disconnect between discrete rendering and continuous display creates the need for some form of motion blur to reconstruct movement information.

The closest approximation of the real motion blur that would occur when capturing an image for the duration of one frame is by using motion vectors, representing direction each point on screen moved the two discrete frames, and blurring along the path that point took. There are many cases where motion blur is not done well, which can destroy enjoyment of games. But when done right, motion blur can remedy one of the core problems in modern real-time rendering. Also, motion blur can be used creatively for effects like extreme speed (most racing games) or slow motion (ala Max Payne 3).

Even if you don't use motion blur, it's still important to correctly render motion vectors. They are key for modern image improvement and reconstruction technologies like Nvidia's DLSS, AMD's FSR2, and Intel's XeSS. Having incorrect motion vectors (or objects with no motion vectors) will often create visible artifacting when using those technologies - One example is missing motion vectors on particles in Death Stranding causing them to appear [with very long black lines behind them](https://old.reddit.com/r/DeathStranding/comments/kglqve/extreme_ghosting_with_dlss/) when DLSS is enabled.

Hopefully this is a good enough motivation why motion vectors are useful, and why caring about their correctness may be important even when users decide to turn off motion blur.

### Implementations in Unity

The HD Render Pipeline has always had high-quality motion blur. If you're using it, you're golden :)

The Universal Render Pipeline (URP) added proper per-object motion blur in URP 15 for Unity 2023. Before that, it only had camera-based motion blur, which doesn't use motion vectors and just blurs based on camera movement, assuming no objects moved between frames. If you want proper motion blur, you'll need to update your Unity version.

Unity's built-in render pipeline has supported motion blur with the post-processing v2 stack since forever. Even though it doesn't receive much active development from Unity, it's still used even on newer projects due to variety of reasons, and of course, some games are in development for very long time and cannot afford solving the issues caused by completely replacing a rendering pipeline late in development.


## Built-In troubles

The way motion vectors are generated is by rendering the scene again with a special shader that renders the motion vector information to a single R16G16_SFloat texture, which represents the shift of position each point on the texture that happened between the current and the previous frame. For each MeshRenderer that has "Motion Vectors" set as "Per Object Motion", Unity looks into their shader for a Pass with `Tags{ "LightMode" = "MotionVectors" }` set, and uses it for rendering the motion vectors into the offscreen buffer. If it doesn't find this lightmode, or if the MeshRenderer motion vector setting is set to "Camera Motion Only", it falls back to the internal `Hidden/Internal-MotionVectors` shader, which implements basic rendering of motion vectors for fully opaque geometry. This is useful, since you don't have to implement your own solution every time you write a custom shader, or even fallback to a shader which has this implemented. It also makes sense to limit this to opaque geometry only, because compared to final image, where one pixel can represent several transparent surfaces behind one another, a single pixel in the motion vector buffer can only represent the movement of a single point.

The issue you might encounter though is with cutout transparency. In that case, we don't render the entire opaque geometry, but only those parts where the alpha channel of the texture is above a cutoff threshold gets shown.

On HDRP and URP, their standard shaders have their own specific implementation of MotionVectors LightMode which handles the special case where cutout transparency is used.

But when using built-in render pipeline, the Standard shader doesn't have its own implementation and always falls back to the internal, completely opaque one. This is fine in most cases, but when you have a moving object with cutout transparency, the motion vectors get completely broken in the transparent region, resulting in broken motion blur and potential issues with DLSS.

TODO ADD IMAGES

## Simple solution

The only solution for this is to write your own shader that specifically implements the `Tags{ "LightMode" = "MotionVectors" }` that samples the texture and properly handles alpha cutoff. Luckily, Unity distributes all of the internal built-in shaders under MIT license and they can be downloaded for free from [Unity download archive](https://unity.com/releases/editor/archive). Just go to "See all", click on "Others", and look for "Shaders".

We're interested in the `Internal-MotionVectors.shader` located under the `DefaultResourcesExtra` folder. We'll basically copy it into our own project, use only the Pass handling per object motion vectors, and make it behave exactly like the regular built-in Standard shader by only implementing this specific pass and adding a `FallBack "Standard"` line at the end.

The only thing that needs adding is the Properties block from the standard shader, and some extra logic so the MotionVectors pass has access to the color, texture and transformed UVs.

```cs
// Unity built-in shader source. Copyright (c) 2016 Unity Technologies. MIT license (see license.txt)

Shader "StandardCutoutMotionVectors"
{
//BEGIN(Tomas.Bubenicek): Properties taken directly from Standard.shader
    Properties
    {
        _Color("Color", Color) = (1,1,1,1)
        _MainTex("Albedo", 2D) = "white" {}

        _Cutoff("Alpha Cutoff", Range(0.0, 1.0)) = 0.5

        _Glossiness("Smoothness", Range(0.0, 1.0)) = 0.5
        _GlossMapScale("Smoothness Scale", Range(0.0, 1.0)) = 1.0
        [Enum(Metallic Alpha,0,Albedo Alpha,1)] _SmoothnessTextureChannel ("Smoothness texture channel", Float) = 0

        [Gamma] _Metallic("Metallic", Range(0.0, 1.0)) = 0.0
        _MetallicGlossMap("Metallic", 2D) = "white" {}

        [ToggleOff] _SpecularHighlights("Specular Highlights", Float) = 1.0
        [ToggleOff] _GlossyReflections("Glossy Reflections", Float) = 1.0

        _BumpScale("Scale", Float) = 1.0
        [Normal] _BumpMap("Normal Map", 2D) = "bump" {}

        _Parallax ("Height Scale", Range (0.005, 0.08)) = 0.02
        _ParallaxMap ("Height Map", 2D) = "black" {}

        _OcclusionStrength("Strength", Range(0.0, 1.0)) = 1.0
        _OcclusionMap("Occlusion", 2D) = "white" {}

        _EmissionColor("Color", Color) = (0,0,0)
        _EmissionMap("Emission", 2D) = "white" {}

        _DetailMask("Detail Mask", 2D) = "white" {}

        _DetailAlbedoMap("Detail Albedo x2", 2D) = "grey" {}
        _DetailNormalMapScale("Scale", Float) = 1.0
        [Normal] _DetailNormalMap("Normal Map", 2D) = "bump" {}

        [Enum(UV0,0,UV1,1)] _UVSec ("UV Set for secondary textures", Float) = 0


        // Blending state
        [HideInInspector] _Mode ("__mode", Float) = 0.0
        [HideInInspector] _SrcBlend ("__src", Float) = 1.0
        [HideInInspector] _DstBlend ("__dst", Float) = 0.0
        [HideInInspector] _ZWrite ("__zw", Float) = 1.0
    }
CGINCLUDE
  #define UNITY_SETUP_BRDF_INPUT MetallicSetup
ENDCG
//END(Tomas.Bubenicek)

    SubShader
    {
        CGINCLUDE
        #include "UnityCG.cginc"

        // Object rendering things

#if defined(USING_STEREO_MATRICES)
        float4x4 _StereoNonJitteredVP[2];
        float4x4 _StereoPreviousVP[2];
#else
        float4x4 _NonJitteredVP;
        float4x4 _PreviousVP;
#endif
        float4x4 _PreviousM;
        bool _HasLastPositionData;
        bool _ForceNoMotion;
        float _MotionVectorDepthBias;


// BEGIN(Tomas.Bubenicek)
// Texture and color access needed for transparent cutout materials
#if defined(_ALPHATEST_ON)
        uniform fixed4 _Color;
        uniform sampler2D _MainTex;
        uniform float4 _MainTex_ST;
        uniform fixed _Cutoff;
#endif
//END(Tomas.Bubenicek)

        struct MotionVectorData
        {
            float4 transferPos : TEXCOORD0;
            float4 transferPosOld : TEXCOORD1;
            float4 pos : SV_POSITION;
//BEGIN(Tomas.Bubenicek)
#if defined(_ALPHATEST_ON)
            float2 uv: TEXCOORD2;
#endif
//END(Tomas.Bubenicek)
            UNITY_VERTEX_OUTPUT_STEREO
        };

        struct MotionVertexInput
        {
            float4 vertex : POSITION;
            float3 oldPos : TEXCOORD4;
//BEGIN(Tomas.Bubenicek)
#if defined(_ALPHATEST_ON)
            float2 texcoord : TEXCOORD0;
#endif
//END(Tomas.Bubenicek)
            UNITY_VERTEX_INPUT_INSTANCE_ID
        };

        MotionVectorData VertMotionVectors(MotionVertexInput v)
        {
            MotionVectorData o;
            UNITY_SETUP_INSTANCE_ID(v);
            UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(o);
            o.pos = UnityObjectToClipPos(v.vertex);

            // this works around an issue with dynamic batching
            // potentially remove in 5.4 when we use instancing
#if defined(UNITY_REVERSED_Z)
            o.pos.z -= _MotionVectorDepthBias * o.pos.w;
#else
            o.pos.z += _MotionVectorDepthBias * o.pos.w;
#endif

#if defined(USING_STEREO_MATRICES)
            o.transferPos = mul(_StereoNonJitteredVP[unity_StereoEyeIndex], mul(unity_ObjectToWorld, v.vertex));
            o.transferPosOld = mul(_StereoPreviousVP[unity_StereoEyeIndex], mul(_PreviousM, _HasLastPositionData ? float4(v.oldPos, 1) : v.vertex));
#else
            o.transferPos = mul(_NonJitteredVP, mul(unity_ObjectToWorld, v.vertex));
            o.transferPosOld = mul(_PreviousVP, mul(_PreviousM, _HasLastPositionData ? float4(v.oldPos, 1) : v.vertex));
#endif

//BEGIN(Tomas.Bubenicek)
#if defined(_ALPHATEST_ON)
            o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
#endif
//END(Tomas.Bubenicek)
            return o;
        }

        half4 FragMotionVectors(MotionVectorData i) : SV_Target
        {
//BEGIN(Tomas.Bubenicek) Clip if we're below the alpha cutoff threshold
#if defined(_ALPHATEST_ON)
            fixed4 texcol = tex2D( _MainTex, i.uv );
            clip( texcol.a*_Color.a - _Cutoff );
#endif
//END(Tomas.Bubenicek)

            float3 hPos = (i.transferPos.xyz / i.transferPos.w);
            float3 hPosOld = (i.transferPosOld.xyz / i.transferPosOld.w);

            // V is the viewport position at this pixel in the range 0 to 1.
            float2 vPos = (hPos.xy + 1.0f) / 2.0f;
            float2 vPosOld = (hPosOld.xy + 1.0f) / 2.0f;

#if UNITY_UV_STARTS_AT_TOP
            vPos.y = 1.0 - vPos.y;
            vPosOld.y = 1.0 - vPosOld.y;
#endif
            half2 uvDiff = vPos - vPosOld;
            return lerp(half4(uvDiff, 0, 1), 0, (half)_ForceNoMotion);
        }

        //Camera rendering things
        UNITY_DECLARE_DEPTH_TEXTURE(_CameraDepthTexture);

        struct CamMotionVectors
        {
            float4 pos : SV_POSITION;
            float2 uv : TEXCOORD0;
            float3 ray : TEXCOORD1;
            UNITY_VERTEX_OUTPUT_STEREO
        };

        struct CamMotionVectorsInput
        {
            float4 vertex : POSITION;
            float3 normal : NORMAL;
            UNITY_VERTEX_INPUT_INSTANCE_ID
        };


        inline half2 CalculateMotion(float rawDepth, float2 inUV, float3 inRay)
        {
            float depth = Linear01Depth(rawDepth);
            float3 ray = inRay * (_ProjectionParams.z / inRay.z);
            float3 vPos = ray * depth;
            float4 worldPos = mul(unity_CameraToWorld, float4(vPos, 1.0));

#if defined(USING_STEREO_MATRICES)
            float4 prevClipPos = mul(_StereoPreviousVP[unity_StereoEyeIndex], worldPos);
            float4 curClipPos = mul(_StereoNonJitteredVP[unity_StereoEyeIndex], worldPos);
#else
            float4 prevClipPos = mul(_PreviousVP, worldPos);
            float4 curClipPos = mul(_NonJitteredVP, worldPos);
#endif
            float2 prevHPos = prevClipPos.xy / prevClipPos.w;
            float2 curHPos = curClipPos.xy / curClipPos.w;

            // V is the viewport position at this pixel in the range 0 to 1.
            float2 vPosPrev = (prevHPos.xy + 1.0f) / 2.0f;
            float2 vPosCur = (curHPos.xy + 1.0f) / 2.0f;
#if UNITY_UV_STARTS_AT_TOP
            vPosPrev.y = 1.0 - vPosPrev.y;
            vPosCur.y = 1.0 - vPosCur.y;
#endif
            return vPosCur - vPosPrev;
        }
        ENDCG

        // 0 - Motion vectors
        Pass
        {
            Tags{ "LightMode" = "MotionVectors" }

            ZTest LEqual
            Cull Back
            ZWrite Off

            CGPROGRAM

//BEGIN(Tomas.Bubenicek): Only use the new path when we actually have alphatest enabled
            #pragma shader_feature_local _ _ALPHATEST_ON
//END(Tomas.Bubenicek)

            #pragma vertex VertMotionVectors
            #pragma fragment FragMotionVectors
            ENDCG
        }

//NOTE(Tomas.Bubenicek): Also removed the passes and functionality handling Camera only motion vectors that was in Internal-MotionVectors.shader, Unity will default to the internal shader anyways
    }

//BEGIN(Tomas.Bubenicek): Behave like standard shader would
    FallBack "Standard"
    CustomEditor "StandardShaderGUI"
//END(Tomas.Bubenicek)
}
```
