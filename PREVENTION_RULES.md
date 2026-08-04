# 防复发机制文档：条文卡片宽度不一致问题

## 问题概述

**问题名称**: 法规页面条文卡片宽度不一致（第一条窄、第二条及之后宽）
**发现日期**: 2026-08-03
**影响文件**: reg-00.html, reg-01.html, reg-02.html, reg-05.html
**复发频率**: 频繁（每次修正后都会重新出现）

## 根本原因

### 技术原因

在法规页面的"知识解析"部分，交互式SVG图表的 `<script>` 标签被放置在 `<div class="analysis-section">` 内部：

```html
<!-- 错误结构（导致问题） -->
<div class="analysis-section">
  <h5>逻辑关系</h5>
  <div class="ix-diagram-card">...</div>
  <div class="ix-detail-panel">...</div>
  <script>
    (function(){ ... })();
  </script>
</div>
```

浏览器的HTML解析器在处理嵌套在analysis-section内的 `<script>` 标签时，会错误地提前关闭父级div，导致 `.container` 容器在第一条之后提前关闭。这使得第二条及之后的条文卡片逃逸到 `<body>` 元素下，失去了容器的padding约束（22px），表现为更宽。

### DOM结构变化

```
修复前:
<body>
  <div class="container">
    <div class="article-card" id="art-1">  <!-- 837px (正确) -->
    </div>  ← container在这里提前关闭！
  </div>
  <div class="article-card" id="art-2">    <!-- 881px (错误，在body下) -->
  <div class="article-card" id="art-3">    <!-- 881px (错误) -->
  ...

修复后:
<body>
  <div class="container">
    <div class="article-card" id="art-1">  <!-- 730px (正确) -->
    <div class="article-card" id="art-2">  <!-- 730px (正确) -->
    <div class="article-card" id="art-3">  <!-- 730px (正确) -->
    ...
  </div>
</body>
```

## 修复方案

将 `<script>` 标签从 `<div class="analysis-section">` 内部移出到其外部（成为兄弟元素）：

```html
<!-- 正确结构 -->
<div class="analysis-section">
  <h5>逻辑关系</h5>
  <div class="ix-diagram-card">...</div>
  <div class="ix-detail-panel">...</div>
</div>
<script>
  (function(){ ... })();
</script>
```

## 防复发规则

### 规则1: Script标签放置规范
**任何 `<script>` 标签不得放置在 `<div class="analysis-section">` 内部。** Script标签应作为analysis-section的兄弟元素，放置在其关闭 `</div>` 之后。

### 规则2: 批量转换脚本检查
在使用Python脚本批量生成或修改HTML时，生成 `<script>` 标签的代码必须将script标签放置在analysis-section的 `</div>` 之后，而非内部。

### 规则3: 新增条文时的验证
新增或修改条文后，必须验证所有条文卡片的父元素是否为 `.container`，而非 `<body>`。可通过以下JavaScript验证：

```javascript
// 验证脚本：在浏览器控制台运行
var container = document.querySelector('.container');
var allCards = document.querySelectorAll('.article-card');
var inContainer = 0;
allCards.forEach(function(card) {
    if (card.parentElement === container) inContainer++;
});
console.log('In container: ' + inContainer + '/' + allCards.length);
// 应输出: In container: 65/65 (全部在container内)
```

### 规则4: 转换脚本模板
生成交互式图表时，使用以下模板结构：

```python
# Python生成代码模板
html = f'''
<div class="analysis-section">
  <h5>{title}</h5>
  <div class="ix-diagram-card" id="ix-card-{art_id}">
    {diagram_html}
  </div>
  <div class="ix-detail-panel empty" id="ix-panel-{art_id}">点击上方节点或箭头，查看详情</div>
</div>
<script>
(function(){{
  var D = {json_data};
  var P = document.getElementById("ix-panel-{art_id}");
  var C = document.getElementById("ix-card-{art_id}");
  // ... 交互逻辑 ...
}})();
</script>
'''
# 注意: </script> 必须在 </div> (analysis-section的关闭标签) 之后
```

## 修复记录

