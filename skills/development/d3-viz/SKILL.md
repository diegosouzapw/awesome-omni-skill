---
name: d3-viz
description: 使用 d3.js 创建交互式数据可视化。此技能应在创建自定义图表、图形、网络图、地理可视化或任何需要对视觉元素、转换或交互进行细粒度控制的复杂基于 SVG 的数据可视化时使用。用于标准图表库之外的定制可视化，无论是 React、Vue、Svelte、原生 JavaScript 还是任何其他环境。
---

# D3.js 可视化

## 概述

此技能提供关于使用 d3.js 创建高级交互式数据可视化的指导。D3.js（Data-Driven Documents）擅长将数据绑定到 DOM 元素并对它们应用数据驱动的转换，以创建具有对每个视觉元素精确控制的自定义、出版质量的可视化效果。这些技术适用于任何 JavaScript 环境，包括原生 JavaScript、React、Vue、Svelte 和其他框架。

## 何时使用 d3.js

**在以下情况下使用 d3.js：**
- 需要独特视觉编码或布局的自定义可视化
- 具有复杂平移、缩放或刷选行为的交互式探索
- 网络/图可视化（力导向布局、树状图、层次结构、弦图）
- 地理可视化与自定义投影
- 需要流畅、协调转换的可视化
- 具有精细样式控制的出版质量图形
- 标准库中不可用的新颖图表类型

**对于以下情况考虑替代方案：**
- 3D 可视化 - 改用 Three.js

## 核心工作流程

### 1. 设置 d3.js

在脚本顶部导入 d3：

```javascript
import * as d3 from 'd3';
```

或者使用 CDN 版本（7.x）：

```html
<script src="https://d3js.org/d3.v7.min.js"></script>
```

所有模块（比例尺、轴、形状、过渡等）都可以通过 `d3` 命名空间访问。

### 2. 选择集成模式

**模式 A：直接 DOM 操作（推荐用于大多数情况）**
使用 d3 选择 DOM 元素并以命令式方式操作它们。这适用于任何 JavaScript 环境：

```javascript
function drawChart(data) {
  if (!data || data.length === 0) return;

  const svg = d3.select('#chart'); // 按 ID、类或 DOM 元素选择

  // 清除之前的内容
  svg.selectAll("*").remove();

  // 设置尺寸
  const width = 800;
  const height = 400;
  const margin = { top: 20, right: 30, bottom: 40, left: 50 };

  // 创建比例尺、轴和绘制可视化
  // ... d3 代码在这里 ...
}

// 在数据更改时调用
drawChart(myData);
```

**模式 B：声明式渲染（用于具有模板的框架）**
使用 d3 进行数据计算（比例尺、布局），但通过您的框架渲染元素：

```javascript
function getChartElements(data) {
  const xScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .range([0, 400]);

  return data.map((d, i) => ({
    x: 50,
    y: i * 30,
    width: xScale(d.value),
    height: 25
  }));
}

// 在 React 中: {getChartElements(data).map((d, i) => <rect key={i} {...d} fill="steelblue" />)}
// 在 Vue 中: v-for 指令遍历返回的数组
// 在原生 JS 中: 根据返回的数据手动创建元素
```

对于具有转换、交互或利用 d3 完整功能的复杂可视化，请使用模式 A。对于更简单的可视化或当您的框架更喜欢声明式渲染时，请使用模式 B。

### 3. 构建可视化代码

在您的绘图函数中遵循此标准结构：

```javascript
function drawVisualization(data) {
  if (!data || data.length === 0) return;

  const svg = d3.select('#chart'); // 或传递选择器/元素
  svg.selectAll("*").remove(); // 清除之前的渲染

  // 1. 定义尺寸
  const width = 800;
  const height = 400;
  const margin = { top: 20, right: 30, bottom: 40, left: 50 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  // 2. 创建带边距的主要组
  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // 3. 创建比例尺
  const xScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.x)])
    .range([0, innerWidth]);

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.y)])
    .range([innerHeight, 0]); // 注意：SVG 坐标系翻转

  // 4. 创建并附加轴
  const xAxis = d3.axisBottom(xScale);
  const yAxis = d3.axisLeft(yScale);

  g.append("g")
    .attr("transform", `translate(0,${innerHeight})`)
    .call(xAxis);

  g.append("g")
    .call(yAxis);

  // 5. 绑定数据并创建视觉元素
  g.selectAll("circle")
    .data(data)
    .join("circle")
    .attr("cx", d => xScale(d.x))
    .attr("cy", d => yScale(d.y))
    .attr("r", 5)
    .attr("fill", "steelblue");
}

// 在数据更改时调用
drawVisualization(myData);
```

