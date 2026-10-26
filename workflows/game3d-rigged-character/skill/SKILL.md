---
name: wf-game3d-rigged-character
description: 用「3D 角色绑骨与动作管线」工作流完成从标准 T-Pose 角色立绘设定、3D PBR 模型生成（asset.model3d）、人形自动绑骨蒙皮（model.rigSkin）、原地游戏动作库重定向（model.retarget）、四边面低模重拓扑（model.retopology）到脚底原点对齐 FBX/GLB 导出的完整 3D 游戏工业流程。
workflow: game3d-rigged-character
workflow-version: 1.0.0
argument-hint: "[3D角色人设/体型服饰/材质风格（写实PBR/二次元卡通渲染/美式风格化LowPoly）/所需动作列表]"
when-to-use: 用户需要制作可导入 Unity / Unreal / Godot 的 3D 游戏角色、自动骨骼蒙皮（Rigging）、绑定跑跳战斗 3D 动画、模型减面重拓扑或导出 FBX/GLB 时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_tile_pbr_process, game_bundle_export
---

# 3D 角色绑骨与动作管线 · 工业管线 SOP（game3d-rigged-character v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，7 节点 / 5 连线）

```
角色标准 T-Pose 正交参考图 (asset.image: tposeRef, 1:1 原生透明背景 + 双臂平举关节无粘连)
  └─out→in-image──→ 生成 3D 角色网格 (asset.model3d: charModel, 输出 PBR GLB 模型)
                       ├─out→in-model──→ 自动骨骼蒙皮 (model.rigSkin: rigSkin, humanoid 人形骨骼权重)
                       │                    └─out→in-model──→ 3D 动作库绑定 (model.retarget: retargetAnim, In-Place 原地动作烘焙)
                       │                                         └─out→in-model──→ 引擎就绪 FBX/GLB 导出 (model.convert: fbxConvert, 脚底原点归中)
                       └─out→in-model──→ 游戏引擎低模重拓扑 (model.retopology: lowPolyModel, 四边面减面 + 贴图烘焙)
```

---

## 二、3D 游戏角色工业级三大黄金铁律

1. **铁律 1 · 严格的 T-Pose / A-Pose 关节分离约束（决定自动绑骨成败）**：
   - `tposeRef` 必须生成正面正交全身像，提示词必须明确：**`双臂向两侧水平展开（T-Pose）或呈 45° 张开（A-Pose），双腿自然分开站立，严禁斗篷/长发把手臂与躯干连成一片，原生透明背景`**；
   - 只有四肢与躯干在 2D 参考图中清晰分离，`charModel` 生成的 3D 网格才不会发生腋下或大腿粘连，`rigSkin` 自动蒙皮才能获得干净的关节旋转！

2. **铁律 2 · 原地动作烘焙（In-Place Animation）与脚底枢轴归中（Bottom-Center Pivot）**：
   - 游戏引擎由角色控制器（CharacterController / CharacterBody3D）驱动位移，因此 `retargetAnim`（`model.retarget`）必须设 `retargetAnimateInPlace: true`（原地动作，防止 Root Motion 乱飞）；
   - `fbxConvert`（`model.convert`）必须开启 `convertPivotToCenterBottom: true`，将模型原点校准到两脚底中心，导入 Unity / Godot / Unreal 时直接贴地站在地面 `(0,0,0)` 上。

3. **铁律 3 · Blender MCP 深度精修协同（可选增强）**：
   - 若本机已连接 Blender 工具面（`/mcp/blender`），可在 `charModel` 或 `rigSkin` 导出后调用 Blender 工具对模型执行自定义材质球调整、碰撞体（Collider）生成或法线光滑组修正。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "game3d-rigged-character", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户的 3D 角色设定，调用 `graph_edit` 更新 `tposeRef` 与 `charModel` 的提示词，以及 `retargetAnim` 的目标动作配置。
3. 调用 `task_run({ assetId })` 依次或整图运行生成 T-Pose 图、3D 网格、自动绑骨、动作绑定、低模重拓扑与 FBX 转换。

## 四、端口与参数速查

详见 `references/ports.md`。
