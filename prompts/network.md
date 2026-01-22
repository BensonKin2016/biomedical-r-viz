# 蛋白互作网络分析 - Protein-Protein Interaction Network

创建蛋白互作网络可视化图表。

## 功能特点

- 基于共享通路/功能构建网络
- Hub基因识别与高亮
- 社区检测 (Louvain算法)
- 多种布局算法 (FR, KK, Circle等)
- 节点属性映射 (大小=连接度, 颜色=表达变化)

## 数据来源

1. **STRING数据库** - 蛋白互作
2. **共享KEGG通路** - 功能关联
3. **共表达网络** - 相关性分析
4. **自定义边列表**

## 输入数据格式

```r
# 边列表
edges <- data.frame(
  from = "蛋白A",
  to = "蛋白B",
  weight = "权重 (可选)"
)

# 节点属性
nodes <- data.frame(
  name = "蛋白名",
  log2FC = "表达变化",
  regulation = "Up/Down",
  degree = "连接度 (可选)"
)
```

## 示例代码

```r
library(igraph)
library(ggraph)
library(tidygraph)

create_network <- function(edges, nodes = NULL) {
  # 创建图对象
  g <- graph_from_data_frame(edges, directed = FALSE, vertices = nodes)

  # 计算节点属性
  V(g)$degree <- degree(g)
  V(g)$community <- membership(cluster_louvain(g))

  # 识别Hub基因
  hub_threshold <- quantile(V(g)$degree, 0.9)
  V(g)$is_hub <- V(g)$degree >= hub_threshold

  # 转换为tidygraph
  tg <- as_tbl_graph(g)

  # 绘图
  ggraph(tg, layout = "fr") +
    geom_edge_link(alpha = 0.3, color = "grey60") +
    geom_node_point(aes(size = degree, color = regulation), alpha = 0.8) +
    geom_node_text(data = . %>% filter(is_hub),
                   aes(label = name), repel = TRUE, size = 3) +
    scale_color_manual(values = c("Up" = "#E64B35", "Down" = "#4DBBD5")) +
    theme_void()
}
```

请提供你的互作数据或基因列表，我将帮你构建网络图。
