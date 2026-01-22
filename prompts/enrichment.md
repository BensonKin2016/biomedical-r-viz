# GO/KEGG富集分析可视化 - Enrichment Analysis

创建GO和KEGG富集分析可视化图表。

## 支持的图表类型

1. **条形图** - 显示富集条目和-log10(p)
2. **气泡图** - 颜色=显著性, 大小=基因数, x轴=Fold Enrichment
3. **点阵图** - 多组比较矩阵
4. **弦图** - 基因-通路关联
5. **KEGG通路图** - 差异基因着色

## 输入数据格式

```r
# 富集分析结果
enrich_data <- data.frame(
  Term = "GO/KEGG条目名称",
  Category = "BP/CC/MF 或 pathway",
  PValue = "p值",
  FoldEnrichment = "富集倍数",
  Count = "基因数",
  Genes = "基因列表 (可选)"
)
```

## 示例代码

```r
# GO条形图
create_go_barplot <- function(data, top_n = 15) {
  data %>%
    arrange(PValue) %>%
    head(top_n) %>%
    mutate(Term = fct_reorder(Term, -log10(PValue))) %>%
    ggplot(aes(x = -log10(PValue), y = Term, fill = Category)) +
    geom_col(alpha = 0.85) +
    scale_fill_manual(values = c("BP" = "#7570B3", "CC" = "#1B9E77", "MF" = "#D95F02")) +
    theme_bw() +
    labs(x = expression(-log[10]~italic(p)), y = NULL)
}

# KEGG气泡图
create_kegg_dotplot <- function(data, top_n = 15) {
  data %>%
    arrange(PValue) %>%
    head(top_n) %>%
    ggplot(aes(x = FoldEnrichment, y = reorder(Term, FoldEnrichment))) +
    geom_point(aes(size = Count, color = -log10(PValue))) +
    scale_color_gradient(low = "#DEEBF7", high = "#08519C") +
    scale_size_continuous(range = c(3, 8)) +
    theme_bw() +
    labs(x = "Fold Enrichment", y = NULL)
}
```

请提供你的富集分析结果，我将帮你创建可视化图表。
