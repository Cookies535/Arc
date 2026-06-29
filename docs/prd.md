## Problem Statement

**项目概述 (Project Overview)**
Arc 是一款面向 Windows 和 Android 平台的跨平台个人记忆管理应用。当前市面上的相册和日记应用大多依赖云端服务（如 Google Photos、iCloud），且在浏览体验上局限于传统的 2D 瀑布流网格。对于注重隐私、希望完全掌控自己数据，并追求极致沉浸式交互体验的用户而言，缺少一个纯本地的、无需账号的、且具有情感联结的记忆管理工具。

**目标用户与使用场景 (Target Users & Scenarios)**
- **目标用户**：极客、摄影爱好者、日记习惯保持者、注重隐私的数字游民。
- **使用场景**：
  - 用户导入单反相机或手机拍摄的照片，应用自动在后台提取时间、地点和主色调。
  - 用户手动将特定照片组织成“节点”（如“大学毕业”、“2026年冰岛旅行”），并在 3D 星图中浏览自己的人生轨迹。
  - 跨设备无缝迁移数据，无需通过第三方服务器。

## Solution

**解决方案 (The Solution)**
Arc 提供一个完全离线的 3D 星图主界面（Starfield View），将用户的记忆抽象为漂浮在深色粒子宇宙中的“节点 (Node)”。应用通过纯算法在本地完成图像处理，利用 Isar 数据库进行极速关联，采用 Flame 引擎实现流畅的 2.5D 视差与镜头推进效果，并提供基于 GitHub Releases 的静默热更新机制。

**核心功能边界 (Feature Phases)**
- **Phase 1: MVP (核心星图与本地存储)**
  - 基础 Isar 数据库搭建（Node 与 Photo 的多对多存储）。
  - 本地照片导入，Isolate 后台队列处理缩略图生成与 K-Means 主色调提取。
  - 基于 Flame 引擎的 Starfield View 原型验证（浮动平台、节点渲染、镜头推进）。如果性能或包体积超标，则 fallback 至 Flutter Canvas 2.5D 投影。
  - 基础 UI Overlay（节点创建、详情查看）。
- **Phase 2: 关联与交互 (可用版)**
  - Node 之间的时间顺序自动串联轨道与手动主题连接。
  - 远距离节点合并为星团 (Cluster) 及视口时间区块 (Chunk) 按需加载。
  - 视觉特效完善（深色沉浸、粒子效果、液态玻璃质感 UI）。
  - GitHub Releases 自动静默更新机制。
- **Phase 3: 跨端与检索 (完整版)**
  - Windows Portable ZIP 与 Android APK 的静默更新闭环。
  - 导出/导入 `.arc` 数据包（包含数据库与缩略图），支持跨设备数据漫游与死链重新定位。
  - 节点标题与描述的 Isar 全文搜索。

## User Stories

1. As a user, I want to import photos from my local storage, so that I can add them to my memory nodes.
2. As a user, I want the app to automatically extract EXIF time and GPS data from imported photos, so that I don't have to enter it manually.
3. As a user, I want the app to extract the dominant color of my photos using K-Means, so that the UI can adapt its theme to my memories.
4. As a user, I want the app to process photos in the background (Processing Queue), so that the main interface remains smooth during mass imports.
5. As a user, I want to create a "Node" with a date, title, description, and type, so that I can organize my memories contextually.
6. As a user, I want to link multiple photos to a single Node, and link one photo to multiple Nodes, so that my memories can intersect naturally.
7. As a user, I want to view my Nodes in a 3D Starfield environment, so that exploring my memories feels immersive and spatial.
8. As a user, I want to see Nodes automatically connected by chronological tracks, so that I can trace my life's timeline.
9. As a user, I want to manually create custom connections between Nodes, so that I can group related events across different times.
10. As a user, I want to navigate the Starfield with camera zooming and panning, so that I can explore both the macro timeline and micro details.
11. As a user, I want distant Nodes to cluster together, so that the Starfield remains visually clean and performs well.
12. As a user, I want missing-GPS Nodes to be placed in an "Unknown Starfield", so that no memory is lost from the visualization.
13. As a user, I want to export my data as an `.arc` archive, so that I can back up my memories or move them to another device.
14. As a user, I want to import an `.arc` archive on a new device and still see thumbnails even if original photos are missing, so that my Starfield remains intact.
15. As a user, I want to re-link missing original photos manually, so that I can restore full-resolution viewing after moving files.
16. As a user, I want the app to silently download updates in the background, so that I don't have to manage manual version upgrades.
17. As a user, I want to see a glowing glassmorphic button when an update is ready, so that I can restart and apply it at my convenience.
18. As a user, I want all my data to remain strictly on-device, so that my privacy is completely guaranteed.
19. As a user, I want AI suggestions to be marked with a special symbol and kept separate from my manual entries, so that I always retain control over my data.

## Implementation Decisions

