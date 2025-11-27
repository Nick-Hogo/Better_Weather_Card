# 提示词: 创建"星汉·拟真流体"天气海报 (V12)

**核心目标:** 创建一个***"新中式 (Neo-Chinese)"***风格的、具有"电影级高拟真"**物理特效的、全响应式单页网页应用。项目代号："星汉·拟真重制版"。

**技术栈 (强制):**
1.  **HTML** (单文件结构)
2.  **Tailwind CSS** (用于UI布局)
3.  **Three.js** (用于3D场景)
4.  **GLSL** (自定义着色器) (用于所有粒子物理模拟)
5. 内联SVG图标（已移除@vicons依赖）

**V8 更新内容 (2025-11-25):**
- 添加实时时钟显示（右上角，格式：时:分 + 公历日期 + 农历日期）
- 新增流星特效（仅晴空模式，使用THREE.Line，快速淡短，0.6秒持续时间，长度80-140px，透明度0.4）
- 新增孔明灯装饰效果（30-50个SVG灯笼，全屏分布，z-index: 0位于卡片下方，金黄色渐变，轻微浮动+呼吸动画）

**V12 视觉增强更新 (2025-11-26):**
- **V12.1 背景设置简化 (2025-11-26)**: 移除暗角渐变效果，采用纯净星空背景方案，使用隐式z-index管理
- **V12.2 雨效果重制优化 (2025-11-27)**: 
  - **修复核心渲染Bug**: 添加 `geometry.instanceCount = this.count` 解决雨完全不显示问题
  - **顶部生成修正**: 改为 `yPos = 600.0 - fallCycle` 让雨从页面顶端自然生成
  - **物理运动优化**: 添加风摆动 `windSway = sin()` 和柔和漂移，告别僵硬直线运动
  - **水墨形态重塑**: 实现上小下大的真实雨滴形态 `fade = pow((1.0 - (vY / 40.0 + 0.5)), 1.5)`
  - **新中式色调**: 青灰烟雨色 `vec3(0.65, 0.78, 0.95)` + 加法混合朦胧光效
- **星空背景色**: 使用深邃星空纯色 `#030712` 营造无尽深空的神秘感
- **地名重新设计**: "长安"使用 `text-4xl md:text-5xl font-bold font-msz location-glow` 配楷书字体和多层辉光效果
- **主信息卡布局优化**: 
  - 温度显示提升至 `text-8xl md:text-9xl`，作为主要信息突出
  - 天气描述采用古风四字诗句（"晴空万里"、"细雨满江"、"漫天飞雪"、"大风起兮"）
  - 温度和天气左对齐垂直居中布局，使用 `flex flex-col items-start justify-center py-8 pl-4`
- **卡片边框强化**: 增强多层阴影和白色外发光，提升与背景对比度

## 1. 核心技术架构 (双引擎渲染)
你必须使用一个***"双引擎"***渲染架构来区分不同形态的粒子:
*   **引擎一: 通用粒子系统 (Particle Engine)**
    *   **技术:** `THREE.Points` + `ShaderMaterial` (GLSL)。
    *   **用途:** 负责渲染"点状"粒子，包括 (模式0) 晴空、(模式2) 飞雪 和 (模式3) 狂风。
    *   **粒子数:** 必须"克制"，保持在 6000 - 8000 颗，以保证画面通透。
    *   **Shader 属性 (Attributes):** 必须包含 `size` (大小), `speed` (速度因子), `randomVal` (随机种子)。
*   **引擎二: 雨水系统 (Rain Engine)**
    *   **技术:** `THREE.InstancedMesh` + `ShaderMaterial` (GLSL)。
    *   **用途:** 仅用于 (模式1) 雨天。
    *   **几何体 (Geometry):** `THREE.PlaneGeometry` (一个细长的平面, 而不是一个点)。
    *   **Shader 属性 (Attributes):** 必须包含 `instancePos`, `instanceSpeed`, `instanceHeight` (用于控制雨丝长度)。
    *   **粒子数:** 1000 - 2000 条雨丝。

