# Unity / 游戏客户端开发面试知识点总结（2026版）

> 本文件是《Unity游戏客户端面试知识库_2026_含回答.md》的“知识点总结版”。
>
> 作用：先建立完整知识图谱，再进入后面的“高频题标准回答库”。
>
> 复习原则：不要孤立背术语，要按照“概念 → 原理 → 为什么 → 边界 → 项目应用 → 追问链”理解。

---

# 0. 总体复习框架

游戏客户端面试通常同时考察四种能力：

1. **语言与计算机基础**：C#、C++、数据结构、操作系统、网络。
2. **Unity 引擎能力**：生命周期、资源、UI、动画、物理、序列化、协程等。
3. **工程与性能能力**：Profiler、内存、CPU/GPU、资源生命周期、Bug 排查。
4. **项目设计能力**：架构、模块职责、通信方式、可扩展性、技术选型和实际问题解决。

优先级建议：

- **P0**：必须能完整口述，并承受 2～3 层追问。
- **P1**：高频进阶，至少理解原理和适用场景。
- **P2**：大厂、专项岗位或加分题，根据目标公司深入。

建议顺序：

```text
项目深挖
  ↓
C# / C++ / 数据结构
  ↓
Unity 核心
  ↓
UGUI / 性能
  ↓
网络
  ↓
图形学
  ↓
OS / Lua / 热更新
  ↓
算法
  ↓
HR / 游戏理解 / AI 工程能力
```

---

# 第一部分：项目深挖【P0】

## 1. 项目面试的核心逻辑

项目不是“介绍做过什么”，而是证明你是否真正理解自己做过的系统。面试官通常围绕以下几个维度追问：

- 项目目标是什么。
- 你本人负责什么。
- 整体架构如何组织。
- 数据怎么流动。
- 模块之间如何通信。
- 最难的问题是什么。
- 为什么选择当前方案。
- 当前方案有什么缺点。
- 性能瓶颈在哪里。
- 生命周期和异常情况如何处理。
- Bug 怎么定位。
- 数据规模扩大后哪里会先出问题。

项目题最好统一采用下面的表达结构：

```text
背景
→ 问题
→ 约束
→ 方案选择
→ 实现
→ 遇到的问题
→ 优化/验证
→ 当前方案缺点
→ 如果重做如何改进
```

## 2. 架构设计

### MVC / MVVM

核心目的都是把数据、业务和表现分离，降低 UI 与业务系统耦合。

- **Model**：业务数据和规则。
- **View**：负责显示和用户交互。
- **Controller / ViewModel**：负责把输入和数据变化转换成行为或显示状态。

需要关注：

- View 不应直接承担复杂业务逻辑。
- ViewModel/Controller 生命周期应与其服务的 UI 或业务模块匹配。
- 全量刷新简单但成本高；局部刷新复杂一些但更适合高频变化。
- 数据变化通常通过事件、绑定或命令通知 View。

### Observer / EventBus

事件用于解耦“谁产生消息”和“谁消费消息”。

优点：

- 降低直接引用。
- 便于多个模块订阅同一事件。

风险：

- 调用链不透明。
- 生命周期复杂，忘记解绑可能形成长期引用。
- 事件过度使用会让系统难追踪。

原则：

> 明确的一对一调用优先接口/直接引用；真正需要一对多广播或模块解耦时再使用事件。

对知识库一些问题的回答：

- 委托、事件、观察者模式区别？：
委托是语言级的“函数引用”；事件是在委托之上的访问约束；观察者模式是设计模式，用来描述一对多通知关系。

- EventBus 是否会导致调用链不可追踪？
会，而且这是 EventBus 最大的问题之一。
如何减少：
  - 限制 EventBus 使用范围。第一种：事件语义必须是“已经发生的事实”，比如 PlayerDiedEvent。第二种：核心业务流程不要依赖全局 EventBus，比如支付->扣金币->发道具->保存数据 这种关键链路显示调用。第三种：做调试工具。

**回答：** EventBus 的确可能导致调用链不可追踪，因为发布者只知道自己发了一个事件，并不知道具体哪些模块响应，依赖关系从显式调用变成了隐式运行时依赖。项目中我一般会限制 EventBus 只用于跨模块的状态通知或者“已经发生的事实”，而关键业务流程和明确的命令操作仍然使用接口调用。同时可以通过统一注册表、事件日志、订阅者追踪等方式提高可调试性。

- 如何解除监听、防止对象泄漏？
- 同步事件与异步事件如何处理？
- 为什么有些模块宁愿直接接口调用，也不用事件？

### FSM / HFSM / Behavior Tree

FSM 适合表达明确状态和转换：

```text
Idle → Move → Attack → Hit → Dead
```

状态数量增加后容易产生状态爆炸，因此可以使用 HFSM 将状态分层，例如：

```text
Grounded
├── Idle
├── Move
└── Attack

Airborne
├── Jump
└── Fall
```

行为树更适合 AI 决策：

- Selector：选择一个可执行分支。
- Sequence：按顺序执行，任意一步失败则失败。

FSM 更强调“当前处于什么状态”，BT 更强调“当前应该做什么”。

### Ability / GAS 思路

典型职责：

- **Ability**：行为和技能生命周期。
- **Gameplay Effect**：数值效果或状态效果。
- **Attribute**：属性及最终数值。
- **Tag**：状态和条件表达。
- **ASC 类容器**：统一管理 Ability、Effect、Tag、Attribute。

