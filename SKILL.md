---
name: mmd-npr-render
description: "MMD PMX 三渲二渲染 / Blender NPR：Blender 4.2 + EEVEE Next 赛璐璐卡通渲染全流程（mmd_tools 导入、卡通硬边材质、几何节点描边、线宽跳变/贴图丢失/肤色发白等坑）。"
version: 0.1.0
author: SilverlingFox
license: MIT
tags: [mmd, pmx, blender, npr, toon-shading, cel-shading, eevee, outline, mmd_tools]
---

# MMD 三渲二出图（Blender 4.2 + EEVEE Next）

> ⚠️ **数值口径**：下文所有具体数值（描边宽度 0.0010、暗部色 `(0.30,0.25,0.33)`、脸部乘色等）都是在**某个全身 + 脸部的动漫风 MMD 模型**上实测得到的**起点值**，不是万能常数。换模型务必按「完成判据」逐项校准，尤其是描边宽度和脸部肤色。

## 何时用

- 用户给 MMD `.pmx` 模型，要求卡通渲染 / 三渲二 / 赛璐璐质感 / 描边
- 要把现成的 MMD 渲染预设（`.blend`）套到另一个模型上
- 症状触发：PMX 导入后贴图全丢、描边粗细改不动、勾线粗细跳变、某部位不该有描边、脸/肤色发白

## 环境准备

- Blender 4.2.x 便携版（下载走清华镜像 + aria2c 多线程；阿里云镜像慢）。工作入口：`<blender_dir>\blender.exe`
- `mmd_tools` 模块名是 **`bl_ext.blender_org.mmd_tools`**（4.2 扩展仓库，**不是** `user_default`）。启用：
  `addon_utils.enable('bl_ext.blender_org.mmd_tools', default_set=True)`
- 安装扩展：`blender.exe --command extension install-file -r user --enable <zip>`
- 命令行导入前先 `$env:PYTHONUTF8="1"`；脚本一律写成 `.py` 用 `-b --python <脚本>` 跑
- 启动时 mmd_tools 会尝试 pip 降级 numpy 报 `WinError 5`，属既有噪声；**Blender 退出码 1 不代表任务失败**，看日志里的完成标记

## 步骤

1. **导入前先把整个 PMX 包复制到英文路径**（中文目录会让贴图全部加载失败：“不能成为 Image 的相对路径”）。导入：

```python
bpy.ops.mmd_tools.import_model(filepath=pmx, scale=0.08,
    types={'MESH','ARMATURE','MORPHS'}, clean_model=True, rename_bones=True)
```
   完成判据：`bpy.data.images` 中 `size>0` 的贴图数 = 材质实际引用数（缺失 0）。

2. **建卡通材质**（逐材质重写）：`Image Texture`(原漫反射) → `Diffuse BSDF` → `Shader to RGB` → `ColorRamp`(亮/暗 2 段硬边) → `Mix(Multiply)` 乘回原色 → `Emission`（保留 Alpha 透明混合）。
   - 起点值：暗部色 `(0.30,0.25,0.33)`（冷调）、饱和度 1.2、亮部提亮遮罩 0.45~0.95（提升 ≤40%），暗部完全不动；**换模型后先只调这两段硬边的位置**
   完成判据：渲染图有硬边明暗分界，暗部不灰。

3. **建描边**（Geometry Nodes 反向壳）：`Merge by Distance`（阈值 0.0005）→ `Set Position` 沿 Normal 外扩（宽度起点值 **0.0010**）→ `Flip Faces` → `Set Material`(纯黑 + `use_backface_culling`) → `Join` 回本体。
   - **改宽度必须改修改器上的覆盖值 / Socket 覆盖，改节点组接口默认值无效**
   - 某材质不描边：用 `Material Selection` 遮罩把外扩系数设 0；薄片部位（如嘴）直接 `Delete Geometry(FACE)` 移出壳体更干净
   - 经验：`口/舌/齒` 常与其它材质 0 顶点共享，可安全排除；排除眼周材质时保留 `眉睫`（睫毛/眉定义线）
   完成判据：轮廓线像素均值接近目标宽度，且 cv < 0.16（沿轮廓采样）。

4. **修“勾线粗细跳变 / 断续”**：本体加 `Weighted Normal` 修改器；描边前先 `Merge by Distance` 焊接；渲染采样 ≥128。
   完成判据：放大图里发丝接缝处不再有细线 / 断点。

5. **脸部专用处理**：脸部材质平涂优先（暗部 ≤7%，不用冷紫阴影）；`白目/目/目光` 完全平涂并微提亮；肤色发白时给脸部乘 `(1.0,0.93,0.86)` 并把脸部 Emission 降到 1.65（仍白再上 `(1.0,0.90,0.80)`）。
   完成判据：脸部无脏阴影，眼睛红瞳/高光锐利，肤色不发白。

6. **渲染设置**：EEVEE Next；色彩管理 Filmic + Very High Contrast；世界强度 0.04~0.06；Sun（angle 1° 硬阴影，energy 3.8）+ Area 补光；Fog Glow（threshold 1.0、mix ≈7%）；采样 64→128→192 按分辨率选。

7. **出图与验收**：先用 1440² 迭代构图与参数，定稿后再按目标分辨率出图（只改分辨率/采样，不动相机/材质）；4K 竖幅用 2160×3240，更高用 3840×5760。
   完成判据：平均亮度达标、黑裙像素仍 <40/255（没变灰）、白层干净、出图文件大小正常。

## 排版与灯光参考

- 全身：正前略低视角（下俯 ~5°），50mm，剪影占框高 ≈89%，脚下留 5% 地面
- 脸部特写：头部占画面 ≥70%，正前略高视角

## 验证

- 每轮渲染后自己读图（view_image）与参考立绘对比，只列差异项再改
- **机器锁屏时窗口截图会空白**（PrintWindow 也一样），以**渲染图**验证，不要用截图
- 参考图可从官方 wiki 页取原图直链（带 Referer + UA 下载 `/images/` 非 `/thumb/`）

## 常见坑速查

| 症状 | 根因 | 修法 |
| --- | --- | --- |
| 贴图全丢 | 中文目录路径 | 复制到英文路径重新导入 |
| 描边宽度改不动 | 改了节点组接口默认 | 改修改器覆盖值 |
| 线宽跳变 / 断续 | UV 接缝拆点、法线不统一 | Merge 焊接 + Weighted Normal + 高采样 |
| 嘴 / 眼有多余黑线 | 描边套到不该描的部位 | Material Selection 遮罩 / Delete Geometry(FACE) |
| 腿像“白袜” | 模型无肤色贴图，肌材质引用衣料图的近白区 | 乘暖肤色或重绘图集该区域 |
| 预设打不开 | 依赖 Goo Engine 自定义节点 | 用普通版预设，边缘光用 Fresnel / LayerWeight 近似 |

## 版权与使用注意

- 本技能**不包含**任何 MMD 模型、贴图或预设；示例模型版权归原作者。
- MMD 模型普遍为“允许改造、禁二次配布 / 禁商用 / 禁 18 禁”，所有处理在副本上进行，原始包只读。
- 请在模型作者授权范围内使用本流程产出的图像。
