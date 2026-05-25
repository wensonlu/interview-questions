# 前端 & AI Agent 面试知识题汇总

> 统计周期：2026-05-19 ~ 2026-05-25（第22波 ~ 第34波，共13波）
> 整理方式：按分类权重聚合，每题保留题目、考察知识点、答案三要素

---

## 📊 分类权重总览

| 权重 | 分类 | 题数 |
|------|------|------|
| 🔴 高频 | Vue3 响应式与组件原理 | 8 |
| 🔴 高频 | React 进阶特性与架构 | 7 |
| 🟠 中频 | 浏览器与网络协议 | 6 |
| 🟠 中频 | 前端工程化（构建/优化） | 5 |
| 🟡 核心 | AI Agent 架构（Multi-Agent/Memory） | 7 |
| 🟡 核心 | RAG 与向量检索 | 5 |
| 🟡 核心 | MCP 协议与 Tool Use | 6 |
| 🟡 核心 | 推理模式（ReAct/CoT） | 4 |

---

## 🔴 一、Vue3 响应式与组件原理

### 【第22波】Vue3 patchFlag 靶向更新

📌 **题目**：Vue3 的 patchFlag 机制是如何实现靶向更新的？

📌 **考察知识点**：Vue3 编译时优化、patchFlag 静态标记、靶向更新原理

✅ **答案**：Vue3 在编译时对动态节点打上 patchFlag 标记（如 `1`=TEXT、`8`=CLASS）。运行时通过位运算快速判断需要更新的节点类型，只比较标记过的动态部分，跳过静态节点。典型场景：`<div>{{msg}}</div>` 标记为 1，只更新文本节点而不触发整个 div 的重新渲染。

---

### 【第22波】Vue3 WeakMap 依赖追踪

📌 **题目**：Vue3 如何用 WeakMap 实现依赖追踪？和 Vue2 的 defineProperty 有何区别？

📌 **考察知识点**：响应式原理、WeakMap 弱引用、依赖收集

✅ **答案**：Vue3 每个 reactive 对象内部维护一个 WeakMap\<对象, Map\<属性, Set[依赖]>\>。访问属性时，Proxy handler 通过 Reflect.get 收集当前 effect 到 WeakMap 中，属性变更时从 WeakMap 中取出所有依赖 effect 触发更新。优势：对象无引用时自动被 GC，避免内存泄漏；可监听新增属性。Vue2 defineProperty 无法监听新增属性，需 $set。

---

### 【第28波】Vue3 ref vs reactive 对比

📌 **题目**：Vue3 中 ref 和 reactive 的区别是什么？何时该用哪个？

📌 **考察知识点**：Vue3 响应式、ref/reactive 使用场景

✅ **答案**：`ref()` 用于基本类型，包裹后返回带 `.value` 的响应式引用，模板自动解包；`reactive()` 用于对象/数组，返回 Proxy 直接代理。底层：ref 内部调用 `reactive()` 包装 `.value`。选择：基本类型/多个值→ref；对象结构→reactive；需要解构保持响应→`toRefs`。

---

### 【第30波】Vue3 defineModel 编译器原理

📌 **题目**：Vue3 defineModel 的编译器输出是什么？实现了什么双向绑定？

📌 **考察知识点**：Vue3 编译器、defineModel 原理、父子组件通信

✅ **答案**：defineModel 将父组件 v-model 属性转换为 prop + update 事件。编译后每个 defineModel 生成：`props: { modelValue: String }`、`emit('update:modelValue', value)`。本质：语法糖，让子组件直接 `defineModel()` 替换原有的 `props.modelValue` + `emit('update:modelValue')`，无需手动编写。

---

### 【第31波】Vue3 Teleport 组件渲染位置解耦

📌 **题目**：Vue3 Teleport 如何实现组件渲染位置解耦？适用场景是什么？

📌 **考察知识点**：Teleport 原理、动态渲染目标、Modal/Dialog 场景

✅ **答案**：Teleport 将组件树渲染到指定 DOM 节点（如 `document.body`），不受父组件 CSS overflow/transform 影响。语法：`<Teleport to="body">` 或 `<Teleport :to="targetEl">`。典型场景：Modal、Toast、通知。渲染时机：在组件 mount 阶段执行 teleport-to，不影响组件生命周期。

---