核心思想是把行为、数值、状态和表现解耦。

---

# 第二部分：C# / .NET【P0】

## 2.1 值类型与引用类型

值类型强调“值本身”，复制时通常复制数据；引用类型变量保存对象引用，复制变量通常复制引用。

不能简单说“值类型一定在栈，引用类型一定在堆”。存储位置取决于上下文：

- 局部值类型可能位于栈或寄存器。
- 值类型如果作为 class 字段，会内嵌在堆对象中。
- 装箱后的值类型会进入托管堆。

`struct` 适合：

- 数据较小。
- 具有值语义。
- 生命周期简单。
- 不希望存在独立对象身份。

Unity 的 `Vector3` 使用 struct，就是因为它本质上是一组数值数据。

## 2.2 装箱与拆箱

装箱：

```text
ValueType → object / interface
```

运行时通常需要在托管堆创建对象并复制值。

拆箱则需要验证对象真实类型，再取得内部值。

性能问题主要来自：

- 堆分配。
- 数据复制。
- 类型检查。
- 后续 GC 压力。

泛型容器比旧式 `object` 容器更高效的重要原因之一，就是减少装箱。

## 2.3 GC

GC 解决托管对象生命周期管理问题。

核心过程：

```text
GC Root
  ↓
可达性分析
  ↓
Mark
  ↓
Sweep
  ↓
必要时 Compact
```

分代：

- Gen0：新创建、短生命周期对象。
- Gen1：中间层。
- Gen2：长生命周期对象。

Unity 性能中重点关注 `GC.Alloc`。常见来源：

- 临时集合。
- 字符串拼接。
- LINQ。
- 闭包。
- 装箱。
- 高频 new。

对象池可以减少频繁创建/销毁和 GC，但不能无脑使用，因为池本身也会占内存并增加生命周期复杂度。

有 GC 仍然可能产生“逻辑泄漏”：对象一直被静态变量、事件、集合等引用，就不会被回收。

Unity 的 Texture、Mesh、AssetBundle 等还涉及 Native/GPU 资源，不能只依赖 C# GC。

## 2.4 string 与 StringBuilder

`string` 是不可变对象。修改字符串实际上往往意味着创建新字符串。

连续拼接：

```csharp
s = s + a + b + c;
```

可能产生多个临时对象。

大量或循环拼接时可以使用 `StringBuilder` 减少临时字符串。

`""` 和 `string.Empty` 语义基本等价，不应解释成一个每次分配、一个不分配。

## 2.5 Delegate / Event / Lambda / Closure

Delegate 是类型安全的函数引用，可以形成调用列表。

`event` 是对 delegate 的访问约束，外部通常只能订阅和取消订阅，不能随意覆盖或主动触发。

`Action` / `Func` 是常用泛型委托。

Lambda 如果捕获外部变量，编译器可能创建 Closure 对象，把捕获变量提升到堆上，因此可能产生额外分配。

Unity 中事件最重要的工程问题是生命周期：订阅者销毁后如果仍被长生命周期对象引用，可能无法回收。

## 2.6 泛型

泛型的主要价值：

- 类型安全。
- 代码复用。
- 减少 object 转换和装箱。

常见约束：

```csharp
where T : class
where T : struct
where T : new()
where T : IInterface
```

在 IL2CPP/AOT 环境下还需要了解泛型实例化和代码裁剪问题。

## 2.7 Reflection / Metadata / Assembly / DLL

这一组必须作为完整知识链理解：

```text
C# 源码
  ↓ 编译
Assembly / DLL
  ├── IL
  ├── Metadata
  └── Manifest
  ↓ Runtime
Type / MethodInfo / FieldInfo
  ↓
Reflection
```

### Metadata

元数据是“描述程序结构的数据”，包括：

- 类型。
- 字段。
- 方法。
- 属性。
- 继承。
- 接口。
- Attribute。
- 泛型信息。

### DLL / Assembly

C# DLL 不只是机器码文件，而是一个程序集，其中通常包含 IL、Metadata 和程序集描述信息。

### Reflection

反射就是运行时查询和操作这些类型元信息。

例如：

```csharp
Type type = typeof(Player);
MethodInfo method = type.GetMethod("Attack");
```

`Invoke` 比直接调用慢，原因包括：

- 运行时查找。
- 参数检查。
- 类型转换。
- 可能的装箱。

Unity Editor、Odin、DI、自动注册、Attribute 系统都会大量使用反射。

IL2CPP 下需要注意 stripping：如果代码只被反射引用，Linker 可能认为它没用，从而裁剪，需要 `[Preserve]` 或 `link.xml` 保留。

## 2.8 List 与 Dictionary

### List

本质是动态数组：

- 连续存储。
- `Count` 表示实际元素数。
- `Capacity` 表示当前内部数组容量。
- 扩容需要重新申请数组并复制。

优点：

- 遍历快。
- Cache Friendly。
- 内存结构简单。

缺点：

- 中间插入/删除需要移动元素。
- 按值查找通常 O(n)。

### Dictionary

核心结构可以理解为：

```text
HashCode
  ↓
Bucket
  ↓
Entry
  ↓
Equals
```

重点：

- 平均查询 O(1)。
- 哈希冲突不可避免，需要冲突处理。
- 扩容有额外成本。
- Key 的 hash/equality 相关状态加入后不应改变。

### 为什么小数据量可能 List 更好

复杂度不等于实际耗时。

小数据量时，List 的 O(n) 只需要比较很少几个元素，而 Dictionary 还存在：

