# Getting Started with Vivify

*Bring your map to life.*

!!! note
    As a reminder, you MUST add "Vivify" as a requirement to your map to activate Vivify. Additionally, Vivify will only load on v3 maps.

## Setting up Unity

First, you should download the [Unity Hub](https://unity3d.com/get-unity/download). Beat Saber 1.29.1 uses version
*2019.4.28f* and Beat Saber 1.30.0+ uses *2021.3.16f1*. For maximum compatibility, you should use **2019.4.28f** found in
the [archive](https://unity3d.com/get-unity/download/archive) and use Swifter's [VivifyTemplate](https://github.com/Swifter1243/VivifyTemplate) to setup your project and build your
bundles.

!!! warning
    Due to changes in Unity between versions, 2019 bundles will have non-functioning shader keywords in 2021. VivifyTemplate runs an included [script by nicoco007](https://github.com/nicoco007/AssetBundleLoadingTools/blob/shader-keyword-rewriter/ShaderKeywordRewriter/Program.cs) that will automatically fix these keywords whenever you build using it.

Make sure you have `Virtual Reality Supported` enabled in your project and your stereo rendering mode is set to `Single Pass`. (Edit > Project Settings > Player > XR Settings > Deprecated Settings > Virtual Reality Supported). VivifyTemplate will do this for you.

## Writing VR shaders

Beat Saber v1.29.1 uses Single Pass Stereo rendering (See [Single Pass Stereo rendering](https://docs.unity3d.com/2019.4/Documentation/Manual/SinglePassStereoRendering.html) for more info). Use the unity
built-in function `UnityStereoTransformScreenSpaceTex` to fix your shaders in vr.

```hlsl
sampler2D _MainTex;

fixed4 frag (v2f i) : SV_Target
{
  return tex2D(_MainTex, UnityStereoTransformScreenSpaceTex(i.uv));
}
```

Beat Saber v1.30.0+ uses Single Pass Instanced rendering. Any incompatible shaders will only appear in the left eye. To
make your shader compatible with this vr rendering method, add instancing support to your shader.
See [Single-pass instanced rendering and custom shaders](https://docs.unity3d.com/Manual/SinglePassInstancing.html) for how to add instancing support. Look under "
Post-Processing shaders" to see how to sample a screen space texture.

!!! tip
    There are many commonly used structs/functions in UnityCG.cginc. As a few examples, `appdata_base`, `appdata_img`, `appdata_full`, and `v2f_img` can usually be used instead of writing your own structs and since most image effect shaders use the same vertex function, the include file has a `vert_img` that can be used with `#pragma vertex vert_img`.
    ??? example
        === "Before"
            ```hlsl linenums="1"
            #include "UnityCG.cginc"

            #pragma vertex vert
            #pragma fragment frag

            UNITY_DECLARE_SCREENSPACE_TEXTURE(_MainTex);

            struct appdata {
              float4 vertex : POSITION;
              float2 texcoord : TEXCOORD0;
              UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
              float4 pos : SV_POSITION;
              float2 uv : TEXCOORD0;
              UNITY_VERTEX_OUTPUT_STEREO
            };

            v2f vert (const appdata v)
            {
              UNITY_SETUP_INSTANCE_ID(v);
              UNITY_INITIALIZE_OUTPUT(v2f, v2f o);
              UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(o);
              o.pos = UnityObjectToClipPos(v.vertex);
              o.uv = v.texcoord;
              return o;
            }

            fixed4 frag (const v2f_img i) : SV_Target
            {
              UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(i);
              const fixed4 col = UNITY_SAMPLE_SCREENSPACE_TEXTURE(_MainTex, UnityStereoTransformScreenSpaceTex(i.uv));
              return fixed4(1 - col.rgb, col.a);
            }
            ```

        === "After"
            ```hlsl linenums="1"
            #include "UnityCG.cginc"

            #pragma vertex vert_img
            #pragma fragment frag

            UNITY_DECLARE_SCREENSPACE_TEXTURE(_MainTex);

            fixed4 frag (const v2f_img i) : SV_Target
            {
              UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(i);
              const fixed4 col = UNITY_SAMPLE_SCREENSPACE_TEXTURE(_MainTex, UnityStereoTransformScreenSpaceTex(i.uv));
              return fixed4(1 - col.rgb, col.a);
            }
            ```

## Creating an asset bundle

(Optional) See [Unity Asset Bundle Browser tool](https://docs.unity3d.com/2019.4/Documentation/Manual/AssetBundles-Browser.html). This tool allows you to browse the contents of
a built asset bundle.

Bundles should be placed in your map folder and called either `bundleWindows2019.vivify`, `bundleWindows2021.vivify`. Although Quest
support does not exist yet, bundles should still be built for them and be called `bundleAndroid2021.vivify`.

```
Map Folder
├── bundleWindows2019.vivify
├── bundleWindows2021.vivify
├── bundleAndroid2021.vivify
├── song.ogg
├── cover.jpg
├── ExpertPlusStandard.dat
└── info.dat
```

When referencing an asset's file path in an event, remember to write in all lower case. You can use the above Asset
Bundle Browser tool to see the path of specific assets.

By default, when Vivify will check against a checksum when loading an asset bundle. This checksum can be found in the file next to wherever you build your bundle in the form of `*.manifest` (VivifyTemplate will instead write a `bundleinfo.json`). You can add the checksum to the map by using the `"_assetBundle"` field in the info.dat.

!!! tip
    This checksum check can be disabled by enabling Debug Mode using the `-aerolunaisthebestmodder` launch parameter. Remember to make sure your map loads without this parameter.

```json
  ...
  "_environmentName": "DefaultEnvironment",
  "_allDirectionsEnvironmentName": "GlassDesertEnvironment",
  "_customData": {
    "_assetBundle": {
      "_windows2019": 1414251160,
      "_windows2021": 6436275894,
      "_android2021": 4262884586,
    }
  },
  "_difficultyBeatmapSets": [
    {
  ...
```