## 2. 物理模拟与视觉特效 (四种模式)
这是项目的核心，必须严格按以下物理模型实现:
*   **模式 0: 晴空 (Stars)**
    *   **引擎:** 引擎一 (Points)。
    *   **粒子配置:**
        1.  **粒子数量:** 桌面端7200颗，移动端3400颗（响应式动态调整）
        2.  **尺寸范围:** `Math.random() * 5 + 4` = 4-9px
        3.  **分布空间:** 
            - X轴: `(Math.random() - 0.5) * 2000` = ±1000
            - Y轴: `(Math.random() - 0.5) * 1400` = ±700
            - Z轴: `(Math.random() - 0.5) * 800` = ±400
        4.  **相机设置:** `camera.position.set(0, 0, 1400)`, FOV=45°, near=0.1, far=4000
    *   **物理与视觉:**
        1.  **位置固定:** 星星不许移动或流动。
        2.  **脉冲闪烁 (Twinkle):** 
            - 公式: `float twinkle = 0.6 + 0.4 * sin(uTime * 2.0 + randomVal * 10.0)`
            - 亮度范围: 0.2-1.0（星星会接近完全消失再亮起）
            - 颜色: `vec3(0.78, 0.83, 1.0)` 淡蓝白色
        3.  **混合模式:** `THREE.AdditiveBlending`（让星星边缘锐利、叠加发光）
        4.  **边缘渲染:** `smoothstep(0.5, 0.0, dist)` 产生柔和边缘衰减
        5.  **流星 (Meteors - V8增强):** 
            - 使用独立的`MeteorSystem`类创建流星系统
            - 技术实现：`THREE.Line` + `LineBasicMaterial` + `AdditiveBlending`
            - 运动参数：持续时间0.6秒，长度80-140px随机，透明度0.4
            - 触发概率：每秒25%随机触发（`dt * 0.25`）
            - 支持多流星同时存在（数组管理）
            - 仅在晴空模式显示（`currentMode < 0.5`时可见）
    *   **交互（视差效果 Parallax）:**
        1.  **鼠标坐标:** 使用像素坐标 `uMouse.set(event.clientX, window.innerHeight - event.clientY)`
        2.  **Viewport设置:** `uViewport` 为完整窗口尺寸 `(window.innerWidth, window.innerHeight)`
        3.  **视差计算:** `vec2 parallax = ((uMouse / uViewport) * 2.0 - 1.0) * 40.0`
            - 归一化到[-1, 1]范围
            - 乘以40.0像素作为最大偏移量
            - 仅影响XY轴，不改变Z轴深度
        4.  **效果:** 鼠标移动时产生3D深度错觉，星空像在旋转
*   **模式 1: 雨夜 (Rain)**
    *   **引擎:** 引擎二 (InstancedMesh)。
    *   **技术要求:**
        1.  **几何体配置:** `InstancedBufferGeometry` 必须正确设置 `instanceCount = this.count`
        2.  **基础形状:** `PlaneGeometry(2, 40)` 创建细长雨丝平面
        3.  **实例化属性:** `instancePos`、`instanceSpeed`、`instanceHeight` 控制每条雨丝
    *   **物理模拟 (V12.2优化):**
        1.  **顶部生成:** `yPos = 600.0 - fallCycle` 确保雨从页面顶端开始
        2.  **自然下落:** `fallCycle = mod(uTime * instanceSpeed * 120.0 + instancePos.y, 1200.0)`
        3.  **风摆效果:** `windSway = sin(uTime * 0.4 + instancePos.z * 0.3) * 15.0`
        4.  **优化漂移:** `xDrift = -fallCycle * 0.12 + windSway`（柔和斜向运动）
    *   **视觉形态 (新中式水墨感):**
        1.  **上小下大:** `fade = pow((1.0 - (vY / 40.0 + 0.5)), 1.5)` 头部细尾部粗
        2.  **烟雨色调:** `rainColor = vec3(0.65, 0.78, 0.95)` 青灰水墨色
        3.  **朦胧光效:** `AdditiveBlending` 雨丝重叠处发光
        4.  **透明度渐变:** 头部透明、尾部实体，符合真实雨滴形态
    *   **交互:** 鼠标产生"避雨"效果，附近雨丝被推开 (`smoothstep(150.0, 50.0, dist)`)。
