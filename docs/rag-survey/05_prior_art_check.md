# RAG 검색/인덱싱 후보 주제 선행연구(Prior-art) 점검 — 2026-11-19 (본문 대조 교정 2026-11-20)

방법: 처음에는 WebSearch(표준/확장)로 후보마다 4~6개 질의를 돌려 후보 논문을 모았다. 그 뒤 각 "가장 가까운 기존 연구"의 arXiv/ACL Anthology 본문(pdftotext)과 arXiv 메타데이터를 직접 읽어 초록·방법·실험·한계를 대조했다. venue는 본문 헤더/ACM Reference Format/arXiv comment로 확인했고, 근거가 없으면 "venue 근거 없음"으로 적었다. 논문이 아니거나 로컬 본문이 없는 항목(T-GRAG, FalkorDB/LightRAG)만 "본문 확인 불가/본문 접근 불가"로 남겼다.

---

## C1. 계층 요약 / GraphRAG 인덱스의 삭제·수정 업데이트 + "stale answer rate"

### 가장 가까운 기존 연구
| 연구 | ID/URL | 연도 | 요약 |
|---|---|---|---|
| EraRAG | arXiv 2506.20963 (venue 근거 없음, "Under review") | 2025 | 계층 요약 그래프에 LSH 버킷으로 청크를 **삽입**하고, 바뀐 세그먼트와 그 조상만 재요약한다. 실험은 초기 50% 구축 후 5%씩 10회 삽입. 문서 삭제·수정은 다루지 않는다(본문 확인) |
| adRAP (Recursive Abstractive Processing for Retrieval in Dynamic Datasets) | arXiv 2410.01736 (venue 근거 없음) | 2024 | adaptive UMAP과 adaptive GMM으로 RAPTOR 트리를 문서 **추가** 시 국소 갱신한다. 삭제는 §4.4 끝 한 문장("removing the chunk from the tree and recomputing summaries for its ancestors")으로만 언급하고 실험하지 않는다. 벤치: MultiHop-RAG, QASPER, QuALITY |
| Stashbird (Episode-Grounded Lifecycle Reconciliation for Conversational Agent Memory) | arXiv 2609.34242 (venue 근거 없음) | 2026 | 대화 메모리 시스템(Microsoft). provenance로 에피소드가 지지하는 관계를 추적해, 삭제 시 지지 없는 레코드만 제거하고 영향받은 커뮤니티 문서는 재생성 전까지 검색에서 뺀다. 업데이트 20개 시퀀스 전부 통과(재조정 없이는 8개), 삭제 개입 263건(LoCoMo 203 + LongMemEval 60)에서 제거·보존 검사 통과. 평가는 **레코드 수준 상태 검사**이며 답변 누출은 측정하지 않는다. 저자 한계: "conversation summaries and rolling checkpoints" 는 삭제 후에도 그대로 남는다 |
| RAGU | arXiv 2607.11683 (venue 근거 없음) | 2026 | GraphRAG 엔진. 계층 Leiden 커뮤니티 리포트를 만들고, 해시 기반 ID·merge 정책·consistency auditor로 upsert/update/delete를 지원한다고 기능 목록에 적는다. 삭제 후 커뮤니티 리포트를 어떻게 처리하는지에 대한 서술이나 평가는 본문에 없다 |
| HyGRAG (A Unified Framework for Context-Aware and Relation-Aware Graph RAG) | arXiv 2606.18075 / WWW '26 | 2026 | 새 내용을 계층에 attach하고 조상 커뮤니티만 재요약한다(§3.4). 실험(RQ3)은 증분 **삽입**만 하며 정적 구축 대비 1–2% 하락. 삭제·수정은 다루지 않는다. 커뮤니티 요약 환각률 17%(수작업 100개) 보고 |
| RAGPerf | arXiv 2603.10765 (venue 근거 없음) | 2026 | 범용 RAG 성능 벤치. Insert/Update/Removal 연산을 지원하고, Update에 대해서는 LLM으로 합성한 사실 변경 QA로 "stale data가 아닌 갱신 사실을 검색하는지" 확인한다. Removal의 누출 QA는 없고, 대상은 벡터 DB(LanceDB 등)이며 GraphRAG 요약은 아니다 |
| T-GRAG / Time-LongQA | ACM MM 2025 (10.1145/3746027.3755628) | 2025 | 시간적 충돌·중복을 다루는 동적 GraphRAG (로컬 본문 없음: 본문 접근 불가) |
| TagRAG | arXiv 2601.05254 / ACL 2026 Findings | 2026 | 태그 계층 KG. 증분 **삽입**(tag/knowledge increment) 전용. 삭제 언급 없음 |
| 산업계 | FalkorDB GraphRAG SDK, LightRAG | — | 문서 삭제 시 orphan 엔티티 정리, provenance 기반 엣지 철회 (논문 아님: 본문 확인 불가). 커뮤니티 요약 무효화 정량 평가는 없음 |

