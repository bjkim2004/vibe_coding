
# 🐉 dragonKue cosine-idf + ChromaDB + KoSimCSE 토크나이저 적용 가이드

이 문서는 기존 dragonKue cosine-idf RAG 구조에서  
**Konlpy 제거 → `BM-K/KoSimCSE-roberta-multitask` 토크나이저 기반으로 전체 파이프라인을 재구성한 공식 버전**입니다.

Dense Vector 기반 ChromaDB + KoSimCSE Embedding + KoSimCSE Tokenizer + IDF → dragonKue cosine-idf Rerank 구조를 완전히 정리합니다.

---

# 🔥 전체 구조 개요

ChromaDB는 Dense Vector 전용이므로 BM25처럼 Sparse 검색이 불가능하다.  
따라서 dragonKue cosine-idf는 다음 구조로 적용한다:

```
[Chroma Dense Search]
        ↓
[KoSimCSE Token → IDF Weight → dragonKue Rerank]
        ↓
[최종 top-k 문서]
```

여기서 핵심 변화는:

- **konlpy 형태소 분석기를 제거**
- **KoSimCSE 토크나이저를 그대로 DF/IDF 계산과 dragonKue에 사용**
- 같은 모델·토크나이저 조합으로 일관된 의미-토큰 매칭 보장

---


# 📌 1. KoSimCSE 토크나이저 준비

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("BM-K/KoSimCSE-roberta-multitask")

def tokenize(text: str):
    # KoSimCSE subword 토큰 리스트
    return tokenizer.tokenize(text)
```

---

# 📌 2. 전역 DF/IDF 계산 (KoSimCSE tokenizer 기반)

```python
from collections import Counter
from math import log
import json

docs = [...]   # 전체 말뭉치 (문장/문서 리스트)
N = len(docs)

df_counter = Counter()
doc_tokens_list = []

for text in docs:
    tokens = list(set(tokenize(text)))  # KoSimCSE subword 기준
    doc_tokens_list.append(tokens)
    df_counter.update(tokens)

idf_dict = {t: log((N + 1) / (df_counter[t] + 1)) for t in df_counter}

with open("idf_dict.json", "w", encoding="utf-8") as f:
    json.dump({"N": N, "idf": idf_dict}, f, ensure_ascii=False)
```

---

# 📌 3. ChromaDB에 문서 + KoSimCSE 토큰 업서트

```python
import chromadb
from chromadb.utils import embedding_functions

client = chromadb.PersistentClient(path="./chroma_kosimcse_db")
collection = client.get_or_create_collection(
    name="dragonkue_kosimcse",
    metadata={"hnsw:space": "cosine"}
)

# 임베딩도 KoSimCSE 직접 사용 (sentence-transformers 경로와 동일)
embedding_fn = embedding_functions.SentenceTransformerEmbeddingFunction(
    model_name="BM-K/KoSimCSE-roberta-multitask"
)

documents = docs
ids = [f"doc_{i}" for i in range(len(docs))]
metadatas = [{"tokens": tokens} for tokens in doc_tokens_list]

embeddings = embedding_fn(documents)

collection.add(
    ids=ids,
    documents=documents,
    metadatas=metadatas,
    embeddings=embeddings
)
```

---

# 📌 4. dragonKue cosine-idf score 함수 (동일하게 사용 가능)

```python
import numpy as np
import json
from math import log

with open("idf_dict.json", "r", encoding="utf-8") as f:
    data = json.load(f)
N = data["N"]
idf_dict = data["idf"]

def idf(term):
    df = idf_dict.get(term, 0)
    return log((N + 1) / (df + 1))

def dragonkue_idf_weight(query_terms, doc_terms, mode="avg"):
    common = set(query_terms) & set(doc_terms)
    if not common:
        return 0.0
    vals = [idf(t) for t in common]
    return sum(vals)/len(vals) if mode=="avg" else sum(vals)

def cosine_sim(q_vec, d_vec):
    q = np.array(q_vec)
    d = np.array(d_vec)
    return float(np.dot(q, d) / ((np.linalg.norm(q) * np.linalg.norm(d)) + 1e-8))

def dragonkue_score(q_vec, d_vec, q_terms, d_terms, alpha=0.2, idf_mode="avg"):
    cos = cosine_sim(q_vec, d_vec)
    w = dragonkue_idf_weight(q_terms, d_terms, idf_mode)
    return cos * (1 + alpha * w)
```

---

# 📌 5. Chroma 검색 + dragonKue Reranking

```python
query_text = "쿼리 문장"
query_terms = tokenize(query_text)
query_vec = embedding_fn([query_text])[0]

candidate_k = 50
final_k = 5

res = collection.query(
    query_embeddings=[query_vec],
    n_results=candidate_k,
    include=["documents", "metadatas", "embeddings"]
)

docs = res["documents"][0]
metas = res["metadatas"][0]
embs = res["embeddings"][0]
ids = res["ids"][0]

scored = []
for doc_id, doc, meta, emb in zip(ids, docs, metas, embs):
    doc_terms = meta["tokens"]
    score = dragonkue_score(query_vec, emb, query_terms, doc_terms)
    scored.append((score, doc_id, doc))

scored.sort(key=lambda x: x[0], reverse=True)
top = scored[:final_k]
```

---

# 📌 6. 출력 예시

```
[dragonKue score=0.8421] doc_03
문서 내용...

[dragonKue score=0.8312] doc_15
문서 내용...
```

---

# 📌 7. 실전 튜닝 팁

### ✔ alpha (가중치)
- 0.1 ~ 0.3 추천  
- subword 기반 IDF 값이 강하므로 0.5 이상은 추천하지 않음

### ✔ IDF mode
- `"avg"`: 긴 문서에 안정적  
- `"sum"`: FAQ/짧은 문서에서 강력

### ✔ KoSimCSE subword는 길이가 짧으므로
- IDF 값이 너무 커지는 경우  
  → `idf = min(idf, 5.0)` 같은 clipping 추천

---

# 📌 8. 결론

| 항목 | Konlpy 기반 | KoSimCSE tokenizer 기반 |
|------|-------------|---------------------------|
| 토큰 일관성 | 낮음 | 매우 높음 |
| 임베딩 모델과의 정합성 | 낮음 | **최적** |
| 의존성 | 많음 | transformers만 필요 |
| RAG 검색 품질 | 중간 | **상승** |

> **따라서 dragonKue cosine-idf + ChromaDB 조합에서는  
KoSimCSE 토크나이저 사용이 더 정확하고 효율적이다.**

---

# 📎 요청 시 생성 가능한 추가 파일

- LangChain용 `DragonKueChromaKosimRetriever` 완성본  
- KoSimCSE + BM25 + dragonKue 삼중 Hybrid 파이프라인  
- PDF / DOCX 변환본  
- FastAPI 기반 Rerank 서버 버전  

원하시면 바로 만들어 드립니다.
