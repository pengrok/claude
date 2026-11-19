# RAG 검색/인덱싱 후보 주제 선행연구(Prior-art) 점검 — 2026-11-19

방법: WebSearch(표준/확장)로 후보마다 4~6개 질의를 돌렸다. alphaxiv.org, emergentmind.com, api.semanticscholar.org는 이 세션 네트워크에서 차단(DNS 실패)되어 WebFetch로 원문을 열지 못했다. 그래서 아래 내용은 검색 결과의 제목/ID/스니펫에 근거한다. 스니펫으로만 확인했거나 ID가 검색 인덱스에 없는 항목은 "미확인"으로 표시했다.

---

## C1. 계층 요약 / GraphRAG 인덱스의 삭제·수정 업데이트 + "stale answer rate"

### 가장 가까운 기존 연구
| 연구 | ID/URL | 연도 | 요약 |
|---|---|---|---|
| EraRAG | arXiv 2506.20963 | 2025 | 계층 요약 그래프에 삽입이 일어나면 영향받는 세그먼트와 조상만 재요약한다. 삭제는 다루지 않는 것으로 보임 |
| adRAP (Recursive Abstractive Processing for Retrieval in Dynamic Datasets) | arXiv 2410.01736 | 2024 | RAPTOR 트리를 문서 **추가/삭제** 환경에서 유지한다(adaptive UMAP + online GMM, 트리 일부만 재계산). 삭제된 청크를 담고 있던 요약을 어떻게 무효화하는지는 스니펫으로 미확인 |
| Stashbird (Episode-Grounded Lifecycle Reconciliation…) | arXiv 2609.34242 | 2026 | 대화 메모리용 시스템. provenance로 에피소드가 영향을 준 엔티티/커뮤니티를 추적하고, 업데이트·삭제 시 stale 커뮤니티 문서를 재생성 전까지 검색에서 제외한다. 업데이트 시퀀스 20개, 삭제 개입 263건으로 정합성을 검증 |
| RAGU | arXiv 2607.11683 | 2026 | GraphRAG 엔진. upsert/update/delete를 해시 기반으로 증분 유지한다(계층 Leiden). 삭제 뒤 커뮤니티 요약을 갱신하는지는 미확인 |
| HyGRAG (Unified Context-/Relation-aware GraphRAG) | arXiv 2606.18075 | 2026 | 동적 업데이트 시 국소 재요약을 수행. 삭제/수정 지원 여부는 미확인 |
| RAGPerf | arXiv 2603.10765 | 2026 | 범용 RAG 벤치마크. insert/update/delete 워크로드에서 갱신된 사실을 검색하는지(stale 여부) 확인한다. GraphRAG 전용은 아님 |
| T-GRAG / Time-LongQA | ACM MM 2025 (10.1145/3746027.3755628) | 2025 | 시간적 충돌·중복을 다루는 동적 GraphRAG |
| TagRAG | arXiv 2601.05254 | 2026 | 증분 삽입 전용 |
| 산업계 | FalkorDB GraphRAG SDK, LightRAG | — | 문서 삭제 시 orphan 엔티티를 정리하고 provenance 기반으로 엣지를 철회한다. 커뮤니티 요약 무효화에 대한 정량 평가는 없음 |

### 판정: **부분적으로 다뤄짐**
- 이미 다뤄진 부분: 트리 삭제 지원(adRAP), provenance 기반 삭제 정합성(Stashbird, 대화 메모리 도메인), 엔지니어링 수준의 delete 지원(RAGU, FalkorDB, LightRAG), 범용 RAG의 stale 검사(RAGPerf).
- 아직 열려 있는 부분: (a) **문서 코퍼스용 GraphRAG/RAPTOR의 커뮤니티·트리 요약**을 대상으로, 삭제·수정 시 provenance로 국소 무효화하고 재요약 비용과 정확도의 trade-off를 체계적으로 평가한 연구. (b) 업데이트 이후 "stale answer rate", "zombie fact rate"(삭제된 사실이 요약에 남아 답변에 나오는 비율)를 측정하는 **GraphRAG 전용 벤치마크**.