### 【第33波】Vue3 watchEffect vs watch 懒执行

📌 **题目**：Vue3 中 watch 和 watchEffect 的区别是什么？

📌 **考察知识点**：Vue3 响应式、watchEffect 自动收集、watch 手动指定

✅ **答案**：`watchEffect` 自动收集依赖，初始化时立即执行，属性变化自动重新运行。`watch` 惰性执行，需要手动指定监视目标，默认不做深度比较，配置 `{ immediate: true }` 可立即执行。场景：需要获取旧值→watch；只关心副作用执行→watchEffect。

---

### 【第34波】Vue3 defineModel 与 v-model 复用

📌 **题目**：Vue3 中同一个组件如何同时使用多个 v-model？

📌 **考察知识点**：Vue3 多 v-model、defineModel 组合、组件设计

✅ **答案**：Vue3 支持在组件上使用多个 v-model，每个绑定不同的 prop 名。子组件通过 `defineModel('propName')` 声明多个。编译后生成多个 prop + emit 对。如 `<UserForm v-model:name="name" v-model:age="age">`，子组件两次调用 defineModel 分别对应 name 和 age 属性。

---

### 【第21波】Vue3 WeakMap 依赖追踪（补充）

📌 **题目**：Vue3 的响应式系统中，副作用收集的触发时机是什么？

📌 **考察知识点**：响应式副作用、依赖收集时机、Proxy handler

✅ **答案**：在 Proxy getter 中触发依赖收集。当前活跃的 activeEffect 会被收集到对应属性的 Dep Set 中。依赖追踪基于"谁在读谁被收集"原则。关键：只有同步代码中的 reactive 访问会被收集，async/await 后的代码需重新触发。

---

## 🔴 二、React 进阶特性与架构

### 【第22波】React Batch 机制（18全局批处理）

📌 **题目**：React 18 的全局批处理（Automatic Batching）解决了什么问题？

📌 **考察知识点**：React 18 批处理、状态更新合并、性能优化

✅ **答案**：React 18 之前只有 React 事件处理中的 setState 会批处理，Promise/setTimeout/原生事件中的 setState 会多次触发。React 18 开始，所有状态更新（包括 Promise、setTimeout、原生事件）自动合并为一次渲染。好处：减少不必要的渲染次数，提升性能。根本实现：将所有回调中的状态更新统一在协调器层面批处理。

---

### 【第31波】React useTransition vs useDeferredValue

📌 **题目**：React 的 useTransition 和 useDeferredValue 有什么区别？各自适用场景？

📌 **考察知识点**：并发模式、useTransition、useDeferredValue、时间切片

✅ **答案**：`useTransition` 将某个状态更新标记为"非紧急"，期间可以响应其他交互，组件返回 `isPending` 状态。`useDeferredValue` 让某个值被"延迟"，仅在紧急渲染完成后更新。场景：useTransition→搜索输入的场景；useDeferredValue→列表过滤、组件延迟展示。

---

### 【第33波】React19 use() Hook

📌 **题目**：React 19 的 use() Hook 解决了什么问题？和 useEffect 有何区别？

📌 **考察知识点**：React 19 新特性、use()、Promise/Context 读取

✅ **答案**：`use()` 可在组件内读取 Promise/AsyncIterator/Context，像同步代码一样。优势：替代 useEffect + 依赖管理，直接处理异步数据加载；读取 Promise 时会 Suspense 等待，读取 Context 支持跨组件传递。和 useEffect 比：use 是同步读取，useEffect 是声明式副作用。

---

### 【第30波】React useSyncExternalStore（并发模式）

📌 **题目**：React 的 useSyncExternalStore 是什么？为什么要用它？

📌 **考察知识点**：useSyncExternalStore、并发模式、外部状态订阅

✅ **答案**：React 18 提供的 Hook，用于订阅外部状态源（如 Redux、第三方状态库）。签名：`useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`。保证组件在并发模式下读取一致的状态快照，避免 torn read。场景：连接外部状态管理时必须使用，确保状态读取的线程安全。

---

### 【第22波】React Hooks 依赖数组

📌 **题目**：useEffect/useMemo/useCallback 的依赖数组如何影响性能？

📌 **考察知识点**：Hooks 依赖、闭包陷阱、性能优化