| 日期 | 文件 | 修复数量 | 说明 |
|------|------|----------|------|
| 2026-08-03 | reg-00.html | 65个script | 反洗钱法，全部65条条文的图表脚本 |
| 2026-08-03 | reg-01.html | 6个script | 刑法相关条文 |
| 2026-08-03 | reg-02.html | 1个script | 单条条文 |
| 2026-08-03 | reg-05.html | 5个script | 反外国制裁法 |

## 验证方法

1. **浏览器验证**: 打开页面，在控制台运行验证脚本（规则3）
2. **视觉验证**: 第一条和第二条的卡片宽度应该完全一致
3. **DOM验证**: 所有 `.article-card` 的 `parentElement` 应为 `.container`
4. **脚本验证**: 运行 `python3 proper_div_check.py` 检查所有文件的div平衡、script位置和HTML损坏

---

## 问题2: JSON数据混入HTML结构（新类型）

### 问题概述
**发现日期**: 2026-08-04
**影响文件**: reg-01.html（刑法）
**问题类型**: script中的`D={...}`JSON数据被错误地插入到HTML结构中

### 根本原因
在生成交互式SVG图表时，script标签内的JSON数据（节点描述数据`D={"nx..."}`）被错误地写入到HTML body中，替代了正常的HTML结构。每处损坏都会：
1. 引入一个额外的`<div`开标签（浏览器将其解析为div open）
2. 丢失原本应该存在的HTML内容（legend items、SVG头部、条文过渡等）
3. 后续结构中出现补偿性的额外`</div>`来平衡

### 检查方法
使用`proper_div_check.py`脚本检查：
```python
# 检查JSON数据混入（<div { 或 <div{" 模式）
if '<div {' in line or '<div{"' in line:
    issues.append((li, 'JSON数据混入div标签'))
```

### 防复发规则5: JSON数据不得出现在HTML body中
生成script标签内容时，确保`var D={...}`等JSON数据只在`<script>`标签内部，不会泄露到HTML结构中。批量生成脚本应分别处理HTML结构和JavaScript内容，不要将两者混合写入同一行。

### 防复发规则6: 使用正确的div检查脚本
检查div平衡时，必须跳过`<script>`和`<style>`标签内的内容，否则script中的HTML字符串（如`d.innerHTML='<div>...'`）会导致假阳性。使用`proper_div_check.py`而非简单的正则匹配。

### 修复记录（续）

| 日期 | 文件 | 修复内容 | 说明 |
|------|------|----------|------|
| 2026-08-04 | reg-01.html | 4处JSON损坏+4处额外close | art-121/191/312/349的图表区域和条文过渡 |

---

## 问题3: 交互式图表三大系统性缺陷（箭头不可见、点击无响应、布局偏移）

### 问题概述
**发现日期**: 2026-08-04
**影响文件**: reg-00.html（64个图表）、reg-01.html、reg-02.html、reg-05.html
**问题类型**: SVG viewBox不匹配、箭头标记过小、JavaScript字符串引号冲突、JS对象结构不完整

### 根本原因（四个子问题）

#### 子问题A: SVG viewBox与CSS aspect-ratio不匹配
- **现象**: 箭头线段与节点位置偏移，箭头被节点遮挡
- **根因**: SVG的`viewBox`尺寸（如`680×480`、`680×500`、`680×520`）与CSS`.ix-diagram-wrap{aspect-ratio:680/425}`不一致，导致SVG内部坐标系与HTML百分比定位的节点错位
- **影响范围**: reg-00.html中3个图表（art51: 680×480, art60: 680×520, art61: 680×480），art27已修复

#### 子问题B: 箭头标记过小不可见
- **现象**: 连接线段看不到箭头头，用户无法判断方向
- **根因**: 
  - marker尺寸仅8×8像素，太小
  - 未使用`markerUnits="userSpaceOnUse"`，marker随stroke-width缩放后更小
  - stroke颜色`#a1a1a6`、透明度0.55，过于浅淡
- **影响范围**: 全部4个文件共63个marker全部为8×8

