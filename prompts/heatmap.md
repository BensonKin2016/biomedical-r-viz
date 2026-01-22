# 增强版热图 - Enhanced Heatmap

使用ComplexHeatmap创建带注释的聚类热图。

## 功能特点

- 行/列聚类与树状图
- 多层注释 (分组、调控方向、基因类别等)
- Z-score标准化
- 自定义配色方案
- 分割热图 (按类别)

## 输入数据格式

```r
# 表达矩阵 (行=基因, 列=样本)
expr_matrix <- matrix(...)
rownames(expr_matrix) <- gene_names
colnames(expr_matrix) <- sample_names

# 列注释 (样本信息)
col_annotation <- data.frame(
  Group = c("Control", "Treatment", ...),
  Batch = c("A", "B", ...),
  row.names = sample_names
)

# 行注释 (基因信息)
row_annotation <- data.frame(
  Regulation = c("Up", "Down", ...),
  Pathway = c("Metabolism", "Signaling", ...),
  row.names = gene_names
)
```

## 示例代码

```r
library(ComplexHeatmap)
library(circlize)

create_heatmap <- function(expr_matrix,
                            col_annotation = NULL,
                            row_annotation = NULL,
                            cluster_rows = TRUE,
                            cluster_cols = TRUE) {

  # Z-score标准化
  mat_scaled <- t(scale(t(expr_matrix)))
  mat_scaled[mat_scaled > 2] <- 2
  mat_scaled[mat_scaled < -2] <- -2

  # 配色
  col_fun <- colorRamp2(c(-2, 0, 2), c("#3C5488", "white", "#E64B35"))

  # 列注释
  if(!is.null(col_annotation)) {
    top_ha <- HeatmapAnnotation(df = col_annotation)
  }

  # 行注释
  if(!is.null(row_annotation)) {
    left_ha <- rowAnnotation(df = row_annotation)
  }

  # 绑制热图
  Heatmap(mat_scaled,
          col = col_fun,
          cluster_rows = cluster_rows,
          cluster_columns = cluster_cols,
          top_annotation = top_ha,
          left_annotation = left_ha,
          show_row_names = nrow(mat_scaled) < 50)
}
```

请提供你的表达矩阵数据，我将帮你创建热图。
