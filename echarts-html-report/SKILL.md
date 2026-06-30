---
name: echarts-html-report
description: 创建基于 ECharts 的单文件交互式 HTML 数据分析报告。包含图表类型选择指南、饼图标签防重叠方案、NPS展示模板、配色方案、CSS tooltip 组件等最佳实践。当用户要求制作数据分析报告、可视化报告、交互式图表、ECharts 报告时使用。
version: 1.1.0
---

# ECharts 交互式 HTML 数据报告

## 一、报告结构模板

单文件 HTML，所有 CSS/JS 内联，仅引入 ECharts CDN：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>报告标题</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/echarts/5.6.0/echarts.min.js"></script>
    <style>/* 见下方 CSS 模板 */</style>
</head>
<body>
<div class="header"><!-- 标题区 --></div>
<div class="container">
    <div class="stats-bar"><!-- 概览统计卡片 --></div>
    <div class="section"><!-- 各数据模块 --></div>
</div>
<div class="footer"><!-- 页脚 --></div>
<script>/* ECharts 初始化 */</script>
</body>
</html>
```

## 二、CSS 基础模板

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
    background: #f0f2f5; color: #333; line-height: 1.8;
}
.header {
    background: linear-gradient(135deg, #1a365d 0%, #2b6cb0 100%);
    color: #fff; padding: 48px 0 40px; text-align: center;
}
.header h1 { font-size: 28px; font-weight: 700; letter-spacing: 2px; margin-bottom: 8px; }
.header .subtitle { font-size: 15px; opacity: 0.85; }
.container { max-width: 1100px; margin: 0 auto; padding: 0 24px; }
.stats-bar { display: flex; gap: 20px; margin: -30px auto 32px; position: relative; z-index: 1; }
.stat-card { flex: 1; background: #fff; border-radius: 12px; padding: 24px; text-align: center; box-shadow: 0 2px 12px rgba(0,0,0,0.08); }
.stat-card .num { font-size: 36px; font-weight: 700; color: #2b6cb0; }
.stat-card .label { font-size: 13px; color: #666; margin-top: 4px; }
.section { background: #fff; border-radius: 12px; padding: 32px; margin-bottom: 24px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); }
.section-title { font-size: 20px; font-weight: 700; color: #1a365d; margin-bottom: 6px; padding-left: 12px; border-left: 4px solid #2b6cb0; }
.section-desc { font-size: 14px; color: #888; margin-bottom: 24px; padding-left: 16px; }
.chart-box { width: 100%; min-height: 380px; }
.chart-row { display: flex; gap: 24px; }
.chart-row .chart-half { flex: 1; min-height: 380px; }
.insight-box { background: #f7fafc; border-left: 3px solid #2b6cb0; padding: 14px 18px; margin-top: 16px; border-radius: 0 8px 8px 0; font-size: 14px; color: #444; }
.insight-box strong { color: #1a365d; }
.tag-cloud { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 12px; }
.tag-cloud .tag { background: #edf2f7; color: #2d3748; padding: 6px 14px; border-radius: 20px; font-size: 13px; }
.tag-cloud .tag.hot { background: #ebf4ff; color: #2b6cb0; font-weight: 600; }
.summary-list { padding: 0 16px; }
.summary-list li { margin-bottom: 12px; font-size: 15px; line-height: 1.8; }
.footer { text-align: center; padding: 32px 0; color: #999; font-size: 13px; }
@media (max-width: 768px) { .stats-bar { flex-direction: column; } .chart-row { flex-direction: column; } }
```

## 三、图表类型选择指南

| 数据特征 | 推荐图表 | 关键配置 |
|----------|----------|----------|
| 占比/构成（≤5项） | 环形饼图 | `radius: ['28%','55%']`，底部水平图例 |
| 占比/构成（>5项） | 横向柱状图 | `yAxis: type:'category'`，按值降序排列 |
| 排名/对比 | 横向柱状图 | 最高值用深色，最低值用浅色 |
| 评分维度 | 雷达图 | `min` 设为3（避免中心空洞），半透明填充 |
| 分布/趋势 | 纵向柱状图 | `xAxis` 标签换行，柱宽 36px |
| 双系列对比 | 分组柱状图 | 两组用同色系深浅区分 |

## 四、图表最佳实践

### 通用初始化函数

```javascript
function initChart(id, option) {
    var dom = document.getElementById(id);
    if (!dom) return null;
    var chart = echarts.init(dom);
    chart.setOption(option);
    window.addEventListener('resize', function() { chart.resize(); });
    return chart;
}
```

### 饼图/环形图（防标签重叠方案）

**核心规则：图例放底部，饼图居中。不要用右侧垂直图例，否则右侧标签会与图例重叠。**

```javascript
{
    tooltip: { trigger: 'item', formatter: '{b}<br/>{c}人 ({d}%)' },
    legend: { orient: 'horizontal', bottom: 0, left: 'center', textStyle: { fontSize: 11 }, itemGap: 16 },
    series: [{
        type: 'pie',
        radius: ['28%', '55%'],
        center: ['50%', '45%'],
        itemStyle: { borderRadius: 6, borderColor: '#fff', borderWidth: 2 },
        label: { show: true, formatter: '{b}\n{d}%', fontSize: 11 },
        labelLine: { length: 15, length2: 10 },
        emphasis: { label: { show: true, fontSize: 13, fontWeight: 'bold', formatter: '{b}\n{c}人 ({d}%)' }},
        data: [/* 数据项 */]
    }]
}
```