*   **模式 2: 飞雪 (Snow)**
    *   **引擎:** 引擎一 (Points)。
    *   **物理:**
        1.  **空气动力学:** 禁止使用简单的螺旋下落。
        2.  必须使用叠加的正弦波 (e.g., `swayX` + `flutter`) 来模拟雪花在空气中"飘荡"和"翻滚"的复杂运动。
        3.  **速度差异:** `fallSpeed` 必须由 `speed` 属性调节, 模拟大雪花落得快, 小雪花飘得慢。
    *   **交互:** 鼠标必须产生强烈的"排斥" (Repel) 效果, 仿佛一阵风吹开了雪花。
*   **模式 3: 狂风 (Wind)**
    *   **引擎:** 引擎一 (Points)。
    *   **物理:**
        1.  **风洞湍流:** 模拟"风洞"效果。
        2.  **高速平移:** 粒子主体运动是高速水平移动 (`xOffset = mod(uTime * windSpeed, 2000.0)`)。
        3.  **剧烈抖动 (Turbulence):** 必须在 Y 轴上叠加高频 `sin` 抖动, 模拟沙尘被卷起的感觉。
    *   **交互:** 鼠标必须产生"漩涡" (Vortex) 效果。

## 3. UI/UX 与交互
*   **风格:** 新中式控制面板设计。UI采用黄金分割比例(1.618:1)的3卡片布局，强调灵动、简约和设计感。
*   **整体背景设置规范 (V12.1更新):** 
    *   **Body背景**: `background-color: #030712` (深邃星空色)
    *   **尺寸设置**: `width: 100vw; height: 100vh; overflow: hidden;`
    *   **Canvas配置**:
        - 使用 `position: fixed` + `top/left: 0` + `width/height: 100%`
        - 添加 `outline: none` 移除聚焦边框
        - **禁止使用** `z-index: -1`（采用隐式层级管理）
    *   **禁止使用暗角效果**: 不添加 `.vignette` 径向渐变遮罩层
    *   **WebGL渲染器**: `alpha: true` 保持画布透明，让背景色透出
    *   **设计理念**: 保持纯净简洁的星空体验，背景色均匀无渐变
*   **卡片样式 (V12强化):** 
    *   **透明玻璃材质**: 保持背景清晰透明，微妙模糊 `backdrop-filter: saturate(120%) blur(1px)`
    *   **多层边框增强**:
        - 背景: `linear-gradient(135deg, rgba(255,255,255,0.03) 0%, rgba(255,255,255,0.01) 100%)`
        - 边框: `1px solid rgba(255, 255, 255, 0.15)`
        - 强化阴影: `var(--card-shadow), 0 0 0 1px rgba(255,255,255,0.1), 0 8px 32px rgba(0,0,0,0.4), 0 0 80px rgba(255,255,255,0.05)`
        - 圆角: `1.5rem`
    *   **卡片标题**: 每个卡片顶部添加诗意标题（`.card-title`）:
        - 主信息卡: "天象 · CELESTIAL" + 地名"长安"（楷书辉光效果）
        - 诗词卡: "诗意 · POETRY"
        - 预报卡: "未来 · FORECAST"
        - 样式: `font-size: 0.75rem, letter-spacing: 0.15em, color: rgba(255,255,255,0.35), font-weight: 300`
    *   **设计原则**: 通过多层阴影和外发光强化与星空背景的对比，保持透明度让粒子效果清晰可见
*   **装饰特效 (V8新增):**
    *   **孔明灯系统:**
        - 数量: 12-20个随机生成
        - 位置: 画面右侧（65%-95%宽度范围，10%-90%高度范围）
        - z-index: 0（位于卡片下方）
        - 尺寸: 25-45px随机大小（宽高比1:1.3）
        - 样式设计: **梯形灯体 + 发光半圆底部**
            * 灯体: `<path d="M10 10 L20 10 L22 25 L8 25 Z"/>` 形成梯形轮廓
            * 外层发光半圆: `<ellipse cx="15" cy="25" rx="8" ry="5"/>` 使用径向渐变（从中心到边缘：HSL亮→暗→透明）
            * 内层高亮半圆: `<ellipse cx="15" cy="25" rx="6" ry="3.5"/>` 模拟光源核心
        - 颜色: HSL(38-56度)暖黄色调，透明度0.6-0.9随机
        - 动画: `lanternFloat`（5-9秒轻微浮动3-6px） + `lanternGlow`（呼吸闪烁0.6-1.0透明度）
        - 发光效果: CSS `filter: drop-shadow(0 0 8px rgba(255,200,80,0.9)) drop-shadow(0 0 16px rgba(255,180,60,0.6))`
        - viewBox: `0 0 30 40`
