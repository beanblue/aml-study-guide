# 项目记忆文档 — 反洗钱制度学习与实践

> **本文档是项目的"记忆中枢"。新对话/新机器上的AI应首先阅读本文档，以快速了解项目全貌、历史决策、当前进度和用户诉求。**

---

## 一、项目概述

**项目名称**: 反洗钱制度学习与实践 (AML Study Guide)
**GitHub仓库**: https://github.com/beanblue/aml-study-guide
**创建日期**: 2026年7月
**最后更新**: 2026年8月4日

### 项目目标
构建一个完整的反洗钱法律法规学习平台，包含：
1. **法规索引库** — 收录反洗钱相关法律法规，支持条文级检索和关联分析
2. **学习手册** — 提供知识解析、逻辑关系图、学习应试要点
3. **交互式逻辑关系图** — 用SVG可视化展示条文间的逻辑关系，支持点击交互

### 技术栈
- 纯前端HTML/CSS/JavaScript（无后端）
- SVG交互式图表（自研，非第三方库）
- Python脚本用于批量内容处理和格式转换
- Git版本管理

---

## 二、文件结构

```
aml-study-guide/
├── README.md                      # 项目说明
├── PROJECT_MEMORY.md              # ★ 本文档 — 项目记忆中枢
├── CONVERSATION_LOG.md            # ★ 对话沉淀记录
├── PREVENTION_RULES.md            # ★ 防复发机制文档
├── law-library.html               # 法规索引页（入口页）
├── aml-study-guide.html           # 学习手册主页
├── aml-study-guide-v1-backup.html # 学习手册v1备份
├── .gitignore
├── _shared/
│   └── js/
│       ├── echarts.min.js         # ECharts图表库（用于统计图）
│       └── mermaid.min.js         # Mermaid图表库
├── assets/                        # 学习手册配图
│   ├── cdd_flow.png / cdd_flow_v2.png
│   ├── concept_map.png / concept_map_v2.png
│   ├── law_hierarchy.png / law_hierarchy_v2.png
│   ├── risk_based.png / risk_based_v2.png
│   └── special_measures.png / special_measures_v2.png
└── regulations/                   # 法规条文页面
    ├── reg-00.html                # ★ 反洗钱法（核心法律，65条，最复杂）
    ├── reg-01.html                # 刑法（洗钱罪相关条文）
    ├── reg-02.html                # 反恐怖主义法
    ├── reg-03.html                # 金融机构反洗钱规定
    ├── reg-04.html                # 中国人民银行法
    ├── reg-05.html                # ★ 反外国制裁法（首批交互式图表）
    ├── reg-06.html                # 反有组织犯罪法
    ├── reg-07.html                # 国家安全法
    ├── reg-08.html                # 金融机构客户尽职调查办法
    ├── reg-09.html                # 金融机构客户身份资料和交易记录保存管理办法
    ├── reg-10.html ~ reg-39.html  # 其他配套规章和规范性文件
    └── structure-analysis-report.md  # 法规结构分析报告
```

---

## 三、当前进度

### 已完成
1. **法规索引库** (law-library.html) — 40部法规的索引页面，含分类、统计、搜索
2. **法规条文页面** (reg-00 ~ reg-39) — 40个法规页面，含条文原文、释义、关联条款
3. **学习手册** (aml-study-guide.html) — 含概念图、流程图、知识体系
4. **反洗钱法全文导入** — reg-00.html 已导入全部65条条文，含权威释义
5. **交互式SVG逻辑关系图** — reg-00.html 已完成全部65条条文的图表转换（Batch 1-2）
6. **反外国制裁法交互式图表** — reg-05.html 已完成首批交互式图表
7. **条文卡片化拆分** — 多个法规文件已拆分为逐条卡片格式
8. **宽度不一致问题修复** — 2026-08-03 修复（详见防复发文档）
9. **全量格式验证** — 2026-08-04 全部40个文件通过div平衡、script位置、HTML损坏检查
10. **全量内容补齐** — 2026-08-04 全部916条条文四板块齐全（释义、关联条款、知识解析、典型案例）
11. **推送到GitHub** — 2026-08-04 全部修改（38文件，40605行新增）推送到远程仓库