### 4. 实现响应式尺寸调整

使可视化对容器大小做出响应：

```javascript
function setupResponsiveChart(containerId, data) {
  const container = document.getElementById(containerId);
  const svg = d3.select(`#${containerId}`).append('svg');

  function updateChart() {
    const { width, height } = container.getBoundingClientRect();
    svg.attr('width', width).attr('height', height);

    // 使用新尺寸重新绘制可视化
    drawChart(data, svg, width, height);
  }

  // 在初始加载时更新
  updateChart();

  // 在窗口调整大小时更新
  window.addEventListener('resize', updateChart);

  // 返回清理函数
  return () => window.removeEventListener('resize', updateChart);
}

// 用法：
// const cleanup = setupResponsiveChart('chart-container', myData);
// cleanup(); // 在组件卸载或元素移除时调用
```

或者使用 ResizeObserver 进行更直接的容器监控：

```javascript
function setupResponsiveChartWithObserver(svgElement, data) {
  const observer = new ResizeObserver(() => {
    const { width, height } = svgElement.getBoundingClientRect();
    d3.select(svgElement)
      .attr('width', width)
      .attr('height', height);

    // 重新绘制可视化
    drawChart(data, d3.select(svgElement), width, height);
  });

  observer.observe(svgElement.parentElement);
  return () => observer.disconnect();
}
```

## 常见可视化模式

### 条形图

```javascript
function drawBarChart(data, svgElement) {
  if (!data || data.length === 0) return;

  const svg = d3.select(svgElement);
  svg.selectAll("*").remove();

  const width = 800;
  const height = 400;
  const margin = { top: 20, right: 30, bottom: 40, left: 50 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const xScale = d3.scaleBand()
    .domain(data.map(d => d.category))
    .range([0, innerWidth])
    .padding(0.1);

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)])
    .range([innerHeight, 0]);

  g.append("g")
    .attr("transform", `translate(0,${innerHeight})`)
    .call(d3.axisBottom(xScale));

  g.append("g")
    .call(d3.axisLeft(yScale));

  g.selectAll("rect")
    .data(data)
    .join("rect")
    .attr("x", d => xScale(d.category))
    .attr("y", d => yScale(d.value))
    .attr("width", xScale.bandwidth())
    .attr("height", d => innerHeight - yScale(d.value))
    .attr("fill", "steelblue");
}

// 用法：
// drawBarChart(myData, document.getElementById('chart'));
```

### 折线图

```javascript
const line = d3.line()
  .x(d => xScale(d.date))
  .y(d => yScale(d.value))
  .curve(d3.curveMonotoneX); // 平滑曲线

g.append("path")
  .datum(data)
  .attr("fill", "none")
  .attr("stroke", "steelblue")
  .attr("stroke-width", 2)
  .attr("d", line);
```

### 散点图

```javascript
g.selectAll("circle")
  .data(data)
  .join("circle")
  .attr("cx", d => xScale(d.x))
  .attr("cy", d => yScale(d.y))
  .attr("r", d => sizeScale(d.size)) // 可选：大小编码
  .attr("fill", d => colourScale(d.category)) // 可选：颜色编码
  .attr("opacity", 0.7);
