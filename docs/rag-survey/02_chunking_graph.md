# 문헌 조사 B: RAG 검색·인덱싱 측면 (A. 청킹/인덱스 구조, B. 그래프/구조화 인덱스)

작성일: 2026-11-19

> **검증 방법에 관한 주의**: 이 세션에서는 arxiv.org, aclanthology.org, huggingface.co에 대한 WebFetch가 네트워크 정책으로 차단되었습니다(ENOTFOUND). 그래서 모든 항목은 **WebSearch 결과**로 확인했습니다. 확인한 것은 arXiv/학회 페이지의 제목·저자·ID·venue, 그리고 초록 스니펫과 2차 요약입니다. 논문 본문 PDF는 직접 열지 못했습니다.
> - 표의 수치는 초록이나 검색 스니펫에 나온 값만 옮겼습니다.
> - 본문 표에서 확인해야 하는 수치와 venue는 "미확인"으로 표시했습니다.
> - "Stated limitations" 열은 저자가 밝힌 한계가 검색으로 확인된 경우에만 따옴표 취지로 적었습니다. 그 밖의 내용은 **[관찰]**(조사자의 해석)로 구분했습니다.

---

## 축 A. 인덱싱 & 청킹 구조

### A-1. 현황 요약
청킹은 "고정 길이냐 의미 단위냐"의 문제에서 출발했습니다. 지금은 두 방향이 경쟁합니다.
- **청크에 문맥을 되돌려 주는 방식**: Late Chunking, Anthropic Contextual Retrieval.
- **검색 단위 자체를 바꾸는 방식**: 명제 단위인 Dense X, 4K 장단위인 LongRAG, 요약 트리인 RAPTOR와 SiReRAG.

대규모 통제 비교 연구(Vectara 2024, Zhou et al. 2026, Shaukat et al. 2026)의 결론은 일관됩니다. "만능 청커는 없으며 과제(in-corpus vs in-document)와 도메인에 따라 최적이 다르다"는 것입니다. 비싼 semantic/LLM 청킹의 이득은 불안정하다는 결과도 반복됩니다.

장문맥 LLM과 RAG의 관계는 "대체" 논쟁에서 "라우팅/하이브리드" 논쟁으로 넘어갔습니다(Self-Route, LaRA). 2025~2026년의 새 흐름은 세 가지입니다.
- 문서 헤딩 계층을 그대로 인덱스로 쓰는 구조 인식 청킹/라우팅(HiChunk, RDR2, MultiDocFusion)
- 청킹 품질의 내재적 지표(MoC의 BC/CS)
- 메모리형 인덱스(MemoRAG, A-Mem, HippoRAG 2)

