# WebGL 着色器编程

> 📅 创建时间：2026-05-30
> 🎯 目标：掌握 WebGL 着色器编程

---

## 一、GLSL 基础

### 1.1 数据类型
```glsl
// 标量
float a = 1.0;
int b = 1;
bool c = true;

// 向量
vec2 v2 = vec2(1.0, 2.0);
vec3 v3 = vec3(1.0, 2.0, 3.0);
vec4 v4 = vec4(1.0, 2.0, 3.0, 4.0);

// 矩阵
mat2 m2 = mat2(1.0, 2.0, 3.0, 4.0);
mat3 m3 = mat3(1.0);
mat4 m4 = mat4(1.0);

// 纹理采样器
sampler2D tex;

// 访问向量分量
float x = v3.x;  // 或 v3.r, v3.s
float y = v3.y;  // 或 v3.g, v3.t
float z = v3.z;  // 或 v3.b, v3.p

// 分量重排
vec3 swizzled = v3.zyx;  // (z, y, x)
vec2 xy = v4.xy;
```

### 1.2 内置函数
```glsl
// 数学函数
float abs(float x);        // 绝对值
float floor(float x);      // 向下取整
float ceil(float x);       // 向上取整
float fract(float x);      // 小数部分
float mod(float x, float y); // 取模
float min(float x, float y); // 最小值
float max(float x, float y); // 最大值
float clamp(float x, float min, float max); // 钳制
float mix(float x, float y, float a); // 线性插值
float step(float edge, float x); // 阶梯函数
float smoothstep(float edge0, float edge1, float x); // 平滑阶梯

// 三角函数
float sin(float x);
float cos(float x);
float tan(float x);
float asin(float x);
float acos(float x);
float atan(float y, float x);

// 几何函数
float length(vec3 v);      // 长度
float distance(vec3 a, vec3 b); // 距离
float dot(vec3 a, vec3 b); // 点积
vec3 cross(vec3 a, vec3 b); // 叉积
vec3 normalize(vec3 v);    // 归一化
vec3 reflect(vec3 I, vec3 N); // 反射
vec3 refract(vec3 I, vec3 N, float eta); // 折射

// 纹理采样
vec4 texture2D(sampler2D sampler, vec2 coord);
vec4 textureCube(samplerCube sampler, vec3 coord);
```

---

## 二、顶点着色器

### 2.1 基础顶点着色器
```glsl
// 属性（从顶点缓冲区读取）
attribute vec3 aPosition;
attribute vec2 aTexCoord;
attribute vec3 aNormal;

// Uniform（整个绘制调用保持不变）
uniform mat4 uModelMatrix;
uniform mat4 uViewMatrix;
uniform mat4 uProjectionMatrix;
uniform mat3 uNormalMatrix;

// 变量（传递给片段着色器）
varying vec2 vTexCoord;
varying vec3 vNormal;
varying vec3 vWorldPosition;

void main() {
    // 计算世界坐标
    vec4 worldPosition = uModelMatrix * vec4(aPosition, 1.0);
    vWorldPosition = worldPosition.xyz;
    
    // 计算裁剪坐标
    gl_Position = uProjectionMatrix * uViewMatrix * worldPosition;
    
    // 传递纹理坐标
    vTexCoord = aTexCoord;
    
    // 变换法线
    vNormal = uNormalMatrix * aNormal;
}
```