#### 子问题C: JavaScript字符串引号冲突（★★★★ 最严重）
- **现象**: 点击节点/箭头无任何反应，详情面板始终空白
- **根因**: JS数据对象的字符串值中，使用ASCII双引号`"..."`作为中文引号（如`"第27条是反洗钱法第三章"反洗钱义务"的统领性条款"`），JS解析器将第二个`"`视为字符串终止符，导致语法错误，整个IIFE静默失败
- **影响范围**: reg-00 art27（已修复）、reg-05（存在类似问题）

#### 子问题D: JS对象结构不完整
- **现象**: 浏览器控制台报`Uncaught SyntaxError: Unexpected token ';'`
- **根因**: JS数据对象最后一个条目缺少闭合`}`（如`c-b4`对象只有`]`没有`]}`）
- **影响范围**: reg-00 art27（已修复）

### 防复发规则7: SVG viewBox必须与CSS aspect-ratio一致
所有交互式SVG图表的`viewBox`必须为`0 0 680 425`，与CSS`.ix-diagram-wrap{aspect-ratio:680/425}`严格一致。生成图表时不得使用其他尺寸。

**验证方法**:
```python
# 检查所有viewBox是否为680x425
import re
mismatches = re.findall(r'<svg id="ix-svg-[^"]+" viewBox="0 0 (\d+) (\d+)"', content)
for w, h in mismatches:
    if w != '680' or h != '425':
        print(f'MISMATCH: {w}x{h}')
```

### 防复发规则8: 箭头标记不得小于12×12且必须使用绝对尺寸
所有`<marker>`元素必须：
1. `markerWidth`和`markerHeight`不小于12
2. 添加`markerUnits="userSpaceOnUse"`属性，确保marker尺寸不随stroke-width缩放
3. fill颜色不浅于`#6e6e73`
4. 配套的`.ix-arrow-path`的stroke不浅于`#6e6e73`，opacity不低于0.75

**标准marker模板**:
```html
<marker id="ix-ah{art_id}" markerUnits="userSpaceOnUse" markerWidth="16" markerHeight="16" refX="14" refY="8" orient="auto">
  <path d="M0,0 L15,8 L0,16 Z" fill="#6e6e73"></path>
</marker>
```

### 防复发规则9: JS字符串内部的中文引号必须使用Unicode弯引号
在JavaScript双引号字符串值内部，中文引号必须使用Unicode弯引号`\u201c`（"）和`\u201d`（"），**严禁**使用ASCII双引号`"`。

**错误示例**（会导致JS语法错误）:
```javascript
"desc": "第27条是反洗钱法第三章"反洗钱义务"的统领性条款"  // ← 第三个"终止了字符串
```

**正确示例**:
```javascript
"desc": "第27条是反洗钱法第三章\u201c反洗钱义务\u201d的统领性条款"
```

**验证方法**: 在浏览器控制台检查是否有`SyntaxError`。或在Python中检查JS数据块内是否有多于2个ASCII双引号且无Unicode弯引号的行。

### 防复发规则10: 生成JS数据对象后必须验证结构完整性
使用脚本生成`var D={...}`数据对象后，必须：
1. 检查每个对象条目是否正确闭合（`}`配对）
2. 在浏览器控制台确认无`SyntaxError`
3. 确认`typeof show`不是`undefined`（IIFE正常执行）

### 修复记录（续2）

| 日期 | 文件 | 修复内容 | 说明 |
|------|------|----------|------|
| 2026-08-04 | reg-00.html art27 | viewBox 680×500→680×425 | 修复SVG与CSS比例不匹配 |
| 2026-08-04 | reg-00.html art27 | marker 8×8→16×16 + userSpaceOnUse | 修复箭头不可见 |
| 2026-08-04 | reg-00.html art27 | ASCII引号→Unicode弯引号 | 修复JS语法错误导致点击无响应 |
| 2026-08-04 | reg-00.html art27 | 补全c-b4对象闭合`}` | 修复Unexpected token语法错误 |
| 2026-08-04 | reg-00.html art27 | stroke颜色加深 #a1a1a6→#6e6e73 | 提升箭头可见度 |
