# Toon_crystal
一个移动端可用的水晶体材质
![image](https://github.com/junyangtong/Toon_crystal/assets/135015047/1a853cfb-306c-4ce3-a512-26191383974c)
在SD中制作贴图
![image](https://github.com/junyangtong/Toon_crystal/assets/135015047/9353ea98-e65a-46bf-aa60-b98e34b555d6)

Unity shader
1. 基础光照
整体思路：
finalRGB = 环境漫反射+光照漫反射+环境镜面反射+光照镜面反射+自发光
环境漫反射：
half3 ambient =  _AmbientCol;
光照漫反射：兰伯特+ 阴影
half3 baseCol = var_MainTex.rgb * ambient ;
half lambert = max(0.0,nDotl);
half Shadow = LIGHT_ATTENUATION(i);
环境镜面反射：菲涅尔+反射探头 + ao
vrDirWS = BoxProjectedCubemapDirection(vrDirWS, i.posWS, unity_SpecCube0_ProbePosition, unity_SpecCube0_BoxMin, unity_SpecCube0_BoxMax);
 half4 rgbm = UNITY_SAMPLE_TEXCUBE(unity_SpecCube0, vrDirWS);
half3 reflection = DecodeHDR(rgbm, unity_SpecCube0_HDR);
half3 fresnel = max(0.0,1-nDotv) * _FresnelPow * var_MainTex.rgb;
half3 diffenvCol = aocol * _AOInt + fresnel * _AmbientCol;
光照镜面反射：blinn-phong
half BlinPhong = pow(max(0.0,nDoth),rough * _SpecPow);
half spec = BlinPhong * max(0.0,nDotl);
spec = spec * _SpecInt;
half3 DirSpec = var_MainTex.rgb * spec * _LightColor0;
自发光：
half3 emis = var_EmitTex.r * _EmitInt;

1. 视差贴图
为了增加模型的通透感，采样了两次视差贴图
half BumpColor= tex2D(_MainTex, (-i.uv0 * _MainTex_ST.xy + _MainTex_ST.zw) + i.offset*0.4).a;
diff = diff + BumpColor;
第一次采样了一次ao，让水晶表面的裂纹层次更加丰富

第二次采样增加了水晶内部的杂质颗粒

性能优化
1.尽可能的合并贴图减少贴图采样次数
2.贴图尺寸不超过512
3.不使用HDR颜色参数
4.不使用高等精度的数据类型