✅ **答案**：依赖数组决定何时重新执行/创建。漏写→闭包 stale 数据；过多→频繁执行/创建。优化：useMemo 缓存计算结果，useCallback 缓存回调函数传给子组件。原则：稳定引用放依赖，不稳定引用用 ref 或重新创建。

---

### 【第21波】React Fiber 架构

📌 **题目**：React Fiber 架构的核心目标是什么？它如何实现可中断渲染？

📌 **考察知识点**：Fiber 架构、Reconciliation、可中断/恢复

✅ **答案**：核心目标：将渲染工作拆成小单元，支持可中断、可恢复、优先级排序。每个 Fiber 节点代表一个工作单元，包含 child/sibling/return 指针连接成链表。工作时：render 阶段（可中断）→ commit 阶段（不可中断）。优先级：交互事件 > 批量更新 > 低优先级更新。Fiber 的 `expirationTime` 决定调度顺序。

---

### 【第30波】Virtual DOM 性能瓶颈

📌 **题目**：Virtual DOM 的性能瓶颈在哪里？React 如何优化？

📌 **考察知识点**：Virtual DOM diff、性能优化、React Compiler

✅ **答案**：瓶颈：虚拟 DOM diff 是全量比较，大结构树更新成本高；频繁 setState 触发多余渲染。优化：React 18 的 `automatic batching` 减少渲染次数；`useTransition/useDeferredValue` 标记非紧急更新；`compiler` 将 JSX 编译为直接操作 DOM 的代码绕过虚拟 DOM；`useMemo/useCallback` 避免子组件不必要的重新渲染。

---

## 🟠 三、浏览器与网络协议

### 【第22波】HTTP/2 多路复用 vs HTTP/1.1 队头阻塞

📌 **题目**：HTTP/2 的多路复用是如何解决队头阻塞问题的？

📌 **考察知识点**：HTTP/2 多路复用、HTTP/1.1 队头阻塞、Stream/Frame

✅ **答案**：HTTP/1.1 队头阻塞：同一TCP连接上只有第一个请求响应完成后才能处理第二个。HTTP/2 多路复用：在同一 TCP 连接上，用 Stream ID 区分多个并行请求/响应，每个请求/响应拆分成 Frame 交错发送，接收端根据 Stream ID 组装。彻底解决队头阻塞，但受 TCP 层丢包阻塞影响（QUIC 解决）。

---

### 【第29波】HTTP/3 vs HTTP/2（QUIC 0-RTT、无队头阻塞）

📌 **题目**：HTTP/3 基于 QUIC 协议，相比 HTTP/2 有哪些核心优势？

📌 **考察知识点**：QUIC 协议、HTTP/3、0-RTT、连接迁移

✅ **答案**：1. **0-RTT**：首次连接后缓存会话凭证，后续连接直接发送数据，无需握手。2. **无 TCP 队头阻塞**：QUIC 基于 UDP，单 Stream 丢包只影响该 Stream，不影响其他。3. **连接迁移**：网络切换时用 Connection ID 维持连接，无需重建。4. **更好的拥塞控制**：QUIC 自己实现，在应用层控制。

---

### 【第28波】Core Web Vitals（LCP/INP/CLS）

📌 **题目**：Core Web Vitals 2024 的三大指标是什么？各自优化方向？

📌 **考察知识点**：Web Vitals、LCP/INP/CLS、用户体验指标

✅ **答案**：**LCP**（最大内容绘制）：目标<2.5s，优化：减少服务器响应时间、启用缓存、优化图片优先级。**INP**（交互到绘制）：目标<200ms，优化：减少长任务、减少主线程阻塞、Web Worker 分担计算。**CLS**（累积布局偏移）：目标<0.1，优化：给图片/广告预留空间、避免动态插入内容。

---

### 【第33波】HTTP 缓存 ETag vs Last-Modified

📌 **题目**：ETag 和 Last-Modified 在 HTTP 缓存中的区别是什么？

📌 **考察知识点**：HTTP 缓存、ETag、Last-Modified、协商缓存

✅ **答案**：`Last-Modified`：服务器返回资源最后修改时间，客户端请求带上 `If-Modified-Since`，服务器比较时间决定返回 304 还是新资源。`ETag`：服务器返回资源的内容指纹（hash），客户端请求带上 `If-None-Match`，服务器比较指纹。ETag 更精确（基于内容而非时间），可避免 VCS 导致的假命中。