```

### 弦图

弦图在圆形布局中显示实体之间的关系，用带状表示它们之间的流动：

```javascript
function drawChordDiagram(data) {
  // 数据格式：包含源、目标和值的对象数组
  // 示例：[{ source: 'A', target: 'B', value: 10 }, ...]

  if (!data || data.length === 0) return;

  const svg = d3.select('#chart');
  svg.selectAll("*").remove();

  const width = 600;
  const height = 600;
  const innerRadius = Math.min(width, height) * 0.3;
  const outerRadius = innerRadius + 30;

  // 从数据创建矩阵
  const nodes = Array.from(new Set(data.flatMap(d => [d.source, d.target])));
  const matrix = Array.from({ length: nodes.length }, () => Array(nodes.length).fill(0));

  data.forEach(d => {
    const i = nodes.indexOf(d.source);
    const j = nodes.indexOf(d.target);
    matrix[i][j] += d.value;
    matrix[j][i] += d.value;
  });

  // 创建弦布局
  const chord = d3.chord()
    .padAngle(0.05)
    .sortSubgroups(d3.descending);

  const arc = d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(outerRadius);

  const ribbon = d3.ribbon()
    .source(d => d.source)
    .target(d => d.target);

  const colourScale = d3.scaleOrdinal(d3.schemeCategory10)
    .domain(nodes);

  const g = svg.append("g")
    .attr("transform", `translate(${width / 2},${height / 2})`);

  const chords = chord(matrix);

  // 绘制带状
  g.append("g")
    .attr("fill-opacity", 0.67)
    .selectAll("path")
    .data(chords)
    .join("path")
    .attr("d", ribbon)
    .attr("fill", d => colourScale(nodes[d.source.index]))
    .attr("stroke", d => d3.rgb(colourScale(nodes[d.source.index])).darker());

  // 绘制组（弧）
  const group = g.append("g")
    .selectAll("g")
    .data(chords.groups)
    .join("g");

  group.append("path")
    .attr("d", arc)
    .attr("fill", d => colourScale(nodes[d.index]))
    .attr("stroke", d => d3.rgb(colourScale(nodes[d.index])).darker());

  // 添加标签
  group.append("text")
    .each(d => { d.angle = (d.startAngle + d.endAngle) / 2; })
    .attr("dy", "0.31em")
    .attr("transform", d => `rotate(${(d.angle * 180 / Math.PI) - 90})translate(${outerRadius + 30})${d.angle > Math.PI ? "rotate(180)" : ""}`)
    .attr("text-anchor", d => d.angle > Math.PI ? "end" : null)
    .text((d, i) => nodes[i])
    .style("font-size", "12px");
}
```

### 热力图

热力图使用颜色在二维网格中编码值，用于显示类别间的模式：

```javascript
function drawHeatmap(data) {
  // 数据格式：包含行、列和值的对象数组
  // 示例：[{ row: 'A', column: 'X', value: 10 }, ...]

  if (!data || data.length === 0) return;

  const svg = d3.select('#chart');
  svg.selectAll("*").remove();

  const width = 800;
  const height = 600;
  const margin = { top: 100, right: 30, bottom: 30, left: 100 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  // 获取唯一行和列
  const rows = Array.from(new Set(data.map(d => d.row)));
  const columns = Array.from(new Set(data.map(d => d.column)));

  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // 创建比例尺
  const xScale = d3.scaleBand()
    .domain(columns)
    .range([0, innerWidth])
    .padding(0.01);

  const yScale = d3.scaleBand()
    .domain(rows)
    .range([0, innerHeight])
    .padding(0.01);

  // 值的颜色比例尺
  const colourScale = d3.scaleSequential(d3.interpolateYlOrRd)
    .domain([0, d3.max(data, d => d.value)]);

  // 绘制矩形
  g.selectAll("rect")
    .data(data)
    .join("rect")
    .attr("x", d => xScale(d.column))
    .attr("y", d => yScale(d.row))
    .attr("width", xScale.bandwidth())
    .attr("height", yScale.bandwidth())
    .attr("fill", d => colourScale(d.value));

  // 添加 x 轴标签
  svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .selectAll("text")
    .data(columns)
    .join("text")
    .attr("x", d => xScale(d) + xScale.bandwidth() / 2)
    .attr("y", -10)
    .attr("text-anchor", "middle")
    .text(d => d)
    .style("font-size", "12px");

  // 添加 y 轴标签
  svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .selectAll("text")
    .data(rows)
    .join("text")
    .attr("x", -10)
    .attr("y", d => yScale(d) + yScale.bandwidth() / 2)
    .attr("dy", "0.35em")
    .attr("text-anchor", "end")
    .text(d => d)
    .style("font-size", "12px");

  // 添加颜色图例
  const legendWidth = 20;
  const legendHeight = 200;
  const legend = svg.append("g")
    .attr("transform", `translate(${width - 60},${margin.top})`);

  const legendScale = d3.scaleLinear()
    .domain(colourScale.domain())
    .range([legendHeight, 0]);

  const legendAxis = d3.axisRight(legendScale)
    .ticks(5);

  // 在图例中绘制颜色渐变
  for (let i = 0; i < legendHeight; i++) {
    legend.append("rect")
      .attr("y", i)
      .attr("width", legendWidth)
      .attr("height", 1)
      .attr("fill", colourScale(legendScale.invert(i)));
  }

  legend.append("g")
    .attr("transform", `translate(${legendWidth},0)`)
    .call(legendAxis);
}
```

### 饼图

```javascript
const pie = d3.pie()
  .value(d => d.value)
  .sort(null);

const arc = d3.arc()
  .innerRadius(0)
  .outerRadius(Math.min(width, height) / 2 - 20);

const colourScale = d3.scaleOrdinal(d3.schemeCategory10);

const g = svg.append("g")
  .attr("transform", `translate(${width / 2},${height / 2})`);

g.selectAll("path")
  .data(pie(data))
  .join("path")
  .attr("d", arc)
  .attr("fill", (d, i) => colourScale(i))
  .attr("stroke", "white")
  .attr("stroke-width", 2);
```

### 力导向网络

```javascript
const simulation = d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links).id(d => d.id).distance(100))
  .force("charge", d3.forceManyBody().strength(-300))
  .force("center", d3.forceCenter(width / 2, height / 2));

