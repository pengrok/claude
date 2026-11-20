# 문헌 조사 B: RAG 검색·인덱싱 측면 (A. 청킹/인덱스 구조, B. 그래프/구조화 인덱스)

작성일: 2026-11-19 (축 A 본문 전수 확인: 2026-11-20)

> **검증 방법**: 처음 작성 때는 네트워크 정책으로 원문을 열지 못해 WebSearch 결과로만 확인했습니다. 이후 축 A의 논문은 모두 **본문 전문 텍스트(arXiv/ACL Anthology PDF, Anthropic 블로그 원문)** 로 다시 확인했습니다.
> - 표의 수치와 벤치마크는 본문 초록·표·실험 절에서 찾은 값입니다. 버전이 여러 개인 arXiv 논문은 확인한 버전(예: LongRAG v3)의 값을 적었습니다.
> - venue는 본문 첫 쪽 헤더·각주나 arXiv comment에 저자가 적은 경우에만 적었습니다. 그런 근거가 없으면 "venue 근거 없음"으로 표시했습니다.
> - "Stated limitations" 열에는 저자가 Limitations, Conclusion, Discussion 절에 직접 적은 한계를 근거 절과 함께 요약했습니다. 그런 절이 없으면 "명시된 한계 없음"으로 적었습니다. 조사자의 해석은 **[관찰]** 로 구분했습니다.

---

## 축 A. 인덱싱 & 청킹 구조

### A-1. 현황 요약
청킹 연구는 "고정 길이냐, 의미 단위냐"라는 질문에서 시작했습니다. 지금은 두 방향이 경쟁합니다.
- **청크에 문맥을 되돌려 주는 방식**: Late Chunking, Anthropic Contextual Retrieval.
- **검색 단위 자체를 바꾸는 방식**: 명제 단위(Dense X), 약 4K 토큰 단위(LongRAG), 요약 트리(RAPTOR, SiReRAG).

대규모 통제 비교 연구 세 편이 있습니다(Vectara 2024, Zhou et al. 2026, Shaukat et al. 2026). 이 연구들은 "최적 청킹은 과제와 도메인에 따라 다르다"는 점에서 결론이 같습니다.
- Zhou et al.: in-corpus 검색에서는 구조 기반 청킹이, in-document 검색에서는 LumberChunker가 가장 좋았습니다.
- Shaukat et al.: 도메인마다 최적 전략이 달랐습니다.

비싼 청킹이 이득이 되는지에 대해서는 결과가 엇갈립니다.
- Vectara와 Zhou et al.: semantic/LLM 청킹의 이득은 일관되지 않았고, 실제 데이터나 in-corpus 검색에서는 고정 크기·구조 기반 청킹이 더 낫거나 비슷했습니다.
- Shaukat et al.: content-aware 청킹(문단 그룹, LLM 기반, 동적 크기)이 단순 고정 길이 문자 청킹보다 유의하게 좋았습니다. 1위는 LLM이 아닌 Paragraph Group Chunking이었습니다. LLM 기반 분할은 상위권이었지만 전처리 메모리 부담이 매우 컸습니다.

장문맥 LLM과 RAG의 관계는 "어느 쪽이 대체하느냐"는 논쟁에서 "어떻게 라우팅하거나 섞느냐"는 논쟁으로 넘어갔습니다(Self-Route, LaRA). 2025~2026년에 보이는 새 흐름은 세 가지입니다.
- **구조 인식 청킹/라우팅**: 문서 헤딩 계층을 그대로 인덱스로 씁니다(HiChunk, RDR2, MultiDocFusion).
- **청킹 품질의 내재적 지표**: MoC의 Boundary Clarity(BC)와 Chunk Stickiness(CS), HiCBench의 계층 청킹 경계 주석.
- **메모리형 인덱스**: MemoRAG, A-Mem, HippoRAG 2.