### 横向柱状图（排名展示）

```javascript
{
    tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' }},
    grid: { left: 160, right: 70, top: 20, bottom: 30 },
    xAxis: { type: 'value', max: 100, axisLabel: { formatter: '{value}%' }},
    yAxis: { type: 'category', data: [/* 从低到高排列 */], axisLabel: { fontSize: 12 }},
    series: [{
        type: 'bar', barWidth: 26,
        data: [/* 用 itemStyle.color 区分高低值 */],
        label: { show: true, position: 'right', formatter: '{c}%', fontSize: 12, fontWeight: 'bold' }
    }]
}
```

### 雷达图（评分展示）

```javascript
{
    radar: {
        indicator: [{ name: '维度名\n4.80分', max: 5 }/* ... */],
        center: ['50%', '52%'], radius: '65%', min: 3,
        axisName: { fontSize: 12, color: '#444' },
        splitArea: { areaStyle: { color: ['#f7fafc', '#edf2f7', '#e2e8f0'] }}
    },
    series: [{ type: 'radar', data: [{ value: [/* ... */], areaStyle: { color: 'rgba(43,108,176,0.2)' }}] }]
}
```

## 五、配色方案

### 蓝色主色系（适合单一数据系列）

```javascript
var bluePalette = ['#1a365d', '#2b6cb0', '#4299e1', '#63b3ed', '#90cdf4', '#bee3f8', '#e2e8f0'];
```

### 红绿对比色（适合正负/好坏对比）

```javascript
var goodBad = { good: '#38a169', bad: '#e53e3e', neutral: '#e2e8f0' };
```

### 渐变色阶（适合从低到高的排名）

```javascript
var rankColors = ['#1a365d', '#2b6cb0', '#4299e1', '#63b3ed', '#90cdf4', '#bee3f8'];
```

### 语义色

- 正向/安全：`#38a169` / `#276749`
- 警告/注意：`#fc8181` / `#fed7d7`
- 危险/负面：`#c53030` / `#e53e3e`
- 中性/未知：`#e2e8f0`

## 六、NPS 展示组件

### 统计卡片中的 NPS（带 ? 提示）

```html
<div class="num">NPS 67 <span class="help-icon">?<span class="tooltip">
<strong>NPS 计算方法</strong><br>
数据来源：【C3】推荐意愿（0-10分）<br><br>
分数映射为5个类别：<br>
• <strong style="color:#90cdf4">推荐者</strong> 非常愿意（9-10分）：204人 → 68.0%<br>
• 被动者 比较愿意（7-8分）：78人 → 26.0%<br>
• 被动者 一般（5-6分）：15人 → 5.0%<br>
• <strong style="color:#fc8181">贬损者</strong> 不太愿意（3-4分）：0人 → 0.0%<br>
• <strong style="color:#fc8181">贬损者</strong> 完全不愿意（0-2分）：3人 → 1.0%<br><br>
NPS = 推荐者占比 − 贬损者占比 = 68.0 − 1.0 = <strong>67</strong>
</span></span></div>
```

### CSS（tooltip 在右侧展示）

```css
.help-icon {
    display: inline-flex; align-items: center; justify-content: center;
    width: 18px; height: 18px; border-radius: 50%;
    background: #e2e8f0; color: #666; font-size: 12px; font-weight: 700;
    cursor: help; position: relative; vertical-align: middle; margin-left: 4px;
}
.help-icon:hover { background: #cbd5e0; color: #333; }
.help-icon .tooltip {
    display: none; position: absolute; top: 50%; left: calc(100% + 12px);
    transform: translateY(-50%); background: #1a365d; color: #fff;
    padding: 14px 18px; border-radius: 10px; font-size: 13px;
    line-height: 1.7; width: 300px; text-align: left;
    box-shadow: 0 4px 20px rgba(0,0,0,0.2); z-index: 100; white-space: normal;
}
.help-icon .tooltip::after {
    content: ''; position: absolute; top: 50%; right: 100%;
    transform: translateY(-50%); border: 8px solid transparent;
    border-right-color: #1a365d;
}
.help-icon:hover .tooltip { display: block; }
```

## 七、报告模块编排

每个数据模块的标准结构：

```html
<div class="section">
    <h2 class="section-title">一、模块标题（题号）</h2>
    <div class="section-desc">题目描述文字</div>
    <div class="chart-row">
        <div id="chart-xxx" class="chart-half"></div>
        <div id="chart-yyy" class="chart-half"></div>
    </div>
    <div class="insight-box">
        <strong>核心发现：</strong>一句话总结关键数据点和业务含义。
    </div>
</div>
```