### 2.2 骨骼动画顶点着色器
```glsl
attribute vec3 aPosition;
attribute vec2 aTexCoord;
attribute vec3 aNormal;
attribute vec4 aBoneIndices;
attribute vec4 aBoneWeights;

uniform mat4 uBoneMatrices[64];
uniform mat4 uModelMatrix;
uniform mat4 uViewMatrix;
uniform mat4 uProjectionMatrix;

varying vec2 vTexCoord;
varying vec3 vNormal;

void main() {
    // 骨骼变换
    mat4 boneMatrix = 
        uBoneMatrices[int(aBoneIndices.x)] * aBoneWeights.x +
        uBoneMatrices[int(aBoneIndices.y)] * aBoneWeights.y +
        uBoneMatrices[int(aBoneIndices.z)] * aBoneWeights.z +
        uBoneMatrices[int(aBoneIndices.w)] * aBoneWeights.w;
    
    // 变换位置
    vec4 position = boneMatrix * vec4(aPosition, 1.0);
    
    // 变换法线
    mat3 normalMatrix = mat3(boneMatrix);
    vNormal = normalMatrix * aNormal;
    
    // 纹理坐标
    vTexCoord = aTexCoord;
    
    // 最终位置
    gl_Position = uProjectionMatrix * uViewMatrix * uModelMatrix * position;
}
```

---

## 三、片段着色器

### 3.1 基础片段着色器
```glsl
precision mediump float;

varying vec2 vTexCoord;
varying vec3 vNormal;
varying vec3 vWorldPosition;

uniform sampler2D uTexture;
uniform vec3 uLightPosition;
uniform vec3 uCameraPosition;

void main() {
    // 采样纹理
    vec4 texColor = texture2D(uTexture, vTexCoord);
    
    // 归一化法线
    vec3 normal = normalize(vNormal);
    
    // 光照方向
    vec3 lightDir = normalize(uLightPosition - vWorldPosition);
    
    // 漫反射
    float diffuse = max(dot(normal, lightDir), 0.0);
    
    // 镜面反射
    vec3 viewDir = normalize(uCameraPosition - vWorldPosition);
    vec3 reflectDir = reflect(-lightDir, normal);
    float specular = pow(max(dot(viewDir, reflectDir), 0.0), 32.0);
    
    // 环境光
    float ambient = 0.1;
    
    // 最终颜色
    vec3 color = texColor.rgb * (ambient + diffuse) + vec3(specular);
    
    gl_FragColor = vec4(color, texColor.a);
}
```

### 3.2 PBR 材质着色器
```glsl
precision highp float;

varying vec2 vTexCoord;
varying vec3 vNormal;
varying vec3 vWorldPosition;

uniform sampler2D uAlbedoMap;
uniform sampler2D uNormalMap;
uniform sampler2D uMetallicRoughnessMap;
uniform sampler2D uEmissiveMap;

uniform vec3 uLightPositions[4];
uniform vec3 uLightColors[4];
uniform vec3 uCameraPosition;

const float PI = 3.14159265359;

// 法线分布函数 (GGX)
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a = roughness * roughness;
    float a2 = a * a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH * NdotH;
    
    float nom = a2;
    float denom = NdotH2 * (a2 - 1.0) + 1.0;
    denom = PI * denom * denom;
    
    return nom / denom;
}

// 几何遮蔽函数
float GeometrySchlickGGX(float NdotV, float roughness) {
    float r = roughness + 1.0;
    float k = (r * r) / 8.0;
    
    float nom = NdotV;
    float denom = NdotV * (1.0 - k) + k;
    
    return nom / denom;
}

float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);
    
    return ggx1 * ggx2;
}

// 菲涅尔方程
vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}

void main() {
    // 材质属性
    vec3 albedo = texture2D(uAlbedoMap, vTexCoord).rgb;
    float metallic = texture2D(uMetallicRoughnessMap, vTexCoord).b;
    float roughness = texture2D(uMetallicRoughnessMap, vTexCoord).g;
    vec3 emissive = texture2D(uEmissiveMap, vTexCoord).rgb;
    
    // 法线贴图
    vec3 tangentNormal = texture2D(uNormalMap, vTexCoord).xyz * 2.0 - 1.0;
    // 需要 TBN 矩阵变换...
    vec3 N = normalize(vNormal);
    
    vec3 V = normalize(uCameraPosition - vWorldPosition);
    
    // 基础反射率
    vec3 F0 = vec3(0.04);
    F0 = mix(F0, albedo, metallic);
    
    // 反射率方程
    vec3 Lo = vec3(0.0);
    for (int i = 0; i < 4; i++) {
        vec3 L = normalize(uLightPositions[i] - vWorldPosition);
        vec3 H = normalize(V + L);
        
        float distance = length(uLightPositions[i] - vWorldPosition);
        float attenuation = 1.0 / (distance * distance);
        vec3 radiance = uLightColors[i] * attenuation;
        
        float NDF = DistributionGGX(N, H, roughness);
        float G = GeometrySmith(N, V, L, roughness);
        vec3 F = fresnelSchlick(max(dot(H, V), 0.0), F0);
        
        vec3 numerator = NDF * G * F;
        float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0) + 0.0001;
        vec3 specular = numerator / denominator;
        
        vec3 kS = F;
        vec3 kD = vec3(1.0) - kS;
        kD *= 1.0 - metallic;
        
        float NdotL = max(dot(N, L), 0.0);
        Lo += (kD * albedo / PI + specular) * radiance * NdotL;
    }
    
    // 环境光
    vec3 ambient = vec3(0.03) * albedo;
    
    vec3 color = ambient + Lo + emissive;
    
    // HDR 色调映射
    color = color / (color + vec3(1.0));
    
    // Gamma 校正
    color = pow(color, vec3(1.0 / 2.2));
    
    gl_FragColor = vec4(color, 1.0);
}
```