### 판정: **부분적으로 다뤄짐** (변경 없음, 공백은 본문으로 더 분명해짐)
- 이미 다뤄진 부분: 계층 요약의 증분 **삽입**(EraRAG, HyGRAG, TagRAG, adRAP), 대화 메모리에서 provenance 기반 삭제 정합성(Stashbird, 레코드 수준), 엔지니어링 수준의 delete API(RAGU), 벡터 RAG에서 update 후 stale 검색 여부(RAGPerf).
- 아직 열려 있는 부분: (a) **문서 코퍼스용 GraphRAG/RAPTOR 커뮤니티·트리 요약**에서 삭제·수정을 실제로 실행하고 재요약 비용과 정확도를 함께 평가한 연구는 없다. adRAP은 삭제를 한 문장으로 주장만 하고, 나머지 계층 요약 연구는 모두 삽입만 실험한다. (b) 일곱 편 모두 **삭제·수정된 문서의 사실이 답변으로 새는 비율**(stale/zombie fact rate)을 측정하지 않는다. Stashbird조차 레코드 검사만 하고, 상위 대화 요약은 삭제 후에도 남는다고 한계로 적었다.

### 더 날카로운 프레이밍
"요약 계층의 잔류 지식(residual knowledge)": 삭제·수정된 사실이 상위 요약에 남아 답변으로 새는 현상을 정의한다. 이를 측정하는 벤치마크(DeleteQA류: 삭제 전후 질의 쌍)를 만들고, provenance 그래프로 무효화 범위를 최소화하는 기법을 adRAP(삭제 확장 구현)·EraRAG·HyGRAG·RAGU·full rebuild와 비교한다. Stashbird와는 도메인(대화 메모리 vs 문서 코퍼스 계층 요약)과 메트릭(레코드 검사 vs 답변 수준 잔류 사실 누출)으로 차별화한다. Stashbird가 스스로 밝힌 "삭제 후 요약은 그대로" 한계가 직접 동기가 된다. "unlearning/compliance(GDPR 삭제)" 관점을 넣으면 더 차별화된다.

---

## C2. 임베딩 압축의 비대칭적 열화: 추론 집약 검색(BRIGHT)·조합 질의(LIMIT) vs MTEB/BEIR

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| SMEC | arXiv 2510.12474 / EMNLP 2025 | 2025 | Sequential MRL + Adaptive Dimension Selection + S-XBM으로 MRL 압축을 개선. 평가는 BEIR(텍스트), Products-10K, Fashion-200K. 양자화·BRIGHT 없음 |
| When Is 0.1% Enough? | arXiv 2606.01074 (venue 근거 없음) | 2026 | 차원 축소(Head, PCA+ROR)와 양자화의 결합 효과를 MTEB 4개 task family(classification, clustering, retrieval, STS)·**5개** 모델로 분석. retrieval이 가장 압축에 민감. retrieval 데이터는 MIRACL·Quora·HotpotQA·DBpedia·NQ·MS MARCO이며 BRIGHT/LIMIT은 없다(본문 확인). 저자 한계: MTEB 일부·영어만 |
| LIMIT (On the Theoretical Limitations of Embedding-Based Retrieval) | arXiv 2508.21038 / ICLR 2026 | 2025 | 차원과 표현 가능한 top-k 조합 수의 이론적 하한을 제시하고, LIMIT에서 MRL truncation(32차원까지) 시 성능이 차원에 비례해 떨어짐을 보인다. 양자화는 논의에서 언급만 하고 실험하지 않음 |
| KaLM-Embedding-V2 | arXiv 2506.20923 / ICLR 2026 | 2025 | 부록 D(Table 11)에서 MRL 차원 축소 시 retrieval·reranking의 하락폭이 가장 크다고 보고(MTEB cmn/eng) |
| Arctic-Embed 2.0 | arXiv 2412.04506 (venue 근거 없음) | 2024 | MRL(256차원)을 사전학습·미세조정 모두에 적용해 truncation 시 MTEB-R 원점수의 99%(m)/98%(l)를 유지한다(MTEB-R, CLEF, MIRACL). 스칼라 양자화와의 결합은 선행연구(Merrick 2024)를 인용할 뿐 **본문에서 실험하지 않는다** |
| Retrieval Needs Multivectors: An Exponential Separation | arXiv 2608.21494 (venue 근거 없음) | 2026 | 단일벡터는 지수 크기가 필요하지만 멀티벡터는 다항 크기로 충분한 질의·문서 집합 계열을 제시하고 ANDOR 벤치를 만든다(LIMIT 계열). 압축 실험 없음 |
| SmallReason-ColBERT | arXiv 2609.29652 / EMNLP 2026 Main | 2026 | 32M late-interaction 추론 검색기, BRIGHT 평균 nDCG@10 21.41(150M Reason-ModernColBERT 22.62). 압축 연구는 아니지만, ablation에서 **질의 토큰 gate를 0.95 미만에서 잘라내는 hard pruning이 BRIGHT를 16.02로 떨어뜨린다**(10/12 split)고 보고한다. 투영 차원 64→128 확장이 추론 split에 필요하다고 서술. 문서측 인덱스 압축·양자화는 없음 |
| ReasonIR / ReasonEmbed (2510.08252, ACL 2026 Main) / GEM (2608.13200) | — | 2025–26 | 추론 검색기 자체만 다룸. 압축·양자화·MRL 분석은 없음(본문 확인) |

