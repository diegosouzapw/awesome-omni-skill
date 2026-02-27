---
name: arch-vis
description: Draw architecture diagrams on Feishu whiteboards via API. Use when user asks to create system architecture diagrams, flowcharts, module relationship diagrams, or visualize code structure on Feishu/Lark whiteboards. Supports shapes, connectors, mind maps, and iterative visual refinement by downloading and viewing results.
---

# Feishu Whiteboard Diagram Skill

Draw architecture diagrams on Feishu whiteboards using the `arch_vis` package.

## Quick Start

```bash
# 创建节点（从 JSON 文件）
uv run python entry.py nodes.json -o /tmp/board.png

# 仅下载当前画板
uv run python entry.py --download -o /tmp/board.png

# 从 stdin 读取 JSON
echo '[...]' | uv run python entry.py - -o /tmp/board.png
```

JSON 文件格式（数组或 `{"nodes": [...]}`）：
```json
[
  {"id": "api", "type": "composite_shape", "x": 100, "y": 100, ...},
  {"id": "c1", "type": "connector", ...}
]
```

## Node Structure

### Shape
```python
{"id": "api", "type": "composite_shape", "x": 100, "y": 100, "width": 140, "height": 55,
 "text": {"text": "API Gateway", "horizontal_align": "center", "vertical_align": "mid"},
 "style": {"theme_fill_color_code": 3, "theme_border_color_code": 6, "fill_color_type": 0},
 "composite_shape": {"type": "round_rect"}}
```

Shape types: `rect`, `round_rect`, `ellipse`, `diamond`, `cylinder`, `cloud`

### Connector
```python
{"id": "c1", "type": "connector", "connector": {
    "start": {"attached_object": {"id": "api", "snap_to": "bottom"}, "arrow_style": "none"},
    "end": {"attached_object": {"id": "db", "snap_to": "top"}, "arrow_style": "triangle_arrow"},
    "shape": "straight"}}
```

### Mind Map
```python
# Root
{"id": "mm:root", "type": "mind_map", "x": 400, "y": 300, "text": {"text": "Topic"},
 "mind_map_root": {"layout": "left_right", "type": "mind_map_round_rect", "line_style": "curve"}}

# Child
{"id": "mm:c1", "type": "mind_map", "text": {"text": "Subtopic"},
 "mind_map_node": {"parent_id": "mm:root", "type": "mind_map_text", "layout_position": "right"}}
```

## Colors

| Purpose | fill_code | border_code |
|---------|-----------|-------------|
| Entry/API | 3 | 6 |
| Services | 5 | 8 |
| Core/Alert | 6 | 9 |
| Database | 2 | 5 |
| Start/End | 17 | 11 |

## Workflow

1. Create nodes (all shapes + connectors in one batch)
2. Download image to `/tmp/board.png`
3. Use Read tool to view result
4. Iterate if needed

## References

- **API 文档**: [references/](references/) 目录包含完整的飞书画板 API 文档

## Common Issues

- **403 (code 2890005)**: Share whiteboard with app ("can edit")
- **Color not working**: Use `theme_fill_color_code` (int), not hex
- **Connector error**: Nodes and connectors must be in same batch
