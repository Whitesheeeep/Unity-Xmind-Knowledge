# Unity / 游戏客户端开发面试知识库（2026版）

> 目标岗位：Unity / 游戏客户端开发（实习、校招、初级客户端）
>
> 构建依据：用户现有《面试》《UGUI 优化》《八股文_2025》三份 XMind + 2025–2026 牛客游戏客户端/Unity 面经 + 游戏客户端高频题统计。
>
> 使用方法：不要按“背题”复习，而要按“定义 → 原理 → 为什么 → 边界/缺点 → 项目应用 → 追问”形成知识链。

---

## 0. 优先级说明

- **P0：必须掌握**。应做到 1–2 分钟完整回答，追问 2–3 层不崩。
- **P1：高频进阶**。至少掌握原理、典型应用和一个项目例子。
- **P2：大厂/方向加分**。根据目标公司和简历选择性深入。

建议复习顺序：

1. 项目深挖与场景设计
2. C# / C++ 与数据结构
3. Unity 核心与资源管理
4. UGUI / 性能优化
5. 网络同步
6. 图形学
7. 算法
8. Lua / 热更新
9. OS / 多线程
10. HR / 游戏理解 / AI 工程能力

---

# 第一部分：项目深挖【P0，最高优先级】

近年游戏客户端面试最明显的特点是：八股是入口，项目是主战场。任何写在简历上的名词，都默认可以被追问到底层。

## 1.1 每个项目必须能回答的 12 个问题

1. 项目是什么？用户/玩法目标是什么？
2. 你负责哪些模块？哪些不是你做的？
3. 项目的整体架构是什么？为什么这样分层？
4. 数据如何流动？输入从哪里进入，最后如何影响表现？
5. 模块之间如何通信？直接引用、接口、事件、消息总线分别怎么选？
6. 最难的问题是什么？为什么难？
7. 你尝试过哪些方案？为什么最终选当前方案？
8. 这个方案的缺点是什么？如果重做你会怎么改？
9. 性能瓶颈在哪里？用什么工具定位？优化前后数据是多少？
10. 如何处理异常、取消、对象销毁、场景切换等生命周期问题？
11. 如何测试？如何定位偶现 Bug？
12. 如果玩家量/数据量扩大 10 倍、100 倍，当前设计哪里先出问题？

## 1.2 架构追问链

### MVC / MVVM
- MVC、MVVM 各解决什么问题？
- Controller/ViewModel 生命周期跟谁？
- View 是否应该直接访问 Model？
- 数据变化如何通知 UI？
- 全量刷新和局部刷新怎么选？
- 为什么不直接让业务对象操作 UI？

### EventBus / Observer
- 委托、事件、观察者模式区别？
- EventBus 是否会导致调用链不可追踪？
- 如何解除监听、防止对象泄漏？
- 同步事件与异步事件如何处理？
- 为什么有些模块宁愿直接接口调用，也不用事件？

### FSM / HFSM / Behavior Tree
- FSM 状态切换条件如何组织？
- 状态爆炸怎么解决？
- HFSM 切换到父状态/子状态时默认进入哪个状态？
- 并行状态怎么处理？
- 攻击、移动、受击、闪避产生竞争时谁仲裁？
- 行为树 Selector / Sequence 的语义？
- BT 与 FSM 的职责边界？

### Gameplay / GAS 类设计
- Ability、Effect、Attribute、Tag 各自职责？
- Buff 为什么配置与 Runtime 分离？
- Infinite Effect 如何移除？
- 同一个 Ability 能否存在多个运行时实例？
- 技能、Locomotion、Root Motion 如何仲裁？
- 技能结束如何判定？动画结束是否等于 Ability End？

---

# 第二部分：C# / .NET【P0】

## 2.1 类型系统与内存

### 必会
- 值类型和引用类型区别。
- 值类型是否一定在栈？为什么不是？
- class 与 struct 的区别及选择原则。
- 为什么 Unity 的 Vector3 是 struct？
- ref / out / in 的区别。
- nullable value type。

### 追问链
- struct 作为 class 字段存在哪里？
- struct 放进 `object` / 接口变量会发生什么？
- 大 struct 为什么可能反而更慢？
- struct 复制语义会产生什么问题？

## 2.2 装箱与拆箱【P0】

必须说明：
- 装箱：值类型 → object / 接口，需要在托管堆创建对象并复制值。
- 拆箱：先检查类型，再取出值。
- 为什么会产生 GC 压力。
- 泛型为什么能减少装箱。
- struct 调用接口为什么可能装箱。
- foreach、Enum、可变参数、日志拼接等常见隐式 GC 来源。

## 2.3 GC【P0】

