# UnityGameFramework 学习记录

## 开始时间

我从 **2026-05-26 14:59:09（UTC+08:00）** 开始学习 UnityGameFramework 框架。

## 说明

这个仓库用于记录我学习 UnityGameFramework 的过程。

后续我会把学习心得、框架理解、源码分析、实践记录和遇到的问题持续更新到这里。

## 学习心得

### 2026-05-30

今天主要梳理了 UnityGameFramework 的底层架构和设计理念：

- 框架整体分为 `GameFramework` 核心层、`UnityGameFramework.Runtime` Unity 适配层和 `Editor` 编辑器工具层。
- `GameFramework` 核心层不直接依赖 `MonoBehaviour`，这样可以让底层逻辑脱离 Unity 生命周期限制，便于统一调度、测试、复用和维护。
- `MonoBehaviour` 主要集中在 Runtime 适配层，用于接入 Unity 生命周期、Inspector 配置、场景对象和引擎能力。
- 底层模块通过 `GameFrameworkEntry` 统一创建、更新和关闭，并通过模块优先级控制轮询顺序。
- 接口隔离让上层依赖 `IEventManager`、`IResourceManager` 等抽象能力，而不是直接依赖具体实现，降低耦合，方便替换实现、扩展 Helper、Mock 测试和维护模块边界。
- 阅读了 `GameFrameworkLinkedList<T>`，理解它是在 .NET `LinkedList<T>` 基础上增加节点缓存池，通过复用 `LinkedListNode<T>` 减少运行时 GC。
- `GameFrameworkLinkedList<T>` 的 `AcquireNode` 和 `ReleaseNode` 体现了框架中常见的对象池思想：移除节点后不立即丢弃，而是清空值并缓存，后续再次添加时复用。

当前理解：UnityGameFramework 的核心思想不是直接帮业务写玩法，而是先搭建一个模块化、低耦合、异步优先、可复用、重视性能和 GC 控制的游戏底层框架。

#### 补充记录：Helper 与整体运行流程

继续梳理了框架中大量 `Helper` 的作用和整体运行链路：

- `Helper` 本质上是框架预留的适配器和策略扩展点，用于承接项目差异、Unity 引擎细节、资源格式、协议格式和第三方库接入。
- 框架整体遵循“`Component` 接 Unity，`Manager` 管流程，`Interface` 定边界，`Helper` 管细节”的设计思路。
- `Manager` 负责稳定的通用流程，例如资源加载调度、UI 打开关闭、实体显示隐藏、网络连接管理等；具体如何实例化、解析、加载、播放、编码解码等细节交给 `Helper`。
- 这种设计可以避免核心 `Manager` 变得臃肿，减少框架核心对 Unity API、业务格式和第三方插件的直接依赖，方便后续替换资源系统、UI 系统、数据表格式、本地化格式或网络协议。
- 整体运行流程可以概括为：Unity 加载 `GameFramework` 对象，各个 `XXXComponent.Awake()` 注册到 Runtime 层 `GameEntry`，组件再通过 `GameFrameworkEntry.GetModule<T>()` 获取或创建核心 `Manager`，最后由 `BaseComponent.Update()` 每帧驱动所有核心模块更新。
- 游戏退出时，`GameEntry.Shutdown()` 会触发 `BaseComponent` 销毁，并由 `GameFrameworkEntry.Shutdown()` 反向关闭所有底层模块，清理引用池、缓存和日志辅助器。
- 通过 UI 打开流程进一步理解了框架协作方式：业务调用 `UIComponent`，`UIManager` 组织打开流程，`ResourceManager` 异步加载资源，`Helper` 处理 Unity 实例化细节，最后通过事件通知业务结果。

当前进一步理解：UnityGameFramework 的整体设计目标是把 Unity 游戏中常见的基础设施标准化、模块化和可替换化，让业务层专注玩法逻辑，而不是反复处理资源、对象生命周期、流程切换和模块耦合问题。
