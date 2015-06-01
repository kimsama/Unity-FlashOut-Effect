# FlashOut Effect

Unity3D shader and tween animation howto for flashing out effect on a 3D model.


<p align="center">
  <img src="https://github.com/kimsama/Unity-FlashOut-Effect/blob/master/image/flashout_shader.gif?raw=true" alt="TweenSlider"/>
</p>

Flash 효과 셰이더 
-----------------

Unity의 Diffuse 셰이더 코드를 아래와 같이 _FlashColor 및 _FlashAmount 프로퍼티를 가지도록 수정.

```
Shader "Custom/DiffuseFlash" 
{
	Properties {		
		_MainTex ("Base (RGB)", 2D) = "white" {}
		_Color ("Main Color", Color) = (1,1,1,1)
		_FlashColor ("Flash Color", Color) = (1, 1, 1, 1)
		_FlashAmount ("Flash Amount", Range(0.0, 1.0)) = 0.0
	}
	SubShader 
	{
		Tags { "RenderType"="Opaque" "IgnoreProjector"="False" }
		LOD 200

		Cull Off
		Blend One OneMinusSrcAlpha

		Pass
		{
	
			CGPROGRAM
			//#pragma surface surf Lambert
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			struct appdata_t
			{
		    	float4 vertex   : POSITION;
		    	float4 color    : COLOR;
		    	float2 texcoord : TEXCOORD0; 
			};

			struct v2f
			{
	    		float4 vertex  : SV_POSITION;
	    		fixed4 color   : COLOR;
	    		half2 texcoord : TEXCOORD0;
			};
	
			fixed4 _Color;
			fixed4 _FlashColor;
			float _FlashAmount;

			v2f vert (appdata_t IN)
			{
			    v2f OUT;
			    OUT.vertex = mul(UNITY_MATRIX_MVP, IN.vertex);
			    OUT.texcoord = IN.texcoord;
			    OUT.color = IN.color * _Color;

			    return OUT;
			}

			sampler2D _MainTex;
	
			fixed4 frag (v2f IN) : COLOR
			{
				fixed4 c = tex2D (_MainTex, IN.texcoord) * IN.color;
				c.rgb = lerp (c.rgb, _FlashColor.rgb, _FlashAmount);
				c.rgb *= c.a;
				return c;
			}
			ENDCG
		}
	}
	
	Fallback "Diffuse"
}
```

Fragment 셰이더에서 lerp 함수를 사용해서 설정된 _FlashAmount값에 따라 원본 텍스쳐의 색상값을 사용할 지, _FlashColor에 설정된 색상을 사용할지를 결정하는 부분이 핵심.

```
	fixed4 frag (v2f IN) : COLOR
	{
		fixed4 c = tex2D (_MainTex, IN.texcoord) * IN.color;
		c.rgb = lerp (c.rgb, _FlashColor.rgb, _FlashAmount);
		c.rgb *= c.a;
		return c;
	}
```


Tween 애니메이션
----------------

Flash-out 효과는 C# 코드에서 tween 애니메이션을 이용해서 설정. Tween 애니메이션은 [DOTween](http://dotween.demigiant.com/index.php)을 이용.

```csharp
	float amount = 0f;
	int blinkCount = 2; // 두 번 연속으로 flash-out

    ...
    
    Tweener tweener = DOTween.To(() => amount, x => amount = x, 0.75f, 0.1f);
    tweener.OnUpdate(() => { mat.SetFloat("_FlashAmount", amount); });
    tweener.SetLoops(blinkCount * 2, LoopType.Yoyo).SetEase(Ease.InOutQuad);

```