---

### 【第22波】V8 垃圾回收与内存泄漏

📌 **题目**：V8 引擎的垃圾回收机制是什么？常见内存泄漏场景有哪些？

📌 **考察知识点**：V8 GC、内存管理、内存泄漏

✅ **答案**：V8 使用分代 GC：新生代（Scavenge，适合短生命周期对象）、老生代（Mark-Sweep/Mark-Compact，适合长生命周期对象）。内存泄漏场景：全局变量（未清理的 setInterval、闭包引用）、DOM 引用（已删除 DOM 还在 JS 中引用）、监听器未清理、Map/Set 持续添加对象。

---

### 【第31波】HTTP/2 Stream 优先级

📌 **题目**：HTTP/2 中 Stream 优先级是如何实现的？

📌 **考察知识点**：HTTP/2 Stream、优先级依赖、WINDOW_UPDATE

✅ **答案**：HTTP/2 用 Weight（权重）+ Dependency（依赖）实现优先级树。父 Stream 关闭时子 Stream 提升。客户端可发送 `PRIORITY` 帧动态调整优先级。WINDOW_UPDATE 控制每个 Stream 的流量。优先级高的 Stream 优先获得带宽，但并非严格时序保证。

---

## 🟠 四、前端工程化（构建/优化）

### 【第29波】Tree Shaking 原理与 sideEffects

📌 **题目**：Webpack/Vite 的 Tree Shaking 原理是什么？sideEffects 配置的作用？

📌 **考察知识点**：Tree Shaking、sideEffects、ESM 静态分析

✅ **答案**：Tree Shaking 基于 ES Module 静态分析（import/export 在顶层，无条件执行），标记未使用的 exports 生成 `usedExports: true`， terser 删除未引用代码。`sideEffects: false` 告知 webpack 该模块若无 export 使用则可直接删除；`sideEffects: ['*.css']` 表示这些文件有副作用不能删除。关键：需开启 `usedExports` + `sideEffects` 一起工作。

---

### 【第30波】Webpack Tree Shaking 副作用标记

📌 **题目**：如何正确标记模块的副作用以优化 Tree Shaking？

📌 **考察知识点**：sideEffects 配置、Tree Shaking、模块级别副作用

✅ **答案**：在 `package.json` 中配置 `sideEffects: false`（所有模块默认无副作用）或数组（指定有副作用的文件）。在源码中用 `/*#__PURE__*/` 注释标记无副作用的函数调用（如 `new Something()/*#__PURE__*/`），帮助 terser 判断可安全删除。CSS 文件需配置 `sideEffects: ['*.css']` 避免被误删。

---

### 【第31波】Tree Shaking 为何只能针对 ESM

📌 **题目**：Tree Shaking 为何只能针对 ESM 模块？CommonJS 为何不支持？

📌 **考察知识点**：ESM vs CommonJS、Tree Shaking 限制

✅ **答案**：ESM 是静态结构（import 在顶层，不可动态 require），编译时即可确定依赖图。CommonJS 是动态结构（require 可在函数内、条件分支），运行时才能确定依赖，无法静态分析。动态 require 使 webpack 无法判断模块是否被使用，因此 CommonJS 模块不能 Tree Shaking。

---

### 【第28波】Module Federation 原理

📌 **题目**：Webpack Module Federation 的核心原理是什么？适用场景？

📌 **考察知识点**：Module Federation、远程模块、共享依赖

✅ **答案**：Federation 将应用拆分为 host 和 remote，host 可动态加载 remote 的模块。原理：在 build 时生成 `remoteEntry.js`，host 通过 `__webpack_require__.l` 异步加载远程模块。共享：配置 `shared` 字段指定共享的第三方库（如 React、Vue），避免重复打包。场景：微前端、多团队协作。

---

### 【第28波】LCP 优化策略

📌 **题目**：如何优化 LCP（Largest Contentful Paint）？

📌 **考察知识点**：LCP 优化、Web Vitals、性能优化

✅ **答案**：1. **移除渲染阻塞资源**：CSS/字体用 `preload`，JS 用 `defer`。2. **优化服务器响应**：CDN、缓存、TTFB<200ms。3. **优化资源大小**：图片压缩、WebP/AVIF、响应式图片 `srcset`。4. **优化渲染阻塞**：内联关键 CSS、延迟非关键资源。5. **预加载 LCP 元素**：`<link rel="preload">` 提前加载首屏大图。

