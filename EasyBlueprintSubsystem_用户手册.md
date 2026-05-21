# Easy Blueprint Subsystem 用户手册

---

## 启用插件

**编辑 → 插件** → 搜索 **Easy Blueprint Subsystem** → 勾选启用 → 按提示重启编辑器。

---

## 插件做什么

用**蓝图**写「子系统」逻辑：全局服务、跟关卡走的服务、只在编辑器里跑的工具等。写好子系统蓝图并**编译保存**后，插件会自动创建实例；其它蓝图里用 **Get** 节点（和引擎自带的 Get Subsystem 同一类菜单）拿到你的子系统再调用上面的函数、变量。

你需要接触的就三块：

1. **内容浏览器右键** — 创建子系统蓝图（四种父类选一种）  
2. **子系统蓝图里** — 写 **On Initialize** / **On Deinitialize**（以及可选 **Tick**、编辑器事件）  
3. **Actor / UI 等蓝图里** — **Get &lt;你的子系统名&gt;** 使用  

---

## 四种子系统类型怎么选

| 右键菜单项 | 用在哪里 | 常见用途 |
|------------|----------|----------|
| **Easy Blueprint Engine Subsystem** | 整款游戏 / 整个编辑器进程里只有一份 | 全局配置、跨关卡都用的管理器 |
| **Easy Blueprint Game Instance Subsystem** | 每次新开一局游戏（含 PIE）各一份 | 存档、玩家会话、跟「这一局」绑定的数据 |
| **Easy Blueprint World Subsystem** | 当前这一个关卡世界（含 PIE 里的那一局）各一份 | 本关规则、本关流程、只在这张图里生效的逻辑 |
| **Easy Blueprint Editor Subsystem** | 只在编辑器里，**不会**打进发布包 | 导入资源后处理、监听保存、编辑器小工具 |

同一类型可以建**多个**蓝图（例如两个 Engine 子系统：`BP_Audio`、`BP_Config`），互不顶替，各管各的。

**Editor** 类型仅编辑器有效；**Engine / Game Instance / World** 会随游戏一起打包。

---

## 创建子系统蓝图

### 操作步骤

1. 在**内容浏览器**目标文件夹空白处 **右键**。  
2. 展开 **Easy Blueprint Subsystem**。  
3. 点击上表四项之一（父类已选好，**不会**再让你挑父类）。  
4. 输入名称（建议带前缀，如 `BP_Engine_XXX`、`BP_GI_Save`、`BP_World_Level`、`BP_Editor_Import`）。  
5. 双击打开蓝图。

也可用 **右键 → Blueprint Class**，在父类列表里搜索 **Easy Blueprint**，手动选四种父类之一，效果相同。

### 打开蓝图后要做的事

1. 在 **Event Graph** 里实现：  
   - **On Initialize** — 子系统「启动」时执行（建缓存、读表、注册你要用的委托等）。  
   - **On Deinitialize** — 子系统「关闭」时执行（解绑委托、清空变量，避免残留引用）。  
2. 需要每帧逻辑时：打开 **类默认值（Class Defaults）** → **Tick** → 勾选 **Can Ever Tick**，再在 Event Graph 里写 **Tick** 事件（Editor 类型没有 Tick，见下文）。  
3. 工具栏 **Compile** → **Save**。

未 **Compile** 的子系统，别的蓝图里往往还搜不到 **Get** 节点。

### 什么时候会执行 On Initialize

| 类型 | 大致时机 |
|------|----------|
| **Engine** | 编辑器启动后、或进入 PIE / 运行游戏后，一般会较早创建 |
| **Editor** | 编辑器启动后（仅编辑器内） |
| **Game Instance** | **开始 PIE 或运行游戏之后**（编辑器里只改资源、不播放时，通常还没有实例） |
| **World** | **进入关卡并开始 PIE / 运行游戏之后**（同上） |

因此：测 Game Instance / World 时，请先 **播放（PIE）** 或 **运行游戏**，再看日志或断点。

### 删除、改名、大改蓝图

- **删除**子系统蓝图资产：与其它蓝图一样在内容浏览器删除即可。  
- **改名、改逻辑**：改完后务必再 **Compile** + **Save**；若 Get 节点或行为不对，可关掉相关蓝图再打开，或重启编辑器。  

---

## 在其它蓝图中使用子系统

### 添加 Get 节点

1. 打开要用子系统的蓝图（Actor、Level Blueprint、Widget 等）的 **Event Graph**。  
2. 在空白处 **右键**。  
3. 按子系统类型在对应菜单里找（名称是你的**子系统蓝图类名**）：  

| 子系统类型 | 右键菜单路径（与引擎一致） |
|------------|---------------------------|
| Engine | **Engine Subsystems** → **Get BP_XXX**（XXX 为你的资产名） |
| Game Instance | **GameInstance Subsystems** → **Get BP_XXX** |
| World | **World Subsystems** → **Get BP_XXX** |
| Editor | **Editor Subsystems** → **Get BP_XXX** |

也可在右键搜索框直接输入 `Get BP_你的资产名`。