---

## 四、特效着色器

### 4.1 模糊效果
```glsl
// 高斯模糊
precision mediump float;

varying vec2 vTexCoord;
uniform sampler2D uTexture;
uniform vec2 uResolution;
uniform vec2 uDirection;

void main() {
    vec2 texelSize = 1.0 / uResolution;
    vec3 result = vec3(0.0);
    
    // 9-tap 高斯核
    float weights[5];
    weights[0] = 0.227027;
    weights[1] = 0.1945946;
    weights[2] = 0.1216216;
    weights[3] = 0.054054;
    weights[4] = 0.016216;
    
    result += texture2D(uTexture, vTexCoord).rgb * weights[0];
    
    for (int i = 1; i < 5; i++) {
        vec2 offset = uDirection * texelSize * float(i);
        result += texture2D(uTexture, vTexCoord + offset).rgb * weights[i];
        result += texture2D(uTexture, vTexCoord - offset).rgb * weights[i];
    }
    
    gl_FragColor = vec4(result, 1.0);
}
```

### 4.2 泛光效果
```glsl
// 泛光 (Bloom)
precision mediump float;

varying vec2 vTexCoord;
uniform sampler2D uScene;
uniform sampler2D uBloomBlur;
uniform float uBloomStrength;

void main() {
    vec3 scene = texture2D(uScene, vTexCoord).rgb;
    vec3 bloom = texture2D(uBloomBlur, vTexCoord).rgb;
    
    // 加法混合
    vec3 result = scene + bloom * uBloomStrength;
    
    // HDR 色调映射
    result = result / (result + vec3(1.0));
    
    // Gamma 校正
    result = pow(result, vec3(1.0 / 2.2));
    
    gl_FragColor = vec4(result, 1.0);
}
```

### 4.3 扭曲效果
```glsl
// 扭曲
precision mediump float;

varying vec2 vTexCoord;
uniform sampler2D uTexture;
uniform float uTime;
uniform vec2 uCenter;
uniform float uRadius;
uniform float uStrength;

void main() {
    vec2 uv = vTexCoord;
    vec2 center = uCenter;
    
    // 计算到中心的距离
    float dist = distance(uv, center);
    
    if (dist < uRadius) {
        // 扭曲强度
        float strength = uStrength * (1.0 - dist / uRadius);
        
        // 扭曲方向
        vec2 direction = normalize(uv - center);
        
        // 应用扭曲
        uv += direction * strength * sin(uTime * 5.0 + dist * 10.0);
    }
    
    vec4 color = texture2D(uTexture, uv);
    gl_FragColor = color;
}
```

