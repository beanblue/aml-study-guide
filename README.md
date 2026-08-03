# 反洗钱规则库与学习手册

反洗钱制度学习与实践项目，包含法规原文、条文释义、知识解析和交互式逻辑关系图。

## 项目结构

```
aml-study-guide/
├── PROJECT_MEMORY.md      # ★ 项目记忆中枢（新对话首先阅读）
├── CONVERSATION_LOG.md    # ★ 对话沉淀记录
├── PREVENTION_RULES.md    # ★ 防复发机制文档
├── aml-study-guide.html    # 学习手册（从零基础到专业能力）
├── law-library.html        # 法规索引页
├── regulations/            # 法规条文详情
│   ├── reg-00.html         # 反洗钱法（65条，含交互式逻辑图）
│   ├── reg-01.html         # 刑法
│   ├── reg-02.html         # 反恐怖主义法
│   ├── ...
│   └── reg-39.html         # 开户管理及可疑交易报告后续控制措施通知
├── _shared/                # 共享资源
│   └── js/                 # 第三方库（echarts, mermaid）
└── assets/                 # 图片资源
```

## 功能特性

- **条文卡片化**：法规条文按条拆分为可折叠卡片，支持目录跳转
- **知识解析**：每条配备权威释义、关联条款、新旧对照、域外规定
- **交互式逻辑关系图**：SVG 可点击节点 + 箭头 + 详情面板（reg-00 已完成65个）
- **响应式设计**：适配桌面和移动端

## 技术栈

纯静态 HTML/CSS/JavaScript，无需后端，可直接用任意 HTTP 服务器托管。

## 本地预览

```bash
cd aml-study-guide
python3 -m http.server 8765
# 浏览器打开 http://localhost:8765/
```

## 新对话接手指南

1. 克隆仓库: `git clone https://github.com/beanblue/aml-study-guide.git`
2. 首先阅读 `PROJECT_MEMORY.md` 了解项目全貌和当前进度
3. 阅读 `CONVERSATION_LOG.md` 了解历史决策和用户诉求
4. 阅读 `PREVENTION_RULES.md` 避免重复已知问题
5. 启动本地服务器预览效果