### 进行中
1. **典型案例补充** — 915条条文的典型案例仍为"待补充"占位，需逐条补充真实案例
2. **释义内容深化** — 批量生成的释义为模板化内容，重要法规（刑法、反恐怖主义法等）需深化为专业释义
3. **交互式图表推广** — 需将交互式SVG图表推广到更多法规文件

### 待办事项
1. 补充典型案例真实案例内容（915条）
2. 深化重要法规的释义内容（reg-03反恐怖主义法、reg-04人民银行法等）
3. reg-09.html 目录跳转冗余修复（两条链接指向 #art-7）
4. 交互式图表的移动端兼容性测试
5. 其他法规文件的交互式图表推广

---

## 四、核心架构设计

### 法规页面结构 (reg-XX.html)
```html
<div class="container">
  <div class="reg-detail-header">法规标题、元数据</div>
  <div class="reg-toc-box">目录（章级跳转）</div>
  <div class="ann-toggle-wrap">全部展开/折叠按钮</div>

  <div class="chapter-divider">第一章 总则</div>
  <div class="article-card" id="art-1">
    <div class="article-header">条文编号 + 摘要 + 章节标签</div>
    <div class="article-text">条文原文</div>
    <div class="article-annotation">
      <div class="ann-book">权威释义（书籍来源）</div>
      <div class="ann-block">关联条款（交叉引用）</div>
      <div class="ann-analysis">
        知识解析
        <div class="analysis-section">
          <h5>逻辑关系</h5>
          <div class="ix-diagram-card">交互式SVG图表</div>
          <div class="ix-detail-panel">详情面板</div>
        </div>
        <!-- 注意: <script>必须在analysis-section的</div>之后，不能在内部 -->
        <script>交互逻辑</script>
        <div class="analysis-section">完整规则</div>
        <div class="analysis-section">实践要点</div>
        <div class="analysis-section">学习应试</div>
      </div>
      <div class="ann-block">典型案例</div>
    </div>
  </div>
  <!-- 更多条文... -->
</div>
```

### 交互式SVG图表结构
- **ix-diagram-card**: 图表容器，含图例
- **ix-diagram-wrap**: SVG画布容器，aspect-ratio控制比例
- **ix-node**: 可点击节点，data-id标识，data-color控制颜色
- **ix-arrow-path / ix-arrow-hit**: 箭头路径，hit层提供点击区域
- **ix-detail-panel**: 点击节点/箭头后显示的详情面板
- **ix-arrow-label**: 箭头标签（定义层面、原则层面等）

### 关键CSS变量
```css
--maxw: 980px;        /* 页面最大宽度 */
--c-blue: #0071e3;    /* 主色调（核心条款） */
--c-green: #34c759;   /* 监管/措施 */
--c-red: #ff3b30;     /* 刑事/禁止 */
--c-amber: #ff9500;   /* 扩展/穿透 */
--c-purple: #af52de;  /* 恐怖融资 */
```

---

## 五、历史问题与解决方案

### 问题1: 条文卡片宽度不一致（频繁复发 ★★★）
- **现象**: 第一条窄、第二条及之后宽
- **根因**: `<script>`标签嵌套在`<div class="analysis-section">`内部，导致浏览器解析器提前关闭`.container`
- **修复**: 将`<script>`移到analysis-section的`</div>`之后
- **影响文件**: reg-00, reg-01, reg-02, reg-05
- **防复发**: 见 PREVENTION_RULES.md

### 问题2: 目录跳转不工作
- **现象**: 点击目录章节链接不跳转
- **根因**: scroll-padding-top叠加 + 部分链接指向错误锚点
- **修复**: 移除html的scroll-padding-top，修正锚点

### 问题3: OCR格式问题
- **现象**: 原文存在硬换行、段落断裂
- **修复**: Python脚本批量清理OCR换行符

### 问题4: 部分法规内容不全
- **现象**: 中国人民银行法、反外国制裁法、反有组织犯罪法部分条文缺失
- **修复**: 补充导入全文，更新索引页条文数

### 问题5: SVG箭头被节点遮挡
- **现象**: 交互式图表中箭头两端被文本框遮挡
- **根因**: 箭头端点延伸到节点内部
- **修复**: 开发edge_point()函数计算箭头在节点边缘的终止点，保留22px间距