**编排原则**：
- 概览统计卡片放在最前面（3-5个核心指标）
- 相关题目合并到一个 section（用 chart-row 左右排列）
- 每个 section 末尾用 insight-box 总结核心发现
- 最后一个 section 放调研总结（summary-list）

### 标签云组件（开放题高频主题展示）

适用场景：开放题/填空题的高频关键词/主题可视化。

```html
<div class="tag-cloud">
    <span class="tag hot">最热主题1</span>
    <span class="tag hot">最热主题2</span>
    <span class="tag">次要主题1</span>
    <span class="tag">次要主题2</span>
    <span class="tag">其他主题</span>
</div>
```

- `.tag.hot` 用于出现频率最高的 2-4 个主题，蓝色高亮
- `.tag` 用于其他普通主题，灰色底

### 开放题典型反馈摘录模板

```html
<div class="section">
    <h2 class="section-title">十二、机构开放式建议摘要</h2>
    <div class="section-desc">部分机构在问卷末尾提出的建议和诉求（共收到 N 条有效反馈）</div>
    <p style="font-size:14px;color:#555;margin-bottom:16px;">以下为出现频率较高的核心诉求主题：</p>
    <div class="tag-cloud">
        <span class="tag hot">主题1</span>
        <span class="tag hot">主题2</span>
        <span class="tag">主题3</span>
        <!-- ... -->
    </div>
    <div style="margin-top:20px;">
        <p style="font-size:14px;color:#555;margin-bottom:8px;font-weight:600;">典型反馈摘录：</p>
        <div class="insight-box" style="margin-top:8px;">"第一条摘录原文。"</div>
        <div class="insight-box">"第二条摘录原文。"</div>
        <div class="insight-box">"第三条摘录原文。"</div>
    </div>
</div>
```

### 调研总结模块模板

```html
<div class="section">
    <h2 class="section-title">调研总结</h2>
    <ul class="summary-list">
        <li><strong>行业结构：</strong>总结行业基本面的1-2句关键发现。</li>
        <li><strong>经营状况：</strong>总结经营压力的1-2句关键发现。</li>
        <li><strong>投入策略：</strong>总结投入方向的1-2句关键发现。</li>
        <li><strong>合规情况：</strong>总结合规管理的1-2句关键发现。</li>
        <li><strong>政策态度：</strong>总结政策评价的1-2句关键发现。</li>
        <li><strong>核心诉求：</strong>总结诉求期待的1-2句关键发现。</li>
    </ul>
</div>
```

## 八、报告必含元素清单（生成前自检）

每份 HTML 报告**必须**包含以下所有元素：

| 序号 | 元素 | 说明 |
|------|------|------|
| ✅ header | 渐变色标题区，含主标题 + 副标题（数据来源+日期） |
| ✅ stats-bar | 3-4 个核心指标概览卡片 |
| ✅ section × N | 每个分析题目一个 section，含 section-title + section-desc + chart + insight-box |
| ✅ insight-box | **每个** section 末尾的"核心发现"文字总结，不可省略 |
| ✅ chart-row | 相关题目用 chart-row 左右并排（如 H1+H2，I2+I3） |
| ✅ section-desc | 每节的题目描述文字，尽量写完整的问卷题干 |
| ✅ 开放题模块 | 如有开放题（Q15 等），需展示标签云 + 典型摘录 |
| ✅ 调研总结 | 最后一个 section，用 summary-list 总结全报告关键发现 |
| ✅ footer | 页脚，含报告名称 + 日期 + 样本量 |
| ✅ JS resize | 所有图表通过 initChart 绑定 resize 事件 |
| ✅ 响应式 | CSS 中配置 @media (max-width: 768px) 移动端适配 |

## 九、常见陷阱

- **饼图标签与图例重叠**：根因是右侧垂直图例与右侧标签空间冲突 → 改用底部水平图例，饼图居中
- **标签默认隐藏**：`label: { show: false }` 导致默认状态无标签 → 必须 `show: true`
- **引导线过长**：`labelLine.length/length2` 值过大会导致标签飞很远 → 保持 15/10 左右
- **tooltip 被遮挡**：`position: absolute` 未设 z-index → 加 `z-index: 100`
- **图表不响应窗口变化**：忘记绑定 `resize` 事件 → 用 initChart 统一处理
- **柱状图标签被截断**：`grid.right` 太小 → 留足 60-80px 给右侧标签
- **全部图表空白（最严重）**：用 Python f-string 生成 HTML 时，`formatter: '{b}\n{d}%'` 中的 `\n` 会被 Python 处理成**真实换行符**，导致 JS 单引号字符串跨行 → `SyntaxError` → 所有图表渲染失败。**修复**：Python 侧必须用 `\\n`（例如 `label: {{ show: true, formatter: '{{b}}\\n{{d}}%' }}`），确保输出到 HTML 的 JS 代码中是 `\n` 两个字符而非真实换行
- **报告缺少总结性文字**：仅有图表没有 insight-box 和调研总结会导致报告"只有图没有结论"。必须：①每个 section 末尾加 insight-box 写一段核心发现；②最后一个 section 放调研总结（summary-list）用文字提炼全局关键发现；③若有开放题，必须加标签云 + 典型摘录模块