### C# / .NET
- GC Root 是什么。
- 可达性分析。
- Mark / Sweep / Compact。
- Gen0 / Gen1 / Gen2。
- 大对象与长生命周期对象。
- GC 为什么会暂停程序。

### Unity
- Managed Heap 与 Native 内存区别。
- Incremental GC 的目的与局限。
- `GC.Alloc` 如何在 Profiler 中定位。
- 常见 GC 来源：临时 List、闭包、LINQ、字符串拼接、装箱、频繁 new、委托捕获。
- 对象池到底减少什么，不是什么对象都值得池化。

### 追问
- 有 GC 是否仍然会内存泄漏？
- 静态事件为什么可能让对象无法回收？
- AssetBundle/Texture 等 UnityEngine.Object 为什么不能只靠 C# GC？

## 2.4 string / StringBuilder【P0】

- string 不可变。
- 字符串驻留（interning）的基本概念。
- `""` 与 `string.Empty` 语义上等价，不要把它理解成每次都会新分配一个空字符串。
- 连续拼接为什么可能产生多个临时字符串。
- StringBuilder 的容量、扩容和适用场景。

## 2.5 委托、事件、Lambda、闭包【P0】

- Delegate 本质：类型安全的函数引用/调用列表。
- multicast delegate。
- event 对 delegate 的访问约束。
- Action / Func。
- Lambda 捕获变量形成闭包。
- 闭包为什么可能产生堆分配。
- Unity 中事件不解绑可能有什么生命周期问题。

## 2.6 泛型【P1】

- 泛型解决什么问题：类型安全 + 减少装箱 + 代码复用。
- 泛型约束。
- `where T : class / struct / new() / interface`。
- 泛型与反射。
- IL2CPP/AOT 下泛型实例化相关问题（了解）。

## 2.7 反射、元数据、Assembly、DLL【P0】

知识链必须串起来：

`C# source → 编译 → Assembly(.dll) → IL + Metadata → Runtime → Type/MethodInfo/FieldInfo → Reflection`

必须回答：
- 元数据是什么？记录类型、字段、方法、继承、Attribute 等描述信息。
- `Type` 是什么？
- `GetMethod` / `GetField` 为什么能工作？
- `MethodInfo.Invoke` 为什么比直接调用慢？
- Attribute 与反射如何配合？
- Unity/Editor/Odin/DI/自动注册中的反射应用。
- IL2CPP + stripping 为什么会影响反射？
- `[Preserve]` / link.xml 的意义。

## 2.8 Dictionary / List【P0】

### Dictionary
- buckets + entries。
- HashCode → bucket → Entry → Equals。
- 哈希冲突。
- 扩容。
- 为什么 key 不应该在加入后改变 hash/equality 相关状态。
- Dictionary 为什么不是线程安全的。
- ConcurrentDictionary 的定位。

### List
- 动态数组。
- Count vs Capacity。
- 扩容与复制。
- Insert/Remove 的成本。
- 为什么连续内存 Cache Friendly。

### 高频比较
- List vs LinkedList。
- List vs Dictionary：小数据量/遍历型场景为什么 List 常更合适。
- Dictionary O(1) 为什么不意味着永远更快。

## 2.9 async / await / Task【P0-P1】

- Task 不等于线程。
- async/await 编译器状态机。
- await 未完成时注册 continuation，而不是一直占用线程。
- `GetAwaiter().GetResult()` 是同步阻塞。
- CancellationToken 是协作式取消。
- 底层异步不支持取消时，只能“取消等待”，不能强杀任务。
- Task.WhenAny 封装取消的语义。
- Unity 主线程上下文和 Unity API 线程限制。

## 2.10 锁与线程安全【P1】

- lock 与 Monitor。
- Mutex / Semaphore / SpinLock / ReaderWriterLockSlim 基本差异。
- 死锁四条件。
- 乐观锁 / 悲观锁。
- 原子操作 / Interlocked。
- 为什么锁粒度越大并发性越差。

---

# 第三部分：C++【冲大厂 P0，普通 Unity 岗 P1】

即使目标是 Unity，腾讯、网易、库洛等客户端岗位仍经常用 C++ 基础筛底层能力。

## 3.1 指针 / 引用
- 指针是什么。
- 指针与引用区别。
- nullptr。
- 野指针、悬空指针。
- void*。
- const pointer / pointer to const。

## 3.2 多态与虚函数【高频】
- 静态多态 vs 动态多态。
- vptr / vtable。
- 虚函数调用流程。
- 构造函数为什么不能 virtual。
- 析构函数为什么常需要 virtual。
- 纯虚函数、抽象类。
- 对象内存布局。

