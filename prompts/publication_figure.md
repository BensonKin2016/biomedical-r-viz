# 发表级组合图 - Publication Quality Figure

创建符合顶级期刊要求的发表级组合图，打通投稿"最后一公里"。

## 支持的期刊模板

| 期刊 | 单栏宽度 | 双栏宽度 | 最大高度 | 字号要求 |
|------|----------|----------|----------|----------|
| Nature | 89 mm | 183 mm | 247 mm | 5-7 pt |
| Cell | 85 mm | 178 mm | 240 mm | 5-7 pt |
| Science | 90 mm | 180 mm | 230 mm | 6-8 pt |
| PNAS | 87 mm | 178 mm | 230 mm | 6-8 pt |
| eLife | 140 mm | 180 mm | 220 mm | 7-9 pt |

## 核心功能

### 1. 混合图形对象组合
支持将不同类型的R图形对象组合到同一张图：
- ggplot2 对象
- ComplexHeatmap 对象
- grid/grob 对象
- base R 图形

### 2. 智能布局系统
- 自定义网格布局
- 自动留白优化
- 跨行/跨列支持
- 嵌套布局

### 3. 专业标注工具
- Panel标签 (A, B, C...)
- 显著性标记 (*, **, ***)
- 箭头与括号
- 局部放大插图

### 4. 图例管理
- 共享图例
- 图例位置微调
- 合并重复图例

## 完整代码模板