- Hash 计算。
- Bucket 查找。
- Entry 访问。
- 更大的内存结构。

List 又具备连续内存优势，所以实际可能更快。

## 2.9 async / await / Task

`Task` 不等于线程。

`async/await` 本质上由编译器生成状态机。

当 `await` 的 Task 未完成时：

```text
保存状态
→ 注册 continuation
→ 当前方法返回
→ Task 完成
→ 恢复执行
```

因此 await 并不是“创建一条线程等在那里”。

`CancellationToken` 是协作式取消：任务必须主动观察 Token 才能停止。

如果底层 API 根本不支持取消，只能取消“等待结果”，不能真正终止底层操作。

Unity API 大多数要求在主线程调用，所以异步代码必须注意线程切换。

## 2.10 锁与线程安全

`lock` 基于 Monitor 提供互斥访问。

常见同步工具：

- Monitor / lock：互斥。
- Mutex：可跨进程，成本较高。
- Semaphore：限制同时进入数量。
- ReaderWriterLockSlim：读多写少。
- SpinLock：短时间忙等。
- Interlocked：原子操作。

死锁四条件：

1. 互斥。
2. 请求并保持。
3. 不可剥夺。
4. 循环等待。

---

# 第三部分：C++【P0-P1】

## 3.1 指针与引用

指针保存地址，可以为空、可以重新指向其他对象；引用通常表示某个已有对象的别名，初始化后不能重新绑定。

需要理解：

- nullptr。
- 野指针。
- 悬空指针。
- `void*`。
- const pointer / pointer to const。

## 3.2 多态与虚函数

运行时多态的核心条件：

- 存在继承关系。
- 基类函数是 virtual。
- 子类 override。
- 通过基类指针/引用调用。

底层常见实现：

```text
对象
 └── vptr
      ↓
    vtable
      ↓
  具体虚函数地址
```

基类作为多态基类时析构函数通常应该是 virtual，否则通过基类指针 delete 派生对象可能无法完整析构。

## 3.3 智能指针

- `unique_ptr`：独占所有权。
- `shared_ptr`：共享所有权，引用计数。
- `weak_ptr`：非拥有观察引用，解决 shared_ptr 循环引用。

RAII 核心思想：资源生命周期绑定对象生命周期。

## 3.4 move 与右值

`std::move` 本身不移动数据，只把表达式转换成右值类别，使移动构造/移动赋值有机会被调用。

移动的价值在于把资源所有权从一个对象转移给另一个对象，减少深拷贝。

`std::forward` 用于完美转发，保持模板参数原本的值类别。

## 3.5 内存对齐

CPU 往往更适合按自然对齐地址访问数据。

结构体大小可能因为 padding 大于字段大小简单相加结果。

字段排序会影响结构体大小和 Cache 利用率。

## 3.6 STL

重点理解底层结构和复杂度：

- vector：动态连续数组。
- list：双向链表。
- map：通常基于平衡树。
- unordered_map：哈希表。
- priority_queue：堆。

选择容器时不仅看 Big-O，还要考虑：

- Cache。
- 内存开销。
- 访问模式。
- 插入删除频率。

## 3.7 编译与链接

典型流程：

```text
源文件
→ 预处理
→ 编译
→ 汇编
→ 目标文件
→ 链接
→ 可执行文件 / 动态库
```

链接负责解析跨目标文件的符号引用。

---

# 第四部分：Unity 核心【P0】

## 4.1 MonoBehaviour 生命周期

核心顺序要理解而不是死背全部函数：

```text
Awake
→ OnEnable
→ Start
→ Update / FixedUpdate / LateUpdate
→ OnDisable
→ OnDestroy
```

- Awake：对象初始化。
- OnEnable：每次启用。
- Start：第一次真正进入运行时逻辑前。
- Update：帧更新。
- FixedUpdate：固定时间步，通常用于物理。
- LateUpdate：Update 后处理，例如摄像机跟随。

## 4.2 Coroutine

Unity Coroutine 不是线程。

它基于 `IEnumerator` 和 PlayerLoop，在每帧检查 yield 条件并继续推进状态机。

用途：

- 跨帧流程。
- 延时。
- 等待异步操作。
- 简化时序逻辑。

优势是表达清晰，不是天然比 Update 更快。

## 4.3 GameObject / Component / Transform

Unity 使用组合而不是深继承来组织对象：

```text
GameObject
  ├── Transform
  ├── Renderer
  ├── Collider
  └── 自定义 MonoBehaviour
```

Component 提供行为和数据，GameObject 是组件容器。

## 4.4 Unity 序列化

Unity 自己有一套序列化系统，并不等价于 .NET 默认序列化。

常见可序列化内容：

- public field。
- `[SerializeField] private field`。
- Unity 支持的类型。

不能简单认为所有 C# 类型、接口、多态对象都会自动支持。

## 4.5 Instantiate / Destroy / Object Pool

Instantiate/Destroy 可能带来：

- Native 对象创建销毁。
- Managed 分配。
- 生命周期函数。
- GC 压力。

对象池适合高频重复出现的对象，例如：

- 子弹。
- 飘字。
- 特效。
- UI Item。

池化重点不是只写 Get/Release，而是设计：

- Spawn。
- OnSpawn。
- Despawn。
- Reset。
- 最大容量。
- 资源释放。

## 4.6 Resources / AssetBundle / Addressables

### Resources

简单方便，但大型项目中容易导致资源组织、构建和内存管理粗放。