## 3.3 智能指针【高频】
- unique_ptr / shared_ptr / weak_ptr。
- shared_ptr 控制块。
- 引用计数。
- 循环引用。
- shared_ptr 的线程安全边界。
- make_shared 的意义。
- RAII。

## 3.4 move / 右值 / 完美转发【P1】
- 左值、右值。
- 右值引用。
- move 只是转换值类别，不负责真正移动。
- 移动构造。
- std::forward 与完美转发。

## 3.5 内存对齐【P1】
- 为什么需要 alignment。
- struct/class 大小计算。
- padding。
- cache line 与 false sharing（加分）。

## 3.6 STL【P0-P1】
- vector 扩容、迭代器失效。
- vector vs list。
- map vs unordered_map。
- 红黑树基本性质。
- emplace_back vs push_back。
- erase 的复杂度。

## 3.7 编译与链接【P1】
- 预处理 → 编译 → 汇编 → 链接。
- 静态链接 vs 动态链接。
- 符号解析。
- DLL / so 基本概念。

---

# 第四部分：Unity 核心【P0】

## 4.1 MonoBehaviour 生命周期

至少能完整讲：

`Awake → OnEnable → Start → Update/FixedUpdate/LateUpdate → OnDisable → OnDestroy`

追问：
- Awake 与 Start 区别。
- Update / FixedUpdate / LateUpdate 用途。
- Time.timeScale=0 时 Update 是否调用？deltaTime 如何？
- Physics 为什么通常配合 FixedUpdate。
- Script Execution Order 的风险。

## 4.2 Coroutine【P0】

- Coroutine 不是线程。
- IEnumerator + 状态机。
- Unity 在 PlayerLoop 中推进。
- `yield return null`、WaitForSeconds、WaitUntil 的意义。
- Coroutine 与 Update 的本质区别：调度/代码组织，不是自动变快。
- Coroutine 与 async/await 的区别。
- GameObject disable/destroy 后协程生命周期。

## 4.3 GameObject / Component / Transform
- Component 模型。
- GetComponent 成本与缓存。
- Transform 层级。
- local/world coordinate。
- 矩阵变换。
- SetParent 的 worldPositionStays。

## 4.4 Unity 序列化
- Unity 能序列化哪些字段。
- `[SerializeField]`。
- UnityEngine.Object 引用。
- ScriptableObject 的定位。
- JsonUtility 与一般 JSON 库差异。
- 配置数据与运行时状态分离。

## 4.5 Instantiate / Destroy / Object Pool【P0】
- Instantiate 的 CPU、内存、生命周期成本。
- Destroy 并非立即销毁所有底层资源。
- Pool：Create / Spawn / Despawn / Reset。
- 为什么 OnSpawn / OnDespawn 比只有 Get/Release 更清晰。
- 池中对象状态污染。
- 池大小策略。
- 不值得池化的对象。

## 4.6 Resources / AssetBundle / Addressables【P0】

### Resources
- 简单，但资源管理粗粒度，容易扩大包体和内存。

### AssetBundle
- Bundle 与 Asset 的关系。
- 依赖关系。
- LoadAsset / Unload。
- `Unload(false)` vs `Unload(true)` 的语义必须理解。
- 重复依赖导致资源冗余。

### Addressables
- Address → Location → Provider → Handle。
- AsyncOperationHandle 生命周期。
- 引用计数。
- Release 时机。
- 资源依赖与远端更新。
- 为什么 Addressables 不是“自动帮你解决一切内存问题”。

## 4.7 场景加载
- LoadScene vs LoadSceneAsync。
- 异步加载为什么仍可能卡顿。
- 激活阶段。
- Loading 页面。
- 大场景分块 / Streaming。

## 4.8 物理系统
- Collider vs Trigger。
- Rigidbody。
- Raycast。
- LayerMask。
- Collision Matrix。
- FixedUpdate。
- Continuous collision detection 基础。
- OverlapSphereNonAlloc 等无分配查询。

## 4.9 动画【P1】
- Animator / Animation。
- Animator State Machine。
- Blend Tree。
- Layer + Avatar Mask。
- Root Motion。
- OnAnimatorMove / deltaPosition / deltaRotation。
- 骨骼动画与 skinning 基础。
- IK 基础。
- 动画与技能/Locomotion 的职责边界。

## 4.10 四元数与欧拉角【P0】
- 欧拉角：三个轴按顺序旋转，直观但有 Gimbal Lock。
- Quaternion：单位四元数表示整体旋转，常从 axis-angle 得到。
- `(x,y,z,w)` 不是四个角度。
- Quaternion 乘法表示旋转组合。
- LookRotation / AngleAxis / Slerp。
- 为什么维护 yaw/pitch float 再生成 Quaternion 往往比反复读取 eulerAngles 更稳定。