### A-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Dense X Retrieval: What Retrieval Granularity Should We Use?](https://arxiv.org/abs/2312.06648) | 2023 arXiv → EMNLP 2024 Main | 검색 단위로 "명제(proposition)", 즉 원자적·자기완결적 사실 단위를 제안. FactoidWiki를 구축하고 passage/sentence/proposition 단위를 비교 | 5개 open-domain QA 데이터셋, 4개 dual-encoder | 미확인. [관찰] 명제 추출에 LLM 비용이 들고, 명제 간 담화 문맥이 소실됨 |
| [RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval](https://arxiv.org/abs/2401.18059) | ICLR 2024 | 청크를 임베딩→클러스터링→요약하는 과정을 재귀적으로 반복해 트리를 만들고, 여러 추상 수준에서 검색 | QuALITY(GPT-4와 결합 시 기존 최고 대비 +20%p 절대), NarrativeQA, QASPER | 미확인. [관찰] 정적 코퍼스를 가정하며, 문서가 추가될 때 트리 재구축 비용이 큼 |
| [Financial Report Chunking for Effective Retrieval Augmented Generation](https://arxiv.org/abs/2402.05131) | 2024 arXiv (venue 미확인) | 제목·표 등 문서 요소 경계에 맞춰 청킹하는 element-based 방식. chunk size 튜닝이 필요 없음을 주장 | FinanceBench (검색 정확도 최대 84.4%) | 회사명·연도 같은 메타데이터가 청킹 과정에서 소실되는 문제를 언급 |
| [LumberChunker: Long-Form Narrative Document Segmentation](https://arxiv.org/abs/2406.17526) | Findings of EMNLP 2024 | LLM이 연속 문단을 읽고 내용이 바뀌는 지점을 골라 청크 경계를 정함. GutenQA 벤치마크 공개 | GutenQA(Gutenberg 도서 100권, QA 3,000쌍). 최강 baseline 대비 DCG@20 +7.37% | LLM 호출 때문에 비용·지연이 큼(2차 요약 기준) |
| [LongRAG: Enhancing Retrieval-Augmented Generation with Long-context LLMs](https://arxiv.org/abs/2406.15319) | 2024 arXiv (venue 미확인) | 약 4K 토큰 단위로 재구성해 Wikipedia 검색 단위를 22M개에서 700K개로 줄임. 검색기는 가볍게, 리더(장문맥 LLM)는 무겁게 하는 "long retriever + long reader" 구조 | NQ EM 62.7, HotpotQA full-wiki 64.3, Qasper F1 25.9, MultiFieldQA-en 57.5 | 미확인. 버전(v1~v3)마다 수치가 다름 |
| [Retrieval Augmented Generation or Long-Context LLMs? A Comprehensive Study and Hybrid Approach (Self-Route)](https://arxiv.org/abs/2407.16833) | EMNLP 2024 Industry | 자원이 충분하면 LC(장문맥)가 평균적으로 RAG보다 우세함을 보임. LLM이 스스로 "답 가능 여부"를 판단해 RAG와 LC 중 하나로 라우팅 | LongBench, ∞Bench 계열 (세부 미확인). Gemini-1.5-Pro 비용 −65%, GPT-4o 비용 −39% | 미확인 |
| [Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models](https://arxiv.org/abs/2409.04701) | 2024 arXiv (Jina AI. venue 미확인) | 문서 전체를 장문맥 임베딩 모델에 통과시킨 뒤 토큰 임베딩을 청크별로 mean-pool. 추가 학습이 필요 없음. long late chunking과 fine-tuning 확장 제안 | BEIR 일부 (세부 미확인) | 임베딩 모델의 입력 길이를 넘는 문서는 별도 처리(long late chunking)가 필요 |
| [MemoRAG: Boosting Long Context Processing with Global Memory-Enhanced Retrieval Augmentation](https://arxiv.org/abs/2409.05591) | WWW 2025 | 경량 장문맥 모델이 코퍼스 전체의 "global memory"를 만들고, 거기서 생성한 draft clue를 검색 질의로 사용하는 이중 시스템 | 세부 미확인 (UltraDomain 등으로 알려져 있으나 본문 미확인) | 미확인 |
| [Anthropic — Introducing Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) | 2024 산업 블로그 | 각 청크 앞에 LLM이 만든 50–100토큰 문맥 설명을 붙인 뒤 임베딩과 BM25 인덱싱. 프롬프트 캐싱으로 비용 절감 | 자체 데이터. top-20 검색 실패율 5.7%→2.9%(−49%), 리랭킹 추가 시 1.9%(−67%) | 자체 평가 결과이며 독립 재현은 제한적. 일반 문서 요약을 붙이는 방식은 이득이 거의 없었다고 언급 |
| [Is Semantic Chunking Worth the Computational Cost?](https://arxiv.org/abs/2410.13070) | 2024 arXiv (Vectara. NAACL 2025 Findings라는 2차 정보 있음, 미확인) | 고정 크기, breakpoint 기반, 클러스터 기반 청커를 문서 검색·증거 검색·답 생성 세 과제에서 비교 | 인공으로 이어 붙인(stitched) 데이터셋과 실제 데이터셋 | semantic 청킹의 이득은 일관되지 않음. stitched 데이터에서만 유리함. 임베딩 품질 같은 다른 요인의 영향이 더 큼 |
| [Long-Context LLMs Meet RAG: Overcoming Challenges for Long Inputs in RAG](https://arxiv.org/abs/2410.05983) | ICLR 2025 | 검색 문서 수를 늘리면 성능이 올랐다가 떨어지는 역U자 현상을 보이고, 원인을 hard negative로 규명. retrieval reordering과 robustness fine-tuning 제안 | NQ, TriviaQA, HotpotQA 등 (세부 미확인) | 강한 검색기일수록 hard negative를 더 많이 가져옴. 즉 precision만으로는 검색 품질을 판단하기 어려움 |
| [SiReRAG: Indexing Similar and Related Information for Multihop Reasoning](https://arxiv.org/abs/2412.06206) | ICLR 2025 | 유사성 트리(RAPTOR식)와 관련성 트리(명제·엔티티 기반)를 함께 구축하고 평탄화해 하나의 검색 풀로 사용 | MuSiQue, 2Wiki, HotpotQA. 평균 F1 +1.9%, 리랭킹과 결합 시 최대 +7.8% | 미확인. [관찰] 두 트리 모두 LLM 요약이 필요해 인덱싱 비용이 큼 |
| [LaRA: Benchmarking Retrieval-Augmented Generation and Long-Context LLMs – No Silver Bullet for LC or RAG Routing](https://arxiv.org/abs/2502.09977) | ICML 2025 | RAG와 LC를 비교하는 벤치마크. 기존 비교의 설계 결함을 지적. 최적 선택은 모델 능력, 문맥 길이, 과제 유형, 검색 특성에 따라 달라짐 | 2,326개 케이스. 소설·논문·재무제표. LLM 11종 | 단일 라우팅 규칙은 없다는 결론 |
| [MoC: Mixtures of Text Chunking Learners for Retrieval-Augmented Generation System](https://arxiv.org/abs/2503.09600) | ACL 2025 Long | 청킹 품질을 직접 재는 지표 Boundary Clarity와 Chunk Stickiness 제안. granularity router와 경량 청킹 전문가를 조합한 MoC | 다수 QA 데이터셋 (세부 미확인) | 미확인 |
| [Reconstructing Context: Evaluating Advanced Chunking Strategies for RAG](https://arxiv.org/abs/2504.19754) | ECIR 2025 Workshop (KEIR) | Late Chunking과 Contextual Retrieval 비교 재현 | NFCorpus 등 | Contextual Retrieval은 응집성이 높지만 계산 비용이 큼. Late Chunking은 효율적이지만 관련성과 완결성을 일부 희생 |
| [HiChunk (Tencent Youtu)](https://arxiv.org/abs/2509.11552) | 2025 arXiv (venue 미확인) | fine-tune한 Qwen3-4B로 청킹 경계와 그 계층 레벨을 함께 예측. 계층 청킹 평가용 HiCBench 공개 | HiCBench 외 (미확인) | 기존 청커가 선형 구조만 다룬다고 지적 |
| [RDR2 (구조 라우팅)](https://arxiv.org/abs/2510.04293) | Findings of EMNLP 2025 (검색 결과 기준) | 문서 헤딩 트리 위에서 LLM 라우터가 Answer/Expand/Refuse를 결정하는 학습 가능한 구조 탐색 | 미확인 | 미확인 |
| [MultiDocFusion: Hierarchical and Multimodal Chunking Pipeline for Enhanced RAG on Long Industrial Documents](https://preview.aclanthology.org/credits/2025.emnlp-main.1062) | EMNLP 2025 Main | 레이아웃 검출, OCR, LLM 섹션 계층 파싱, DFS 기반 계층 청크를 잇는 파이프라인 | 산업 문서 (미확인) | 미확인 |
| [Beyond Chunk-Then-Embed: A Comprehensive Taxonomy and Evaluation of Document Chunking Strategies for IR](https://arxiv.org/abs/2602.16974) | 2026 arXiv (Zuccon 그룹) | 청킹 분류 체계와 재현 평가. in-corpus에서는 구조 기반 청킹이, in-document에서는 LumberChunker가 우세 | BEIR 6종, GutenQA. 임베딩 4종 | 문맥화 청킹은 in-corpus 성능을 올리지만 in-document 판별력을 떨어뜨림 |
| [A Systematic Investigation of Document Chunking Strategies and Embedding Sensitivity](https://arxiv.org/abs/2603.06976) | 2026 arXiv | 청킹 전략 36종 × 도메인 6개 × 임베딩 5종, 총 1,080개 구성의 통제 비교 | UltraDomain | 청크를 잘게 나누면 인덱스 크기와 지연이 증가 |

### A-3. 반복되는 미해결 문제, 암묵적 가정, 벤치마크 공백
1. **"최적 청킹은 과제에 따라 다르다"는 결론은 반복되지만, 과제에 맞춰 청킹을 고르는 원리는 없습니다.** Vectara 2024, Zhou et al. 2026, Shaukat et al. 2026, LaRA가 이를 뒷받침합니다. MoC의 router는 granularity만 고르고, 질의 분포나 코퍼스 특성에 조건화되지 않습니다.
2. **평가 데이터의 인공성.** semantic 청킹은 stitched(인공 연결) 데이터에서만 이깁니다(Vectara). 그래서 경계 검출 능력과 실제 검색 효용이 혼동됩니다. GutenQA처럼 서사 텍스트에 치우친 벤치마크도 있습니다.
3. **정적 코퍼스 가정.** RAPTOR, SiReRAG, MemoRAG, Contextual Retrieval은 모두 문서 단위나 코퍼스 단위로 한 번 전처리합니다. 문서가 수정되면 요약·문맥 접두사·트리의 어느 부분이 무효가 되는지 다루지 않습니다.
4. **문맥 주입의 부작용.** 문맥화 청킹은 in-document 판별력을 떨어뜨립니다(Zhou et al. 2026). 같은 문서의 청크들이 공통 접두사 때문에 임베딩 공간에서 서로 가까워질 것으로 [관찰] 추정되며, 이 trade-off를 정량화한 연구는 없습니다.
5. **LC vs RAG 비교가 일정 시점의 모델에 묶여 있음.** Self-Route(2024)는 "LC 우세"를, LaRA(2025)는 "no silver bullet"을 결론으로 냅니다. 모델 세대에 따라 결론이 바뀌고, 비용(토큰·지연)을 반영한 Pareto 비교는 드뭅니다.
6. **청킹 품질 지표와 다운스트림 성능의 상관이 검증되지 않음.** MoC의 BC/CS 외에는 내재적 지표가 거의 없습니다. 이 지표들이 다른 코퍼스나 검색기에서도 다운스트림 성능을 예측하는지는 미확인입니다.

### A-4. 덜 탐구된 연구 공백 (구체안)
1. **질의 분포 조건부 청킹/인덱스 선택(chunking-as-a-policy).**
   - 내용: 운영 로그의 질의 분포(사실형, 집계형, 문서 내 탐색형)를 보고 문서별 granularity나 인덱스 유형(명제, 문단, 4K, 요약 트리)을 고르거나 혼합하는 학습 정책입니다.
   - 가장 가까운 연구: MoC(granularity router), StructRAG(구조 유형 router, 추론 시점), Self-Route(LC/RAG 라우팅).
   - 왜 미해결인가: 모두 질의 시점 라우팅이거나 텍스트 내재 특성만 봅니다. 인덱싱 시점에 예상 질의 분포에 맞춰 최적화하지 않으며, 비용 제약도 넣지 않습니다.
2. **문맥화 청킹의 "in-document 판별력 붕괴" 측정과 완화.**
   - 내용: 공통 문맥 접두사 때문에 같은 문서 청크들이 비슷해지는 현상을 정량화하고, 문서 공통 성분을 빼는 debiasing이나 청크 고유 성분 강조 같은 기법을 만드는 것입니다.
   - 가장 가까운 연구: Zhou et al. 2026(현상 보고), Late Chunking, Anthropic Contextual Retrieval.
   - 왜 미해결인가: 현상은 보고되었지만 메커니즘 분석과 해결책이 없습니다.
3. **증분·수정에 강한 계층 요약 인덱스.**
   - 내용: RAPTOR/SiReRAG식 트리에서 문서 추가·삭제·수정이 일어날 때 무효화되는 요약 노드만 국소적으로 다시 만드는 방법, 그리고 품질 drift를 평가하는 프로토콜입니다.
   - 가장 가까운 연구: EraRAG(LSH 기반으로 그래프 삽입만 지원), LightRAG(삽입만 지원).
   - 왜 미해결인가: 수정과 삭제, 그리고 요약의 일관성(오래된 사실이 요약에 남는 문제)을 다루는 평가가 없습니다.
4. **비용을 반영한 LC/RAG/구조 인덱스 3자 Pareto 벤치마크.**
   - 내용: 같은 코퍼스에서 인덱싱 비용(LLM 토큰), 질의 비용, 지연, 정확도를 함께 보고하는 표준 프로토콜입니다. 모델 세대가 바뀔 때 재측정할 수 있어야 합니다.
   - 가장 가까운 연구: LaRA(비용 축 약함), Self-Route(비용 보고는 있으나 구조 인덱스 없음), "BM25 Wins at Scale"(2026, 축 B 참고).
5. **레이아웃·구조 기반 인덱스의 실패 모드 연구.**
   - 내용: 헤딩 계층이 깨지거나 거짓인 문서(스캔 PDF, 슬라이드, 웹 보일러플레이트)에서 구조 인식 청킹이 얼마나 견고한지 보는 연구입니다.
   - 가장 가까운 연구: HiChunk/HiCBench, RDR2, MultiDocFusion, Financial Report Chunking.
   - 왜 미해결인가: 대부분 "구조가 올바르게 파싱된다"고 가정합니다. 파싱 오류가 검색 오류로 전파되는 과정의 분석은 2026년 재무 PDF 파싱 연구([2604.12047](https://arxiv.org/abs/2604.12047), 세부 미확인) 정도에 그칩니다.

---

## 축 B. 그래프 / 구조화 인덱스

### B-1. 현황 요약
2024년에 Microsoft GraphRAG(엔티티 KG와 커뮤니티 요약)가 "글로벌 sensemaking 질의"라는 새 과제 범주를 열었습니다. 이어 LightRAG(이중 수준 검색, 증분 삽입), HippoRAG(PPR 기반 단일 단계 multi-hop), G-Retriever/GRAG(텍스트 그래프와 GNN soft prompt)가 방법론의 큰 갈래를 만들었습니다.

2025년의 주제는 **비용 절감과 노이즈 제거**였습니다. KET-RAG, LazyGraphRAG, ArchRAG, PathRAG, LinearRAG가 여기에 속합니다. 동시에 **비판적 재평가**가 대거 나왔습니다. RAG vs GraphRAG, GraphRAG-Bench, Unbiased Eval, Unified Framework(VLDB) 같은 연구가 그 예입니다. 공통 결론은 다음과 같습니다.
- 단일 사실 질의에서는 vanilla RAG가 동등하거나 우세합니다.
- 그래프는 multi-hop과 요약에서 조건부로 이득을 줍니다.
- 기존에 보고된 이득의 상당 부분은 LLM-judge 편향(위치, 길이, 시행 편향) 때문입니다.

2026년에는 실제 웹 소스를 쓰는 노이즈 벤치마크(WildGraphBench)와 스케일 연구가 나왔습니다. 스케일 연구는 약 10M 토큰 이상에서 BM25가 우세하다고 보고했습니다(BM25 Wins at Scale). 이로써 "인덱싱 비용 대비 이득"이 핵심 쟁점이 되었습니다. 표와 코드 영역은 각각 독립된 벤치마크 생태계(TARGET, CoIR, CodeRAG-Bench)가 있습니다. 반면 저장소 그래프(RepoGraph) 같은 구조 인덱스와 텍스트 GraphRAG 연구는 거의 교류하지 않습니다.

### B-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [From Local to Global: A GraphRAG Approach to Query-Focused Summarization](https://arxiv.org/abs/2404.16130) | 2024 arXiv (v2 2025-02) | LLM으로 엔티티 KG를 만들고 커뮤니티별 요약을 미리 생성한 뒤, map-reduce 방식으로 글로벌 질의에 답함 | 약 1M 토큰 코퍼스 2종(팟캐스트, 뉴스). LLM-judge로 포괄성·다양성 평가 | directness(짧고 직접적인 답) 기준에서는 vector RAG가 우세. [관찰] 인덱싱 비용이 매우 큼 |
| [G-Retriever: Retrieval-Augmented Generation for Textual Graph Understanding and Question Answering](https://arxiv.org/abs/2402.07630) | NeurIPS 2024 | 텍스트 그래프에서 PCST(Prize-Collecting Steiner Tree)로 부분그래프를 검색하고 GNN soft prompt로 LLM에 전달. GraphQA 벤치마크 | ExplaGraphs, SceneGraphs, WebQSP | 미확인. 환각 −54% 수치는 2차 요약 출처 |
| [HippoRAG: Neurobiologically Inspired Long-Term Memory for LLMs](https://arxiv.org/abs/2405.14831) | NeurIPS 2024 | OpenIE로 KG를 만들고 질의 개념에서 Personalized PageRank를 돌려 단일 단계로 multi-hop 검색 | MuSiQue, 2Wiki, HotpotQA. IRCoT 대비 10–20배 저렴, 6–13배 빠름. 최대 +20% | 미확인. [관찰] OpenIE 품질에 의존 |
| [GRAG: Graph Retrieval-Augmented Generation](https://arxiv.org/abs/2405.16506) | Findings of NAACL 2025 | k-hop ego-graph를 검색해 병합하고 soft pruning으로 불필요한 노드를 약화. 텍스트 view와 그래프 view(soft prompt)를 함께 사용 | WebQSP, ExplaGraphs 등 (미확인) | 정답과 무관한 노드·엣지가 남는 문제를 soft pruning으로 완화한다고 명시 |
| [Graph Retrieval-Augmented Generation: A Survey](https://arxiv.org/abs/2408.08921) | 2024 arXiv (survey) | GraphRAG를 G-Indexing, G-Retrieval, G-Generation의 3단계로 정리 | 해당 없음 | — |
| [LightRAG: Simple and Fast Retrieval-Augmented Generation](https://arxiv.org/abs/2410.05779) | 2024 arXiv (v3 2025-04. EMNLP 2025 Findings라는 정보 있으나 미확인) | 그래프+벡터 인덱스에서 저수준(엔티티)과 고수준(주제) 키워드로 이중 검색. 그래프 재구축 없이 증분 삽입 | UltraDomain (Agriculture/CS/Legal/Mix). LLM-judge 승률 | 미확인. [관찰] 삭제와 수정은 지원하지 않음(Preprints.org 2025 논문이 지적) |
| [RepoGraph: Enhancing AI Software Engineering with Repository-level Code Graph](https://arxiv.org/abs/2410.14684) | ICLR 2025 | 코드 줄 단위 노드와 정의-참조 엣지로 저장소 그래프를 만들어 SWE 에이전트에 플러그인 | SWE-bench(4개 프레임워크 모두 향상), CrossCodeEval | 미확인 |
| [StructRAG: Boosting Knowledge Intensive Reasoning of LLMs via Inference-time Hybrid Information Structurization](https://arxiv.org/abs/2410.08815) | ICLR 2025 | 추론 시점에 router(DPO로 학습)가 표·그래프·알고리즘·카탈로그·청크 중 하나를 골라 문서를 재구조화한 뒤 추론 | Loong 등 (미확인) | 미확인. [관찰] 질의마다 구조화 비용이 발생 |
| [CoIR: A Comprehensive Benchmark for Code Information Retrieval Models](https://arxiv.org/abs/2407.02883) | ACL 2025 Long | 10개 코드 검색 데이터셋. MTEB/BEIR 스키마와 호환 | 자체 데이터 | SOTA 모델도 코드 검색에서 고전함 |
| [CodeRAG-Bench: Can Retrieval Augment Code Generation?](https://arxiv.org/abs/2406.14497) | Findings of NAACL 2025 | 약 9k 코딩 과제와 25M 검색 문서(튜토리얼, 문서, SO, GitHub). 검색기 10종 × LM 10종 | 기초, open-domain, 저장소 수준 과제 | 검색기가 유용한 문맥을 자주 놓치고, 생성기도 주어진 문맥을 잘 쓰지 못함 |
| [KET-RAG: A Cost-Efficient Multi-Granular Indexing Framework for Graph-RAG](https://arxiv.org/abs/2502.09304) | 2025 arXiv (KDD 2025 여부 미확인) | KNN 그래프의 PageRank로 핵심 청크만 골라 KG 골격을 만들고, 나머지는 텍스트-키워드 이분 그래프로 처리 | multi-hop QA (미확인). GraphRAG 대비 인덱싱 비용 10배 이상 절감, 생성 품질 최대 +32.4% | 미확인 |
| [ArchRAG: Attributed Community-based Hierarchical RAG](https://arxiv.org/abs/2502.09891) | 2025 arXiv (v3 2025-08) | 속성 기반 커뮤니티로 계층 인덱스를 만들고 C-HNSW로 계층 간 ANN 검색 | specific/abstract QA. GraphRAG 대비 토큰 최대 250배 절감, 정확도 +10% | 미확인 |
| [RAG vs. GraphRAG: A Systematic Evaluation and Key Insights](https://arxiv.org/abs/2502.11371) | 2025 arXiv (v3 2026-03) | 같은 프로토콜로 비교. RAG는 단일 hop과 세부 질의에, GraphRAG는 multi-hop과 다양성 요약에 유리하며 둘은 상호보완적. Selection/Integration 하이브리드 | NQ, HotpotQA, MultiHop-RAG, 요약 과제. 인덱싱 시간 KG-GraphRAG 약 7,702s vs RAG 135s(2차 요약) | LLM-judge의 순서에 따라 결과가 바뀜 |
| [From RAG to Memory: Non-Parametric Continual Learning for LLMs (HippoRAG 2)](https://arxiv.org/abs/2502.14802) | ICML 2025 | PPR에 passage 노드를 깊게 통합하고 LLM 필터링(recognition memory)을 추가. 사실, sense-making, 연상 기억 과제를 모두 다룸 | NQ, PopQA, MuSiQue, 2Wiki, HotpotQA, LV-Eval, NarrativeQA. 연상 과제에서 SOTA 임베딩 대비 +7% | 미확인 |
| [PathRAG: Pruning Graph-based RAG with Relational Paths](https://arxiv.org/abs/2502.14902) | AAAI 2026 | 문제는 정보 부족이 아니라 중복이라고 진단. flow 기반 pruning으로 핵심 관계 경로만 골라 신뢰도 오름차순으로 프롬프트에 배치 | 데이터셋 6종 × 평가 차원 5개 (UltraDomain 계열, LLM-judge) | 인덱스 그래프 구축 자체는 다루지 않음. 경로 검색에 학습이 없어 성능이 제한될 수 있음 |
| [In-depth Analysis of Graph-based RAG in a Unified Framework](https://arxiv.org/abs/2503.04338) | PVLDB 18(13), 2025 | 그래프 구축, 인덱스, operator, 검색·생성의 4단계 통합 프레임워크로 12개 방법 비교. 새 조합 변형 제안 | specific QA와 abstract QA | 미확인 |
| [How Significant Are the Real Performance Gains? An Unbiased Evaluation Framework for GraphRAG](https://arxiv.org/abs/2506.06331) | 2025 arXiv | LLM-judge의 위치·길이·시행 편향을 제거한 뒤 재평가. GraphRAG의 이득이 크게 줄거나 사라짐 | 3개 GraphRAG(LightRAG 포함) vs NaiveRAG | 자신의 프레임워크도 결함이 있을 수 있다고 명시 |
| [When to use Graphs in RAG: A Comprehensive Analysis for Graph RAG (GraphRAG-Bench)](https://arxiv.org/abs/2506.05690) | ICLR 2026 (arXiv v3 기준) | 사실 검색, 복합 추론, 문맥 요약, 창의 생성의 난이도 계층 과제와 구조화 수준이 다른 코퍼스 | GraphRAG-Bench | GraphRAG가 실제 과제에서 vanilla RAG보다 못한 경우가 많다는 점이 출발점 (예: NQ −13.4%, Han et al. 인용) |
| [TARGET: Benchmarking Table Retrieval for Generative Tasks](https://arxiv.org/abs/2505.11545) | 2025 arXiv (NeurIPS 2024 워크숍 페이지 존재, 세부 미확인) | QA, 사실 검증, text-to-SQL 하류 과제로 표 검색 평가 | 여러 표 데이터셋 | dense가 BM25보다 우세하지만 과제 간 편차가 큼. 표 제목 같은 메타데이터가 없으면 성능이 민감하게 떨어짐 |
| [EraRAG: Efficient and Incremental RAG for Growing Corpora](https://arxiv.org/abs/2506.20963) | 2025 arXiv | 초평면 LSH로 계층 그래프를 만들어 국소 삽입만으로 증분 갱신 | 5%씩 10라운드 삽입. 토큰 −57.6%, 구축 시간 −77.5% | 기존 방법들이 정적 코퍼스를 가정한다고 지적. 삭제와 수정은 다루지 않음(관찰) |
| [LinearRAG: Linear Graph RAG on Large-scale Corpora](https://arxiv.org/abs/2510.10114) | ICLR 2026 (검색 결과 기준) | 관계 추출 없이 엔티티와 의미 링크만으로 Tri-Graph를 구성 | multi-hop QA (미확인). 인덱싱 시간 77% 이상 절감 | 그래프 구축 오류가 검색 노이즈와 모호성을 유발한다는 문제의식 |
| [Temporal GraphRAG (TG-RAG)](https://arxiv.org/abs/2510.13590) | 2025 arXiv | 이중 수준 시간 그래프로 갱신 친화적 설계 | 미확인 | 기존 평가가 갱신 비용과 검색 안정성을 보지 않는다고 지적 |
| [WildGraphBench: Benchmarking GraphRAG with Wild-Source Corpora](https://arxiv.org/abs/2602.02053) | Findings of ACL 2026 | Wikipedia 인용 문장을 정답으로, 인용된 외부 원문(뉴스·블로그·PDF)을 코퍼스로 씀. 질문 1,100개 | 단일 사실, 다중 사실, 섹션 요약 | 단일 사실에서는 NaiveRAG가 대부분의 GraphRAG보다 우세. 그래프의 집계 패러다임은 세부 정보를 희생 |
| [BM25 Wins at Scale: A Scaling Study of RAG Paradigms (v1: Which RAG Paradigm Wins at Scale?)](https://arxiv.org/abs/2607.26497) | 2026 arXiv | 어휘·dense·그래프·에이전트 RAG를 코퍼스 규모별로 비교. 약 10M 토큰 이후 BM25가 정확도를 주도하고, 모든 규모에서 저비용 Pareto 끝점을 차지 | 다층 규모 코퍼스 | 무거운 그래프 빌더는 인덱싱 토큰당 최대 24.6 LLM 토큰을 쓰며, 전체 코퍼스의 처음 2% 안에서 구축이 멈춤(스케일 한계) |
| [LazyGraphRAG (Microsoft Research blog)](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/) | 2024 산업 블로그 | 인덱싱 시 LLM 요약을 생략하고 NLP로 처리한 뒤 질의 시점으로 지연. 인덱싱 비용은 vector RAG와 같고 full GraphRAG의 0.1% | 자체 평가 | 질의 시 비용이 발생함(700배 절감 등의 수치는 2차 출처) |

### B-3. 반복되는 미해결 문제, 암묵적 가정, 벤치마크 공백
1. **평가 프로토콜의 신뢰성.** GraphRAG 계열(GraphRAG, LightRAG, PathRAG)의 주장은 주로 LLM-judge 쌍대 승률에 기반합니다. 그런데 위치·길이 편향을 제거하면 이득이 축소되거나 사라집니다(Unbiased Eval 2506.06331, RAG vs GraphRAG의 judge 순서 효과). UltraDomain 같은 정답 없는 요약 평가에 지나치게 의존하는 것도 문제입니다.
2. **"그래프 = 이득"이라는 가정이 조건부로 깨짐.** 단일 사실에서는 RAG가 우세하고, multi-hop과 요약에서는 조건부로 그래프가 우세합니다(RAG vs GraphRAG, GraphRAG-Bench, WildGraphBench). 그러나 어떤 코퍼스·질의 특성이 그래프의 이득을 예측하는지 보여주는 정량 모델은 없습니다. GraphRAG-Bench가 "그래프 밀도-검색 품질 상관"을 보고했다는 2차 정보 정도가 있을 뿐입니다.
3. **인덱싱 비용과 스케일 한계.** LLM 기반 추출은 비용이 매우 큽니다(GraphRAG, KET-RAG, ArchRAG, LazyGraphRAG). 코퍼스가 커지면 BM25가 이깁니다(BM25 Wins at Scale). 대부분의 그래프 RAG 실험 코퍼스는 1M 토큰 안팎입니다.
4. **정적 코퍼스와 삽입 전용 갱신.** LightRAG와 EraRAG는 삽입만 지원합니다. 삭제·수정·시간적 무효화는 TG-RAG와 Preprints.org 2025 논문 정도가 다룹니다. 커뮤니티 요약이 낡는 문제(stale summary)를 측정하는 벤치마크도 없습니다.
5. **그래프 구축 오류의 전파.** OpenIE와 엔티티 추출의 노이즈가 검색 노이즈로 이어집니다(LinearRAG, PathRAG의 중복 진단, WildGraphBench의 노이즈 코퍼스). 구축 품질과 하류 성능의 관계를 분해해 측정한 연구가 부족합니다.
6. **도메인 사일로.** 텍스트 GraphRAG, 표 검색(TARGET), 코드 그래프(RepoGraph, CoIR, CodeRAG-Bench)가 서로 다른 벤치마크와 인덱스 개념을 씁니다. 표와 코드처럼 구조가 원래 존재하는 데이터에서 LLM이 만든 KG가 필요한지는 검증된 적이 없습니다.

### B-4. 덜 탐구된 연구 공백 (구체안)
1. **"그래프 이득 예측기": 인덱싱 전에 코퍼스·질의 특성으로 GraphRAG의 기대 이득을 추정하기.**
   - 내용: 엔티티 중복도, 문서 간 공유 엔티티 비율, 질의 hop 수 분포 같은 값싼 통계로 "그래프를 지을 가치"를 예측하고, 지을 경우 어떤 그래프 유형을 쓸지 정합니다.
   - 가장 가까운 연구: GraphRAG-Bench(사후 분석), RAG vs GraphRAG의 Selection 하이브리드(질의 시점), Unified Framework(VLDB).
   - 왜 미해결인가: 모두 그래프를 다 짓고 난 뒤 비교합니다. 인덱싱 전 의사결정이나 비용 예측은 다루지 않습니다.
2. **편향을 통제하고 정답이 있는 글로벌 sensemaking 벤치마크.**
   - 내용: 글로벌 질의에 검증 가능한 정답을 붙이는 방식입니다. 예를 들어 집계·계수형 질문이나, 코퍼스 생성 과정에서 심어 둔 주제 분포를 쓸 수 있습니다. LLM-judge 승률 대신 이런 정답으로 평가합니다.
   - 가장 가까운 연구: Unbiased Eval(편향 제거, 여전히 judge 사용), WildGraphBench(Wikipedia 문장을 정답으로 사용, 섹션 요약 포함), GraphRAG-Bench.
   - 왜 미해결인가: "포괄성·다양성"이라는 GraphRAG 본래의 주장 영역은 여전히 judge 의존적입니다.
3. **수정·삭제·시간 무효화를 포함한 동적 그래프 인덱스와 갱신 비용 벤치마크.**
   - 내용: 엣지와 요약에 출처 유효성(provenance)을 기록해, 커뮤니티 요약을 국소적으로 무효화하고 다시 생성합니다. "갱신 후 낡은 답변 비율"을 지표로 삼습니다.
   - 가장 가까운 연구: LightRAG(삽입), EraRAG(삽입, LSH), TG-RAG(시간 그래프), LazyGraphRAG(요약을 질의 시점으로 미뤄 문제를 우회).
   - 왜 미해결인가: 삭제와 수정 이후의 일관성, 그리고 갱신 토큰 비용 대비 품질 곡선을 함께 다룬 표준 평가가 없습니다.
4. **스케일 인지 하이브리드: BM25/dense를 기본으로 하고 그래프를 선택적으로 국소 구축하기.**
   - 내용: 10M 토큰 이상 코퍼스에서는 전체 그래프를 짓지 않습니다. 대신 질의 로그상 hot 영역이나 multi-hop 질의가 자주 닿는 문서 군집에만 그래프를 지연·국소 구축합니다.
   - 가장 가까운 연구: KET-RAG(PageRank로 핵심 청크만 KG화, 정적), LazyGraphRAG(질의 시점 처리), BM25 Wins at Scale(문제 제기).
   - 왜 미해결인가: 질의 분포에 적응하는 부분 그래프 구축과 그 비용-정확도 Pareto를 대규모 코퍼스에서 실험한 연구는 확인되지 않았습니다.
5. **원래 구조가 있는 데이터(표·코드)에서 LLM 추출 KG vs 원래 구조 그래프 비교.**
   - 내용: 표 스키마나 외래키, 코드의 호출·정의-참조 그래프를 그대로 인덱스로 쓰는 경우와 LLM으로 엔티티 KG를 추출하는 경우를 같은 과제에서 비교합니다. 혼합 문서(표가 포함된 보고서, 코드와 문서)에서의 통합 인덱스도 포함합니다.
   - 가장 가까운 연구: RepoGraph(원래 구조), TARGET(표 검색 평가만), StructRAG(추론 시점 구조화), Financial Report Chunking(요소 단위).
   - 왜 미해결인가: 텍스트 GraphRAG와 표·코드 검색 커뮤니티가 분리되어 있어 공통 벤치마크와 비교가 없습니다.

---

### 부록: 검증 상태 요약
- 모든 항목의 제목, arXiv ID 또는 학회 페이지 URL은 WebSearch 결과로 존재를 확인했습니다.
- 다음 venue는 검색 결과에서 직접 확인했습니다: Dense X(EMNLP'24), LumberChunker(Findings EMNLP'24), RAPTOR(ICLR'24), Self-Route(EMNLP'24 Industry), Long-Context LLMs Meet RAG(ICLR'25), SiReRAG(ICLR'25), LaRA(ICML'25), MoC(ACL'25), MemoRAG(WWW'25), HippoRAG(NeurIPS'24), HippoRAG 2(ICML'25), G-Retriever(NeurIPS'24), GRAG(Findings NAACL'25), RepoGraph(ICLR'25), StructRAG(ICLR'25), CoIR(ACL'25), CodeRAG-Bench(Findings NAACL'25), Unified Framework(PVLDB'25), PathRAG(AAAI'26), WildGraphBench(Findings ACL'26), A-Mem(NeurIPS'25).
- 다음은 "미확인"입니다: LightRAG, KET-RAG, Late Chunking, LongRAG, HiChunk의 정식 venue. 본문 표 수치의 다수.
- 메모리형 인덱스 관련: [A-Mem: Agentic Memory for LLM Agents](https://arxiv.org/abs/2502.12110) (NeurIPS 2025)는 Zettelkasten식 노트 연결과 memory evolution을 제안합니다. 축 A의 메모리형 인덱스 범주에 해당합니다.