4. 从 Get 节点的返回值拖线，调用该子系统蓝图里自己建的 **函数**，或读 **变量**。

### Get 节点什么时候会出现

必须满足：

1. 子系统蓝图已 **Compile** 且 **Save**；  
2. 当前正在编辑的蓝图类型允许放这类节点（普通 Event Graph 均可）。  

若仍没有：保存子系统 → 关闭并重新打开当前蓝图 → 仍无则 **重启编辑器**。

### 使用上注意

- 在 **BeginPlay**、按钮事件等**需要的时候**再 Get 即可，不必在关卡里放一个「全局引用」变量长期挂着（关卡 Actor 销毁后，子系统可能还在，引用关系容易乱）。  
- 子系统已关闭（例如结束 PIE、换关卡）之后，不要再使用上一次 Get 到的引用。

---

## On Initialize、On Deinitialize、Tick

### On Initialize / On Deinitialize

在**子系统蓝图自己的** Event Graph 里，和自定义事件一样使用：

| 事件 | 含义 |
|------|------|
| **On Initialize** | 插件刚创建好这个子系统实例时调用一次 |
| **On Deinitialize** | 实例即将被销毁前调用一次 |

**On Deinitialize** 里应做与 **On Initialize** 相反的清理解绑；不要在这里再启动新逻辑。

### Tick（仅 Engine / Game Instance / World）

1. 打开子系统蓝图 → **类默认值**。  
2. **Tick** 分类：  
   - **Can Ever Tick**：勾选后才会跑 **Tick** 事件（默认不勾）。  
   - **Tick Interval**：`0` 表示每帧；填 `0.5` 表示大约每 0.5 秒一次。  
   - **Tick Even When Paused**：游戏暂停时是否仍 Tick。  
3. 在 Event Graph 添加 **Tick** 事件写逻辑。  

**Editor** 子系统没有 Tick；编辑器里要「监听变化」请用下一节的 **Editor Events**。

Tick 一般在 **PIE / 运行游戏** 时才有意义；只开编辑器不播放时，Game Instance / World 通常还不会 Tick。

---

## 编辑器子系统与事件

仅 **Easy Blueprint Editor Subsystem** 有本节内容；用于资源导入、保存地图等**编辑器内**自动化。

### 创建

与其它类型相同：**内容浏览器右键 → Easy Blueprint Subsystem → Easy Blueprint Editor Subsystem**。

### 绑定事件（两种方式，任选）

**方式 A — 类默认值（适合长期监听）**

1. 打开 Editor 子系统蓝图。  
2. 点 **Class Defaults**。  
3. 找到 **Editor Events** 分类。  
4. 在某个事件（如 **On Asset Added**）右侧点 **+**，选择 **Add …** 或绑定到自定义事件。  
5. 在绑定到的事件里写逻辑（例如打印路径、刷新列表）。  

**方式 B — On Initialize 里绑定**

1. 在 Event Graph 的 **On Initialize** 里，从 **On Asset Added** 等委托拖出 **Assign**。  
2. 接到你的自定义事件。  
3. 在 **On Deinitialize** 里对同一委托 **Clear** 或解绑，避免编辑器关闭后仍被回调。  

### 事件说明

| 事件 | 什么时候会触发 | 参数（蓝图里能看到） |
|------|----------------|----------------------|
| **On Asset Added** | 新建、导入、保存后登记到内容浏览器的资源 | Asset、Asset Path、Asset Class |
| **On Asset Removed** | 从内容浏览器删除资源 | 同上 |
| **On Asset Updated** | 重命名、移动、覆盖保存等（触发较勤，可做节流） | 同上 |
| **On Asset Editor Opened** | 双击打开某资源的编辑器窗口 | Asset |
| **On Asset Editor Closed** | 关闭该资源的编辑器窗口 | Asset |
| **On Asset Post Import** | 导入或重新导入完成 | Factory、Imported Object |
| **On World Saved** | 保存关卡 / 地图完成 | World |

**Asset** 有时为空，可同时看 **Asset Path**（字符串路径）判断是否是你关心的资源。

### 在别的蓝图里 Get Editor 子系统

在**仅在编辑器运行**的蓝图（如 Editor Utility 相关）Event Graph 中：**右键 → Editor Subsystems → Get &lt;你的 Editor 子系统名&gt;**。

---

## 常见问题

**Q：搜不到 Get 节点？**  
A：先对子系统蓝图 **Compile + Save**；重开正在编辑的蓝图；仍没有就重启编辑器。

**Q：Game Instance / World 的 On Initialize 没执行？**  
A：先 **PIE** 或 **运行游戏**；只编辑资源、不播放时通常还没有实例。

**Q：能有多个 Engine 子系统吗？**  
A：可以，每个子系统蓝图类各自一个实例。

**Q：打包后的游戏里能用 Editor 子系统吗？**  
A：不能，Editor 类型只在编辑器存在。

**Q：子系统蓝图里能写普通函数、变量吗？**  
A：可以，和普通蓝图一样；其它蓝图通过 Get 节点拿到实例后调用即可。
