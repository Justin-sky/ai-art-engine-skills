# 节点拓扑与端口速查（game3d-rigged-character v1.0）

## 默认种子节点清单（共 7 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `tposeRef` | `asset.image` | 角色标准 T-Pose 正交参考图（`1:1`, `generateTransparentBg: true`） |
| `charModel` | `asset.model3d` | 生成 3D 角色网格（GLB） |
| `rigSkin` | `model.rigSkin` | 自动骨骼蒙皮（`generateRigType: "humanoid"`） |
| `retargetAnim` | `model.retarget` | 3D 动作库绑定（`retargetAnimateInPlace: true`） |
| `lowPolyModel` | `model.retopology` | 游戏引擎低模重拓扑（`retopologyQuad: true, retopologyBake: true`） |
| `fbxConvert` | `model.convert` | 引擎就绪 FBX/GLB 标准化导出（`convertFormat: "FBX", convertPivotToCenterBottom: true`） |
| `note` | `note.text` | 3D 游戏角色全流程管线说明 |

## 默认连线关系（共 5 条连线）

```
tposeRef:out → charModel:in-image
charModel:out → rigSkin:in-model
rigSkin:out → retargetAnim:in-model
retargetAnim:out → fbxConvert:in-model
charModel:out → lowPolyModel:in-model
```