### A-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Dense X Retrieval: What Retrieval Granularity Should We Use?](https://arxiv.org/abs/2312.06648) | 2023 arXiv → EMNLP 2024 Main(본문 헤더) | 원자적이고 자기완결적인 사실 단위인 "명제(proposition)"를 검색 단위로 제안. FactoidWiki를 구축하고 passage/sentence/proposition 단위를 비교 | open-domain QA 5종(NQ, TriviaQA, WebQ, SQuAD, EntityQuestions). dense retriever 4종(§4.2. Limitations 절에는 "6종"으로 적혀 있어 본문 안에서 서로 다름) | Limitations 절: Wikipedia 코퍼스만 사용, bi/dual-encoder 위주로 평가, 영어만 다룸. [관찰] 명제 추출에 LLM 비용이 들고, 명제 사이의 담화 문맥이 사라짐 |
| [RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval](https://arxiv.org/abs/2401.18059) | ICLR 2024(본문 헤더) | 청크를 임베딩→클러스터링→요약하는 과정을 재귀적으로 반복해 트리를 만들고, 여러 추상 수준에서 검색 | QuALITY(GPT-4와 결합 시 정확도 82.6 / hard 76.2. 기존 최고 CoLISA 62.3 / 54.7 대비 약 +20%p, 표 7), NarrativeQA, QASPER(GPT-4 F1 55.7) | 명시된 한계 없음(Limitations 절 없음). §3에서 요약의 약 4%에 경미한 환각이 있었다고 보고(부록 E, 150개 노드 중 6개). QA에는 영향이 미미했다고 함. [관찰] 정적 코퍼스를 가정하며, 문서가 추가될 때의 갱신은 다루지 않음 |
| [Financial Report Chunking for Effective Retrieval Augmented Generation](https://arxiv.org/abs/2402.05131) | 2024 arXiv(Unstructured. venue 근거 없음) | 제목·표 같은 문서 요소 경계에 맞춰 청킹하는 element 기반 방식. chunk size를 튜닝할 필요가 없다고 주장 | FinanceBench. 페이지 검색 정확도: 여러 청킹 결과를 합친 "Chipper Aggregation"이 84.4%, 단일 element 청킹 최고는 67.4%(표 4). Q&A 수동 정확도 53.19%, 기존 50%(표 5, §5) | §5: 결합 전략은 GPT-4 토큰 한도를 넘어 Q&A에 쓸 수 없었음. §6: 다른 도메인(바이오 등)에서의 검증과 요소 간 관계 활용은 future work |
| [LumberChunker: Long-Form Narrative Document Segmentation](https://arxiv.org/abs/2406.17526) | 2024 arXiv(venue 근거 없음. 본문과 arXiv comment에 표기 없음. Zhou et al. 2026은 Findings of EMNLP 2024로 인용) | LLM이 연속 문단을 읽고 내용이 바뀌는 지점을 골라 청크 경계를 정함. GutenQA 벤치마크 공개 | GutenQA(Gutenberg 도서 100권, QA 3,000쌍). 가장 강한 baseline 대비 DCG@20 +7.37% | §7 Limitations: LLM이 필요해 비싸고 느림. 법률 문서처럼 구조화된 텍스트에서는 Markdown 구조 파싱과 결과가 비슷한데 비용만 커서 불필요하게 복잡함. 문서 길이·문서 수가 늘면 확장성 문제 |
| [LongRAG: Enhancing Retrieval-Augmented Generation with Long-context LLMs](https://arxiv.org/abs/2406.15319) | 2024 arXiv(comment "Technical Report") | 관련 문서를 묶어 약 4K 토큰 단위로 재구성하고, Wikipedia 검색 단위를 22M개에서 600K개로 줄임(v3 기준). "long retriever + long reader" 구조 | NQ EM 62.7, HotpotQA(full-wiki) EM 64.3(GPT-4o), Qasper F1 25.9, MultiFieldQA-en F1 57.5 | 명시된 한계 없음(Limitations 절 없음). §3과 §4에서 장문 임베딩 모델이 아직 약해 청크별 최대 점수로 근사했다고 밝히고, 장문 임베딩이 발전하면 더 나아질 것이라고 언급 |
| [Retrieval Augmented Generation or Long-Context LLMs? A Comprehensive Study and Hybrid Approach (Self-Route)](https://arxiv.org/abs/2407.16833) | EMNLP 2024 Industry(arXiv comment) | 자원이 충분하면 LC가 평균적으로 RAG보다 우세함을 보임. LLM이 스스로 "답할 수 있는지"를 판단해 질의를 RAG 또는 LC로 라우팅 | LongBench 7종(NarrativeQA, Qasper, MultiFieldQA, HotpotQA, 2WikiMultihopQA, MuSiQue, QMSum), ∞Bench 2종(En.QA, En.MC), 합성 PassKey. 모델: Gemini-1.5-Pro, GPT-4o, GPT-3.5-Turbo. 비용 절감: Gemini-1.5-Pro −65%, GPT-4o −39% | 명시된 한계 없음(Limitations 절 없음). §5.4: 합성 데이터셋은 구성 과정의 artifact에 결과가 크게 좌우됨. §5.5: LLM 내부 지식(데이터 누출)을 배제한 비교는 future work |
| [Late Chunking: Contextual Chunk Embeddings Using Long-Context Embedding Models](https://arxiv.org/abs/2409.04701) | 2024 arXiv(Jina AI·Weaviate. comment "3rd draft". venue 근거 없음) | 문서 전체를 장문맥 임베딩 모델에 통과시킨 뒤, 토큰 임베딩을 청크별로 mean-pool. 추가 학습이 필요 없음. long late chunking과 span pooling 학습 방법을 확장으로 제안 | BeIR 소형 과제(SciFact, NFCorpus, FiQA, TREC-COVID), LongEmbed(NarrativeQA 등 독해 과제와 Needle/Passkey 합성 과제) | 명시된 한계 없음(Limitations 절 없음). §4.2: 일부 독해 과제, 특히 합성 Needle/Passkey에서는 큰 청크를 쓸 때 naive 청킹이 더 나음. 모델 입력 길이를 넘는 문서에는 long late chunking이 필요(§3, 부록 A.1) |
| [MemoRAG: Boosting Long Context Processing with Global Memory-Enhanced Retrieval Augmentation](https://arxiv.org/abs/2409.05591) | WWW 2025(본문 ACM 각주, arXiv comment) | 경량 장문맥 모델이 코퍼스 전체의 "global memory"를 만들고, 여기서 생성한 draft clue를 검색 질의로 쓰는 이중 시스템 | LongBench, InfiniteBench, 저자들이 새로 만든 UltraDomain(20개 데이터셋, 128K 토큰 초과 문맥 다수) | 명시된 한계 없음(Limitations 절 없음. Conclusion에도 한계 언급 없음) |
| [Anthropic — Introducing Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) | 2024-09-19 산업 블로그 | 각 청크 앞에 LLM이 만든 50–100토큰 문맥 설명을 붙인 뒤 임베딩과 BM25 인덱싱. 프롬프트 캐싱으로 비용 절감(1회성 비용 $1.02/백만 문서 토큰) | 자체 데이터(코드베이스, 소설, arXiv 논문, 과학 논문 등). top-20 검색 실패율: Contextual Embeddings 단독 5.7%→3.7%(−35%), BM25 결합 시 2.9%(−49%), 리랭킹 추가 시 1.9%(−67%) | 일반 문서 요약을 청크에 붙이는 방식은 이득이 매우 제한적이었다고 언급. 리랭킹은 지연과 비용을 늘림. [관찰] 자체 평가 결과이며 독립 재현은 제한적 |
| [Is Semantic Chunking Worth the Computational Cost?](https://arxiv.org/abs/2410.13070) | 2024 arXiv(Vectara. venue 근거 없음) | 고정 크기, breakpoint 기반, 클러스터 기반 청커를 문서 검색·증거 검색·답 생성 세 과제에서 비교. 결론: semantic 청킹의 이득은 일관되지 않고 stitched 데이터에서만 유리하며, 임베딩 품질 같은 다른 요인의 영향이 더 큼(§5) | 문서 검색 10개 데이터셋(짧은 문서 6종은 이어 붙인 stitched 버전), 증거 검색 RAGBench 5종 | Limitations 절: 문장 단위 분할이라 문장 임베딩에 문맥이 없음. 청크 품질을 직접 재는 지표가 없음. 적합한 데이터셋이 없음(stitch로 주제 다양성이 인공적으로 높아지고, RAGBench 답은 LLM이 생성함) |
| [Long-Context LLMs Meet RAG: Overcoming Challenges for Long Inputs in RAG](https://arxiv.org/abs/2410.05983) | 2024 arXiv(Google Cloud AI Research·UIUC. venue 근거 없음) | 검색 문서 수를 늘리면 성능이 올랐다가 떨어지는 역U자 현상을 보이고, 원인을 hard negative로 규명. precision만으로는 그 영향을 판단할 수 없다는 점도 보임. 해법: retrieval reordering, RAG 특화 implicit fine-tuning, 중간 추론을 넣은 fine-tuning | 분석: NQ, PopQA. FT 학습: NQ, WoW, FEVER, MMLU. FT 평가: TriviaQA, PopQA, HotpotQA, 2WikiMultihopQA, WebQuestions, Bamboogle, ASQA, T-REx, zsRE | 명시된 한계 없음(Limitations 절 없음). §7: 자동 위치 최적화와 다단계 추론 fine-tuning은 future work |
| [SiReRAG: Indexing Similar and Related Information for Multihop Reasoning](https://arxiv.org/abs/2412.06206) | ICLR 2025(본문 헤더) | 유사성 트리(RAPTOR식)와 관련성 트리(명제·엔티티 기반)를 함께 만들고 평탄화해 하나의 검색 풀로 사용 | MuSiQue, 2WikiMultiHopQA, HotpotQA. 평균 F1 +1.9%, 리랭킹과 결합 시 최대 +7.8% | 명시된 한계 없음(Limitations 절 없음). §6.4: 검색 풀이 커서 RAPTOR보다 질의 시간이 김(저자는 TPER<1이라 효율이 합리적이라고 주장). [관찰] 두 트리 모두 LLM 요약이 필요해 인덱싱 비용이 큼. 다만 요약 LLM을 GPT-3.5-Turbo로 바꿔도 성능 차이가 작다고 보고함 |
| [LaRA: Benchmarking Retrieval-Augmented Generation and Long-Context LLMs – No Silver Bullet for LC or RAG Routing](https://arxiv.org/abs/2502.09977) | 2025 arXiv(venue 근거 없음. ICML 양식의 Impact Statement만 있음) | RAG와 LC를 비교하는 벤치마크. 기존 비교의 설계 결함을 지적. 최적 선택은 모델 크기, 장문 처리 능력, 문맥 길이, 과제 유형, 검색 청크 특성에 따라 달라짐 | 2,326개 케이스, QA 과제 4종(위치 찾기, 비교, 추론, 환각 탐지), 문서 3종(소설, 논문, 재무제표), LLM 11종(오픈 7 + 상용 4) | 명시된 한계 없음(Limitations 절 없음). "단일 라우팅 규칙은 없다"는 것은 결론(§5)임 |
| [MoC: Mixtures of Text Chunking Learners for Retrieval-Augmented Generation System](https://arxiv.org/abs/2503.09600) | 2025 arXiv(venue 근거 없음) | 청킹 품질을 직접 재는 지표 Boundary Clarity와 Chunk Stickiness 제안. granularity router와 경량 meta-chunker를 조합한 MoC | CRUD(single-hop, two-hop), DuReader(LongBench), WebCPM. 모두 중국어 | §6 Limitations: 학습 데이터가 약 2만 건으로 실제 텍스트의 규모와 다양성에 비해 작음. 다국어 적응과 검증이 부족함 |
| [Reconstructing Context: Evaluating Advanced Chunking Strategies for RAG](https://arxiv.org/abs/2504.19754) | ECIR 2025 Workshop(KEIR. arXiv comment) | Late Chunking과 Contextual Retrieval의 비교 평가 | 검색 NFCorpus, 생성 MS MARCO. Contextual Retrieval 실험(RQ2)은 NFCorpus의 20%, 50개 질의, 약 300개 문서 규모 | 초록: Contextual Retrieval은 의미 응집성이 높지만 계산 비용이 큼. Late Chunking은 효율적이지만 관련성과 완결성을 일부 희생. §4: GPU 메모리(24GB) 제약으로 소규모 부분집합만 사용 |
| [HiChunk: Evaluating and Enhancing Retrieval-Augmented Generation with Hierarchical Chunking](https://arxiv.org/abs/2509.11552) | 2025 arXiv(Tencent Youtu. venue 근거 없음) | fine-tune한 Qwen3-4B로 다단계 문서 구조(청킹 경계와 계층 레벨)를 예측하고, Auto-Merge 검색으로 청크 granularity를 조절. 기존 벤치마크의 evidence sparsity를 지적하고 계층 청킹 평가용 HiCBench 공개 | 청킹 정확도: Gov-report, Qasper. RAG 전체: HiCBench, LongBench, Qasper, GutenQA, OHRBench | 명시된 한계 없음(Limitations 절 없음). "기존 청커는 선형 구조만 다룬다"는 것은 선행 연구에 대한 비판(§1)임 |
| [RDR2: Equipping Retrieval-Augmented Large Language Models with Document Structure Awareness](https://arxiv.org/abs/2510.04293) | Findings of EMNLP 2025(arXiv comment) | 문서 구조 트리 위에서 LLM 라우터가 [ANS]/[EXP]/[REF](Answer/Expand/Refuse)를 고르는 학습 가능한 구조 탐색(Retrieve-DocumentRoute-Read) | TriviaQA, HotpotQA, ASQA, QAMPARI, ELI5(각 최대 1,000개 샘플). 데이터스토어는 Wikipedia | Limitations 절: 문서를 하나씩 독립적으로 처리해 문서 간 지식 통합이 제한됨. 데이터스토어 전체의 문서 구조 트리를 오프라인으로 구축해야 함(Wikipedia 약 20분). 반복 라우팅에 계산 오버헤드가 있음 |
| [MultiDocFusion: Hierarchical and Multimodal Chunking Pipeline for Enhanced RAG on Long Industrial Documents](https://preview.aclanthology.org/credits/2025.emnlp-main.1062) | EMNLP 2025 Main(본문 각주, pp. 20985–21004) | 레이아웃 검출, OCR, LLM 기반 섹션 계층 파싱(DSHP-LLM), DFS 기반 계층 청크 그룹핑을 잇는 파이프라인 | 계층 파싱: DocHieNet, HRDH. RAG-VQA: DUDE, MPVQA, CUAD, MOAMOB(재무 보고서, 계약서, 스캔 문서). 검색 precision +8–15%, ANLS +2–3%. DP 모델과 OCR 모델을 바꿔 가며 강건성 실험(§5.4) | Limitations 절: DSHP-LLM이 폰트, 여백, 열 구조 같은 세밀한 레이아웃 신호를 쓰지 못함. 그래프 기반 검색은 검증하지 않음. 부모 문맥을 중복 저장해 인덱스 크기, 지연, 저장 비용이 늘어남 |
| [Beyond Chunk-Then-Embed: A Comprehensive Taxonomy and Evaluation of Document Chunking Strategies for IR](https://arxiv.org/abs/2602.16974) | 2026 arXiv(Zuccon 그룹, UQ·CSIRO. venue 근거 없음) | 청킹 분류 체계를 만들고 재현 평가 수행. in-corpus에서는 구조 기반 청킹, in-document에서는 LumberChunker가 우세. 여기서 "문맥화 청킹(contextualized chunking)"은 late chunking 방식을 가리킴 | BEIR 6종(FiQA, ArguAna, SciDocs, TREC-COVID, SciFact, NFCorpus), GutenQA. 임베딩 4종(Jina-v2, Jina-v3, Nomic, E5-large) | 명시된 한계 없음(Limitations 절 없음). 주요 발견: 문맥화 청킹은 in-corpus 성능을 올리지만 in-document 성능은 모든 설정에서 떨어뜨림(Nomic −53.17%, Jina-v3 −29.82%, Jina-v2 −5.16%, E5-large −3.83%) |
| [A Systematic Investigation of Document Chunking Strategies and Embedding Sensitivity](https://arxiv.org/abs/2603.06976) | 2026 arXiv(venue 근거 없음) | 청킹 전략 36종 × 도메인 6개 × 임베딩 5종, 총 1,080개 구성의 통제 비교. content-aware 청킹이 고정 길이보다 유의하게 우수. 1위는 Paragraph Group Chunking(nDCG@5 ≈ 0.459) | UltraDomain(Biology, Physics, Health, Legal, Maths, Agriculture). LLM 판정자로 graded relevance 부여 | Limitations and Future Work 절: 인덱스 크기와 청크 수가 의미 품질 외의 교란 요인이 됨. 관련성 판정을 사람이 아닌 LLM이 함. UltraDomain의 일부 도메인만 사용 |

### A-3. 반복되는 미해결 문제, 암묵적 가정, 벤치마크 공백
1. **"최적 청킹은 과제에 따라 다르다"는 결론은 반복되지만, 과제에 맞춰 청킹을 고르는 원리는 없습니다.**
   - 근거: Vectara 2024, Zhou et al. 2026, Shaukat et al. 2026, LaRA.
   - MoC의 router는 텍스트를 보고 granularity만 고릅니다. 질의 분포나 코퍼스 특성에는 조건화되지 않습니다.
2. **평가 데이터가 인공적이거나 부족합니다.**
   - semantic 청킹은 인공으로 이어 붙인(stitched) 데이터에서만 이깁니다(Vectara). 그래서 경계 검출 능력과 실제 검색 효용이 혼동됩니다. Vectara 저자들도 Limitations 절에서 적합한 데이터셋이 없다고 직접 밝혔습니다.
   - HiChunk는 기존 RAG 벤치마크가 evidence sparsity 때문에 청킹 품질을 판별하지 못한다고 지적합니다.
   - GutenQA처럼 서사 텍스트에 치우친 벤치마크도 있습니다.
3. **정적 코퍼스를 가정합니다.**
   - RAPTOR, SiReRAG, MemoRAG, Contextual Retrieval은 모두 문서 단위나 코퍼스 단위로 한 번 전처리합니다.
   - 문서가 수정되면 요약, 문맥 접두사, 트리의 어느 부분이 무효가 되는지는 어느 본문도 다루지 않습니다.
   - 반면 에이전트 메모리 쪽의 A-Mem은 새 기억이 들어올 때 기존 노트를 갱신하는 memory evolution을 제안합니다. 다만 대상이 대화 기억이며, 문서 코퍼스나 계층 요약 인덱스는 아닙니다.
4. **문맥 주입에는 부작용이 있습니다.**
   - Zhou et al. 2026은 late chunking 방식의 문맥화 청킹이 in-document 판별력을 모든 설정에서 떨어뜨린다는 것을 정량화했습니다(모델별 −3.8%~−53.2%).
   - 저자들은 원인으로 "청크 임베딩이 청크 고유 내용보다 문서 전체 주제를 담게 되어, 같은 문서의 청크들이 서로 비슷해진다"는 가설을 제시했습니다. 이 가설은 검증하지 않았습니다.
   - Anthropic식으로 청크 앞에 문맥 접두사를 붙이는 방식에서도 같은 현상이 생기는지는 이 연구에서 실험하지 않았습니다.
5. **LC vs RAG 비교 결과가 특정 시점의 모델에 묶여 있습니다.**
   - Self-Route(2024)는 "LC 우세"를, LaRA(2025)는 "no silver bullet"을 결론으로 냅니다. 모델 세대에 따라 결론이 바뀝니다.
   - Self-Route는 비용을 보고하지만, 비용(토큰·지연)과 정확도를 함께 놓고 보는 Pareto 비교는 드뭅니다.
6. **청킹 품질 지표가 다운스트림 성능을 얼마나 잘 예측하는지 검증되지 않았습니다.**
   - Vectara 저자들은 청크 품질을 직접 재는 지표가 없다는 점을 한계로 밝혔습니다.
   - 그 뒤 MoC의 BC/CS와 HiCBench의 경계 주석이 나왔지만, 이 지표들이 다른 코퍼스·언어·검색기에서도 다운스트림 성능을 예측하는지는 확인되지 않았습니다. MoC는 중국어 데이터셋 3종으로만 평가했습니다.
   - Zhou et al.은 청크 크기와 성능의 상관이 in-document에서는 중간, in-corpus에서는 약하다고 보고했습니다.

### A-4. 덜 탐구된 연구 공백 (구체안)
1. **질의 분포에 조건화한 청킹/인덱스 선택(chunking-as-a-policy).**
   - 내용: 운영 로그의 질의 분포(사실형, 집계형, 문서 내 탐색형)를 보고 문서별 granularity나 인덱스 유형(명제, 문단, 4K 단위, 요약 트리)을 고르거나 섞는 학습 정책입니다.
   - 가장 가까운 연구: MoC(granularity router), StructRAG(추론 시점에 구조 유형을 고르는 router), Self-Route(LC/RAG 라우팅).
   - 왜 미해결인가: 모두 질의 시점에 라우팅하거나 텍스트 자체의 특성만 봅니다. 인덱싱 시점에 예상 질의 분포에 맞춰 최적화하지 않으며, 비용 제약도 넣지 않습니다.
2. **문맥화 청킹의 "in-document 판별력 붕괴" 메커니즘 규명과 완화.**
   - 내용: 문서 수준 문맥 때문에 같은 문서의 청크 임베딩들이 서로 비슷해진다는 가설을 직접 측정합니다. 측정 지표 예: 문서 내 청크 간 코사인 유사도 분포, 이방성. late chunking과 접두사형 Contextual Retrieval을 함께 비교하고, 문서 공통 성분을 빼는 debiasing이나 청크 고유 성분 강조 같은 완화 기법을 만듭니다.
   - 가장 가까운 연구: Zhou et al. 2026(late chunking에서 현상을 정량화하고 가설 제시), Late Chunking, Anthropic Contextual Retrieval, Reconstructing Context(두 방식 비교. 단 in-document 과제는 다루지 않음).
   - 왜 미해결인가: 현상과 가설은 보고되었지만 메커니즘 검증과 해결책이 없습니다. 접두사형 방식에서도 같은 현상이 생기는지도 확인되지 않았습니다.
3. **증분 갱신과 수정에 강한 계층 요약 인덱스.**
   - 내용: RAPTOR/SiReRAG식 트리에서 문서 추가·삭제·수정이 일어날 때 무효가 되는 요약 노드만 국소적으로 다시 만드는 방법입니다. 품질 drift를 평가하는 프로토콜도 함께 필요합니다.
   - 가장 가까운 연구: EraRAG(LSH 기반, 그래프 삽입만 지원), LightRAG(삽입만 지원), A-Mem(새 기억이 들어오면 기존 노트를 갱신. 단 대화 기억이 대상이고 요약 트리는 아님).
   - 왜 미해결인가: 수정과 삭제, 그리고 요약의 일관성(오래된 사실이 요약에 남는 문제)을 다루는 평가가 없습니다. RAPTOR, SiReRAG, MemoRAG 본문은 갱신을 다루지 않습니다.
4. **비용을 반영한 LC/RAG/구조 인덱스 3자 Pareto 벤치마크.**
   - 내용: 같은 코퍼스에서 인덱싱 비용(LLM 토큰), 질의 비용, 지연, 정확도를 함께 보고하는 표준 프로토콜입니다. 모델 세대가 바뀔 때마다 다시 측정할 수 있어야 합니다.
   - 가장 가까운 연구: LaRA(비용 축이 약함), Self-Route(비용은 보고하지만 구조 인덱스가 없음), Shaukat et al. 2026(청킹 전략의 인덱스 크기와 지연을 보고하지만 LC는 비교하지 않음), "BM25 Wins at Scale"(2026, 축 B 참고).
5. **레이아웃·구조 기반 인덱스의 실패 모드 연구.**
   - 내용: 헤딩 계층이 깨지거나 잘못된 문서(스캔 PDF, 슬라이드, 웹 보일러플레이트)에서 구조 인식 청킹이 얼마나 견고한지 보는 연구입니다.
   - 가장 가까운 연구: HiChunk/HiCBench, RDR2, MultiDocFusion, Financial Report Chunking.
   - 왜 미해결인가: RDR2는 헤딩이 잘 정리된 Wikipedia만으로 평가했습니다. HiChunk와 Financial Report Chunking도 구조가 제대로 파싱된다고 가정합니다. 부분적인 분석은 두 편 있습니다.
     - MultiDocFusion(§5.4): 레이아웃 검출(DP) 모델과 OCR 모델을 바꿔 가며 강건성을 실험했습니다. 다만 저자들은 세밀한 레이아웃 신호를 쓰지 못한다는 점을 한계로 밝혔습니다.
     - 2026년 재무 PDF 파싱 연구([2604.12047](https://arxiv.org/abs/2604.12047), ICSE-SEIP 2026): PyMuPDF, Unstructured 등 PDF 파서와 청킹 방식의 조합을 FinanceBench와 새로 만든 TableQuest에서 비교했습니다.
   - 남은 공백: 헤딩 계층 자체의 오류(누락되거나 잘못된 레벨)를 통제해 주입하고, 그 오류가 검색 오류로 이어지는 과정을 분석한 연구는 없습니다.


---

## 축 B. 그래프 / 구조화 인덱스

### B-1. 현황 요약
2024년에 Microsoft GraphRAG(엔티티 KG와 커뮤니티 요약)가 "글로벌 sensemaking 질의"라는 새 과제 범주를 열었습니다. 이어 LightRAG(이중 수준 검색, 증분 삽입), HippoRAG(PPR 기반 단일 단계 multi-hop), G-Retriever/GRAG(텍스트 그래프와 GNN soft prompt)가 방법론의 큰 갈래를 만들었습니다.

2025년의 주제는 **비용 절감과 노이즈 제거**였습니다. KET-RAG, LazyGraphRAG, ArchRAG, PathRAG, LinearRAG가 여기에 속합니다. 동시에 **비판적 재평가**가 대거 나왔습니다. RAG vs GraphRAG(KDD'26), GraphRAG-Bench(ICLR'26), Unbiased Eval, Unified Framework(PVLDB) 같은 연구가 그 예입니다. 공통 결론은 다음과 같습니다.
- 단일 사실 질의에서는 vanilla RAG가 동등하거나 우세합니다.
- 그래프는 multi-hop과 요약에서 조건부로 이득을 줍니다.
- 기존에 보고된 이득의 상당 부분은 LLM-judge 편향(위치, 길이, 시행 편향)과 코퍼스와 무관한 질문 생성 때문입니다. 편향을 제거하면 이득이 "훨씬 완만"해지고, 일부 비교(LightRAG vs NaiveRAG)에서는 역전됩니다.

2026년에는 실제 웹 소스를 쓰는 노이즈 벤치마크(WildGraphBench)와 스케일 연구가 나왔습니다. 스케일 연구(BM25 Wins at Scale)는 28단계, 최대 600.8M 토큰의 기업 코퍼스에서 약 10M 토큰부터 BM25가 모든 공유 규모에서 정확도 선두를 차지한다고 보고했습니다. LLM 추출형 그래프 빌더는 그보다 훨씬 작은 규모에서 구축이 멈췄습니다("construction wall"). 이로써 "인덱싱 비용 대비 이득"이 핵심 쟁점이 되었습니다. 표와 코드 영역은 각각 독립된 벤치마크 생태계(TARGET, CoIR, CodeRAG-Bench)가 있습니다. 반면 저장소 그래프(RepoGraph) 같은 구조 인덱스와 텍스트 GraphRAG 연구는 거의 교류하지 않습니다.

### B-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [From Local to Global: A GraphRAG Approach to Query-Focused Summarization](https://arxiv.org/abs/2404.16130) | 2024 arXiv (v2 2025-02, 본문 표기 "Preprint. Under review.") | LLM으로 엔티티 KG를 만들고 커뮤니티별 요약을 미리 생성한 뒤, map-reduce 방식으로 글로벌 질의에 답함. vector RAG 대비 포괄성 승률 72–83%, 다양성 승률 62–82% | 팟캐스트(약 1M 토큰)와 뉴스(약 1.7M 토큰) 코퍼스 2종. LLM 쌍대 비교로 포괄성·다양성·empowerment·directness 평가, claim 기반 지표 병행 | 평가가 약 1M 토큰 코퍼스 2개의 sensemaking 질문에 한정되어 일반화 검증이 더 필요하고, fabrication rate 비교가 없음(§6.1). directness(통제 기준)에서는 vector RAG가 우세. [관찰] 인덱싱 비용이 매우 큼 |
| [G-Retriever: Retrieval-Augmented Generation for Textual Graph Understanding and Question Answering](https://arxiv.org/abs/2402.07630) | NeurIPS 2024 (본문·arXiv comment에 venue 근거 없음) | 텍스트 그래프에서 PCST(Prize-Collecting Steiner Tree)로 부분그래프를 검색하고 GNN soft prompt로 LLM에 전달. GraphQA 벤치마크 제안. 수작업 검토 100건에서 기준선 대비 환각 54% 감소(§6.4) | GraphQA(ExplaGraphs, SceneGraphs, WebQSP) | 검색 구성요소가 정적이어서 학습되지 않음. 학습 가능한 검색은 향후 과제 |
| [HippoRAG: Neurobiologically Inspired Long-Term Memory for LLMs](https://arxiv.org/abs/2405.14831) | NeurIPS 2024 | OpenIE로 KG를 만들고 질의 개념에서 Personalized PageRank를 돌려 단일 단계로 multi-hop 검색 | MuSiQue, 2Wiki, HotpotQA. 최대 +20%. IRCoT 대비 10–20배 저렴(본문 일부는 10–30배), 6–13배 빠름 | 구성요소를 학습 없이 사용하며 오류 대부분이 NER·OpenIE에서 발생. PPR 이상의 그래프 탐색 개선 여지, 긴 문서의 OpenIE 일관성 문제, 벤치마크를 넘는 규모로의 확장성 미검증(§7) |
| [GRAG: Graph Retrieval-Augmented Generation](https://arxiv.org/abs/2405.16506) | Findings of NAACL 2025 (본문·arXiv comment에 venue 근거 없음) | k-hop ego-graph를 검색해 병합하고 soft pruning으로 불필요한 노드를 약화. 텍스트 view와 그래프 view(soft prompt)를 함께 사용 | WebQSP, ExplaGraphs | 부분그래프 검색 효율이 초기 노드 랭킹과 pruning 품질에 의존하므로, 그래프 구조나 노드 중요도를 추정하기 어려우면 검색 성능이 떨어질 수 있음(§7) |
| [Graph Retrieval-Augmented Generation: A Survey](https://arxiv.org/abs/2408.08921) | 2024 arXiv (survey, "Ongoing work") | GraphRAG를 G-Indexing, G-Retrieval, G-Generation의 3단계로 정리 | 해당 없음 | 해당 없음(서베이). 향후 방향으로 동적 그래프, 멀티모달, 확장성, 표준 벤치마크를 제시(§10) |
| [LightRAG: Simple and Fast Retrieval-Augmented Generation](https://arxiv.org/abs/2410.05779) | 2024 arXiv (v3 2025-04, venue 근거 없음) | 그래프+벡터 인덱스에서 저수준(엔티티)과 고수준(주제) 키워드로 이중 검색. 그래프 재구축 없이 새 문서를 증분 삽입 | UltraDomain (Agriculture/CS/Legal/Mix). LLM-judge 승률 | 명시된 한계 없음. [관찰] 증분 갱신은 새 문서 추가만 서술하며 삭제·수정은 다루지 않음(외부 Preprints.org 2025 논문의 지적은 본문 확인 불가) |
| [RepoGraph: Enhancing AI Software Engineering with Repository-level Code Graph](https://arxiv.org/abs/2410.14684) | ICLR 2025 | 코드 줄 단위 노드와 정의-참조 엣지로 저장소 그래프를 만들어 SWE 에이전트에 플러그인. 평균 상대 향상 32.8% | SWE-bench Lite(4개 프레임워크 모두 향상), CrossCodeEval | GPT-4 계열 등 상용 모델만 사용, 비용 때문에 SWE-bench Lite만 실험, Python만 다룸(App. E) |
| [StructRAG: Boosting Knowledge Intensive Reasoning of LLMs via Inference-time Hybrid Information Structurization](https://arxiv.org/abs/2410.08815) | ICLR 2025 (본문 v2·arXiv comment에 venue 근거 없음) | 추론 시점에 router(DPO로 학습)가 표·그래프·알고리즘·카탈로그·청크 중 하나를 골라 문서를 재구조화한 뒤 추론 | Loong(4개 과제), Podcast Transcripts | 명시된 한계 없음. [관찰] 질의마다 구조화 비용이 발생 |
| [CoIR: A Comprehensive Benchmark for Code Information Retrieval Models](https://arxiv.org/abs/2407.02883) | ACL 2025 Main (arXiv comment) | 10개 코드 검색 데이터셋(기존 8, 신규 2), 4개 주 과제·8개 하위 과제, 14개 언어. MTEB/BEIR 스키마와 호환. 검색 모델 10종 평가에서 SOTA도 고전 | 자체 데이터 | 모든 데이터셋이 영어이고, 메타데이터(언어·라이브러리 버전 등)를 쓰지 않는 텍스트 기반 질의에 한정(§7) |
| [CodeRAG-Bench: Can Retrieval Augment Code Generation?](https://arxiv.org/abs/2406.14497) | Findings of NAACL 2025 (본문·arXiv comment에 venue 근거 없음) | 약 9k 코딩 과제와 25M 검색 문서(대회 풀이, 튜토리얼, 라이브러리 문서, StackOverflow, GitHub). 검색기 10종 × LM 10종. 검색기가 유용한 문맥을 자주 놓치고 생성기도 문맥을 잘 쓰지 못함 | 기초 프로그래밍, open-domain, 저장소 수준, 코드 검색 | 과제·언어 다양성이 부족(디버깅 등 미포함, Python 중심)하고 vanilla 검색·리랭킹·생성 위주로 실험(Limitations) |
| [KET-RAG: A Cost-Efficient Multi-Granular Indexing Framework for Graph-RAG](https://arxiv.org/abs/2502.09304) | KDD 2025 (본문 ACM 표기) | KNN 그래프의 PageRank로 핵심 청크만 골라 KG 골격을 만들고, 나머지는 텍스트-키워드 이분 그래프로 처리 | MuSiQue, HotpotQA, RAG-QA Arena(각 500문항). Microsoft GraphRAG와 동등 이상 검색 품질에 인덱싱 비용 10배 이상 절감. 별도 설정에서 인덱싱 비용 약 20% 절감과 함께 생성 품질 최대 +32.4% | 로컬 검색만 다룸. 글로벌 검색 시나리오 확장과 적응형 패러다임은 향후 과제(§7) |
| [ArchRAG: Attributed Community-based Hierarchical RAG](https://arxiv.org/abs/2502.09891) | AAAI 2026 (v4 2026-05) | 속성 기반 커뮤니티로 계층 인덱스를 만들고 C-HNSW로 계층 간 ANN 검색 | specific QA(Multihop-RAG, HotpotQA, NarrativeQA)와 abstract QA(Multihop-RAG 코퍼스 생성 질문). specific 질문에서 정확도 +10%, GraphRAG 대비 토큰 최대 250배 절감 | 명시된 한계 없음. 대규모 코퍼스 처리를 향후 과제로 언급 |
| [RAG vs. GraphRAG: A Systematic Evaluation and Key Insights](https://arxiv.org/abs/2502.11371) | KDD 2026 (v4 2026-11 본문 ACM 표기) | 같은 프로토콜로 비교. RAG는 단일 hop과 세부 질의에, GraphRAG는 multi-hop과 다양성 요약에 유리하며 둘은 상호보완적. Selection/Integration 하이브리드 | QA: NQ, HotpotQA, MultiHop-RAG, NovelQA. 요약: SQuALITY, QMSum, ODSum-story/meeting. MultiHop-RAG 구축 시간 RAG 135s, KG-GraphRAG 7,702s, Community-GraphRAG 5,560s(Table 4) | 요약 평가의 LLM-judge가 후보 제시 순서에 민감(위치 효과). GraphRAG는 구축 비용·검색 지연·저장 공간이 더 들고 그래프 구축 품질에 민감 |
| [From RAG to Memory: Non-Parametric Continual Learning for LLMs (HippoRAG 2)](https://arxiv.org/abs/2502.14802) | ICML 2025 | PPR에 passage 노드를 깊게 통합하고 LLM 필터링(recognition memory)을 추가. 사실, sense-making, 연상 기억 과제를 모두 다룸 | NQ, PopQA, MuSiQue, 2Wiki, HotpotQA, LV-Eval, NarrativeQA. 연상 과제에서 SOTA 임베딩 대비 +7% | 명시된 한계 없음. 긴 대화의 일화 기억 확장을 향후 과제로 언급 |
| [PathRAG: Pruning Graph-based RAG with Relational Paths](https://arxiv.org/abs/2502.14902) | AAAI 2026 | 문제는 정보 부족이 아니라 중복이라고 진단. flow 기반 pruning으로 핵심 관계 경로만 골라 신뢰도 오름차순으로 프롬프트에 배치 | 데이터셋 6종(UltraDomain의 Legal·History·Biology·Mix, SQuALITY, SummScreen, 18만–500만 토큰) × 평가 차원 5개, LLM-judge 승률 | 인덱스 그래프 구축 최적화, 사람 주석 데이터셋 수집, 경로 외 부분구조 탐색을 향후 과제로 제시 |
| [In-depth Analysis of Graph-based RAG in a Unified Framework](https://arxiv.org/abs/2503.04338) | PVLDB 18(13), 2025 | 그래프 구축, 인덱스, operator, 검색·생성의 4단계 통합 프레임워크로 12개 방법 비교. 새 조합 변형 제안 | specific QA 6종(MultihopQA, Quality, PopQA, MusiqueQA, HotpotQA, ALCE)과 abstract QA 5종(Mix, MultihopSum, Agriculture, CS, Legal), 약 0.6M–13.5M 토큰 | 모든 기존 방법이 정적 코퍼스를 가정함(O1). 질의 전에 그래프 품질을 평가하는 문제는 미해결이고 구축 토큰 비용이 큼(O2). 대규모 데이터에서 GraphRAG 커뮤니티 리포트 비용이 감당하기 어려움(L6) |
| [How Significant Are the Real Performance Gains? An Unbiased Evaluation Framework for GraphRAG](https://arxiv.org/abs/2506.06331) | 2025 arXiv (v2 2026-08) | 기존 평가의 두 결함(코퍼스와 무관한 질문, LLM-judge의 위치·길이·시행 편향)을 지적. 그래프-텍스트 기반 질문 생성과 편향 제거 절차로 재평가하니 GraphRAG의 이득이 "훨씬 완만"해짐(LightRAG는 NaiveRAG에 근소하게 역전됨) | MGRAG, LightRAG, FGRAG vs NaiveRAG. Harry Potter 소설, Mix·Agriculture·Music | 자신의 프레임워크도 결함이 있을 수 있다고 명시 |
| [When to use Graphs in RAG: A Comprehensive Analysis for Graph RAG (GraphRAG-Bench)](https://arxiv.org/abs/2506.05690) | ICLR 2026 | 사실 검색, 복합 추론, 문맥 요약, 창의 생성의 난이도 계층 과제와 구조화 수준이 다른 코퍼스. 출발점은 GraphRAG가 실제 과제에서 vanilla RAG보다 못한 경우가 많다는 점(예: Han et al. 인용 NQ 정확도 −13.4%) | GraphRAG-Bench(Novel, Medical 데이터셋) | 텍스트 단일 모달만 다루며 표·도식·시계열 같은 멀티모달 통합은 향후 과제(App. J) |
| [TARGET: Benchmarking Table Retrieval for Generative Tasks](https://arxiv.org/abs/2505.11545) | 2025 arXiv (venue 근거 없음) | QA, 사실 검증, text-to-SQL 하류 과제로 표 검색 평가. dense가 BM25보다 크게 우세하나 과제·데이터셋 간 편차가 크고, 표 제목 같은 메타데이터가 없으면 성능이 민감하게 떨어짐 | OTTQA, FeTaQA, TabFact, Spider, BIRD | 소스 코드가 없는 기존 표 검색 방법 일부를 포함하지 못함. 열·행 단위 관련성 라벨이 없어 표 내부 검색은 간접 평가만 가능(§7) |
| [EraRAG: Efficient and Incremental RAG for Growing Corpora](https://arxiv.org/abs/2506.20963) | 2025 arXiv (Under review) | 초평면 LSH로 계층 그래프를 만들어 국소 삽입만으로 증분 갱신 | PopQA, MultiHopQA, HotpotQA, QuALITY, MuSiQue. 5%씩 10라운드 삽입. 차선 기준선 대비 토큰 −57.6%(PopQA), 재구축 시간 −77.5%(QuALITY) | 명시된 한계 없음. 기존 방법들이 정적 코퍼스를 가정한다고 지적. [관찰] 삭제와 수정은 다루지 않음 |
| [LinearRAG: Linear Graph RAG on Large-scale Corpora](https://arxiv.org/abs/2510.10114) | 2025 arXiv (v4 2025-11, venue 근거 없음) | 관계 추출 없이 엔티티 추출과 의미 링크만으로 Tri-Graph(엔티티·문장·passage)를 구성. 인덱싱 시간 77% 이상 절감 | HotpotQA, 2Wiki, MuSiQue, Medical(GraphRAG-Bench) | 명시된 한계 없음. 관계 추출 오류가 노이즈 그래프와 모호성을 낳는다는 것이 문제의식 |
| [RAG Meets Temporal Graphs: Time-Sensitive Modeling and Retrieval for Evolving Knowledge (TG-RAG)](https://arxiv.org/abs/2510.13590) | 2025 arXiv | 타임스탬프 관계 KG와 계층 시간 그래프의 이중 수준 시간 그래프. 같은 사실의 시점별 값을 별도 엣지로 유지하고, 새 코퍼스의 사실을 추출·병합하는 증분 갱신 | ECT-QA(실적 발표 녹취, 2020–2024, 증분 평가 프로토콜 포함), 일반 QA(NQ, 2Wiki, HotpotQA) | 명시된 한계 없음. 기존 평가가 갱신 비용과 검색 안정성을 보지 않는다고 지적 |
| [WildGraphBench: Benchmarking GraphRAG with Wild-Source Corpora](https://arxiv.org/abs/2602.02053) | 2026 arXiv (본문 "Preprint. Work in progress.", venue 근거 없음) | Wikipedia 인용 문장을 정답으로, 인용된 외부 원문을 코퍼스로 씀. 질문 1,100개(초록 기준, 통계표 합계 1,197). 단일 사실에서는 NaiveRAG가 대부분의 GraphRAG보다 우세, 다중 사실에서는 GraphRAG(global)가 최고, 요약에서는 그래프의 집계가 세부 정보를 희생 | 단일 사실, 다중 사실, 섹션 요약 | 정답이 Wikipedia 편집 합의를 반영하므로 누락·오류를 물려받을 수 있음. LLM 판정과 문장 매칭에 체계적 편향이 있을 수 있음 |
| [BM25 Wins at Scale: A Scaling Study of Retrieval-Augmented Generation Paradigms](https://arxiv.org/abs/2607.26497) (v1 제목 "Which RAG Paradigm Wins at Scale?"은 본문 확인 불가) | 2026 arXiv (v3 2026-07) | 어휘·dense·그래프·에이전트 RAG를 28단계(약 450배) 중첩 코퍼스에서 비교. 약 10M 토큰에서 BM25가 File-System Agent를 추월한 뒤 모든 공유 규모에서 선두(전체 규모 격차 약 20점)이고, 저비용 Pareto 끝점을 차지. 그래프 RAG는 배포 규모 전에 구축 한계에 부딪힘 | EnterpriseRAG-Bench(511,957 문서, 600.8M 토큰, 500문항) | 명시된 한계 없음. 본문 관찰: 구축이 LightRAG는 3.0M 토큰(전체의 약 0.5%), MS-GraphRAG는 10.6M 토큰(약 1.8%)에서 멈추고 HippoRAG 2·LinearRAG는 약 155M 토큰까지 감. LightRAG의 구축 비용은 코퍼스 토큰당 약 24 LLM 토큰(Table 2 기준 계산) |
| [LazyGraphRAG (Microsoft Research blog)](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/) | 2024-11 산업 블로그 | 인덱싱 시 LLM 요약을 생략하고 NLP 명사구 추출로 개념 그래프를 만든 뒤 LLM 사용을 질의 시점으로 지연. 인덱싱 비용은 vector RAG와 같고 full GraphRAG의 0.1%. 글로벌 질의에서 GraphRAG Global과 비슷한 품질을 질의 비용 700배 이상 낮게 달성 | AP 뉴스 5,590건, 합성 질의 100개(로컬 50·글로벌 50), LLM 쌍대 비교 | 모든 그래프 RAG가 lazy여야 하는 것은 아니라고 명시: 요약 인덱스는 QA 외 활용 가치가 있고, 요약 인덱스와 lazy 검색을 결합하는 편이 더 나을 것으로 봄. [관찰] 비용이 질의 시점으로 옮겨감 |

### B-3. 반복되는 미해결 문제, 암묵적 가정, 벤치마크 공백
1. **평가 프로토콜의 신뢰성.** GraphRAG 계열(GraphRAG, LightRAG, PathRAG)의 주장은 주로 LLM-judge 쌍대 승률에 기반합니다. 그런데 위치·길이 편향을 제거하고 코퍼스에 근거한 질문을 쓰면 이득이 크게 축소됩니다(Unbiased Eval 2506.06331, RAG vs GraphRAG의 judge 순서 효과). UltraDomain 같은 정답 없는 요약 평가에 지나치게 의존하는 것도 문제입니다.
2. **"그래프 = 이득"이라는 가정이 조건부로 깨짐.** 단일 사실에서는 RAG가 우세하고, multi-hop과 요약에서는 조건부로 그래프가 우세합니다(RAG vs GraphRAG, GraphRAG-Bench, WildGraphBench). 그러나 어떤 코퍼스·질의 특성이 그래프의 이득을 예측하는지 보여주는 정량 모델은 없습니다. GraphRAG-Bench가 "더 조밀한 그래프(HippoRAG2)가 연결성과 커버리지를 높여 성능에 기여한다"는 정성적 관찰(Obs.7)을 제시한 정도입니다.
3. **인덱싱 비용과 스케일 한계.** LLM 기반 추출은 비용이 매우 큽니다(GraphRAG, KET-RAG, ArchRAG, LazyGraphRAG, Unified Framework L6). 코퍼스가 커지면 BM25가 이기고, LLM 추출형 그래프 빌더는 수백만~수천만 토큰에서 구축이 멈춥니다(BM25 Wins at Scale). 대부분의 그래프 RAG 실험 코퍼스는 수십만~수백만 토큰 규모입니다(GraphRAG 약 1–1.7M, KET-RAG 0.6–1.4M, PathRAG 0.18–5M, 가장 큰 축인 Unified Framework도 최대 약 13.5M).
4. **정적 코퍼스와 삽입 전용 갱신.** Unified Framework는 기존 그래프 RAG 방법이 모두 정적 코퍼스를 가정한다고 명시합니다(O1). LightRAG와 EraRAG는 삽입만 다룹니다. TG-RAG는 시점별 사실을 별도 엣지로 보존하며 증분 병합하지만, 삭제·수정·무효화는 다루지 않습니다(Preprints.org 2025 논문의 관련 지적은 본문 확인 불가). 커뮤니티 요약이 낡는 문제(stale summary)를 측정하는 벤치마크도 없습니다.
5. **그래프 구축 오류의 전파.** OpenIE와 엔티티·관계 추출의 노이즈가 검색 노이즈로 이어집니다(LinearRAG의 관계 추출 노이즈, PathRAG의 중복 진단, WildGraphBench의 노이즈 코퍼스에서 불완전한 구축이 요약 재현율을 떨어뜨림, HippoRAG 오류의 대부분이 NER·OpenIE). 부분적 분석은 있습니다. HippoRAG는 OpenIE 내재 평가와 검색 성능을 함께 보고했고, RAG vs GraphRAG는 구축 LLM을 바꿔 가며 영향을 쟀습니다(§4.7). 그러나 구축 품질과 하류 성능의 관계를 체계적으로 분해해 측정한 연구는 부족하며, Unified Framework도 질의 전 그래프 품질 평가를 미해결 문제로 꼽습니다(O2).
6. **도메인 사일로.** 텍스트 GraphRAG, 표 검색(TARGET), 코드 그래프(RepoGraph, CoIR, CodeRAG-Bench)가 서로 다른 벤치마크와 인덱스 개념을 씁니다. 표와 코드처럼 구조가 원래 존재하는 데이터에서 LLM이 만든 KG가 필요한지는 검증된 적이 없습니다.

### B-4. 덜 탐구된 연구 공백 (구체안)
1. **"그래프 이득 예측기": 인덱싱 전에 코퍼스·질의 특성으로 GraphRAG의 기대 이득을 추정하기.**
   - 내용: 엔티티 중복도, 문서 간 공유 엔티티 비율, 질의 hop 수 분포 같은 값싼 통계로 "그래프를 지을 가치"를 예측하고, 지을 경우 어떤 그래프 유형을 쓸지 정합니다.
   - 가장 가까운 연구: GraphRAG-Bench(사후 분석), RAG vs GraphRAG의 Selection 하이브리드(질의 시점), Unified Framework(VLDB, O2에서 "질의 전 그래프 품질 평가"를 오픈 문제로 명시).
   - 왜 미해결인가: 모두 그래프를 다 짓고 난 뒤 비교합니다. 인덱싱 전 의사결정이나 비용 예측은 다루지 않습니다.
2. **편향을 통제하고 정답이 있는 글로벌 sensemaking 벤치마크.**
   - 내용: 글로벌 질의에 검증 가능한 정답을 붙이는 방식입니다. 예를 들어 집계·계수형 질문이나, 코퍼스 생성 과정에서 심어 둔 주제 분포를 쓸 수 있습니다. LLM-judge 승률 대신 이런 정답으로 평가합니다.
   - 가장 가까운 연구: Unbiased Eval(편향 제거와 코퍼스 기반 질문 생성, 여전히 judge 사용), WildGraphBench(Wikipedia 문장을 정답으로 사용, 섹션 요약 포함. 단 판정은 LLM과 문장 매칭이며 저자도 편향 가능성을 한계로 명시), GraphRAG-Bench.
   - 왜 미해결인가: "포괄성·다양성"이라는 GraphRAG 본래의 주장 영역은 여전히 judge 의존적입니다.
3. **수정·삭제·시간 무효화를 포함한 동적 그래프 인덱스와 갱신 비용 벤치마크.**
   - 내용: 엣지와 요약에 출처 유효성(provenance)을 기록해, 커뮤니티 요약을 국소적으로 무효화하고 다시 생성합니다. "갱신 후 낡은 답변 비율"을 지표로 삼습니다.
   - 가장 가까운 연구: LightRAG(삽입), EraRAG(삽입, LSH), TG-RAG(시점별 엣지 보존과 증분 병합, ECT-QA 증분 평가), LazyGraphRAG(요약을 질의 시점으로 미뤄 문제를 우회), Unified Framework(O1에서 동적 코퍼스를 오픈 문제로 제기).
   - 왜 미해결인가: 삭제와 수정 이후의 일관성, 그리고 갱신 토큰 비용 대비 품질 곡선을 함께 다룬 표준 평가가 없습니다.
4. **스케일 인지 하이브리드: BM25/dense를 기본으로 하고 그래프를 선택적으로 국소 구축하기.**
   - 내용: 10M 토큰 이상 코퍼스에서는 전체 그래프를 짓지 않습니다. 대신 질의 로그상 hot 영역이나 multi-hop 질의가 자주 닿는 문서 군집에만 그래프를 지연·국소 구축합니다.
   - 가장 가까운 연구: KET-RAG(PageRank로 핵심 청크만 KG화, 정적. 적응형 패러다임은 향후 과제로 언급), LazyGraphRAG(질의 시점 처리), GraphRAG 원 논문 §6.2(임베딩 매칭과 just-in-time 커뮤니티 리포트를 결합한 하이브리드를 향후 방향으로 제시), BM25 Wins at Scale(문제 제기).
   - 왜 미해결인가: 질의 분포에 적응하는 부분 그래프 구축과 그 비용-정확도 Pareto를 대규모 코퍼스에서 실험한 연구는 확인되지 않았습니다.
5. **원래 구조가 있는 데이터(표·코드)에서 LLM 추출 KG vs 원래 구조 그래프 비교.**
   - 내용: 표 스키마나 외래키, 코드의 호출·정의-참조 그래프를 그대로 인덱스로 쓰는 경우와 LLM으로 엔티티 KG를 추출하는 경우를 같은 과제에서 비교합니다. 혼합 문서(표가 포함된 보고서, 코드와 문서)에서의 통합 인덱스도 포함합니다.
   - 가장 가까운 연구: RepoGraph(원래 구조, Python·SWE-bench Lite 한정), TARGET(표 검색 평가만), StructRAG(추론 시점 구조화), Financial Report Chunking(요소 단위).
   - 왜 미해결인가: 텍스트 GraphRAG와 표·코드 검색 커뮤니티가 분리되어 있어 공통 벤치마크와 비교가 없습니다. GraphRAG-Bench도 텍스트 단일 모달 한정을 한계로 밝혔습니다.

---

### 부록: 검증 상태 요약
- **축 B 본문 전수 확인.** 축 B 표의 24개 항목(arXiv 23편과 LazyGraphRAG 블로그)은 로컬 본문 전문(pdftotext)과 arXiv 메타데이터로 서지, 핵심 기여, 수치, 벤치마크, 저자 명시 한계를 대조했습니다. 본문에 접근하지 못한 항목은 없습니다. 상세 대조표는 별도 검증 파일(b02b_verify.md)에 있습니다.
- **본문 헤더·각주 또는 arXiv comment로 확인한 venue:** HippoRAG(NeurIPS'24), HippoRAG 2(ICML'25), RepoGraph(ICLR'25), CoIR(ACL'25 Main), KET-RAG(KDD'25), Unified Framework(PVLDB 18(13)), ArchRAG(AAAI'26), PathRAG(AAAI'26), GraphRAG-Bench(ICLR'26), RAG vs GraphRAG(KDD'26, v4), A-Mem(NeurIPS'25).
- **본문·comment에 venue 근거 없음:** G-Retriever, GRAG, StructRAG, CodeRAG-Bench(이상 이전 웹 검색 결과의 venue 표기 유지), LightRAG, TARGET, LinearRAG, WildGraphBench(본문은 "Preprint. Work in progress."). arXiv 전용: GraphRAG, Survey, Unbiased Eval, EraRAG, TG-RAG, BM25 Wins at Scale.
- **본문 확인으로 바뀐 주요 사항:** KET-RAG·ArchRAG·RAG vs GraphRAG의 venue 확정, WildGraphBench의 Findings ACL 표기 철회, TG-RAG 정식 제목 교정, KET-RAG 수치의 설정 구분(10배 절감과 +32.4%는 다른 설정), BM25 Wins at Scale의 "24.6 토큰/2%" 수치는 v3 본문에 없어 Table 2 기준 값으로 교체, 저자가 명시한 한계 열 전면 교체.
- **여전히 본문 확인 불가:** BM25 Wins at Scale의 v1 제목, LightRAG의 삭제·수정 미지원을 지적한 Preprints.org 2025 논문(로컬 본문 없음).
- **축 A 항목**(Dense X, LumberChunker, RAPTOR, Self-Route, Long-Context LLMs Meet RAG, SiReRAG, LaRA, MoC, MemoRAG, Late Chunking, LongRAG, HiChunk 등)의 venue와 수치는 축 A 검증 결과에 따라 별도로 갱신합니다.
- 메모리형 인덱스 관련: [A-Mem: Agentic Memory for LLM Agents](https://arxiv.org/abs/2502.12110) (NeurIPS 2025, arXiv comment로 확인)는 Zettelkasten식 노트 연결(link generation)과 memory evolution을 제안합니다. 축 A의 메모리형 인덱스 범주에 해당합니다.
