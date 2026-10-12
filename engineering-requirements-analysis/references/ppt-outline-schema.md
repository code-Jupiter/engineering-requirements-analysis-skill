# PPT 大纲输出结构

仅在演示思路获明确确认后使用。输出是 JSON，不是 PPTX 文件。标签内不得包含注释、尾逗号或额外说明。字段结构如下（示例文字应替换为材料支持的内容）：

```text
[PPT_OUTLINE]
{
  "ppt_outline": {
    "cover": {
      "title": "主标题",
      "sub_title": "副标题",
      "content": []
    },
    "table_of_contents": {
      "title": "目录",
      "content": ["第一部分"]
    },
    "parts": [
      {
        "part_title": "第一部分",
        "pages": [
          {"title": "页面标题", "content": ["材料支持的观点"]}
        ]
      }
    ],
    "end_page": {
      "title": "总结与展望",
      "content": []
    }
  }
}
[/PPT_OUTLINE]
```

标题字段为字符串，所有 content 为字符串数组，parts 和 pages 为数组。检查目录与章节对应、逻辑顺序、观点依据；实际页数为封面 1 + 目录 1 + 所有 pages 数量 + 尾页 1，与确认页数保持一致。若用户要求无封面/目录或其他结构，先确认格式变更，不默默违反固定结构。
