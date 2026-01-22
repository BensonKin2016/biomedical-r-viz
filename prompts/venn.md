# 韦恩图/UpSet图 - Venn & UpSet Diagram

创建集合交集可视化图表。

## 支持的图表类型

1. **韦恩图** (2-4组) - 经典圆形/椭圆交集图
2. **UpSet图** (5组以上) - 矩阵式交集展示
3. **花瓣图** - 多组美观展示

## 输入数据格式

```r
# 命名列表，每个元素是一组基因/蛋白
gene_lists <- list(
  "Group_A" = c("gene1", "gene2", "gene3", ...),
  "Group_B" = c("gene2", "gene3", "gene4", ...),
  "Group_C" = c("gene1", "gene3", "gene5", ...)
)
```

## 功能特点

- 自动计算交集
- 显示交集基因数
- 导出交集基因列表
- 自定义配色
- 添加百分比标签

## 示例代码

```r
library(ggVennDiagram)
library(VennDiagram)

# ggVennDiagram版本 (推荐, 更美观)
create_venn <- function(gene_lists, title = "Gene Overlap") {

  p <- ggVennDiagram(gene_lists,
                     label = "count",
                     label_alpha = 0) +
    scale_fill_gradient(low = "#F4FAFE", high = "#4981BF") +
    labs(title = title) +
    theme(plot.title = element_text(hjust = 0.5, face = "bold"))

  return(p)
}

# 导出交集基因
export_intersections <- function(gene_lists) {
  all_genes <- unique(unlist(gene_lists))
  membership <- sapply(gene_lists, function(x) all_genes %in% x)

  result <- data.frame(
    Gene = all_genes,
    membership,
    Intersection = apply(membership, 1, function(x) paste(names(gene_lists)[x], collapse = " & "))
  )

  return(result)
}
```

## UpSet图 (适合5组以上)

```r
library(UpSetR)

create_upset <- function(gene_lists) {
  # 转换为二进制矩阵
  all_genes <- unique(unlist(gene_lists))
  mat <- sapply(gene_lists, function(x) as.integer(all_genes %in% x))
  rownames(mat) <- all_genes

  # 绘制UpSet图
  upset(as.data.frame(mat),
        sets = names(gene_lists),
        order.by = "freq",
        mb.ratio = c(0.6, 0.4))
}
```

请提供你的基因列表，我将帮你创建韦恩图。