### 판정: **열려 있음** (변경 없음, 단 진입 장벽이 낮고 경쟁이 빠를 수 있음)
본문 기준으로 BRIGHT에서 MRL truncation, int8/binary 양자화, 멀티벡터 문서 토큰/패치 pruning을 체계적으로 비교한 연구는 없다. 압축을 평가한 논문(2606.01074, SMEC, KaLM-V2, Arctic-Embed 2.0)은 모두 MTEB/BEIR/MIRACL류만 쓰고, LIMIT은 truncation만 실험했다. 인접 단서는 두 가지다. (1) "retrieval이 classification·STS보다 압축에 민감하다"는 결과는 MTEB 안에서 이미 반복 보고되었으므로, 기여는 **BEIR 대비 BRIGHT/LIMIT의 추가 비대칭**이어야 한다. (2) SmallReason-ColBERT의 질의측 hard pruning 결과(21.41→16.02)는 단일 ablation이지만, 추론 검색에서 토큰 pruning이 크게 손해일 수 있다는 예비 증거다. "compression-aware training for reasoning retrievers"는 찾지 못했다.

### 더 날카로운 프레이밍
"압축은 공짜가 아니다, 추론 검색에서는": 같은 압축률에서 BEIR 대비 BRIGHT/LIMIT의 상대 열화비(asymmetry ratio)를 정량화한다. 열화가 어느 차원/비트에 집중되는지 분석한다(예: 추론 신호가 MRL 저차원 prefix에 잘 담기지 않음). 그다음 추론 데이터(ReasonIR 합성 쿼리 등)로 MRL/QAT를 하는 compression-aware 학습이 그 격차를 줄이는지 보인다. 2606.01074(Head/PCA+ROR × 비트폭 격자)와 SMEC를 직접 baseline으로 두고, 멀티벡터 쪽은 SmallReason-ColBERT의 질의 pruning 결과를 문서측 pruning으로 확장한다.

---

## C3. Set-level utility(방해 문서 인지 listwise) 검색기 학습 + 생성기 간 전이 + closed-book 반사실 보정

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| OptiSet | arXiv 2601.05027 (venue 근거 없음) | 2026 | Expand-then-Refine로 후보 집합을 만들고, 생성기의 집합 조건부 정답 엔트로피 H = log PPL(a\|q,S)에서 **passage 없는 기준 엔트로피 H_o를 뺀 ∆H = H_p − H_o**로 라벨을 만든다("low entropy may reflect the generator's internal knowledge"). LLaMA3.1-8B LoRA **LLM 집합 선택기**를 CE + set-listwise KL로 학습(검색 후보는 Contriever top-20). 단일 생성기이며, 저자는 생성기마다 선호 조합이 다를 수 있어 일반화가 향후 과제라고 적는다. **set-level utility + listwise + closed-book 차분까지 겹침** |
| SetR (Shifting from Ranking to Set Selection) | arXiv 2507.06838 / ACL 2025 main | 2025 | CoT로 정보 요구를 식별하고 집합 단위로 passage를 선택 |
| LLM-Specific Utility | arXiv 2510.11358 / CIKM '26 | 2025 | utility를 **u(d;L,q) = Acc(L(q,d)) − Acc(L(q,∅))**(closed-book 대비 정확도 차, pointwise)로 정의하고, Qwen3-8B/14B/32B·Llama3.1-8B와 6개 QA에서 gold utility passage가 LLM 간에 전이되지 않음을 보인다. 검색기 학습은 하지 않으며, 저자는 pointwise라 조합 효과를 반영하지 못한다고 한계로 적었다 |
| SCARLet (Shared Context Attribution) | arXiv 2504.00573 / EMNLP 2025 Main | 2025 | 공유 컨텍스트에서 passage를 무작위로 빼는 섭동 후 정답 토큰 logit 합을 ridge 회귀로 귀속해 utility를 얻고(LIME식), 이를 positive/negative로 나눠 dense retriever(Contriever, BGE)를 대조학습한다. 학습 신호는 LLaMA-3-8B로 만들고, LLaMA-3-8B와 Qwen2.5-3B 두 생성기에서 개선을 보여 "preliminarily" 생성기 적응성을 주장한다 |
| GainRAG | ACL 2025 (2025.acl-long.527) | 2025 | passage "gain"을 contrastive decoding으로 정의한다: logit을 (1+α)·logit(a\|c,q) − α·logit(a\|q)로 보정한 perplexity. 즉 **closed-book 사전확률을 토큰 수준에서 빼는 반사실 보정**이다. 단일 passage, BGE-reranker 선택기, LLama3-8B |
| Aligning Dense Retrievers with LLM Utility via Distillation (UAE) | arXiv 2604.22722 (venue 근거 없음) | 2026 | utility U(q,d,a) = exp(평균 log p(a\|q,d))(본문 식은 closed-book 항 없는 절대 우도)를 BERT reward model로 근사하고 Utility-Modulated InfoNCE로 bi-encoder에 증류한다. 생성기 Llama-3-8B-Instruct 고정("generator-specific alignment"), QASPER, Token F1 +17.3%(vs BGE-Base) |
| UDCG | arXiv 2510.21440 (venue 근거 없음) | 2025 | 방해 문서를 반영한 집합 수준 평가 메트릭, 사람 판단과의 상관을 최대 36% 개선 |
| Know Before You Fetch | arXiv 2606.29959 (venue 근거 없음) | 2026 | closed-book confidence로 질의별 검색 passage 수(0/1/5)를 할당하는 정책을 같은 평균 예산의 최적 고정 깊이와 비교. 게이팅/예산 연구이며 검색기 학습은 아님 |
| REPLUG, Power of Noise | — | 2023–24 | 고전적 배경 연구 |