### 4.4 噪声效果
```glsl
// Perlin 噪声
precision mediump float;

varying vec2 vTexCoord;
uniform float uTime;
uniform float uScale;

// 随机函数
float random(vec2 st) {
    return fract(sin(dot(st.xy, vec2(12.9898, 78.233))) * 43758.5453123);
}

// 噪声函数
float noise(vec2 st) {
    vec2 i = floor(st);
    vec2 f = fract(st);
    
    // 四个角的值
    float a = random(i);
    float b = random(i + vec2(1.0, 0.0));
    float c = random(i + vec2(0.0, 1.0));
    float d = random(i + vec2(1.0, 1.0));
    
    // 平滑插值
    vec2 u = f * f * (3.0 - 2.0 * f);
    
    // 插值
    return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}

// 分形噪声
float fbm(vec2 st) {
    float value = 0.0;
    float amplitude = 0.5;
    
    for (int i = 0; i < 6; i++) {
        value += amplitude * noise(st);
        st *= 2.0;
        amplitude *= 0.5;
    }
    
    return value;
}

void main() {
    vec2 st = vTexCoord * uScale;
    
    float n = fbm(st + uTime * 0.1);
    
    vec3 color = vec3(n);
    gl_FragColor = vec4(color, 1.0);
}
```

---

## 五、后处理效果

### 5.1 色调映射
```glsl
// Reinhard 色调映射
vec3 reinhardToneMapping(vec3 color) {
    return color / (color + vec3(1.0));
}

// ACES 色调映射
vec3 acesToneMapping(vec3 color) {
    float a = 2.51;
    float b = 0.03;
    float c = 2.43;
    float d = 0.59;
    float e = 0.14;
    return clamp((color * (a * color + b)) / (color * (c * color + d) + e), 0.0, 1.0);
}

// Uncharted 2 色调映射
vec3 uncharted2ToneMapping(vec3 color) {
    float A = 0.15;
    float B = 0.50;
    float C = 0.10;
    float D = 0.20;
    float E = 0.02;
    float F = 0.30;
    
    vec3 x = color * 2.0;
    return ((x * (A * x + C * B) + D * E) / (x * (A * x + B) + D * F)) - E / F;
}
```

### 5.2 色彩校正
```glsl
// 亮度/对比度/饱和度
vec3 adjustBrightness(vec3 color, float brightness) {
    return color + brightness;
}

vec3 adjustContrast(vec3 color, float contrast) {
    return (color - 0.5) * contrast + 0.5;
}

vec3 adjustSaturation(vec3 color, float saturation) {
    float luminance = dot(color, vec3(0.2126, 0.7152, 0.0722));
    return mix(vec3(luminance), color, saturation);
}

// 色调分离
vec3 posterize(vec3 color, float levels) {
    return floor(color * levels) / levels;
}

// 色彩映射
vec3 colorGrade(vec3 color, sampler2D lut) {
    // 3D LUT 查找
    float scale = 63.0 / 64.0;
    float offset = 1.0 / 128.0;
    
    vec3 uvw = color * scale + offset;
    
    float slice = uvw.z * 63.0;
    float sliceFloor = floor(slice);
    float sliceFract = slice - sliceFloor;
    
    vec2 uv1 = vec2(
        (sliceFloor + uvw.x) / 64.0,
        uvw.y
    );
    vec2 uv2 = vec2(
        (sliceFloor + 1.0 + uvw.x) / 64.0,
        uvw.y
    );
    
    vec3 color1 = texture2D(lut, uv1).rgb;
    vec3 color2 = texture2D(lut, uv2).rgb;
    
    return mix(color1, color2, sliceFract);
}
```

---

## 六、学习资源

| 资源 | 说明 |
|------|------|
| The Book of Shaders | 着色器入门 |
| Shadertoy | 在线着色器实验 |
| WebGL Fundamentals | WebGL 基础教程 |
| Learn OpenGL | OpenGL 教程 |

---

*本资料由 AI 整理，建议结合实际项目练习*