---

# 第五部分：UGUI / UI【P0】

## 5.1 Canvas
- Canvas 的职责。
- Screen Space Overlay / Camera / World Space。
- Canvas rebuild。
- Nested Canvas。
- 一个窗口一个 Canvas 的利弊。

## 5.2 动静分离【P0】
- 目标：缩小 Canvas Rebuild 范围。
- 静态背景/装饰与高频变化的血条/倒计时分离。
- Canvas 不能无限拆，因为会破坏批处理并增加管理开销。
- 本质是在 CPU rebuild 与 DrawCall 之间权衡。

## 5.3 UGUI 合批【P0】
- 相同材质/纹理只是条件之一。
- UI 渲染顺序与层级穿插会打断 batch。
- Atlas 的意义。
- Mask 可能带来的额外材质/Stencil 开销。
- RectMask2D 与 Mask 的差异。

## 5.4 Layout Rebuild【P0-P1】
- LayoutGroup。
- ContentSizeFitter。
- LayoutElement。
- 为什么深层嵌套自动布局昂贵。
- 高频变化文本/尺寸为什么容易触发布局脏标记。
- 尽量局部刷新。

## 5.5 EventSystem / Raycaster【P1】
- EventSystem。
- GraphicRaycaster。
- PointerEventData。
- Raycast Target。
- UI 点击穿透。
- Scene Raycast 与 UI EventSystem 如何协调。

## 5.6 UI 适配【P0】
- Anchor / Pivot。
- CanvasScaler。
- Scale With Screen Size。
- matchWidthOrHeight。
- Safe Area。
- 刘海屏/折叠屏/异形屏。

## 5.7 长列表 / 背包
- ScrollRect。
- 对象池。
- 可视区域复用。
- 虚拟列表。
- 增量刷新。
- 排序与索引。

## 5.8 UI 性能检查表
- 减少 Canvas Rebuild。
- 降低 Overdraw。
- Atlas 合理打包。
- 关闭无用 RaycastTarget。
- 减少 Mask。
- 避免频繁 SetActive/布局重算。
- TextMeshPro 字体 Atlas 管理。
- 动画 UI 单独 Canvas。

---

# 第六部分：性能优化【P0】

面试时不能只说“对象池、少 Update”。应该按 CPU / GPU / Memory / IO 分类。

## 6.1 CPU
- Profiler 找热点。
- 减少频繁反射、LINQ、字符串、GetComponent。
- 降低复杂算法复杂度。
- 批处理重复工作。
- Job System / Burst（加分）。
- Cache locality。

## 6.2 GPU
- DrawCall / SetPassCall。
- Static Batching / Dynamic Batching / GPU Instancing / SRP Batcher。
- Overdraw。
- Fill Rate。
- Shader complexity。
- LOD。
- Occlusion Culling。
- Shadow / Post Processing 成本。

## 6.3 Memory
- Managed / Native / GPU memory。
- Texture 压缩。
- Mesh。
- AudioClip。
- Asset 引用。
- GC allocation。
- AssetBundle/Addressables 生命周期。

## 6.4 IO / Loading
- 异步加载。
- 分帧初始化。
- 预加载。
- 避免大批量同帧 Instantiate。
- 资源分包。

## 6.5 为什么连续内存更好【P0】
- Cache line。
- 空间局部性。
- Cache miss。
- Pointer chasing。
- SIMD/Burst 的基础。
- ECS/DOD 为什么强调数据布局。

## 6.6 性能问题标准回答模板

1. 先确认是 CPU、GPU、内存还是 IO 瓶颈。
2. 使用 Profiler / Frame Debugger / Memory Profiler / RenderDoc 等定位。
3. 找到具体热点，不凭感觉优化。
4. 设计改动。
5. 对比优化前后数据。
6. 验证是否产生副作用。

---

# 第七部分：计算机图形学【P0-P1】

## 7.1 渲染管线【P0】
- Object → World → View → Clip。
- Vertex Shader。
- Primitive Assembly。
- Clipping。
- Perspective Divide。
- Viewport Transform。
- Rasterization。
- Fragment/Pixel Shader。
- Depth/Stencil/Blend。

## 7.2 MVP 矩阵
- Model。
- View。
- Projection。
- 正交 vs 透视。
- 近大远小如何产生。

## 7.3 Depth / Transparency【P0】
- Z Buffer。
- ZTest / ZWrite。
- 不透明物体通常前到后有利于 Early-Z。
- 透明物体通常关闭 ZWrite、保留 ZTest、从后往前排序并 Blend。
- 为什么透明排序无法对任意互相穿插几何完美解决。