### 판정: **부분적으로 다뤄짐** (등급 유지, 열려 있는 범위는 크게 좁아짐)
- 다뤄진 부분: set-level utility를 쓰는 listwise 학습(OptiSet, SetR). **closed-book 반사실 utility 자체**도 이미 있다. OptiSet은 집합 수준 엔트로피 차 ∆H = H_p − H_o로 학습 라벨을 만들고, GainRAG는 contrastive decoding으로 closed-book 사전확률을 빼며, 2510.11358은 Acc(q,d) − Acc(q,∅)로 utility를 정의한다. 섭동 기반 dense retriever 학습(SCARLet)과 utility의 LLM 특이성 문제(2510.11358)도 다뤄졌다.
- 열려 있는 부분: (a) closed-book 차분 **set-level** utility를 **dense bi-encoder** 검색기에 학습시킨 연구는 찾지 못했다. OptiSet은 같은 신호로 8B LLM 선택기를 학습하고, dense retriever를 학습한 SCARLet·UAE는 closed-book 차분을 쓰지 않는다. (b) 여러 생성기에 걸쳐 전이되는 utility 목적함수(생성기 앙상블/불변성 정규화)와, **학습에 쓰지 않은 생성기 계열**에서의 체계적 평가는 없다. SCARLet의 2개 생성기 결과는 저자 스스로 "예비적"이라 했고, OptiSet은 이를 향후 과제로 남겼다.

### 더 날카로운 프레이밍
"Generator-invariant set utility retriever": closed-book 차분 set utility(OptiSet의 ∆H, 2510.11358의 Acc 차를 집합으로 확장)를 **여러 생성기에서 동시에** 계산하고, 생성기 간 불변 성분만 남기는 목적함수(앙상블 평균 + 분산 페널티 등)로 dense bi-encoder를 학습한다. 핵심 실험은 학습에 쓰지 않은 생성기 패밀리(크기/계열이 다른 LLM)로의 zero-shot 전이다. 차별점은 closed-book 차분이 아니라 **(i) 검색기 형태(LLM 선택기가 아닌 ANN 가능한 bi-encoder)와 (ii) held-out 생성기 전이**에 있다. OptiSet·SCARLet·UAE를 직접 baseline으로 두고, 2510.11358의 비전이성 결과를 완화하는지 검증한다.

---

## C4. 인덱스 구축 전 예측기: GraphRAG/RAPTOR/청크 크기가 이득일지 사전 예측

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| When to use Graphs in RAG (GraphRAG-Bench) | arXiv 2506.05690 / ICLR 2026(본문 헤더로 확인) | 2025 | 의료 가이드라인(NCCN)과 소설 코퍼스, 난이도별 4개 과제로 GraphRAG가 RAG를 이기는 조건을 분석. 결론은 정성 지침("tightly connected communities" 등)이며 예측 모델은 없음. 한계: 텍스트 단일 모달 |
| RAG vs GraphRAG | arXiv 2502.11371 / KDD '26 | 2025 | 공통 프로토콜로 QA·질의 기반 요약에서 RAG와 GraphRAG를 체계적으로 비교 |
| EA-GraphRAG (Use Graph When It Needs) | arXiv 2602.03578 (본문 헤더 "Under Review") | 2026 | **질의 단위**: 구문 특징 → MLP adapter 복잡도 점수 → RAG/GraphRAG/융합 라우팅. "a pre-built graph G over C"를 전제로 하며(§3.1 Offline Indexing), 코퍼스 단위·구축 전 예측은 없음. 벤치 NQ, PopQA, HotpotQA, 2Wiki, Mix |
| MoC | arXiv 2503.09600 (venue 근거 없음) | 2025 | Boundary Clarity·Chunk Stickiness로 청킹 품질을 직접 측정하고 granularity-aware mixture-of-chunkers 제안 |
| BM25 Wins at Scale | arXiv 2607.26497 (venue 근거 없음) | 2026 | 28개 중첩 규모(약 450배)로 코퍼스를 키우며 비교: 약 1천만 코퍼스 토큰에서 BM25가 File-System Agent를 추월하고, 그래프 RAG는 배포 규모 전에 구축 비용 벽에 부딪힌다. **코퍼스 크기라는 단일 코퍼스 변수에 따른 역전**을 보이지만 예측기는 아님 |
| Does Document Structure Help Dense Retrieval? | arXiv 2610.10170 (comment "Initial draft") | 2026 | Wikipedia FA 200편(951 질의)·QASPER 1,585편(4,303 질의)에서 placebo 통제 ablation. 구조 정렬 청킹 +0.022/+0.012, 단순 계층 2단계 검색 −0.033/−0.015. 계층 검색 성패를 "section inventory" 크기로 사후 설명하지만 사전 예측기는 없음 |
| AutoRAG, RankML류 메타러닝 | — | — | 파이프라인 탐색/메타러닝. RAG 인덱스 유형을 사전 예측하는 연구는 찾지 못함(본문 확인 대상 아님) |