### AssetBundle

用于把资源打成可独立加载的包，需要自行处理：

- 依赖。
- 加载。
- 卸载。
- 版本。

### Addressables

是基于 AssetBundle 等底层能力构建的资源管理系统，提供：

- 地址化加载。
- 异步接口。
- 依赖管理。
- Catalog。
- Handle / 引用计数思想。

重点不是只会 `LoadAssetAsync`，而是理解 Handle 生命周期和 Release。

## 4.7 场景加载

重点：

- 同步 vs 异步。
- Additive 场景。
- 场景切换过程中的资源生命周期。
- 加载界面和进度。
- 常驻对象管理。

## 4.8 物理系统

需要掌握：

- Rigidbody。
- Collider。
- Trigger。
- Raycast。
- Collision Layer。
- FixedUpdate。

性能上应减少无意义碰撞检测，合理配置 Layer Collision Matrix。

## 4.9 动画

重点包括：

- Animator Controller。
- Layer。
- Blend Tree。
- Avatar Mask。
- Root Motion。
- Animation Event。
- 状态机与动画逻辑解耦。

Root Motion 的核心是由动画提供位移/旋转增量，再由角色运动系统决定如何应用。

## 4.10 四元数与欧拉角

欧拉角直观，但存在：

- 旋转顺序问题。
- Gimbal Lock。
- 插值不自然。

单位四元数是一种旋转数学表示，可以由 Axis-Angle 映射得到：

```text
q = (x, y, z, w)
```

它不是“四个角度”。

Unity 常用：

- `Quaternion.Euler`。
- `LookRotation`。
- `AngleAxis`。
- `Slerp`。

---

# 第五部分：UGUI / UI【P0】

## 5.1 Canvas

UGUI 最终需要把 Graphic 转成 Mesh 并组织 Batch，然后提交绘制。

Canvas 是 UGUI 的重要重建和排序边界。

## 5.2 动静分离

把高频变化 UI 与基本不变化 UI 分到不同 Canvas/Sub Canvas，核心目的是减少 Canvas Rebuild 范围。

例如：

```text
Static Canvas
├── 背景
├── 边框
└── 装饰

Dynamic Canvas
├── HP
├── Buff
└── Timer
```

但 Canvas 不能无脑拆，因为 Canvas 之间无法直接合批，可能增加 DrawCall。

因此动静分离本质上是：

> Canvas Rebuild 开销和 DrawCall 之间的权衡。

## 5.3 UGUI 合批

影响批次的常见因素：

- Canvas 边界。
- Material。
- Texture/Atlas。
- Mask/Stencil。
- 排序关系。

图集的主要作用之一是让多个 Image 使用同一纹理资源，从而增加合批机会。

## 5.4 Layout Rebuild

`LayoutGroup`、`ContentSizeFitter` 等可能导致布局重新计算。

复杂嵌套 + 高频修改容易产生较大的 CPU 开销。

优化思路：

- 减少 Layout 深度。
- 避免每帧改布局属性。
- 静态布局尽量预计算。

## 5.5 EventSystem / Raycaster

UGUI 输入链通常涉及：

```text
Input
→ EventSystem
→ Raycaster
→ Raycast Result
→ Event Handler
```

Canvas 中 Graphic 是否参与 Raycast，可以通过 Raycast Target 控制。

大量无需交互的 Image/Text 应关闭 Raycast Target。

## 5.6 UI 适配

重点是：

- CanvasScaler。
- Reference Resolution。
- Anchor。
- Pivot。
- Safe Area。
- 不同宽高比适配。

## 5.7 长列表 / 背包

大量 Item 不应该全部创建并长期激活。

常见方案：

```text
Viewport
  ↓
计算可见范围
  ↓
复用有限 Item View
  ↓
滚动时重新绑定数据
```

也就是虚拟列表 / 循环列表。

## 5.8 UI 性能关注点

- Canvas Rebuild。
- Layout Rebuild。
- DrawCall。
- Overdraw。
- Mask。
- Text Mesh 重建。
- GraphicRaycaster。
- SetActive。
- 长列表。
- 动态图集/字体。

---

# 第六部分：性能优化【P0】

性能问题必须先定位瓶颈，再优化。

## 6.1 CPU

重点：

- Script Update。
- GC。
- Physics。
- Animation。
- Canvas Rebuild。
- 高频查找。
- 算法复杂度。

优化手段：

- 减少每帧工作。
- 缓存查询结果。
- 分帧。
- 批处理。
- Job/Burst（合适场景）。
- 对象池。

## 6.2 GPU

重点：

- DrawCall。
- SetPass。
- Shader 复杂度。
- Overdraw。
- 像素填充率。
- 阴影。
- 后处理。
- 分辨率。

## 6.3 Memory

区分：

- Managed Heap。
- Native Heap。
- Texture / Mesh。
- GPU Memory。
- AssetBundle/Addressables 资源。

优化方向：

- 资源压缩。
- 生命周期管理。
- Release。
- 避免无界缓存。
- 控制对象池容量。

## 6.4 IO / Loading

核心思路：

- 异步加载。
- 预加载。
- 分帧初始化。
- 避免一次性主线程反序列化大量数据。
- Loading Screen 隐藏不可避免的加载过程。

## 6.5 为什么连续内存更好

现代 CPU 以 Cache Line 为单位读取数据。

连续数据具有空间局部性：

```text
访问 array[0]
→ 邻近 array[1..n] 一并进入 Cache
```

