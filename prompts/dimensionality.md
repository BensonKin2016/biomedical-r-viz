# PCA/UMAP降维分析 - Dimensionality Reduction

创建主成分分析和UMAP降维可视化。

## 支持的方法

1. **PCA** - 主成分分析 (线性降维)
2. **UMAP** - 统一流形近似与投影 (非线性)
3. **t-SNE** - t分布随机邻域嵌入 (非线性)

## 功能特点

- 样本分组着色
- 95%置信椭圆
- 方差解释比例标注
- 支持3D可视化
- 批次效应检测

## 输入数据格式

```r
# 表达矩阵 (行=基因/位点, 列=样本)
expr_matrix <- matrix(...)

# 样本信息
sample_info <- data.frame(
  Sample = colnames(expr_matrix),
  Group = c("Control", "Treatment", ...),
  Batch = c("A", "B", ...)
)
```

## 示例代码

```r
library(FactoMineR)
library(factoextra)
library(umap)

# PCA分析
create_pca <- function(expr_matrix, sample_info, color_by = "Group") {

  # 转置: 行=样本, 列=基因
  mat_t <- t(expr_matrix)

  # PCA
  pca_result <- PCA(mat_t, scale.unit = TRUE, ncp = 5, graph = FALSE)

  # 提取坐标
  pca_coords <- as.data.frame(pca_result$ind$coord)
  pca_coords$Sample <- rownames(pca_coords)
  pca_coords <- merge(pca_coords, sample_info, by = "Sample")

  # 方差解释
  var_exp <- pca_result$eig[, 2]

  # 绑图
  ggplot(pca_coords, aes(x = Dim.1, y = Dim.2, color = .data[[color_by]])) +
    stat_ellipse(level = 0.95, linetype = "dashed") +
    geom_point(size = 3, alpha = 0.8) +
    labs(x = paste0("PC1 (", round(var_exp[1], 1), "%)"),
         y = paste0("PC2 (", round(var_exp[2], 1), "%)")) +
    theme_bw()
}

# UMAP分析
create_umap <- function(expr_matrix, sample_info, color_by = "Group") {

  mat_t <- t(expr_matrix)

  # UMAP配置
  config <- umap.defaults
  config$n_neighbors <- min(15, nrow(mat_t) - 1)

  set.seed(42)
  umap_result <- umap(mat_t, config = config)

  # 提取坐标
  umap_coords <- as.data.frame(umap_result$layout)
  colnames(umap_coords) <- c("UMAP1", "UMAP2")
  umap_coords$Sample <- rownames(mat_t)
  umap_coords <- merge(umap_coords, sample_info, by = "Sample")

  # 绘图
  ggplot(umap_coords, aes(x = UMAP1, y = UMAP2, color = .data[[color_by]])) +
    stat_ellipse(level = 0.95, linetype = "dashed") +
    geom_point(size = 3, alpha = 0.8) +
    theme_bw()
}
```

请提供你的表达矩阵和样本信息，我将帮你进行降维分析。