### 판정: **열려 있음** (변경 없음; 질의 단위 라우팅은 다뤄짐, 코퍼스 단위 사전 예측은 비어 있음)
EA-GraphRAG는 이미 구축된 그래프를 전제로 한 질의 라우터이고, GraphRAG-Bench·RAG vs GraphRAG는 사후 비교와 정성 지침만 준다. 본문 확인 결과 코퍼스 수준 변수가 결과를 가른다는 **사후 증거**는 있다: 코퍼스 크기(BM25 Wins at Scale, 약 1천만 토큰 교차점), section inventory(2610.10170), 그래프 커뮤니티 밀도(GraphRAG-Bench). 그러나 엔티티 밀도·교차 문서 참조율·질의 분포 같은 값싼 통계로 "인덱스를 만들기 전에" 투자 가치를 예측하는 모델은 없다.

### 더 날카로운 프레이밍
"Index-or-not 메타 예측기": 다양한 코퍼스 수십 개에서 (vanilla, chunk 크기 3종, RAPTOR, GraphRAG, LightRAG)를 실제로 구축해 메타 데이터셋을 만든다. 코퍼스 1~5% 샘플에서 얻는 값싼 특징(엔티티 밀도, 공출현 그래프 모듈성, 질의의 다중 엔티티 비율, BM25-dense 일치도)으로 이득 부호와 비용 대비 효과를 예측한다. 이미 보고된 코퍼스 크기(BM25 Wins at Scale)와 section inventory(2610.10170)는 단일 변수 baseline으로 둔다. 평가 지표는 "회피한 인덱싱 비용 대비 손실 정확도"(regret)로 한다. EA-GraphRAG와는 "사전(pre-indexing)·코퍼스 단위"로 구분한다.

---


---

## C5. 문맥화 청킹(Contextual Retrieval, Late Chunking)의 문서 내 판별력 붕괴

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| Beyond Chunk-Then-Embed: A Comprehensive Taxonomy and Evaluation of Document Chunking Strategies for Information Retrieval | arXiv 2602.16974 (익명 심사 중, venue 근거 없음) | 2026 | 문맥화 청킹(Late Chunking)이 in-corpus(BEIR) 성능은 높이지만 **in-document(GutenQA) 검색은 모든 설정에서 떨어뜨린다**고 보고했다(평균 하락: Nomic −53.17%, Jina-v3 −29.82%, Jina-v2 −5.16%, E5-large −3.83%). 원인으로 든 "같은 문서 청크끼리 더 비슷해진다"는 가설일 뿐 측정하지 않았다. LLM 접두사 방식은 다루지 않는다 |
| Context is Gold to find the Gold Passage (ConTEB / InSeNT) | arXiv 2505.24782 (v2 "Under Review", 로컬 venue 근거 없음) | 2025 | 같은 문서 청크를 hard negative로 쓰는 in-sequence 대조 학습으로 "embedding collapse"를 막는다. 순진한 late chunking은 −21 nDCG@10이고 학습으로 +16을 회복한다. **Anthropic Contextual Retrieval을 baseline으로 평가했다**(ConTEB 평균 72.4, 120배 느림). 다만 집계 nDCG만 보고하고 문서 내 판별 지표는 없다 |
| Contextual Embeddings Beyond the Gold Chunk (pplx-embed-v2-context, Perplexity AI·turbopuffer) | arXiv 2610.38854 | 2026-10 | query-aware context compression 교사의 청크 단위 graded relevance를 forward KL로 증류하고 문서 수준 InfoNCE를 더한다. gold-chunk 라벨이 필요 없다. 새 벤치마크 context-bench와 Q2D-Chunk(in-document evidence recall 변형 포함)를 냈다. **학습 기반**이며 공통 성분 제거나 디바이어싱은 없다. Anthropic CR은 비교 대상이 아니다. (기존 표의 "Perplexity 문맥 임베딩 공개, 업계 발표" 행과 같은 작업물이다) |
| Calibrating Document-Local Boundaries for Contextual Chunk Retrieval (DcLBC, Alibaba Qwen) | arXiv 2611.12097 (venue 근거 없음) | 2026-11 | InSeNT 위에서 AMS(gold와 가장 어려운 형제 사이 margin으로 인스턴스별 scale을 정함)와 CKR(post-RoPE key에 청크 파티션 offset 주입)을 **post-training**한다. InSeNT 대비 최대 +2.45 NDCG@10. 실패 질의 6,976/19,974개 중 82.21%에서 형제 청크가 gold를 이긴다. C5의 "fix"와는 **학습 기반 해법 수준에서만** 겹친다. training-free 또는 post-hoc 공통 성분 제거는 아니다 |
| Visual Late Chunking (ColChunk) | arXiv 2604.10167 (Preprint) | 2026 | 시각 문서의 patch 임베딩을 학습 없이 계층 군집해 페이지당 약 40개 멀티벡터로 만든다(저장 90% 이상 절감, 단일 벡터 대비 nDCG@5 +9). 효율 연구이며 문서 내 판별은 다루지 않는다 |
| All-but-the-Top | arXiv 1702.01417 / ICLR 2018 | 2017 | 공통 평균 벡터와 상위 우세 방향을 제거하는 단어·문장 벡터 후처리. 청크/문서 수준 적용은 없다 |

