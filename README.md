# FlashOut 이펙트

RPG 게임에서 그림과 같이 피격시 데미지 효과 처리를 위해 사용하는 플래쉬 효과를 위한 셰이더와 트위닝 애니메이션에 대한 설명입니다. 


<p align="center">
  <img src="https://github.com/kimsama/Unity-FlashOut-Effect/blob/master/image/flashout_shader.gif?raw=true" alt="FlashOutEffect"/>
</p>

Flash 효과 셰이더 
-----------------

Unity의 Diffuse 셰이더 코드를 아래와 같이 _FlashColor 및 _FlashAmount 프로퍼티를 가지도록 수정한다.

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
        
        //any other setting if it needs...

		Blend One OneMinusSrcAlpha

		Pass
		{
	
			CGPROGRAM
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

Fragment 셰이더에서 lerp 함수를 사용해서 설정된 _FlashAmount값에 따라 원본 텍스쳐의 색상값을 사용할 지, _FlashColor에 설정된 색상을 사용할지를 결정한다. _FlashAmount 값이 0이면 완전한 원본 텍스쳐 색상을 사용, 1이면 _FlashColor에 설정된 색상으로 픽셀값을 결정한다.

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

Flash-out 효과는 C# 코드에서 tween 애니메이션을 이용해서 손쉽게 설정할 수 있다. 여기에서는 Tween 애니메이션으로 [DOTween](http://dotween.demigiant.com/index.php)을 이용한다.

```csharp
    using DG.Tweening;

    ...

	float amount = 0f;
	int blinkCount = 2; // 두 번 연속으로 flash-out

    ...

    Tweener tweener = DOTween.To(() => amount, x => amount = x, 0.75f, 0.1f);
    tweener.OnUpdate(() => { mat.SetFloat("_FlashAmount", amount); });
    tweener.SetLoops(blinkCount * 2, LoopType.Yoyo).SetEase(Ease.InOutQuad);

```

_FlashColor 값에 설정된 색상값으로 tween 애니메이션하면서 변경된 값을 Material.SetFloat() 함수를 시용해서 셰이더쪽에 적용하면 그림에서와 같은 효과가 나타난다. 

깜박이는 효과를 위해 LoopType을 Yoyo로 설정하고 yoyo 처리를 위해서는 loop 회수가 깜박임 회수의 2배가 되도록 설정해야 한다. 


(c)2015 Kim, Hyoun Woo

Note: The model used for the image borrowed from Torchlight, Runic Games.
