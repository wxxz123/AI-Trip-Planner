# AI Trip Planner

<div align="center">
  <p>基于 <strong>HelloAgents</strong> 框架构建的智能旅行规划助手。集成高德地图 MCP 服务、Unsplash 服务以及强大的前沿 LLM，为您提供一站式、个性化的原生级多日旅行计划生成体验。</p>
  <p>
    <img src="https://img.shields.io/badge/Vue-3.x-brightgreen.svg" alt="Vue3">
    <img src="https://img.shields.io/badge/TypeScript-5.x-blue.svg" alt="TypeScript">
    <img src="https://img.shields.io/badge/Vite-5.x-646CFF.svg" alt="Vite">
    <img src="https://img.shields.io/badge/FastAPI-0.100+-009688.svg" alt="FastAPI">
    <img src="https://img.shields.io/badge/Python-3.10+-blue.svg" alt="Python">
  </p>
</div>

---

## ✨ 核心亮点 & 绚丽界面

本项目在兼顾强大的 AI 逻辑的同时，也注重呈现**专业级、数字互动的用户界面**：
- 🎨 **暗色拟态玻璃风 (Dark Glassmorphism)**：Home主页采用前沿的深色毛玻璃美学设计，科技感十足。
- 📊 **仪表盘式展示 (Dashboard Layout)**：Result结果页采用清晰的仪表盘布局，一目了然。
- ⏱️ **时间轴行程规划 (Timeline Itinerary)**：互动式时间轴呈现每日游玩顺序。
- 🗺️ **交互式地图 (Interactive Map)**：直观标记各个景点POI，支持点击与缩放。
- 💰 **可视化预算拆解 (Visual Budget Breakdown)**：以生动的方式拆解各个旅行环节的费用明细。
- 🤖 **MCP 协议驱动**：Agent 自动调用高德地图 MCP 工具，精准获取周边路线、天气预报及最新的景点 POI 信息。

## 🏗️ 全栈架构与请求生命周期

本项目是一个前后端分离的agent应用，以下展示了项目的架构流转：

### 1. 端口与网络流转
- **前端 (Vue 3 + Vite)**：默认运行在 `http://localhost:5173`。负责与用户交互，呈现优美的 UI，接收表单输入。
- **后端 (FastAPI)**：默认运行在 `http://localhost:8000`。负责暴露并提供核心 API 接口 (`/api/trip/plan` 等)，并代理与所有外部 API 服务（如模型提供商、高德开放平台、图片库）的交互，避免前端产生跨域或 API Token 泄露问题。

### 2. 核心 API Key 及其作用
- **LLM 提供商 (如 DeepSeek, OpenAI)**：在后端处理自然语言理解，通过 `HelloAgents` 框架实例化 `SimpleAgent`，推理并生成精细化的行程细节。
- **高德地图 (Amap)**：
  - **后端（Web 服务 API）**：通过 `amap-mcp-server` 供 Agent 自动检索天气、查询步行/驾车路线、搜索 POI。
  - **前端（Web 端 JS API）**：用于在 Result 页面渲染高德交互式可视化地图组件。
- **Unsplash API**：后端负责调用 Unsplash API，为每个行程目的地或景点获取高清的封面图和唯美配图，提升结果展示的生动性。

### 3. 请求-响应生命周期
1. 用户在前端填写**目的地、天数、交通与住宿偏好**等细致参数并提交表单。
2. 前端向后端发起 `POST /api/trip/plan` 携带参数数据的请求。
3. 后端 `trip_planner_agent` 被激活：
   - LLM 分析出行业务需求并提炼必须信息。
   - LLM 自主决定是否/如何调用 `amap-mcp-server` 暴露的各类 MCP Tools。
   - 搜集好动态地图与生活维度数据后，LLM 组合成标准的行程 JSON 格式对象返回给 FastAPI 应用层。
   - FastAPI 取用 JSON，结合服务层（查询到的 Unsplash 图片资源）后，将其作为统一的 HTTP 响应体抛回给前端。
4. 前端 Axios 接收到数据流，利用 Vue 的响应式系统，动态注入到时间轴及控制面板中。

---

## 🚀 快速开始

本项目由 `frontend` 和 `backend` 两大部分组成。部署前请确保您的系统环境已包含 **Node.js 16+** 及 **Python 3.10+**。

### 第一步：后端服务部署 (Port: 8000)