因此数组、vector、ECS Chunk 等高频遍历结构通常比随机指针跳转更 Cache Friendly。

这也是为什么 `vector` 在很多实际遍历场景中即使理论插入复杂度更高，也可能比 `list` 更快。

## 6.6 性能问题标准思路

```text
先确认现象
→ Profiler 定位 CPU/GPU/Memory/IO
→ 找最大瓶颈
→ 提出针对性方案
→ 对比优化前后数据
→ 检查副作用
```

不要一看到卡顿就直接说“减少 DrawCall”或“对象池”。

---

# 第七部分：计算机图形学【P0-P1】

## 7.1 渲染管线

简化理解：

```text
模型顶点
→ Vertex Shader
→ Clip Space
→ Primitive Assembly
→ Rasterization
→ Fragment/Pixel Shader
→ Depth/Stencil Test
→ Blend
→ FrameBuffer
```

## 7.2 MVP

常见变换：

```text
Local Space
→ Model
World Space
→ View
View Space
→ Projection
Clip Space
→ Perspective Divide
NDC
→ Viewport
Screen Space
```

## 7.3 Depth 与 Transparency

不透明物体通常可以依赖深度测试和深度写入。

透明物体往往需要：

- 深度测试。
- 通常关闭深度写入。
- 后向前排序。
- Alpha Blend。

透明排序问题是实时渲染常见难点。

## 7.4 DrawCall

DrawCall 是 CPU 向 GPU 提交一次绘制任务的调用。

优化目标不是“数量越少越好”，而是减少无意义状态切换和提交开销。

常见技术：

- Static Batching。
- Dynamic Batching。
- GPU Instancing。
- SRP Batcher。
- Mesh 合并。
- Atlas。

## 7.5 Texture

重点：

- 分辨率。
- 压缩格式。
- Mipmap。
- Filter Mode。
- Wrap Mode。
- Atlas。

Mipmap 用空间换取远距离采样质量和 Cache 效率。

## 7.6 PBR

核心概念：

- Albedo。
- Metallic。
- Roughness/Smoothness。
- Normal。
- BRDF。
- Energy Conservation。

不需要推公式，但要理解为什么 PBR 能在不同光照环境下保持相对一致的材质表现。

## 7.7 Anti-Aliasing

常见：

- MSAA。
- FXAA。
- SMAA。
- TAA。

需要理解它们大致是在哪个阶段解决锯齿、成本和副作用。

## 7.8 空间加速

常见结构：

- QuadTree。
- Octree。
- BVH。
- Grid。

本质都是减少“所有对象两两检测”的搜索空间。

---

# 第八部分：网络【P0】

## 8.1 网络分层

常见五层模型：

```text
应用层       HTTP / 游戏协议
传输层       TCP / UDP
网络层       IP
数据链路层   Ethernet / Wi-Fi
物理层       信号传输
```

## 8.2 TCP vs UDP

### TCP

特点：

- 面向连接。
- 可靠。
- 有序。
- 字节流。
- 有流量控制和拥塞控制。

### UDP

特点：

- 无连接。
- 不保证可靠。
- 不保证顺序。
- 数据报。
- 开销较小、应用控制更灵活。

游戏不是简单“UDP 快所以用 UDP”，而是因为某些实时数据不需要 TCP 强制提供的全可靠、有序语义。

## 8.3 三次握手

核心目的：双方确认彼此收发能力，并建立初始序列号状态。

三次而不是两次的重要原因之一，是服务器需要确认客户端也收到了服务器的响应，避免仅凭旧 SYN 建立错误连接。

## 8.4 四次挥手

TCP 是全双工，因此两个方向需要分别关闭。

通常：

```text
FIN
ACK
FIN
ACK
```

## 8.5 TCP 可靠性

主要依靠：

- Sequence Number。
- ACK。
- Checksum。
- RTO。
- Fast Retransmit。
- Sliding Window。
- Flow Control。
- Congestion Control。

### 如何判断丢包

TCP 无法直接观察“某包在网络中消失”，只能推断：

- 超时未收到确认 → RTO 重传。
- 连续收到重复 ACK → 快速重传。

快速重传负责尽快发现，RTO 负责兜底。

## 8.6 粘包与拆包

TCP 是字节流，没有消息边界。

例如发送两次：

```text
ABC
DEF
```

接收端可能读取成：

```text
AB
CDEF
```

因此应用层协议需要定义边界，例如：

- 固定长度。
- 长度字段 + Body。
- 分隔符。

## 8.7 HTTP / HTTPS

HTTP 是应用层 Request/Response 协议。

HTTP/1.1、HTTP/2 通常基于 TCP；HTTP/3 基于 QUIC/UDP。

HTTP 规定：

- Method。
- URL。
- Header。
- Body。
- Status Code。

HTTPS = HTTP + TLS，增加：

- 加密。
- 完整性。
- 身份认证。

## 8.8 Socket

Socket 是应用程序访问传输层通信能力的编程抽象。

可以创建：

- TCP Socket。
- UDP Socket。

它不是一种新的网络协议。

## 8.9 Reliable UDP

如果业务需要在 UDP 上实现可靠性，需要自行补：

- Sequence。
- ACK。
- 重传。
- 去重。
- 乱序重排。
- 滑动窗口。

优势在于可以针对不同消息定义不同可靠等级。

## 8.10 KCP

KCP 是建立在 UDP 上的可靠传输协议，目标之一是降低传统 TCP 某些场景下的重传等待延迟。