### 판정: **부분적으로 다뤄짐** (현상 보고와 학습 기반 해법은 있음, training-free 해법은 열려 있음)
- 현상: 2602.16974(Late Chunking 한정, 메커니즘 미측정). 학습 기반 해법: ConTEB/InSeNT, 2610.38854, 2611.12097(DcLBC). 여섯 편 본문 모두 평균 제거, PC 제거, 센터링, 디바이어싱 같은 post-hoc 처리를 쓰지 않았다(본문 검색 결과 0건).
- 열려 있는 부분 1: **학습이 필요 없는(post-hoc) 문서 공통 성분 제거/디바이어싱**(문서별 평균 또는 top-PC 제거, 질의 의존 혼합 계수)으로 in-corpus 이득을 유지하면서 in-document 판별력을 회복하는 연구는 여전히 없다. 2611.12097과 2610.38854도 이 부분을 다루지 않는다는 것을 원문으로 확인했다.
- 열려 있는 부분 2: Anthropic Contextual Retrieval(LLM 접두사)은 ConTEB와 2611.12097 Table 1에서 **집계 성능으로만** 평가됐다. 같은 문서 청크 간 유사도 붕괴나 in-document 판별력 저하를 정량화한 연구는 없다.

### 더 날카로운 프레이밍
"Context without collapse": (1) 붕괴 지표를 정의한다(문서 내 청크 쌍 유사도 분포, intra-doc rank entropy, 2611.12097식 gold와 가장 어려운 형제 margin). 2602.16974가 가설로만 남긴 메커니즘을 Late Chunking, Anthropic Contextual Retrieval, ConTEB/InSeNT·DcLBC·pplx-embed-v2-context 사이에서 측정한다. (2) 학습 없는 해법으로 e_chunk − α·e_doc_common을 쓰되 α를 질의 유형(in-corpus vs in-document)에 따라 조정한다. 이때 학습 기반 InSeNT·DcLBC를 상한 baseline으로 둔다. (3) 2단계 검색(문서는 문맥화 벡터로, 문서 내 청크는 디바이어싱 벡터로)을 쓴다. 평가는 ConTEB, GutenQA, Q2D-Chunk(in-document 변형)를 사용한다.

---

## C6. 멀티벡터(ColBERT/ColPali)·양자화(PQ/RaBitQ) 인덱스의 다세대 하위호환 임베딩 업그레이드

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| Drift-Adapter | EMNLP 2025 (aclanthology 2025.emnlp-main.805) | 2025 | 새 쿼리를 레거시 공간으로 매핑해 기존 ANN(FAISS HNSW) 인덱스를 재사용한다. recall 95–99% 회복, 질의 지연 10µs 미만 증가. 단일 벡터만 다룬다. Limitations에서 **연쇄 적응(A→B→C)의 오차 누적은 연구하지 않았다**고 명시한다 |
| Query Drift Compensation | arXiv 2506.00037 / CoLLAs 2025 | 2025 | 연속 학습 중 태스크 전환마다 쿼리 drift 벡터를 추정하고 **누적합으로 옛 공간에 투영**한다. 5단계 체인(MS MARCO→NQ→HotpotQA→FEVER→FiQA)이며 nomic-embed 단일 벡터, 같은 모델의 연속 미세조정이다 |
| MKP-Adapter (Multi-modal Knowledge Preserving Adapter for Embedding BC) | arXiv 2609.16875 / ECCV 2026 | 2026 | MLLM 임베딩용 adapter-only BCT. ViDoRe 문서도 단일 이미지·단일 벡터로 취급한다(CLIP→GME) |
| RAGFlip | arXiv 2610.07266 | 2026 | 검색기 업그레이드(BM25→BGE/E5/SPLADE) 시 query 단위 negative flip을 측정한다(k=1에서 8.6–37.5%). 완화책은 고정 예산 hybrid뿐이며 호환성 학습은 다루지 않는다 |
| BCT 계열(이미지 검색) | arXiv 2201.09724 (RACT, ICLR 2022), 2303.04766 (FastFill, ICLR 2023), 2301.03767 (online backfilling) | 2022–23 | 하위호환/backfilling 고전 연구. 모두 단일 벡터다. RACT는 §5.4에서 **old→1→2→3세대 연속 업그레이드**를 실험했다 |
| Binary Embedding Retrieval at Tencent (BEBR) | arXiv 2302.08714 | 2023 | 학습형 recurrent 이진화 + 하위호환 학습으로 여러 임베딩 버전을 한 시스템에서 색인한다. 양자화와 호환성을 함께 다룬 드문 사례지만, 단일 벡터이고 양자화도 PQ/RaBitQ 코드북이 아니다 |
| ColBERTSaR | arXiv 2606.05568 / SIGIR 2026 short | 2026 | 잔차 없는 PQ로 ColBERT 인덱스를 inverted index로 바꾼다(1-bit PLAID 대비 50–70% 작음). 업그레이드는 다루지 않는다 |