## 7.4 DrawCall【P0】
- DrawCall 是 CPU 向 GPU 提交一次绘制命令。
- 状态切换 / SetPass。
- Atlas。
- Batching。
- Instancing。
- SRP Batcher。
- Mesh combine。
- 为什么 DrawCall 不是越少越好。

## 7.5 Texture【P1】
- Mipmap。
- Bilinear / Trilinear / Anisotropic filtering。
- Texture Compression。
- Normal Map。
- Gamma vs Linear。

## 7.6 光照 / PBR【P1】
- Lambert。
- Blinn-Phong。
- BRDF 基础。
- Metallic/Roughness。
- Direct / Indirect lighting。
- Shadow Map。
- AO / SSAO。

## 7.7 Anti-Aliasing【P1-P2】
- MSAA。
- FXAA。
- TAA。
- 各自成本和缺陷。

## 7.8 空间加速结构【P1】
- Quadtree / Octree。
- BVH。
- Spatial Hash。
- 用于碰撞、AOI、可见性、范围查询。

---

# 第八部分：网络【P0】

## 8.1 五层/四层模型
- 应用层。
- 传输层。
- 网络层。
- 数据链路层。
- 物理层。

## 8.2 TCP vs UDP【P0】

TCP：
- 面向连接。
- 字节流。
- 可靠、有序。
- ACK、序号、重传。
- 流量控制、拥塞控制。
- 队头阻塞。

UDP：
- 无连接。
- 数据报。
- 不保证可靠、顺序、去重。
- 开销低、实时性好、业务层可自定义可靠性。

## 8.3 TCP 三次握手【P0】
- SYN。
- SYN+ACK。
- ACK。
- 为什么不是两次。
- 每一次丢包怎么办。
- 初始序列号。

## 8.4 TCP 四次挥手【P1】
- FIN/ACK。
- 半关闭。
- TIME_WAIT 为什么存在。

## 8.5 TCP 可靠性【P0】
- Sequence Number。
- ACK。
- Checksum。
- 超时重传 RTO。
- 快速重传。
- 滑动窗口。
- 流量控制。
- 拥塞控制。

### 丢包判断
TCP 并不能“看到包掉在网络里”，而是根据确认状态推断。重点掌握 RTO 和 Duplicate ACK/Fast Retransmit。

## 8.6 TCP 粘包/拆包【P0】
- TCP 是 byte stream，没有消息边界。
- 定长协议。
- 分隔符。
- Length + Body（游戏最常见）。

## 8.7 HTTP / HTTPS【P0】
- HTTP 是应用层请求/响应协议。
- Method、URL、Header、Body、Status Code。
- HTTP/1.1、HTTP/2 通常基于 TCP。
- HTTP/3 基于 QUIC。
- HTTPS = HTTP over TLS（概念上）。
- HTTP 无状态与 Token/Session。

## 8.8 Socket【P1】
- Socket 是网络编程 API 抽象，不等于 TCP。
- TCP Socket / UDP Socket。
- connect/send/recv 等基本流程。

## 8.9 UDP 实现可靠通信【P0-P1】
必须能自己设计：
- sequence。
- ack。
- 重传队列。
- 超时。
- 去重。
- 乱序缓存。
- 滑动窗口。
- RTT/RTO。
- 拥塞控制是否需要。

## 8.10 KCP【P1】
- 基于 UDP 的 ARQ 可靠传输。
- 相比 TCP 更偏向低延迟策略。
- fast retransmit。
- 可配置窗口/刷新频率。
- 不能简单说“KCP 一定比 TCP 快”。

## 8.11 状态同步 vs 帧同步【P0】

### 状态同步
- 服务端维护权威状态。
- 客户端接收状态并插值/预测表现。
- 容错相对容易。
- 带宽通常较高。

### 帧同步
- 同步输入/指令，各客户端执行确定性逻辑。
- 网络带宽低。
- 对确定性、重连、校验要求高。
- 浮点确定性、随机数、容器遍历顺序等都可能导致不同步。

## 8.12 实时游戏进阶【P1-P2】
- Client Prediction。
- Server Reconciliation。
- Interpolation。
- Extrapolation。
- Rollback。
- Snapshot。
- Tick / Render Frame / Logic Frame。
- Heartbeat。
- Reconnect。
- Idempotency。

---

# 第九部分：操作系统 / 多线程【P1】

## 9.1 进程、线程、协程
- 进程是资源隔离/分配基本单位。
- 线程是 CPU 调度执行单位。
- 协程是用户态协作式调度抽象。
- Unity Coroutine 不等于通用语言层面的 stackful coroutine。

## 9.2 Stack vs Heap
- 生命周期管理方式。
- 分配成本。
- 空间规模。
- 碎片。
- 不要回答“值类型都在栈、引用类型都在堆”。

