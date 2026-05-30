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
