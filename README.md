# FULL_DOC_MARKDOWN 文件关系图召回 Mini Demo

这个目录是一份可分享给同事分析的最小完整实验包，用于验证：

- 文档级 embedding 召回
- 文件之间的语义关系图
- A* 风格图扩展召回
- top-k 召回评估
- 低召回样本分析
- 无模型 rerank 对照实验

## 目录内容

```text
minidemo/
  README.md
  test.ipynb
  evaluate_recall.jsonl
  FULL_DOC_MARKDOWN/
  models/
    Qwen3-Embedding-0.6B/
  graph_recall_cache/
    documents.jsonl
    document_embeddings.npy
    document_vectors.faiss
    relation_edges.jsonl
    embedding_fingerprint.json
    graph_fingerprint.json
    vector_store_fingerprint.json
    evaluate_recall_top5_detail.csv
    evaluate_recall_top5_summary.csv
    evaluate_recall_top10_detail.csv
    evaluate_recall_top10_summary.csv
    evaluate_recall_top20_detail.csv
    evaluate_recall_top20_summary.csv
    evaluate_recall_low100_analysis.csv
    evaluate_recall_rerank_from_top20_top5_detail.csv
    evaluate_recall_rerank_from_top20_top5_summary.csv
```

## 已包含内容

- 原始测试文档：`FULL_DOC_MARKDOWN/`
- 本地 embedding 模型：`models/Qwen3-Embedding-0.6B/`
- 实验 notebook：`test.ipynb`
- 评估样本：`evaluate_recall.jsonl`
- 已生成的 embedding、FAISS 向量库和文件关系图缓存
- top-5、top-10、top-20 召回结果
- 低召回样本分析
- 无模型 rerank 对比结果

## 核心缓存说明

- `documents.jsonl`：文档元数据，每行对应一个 Markdown 文件。
- `document_embeddings.npy`：文档 embedding 矩阵。
- `document_vectors.faiss`：FAISS 向量索引。
- `relation_edges.jsonl`：文件之间的无向语义关系边。
- `embedding_fingerprint.json`：embedding 缓存指纹，用于判断是否需要重算。
- `graph_fingerprint.json`：图关系缓存指纹。
- `vector_store_fingerprint.json`：FAISS 向量库缓存指纹。

## 当前实验结果概览

基于 `evaluate_recall.jsonl` 的 2764 个 query 测试：

```text
overall recall@5 约 0.907
overall mean_rank 约 1.43
```

含义：

- `recall@5`：目标文件是否出现在 top-5 结果中。
- `mean_rank`：命中的目标文件平均排第几名，越接近 1 越好。

当前结果说明：文件级 embedding + 图扩展召回效果较好，而且文件关系图可以作为文件之间的关联信息使用。

## 如何运行

建议在 `minidemo` 目录下启动 notebook：

```powershell
cd D:\nyonic\minidemo
```

然后打开：

```text
test.ipynb
```

Notebook 已经改成使用 `minidemo` 内部相对路径：

```python
ROOT_DIR = Path('FULL_DOC_MARKDOWN')
LOCAL_MODEL_PATH = Path('models/Qwen3-Embedding-0.6B')
CACHE_DIR = Path('graph_recall_cache')
```

因此在 `minidemo` 目录下打开 notebook 时，不需要再手动修改路径。

## 依赖

建议 Python 3.11。常用依赖：

```powershell
pip install numpy pandas scikit-learn tqdm faiss-cpu sentence-transformers transformers torch
```

如果只分析 CSV，不跑新 query，通常只需要：

```powershell
pip install pandas
```

## Kernel 一直 Connecting 怎么办

如果 VS Code/Jupyter 打开 `test.ipynb` 时一直停在 `Connecting to kernel Python 3.11.1`，优先按下面顺序处理：

1. 在 VS Code 右上角点击 kernel 名称，重新选择当前机器可用的 Python 解释器。
2. 建议选择系统 Python 3.11，或项目里的 `.venv` 解释器。
3. 确认解释器里安装了 `ipykernel`：

```powershell
python -m pip install ipykernel
```

4. 如果依赖不完整，再安装实验依赖：

```powershell
python -m pip install numpy pandas scikit-learn tqdm faiss-cpu sentence-transformers transformers torch
```

5. 在 VS Code 里执行 `Jupyter: Restart Kernel`，或者关闭 notebook 重新打开。
6. 如果还是卡住，重启 VS Code 的 Python/Jupyter 扩展宿主，最简单是重启 VS Code。

注意：kernel 成功连接前不会执行 notebook 代码，所以这类卡住通常不是 embedding 模型太大导致的，而是 Python kernel 环境或 VS Code/Jupyter 连接状态的问题。

## 注意事项

- 已包含本地模型，所以可以离线跑新 query。
- 已包含原始 `FULL_DOC_MARKDOWN`，所以也可以重新构建 embedding 和图。
- 如果只是看指标和结果，不需要重新跑 embedding。
- 重新跑全量 embedding 会占用 GPU 显存；当前 notebook 里有低显存参数。
- 无模型 rerank 已测试，当前没有提升，主要用于对照分析。

## 建议分析点

- 查看 `evaluate_recall_top5_summary.csv` 理解整体召回质量。
- 查看 `evaluate_recall_top5_detail.csv` 分析具体 query 的 top-5 命中情况。
- 查看 `evaluate_recall_low100_analysis.csv` 分析未命中和低排名问题。
- 查看 `relation_edges.jsonl` 分析文件之间的语义关联边。
- 对比 `evaluate_recall_rerank_from_top20_top5_summary.csv`，观察无模型 rerank 是否改善排序。