## 9.3 虚拟内存【P1】
- Virtual Address Space。
- Page Table。
- Paging。
- 地址隔离。
- 按需加载。
- 交换/页面置换基础。

## 9.4 Cache【P0-P1】
- L1/L2/L3。
- Cache Line。
- temporal locality。
- spatial locality。
- cache miss。
- false sharing（加分）。

## 9.5 IO 多路复用【P2】
- select/poll/epoll 基本概念。
- 为什么服务端常问，纯 Unity 客户端不必优先投入太多时间。

---

# 第十部分：Lua / xLua / HybridCLR【P1，大量国内游戏公司加分】

## 10.1 Lua 基础
- 动态类型。
- table。
- closure。
- upvalue。
- metatable。
- `__index` / `__newindex`。
- Lua 如何模拟 OOP。

## 10.2 Lua Table 底层
- array part + hash part。
- ipairs / pairs 语义。
- 扩容/rehash 基础。

## 10.3 Lua GC
- Mark-Sweep。
- Incremental GC。
- 三色标记。
- Write Barrier。
- Lua 5.4 generational mode（了解）。

## 10.4 C# ↔ Lua
- Lua 调 C#。
- C# 调 Lua。
- wrapper / bridge / code generation。
- delegate bridge。
- 反射方式与生成代码方式的性能差异。

## 10.5 热更新
- 为什么 Lua 能热更新：脚本运行时解释/执行，逻辑可以替换而无需重新发布 native binary。
- 已经创建的实例如何受更新影响：取决于实例如何查找函数/原型/元表。
- 资源热更新与代码热更新是两件事。

## 10.6 HybridCLR【P1】
- IL2CPP AOT 的背景。
- 热更新程序集。
- AOT 泛型补充元数据等核心问题只需掌握概念和项目使用链路。
- stripping / metadata / assembly 的知识要与反射部分串起来。

---

# 第十一部分：数据结构与算法【P0】

## 11.1 必会复杂度
- Array/List。
- LinkedList。
- Stack/Queue。
- HashTable。
- Heap。
- BST / balanced tree。
- Graph。

## 11.2 排序【P0】
- QuickSort。
- MergeSort。
- HeapSort。
- insertion / selection / bubble 基础。
- 时间复杂度。
- 空间复杂度。
- 稳定性。
- 快排最坏情况与优化。

## 11.3 高频算法题型
- 双指针。
- 滑动窗口。
- 前缀和。
- HashMap。
- Stack / monotonic stack。
- Binary Search。
- Linked List。
- Tree BFS/DFS。
- Heap / TopK。
- LRU。
- DP 基础。

## 11.4 游戏特色算法【P0-P1】

### A*
- `f=g+h`。
- Open/Closed set。
- admissible heuristic。
- consistent heuristic（进阶）。
- 为什么 h 高估可能丢失最优性。

### TopK
- 小顶堆维护 TopK 门槛。
- 动态排行榜：平衡树 / skiplist / sorted set 思路。

### Timer
- 小量定时器：min heap。
- 大量高频：time wheel（加分）。

### Geometry
- 点乘判断前后/夹角。
- 叉乘判断左右/法线。
- 点在三角形内：叉积/重心坐标。
- 扇形攻击范围：distance squared + dot。

### Spatial Query
- Quadtree/Octree。
- spatial hash/grid。
- BVH。

---

# 第十二部分：常见游戏系统设计【P0-P1】

## 12.1 背包
- ItemConfig vs ItemInstance。
- Stack。
- Sort。
- 快速索引。
- List + Dictionary 是否组合。
- UI 虚拟列表。
- 增量刷新。

## 12.2 Buff
- BuffConfig。
- BuffInstance。
- BuffContainer。
- duration / period / stack。
- modifier。
- apply/tick/remove。
- source/target。
- dispel。

## 12.3 Attribute
- BaseValue / CurrentValue。
- modifier aggregation。
- MaxHealth 改变如何处理 CurrentHealth。
- 来源可追踪、可移除。

## 12.4 Quest
- QuestDefinition / Runtime State。
- 条件节点。
- 顺序/并行目标。
- Event-driven progress。
- Save/Load。
- 数据库与运行时实例的职责分离。

## 12.5 Red Dot
- path/tree/trie。
- 数据节点聚合。
- UI 仅订阅对应节点，不反向依赖业务模块。

## 12.6 UI Manager
- 层级。
- 窗口栈。
- 生命周期。
- 异步资源。
- 输入焦点。
- Modal。
- Back 行为。

## 12.7 Combat / Character Controller
- Input。
- Locomotion。
- Ability。
- Animation。
- Motion/RootMotion。
- Physics。
- Arbitration。
- Camera。

