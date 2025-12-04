# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

StoryGen Atelier 是一个 AI 辅助的分镜和视频生成工具，使用 Gemini 生成分镜文本和帧，Vertex AI Veo 生成过渡片段，并使用 ffmpeg 合成最终视频。内置日志和画廊管理功能。

## 核心架构

### 视频生成算法（插值链/滑动窗口策略）
该系统采用三阶段自动化流程将静态分镜转换为连贯视频：

1. **过渡分析（Gemini）**：使用滑动窗口分析相邻分镜，生成具体过渡提示和建议时长
2. **片段生成（Vertex AI Veo）**：基于 Gemini 生成的提示，并行生成连接视频片段
3. **最终合成（FFmpeg）**：使用 `concat` 协议和 copy 模式无损合并所有片段

### 技术栈
- **Frontend**: React, Vite, Mantine UI
- **Backend**: Node.js (Express), better-sqlite3, fluent-ffmpeg
- **AI Services**: Google Gemini (Text/Image), Google Vertex AI Veo (Video Generation)

## 常用开发命令

### 快速启动（推荐）
```bash
# 赋予执行权限（仅需一次）
chmod +x start_servers.sh
# 启动前后端服务
./start_servers.sh
```

### 手动启动
后端（端口 3005）：
```bash
cd backend
npm install
cp .env.example .env  # 填入真实 API 密钥
npm run dev          # 或 npm start
```

前端（端口 5180）：
```bash
cd frontend
npm install
npm run dev
```

### 构建与停止
```bash
# 构建前端
cd frontend && npm run build

# 停止所有服务
./stop_servers.sh
```

## 环境配置

在 `backend/.env` 中配置（从 `.env.example` 复制）：
```
PORT=3005
GEMINI_API_KEY=your_gemini_api_key
GEMINI_TEXT_MODEL=gemini-3-pro-preview
GEMINI_IMAGE_MODEL=gemini-3-pro-image-preview

# Gemini API 自定义端点（可选）
# 取消注释并设置以使用自定义端点代替默认 Google 端点
# GEMINI_BASE_URL=https://your-custom-endpoint.com

# Gemini API 自定义请求头（可选）
# JSON 格式的额外 HTTP 请求头
# GEMINI_CUSTOM_HEADERS={"User-Agent": "StoryGen-Atelier/1.0", "X-Custom-Header": "value"}

# Vertex AI (视频生成必需)
VERTEX_PROJECT_ID=your_gcp_project_id
VERTEX_LOCATION=us-central1
VERTEX_VEO_MODEL=veo-3.1-generate-preview
```

## 关键目录结构

```
backend/          # Node.js + Express API，调用 Gemini/Vertex，管理日志和数据
frontend/         # React + Vite + Mantine UI
guide/            # 提示词指南
exampleImg/       # README 示例图片（从本地数据导出）
backend.log / frontend.log  # 运行时日志
```

## 核心服务架构

### Backend Routes
- `/api/storyboard` - 分镜生成和管理
- `/api/gallery` - 画廊存储和检索
- `/api/video-logs` - 视频生成日志
- `/api/storyboard-logs` - 分镜生成日志

### Frontend Components
- `App.jsx` - 主应用组件，包含所有核心功能
- `VideoLogs.jsx` - 日志查看组件
- `api.js` - API 调用封装

### 数据存储
- SQLite 数据库：`backend/data/gallery.db`
- 视频文件存储：`backend/data/videos/`
- 临时文件：`backend/data/tmp/`

## 开发注意事项

1. **代理支持**：后端支持通过环境变量配置 HTTP 代理
2. **CORS 配置**：已为所有路由启用 CORS
3. **文件大小限制**：JSON 解析限制为 20MB 以支持 base64 图片
4. **静态文件服务**：视频文件通过 `/videos` 路径提供静态服务
5. **日志管理**：所有操作都有详细的日志记录，支持查看、导出和清空

## 风格预设

应用包含 15 种预设风格（赛博朋克、电影写实、水彩画、动漫风等），支持自定义风格描述。所有风格定义在前端的 `stylePresets` 数组中。

## API 安全与合规

项目内置完整的安全策略和提示词指南，详见 `guide/VideoGenerationPromptGuide.md`，确保生成内容符合 Google Cloud 政策要求。