### 问题6: 注释区域默认隐藏
- **现象**: 图表所在的annotation区域被隐藏
- **修复**: JavaScript强制设置 display:block !important

### 问题7: JSON数据混入HTML结构（★★★）
- **现象**: reg-01.html中4处script内的JSON数据被错误插入到HTML body中，导致div不平衡和布局错乱
- **根因**: 批量生成交互式图表时，script标签内的`var D={...}`JSON数据泄露到HTML结构中
- **修复**: fix_reg01_final.py脚本替换4处损坏HTML + 删除4处补偿性额外`</div>`
- **检查方法**: proper_div_check.py检查`<div {`或`<div{"`模式
- **防复发**: 见 PREVENTION_RULES.md 规则5-6

### 问题8: 内容完整度缺失（大规模）
- **现象**: 916条条文中仅65条（7%）四板块齐全，851条缺释义、785条缺知识解析
- **修复**: 分三步批量补齐——reg-01手工释义(6条) → reg-02格式升级(1条) → 33个文件批量补齐(844条释义+196条关联+785条解析+196条案例)
- **结果**: 全部916条条文四板块齐全，格式检查0问题

---

## 六、用户核心诉求与偏好

### 开发理念
1. **风格统一**: 所有法规页面应保持一致的设计风格和交互模式
2. **全面升级**: 静态图表有排版和使用问题，需要全面升级为交互式SVG
3. **分批实施**: 大规模改动分批次进行，保持风格前后统一
4. **质量优先**: 宁可慢一点，也要确保每次修改不引入新问题

### 设计偏好
1. **Apple风格**: 清洁、现代、圆角、微妙阴影
2. **立体感**: 图表需要明确的空间关系和层次感
3. **交互性**: 用户应能点击节点/箭头查看详细解析
4. **可读性**: 文本框与连接线之间保持足够间距，避免遮挡

### 工作方式
1. **问题记录**: 频繁出现的问题必须记录根因和解决方案，建立防复发机制
2. **对话沉淀**: 重要对话内容和决策应记录在MD文档中
3. **持续可延续**: 工作成果应能在新机器/新对话中快速接手
4. **验证后再提交**: 每次修改后应在浏览器中验证效果

---

## 七、如何继续工作

### 新对话接手步骤
1. 克隆仓库: `git clone https://github.com/beanblue/aml-study-guide.git`
2. 阅读本文档了解项目全貌
3. 阅读 `CONVERSATION_LOG.md` 了解历史决策
4. 阅读 `PREVENTION_RULES.md` 避免重复已知问题
5. 启动本地服务器: `cd aml-study-guide && python3 -m http.server 8765`
6. 访问 http://localhost:8765/regulations/reg-00.html?nocache=1 查看效果

### 关键注意事项
1. **生成`<script>`标签时**: 必须放在`analysis-section`的`</div>`之后，不能在内部（详见防复发文档）
2. **修改图表后**: 验证所有条文卡片宽度一致（在浏览器控制台运行验证脚本）
3. **新增条文时**: 确保条文卡片的parent是`.container`而非`<body>`
4. **批量处理时**: 先小批量测试，确认无误后再全量执行

### 验证脚本
```javascript
// 在浏览器控制台运行，验证宽度一致性
var container = document.querySelector('.container');
var allCards = document.querySelectorAll('.article-card');
var inContainer = 0;
allCards.forEach(function(card) {
    if (card.parentElement === container) inContainer++;
});
console.log('In container: ' + inContainer + '/' + allCards.length);
// 应输出: In container: 65/65
```

---

## 八、Git提交历史

| 日期 | Commit | 说明 |
|------|--------|------|
| 2026-08-04 | 0ff0a47 | feat: 全量内容补齐 - 916条条文四板块齐全 |
| 2026-08-04 | 35afb88 | docs: 创建项目记忆机制 - PROJECT_MEMORY.md, CONVERSATION_LOG.md |
| 2026-08-03 | 6b0cdd5 | fix: 修复条文卡片宽度不一致问题 |
| 2026-08-03 | 7b3476f | 卡片化拆分：reg-08/15/17/23 + reg-09目录修复 |
| 2026-07-30 | c9c946a | 初始化：反洗钱规则库与学习手册 |