不要把所有职责塞进 PlayerController。

---

# 第十三部分：调试、工程与生产能力【P1，近年越来越重要】

## 13.1 Bug 排查方法

推荐回答顺序：
1. 稳定复现。
2. 缩小范围。
3. 看日志/调用栈/Profiler。
4. 对比正常与异常状态。
5. 二分注释/版本 bisect。
6. 构造最小复现。
7. 修复后写回归测试或监控。

## 13.2 Unity 工具
- Profiler。
- Memory Profiler。
- Frame Debugger。
- Deep Profile 的代价。
- Development Build。
- Android Logcat。
- RenderDoc（加分）。

## 13.3 Git
- merge / rebase。
- conflict。
- cherry-pick。
- revert / reset 区别。
- bisect（加分）。
- 大文件与 Git LFS。

## 13.4 跨平台
- Windows/Android/iOS 差异意识。
- 路径。
- 文件权限。
- 输入。
- 分辨率与 Safe Area。
- Graphics API。
- IL2CPP。
- native plugin。

---

# 第十四部分：HR / 行为面【P0】

## 14.1 自我介绍
必须控制在 1–2 分钟：
- 学历/方向。
- Unity/C# 技术栈。
- 1–2 个最强项目。
- 对岗位最相关的能力。
- 求职目标。

## 14.2 为什么游戏 / 为什么客户端
不要回答“因为喜欢玩游戏”就结束。
应包含：
- 对实时交互和复杂客户端系统感兴趣。
- 有持续的项目投入。
- 技术栈匹配。
- 未来希望深入的方向。

## 14.3 游戏理解
准备至少：
- 最近深度玩的 2–3 款游戏。
- 一款游戏从程序角度分析：战斗/UI/网络/性能/资源。
- 一个你觉得设计不好的系统，以及你会怎么改。

## 14.4 团队协作
- 如何和策划、美术、测试沟通。
- 需求不明确怎么办。
- 多个需求冲突怎么办。
- 发现排期风险怎么办。
- Code Review 意见冲突怎么办。

## 14.5 家庭信息 / 城市 / 稳定性
HR 可能通过家庭所在地、是否接受工作城市、家人意见等判断入职和长期稳定性。只回答与工作相关的信息即可，无需扩展到不必要的私人细节。

## 14.6 横评
技术面结束不等于 offer。横评通常是把同一 HC 的多名候选人从能力、项目、匹配度、到岗时间等维度综合比较。

---

# 第十五部分：2026 新增加分项——AI 辅助开发【P2，但建议准备】

部分 2026 游戏客户端面经已经出现 AI Coding/Agent 相关追问。

建议准备：
- 平时如何使用 Copilot/Codex/Claude/ChatGPT 等进行代码开发。
- 如何避免 AI 生成代码破坏架构约束。
- 大代码库上下文如何拆解：文档、模块边界、检索、分任务、局部上下文。
- 如何验证 AI 代码：编译、测试、静态分析、Code Review、Profiler。
- 哪些任务不应该完全交给 AI：关键架构、性能敏感路径、网络安全逻辑等。
- 多 Agent 是否真正必要，如何定义职责和共享约束。

回答核心：**AI 是工程效率工具，不是替代基本功。**

---

# 第十六部分：高频“追问链”清单

这些题不要只背第一问。

## Dictionary
Dictionary 原理 → 哈希冲突 → GetHashCode/Equals → key 为什么不能变 → 扩容 → 为什么 O(1) → 为什么小数据 List 可能更快 → ConcurrentDictionary。

## GC
GC 是什么 → Root → Mark/Sweep/Compact → 分代 → Unity Incremental GC → GC Alloc 来源 → 如何优化 → 有 GC 为什么还会泄漏。

## TCP
TCP vs UDP → 三次握手 → 可靠性 → ACK → RTO → Fast Retransmit → 滑动窗口 → 流量控制 → 拥塞控制 → 队头阻塞 → 为什么游戏用 UDP/KCP。

## UGUI
Canvas → batch → rebuild → 什么操作 dirty → 动静分离 → nested canvas → atlas → mask → overdraw → 长列表 → profiler 如何验证。

## DrawCall
是什么 → 为什么贵 → batch → static/dynamic → instancing → SRP Batcher → 为什么不能无脑合批 → CPU/GPU bottleneck 如何判断。

## Coroutine
是什么 → IEnumerator → 状态机 → PlayerLoop → yield 指令 → Coroutine vs Thread → Coroutine vs async → disable/destroy 生命周期。

## Quaternion
欧拉角问题 → gimbal lock → axis-angle → quaternion 四个数含义 → 乘法 → Slerp → 为什么不要直接编辑 x/y/z/w。