---

## 🟡 五、AI Agent 架构（Multi-Agent/Memory）

### 【第22波】Agent 短长期记忆实现方案

📌 **题目**：AI Agent 的短期记忆和长期记忆如何实现？各自特点？

📌 **考察知识点**：Agent Memory、短期记忆、长期记忆、RAG

✅ **答案**：**短期记忆**：基于上下文窗口（如 128K tokens），维护会话历史，容量有限，适合当前任务上下文。**长期记忆**：外部存储 + RAG 检索，将历史经验向量化为向量库，检索最相关的片段注入上下文。实现方案：用 FAISS/Milvus 向量库存储，用 `summary + vector` 双轨管理，定期清理低价值记忆。

---

### 【第22波】Multi-Agent 任务分发汇聚模式

📌 **题目**：Multi-Agent 系统中，任务分发和结果汇聚的典型模式有哪些？

📌 **考察知识点**：Multi-Agent、任务分发、结果汇聚、Supervisor

✅ **答案**：1. **树形分发**：Supervisor 分解任务，分发给多个子 Agent，最后汇聚结果。2. **广播并行**：任务同时发给多个 Agent，结果汇总后做决策（如投票）。3. **流水线**：A→B→C 串行执行，每步输出作为下步输入。4. **层次化**：高层 Agent 负责任务规划，低层 Agent 负责任务执行。

---

### 【第22波】多 Agent 通信协议设计

📌 **题目**：Multi-Agent 系统中如何设计 Agent 间的通信协议？

📌 **考察知识点**：Multi-Agent 通信、协议设计、消息格式

✅ **答案**：协议设计三要素：消息格式（JSON/结构化）、传输层（HTTP/WebSocket/gRPC）、语义层（意图/内容分离）。常见模式：1. **发布-订阅**：Agent 订阅特定主题消息。2. **请求-响应**：同步调用获取结果。3. **消息队列**：用 Kafka/RabbitMQ 解耦。关键设计：消息唯一ID防止重复处理、超时机制、幂等性保证。

---

### 【第21波】多 Agent 消息风暴防护

📌 **题目**：Multi-Agent 系统中如何防止消息风暴和循环调用？

📌 **考察知识点**：Agent 循环防护、消息风暴、幂等设计

✅ **答案**：1. **消息去重**：消息 ID + 已处理集合，丢弃重复消息。2. **调用链路限制**：设置最大深度（如 5 层），超深停止。3. **超时机制**：单次交互超过 N 秒则超时放弃。4. **环检测**：追踪消息链路，发现闭环立即中断。5. **幂等设计**：同一消息多次处理结果一致。

---

### 【第31波】Supervisor vs Hierarchical 多 Agent 架构

📌 **题目**：Supervisor vs Hierarchical 多 Agent 架构的区别是什么？

📌 **考察知识点**：Multi-Agent 架构、Supervisor、Hierarchical

✅ **答案**：**Supervisor 模式**：一个中心 Agent 负责任务分发和结果汇总，下属 Agent 是叶子节点，适合任务明确、可分解的场景。**Hierarchical 模式**：多层 Agent 形成树形结构，上层 Agent 可再拆分任务给下层，适合复杂、层级分明的任务。Supervisor 是两层，Hierarchical 是多层。

---

### 【第29波】Multi-Agent 并行度与通信成本

📌 **题目**：多 Agent 架构中，如何平衡并行度和通信成本？

📌 **考察知识点**：Multi-Agent 并行度、Agent 数量、通信成本

✅ **答案**：并行度↑ → Agent 数↑ → 通信成本↑，需要权衡。策略：1. **任务类型分流**：CPU 型（分析）可并行，I/O 型（搜索）并发高。2. **动态调度**：先并行探测，结果收敛后串行聚合。3. **Agent 池化**：复用 Agent 而非每任务新创建。4. **批量通信**：多个小消息合并为一批，减少网络往返。

---

### 【第22波】Tool Use 参数幻觉缓解策略

📌 **题目**：如何缓解 Function Calling / Tool Use 中的参数幻觉问题？

