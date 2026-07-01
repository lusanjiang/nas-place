# 粒子 3D 模型查看器 · Demo

Web 端 Three.js 实现的粒子 3D 模型加载器,包含:

- ✨ **粒子凝聚入场动画** — 从爆炸弥散状态 3 秒回流凝聚成 3D 模型(GSAP + 自定义 vertex shader + 柏林噪声)
- 🖱 **鼠标流体扰动** — 屏幕上滑动时,GPU 实时推开附近粒子(屏幕空间径向位移 + 波纹衰减)
- 📦 **PLY 点云加载器** — 支持本地 .ply 文件拖拽 / 选择

## 打开 Demo

直接在浏览器打开 [index.html](https://lusanjiang.github.io/nas-place/particle-demo/index.html)。

默认加载狮子头模型(简化版 20 万点 ≈ 2.9MB)。
点击界面"⬇ 加载完整版"按钮可切换至 127 万点完整模型(19MB)。

## 鼠标操作

| 操作 | 效果 |
|---|---|
| 左键拖动 | 旋转视角 |
| 滚轮 | 缩放 |
| 鼠标滑动 | 推开附近粒子(流体效果) |
| 右键 / Shift+左键 | 平移 |

## 文件清单

| 文件 | 大小 | 说明 |
|---|---|---|
| `index.html` | 19 KB | 主程序 |
| `lion_simple.ply` | 2.9 MB | 简化版狮子头(20 万点,默认加载) |
| `lion.ply` | 19 MB | 完整版狮子头(127 万点,可选加载) |

## 技术栈

- **Three.js 0.160**(WebGL + 自定义 GLSL Shader)
- **GSAP 3.12**(凝聚动画缓动)
- **PLYLoader**(二进制点云加载)

## 三个核心实现要点

### 1. 凝聚入场动画

```glsl
// vertex shader
vec3 scatteredPos = position + aScatter * uScatterStrength * aAmplitude;
float p = smoothstep(0.0, 1.0, uProgress);
vec3 modelPos = mix(scatteredPos, position, p);
```

每个粒子预生成一个随机散布方向(`aScatter`)和一个振幅(`aAmplitude`),用 GSAP 在 3 秒内把 `uProgress` 从 0 过渡到 1,实现"爆炸 → 凝聚"。

### 2. 鼠标流体扰动(屏幕空间)

```glsl
vec2 toMouse = ndc - uMouseNDC;
float d = length(toMouse);
float falloff = smoothstep(0.55, 0.0, d);
float wave = sin(d * 22.0 - uTime * 5.5) * 0.5 + 0.5;
float push = falloff * (0.55 + 0.45 * wave) * uMouseStrength;
clipPos.xy += pushDir * push * 0.45 * depthFactor * clipPos.w;
```

把粒子投影到 NDC,与鼠标距离做径向位移,叠加时间相关的正弦波纹。

### 3. PLY 加载

```js
const loader = new PLYLoader();
loader.load('./lion_simple.ply', (geometry) => {
  const points = new THREE.Points(geometry, material);
  scene.add(points);
});
```