**技术架构 (Technical Architecture)**
- **UI & App Framework**: Flutter (Windows & Android 双端).
- **3D 渲染引擎**: Flame Engine (负责核心的 Starfield View，渲染浮动平台、粒子系统、视差背景)。如果原型验证包体积增加 > 20MB 或兼容性不佳，Fallback 至纯 Flutter Canvas 2.5D 投影。
- **UI 分层**: 所有的设置、画廊、详情弹窗均作为纯 Flutter Widget 覆盖 (UI Overlay) 在 Flame 画布之上。
- **本地数据库**: Isar Database (强类型，支持 IsarLinks 多对多关系与极速 Chunk 查询)。
- **后台计算**: Dart Isolate 队列处理 K-Means 色调提取与 EXIF 解析。
- **更新机制**: 依赖 GitHub API 轮询 Releases。后台静默下载全量包，通过 `updater.exe` (Windows Portable) 或系统 Intent (Android) 完成热更新。

**数据模型 (Isar Schema Appendix)**
```dart
// Nodes Collection
@collection
class Node {
  Id id = Isar.autoIncrement;
  @Index() DateTime date;
  @Index(type: IndexType.value) String title;
  String description;
  @Enumerated(EnumType.name) NodeType type;
  double? latitude;
  double? longitude;
  DateTime createdAt;

  // Shadow Data fields (Reserved for future AI)
  String? aiDescription;
  List<String>? aiSuggestedTags;

  // Many-to-Many
  @Backlink(to: 'nodes')
  final photos = IsarLinks<Photo>();
}

// Photos Collection
@collection
class Photo {
  Id id = Isar.autoIncrement;
  String filePath; // Original URI
  String thumbPath; // Internal Sandbox Thumbnail
  @Index() DateTime? takenAt;
  double? latitude;
  double? longitude;
  int? dominantColor;
  @Index(unique: true) String hash; // Deduplication
  int width;
  int height;

  final nodes = IsarLinks<Node>();
}
```

**视觉与交互规范 (Visual & Interaction)**
- **整体风格**: 类似 Mineradio 的深色沉浸感 + 深空粒子特效；类似《星之卡比》选关界面的 3D 浮动平台。
- **UI 质感**: 液态玻璃 (Glassmorphism)，包含半透明模糊与微光边框。
- **轨道连线**: 发光粒子轨迹，基于 Node Type 赋予不同光晕颜色。
- **AI 标识**: 涉及 Shadow Data 的内容强制加上“✦”星芒图标。

**性能边界与 Fallback (Performance)**
- 渲染性能：Starfield 必须维持 60fps。
- 内存控制：视口按 Chunk（如 ±3 个月）懒加载 Node。远距离节点聚合成 Cluster。
- 图像渲染：Flame 内部仅加载极低分辨率的纹理；高清原图仅在 UI Overlay 中查看时由 Flutter 加载。

**安全与隐私 (Security & Privacy)**
- 纯本地存储架构，禁止任何未授权的网络请求。
- AI 功能（如引入 API）仅保存提供商配置（Provider/Key），并且严格执行 Opt-in 策略。发送至 API 的图像会被强制下采样至 < 512x512。

**发布计划 (Distribution)**
- Windows: 提供 Portable ZIP 格式（便携版分发，无注册表污染）。
- Android: 提供全量 APK。
- 统一通过 GitHub Releases 提供下载链路。

## Testing Decisions

To ensure the system remains maintainable and performance bottlenecks are easily isolated from rendering logic, testing will target the highest possible seams:

- **Local Database Seam (`IsarRepository`)**:
  - We will test the data layer entirely independently of Flutter/Flame.
  - Tests will verify that `IsarLinks` correctly map photos to nodes, and that temporal range queries (Chunks) return accurate subsets using mock Isar instances.
  - Prior art: Standard unit testing for DAO/Repository patterns using in-memory databases.
- **Background Processing Seam (`ProcessingQueue`)**:
  - The Dart Isolate responsible for image processing will be tested by feeding it mock byte arrays representing images.
  - Tests will assert that EXIF extraction and K-Means clustering output deterministic results without relying on UI thread execution.
- **Rendering State Seam (`StarfieldState`)**:
  - The conversion of database models to renderable entities (Chunks and Clusters) will be tested via pure Dart view-models.
  - We will verify the math behind grouping distant nodes into clusters based on simulated camera coordinates, bypassing the need to instantiate the Flame engine during tests.

Tests will explicitly **not** cover pixel-perfect matching of the Flame Canvas, as this is an implementation detail subject to frequent aesthetic tweaks.

## Out of Scope

- Cloud synchronization or server-based accounts.
- Collaborative or multi-user features.
- Actual invocation of AI models (LLMs/VLMs) in Phase 1-3 (only Shadow Data schemas and config UI are prepared).
- Web or iOS platform support.
- Native C++/Rust FFI integration for image processing.

## Further Notes

All architectural decisions are documented in the `docs/adr/` directory. Please refer to `CONTEXT.md` for the strict domain vocabulary used in this project.
