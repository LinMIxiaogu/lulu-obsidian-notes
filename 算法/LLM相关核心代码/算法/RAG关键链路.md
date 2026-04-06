# RAG关键链路

如果你面试的是应用开发岗，检索增强生成（RAG）几乎必测。

## 核心考点

- **高频题：** 手写余弦相似度（Cosine Similarity）计算函数。
- **高频题：** 实现简单的滑动窗口切分文本（Text Splitter）算法。
- **高频题：** 手写倒排索引的简化实现。
- **建议：** 重点掌握相似度计算公式。

$$\text{similarity} = \frac{A \cdot B}{\|A\| \|B\|}$$

## Cosine Similarity

**背诵要点：** 向量点积除以两个向量模长的乘积。

```python
def cosine_similarity(vec_a, vec_b):
    # vec_a, vec_b 形状为 (n_features,)
    dot_product = torch.dot(vec_a, vec_b)
    norm_a = torch.norm(vec_a)
    norm_b = torch.norm(vec_b)

    # 防止除零
    return dot_product / (norm_a * norm_b + 1e-8)

# 扩展：如果是矩阵计算（用于 RAG 检索）
# similarity = F.cosine_similarity(query_embedding, document_embeddings)
```