📌 **考察知识点**：Function Calling、Tool Use、幻觉缓解、JSON Schema

✅ **答案**：1. **严格 JSON Schema**：定义 precise 类型的参数，避免 ambiguous 类型。2. **Few-shot 示例**：提供多个正确调用的示例，让模型学习。3. **输出验证层**：LLM 输出后加验证器（如 JSON.parse + schema validate），不合法则重试。4. **纠错 Prompt**：在 system prompt 中加入"若不确定参数则不调用"指令。5. **降低温度**：降低随机性参数取值。

---

## 🟡 六、RAG 与向量检索

### 【第22波】RAG 向量检索 HNSW vs IVF-PQ 对比

📌 **题目**：向量数据库中 HNSW 和 IVF-PQ 索引的区别是什么？各自适用场景？

📌 **考察知识点**：向量索引、HNSW、IVF-PQ、相似度检索

✅ **答案**：**HNSW**（分层可导航小世界图）：用多层图实现近邻快速搜索，上层粗筛下层精找。优势：QPS 高、适合在线召回，内存占用高。**IVF-PQ**（倒排索引+乘积量化）：先聚类构建倒排，搜索时只在最近的几个聚类中找，PQ 压缩向量。优势：内存占用低、适合大数据量，QPS 相对低。选型：在线服务→HNSW；离线大数据→IVF-PQ。

---

### 【第22波】向量数据库 vs 普通数据库

📌 **题目**：向量数据库和普通数据库的核心区别是什么？为什么需要专门向量库？

📌 **考察知识点**：向量数据库、向量检索、索引算法

✅ **答案**：普通数据库索引（B-tree/Hash）适合等值查询，无法高效做向量相似度搜索。向量数据库专为高维向量设计，使用 HNSW/IVF-PQ 等索引实现 O(logN) 或 O(1) 级别的相似度检索。核心能力：余弦相似度/欧氏距离计算、Top-K 高效召回、向量聚类、过滤条件组合。典型：Milvus/Pinecone/Chroma。

---

### 【第31波】Hybrid Search（稀疏+稠密向量融合）

📌 **题目**：Hybrid Search（BM25 + Embedding）融合检索的原理是什么？

📌 **考察知识点**：Hybrid Search、BM25、稠密向量、稀疏检索

✅ **答案**：Hybrid Search 同时使用稀疏检索（BM25/TF-IDF）和稠密检索（Embedding 向量）。流程：query 同时生成 sparse vector（关键词）和 dense vector（语义），两个通道并行检索，各自得到 Top-K 结果，然后用 RRFR（倒数排名融合）或加权方式合并两组排名，最终得到综合最优结果。解决：单纯关键词检索语义不匹配，单纯向量检索专有名词召回差。

---

### 【第31波】Embedding 模型选型维度

📌 **题目**：RAG 中如何选择 Embedding 模型？有哪些关键维度？

📌 **考察知识点**：Embedding 模型选型、向量维度、语义能力

✅ **答案**：选型维度：1. **向量维度**：维度越高表达能力越强，但存储/检索成本高（常见 768/1024/1536）。2. **语义能力**：在 MTEB 等基准测试上的表现。3. **上下文窗口**：是否支持长文本。4. **成本**：API 调用费用。5. **部署方式**：开源（BGE/Multilingual-E5）vs 闭源（OpenAI/Cohere）。中文场景推荐：bge-m3、text-embedding-3-large。

---

### 【第30波】Rerank 重排序解决语义不匹配

📌 **题目**：RAG 流程中 Rerank 的作用是什么？两阶段检索（召回+排序）如何配合？

📌 **考察知识点**：Rerank、两阶段检索、Cross-Encoder

✅ **答案**：两阶段检索：第一阶段用向量检索（ANNS）快速从百万级候选中召回 Top-100，第二阶段用 Cross-Encoder 重新排序得到 Top-10。Cross-Encoder 将 query 和 doc 同时输入模型输出相似度分数，比向量检索的近似计算更精准。Rerank 解决：向量检索的"近似不等于相关"问题，尤其是关键词相关但语义不够匹配的情况。

---

## 🟡 七、MCP 协议与 Tool Use

### 【第22波】MCP Token 认证流程

📌 **题目**：MCP（Model Context Protocol）协议中的 Token 认证流程是什么？