*   **字体 (V12增强):** 
    *   **主要字体**: `Noto Serif SC` (衬线体)
    *   **书法字体**: `Ma Shan Zheng` (楷书体) - 用于地名"长安"和天气古风描述
    *   **地名辉光效果**: `.location-glow { text-shadow: 0 0 20px rgba(255,255,255,0.3), 0 0 40px rgba(255,255,255,0.2), 0 0 60px rgba(255,255,255,0.1); }`
*   **布局 (V12更新 - 星空诗意控制面板):**
    1.  **整体尺寸:** `h-[80vh]`高度，Grid列比例 `1.618fr : 1fr`（黄金分割），行比例 `2fr : 1fr`（诗词卡2/3，预报卡1/3）
    2.  **时钟显示重新布局 (V12优化):** 
        - **页面右上角**: 仅显示公历日期 `text-sm md:text-base font-medium`
        - **主卡片右上角**: 时间 `text-2xl md:text-3xl font-light` + 固定农历"乙丑年 十月初三"
        - 位置: `absolute top-5 right-6`
        - 简化JavaScript: 移除solarLunar动态计算，农历写死
    3.  **左侧主卡片重新设计 (V12核心优化):** 
        - **顶部区域**: 
            * 卡片标题"天象 · CELESTIAL"
            * 地名"长安"：`text-4xl md:text-5xl font-bold font-msz location-glow`（楷书+多层辉光）
        - **中央区域（垂直居中左对齐）**:
            * 超大温度显示：`text-8xl md:text-9xl font-light`（主要信息）
            * 古风天气描述：`text-3xl font-light font-msz`配小图标（次要信息）
                - 晴空万里、细雨满江、漫天飞雪、大风起兮
            * 布局：`flex flex-col items-start justify-center py-8 pl-4`（左对齐垂直居中）
        - **底部区域**: 
            * **气象指标区 (1+2非对称布局):**
                - 左侧：AQI圆形指示器 `w-16 h-16`（独立突出）
                - 右侧：2x3网格排列其他6个指标（气压、湿度、风速、体感、紫外线、能见度）
    4.  **右上卡片 (1fr，2fr行高):** 
        - 标题: "诗意 · POETRY"
        - 诗词: 所有屏幕尺寸均为竖排 (`writing-mode: vertical-rl`)，flex-col布局，居中对齐
    5.  **右下卡片 (1fr，1fr行高):** 
        - 标题: "未来 · FORECAST"
        - 未来3天天气预报，每天之间添加 `.forecast-divider` 分割线:
            * 样式: `height: 1px, linear-gradient(90deg, transparent 0%, rgba(255,255,255,0.08) 20%, rgba(255,255,255,0.12) 50%, ...)`
            * 间距: `margin: 0.5rem 0`
    6.  **底部:** Dock栏（`transparent-glass`样式），包含4个天气切换按钮
*   **设计原则 (V7更新):**
    *   **灵动简约:** 使用3列网格布局替代非对称设计，保持视觉平衡
    *   **视觉层次分级:** 
        - 一级（最突出）：AQI圆形指示器（xl字号，径向渐变，w-14 h-14）
        - 二级（重要）：气压/湿度中等卡片
        - 三级（次要）：其他指标小型卡片
    *   **呼吸感:** 使用`gap-3`和适度padding增加留白
    *   **克制装饰:** 渐变仅用于功能性设计（AQI健康状态、卡片分割线）
*   **交互逻辑:**
    1.  **平滑过渡:** 切换天气时, `uMode` (GLSL) 和 `uGlobalOpacity` (Rain) 必须使用 `requestAnimationFrame` 进行平滑插值 (Lerp) 过渡, 时长约 800ms。
    2.  **UI 动画:** 切换天气时, 所有文本 (温度、诗词) 必须有"淡出 -> 更新内容 -> 淡入"的动画效果（300ms）。
    3.  **响应式:** 必须完美适配移动端（Grid切换为单列，flex布局自动换行）。
    4.  **实时时钟 (V8):** JavaScript每分钟更新时间显示，确保始终显示当前时间。