需要理解它不是“UDP 自带可靠”，而是在 UDP 之上实现可靠机制。

## 8.11 状态同步与帧同步

### 状态同步

服务器同步世界状态：

```text
Server 状态
→ Client
→ 插值/表现
```

特点：实现相对直观，带宽压力可能更大。

### 帧同步

同步输入或指令：

```text
Input
→ Server 汇总
→ 每个客户端执行同一逻辑
```

特点：带宽低，但要求确定性，并且需要处理延迟、丢包和状态一致性。

## 8.12 实时游戏进阶

需要知道基本概念：

- Snapshot。
- Interpolation。
- Extrapolation。
- Client Prediction。
- Server Reconciliation。
- Rollback。
- Authority。

---

# 第九部分：操作系统 / 多线程【P1】

## 9.1 进程、线程、协程

- Process：资源隔离和程序运行单位。
- Thread：CPU 调度执行单位之一，共享进程内存。
- Coroutine：用户态协作式任务抽象，通常不是 OS 线程。

## 9.2 Stack vs Heap

Stack：

- 调用帧。
- 生命周期规则明确。
- 分配释放快。

Heap：

- 生命周期更灵活。
- 需要 allocator / GC 管理。

不要简单把“值类型=栈、引用类型=堆”绑定到这里。

## 9.3 虚拟内存

虚拟地址把进程看到的地址空间与真实物理内存解耦。

作用：

- 地址空间隔离。
- 权限保护。
- 更灵活的内存映射。
- 支持分页和换页。

## 9.4 CPU Cache

层级：

```text
Register
→ L1
→ L2
→ L3
→ RAM
```

核心概念：

- 时间局部性。
- 空间局部性。
- Cache Line。
- Cache Miss。

这部分和“连续内存”“ECS/DOD”“vector vs list”直接相关。

## 9.5 IO 多路复用

重点了解 select/poll/epoll 的设计目的：

> 一个线程同时等待多个 IO 事件，而不是一个连接绑定一个阻塞线程。

---

# 第十部分：Lua / xLua / HybridCLR【P1】

## 10.1 Lua 基础

重点：

- 动态类型。
- Table。
- Function First-Class。
- Closure。
- Metatable。

## 10.2 Table

Lua Table 同时承担数组和哈希表能力。

因此既可以：

```lua
t[1]
```

也可以：

```lua
t.name
```

## 10.3 Lua GC

核心仍是可达性分析和 Mark/Sweep 思想，并具有增量/分代等机制。

需要理解三色标记、Write Barrier 的基本意义。

## 10.4 C# ↔ Lua

常见方案如 xLua，需要解决：

- 类型转换。
- 方法调用。
- Delegate。
- Reflection / CodeGen。

大量高频调用如果完全依赖反射可能成本较高，因此框架会做代码生成和绑定优化。

## 10.5 热更新

热更新核心问题：

- 哪些逻辑可以动态更新。
- 资源如何更新。
- AOT 平台限制。
- 版本兼容。
- Patch 生命周期。

## 10.6 HybridCLR

核心理解：

- Unity IL2CPP 是 AOT。
- HybridCLR 为 AOT 环境补充动态执行热更新程序集的能力。
- 需要处理 AOT 泛型、补充元数据等问题。

---

# 第十一部分：数据结构与算法【P0】

## 11.1 复杂度

必须熟悉：

```text
O(1)
O(log n)
O(n)
O(n log n)
O(n²)
```

同时理解：

> Big-O 只描述增长趋势，不代表真实运行时间，常数、Cache 和数据规模同样重要。

## 11.2 排序

重点：

- 快速排序。
- 归并排序。
- 堆排序。
- 桶/计数排序适用条件。

背包排序通常直接使用框架排序即可，除非数据规模和规则明确要求专项优化。

## 11.3 高频题型

必须熟练：

- HashMap。
- 双指针。
- 滑动窗口。
- 前缀和。
- 单调栈。
- 链表。
- 二叉树。
- BFS / DFS。
- Heap / TopK。
- LRU。

## 11.4 A*

核心公式：

```text
f(n) = g(n) + h(n)
```

- `g`：起点到当前节点真实成本。
- `h`：当前节点到终点估价。

如果 heuristic 不高估真实成本，A* 可以保证最优性。

## 11.5 TopK

求最大的 K 个元素通常维护一个大小为 K 的**小顶堆**：

- 堆顶是 TopK 中最小的。
- 新元素比堆顶大就替换。

## 11.6 游戏几何

常见：

- Dot：夹角/方向判断。
- Cross：法向和左右关系。
- 点到线/平面。
- 扇形攻击范围。
- Raycast。

攻击扇形判断常见组合：

```text
Distance²
→ Dot
→ 必要时 Raycast 做遮挡
```

---

# 第十二部分：游戏系统设计【P0-P1】

## 12.1 背包

典型数据：

```text
ItemConfig
→ ItemInstance / Stack
→ Inventory Model
→ ViewModel/Controller
→ UI
```

关注：

- Stack。
- Sort。
- Filter。
- Save。
- ItemId 到 Config 查询。
- 局部刷新。

## 12.2 Buff

推荐拆分：

```text
BuffConfig
→ BuffInstance
→ BuffContainer
→ Modifier / Effect
→ Attribute
```

配置和运行时实例分离。

重点：

- Duration。
- Stack。
- Refresh。
- Period。
- Infinite。
- Remove。

