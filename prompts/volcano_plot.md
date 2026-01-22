# 火山图 - Volcano Plot

创建发表级火山图，用于展示差异表达分析结果。

## 功能特点

- 自动识别上调/下调基因并着色
- 可自定义阈值线 (log2FC和p值)
- 自动标注Top显著基因名
- 支持多组比较分面展示
- Nature/Science期刊风格配色

## 输入数据格式

```r
# 必需列
data.frame(
  gene = "基因名",
  log2FC = "log2 fold change",
  pvalue = "p值或FDR",
  regulation = "Up/Down/NS (可选)"
)
```

## 参数选项

| 参数 | 说明 | 默认值 |
|------|------|--------|
| fc_threshold | log2FC阈值 | 1 |
| p_threshold | p值阈值 | 0.05 |
| top_n | 标注的基因数 | 10 |
| point_size | 点大小 | 1 |
| colors | 配色方案 | Nature风格 |

## 示例代码

```r
create_volcano_plot <- function(data,
                                 fc_threshold = 1,
                                 p_threshold = 0.05,
                                 top_n = 10) {

  data <- data %>%
    mutate(
      neg_log10_p = -log10(pvalue),
      significance = case_when(
        log2FC > fc_threshold & pvalue < p_threshold ~ "Up",
        log2FC < -fc_threshold & pvalue < p_threshold ~ "Down",
        TRUE ~ "NS"
      )
    )

  ggplot(data, aes(x = log2FC, y = neg_log10_p)) +
    geom_point(aes(color = significance), alpha = 0.7, size = 1) +
    geom_vline(xintercept = c(-fc_threshold, fc_threshold),
               linetype = "dashed", color = "grey50") +
    geom_hline(yintercept = -log10(p_threshold),
               linetype = "dashed", color = "grey50") +
    scale_color_manual(values = c("Up" = "#E64B35", "Down" = "#4DBBD5", "NS" = "#CCCCCC")) +
    theme_bw() +
    labs(x = expression(log[2]~FC), y = expression(-log[10]~italic(p)))
}
```

请提供你的差异表达数据，我将帮你创建火山图。