📌 **考察知识点**：MCP 协议、Token 认证、安全机制

✅ **答案**：MCP 认证流程：1. Client 向 Server 发起连接请求。2. Server 返回 auth challenge。3. Client 用预设的 Token 响应 challenge。4. Server 验证 Token 有效性。5. 验证通过后建立持久会话，后续请求带上 Token。MCP 支持 Bearer Token 和 API Key 两种方式，可配置权限范围控制工具访问。

---

### 【第22波】MCP 协议核心原理

📌 **题目**：MCP（Model Context Protocol）的核心设计目标是什么？解决了什么问题？

📌 **考察知识点**：MCP 协议、Anthropic、工具标准化

✅ **答案**：MCP 是 Anthropic 提出的 AI 工具调用标准协议，解决 AI 工具生态碎片化问题。核心设计目标：让 AI 客户端用统一的方式调用各种工具和访问资源，无需为每个工具写独立适配器。MCP 通过 JSON-RPC over stdio/SSE 实现，定义标准化的 Resource/Tool/Prompt 接口，工具开发者实现 MCP Server 即可被任何 MCP 兼容客户端使用。

---

### 【第30波】MCP Resource vs Tool 本质区别

📌 **题目**：MCP 协议中 Resource 和 Tool 的本质区别是什么？

📌 **考察知识点**：MCP Resource、MCP Tool、接口设计

✅ **答案**：**Tool**：AI 主动调用执行操作（如搜索、发送消息），有副作用，会改变系统状态。**Resource**：AI 读取数据但不执行操作（如查询文档、获取配置），无副作用。Tool 用 `tools/call` 调用，返回执行结果；Resource 用 `resources/read` 读取，返回资源内容。设计原则：读取信息→Resource，执行操作→Tool。

---

### 【第31波】MCP Sampling 机制与 AI 主动感知

📌 **题目**：MCP 的 Sampling 机制是什么？它如何实现 AI 的主动感知能力？

📌 **考察知识点**：MCP Sampling、主动感知、回调机制

✅ **答案**：Sampling 允许 MCP Server 向 Client 发起采样请求，让 AI 生成内容（如生成图片、补充文本）。实现：Server 发 `sampling/createMessage` 请求，Client 的 LLM 生成响应并返回。典型场景：AI 生成中间步骤的解释文本、生成代码、生成图像。Sampling 让 Agent 从被动响应变为主动生成，拓展了工具调用边界。

---

### 【第31波】Tool Use 工具 description 设计最佳实践

📌 **题目**：Tool Use 中工具的 description 如何设计才能提升调用准确率？

📌 **考察知识点**：Tool Use、description 设计、Prompt 工程

✅ **答案**：description 设计原则：1. **清晰描述功能**：一句话说明工具做什么。2. **明确输入参数**：每个参数用 JSON Schema 定义类型和约束。3. **提供使用示例**：description 中加入常见调用示例。4. **标注边界**：说明不适用的场景（如"不支持的文件类型"）。5. **区分相似工具**：多个相似工具间明确差异化描述，帮助模型选对。

---

### 【第30波】MCP 协议与 Function Calling 区别

📌 **题目**：MCP 协议和 Function Calling / Tool Use 的关系是什么？是否互斥？

📌 **考察知识点**：MCP vs Function Calling、协议层对比

✅ **答案**：MCP 是更广义的协议，Function Calling 是具体实现方式。MCP 不仅包含工具调用（Tool），还定义了资源访问、采样、提示等能力。Function Calling 是 LLM 原生的工具调用机制，通过 prompt 引导模型输出符合 schema 的调用。关系：MCP Server 可通过 Function Calling 输出调用指令，两者协同工作而非互斥。

---

## 🟡 八、推理模式（ReAct/CoT）

### 【第22波】ReAct vs CoT vs Acting 推理模式对比

📌 **题目**：ReAct、Chain-of-Thought、Acting 模式的区别是什么？各自适用场景？

📌 **考察知识点**：ReAct、CoT、Acting、推理模式

✅ **答案**：**CoT（Chain-of-Thought）**：逐步推理展示思考链，无行动，纯逻辑推导。适用：数学/逻辑问题。**ReAct（Reasoning + Acting）**：推理→行动→观察→继续推理，闭环反馈。适用：复杂多步任务、需要外部工具的场景。**Acting**：模型扮演特定角色执行操作，关注行为模拟而非推理。ReAct 融合推理和行动，CoT 专注推理。

