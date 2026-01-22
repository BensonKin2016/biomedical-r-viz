# 发表级组合图 - Publication Quality Figure

创建符合期刊要求的A4发表级组合图。

## 设计规范

### 尺寸标准
- **单栏**: 3.5 inch (89 mm)
- **1.5栏**: 5 inch (127 mm)
- **双栏/全页**: 7.25 inch (184 mm)
- **A4可用区域**: 约6.27 × 9.69 inch

### 字号标准
- Panel标签 (A, B, C): 9-12 pt, 粗体
- 坐标轴标题: 8-10 pt
- 坐标轴刻度: 7-9 pt
- 图例: 7-8 pt

### 分辨率
- PDF: 矢量格式 (推荐投稿)
- PNG: 300 dpi (网页/PPT)
- TIFF: 600 dpi (印刷出版)

## 输入要求

```r
# 准备好各个Panel的ggplot对象
panel_a <- create_volcano_plot(...)
panel_b <- create_heatmap(...)
panel_c <- create_enrichment_plot(...)
panel_d <- create_network(...)
```

## 示例代码

```r
library(patchwork)
library(cowplot)

# 定义A4参数
A4_WIDTH <- 6.27  # inch
A4_HEIGHT <- 9.69 # inch

# 定义统一主题
THEME_PUB <- theme_bw(base_size = 8) +
  theme(
    plot.title = element_blank(),
    axis.title = element_text(size = 8),
    axis.text = element_text(size = 7, color = "black"),
    legend.title = element_text(size = 7, face = "bold"),
    legend.text = element_text(size = 6),
    panel.grid.minor = element_blank(),
    plot.margin = margin(2, 2, 2, 2)
  )

# 添加Panel标签
add_label <- function(p, label) {
  p + plot_annotation(
    title = label,
    theme = theme(
      plot.title = element_text(size = 10, face = "bold", hjust = 0)
    )
  )
}

# 组合图
create_combined_figure <- function(panels, layout = NULL) {

  # 添加标签
  labeled_panels <- lapply(seq_along(panels), function(i) {
    add_label(panels[[i]], LETTERS[i])
  })

  # 使用patchwork组合
  if(is.null(layout)) {
    # 默认2列布局
    combined <- wrap_plots(labeled_panels, ncol = 2)
  } else {
    # 自定义布局
    combined <- wrap_plots(labeled_panels, design = layout)
  }

  # 保存
  ggsave("Figure_combined.pdf", combined,
         width = A4_WIDTH, height = A4_HEIGHT * 0.7)
  ggsave("Figure_combined.png", combined,
         width = A4_WIDTH, height = A4_HEIGHT * 0.7, dpi = 300)
  ggsave("Figure_combined.tiff", combined,
         width = A4_WIDTH, height = A4_HEIGHT * 0.7,
         dpi = 600, compression = "lzw")

  return(combined)
}

# 使用示例
layout <- "
  AAB
  CCD
  EEE
"
fig <- create_combined_figure(panels, layout)
```

## 配色方案 (Nature风格)

```r
COLORS <- list(
  groups = c("#00A087", "#E64B35", "#3C5488", "#F39B7F"),
  regulation = c("Up" = "#E64B35", "Down" = "#4DBBD5"),
  heatmap = c("#3C5488", "white", "#E64B35"),
  gradient = c("#DEEBF7", "#3182BD")
)
```

请提供你的Panel列表和布局要求，我将帮你创建发表级组合图。