```r
library(ggplot2)
library(patchwork)
library(cowplot)
library(grid)
library(gridExtra)
library(ComplexHeatmap)

# ============================================================
# 期刊模板配置
# ============================================================

JOURNAL_SPECS <- list(
  Nature = list(
    single_col = 89/25.4,   # inch
    double_col = 183/25.4,
    max_height = 247/25.4,
    base_size = 6,
    font_family = "Helvetica"
  ),
  Cell = list(
    single_col = 85/25.4,
    double_col = 178/25.4,
    max_height = 240/25.4,
    base_size = 6,
    font_family = "Arial"
  ),
  Science = list(
    single_col = 90/25.4,
    double_col = 180/25.4,
    max_height = 230/25.4,
    base_size = 7,
    font_family = "Helvetica"
  ),
  PNAS = list(
    single_col = 87/25.4,
    double_col = 178/25.4,
    max_height = 230/25.4,
    base_size = 7,
    font_family = "Helvetica"
  ),
  Default = list(
    single_col = 3.5,
    double_col = 7.25,
    max_height = 9.5,
    base_size = 8,
    font_family = "sans"
  )
)

# ============================================================
# 统一发表级主题
# ============================================================

theme_publication <- function(journal = "Nature", base_size = NULL) {
  spec <- JOURNAL_SPECS[[journal]] %||% JOURNAL_SPECS$Default
  size <- base_size %||% spec$base_size

  theme_bw(base_size = size, base_family = spec$font_family) +
    theme(
      # 标题
      plot.title = element_blank(),
      plot.subtitle = element_blank(),

      # 坐标轴
      axis.title = element_text(size = size, face = "bold"),
      axis.text = element_text(size = size - 1, color = "black"),
      axis.line = element_line(color = "black", linewidth = 0.5),
      axis.ticks = element_line(color = "black", linewidth = 0.3),

      # 面板
      panel.background = element_blank(),
      panel.border = element_blank(),
      panel.grid = element_blank(),

      # 图例
      legend.title = element_text(size = size - 1, face = "bold"),
      legend.text = element_text(size = size - 2),
      legend.key.size = unit(3, "mm"),
      legend.background = element_blank(),

      # 边距
      plot.margin = margin(2, 2, 2, 2, "mm")
    )
}

# ============================================================
# Panel标签添加函数
# ============================================================

add_panel_label <- function(plot, label, x = -0.05, y = 1.05,
                            size = 10, fontface = "bold") {
  # 对于ggplot对象
  if (inherits(plot, "gg")) {
    plot +
      coord_cartesian(clip = "off") +
      annotation_custom(
        grob = textGrob(label, x = unit(x, "npc"), y = unit(y, "npc"),
                       gp = gpar(fontsize = size, fontface = fontface))
      )
  } else {
    # 返回原对象，标签在组合时添加
    plot
  }
}

# ============================================================
# ComplexHeatmap转换为grob
# ============================================================

heatmap_to_grob <- function(heatmap_obj) {
  # 将ComplexHeatmap对象转换为grid grob以便与ggplot组合
  grob <- grid.grabExpr(draw(heatmap_obj), wrap = TRUE)
  return(grob)
}

# ============================================================
# 混合图形组合函数
# ============================================================

combine_mixed_plots <- function(plot_list, layout,
                                 labels = LETTERS[1:length(plot_list)],
                                 label_size = 10,
                                 journal = "Nature") {

  spec <- JOURNAL_SPECS[[journal]] %||% JOURNAL_SPECS$Default

  # 转换所有图形为grob
  grob_list <- lapply(seq_along(plot_list), function(i) {
    p <- plot_list[[i]]

    if (inherits(p, "Heatmap") || inherits(p, "HeatmapList")) {
      # ComplexHeatmap对象
      grob <- heatmap_to_grob(p)
    } else if (inherits(p, "gg")) {
      # ggplot对象 - 应用发表主题
      p <- p + theme_publication(journal)
      grob <- ggplotGrob(p)
    } else if (inherits(p, "grob") || inherits(p, "gTree")) {
      # 已经是grob
      grob <- p
    } else {
      # base R图形 - 需要特殊处理
      grob <- nullGrob()
      warning(paste("Plot", i, "type not recognized"))
    }

    return(grob)
  })

  # 解析布局字符串
  if (is.character(layout)) {
    layout_matrix <- parse_layout(layout)
  } else {
    layout_matrix <- layout
  }

  # 使用gridExtra组合
  combined <- arrangeGrob(
    grobs = grob_list,
    layout_matrix = layout_matrix
  )

  # 添加Panel标签
  combined <- add_labels_to_grob(combined, labels, label_size)

  return(combined)
}

# ============================================================
# 布局解析函数
# ============================================================

parse_layout <- function(layout_string) {
  # 将patchwork风格的布局字符串转换为矩阵
  lines <- strsplit(trimws(layout_string), "\n")[[1]]
  lines <- lines[lines != ""]

  matrix(
    unlist(lapply(lines, function(x) strsplit(gsub(" ", "", x), "")[[1]])),
    nrow = length(lines),
    byrow = TRUE
  )
}

# ============================================================
# 显著性标注函数
# ============================================================

add_significance <- function(plot, comparisons, y_position = NULL,
                              tip_length = 0.02,
                              label = c("p.signif", "p.format")) {
  # 需要 ggpubr 包
  if (!requireNamespace("ggpubr", quietly = TRUE)) {
    warning("请安装 ggpubr 包: install.packages('ggpubr')")
    return(plot)
  }

  plot +
    ggpubr::stat_compare_means(
      comparisons = comparisons,
      method = "wilcox.test",
      label = label[1],
      tip.length = tip_length,
      bracket.size = 0.3,
      size = 2.5,
      vjust = 0.5
    )
}

# ============================================================
# 局部放大插图函数
# ============================================================

add_inset <- function(main_plot, inset_plot,
                       x = 0.7, y = 0.7,
                       width = 0.3, height = 0.3) {

  main_plot +
    annotation_custom(
      grob = ggplotGrob(inset_plot),
      xmin = x - width/2, xmax = x + width/2,
      ymin = y - height/2, ymax = y + height/2
    )
}

# ============================================================
# 共享图例提取与添加
# ============================================================

extract_legend <- function(plot) {
  tmp <- ggplotGrob(plot)
  leg <- which(sapply(tmp$grobs, function(x) x$name) == "guide-box")
  if (length(leg) > 0) {
    return(tmp$grobs[[leg]])
  }
  return(NULL)
}

add_shared_legend <- function(combined_grob, legend,
                               position = "right",
                               legend_width = 0.15) {

  if (position == "right") {
    grid.arrange(
      combined_grob, legend,
      ncol = 2,
      widths = c(1 - legend_width, legend_width)
    )
  } else if (position == "bottom") {
    grid.arrange(
      combined_grob, legend,
      nrow = 2,
      heights = c(1 - legend_width, legend_width)
    )
  }
}

# ============================================================
# 主函数：创建发表级组合图
# ============================================================

create_publication_figure <- function(
    panels,                          # 图形对象列表
    layout = NULL,                   # 布局 ("AB\nCD" 或矩阵)
    labels = "AUTO",                 # Panel标签
    journal = "Nature",              # 期刊模板
    width = "double",                # "single" 或 "double" 或具体数值
    height = NULL,                   # 高度 (NULL=自动)
    shared_legend = FALSE,           # 是否使用共享图例
    legend_from = 1,                 # 从哪个panel提取共享图例
    output_prefix = "Figure",        # 输出文件名前缀
    formats = c("pdf", "png", "tiff") # 输出格式
) {

  spec <- JOURNAL_SPECS[[journal]] %||% JOURNAL_SPECS$Default

  # 确定尺寸
  if (is.character(width)) {
    fig_width <- if (width == "single") spec$single_col else spec$double_col
  } else {
    fig_width <- width
  }

  fig_height <- height %||% (fig_width * 0.8)
  fig_height <- min(fig_height, spec$max_height)

  # 自动标签
  if (identical(labels, "AUTO")) {
    labels <- LETTERS[1:length(panels)]
  }

  # 默认布局
  if (is.null(layout)) {
    n <- length(panels)
    ncol <- ceiling(sqrt(n))
    layout <- matrix(1:n, ncol = ncol, byrow = TRUE)
  }

  # 判断是否包含非ggplot对象
  has_non_ggplot <- any(sapply(panels, function(p) {
    inherits(p, "Heatmap") || inherits(p, "HeatmapList") ||
    inherits(p, "grob") || inherits(p, "recordedplot")
  }))

  if (has_non_ggplot) {
    # 使用混合组合方法
    combined <- combine_mixed_plots(panels, layout, labels,
                                     label_size = spec$base_size + 4,
                                     journal = journal)

    # 保存
    for (fmt in formats) {
      filename <- paste0(output_prefix, ".", fmt)
      if (fmt == "pdf") {
        pdf(filename, width = fig_width, height = fig_height)
        grid.draw(combined)
        dev.off()
      } else if (fmt == "png") {
        png(filename, width = fig_width, height = fig_height,
            units = "in", res = 300)
        grid.draw(combined)
        dev.off()
      } else if (fmt == "tiff") {
        tiff(filename, width = fig_width, height = fig_height,
             units = "in", res = 600, compression = "lzw")
        grid.draw(combined)
        dev.off()
      }
    }

  } else {
    # 纯ggplot对象 - 使用patchwork

    # 应用发表主题并添加标签
    panels_themed <- lapply(seq_along(panels), function(i) {
      panels[[i]] +
        theme_publication(journal) +
        labs(tag = labels[i]) +
        theme(plot.tag = element_text(size = spec$base_size + 4,
                                       face = "bold"))
    })

    # 处理共享图例
    if (shared_legend) {
      shared_leg <- extract_legend(panels_themed[[legend_from]])
      panels_themed <- lapply(panels_themed, function(p) {
        p + theme(legend.position = "none")
      })
    }

    # 组合
    if (is.character(layout)) {
      combined <- wrap_plots(panels_themed, design = layout)
    } else {
      combined <- wrap_plots(panels_themed,
                             ncol = ncol(layout),
                             nrow = nrow(layout))
    }

    # 添加共享图例
    if (shared_legend && !is.null(shared_leg)) {
      combined <- combined +
        inset_element(shared_leg,
                      left = 0.85, right = 1,
                      bottom = 0.3, top = 0.7)
    }

    # 保存
    for (fmt in formats) {
      filename <- paste0(output_prefix, ".", fmt)
      if (fmt == "pdf") {
        ggsave(filename, combined, width = fig_width, height = fig_height,
               device = cairo_pdf)
      } else if (fmt == "png") {
        ggsave(filename, combined, width = fig_width, height = fig_height,
               dpi = 300)
      } else if (fmt == "tiff") {
        ggsave(filename, combined, width = fig_width, height = fig_height,
               dpi = 600, compression = "lzw")
      }
    }
  }

  message(sprintf("Figure saved: %s x %s inches (%s format)",
                  round(fig_width, 2), round(fig_height, 2), journal))
  message(sprintf("Output files: %s",
                  paste(paste0(output_prefix, ".", formats), collapse = ", ")))

  return(invisible(combined))
}

# ============================================================
# 便捷函数：快速生成常见布局
# ============================================================

# 2x2 标准布局
layout_2x2 <- "
AB
CD
"

# 1大3小布局 (大图在左)
layout_1big_3small <- "
AAB
AAC
AAD
"

# 上下布局 (上大下小)
layout_top_bottom <- "
AAA
BCD
"

# 金字塔布局
layout_pyramid <- "
#A#
BCD
"

# ============================================================
# 使用示例
# ============================================================

# 示例1: 基础ggplot组合
# panels <- list(p_volcano, p_heatmap, p_enrichment, p_network)
# create_publication_figure(
#   panels,
#   layout = layout_2x2,
#   journal = "Nature",
#   output_prefix = "Figure1"
# )

# 示例2: 混合ComplexHeatmap
# panels <- list(p_volcano, heatmap_obj, p_enrichment, p_network)
# create_publication_figure(
#   panels,
#   layout = layout_1big_3small,
#   journal = "Cell",
#   output_prefix = "Figure2"
# )

# 示例3: 自定义布局
# custom_layout <- "
#   AABB
#   CCDD
#   EEEE
# "
# create_publication_figure(
#   panels,
#   layout = custom_layout,
#   journal = "Science",
#   shared_legend = TRUE,
#   output_prefix = "Figure3"
# )
```

## 快速使用指南

### 基础用法
```r
# 准备好各个Panel (ggplot对象)
panels <- list(p_volcano, p_heatmap, p_enrichment, p_network)

# 一键生成Nature格式组合图
create_publication_figure(panels, journal = "Nature")
```

### 自定义布局
```r
# 左侧大图 + 右侧3个小图
layout <- "
  AAB
  AAC
  AAD
"
create_publication_figure(panels, layout = layout)
```

### 混合ComplexHeatmap
```r
# 火山图(ggplot) + 热图(ComplexHeatmap) + 其他
panels <- list(volcano_gg, heatmap_complex, enrichment_gg, network_gg)
create_publication_figure(panels, layout = "AB\nCD")
```

请告诉我你的具体需求：
1. 需要组合哪些类型的图？
2. 目标期刊是哪个？
3. 期望的布局方式？