---

### 【第31波】ReAct 终止条件与死循环避免

📌 **题目**：ReAct 推理模式的终止条件是什么？如何避免死循环？

📌 **考察知识点**：ReAct 终止条件、死循环防护、最大步数限制

✅ **答案**：终止条件：1. 达到最大步数限制（如 20 步）。2. 产生最终答案（confidence > threshold）。3. 连续 N 步无状态变化（重复检测）。4. 外部终止信号（用户取消）。死循环防护：维护 `visited_states` 集合，新状态已在集合中则终止；每步记录 reasoning trace，方便回溯审计。

---

### 【第29波】Tree-of-Thought 与 ReAct 对比

📌 **题目**：Tree-of-Thought（ToT）和 ReAct 的区别是什么？ToT 的优势在哪里？

📌 **考察知识点**：ToT、ReAct、CoT、推理范式对比

✅ **答案**：**CoT**：线性推理链，一步接一步。**ToT**：树形探索，每步可分支多个推理路径，用 BFS/DFS 搜索最优路径。**ReAct**：推理和行动交替，闭环反馈。ToT 优势：在需要"探索-回溯"的复杂问题（如规划、创意生成）上比 ReAct 更好，可系统性探索多条路径再选最优。

---

### 【第31波】CoT 推理的优缺点

📌 **题目**：CoT（Chain-of-Thought）推理的优缺点是什么？什么场景不适合用？

📌 **考察知识点**：CoT 局限性、推理开销、适用场景

✅ **答案**：CoT 优点：提升复杂推理任务准确率、可解释性强。缺点：增加 token 消耗（推理链长→成本高）、增加延迟（推理时间）。不适用的场景：简单查询（不需要推理）、实时性要求高（延迟敏感）、资源受限（token 预算有限）。选择原则：需要推理→CoT/ToT；简单等值查询→直接回答。

---

## 📌 附录：波次对照表

| 波次 | 日期 (UTC) | 前端篇主题 | AI Agent篇主题 |
|------|-----------|-----------|---------------|
| 第22波 | 2026-05-22 01:40 | React Batch/Vue3 patchFlag/Web Vitals/V8 GC/PWA | MCP Token/向量库/Tool Use/多Agent/Memory |
| 第26波 | 2026-05-22 07:24 | CSS选择器/Vue3 ref-reactive/Event Loop/CSS Grid/LCP | MCP/ReAct/Multi-Agent/Memory/HNSW vs IVF |
| 第28波 | 2026-05-22 08:06 | CSS选择器/Vue3 ref-reactive/Event Loop/CSS Grid/LCP | MCP/ReAct/Multi-Agent/Memory/HNSW vs IVF |
| 第29波 | 2026-05-22 12:03 | watch vs watchEffect/宏微任务/HTTP3/Core Web Vitals/Tree Shaking | MCP+SSE/BM25+Embedding/ReAct vs CoT vs ToT/多Agent架构/Embedding选型 |
| 第30波 | 2026-05-23 18:07 | Vue3 defineModel/React useSyncExternalStore/Virtual DOM/Webpack tree shaking/React事件委托 | MCP Resource vs Tool/HDE/Rerank/ReAct终止条件/Function Calling hallucination |
| 第31波 | 2026-05-23 20:05 | Vue3 Teleport/useTransition vs useDeferredValue/requestIdleCallback/HTTP2多路复用/Tree Shaking | MCP Sampling/Hybrid Search/ReAct vs CoT/Multi-Agent通信/Tool description设计 |
| 第33波 | 2026-05-24 14:01 | watchEffect vs watch/React19 use()/回流重绘优化/Tree Shaking原理/HTTP缓存ETag | MCP/Hybrid Search混合检索/ReAct vs CoT/Multi-Agent任务分配/Function Calling参数校验 |
| 第34波 | 2026-05-25 02:05 | Vue3/React/性能优化/工程化/浏览器原理（综合） | MCP/RAG/Tool Use/ReAct/Multi-Agent（综合） |

---

*文档生成时间：2026-05-25 10:59 GMT+8*
*来源：OpenClaw 自动整理自最近一周面试题推送记录*