### 판정: **열려 있음** (단, 범위를 좁혀야 함)
- 다세대 체인 자체는 선행 연구가 있다. RACT(3세대, 이미지)와 QDC(5단계 연속 학습, 텍스트)가 다뤘으므로 "다세대"만으로는 새롭지 않다. 둘 다 단일 벡터다.
- 열린 부분은 다음과 같다. **멀티벡터 late-interaction 인덱스**(토큰 단위 MaxSim 공간, PLAID centroid/residual 코드)와 **PQ/RaBitQ 코드북 인덱스**에서 하위호환 업그레이드를 다룬 연구는 없다. 아홉 편 본문 모두 이 두 경우를 다루지 않는다. 이종 모델 체인(서로 다른 아키텍처 v1→v2→v3)에서 adapter 연쇄의 오차 누적도 Drift-Adapter 저자가 미해결로 명시했다.

### 더 날카로운 프레이밍
"Compatible late interaction": (a) 토큰 수준 어댑터(query token → legacy token space)를 MaxSim 보존 손실로 학습한다. (b) PLAID centroid·PQ 코드북을 고정한 채 새 모델의 잔차만 재계산하는 "codebook-anchored BCT"를 제안한다. (c) v1→v2→v3 체인에서 호환성 오차 누적과 negative flip(RAGFlip 지표 차용)을 평가한다. baseline은 세 가지다: 단일 벡터 Drift-Adapter를 멀티벡터에 순진하게 확장한 방식, QDC식 drift 누적합, RACT식 연속 BCT.

---

## C7. 검색 여부·질의 확장 유형·k·압축률을 하나의 토큰/지연 예산 아래 결합하고 conformal 보장을 주는 통합 정책

### 가장 가까운 기존 연구
| 연구 | ID | 연도 | 요약 |
|---|---|---|---|
| Know Before You Fetch | arXiv 2606.29959 | 2026 | 0/1/5 passage 중에서 closed-book **신뢰도 threshold**로 고르고, 평균 passage cap과 probe 지연 아래에서 fixed/random-mixture 정책과 비교한다. abstain 행동은 없다. 보정(calibration)은 결정을 바꾸지 않았고(§5.5), pilot 기반 선택 규칙도 위반률을 보장하지 않는다고 명시한다. 압축·질의 확장은 없다 |
| When Should Active RAG Retrieve? | arXiv 2607.24010 / KDD 2026 Workshop | 2026 | 예산 인지 평가 프로토콜(실현 사용률, threshold 전이 오차)을 제안한다. 단순 불확실성 baseline이 학습 라우터와 비슷하다. **DKW형 slack으로 실현 사용률 위반을 줄이는 budget-safe threshold**도 제안했지만, 이진(검색/비검색) 행동에 한정되고 경험적으로만 다룬다. 향후 과제로 "검색 시점과 컨텍스트 압축이 함께 예산을 결정하는 경우"를 든다 |
| When Knowledge Is Not Free | arXiv 2606.02245 | 2026 | 접근 비용 tier가 다른 증거를 예산 아래에서 선택한다. 고정 selector 중 우월한 것이 없고, LLM 에이전트 컨트롤러는 모델·과제 의존적이다 |
| Adaptive Retrieval, Selective Answers | arXiv 2610.34872 | 2026 | 에이전트형 RAG에서 생성 **후** claim 필터링에 conformal factuality를 적용한다. 검색 비용은 고정(최대 3회)이고 보장 대상이 아니다 |
| Principled Context Engineering for RAG (conformal filtering) | arXiv 2511.17908 / ECIR 2026 | 2025 | 검색 후 conformal 필터로 **관련 snippet coverage**를 보장한다. 컨텍스트 2–3배 감소는 결과일 뿐 보장 대상이 아니다 |
| TRAQ | arXiv 2307.04642 / NAACL 2024 | 2024 | 검색 집합과 생성 집합의 conformal 보장을 union bound로 결합한다. Bayesian optimization으로 집합 크기를 줄이지만(평균 −16.2%) 크기를 보장하지는 않는다 |
| C-RAG: Certified Generation Risks for RAG | arXiv 2402.03181 / ICML 2024 | 2024 | RAG 설정 λ=[N_rag, λ_g, λ_s]에 대해 conformal generation risk 상한을 주고, FWER 다중검정(LTT류)으로 위험 ≤ α인 valid configuration set을 낸다. 비용 제약은 없고 전역 설정이며 질의별 정책이 아니다 |
| AdaMem | arXiv 2609.22100 | 2026 | 고정 memory-token 예산을 passage 관련도에 따라 할당한다(soft compression) |
| TARG | arXiv 2511.09803 / TMLR 2026 | 2025 | 짧은 draft의 prefix logit 불확실성으로 하는 학습 없는 검색 게이팅. 검색 70–90% 감소. 보장은 없다 |
| Adaptive compression for edge RAG | arXiv 2608.19535 / ACM AI Leadership Summit 2026 | 2026 | Jetson에서 압축률 trade-off를 실측하고 telemetry 기반 런타임 압축 제어를 제안하는 **비전 논문**이다. controller는 구현하지 않았다 |
| UR² | arXiv 2508.06165 / ACL 2026 (Oral) | 2025 | RL로 검색과 추론을 조정한다(difficulty-aware curriculum, LLM 요약 corpus). 예산 통합 정책이나 보장은 없다 |
| AdaRankLLM (Rethinking the Necessity of Adaptive RAG…) | arXiv 2604.15621 | 2026 | zero-shot listwise ranker + passage dropout으로 k/pruning을 적응시킨다. ASQA/QAMPARI/ELI5 |
| Provence | arXiv 2501.16214 | 2025 | (호출자 제공) context pruning. 본문 확인 불가 |