### 더 날카로운 프레이밍
"요약 계층의 잔류 지식(residual knowledge)": 삭제·수정된 사실이 상위 요약에 남아 답변으로 새는 현상을 정의한다. 이를 측정하는 벤치마크(DeleteQA류: 삭제 전후 질의 쌍)를 만들고, provenance 그래프로 무효화 범위를 최소화하는 기법을 adRAP·EraRAG·RAGU·full rebuild와 비교한다. Stashbird와는 도메인(대화 메모리 vs 문서 코퍼스 계층 요약)과 메트릭(잔류 사실 누출)으로 차별화한다. "unlearning/compliance(GDPR 삭제)" 관점을 넣으면 더 차별화된다.

---

## C2. 임베딩 압축의 비대칭적 열화: 추론 집약 검색(BRIGHT)·조합 질의(LIMIT) vs MTEB/BEIR

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| SMEC | arXiv 2510.12474 | 2025 | MRL 압축을 개선. 평가는 BEIR류 |
| When Is 0.1% Enough? | arXiv 2606.01074 | 2026 | 차원 축소와 스칼라 양자화의 결합 효과를 MTEB 4개 task family, 4개 모델로 분석. "최적 압축은 태스크마다 다르다"는 결론이며 BRIGHT/LIMIT은 포함되지 않은 것으로 보임(스니펫 기준) |
| LIMIT (On the Theoretical Limitations of Embedding-Based Retrieval) | arXiv 2508.21038 | 2025 | 차원과 표현 가능한 top-k 조합 수의 이론적 한계를 제시하고 MRL truncation 시 큰 열화를 보고. 양자화는 다루지 않음 |
| KaLM-Embedding-V2 | arXiv 2506.20923 | 2025 | 부록에서 차원 축소 시 retrieval/reranking의 하락폭이 가장 크다고 보고 |
| Arctic-Embed 2.0 | arXiv 2412.04506 | 2024 | MRL과 스칼라 양자화를 결합해도 열화가 작다(BEIR/MTEB) |
| Retrieval Needs Multivectors: An Exponential Separation | arXiv 2608.21494 | 2026 | 이론적으로 멀티벡터가 필요하다는 주장(LIMIT 계열) |
| SmallReason-ColBERT | arXiv 2609.29652 | 2026 | 초소형 late-interaction 추론 검색기(BRIGHT). 압축 연구와 인접 |
| ReasonIR / ReasonEmbed (2510.08252) / GEM (2608.13200) | — | 2025–26 | 추론 검색기 자체만 다룸. 압축 분석은 없음 |

### 판정: **열려 있음** (단, 진입 장벽이 낮고 경쟁이 빠를 수 있음)
BRIGHT에서 MRL truncation, int8/binary 양자화, 멀티벡터 토큰/패치 pruning을 체계적으로 비교한 연구는 찾지 못했다. LIMIT에 대해서는 truncation 열화만 보고되었고 양자화는 미검증이다. "compression-aware training for reasoning retrievers"도 찾지 못했다.

### 더 날카로운 프레이밍
"압축은 공짜가 아니다, 추론 검색에서는": 같은 압축률에서 BEIR 대비 BRIGHT/LIMIT의 상대 열화비(asymmetry ratio)를 정량화한다. 열화가 어느 차원/비트에 집중되는지 분석한다(예: 추론 신호가 MRL 저차원 prefix에 잘 담기지 않음). 그다음 추론 데이터(ReasonIR 합성 쿼리 등)로 MRL/QAT를 하는 compression-aware 학습이 그 격차를 줄이는지 보인다. 2606.01074와 SMEC를 직접 baseline으로 둔다.

---