## 12.3 Attribute

属性系统通常区分：

- BaseValue。
- Modifier。
- Current/Final Value。

需要处理属性之间依赖，例如 MaxHealth 变化时 Health 的约束。

## 12.4 Quest

常见流程：

```text
Quest Config
→ Accept
→ Runtime State
→ Objective Progress
→ Complete
→ Submit
→ Reward
```

任务监听应与具体 UI 解耦。

## 12.5 Red Dot

红点本质是状态聚合和通知问题。

常见树：

```text
Root
├── Bag
│   ├── Weapon
│   └── Artifact
└── Quest
```

叶节点状态变化向父节点聚合，UI 只绑定对应节点。

## 12.6 UI Manager

需要负责：

- Open/Close。
- Layer。
- Window Stack。
- Resource Loading。
- Cache/Pool。
- Input Focus。

## 12.7 Combat / Character Controller

复杂角色系统最重要的是“竞争仲裁”。

例如：

```text
Locomotion
Attack Ability
Hit
Dodge
Root Motion
```

不能让多个模块直接同时写 CharacterController/Transform。

通常需要统一 Motion/Action Arbitration 层决定：

- 谁当前拥有移动权。
- 谁能覆盖旋转。
- 谁可以被打断。
- 优先级和 Tag/状态条件。

---

# 第十三部分：调试与工程能力【P1】

## 13.1 Bug 排查

标准流程：

```text
稳定复现
→ 缩小范围
→ 收集日志/Profiler/调用栈
→ 建立假设
→ 验证
→ 修复
→ 回归测试
```

不要只回答“Debug 打断点”。

## 13.2 Unity 工具

重点熟悉：

- Profiler。
- Memory Profiler。
- Frame Debugger。
- Rendering Debugger。
- Console / StackTrace。
- Deep Profile 的代价。

## 13.3 Git

至少理解：

- branch。
- merge。
- rebase。
- cherry-pick。
- revert。
- reset。
- conflict。

## 13.4 跨平台

Unity 能跨平台的原因不是“一份 C# 代码自动到处运行”这么简单，而是：

```text
Unity API
→ 平台抽象层
→ 平台后端
```

代码执行还涉及 Mono / IL2CPP，各平台图形 API、文件系统、输入、权限也不同。

---

# 第十四部分：HR / 行为面【P0】

## 14.1 自我介绍

建议结构：

```text
背景
→ 技术方向
→ 代表项目
→ 最有价值的技术点
→ 为什么匹配岗位
```

## 14.2 为什么游戏 / 为什么客户端

不要只说“喜欢玩游戏”。应说明：

- 对实时交互和复杂系统感兴趣。
- 喜欢解决性能、架构、表现等综合问题。
- 过去项目经历与客户端开发方向一致。

## 14.3 游戏理解

至少准备 1～2 款目标公司的产品，从：

- 核心玩法。
- UI/交互。
- 战斗反馈。
- 技术实现猜测。
- 优缺点。

进行分析。

## 14.4 团队协作

准备：

- 与策划意见冲突。
- 与美术资源规范沟通。
- 与测试排 Bug。
- 需求变化。

回答重点是“如何基于目标和数据解决问题”，而不是证明自己一定正确。

## 14.5 家庭 / 城市 / 稳定性

HR 通常是在评估：

- 入职意愿。
- 城市稳定性。
- 长期规划。

可以回答与职业规划相关的信息，不需要主动暴露无关隐私。

## 14.6 横评

横评意味着公司正在把同一 HC 的多位候选人进行综合比较，并不等于已经发 Offer。

通常比较：

- 技术能力。
- 项目。
- 岗位匹配。
- 沟通。
- 入职时间。
- 稳定性。

---

# 第十五部分：AI 辅助开发【P2】

2026 游戏客户端面试开始逐渐出现 AI Coding / Agent 工程化问题。

重点不是“会不会用 ChatGPT”，而是：

- 如何给 AI 正确上下文。
- 如何限制 AI 不破坏架构。
- 如何拆分大型代码库任务。
- 如何验证生成代码。
- 哪些问题不能完全交给 AI。

可靠流程：

```text
定义任务
→ 提供局部上下文和约束
→ 生成/修改
→ 编译
→ 测试
→ 静态检查
→ Code Review
→ Profiler 验证性能敏感代码
```

核心观点：

> AI 是工程效率工具，但架构判断、性能敏感路径和关键业务逻辑仍需要工程师负责。

---

# 第十六部分：高频追问链

## Dictionary

```text
Dictionary 原理
→ HashCode
→ Bucket / Entry
→ Collision
→ Equals
→ Key 为什么不能变
→ Resize
→ 为什么平均 O(1)
→ 为什么小数据 List 可能更快
→ ConcurrentDictionary
```

## GC

```text
GC
→ Root
→ Reachability
→ Mark/Sweep/Compact
→ Generation
→ Incremental GC
→ GC.Alloc
→ 如何优化
→ 为什么仍会泄漏
```

## TCP

```text
TCP vs UDP
→ 三次握手
→ Sequence / ACK
→ RTO
→ Fast Retransmit
→ Sliding Window
→ Flow Control
→ Congestion Control
→ HOL Blocking
→ 游戏为什么考虑 UDP / KCP
```

## UGUI

```text
Canvas
→ Mesh / Batch
→ Rebuild
→ Dirty
→ 动静分离
→ Nested Canvas
→ Atlas
→ Mask
→ Overdraw
→ Virtual List
→ Profiler
```