## Object Pool
为什么 → 生命周期 → Reset → 容量 → 内存换 CPU → 哪些对象不适合 → Addressables 资源池如何管理引用。

## AssetBundle/Addressables
加载 → 依赖 → handle/refcount → unload/release → 内存什么时候真正释放 → 场景切换 → 异步取消 → 重复加载。

## FSM
状态定义 → 切换 → Enter/Update/Exit → 状态爆炸 → HFSM → 并行状态 → arbitration → Ability/Locomotion 冲突。

---

# 第十七部分：你当前题库的“高风险错误点”

已有笔记中有些常见说法需要在正式面试前统一纠正：

1. **“值类型在栈、引用类型在堆”不准确。** 值类型存储位置取决于它所在的位置；作为 class 字段时它内嵌在堆对象中。
2. **`string.Empty` 和 `""` 不应解释成前者不分配、后者每次分配。** 空字符串通常是运行时共享的字符串实例，二者语义基本等价。
3. **不要把 Dictionary 的 bucket 定位死记成 `hash % bucketCount`。** 实际实现细节随 .NET 版本可能变化；面试回答到“hash 映射 bucket”即可，再讲 Entry 链/冲突。
4. **不要说 HTTP 一定基于 TCP。** HTTP/3 基于 QUIC/UDP。
5. **不要说 UDP 一定比 TCP 快。** 它只是机制更轻且应用能自行控制可靠性；真实延迟受网络和实现影响。
6. **不要说协程比 Update 快。** Unity Coroutine 本质仍由主线程 PlayerLoop 调度，优势主要是把跨帧逻辑写得更自然。
7. **不要把 DrawCall 优化和 Canvas Rebuild 优化混成一件事。** 前者关注提交/批次，后者关注 UGUI CPU 重建。
8. **不要把四元数简单说成“保存旋转轴和角度”。** 更准确是：单位四元数是一种旋转表示，可由 axis-angle 映射得到。
9. **不要把“有 GC 就不会泄漏”作为结论。** 托管对象仍可能因错误引用长期存活，Native/GPU 资源更需要显式生命周期管理。
10. **不要背已过时的 Unity 合批硬条件数字。** 版本/SRP 不同会变化，面试更应讲机制和限制。

---

# 第十八部分：30 天复习路线

## 第 1 周：语言 + 数据结构
- C# 内存/GC/装箱/委托/泛型/反射。
- Dictionary/List。
- C++ 虚函数/智能指针/vector/unordered_map。
- 每天 2 道算法。

## 第 2 周：Unity + UI + 资源
- 生命周期/协程/物理/动画。
- UGUI rebuild/batch/layout/eventsystem。
- AB/Addressables/资源生命周期。
- Profiler。

## 第 3 周：网络 + 图形学 + 性能
- TCP/UDP/HTTP/KCP/同步。
- 渲染管线/Depth/透明/DrawCall/PBR 基础。
- CPU/GPU/Memory 优化。

## 第 4 周：项目 + 模拟面试
- 每个项目写 10 个追问。
- 每天 30 分钟口述。
- 练 5 个场景设计题。
- HR/游戏分析。
- 根据目标公司补 C++、Lua、图形学或网络专项。

---

# 第十九部分：面试前最终 Checklist

### P0 题必须无卡顿
- C# 值/引用类型、装箱、GC、Dictionary、List、委托事件。
- Unity 生命周期、协程、对象池、资源加载、UGUI、DrawCall。
- TCP/UDP、可靠性、HTTP、粘包拆包、状态/帧同步。
- 渲染管线、深度、透明、四元数、点乘叉乘。
- 快排、哈希表、堆、TopK、链表、树、A*。
- 项目架构、难点、性能优化、Bug 排查。

### 简历上的每个名词必须回答三层
例如写了 Addressables：
1. 怎么用？
2. 为什么这么设计？
3. Handle/Release/依赖/内存问题怎么处理？

写了 GAS：
1. Ability/Effect/Tag/Attribute 分别做什么？
2. 生命周期和仲裁怎么做？
3. Infinite/Stack/Cancel/Locomotion 冲突怎么解决？

写了网络同步：
1. 状态/帧同步流程？
2. 延迟、丢包、重连怎么办？
3. 预测/插值/回滚/权威性怎么设计？

---

# 第二十部分：后续知识库维护方式

建议以后每次真实面试结束，都把题目按以下格式加入知识库：

```text
[公司 / 岗位 / 轮次]
题目：
我的回答：
面试官追问：
卡住的点：
正确知识链：
下次 90 秒答案：
项目中的例子：
优先级：P0/P1/P2
```

这样知识库会逐渐从“题目收藏夹”变成真正的“可输出能力库”。