## C3. Set-level utility(방해 문서 인지 listwise) 검색기 학습 + 생성기 간 전이 + closed-book 반사실 보정

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| OptiSet | arXiv 2601.05027 / Findings EMNLP 2026 | 2026 | 집합 선택과 집합 수준 랭킹을 통합. 생성기의 집합 조건부 perplexity 변화를 utility로 쓰고 set-listwise로 학습. **set-level utility + listwise 부분과 상당히 겹침** |
| SetR (Shifting from Ranking to Set Selection) | arXiv 2507.06838 | 2025 | 집합 단위 passage 선택 |
| LLM-Specific Utility | arXiv 2510.11358 (CIKM '26) | 2025 | gold utility 집합이 LLM 간에 전이되지 않음을 보임. 전이 문제의 동기를 제공 |
| SCARLet (Shared Context Attribution) | arXiv 2504.00573 / EMNLP 2025 | 2025 | 공유 컨텍스트 perturbation으로 utility를 귀속해 검색기를 학습 |
| GainRAG | ACL 2025 | 2025 | passage "gain"을 정의하고 LLM 과신 편향을 완화. closed-book 대비 차분인지는 미확인 |
| Aligning Dense Retrievers with LLM Utility via Distillation | arXiv 2604.22722 | 2026 | perplexity 감소 기반 utility 분포를 dense retriever로 증류 |
| UDCG | arXiv 2510.21440 | 2025 | 방해 문서를 반영한 집합 수준 평가 메트릭 |
| Know Before You Fetch | arXiv 2606.29959 | 2026 | closed-book 충분성을 보정 확률로 예측. 게이팅용이며 검색기 학습은 아님 |
| REPLUG, Power of Noise | — | 2023–24 | 고전적 배경 연구 |

### 판정: **부분적으로 다뤄짐**
- 다뤄진 부분: set-level utility를 쓰는 listwise 학습(OptiSet, SetR), utility의 LLM 특이성 문제 제기(2510.11358), perturbation 기반 utility(SCARLet).
- 열려 있는 부분: (a) **closed-book 정답 가능성을 빼는 반사실 utility**(utility = P(정답|집합) − P(정답|∅))로 dense retriever를 학습한 연구는 찾지 못했다. (b) **여러 생성기에 걸쳐 전이되는** utility 목적함수(예: 생성기 앙상블/불변성 정규화)와 이를 held-out LLM에서 평가한 연구도 찾지 못했다.

### 더 날카로운 프레이밍
"Knowledge-gap-aware retriever": 검색기가 LLM이 이미 아는 것을 다시 가져오지 않고, 모르는 것과 방해 문서에 강한 집합을 고르도록 counterfactual set utility로 학습한다. 핵심 실험은 학습에 쓰지 않은 생성기 패밀리(크기/계열이 다른 LLM)로의 zero-shot 전이다. OptiSet과는 "closed-book 차분 + 생성기 불변성"으로 구분한다. 2510.11358의 비전이성 결과를 직접 반박하거나 완화하는 실험을 포함한다.

---

## C4. 인덱스 구축 전 예측기: GraphRAG/RAPTOR/청크 크기가 이득일지 사전 예측

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| When to use Graphs in RAG (GraphRAG-Bench) | arXiv 2506.05690 (ICLR 2026으로 인용됨, 미확인) | 2025 | 태스크 복잡도와 코퍼스 구조에 따라 GraphRAG 효과가 달라짐을 정성적·벤치마크로 분석. 예측기는 없음 |
| RAG vs GraphRAG | arXiv 2502.11371 | 2025 | 체계적 비교 |
| EA-GraphRAG (Use Graph When It Needs) | arXiv 2602.03578 | 2026 | **질의 단위** MLP 분류기로 GraphRAG가 RAG보다 나을지 예측해 라우팅한다. 이미 구축된 인덱스를 전제함 |
| MoC | arXiv 2503.09600 | 2025 | 청킹 품질 평가와 mixture-of-chunkers |
| BM25 Wins at Scale | arXiv 2607.26497 | 2026 | 규모에 따른 역전 현상(호출자 제공, 이번 검색에서 직접 재확인하지 못함) |
| Does Document Structure Help Dense Retrieval? | arXiv 2610.10170 | 2026 | 문서 구조 메커니즘에 대한 placebo 통제 ablation. 인접 연구 |
| AutoRAG, RankML류 메타러닝 | — | — | 파이프라인 탐색/메타러닝. RAG 인덱스 유형을 사전 예측하는 연구는 찾지 못함 |

### 판정: **열려 있음** (질의 단위 라우팅은 다뤄짐, 코퍼스 단위 사전 예측은 비어 있음)
엔티티 밀도, 교차 문서 참조율, 질의 분포 같은 값싼 통계로 "인덱스를 만들기 전에" 투자 가치를 예측하는 연구는 찾지 못했다. 블로그·실무 휴리스틱(엔티티 1,000개 미만이면 벡터 RAG 등)만 있다.

### 더 날카로운 프레이밍
"Index-or-not 메타 예측기": 다양한 코퍼스 수십 개에서 (vanilla, chunk 크기 3종, RAPTOR, GraphRAG, LightRAG)를 실제로 구축해 메타 데이터셋을 만든다. 코퍼스 1~5% 샘플에서 얻는 값싼 특징(엔티티 밀도, 공출현 그래프 모듈성, 질의의 다중 엔티티 비율, BM25-dense 일치도)으로 이득 부호와 비용 대비 효과를 예측한다. 평가 지표는 "회피한 인덱싱 비용 대비 손실 정확도"(regret)로 한다. EA-GraphRAG와는 "사전(pre-indexing)·코퍼스 단위"로 구분한다.

---

## C5. 문맥화 청킹(Contextual Retrieval, Late Chunking)의 문서 내 판별력 붕괴

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| Beyond Chunk-Then-Embed | arXiv 2602.16974 (SIGIR 2026으로 표기, 미확인) | 2026 | 문맥화 청킹이 in-corpus 성능은 높이지만 **in-document 검색은 떨어뜨린다**는 현상을 보고 |
| ConTEB / InSeNT (Context is Gold…) | arXiv 2505.24782 / EMNLP 2025 | 2025 | 같은 문서 청크를 negative로 쓰는 대조 학습으로 청크 특이성을 유지 |
| Contextual Embeddings for Document Retrieval | arXiv 2610.38854 (ID 검증 불가, 미확인) | 2026 | 스니펫 기준: KL 목적함수 + 문서 수준 대조 손실, gold chunk 라벨 없이 형제 청크를 처리 |
| Calibrating Document-Local Boundaries for Contextual Chunk Retrieval | arXiv 2611.12097 (스니펫 1건에서만 확인, 미확인) | 2026-11 | gold 청크와 가장 어려운 형제 청크 사이의 결정 경계를 보정하는 것으로 보임. **C5의 "fix" 부분과 직접 겹칠 가능성이 높음** |
| Visual Late Chunking | arXiv 2604.10167 | 2026 | 시각 문서에서의 문맥화 청킹 실증 연구 |
| Perplexity 문맥 임베딩 공개 | 업계 발표 (2026-10) | 2026 | 문서 전체를 인코딩한 뒤 청크별 벡터를 생성 |
| All-But-The-Top 등 anisotropy 보정 | arXiv 1702.01417 등 | 고전 | 공통 성분 제거. 청크/문서 수준 적용은 찾지 못함 |

### 판정: **부분적으로 다뤄짐** (경쟁이 매우 빠름)
- 현상 보고: 2602.16974. 학습 기반 해법: ConTEB/InSeNT, 2610.38854. 경계 보정: 2611.12097(미확인).
- 열려 있는 부분: **학습이 필요 없는(post-hoc) 문서 공통 성분 제거/디바이어싱**(문서별 평균 또는 top-PC 제거, 쿼리 의존 혼합 계수)으로 in-corpus 이득을 유지하면서 in-document 판별력을 회복하는 연구는 찾지 못했다. Contextual Retrieval(LLM 접두사 방식)에서 같은 붕괴가 일어나는지 정량화한 연구도 찾지 못했다.

### 더 날카로운 프레이밍
"Context without collapse": (1) 붕괴 지표를 정의한다(문서 내 청크 쌍 유사도 분포, intra-doc rank entropy). Late Chunking, Anthropic Contextual Retrieval, ConTEB 모델 사이에서 이를 비교한다. (2) 학습 없는 해법으로 e_chunk − α·e_doc_common을 쓰되 α를 질의 유형(in-corpus vs in-document)에 따라 조정한다. (3) 2단계 검색(문서는 문맥화 벡터로, 문서 내 청크는 디바이어싱 벡터로)을 쓴다. 2611.12097과 2610.38854를 먼저 원문으로 확인하는 것이 필수다.

---

## C6. 멀티벡터(ColBERT/ColPali)·양자화(PQ/RaBitQ) 인덱스의 다세대 하위호환 임베딩 업그레이드

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| Drift-Adapter | EMNLP 2025 (aclanthology 2025.emnlp-main.805) | 2025 | 새 쿼리를 레거시 공간으로 매핑해 기존 ANN 인덱스를 재사용. 단일 벡터만 다룸 |
| Query Drift Compensation | arXiv 2506.00037 | 2025 | 쿼리 측 드리프트 보정 |
| MKP-Adapter (Multi-modal Knowledge Preserving Adapter for Embedding BC) | arXiv 2609.16875 | 2026 | MLLM 임베딩용 adapter-only BCT. 단일 벡터 |
| RAGFlip | arXiv 2610.07266 | 2026 | 검색기 업그레이드 시 query 단위 negative flip 측정(BM25→dense). 호환성 학습은 다루지 않음 |
| BCT 계열(이미지 검색) | arXiv 2201.09724 (RACT), 2303.04766 (FastFill), 2301.03767 (online backfilling) | 2022–23 | 하위호환/backfilling 고전 연구. 단일 벡터 |
| Binary Embedding Retrieval at Tencent | arXiv 2302.08714 | 2023 | 이진 임베딩 + 하위호환 학습. 양자화와 호환성을 함께 다룬 드문 사례 |
| ColBERTSaR | arXiv 2606.05568 | 2026 | PQ로 ColBERT 인덱스를 희소화. 업그레이드는 다루지 않음 |

### 판정: **열려 있음**
멀티벡터 late-interaction 인덱스(토큰 단위 MaxSim 공간, PLAID centroid/residual 코드)와 PQ/RaBitQ 코드북 인덱스에서, **여러 세대에 걸친** 하위호환 업그레이드를 다룬 연구는 찾지 못했다.

### 더 날카로운 프레이밍
"Compatible late interaction": (a) 토큰 수준 어댑터(query token → legacy token space)를 MaxSim 보존 손실로 학습한다. (b) PLAID centroid·PQ 코드북을 고정한 채 새 모델의 잔차만 재계산하는 "codebook-anchored BCT"를 제안한다. (c) v1→v2→v3 체인에서 호환성 오차 누적과 negative flip(RAGFlip 지표 차용)을 평가한다. 단일 벡터 Drift-Adapter를 멀티벡터에 순진하게 확장한 방식을 baseline으로 둔다.

---

## C7. 검색 여부·질의 확장 유형·k·압축률을 하나의 토큰/지연 예산 아래 결합하고 conformal 보장을 주는 통합 정책

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| Know Before You Fetch | arXiv 2606.29959 | 2026 | closed-book / k=1 / k=5 / abstain 중에서 **보정된 확률**로 선택하고 지연·토큰 trade-off를 분석. k와 검색 여부를 결합. 압축·질의 확장은 없음 |
| When Should Active RAG Retrieve? | arXiv 2607.24010 | 2026 | 예산 인지 평가 프로토콜(실현 사용률, threshold 전이 오차). 단순 불확실성 baseline이 학습 라우터와 비슷함을 보임 |
| When Knowledge Is Not Free | arXiv 2606.02245 | 2026 | 접근 비용이 다른 증거를 예산 아래에서 선택. LLM 컨트롤러는 아직 불안정함 |
| Adaptive Retrieval, Selective Answers | arXiv 2610.34872 | 2026 | 에이전트형 RAG에 conformal factuality를 적용 |
| Principled Context Engineering for RAG (conformal filtering) | arXiv 2511.17908 | 2025 | 검색 후 conformal 필터로 컨텍스트 크기를 제어하고 coverage를 보장 |
| TRAQ | arXiv 2307.04642 / NAACL 2024 | 2024 | 검색 집합과 생성 집합 각각에 conformal 보장 |
| Conformal risk control for RAG configs | arXiv 2402.03181 | 2024 | k 등 RAG 설정을 conformal risk로 선택(C-RAG 계열로 보임, 제목 미확인) |
| AdaMem | arXiv 2609.22100 | 2026 | soft compression 토큰 예산을 passage별로 할당 |
| TARG | arXiv 2511.09803 | 2025 | 학습 없는 검색 게이팅 |
| Adaptive compression for edge RAG | arXiv 2608.19535 | 2026 | 런타임 압축률 제어 |
| UR² | ACL 2026 (2508.06165) | 2026 | RL로 검색과 추론을 조정. 요약 압축 포함. 예산 통합 정책은 아님 |
| AdaRankLLM 2604.15621, Provence | — | — | (호출자 제공) k/pruning 적응 |

### 판정: **부분적으로 다뤄짐**
- 다뤄진 부분: 검색 여부 + k의 보정 기반 결합(2606.29959), 예산 평가 프로토콜(2607.24010), conformal k/컨텍스트 크기(TRAQ, 2511.17908, 2402.03181), 압축 예산 할당(AdaMem).
- 열려 있는 부분: **네 결정(검색 여부 × 질의 확장 유형 × k × 압축률)을 하나의 조합 행동 공간에서 단일 비용 제약으로 고르면서**, 실현 예산과 오류율 모두에 대해 분포 무관 보장(예: Learn-then-Test / conformal risk control의 다중 제약 버전)을 주는 연구는 찾지 못했다.

### 더 날카로운 프레이밍
"Certified RAG budget controller": 행동 공간을 조합 격자로 두고, 오류 위험 ≤ ε와 **실현** 토큰 비용 ≤ B를 동시에 보장하는 LTT(Learn-then-Test) 기반 다중 제약 보정을 설계한다. 2607.24010이 지적한 "명목 예산 ≠ 실현 예산" 문제를 보장 대상에 포함하는 것이 핵심 차별점이다. 확장/압축 축이 k·게이팅 축과 상호작용함을 보이는 ablation이 필요하다(예: 압축하면 큰 k가 유리해지는 현상).

---

## 요약표
| 후보 | 판정 | 가장 많이 겹치는 연구 |
|---|---|---|
| C1 | 부분적으로 다뤄짐 | adRAP 2410.01736, Stashbird 2609.34242, RAGU 2607.11683 |
| C2 | 열려 있음 | 2606.01074, SMEC 2510.12474, LIMIT 2508.21038 |
| C3 | 부분적으로 다뤄짐 | OptiSet 2601.05027, 2510.11358, SCARLet 2504.00573 |
| C4 | 열려 있음(코퍼스 단위) | EA-GraphRAG 2602.03578, GraphRAG-Bench 2506.05690 |
| C5 | 부분적으로 다뤄짐(경쟁 빠름) | 2602.16974, ConTEB 2505.24782, 2611.12097(미확인)/2610.38854(미확인) |
| C6 | 열려 있음 | Drift-Adapter, MKP-Adapter 2609.16875, RAGFlip 2610.07266 |
| C7 | 부분적으로 다뤄짐 | 2606.29959, 2607.24010, 2511.17908 |

주의: 2026년 하반기 preprint(특히 C5의 2611.12097, 2610.38854)는 원문을 열어보지 못했다. 착수 전에 arXiv 원문 확인이 필요하다.