const link = g.selectAll("line")
  .data(links)
  .join("line")
  .attr("stroke", "#999")
  .attr("stroke-width", 1);

const node = g.selectAll("circle")
  .data(nodes)
  .join("circle")
  .attr("r", 8)
  .attr("fill", "steelblue")
  .call(d3.drag()
    .on("start", dragstarted)
    .on("drag", dragged)
    .on("end", dragended));

simulation.on("tick", () => {
  link
    .attr("x1", d => d.source.x)
    .attr("y1", d => d.source.y)
    .attr("x2", d => d.target.x)
    .attr("y2", d => d.target.y);

  node
    .attr("cx", d => d.x)
    .attr("cy", d => d.y);
});

function dragstarted(event) {
  if (!event.active) simulation.alphaTarget(0.3).restart();
  event.subject.fx = event.subject.x;
  event.subject.fy = event.subject.y;
}

function dragged(event) {
  event.subject.fx = event.x;
  event.subject.fy = event.y;
}

function dragended(event) {
  if (!event.active) simulation.alphaTarget(0);
  event.subject.fx = null;
  event.subject.fy = null;
}
```

## 添加交互性

### 工具提示

```javascript
// 创建工具提示 div（在 SVG 外部）
const tooltip = d3.select("body").append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("visibility", "hidden")
  .style("background-color", "white")
  .style("border", "1px solid #ddd")
  .style("padding", "10px")
  .style("border-radius", "4px")
  .style("pointer-events", "none");

// 添加到元素
circles
  .on("mouseover", function(event, d) {
    d3.select(this).attr("opacity", 1);
    tooltip
      .style("visibility", "visible")
      .html(`<strong>${d.label}</strong><br/>值: ${d.value}`);
  })
  .on("mousemove", function(event) {
    tooltip
      .style("top", (event.pageY - 10) + "px")
      .style("left", (event.pageX + 10) + "px");
  })
  .on("mouseout", function() {
    d3.select(this).attr("opacity", 0.7);
    tooltip.style("visibility", "hidden");
  });
```

### 缩放和平移

```javascript
const zoom = d3.zoom()
  .scaleExtent([0.5, 10])
  .on("zoom", (event) => {
    g.attr("transform", event.transform);
  });

svg.call(zoom);
```

### 点击交互

```javascript
circles
  .on("click", function(event, d) {
    // 处理点击（分派事件、更新应用程序状态等）
    console.log("点击了:", d);

    // 视觉反馈
    d3.selectAll("circle").attr("fill", "steelblue");
    d3.select(this).attr("fill", "orange");

    // 可选：分派自定义事件供您的框架/应用程序监听
    // window.dispatchEvent(new CustomEvent('chartClick', { detail: d }));
  });
}
```

## 过渡和动画

为视觉变化添加平滑过渡：

```javascript
// 基本过渡
circles
  .transition()
  .duration(750)
  .attr("r", 10);

// 链式过渡
circles
  .transition()
  .duration(500)
  .attr("fill", "orange")
  .transition()
  .duration(500)
  .attr("r", 15);

// 错开过渡
circles
  .transition()
  .delay((d, i) => i * 50)
  .duration(500)
  .attr("cy", d => yScale(d.value));

// 自定义缓动
circles
  .transition()
  .duration(1000)
  .ease(d3.easeBounceOut)
  .attr("r", 10);
```

## 比例尺参考

### 定量比例尺

```javascript
// 线性比例尺
const xScale = d3.scaleLinear()
  .domain([0, 100])
  .range([0, 500]);

// 对数比例尺（用于指数数据）
const logScale = d3.scaleLog()
  .domain([1, 1000])
  .range([0, 500]);