### 판정: **부분적으로 다뤄짐** (차별점이 좁아짐)
- 다뤄진 부분: 검색 여부 + k의 신뢰도 기반 결합(2606.29959). 이진 gating의 **실현 예산** 보수적 보정(2607.24010, DKW slack, 경험적). 위험 ≤ α를 만족하는 RAG 설정 집합의 LTT류 다중검정(C-RAG). 컨텍스트 coverage 보장(2511.17908, TRAQ). 압축 예산 할당(AdaMem).
- 열려 있는 부분: 네 결정(검색 여부 × 질의 확장 유형 × k × 압축률)을 **질의별 조합 행동 공간**에서 고르면서 **오류 위험과 실현 토큰/지연 비용을 동시에** 분포 무관하게 보장하는 연구는 없다(다중 제약 LTT / conformal risk control). 12편 본문 중 질의 확장을 행동으로 넣은 연구는 없다. 2607.24010 저자도 압축과 검색 시점의 결합을 향후 과제로 남겼다.

### 더 날카로운 프레이밍
"Certified RAG budget controller": 행동 공간을 조합 격자로 두고, 오류 위험 ≤ ε와 **실현** 토큰 비용 ≤ B를 동시에 보장하는 LTT 기반 다중 제약 보정을 설계한다. 예산만 보장하는 것(2607.24010의 이진 DKW 보정)이나 위험만 보장하는 전역 설정 선택(C-RAG)은 이미 있다. 따라서 핵심 차별점은 **두 제약을 하나의 질의별 조합 정책에서 함께 보장하는 것**이다. 비교 baseline은 2607.24010 budget-safe threshold, C-RAG valid set에서 가장 싼 설정 고르기, 2606.29959의 fixed/random-mixture 정책이다. 확장/압축 축이 k·게이팅 축과 상호작용함을 보이는 ablation도 필요하다(예: 압축하면 큰 k가 유리해지는 현상).

---

## 요약표
| 후보 | 판정 | 가장 많이 겹치는 연구 |
|---|---|---|
| C1 | 부분적으로 다뤄짐 | adRAP 2410.01736, Stashbird 2609.34242, RAGU 2607.11683 |
| C2 | 열려 있음 | 2606.01074, SMEC 2510.12474, LIMIT 2508.21038 |
| C3 | 부분적으로 다뤄짐 | OptiSet 2601.05027, 2510.11358, SCARLet 2504.00573 |
| C4 | 열려 있음(코퍼스 단위) | EA-GraphRAG 2602.03578, GraphRAG-Bench 2506.05690 |
| C5 | 부분적으로 다뤄짐(현상·학습 해법은 있음, training-free 해법과 CR 붕괴 정량화는 열려 있음) | 2602.16974, ConTEB 2505.24782, DcLBC 2611.12097, pplx-embed-v2-context 2610.38854 |
| C6 | 열려 있음(멀티벡터·PQ/RaBitQ 한정, 단일 벡터 다세대 체인은 RACT·QDC가 다룸) | Drift-Adapter, QDC 2506.00037, RACT 2201.09724, MKP-Adapter 2609.16875, RAGFlip 2610.07266 |
| C7 | 부분적으로 다뤄짐(오류+실현 예산 동시 보장과 조합 행동 공간은 열려 있음) | 2606.29959, 2607.24010, C-RAG 2402.03181, 2511.17908 |

주의: C5–C7에 인용한 연구는 모두 원문(로컬 본문)으로 확인했다. 예외는 Provence(본문 확인 불가)이다.