1. **进入开发目录与创建虚拟环境**
```bash
cd backend
python -m venv venv
# 激活环境
source venv/bin/activate       # Mac/Linux 系统
# .\venv\Scripts\activate      # Windows 系统
```

2. **安装工程依赖包**
```bash
pip install -r requirements.txt
```

3. **配置系统环境变量**
复制示例配置 `.env.example` 并重命名为 `.env`。
```bash
cp .env.example .env
```
使用文本编辑器修改 `.env` 文件，完善你的外部服务认证凭证（API Keys）：
```env
# LLM 配置及安全接入
LLM_MODEL_ID="deepseek-chat"
LLM_API_KEY="your_llm_api_key_here"

# 高德地图 Web 服务 API Key (供给后端MCP工具服务网关使用)
AMAP_API_KEY="your_amap_web_service_api_key"

# Unsplash 图片服务 API
UNSPLASH_ACCESS_KEY="your_unsplash_access_key"
UNSPLASH_SECRET_KEY="your_unsplash_secret_key"
```

4. **启动服务引擎**
```bash
python run.py
```
> [!NOTE] 
> 后端启动后将监听 `http://localhost:8000`。您可通过浏览器访问 `http://localhost:8000/docs` 查看并测试 Fast API 自带的 Swagger 互动式接口文档。

### 第二步：前端服务部署 (Port: 5173)

1. **进入目录与安装全栈依赖**
⚠️ **请额外开启一个新的终端窗口（Terminal）**：
```bash
cd frontend
npm install
```

2. **配置客户端环境变量**
与后端类似，复制示例配置文件：
```bash
cp .env.example .env
```
在 `.env` 中填写服务访问地址与高德地图**前端专用的**密钥：
```env
VITE_API_BASE_URL=http://localhost:8000
# 高德地图 Web端 JS API Key 及相应的安全验证密钥
VITE_AMAP_WEB_JS_KEY="your_js_api_key"
VITE_AMAP_SECURITY_JS_CODE="your_security_code"
```

3. **启动热加载的研发客户端**
```bash
npm run dev
```
打开您偏好的浏览器，访问 `http://localhost:5173` 体验全屏界面。

---

### 展示规划成功后的页面
![展示图](image1)

## 📁 主要项目结构视图

```
helloagents-trip-planner/
├── backend/                    # Python FastAPI 后端服务
│   ├── app/
│   │   ├── agents/             # HelloAgents 智能规划代理层
│   │   ├── api/                # FastAPI 路由控制器配置模块
│   │   ├── services/           # 外部生态挂载 (Amap, Unsplash, LLM等)
│   │   ├── models/             # Pydantic 健壮性数据规范类型
│   │   └── config.py           # 中心化环境变量记载与反序列化
│   └── requirements.txt
├── frontend/                   # Vue 3 渲染客户端应用
│   ├── src/
│   │   ├── components/         # 低耦合、独立的可复用 UI 视图组件
│   │   ├── views/              # 主路由呈现视图 (Home.vue / Result.vue)
│   │   ├── services/           # Axios 抽象网络通信网关
│   │   └── types/              # TS 声明类型
│   └── package.json
└── README.md
```

## 🤝 贡献与常见问题排查建议

欢迎以 Pull Request (PR) 形式对交互动画效果、Prompt 指令及业务逻辑进行完善贡献。
在本地部署时经常遇到的常见问题解决清单：
1. **CORS 跨站访问冲突报错**：请检查你正在访问的端口是否为 5173。并且 `backend/.env` 中的 `CORS_ORIGINS` 配置是否精确包罗了您的前端实际运行地址(如`http://localhost:5173`)。
2. **地图白屏或加载失败**：很大可能源自您未将 `VITE_AMAP_SECURITY_JS_CODE` 匹配填入 `frontend/.env` 里，高德在最新版引入了多一道鉴权校验防白嫖。
3. **照片资源无法浏览返回全白占位符**：因 Unsplash 返回限额可能已过，或是您的 `UNSPLASH_ACCESS_KEY` 无效导致。

## 📜 开源协议与特别致谢
- **开源协议**: CC BY-NC-SA 4.0
- **特别致谢**: 感谢 [Datawhale](https://github.com/datawhalechina)的[HelloAgents框架](https://github.com/datawhalechina/Hello-Agents) 以及高德地图 MCP Server 为本开源系统构筑了夯实的代理底层体系与基石工具集。

---

**AI Trip Planner** - 让您的每一次起航，都满载科技感与掌控感。🌈