// 幂比例尺
const powScale = d3.scalePow()
  .exponent(2)
  .domain([0, 100])
  .range([0, 500]);

// 时间比例尺
const timeScale = d3.scaleTime()
  .domain([new Date(2020, 0, 1), new Date(2024, 0, 1)])
  .range([0, 500]);
```

### 序数比例尺

```javascript
// 带比例尺（用于条形图）
const bandScale = d3.scaleBand()
  .domain(['A', 'B', 'C', 'D'])
  .range([0, 400])
  .padding(0.1);

// 点比例尺（用于折线/散点类别）
const pointScale = d3.scalePoint()
  .domain(['A', 'B', 'C', 'D'])
  .range([0, 400]);

// 序数比例尺（用于颜色）
const colourScale = d3.scaleOrdinal(d3.schemeCategory10);
```

### 序列比例尺

```javascript
// 序列颜色比例尺
const colourScale = d3.scaleSequential(d3.interpolateBlues)
  .domain([0, 100]);

// 发散颜色比例尺
const divScale = d3.scaleDiverging(d3.interpolateRdBu)
  .domain([-10, 0, 10]);
```

## 最佳实践

### 数据准备

始终在可视化之前验证和准备数据：

```javascript
// 过滤无效值
const cleanData = data.filter(d => d.value != null && !isNaN(d.value));

// 如果顺序很重要则排序数据
const sortedData = [...data].sort((a, b) => b.value - a.value);

// 解析日期
const parsedData = data.map(d => ({
  ...d,
  date: d3.timeParse("%Y-%m-%d")(d.date)
}));
```

### 性能优化

对于大型数据集（>1000 个元素）：

```javascript
// 使用画布而不是 SVG 用于许多元素
// 使用四叉树进行碰撞检测
// 使用 d3.line().curve(d3.curveStep) 简化路径
// 为大列表实现虚拟滚动
// 使用 requestAnimationFrame 进行自定义动画
```

### 无障碍性

使可视化具有无障碍性：

```javascript
// 添加 ARIA 标签
svg.attr("role", "img")
   .attr("aria-label", "显示季度收入的条形图");

// 添加标题和描述
svg.append("title").text("2024 年季度收入");
svg.append("desc").text("显示四个季度收入增长的条形图");

// 确保足够的颜色对比度
// 为交互元素提供键盘导航
// 包含数据表替代方案
```

### 样式

使用一致、专业的样式：

```javascript
// 提前定义颜色调色板
const colours = {
  primary: '#4A90E2',
  secondary: '#7B68EE',
  background: '#F5F7FA',
  text: '#333333',
  gridLines: '#E0E0E0'
};

// 应用一致的排版
svg.selectAll("text")
  .style("font-family", "Inter, sans-serif")
  .style("font-size", "12px");

// 使用微妙的网格线
g.selectAll(".tick line")
  .attr("stroke", colours.gridLines)
  .attr("stroke-dasharray", "2,2");
```

## 常见问题及解决方案

**问题**：轴不出现
- 确保比例尺具有有效域（检查 NaN 值）
- 验证轴附加到正确的组
- 检查变换翻译是否正确

**问题**：过渡不起作用
- 在属性更改之前调用 `.transition()`
- 确保元素具有唯一的键以进行适当的数据绑定
- 检查 useEffect 依赖项是否包含所有更改的数据

**问题**：响应式尺寸调整不起作用
- 使用 ResizeObserver 或窗口调整大小监听器
- 更新状态中的尺寸以触发重新渲染
- 确保 SVG 具有宽度/高度属性或 viewBox

**问题**：性能问题
- 限制 DOM 元素的数量（>1000 项时考虑画布）
- 防抖调整大小处理程序
- 使用 `.join()` 而不是单独的进入/更新/退出选择
- 通过检查依赖项避免不必要的重新渲染

## 资源

### references/
包含详细的参考资料：
- `d3-patterns.md` - 可视化模式和代码示例的综合集合
- `scale-reference.md` - d3 比例尺的完整指南及示例
- `colour-schemes.md` - D3 颜色调色板和推荐

### assets/

包含样板模板：

- `chart-template.js` - 基本图表的起始模板
- `interactive-template.js` - 带有工具提示、缩放和交互的模板
- `sample-data.json` - 用于测试的示例数据集

这些模板适用于原生 JavaScript、React、Vue、Svelte 或任何其他 JavaScript 环境。根据您的特定框架需要进行调整。

要使用这些资源，在需要特定可视化类型或模式的详细指导时，请阅读相关文件。