## DrawCall

```text
DrawCall
→ 为什么有 CPU 成本
→ Material / State Change
→ Batching
→ GPU Instancing
→ SRP Batcher
→ 为什么不能无脑合批
→ CPU/GPU Bottleneck 判断
```

## Coroutine

```text
IEnumerator
→ 编译器状态机
→ PlayerLoop
→ yield
→ Coroutine vs Thread
→ Coroutine vs async
→ 生命周期
```

## Quaternion

```text
Euler
→ Gimbal Lock
→ Axis-Angle
→ Quaternion
→ 四个分量含义
→ Quaternion Multiplication
→ Slerp
```

## Object Pool

```text
为什么池化
→ Get/Release
→ Spawn/Reset
→ 容量
→ 内存换 CPU
→ 哪些对象不适合
→ Addressables 引用怎么处理
```

## Addressables

```text
Address
→ LoadAsync
→ Handle
→ Dependencies
→ Reference Count
→ Release
→ Native Memory
→ Scene/Pool 生命周期
```

## FSM

```text
State
→ Transition
→ Enter/Update/Exit
→ State Explosion
→ HFSM
→ Parallel State
→ Arbitration
→ Ability vs Locomotion
```

---

# 第十七部分：高风险易错点

1. **值类型不一定在栈，引用类型也不能简单理解为“变量就在堆里”。**
2. **`string.Empty` 与 `""` 不应解释成“一个分配一个不分配”。**
3. **Dictionary 的具体取模实现不要死背某个 .NET 版本细节。**
4. **HTTP 不一定基于 TCP，HTTP/3 使用 QUIC。**
5. **UDP 不等于一定更快，只是可靠性策略更少、控制更自由。**
6. **Coroutine 不比 Update 天然更快，本质仍在 PlayerLoop 推进。**
7. **DrawCall 优化和 Canvas Rebuild 优化是不同问题。**
8. **Quaternion 不是简单保存四个角度，也不应把 x/y/z 直接解释成旋转轴。**
9. **有 GC 仍然可能出现托管对象长期存活和 Native 资源泄漏。**
10. **Unity 合批条件会随渲染管线和版本变化，应理解机制而不是死背老版本数字。**

---

# 第十八部分：复习顺序建议

## 第一阶段：语言与基础

- C# 内存模型。
- GC。
- Delegate/Event。
- Reflection。
- List/Dictionary。
- async/await。
- C++ 虚函数和智能指针。
- 每天算法练习。

## 第二阶段：Unity

- 生命周期。
- Coroutine。
- Object Pool。
- Serialization。
- Addressables。
- Physics。
- Animation。
- UGUI。

## 第三阶段：性能 + 网络 + 图形

- CPU/GPU/Memory。
- Profiler。
- TCP/UDP/HTTP/KCP。
- 状态同步/帧同步。
- Render Pipeline。
- DrawCall。
- Depth/Transparency。

## 第四阶段：项目输出

对简历上的每个技术名词至少准备：

```text
1. 是什么？
2. 为什么项目用了它？
3. 怎么实现？
4. 遇到了什么问题？
5. 有什么缺点？
6. 如果重做怎么改？
```

---

# 第十九部分：面试前 P0 Checklist

## C#

- 值类型 / 引用类型。
- 装箱拆箱。
- GC。
- string。
- Delegate / Event。
- Reflection / Metadata / DLL。
- List / Dictionary。
- async / await。

## C++

- 指针引用。
- 虚函数。
- 智能指针。
- vector / list / map / unordered_map。

## Unity

- 生命周期。
- Coroutine。
- Object Pool。
- Resources / AB / Addressables。
- Physics。
- Animation / Root Motion。
- Quaternion。

## UGUI

- Canvas。
- Rebuild。
- Batch。
- 动静分离。
- Layout。
- EventSystem。
- UI Adaptation。
- Virtual List。

## Performance

- Profiler 定位流程。
- CPU。
- GPU。
- Memory。
- DrawCall。
- Cache / 连续内存。

## Network

- TCP / UDP。
- 三次握手。
- TCP 可靠性。
- ACK / RTO / Fast Retransmit。
- 粘包拆包。
- HTTP / HTTPS。
- Reliable UDP / KCP。
- 状态同步 / 帧同步。

## Graphics

- Rendering Pipeline。
- MVP。
- Depth。
- Transparency。
- DrawCall。
- Texture/Mipmap。

## Algorithm

- Hash。
- Heap / TopK。
- Linked List。
- Tree。
- BFS / DFS。
- Sliding Window。
- Prefix Sum。
- A*。

## Project

- 项目架构。
- 个人职责。
- 最难问题。
- 技术选型。
- 性能优化。
- Bug 排查。
- 系统扩展性。

---

# 第二十部分：这份总结与“含回答版”的配合方式

建议使用三层复习结构：

```text
第一层：本文件
《Unity游戏客户端面试知识点总结_2026.md》
作用：建立完整知识体系。

第二层：原知识库前半部分
作用：查看题目、优先级和追问方向。

第三层：《Unity游戏客户端面试知识库_2026_含回答.md》中的标准回答库
作用：训练 60～90 秒口述答案。
```

最终目标不是记住文档，而是能够在没有文档的情况下做到：

```text
面试官提出一个知识点
→ 先给定义
→ 解释底层原理
→ 说明优缺点/边界
→ 给 Unity/游戏项目中的实际场景
→ 主动承接下一层追问
```

