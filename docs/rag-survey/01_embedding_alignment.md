# RAG 검색/인덱싱 측 문헌 조사: (A) 표현·임베딩, (B) Retriever–LLM 정렬

작성일: 2026-11-19

## 검증 방법과 주의 (꼭 읽어 주세요)
- 이 세션에서는 WebFetch가 arxiv.org, aclanthology.org, huggingface.co 를 모두 열지 못했습니다(네트워크 허용 목록 밖). curl로 arXiv API에 접근해도 프록시가 403으로 막았습니다.
- 그래서 **모든 서지(제목, 저자, 연도, venue, arXiv ID)는 WebSearch 결과로만 확인했습니다.** 검색 결과에는 arXiv, ACL Anthology, ICLR/NeurIPS 프로시딩 페이지가 1차 출처로 나왔습니다. **PDF 본문은 직접 열지 못했습니다.**
- 수치는 검색 결과에 나온 초록 수준의 값만 적었습니다. 2차 요약 사이트에서만 나온 수치나 서술에는 "(2차 출처)"를 붙였습니다.
- "Stated limitations" 열은 원래 본문의 Limitations 절을 읽어야 채울 수 있습니다. 초록이나 검색 결과에서 확인된 것만 적었고, 그 밖에는 **미확인**으로 표시했습니다.
- 링크는 검색 결과에 나온 arXiv abs 또는 공식 프로시딩/Anthology URL입니다.

---

## 축 A. 표현 / 임베딩

### A-1. 현황 요약
LLM 백본 단일 벡터 임베더가 사실상 표준이 되었습니다. 대표적으로 E5-Mistral, LLM2Vec, NV-Embed, Gemini Embedding, Qwen3-Embedding이 있습니다. 공통 레시피도 굳어졌습니다. 양방향 attention 또는 pooling을 바꾸고, LLM으로 합성 데이터를 만들고, 다단계 대조학습을 하는 방식입니다. 그 결과 MTEB/MMTEB 점수는 포화 단계에 들어섰고, MMTEB에서는 560M 규모 모델(multilingual-e5-large-instruct)이 공개 모델 최고였습니다.

움직임이 가장 큰 곳은 두 곳입니다. 첫째는 **추론 집약 검색(reasoning-intensive retrieval)**입니다. BRIGHT 공개 당시 MTEB 59.0 모델이 BRIGHT에서 nDCG@10 18.3에 그쳤는데, 2025~2026년 ReasonIR(29.9)와 ReasonEmbed(38.1) 같은 전용 학습으로 빠르게 따라잡고 있습니다. 둘째는 **표현력의 이론적 한계**입니다. LIMIT(2025)이 단일 벡터의 top-k 부분집합 표현 한계를 sign-rank로 주장했고, 2026년 반론이 "차원이 아니라 drowning 효과"라고 재해석했습니다. Late interaction(ColBERT 계열, MUVERA)과 learned sparse(SPLADE-v3)는 효율화 단계에 있어 성숙한 편입니다. 다만 LIMIT 논쟁과 에이전트 검색에서 BM25가 강세를 보이면서(SAGE) 다시 주목받고 있습니다.

### A-2. 논문 표

| Title | Year/venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Improving Text Embeddings with Large Language Models](https://aclanthology.org/2024.acl-long.642/) (Wang et al.) | 2024 / ACL | Mistral-7B + LoRA. 독점 LLM으로 93개 언어, 수십만 태스크의 합성 데이터를 만들고 1k step 미만으로 학습(E5-Mistral). | MTEB(합성만 63.1, +레이블 66.6), BEIR | 독점 LLM 합성 데이터에 의존. 그 밖은 미확인 |
| [LLM2Vec: Large Language Models Are Secretly Powerful Text Encoders](https://arxiv.org/abs/2404.05961) | 2024 / COLM | 양방향 attention + MNTP + 비지도 SimCSE로 decoder LLM을 인코더로 전환 | MTEB(비지도 SOTA) | 미확인 |
| [NV-Embed: Improved Techniques for Training LLMs as Generalist Embedding Models](https://arxiv.org/abs/2405.17428) | 2025 / ICLR Spotlight | latent-attention pooling, causal mask 제거, 2단계 instruction 대조학습, 공개 데이터만 사용 | MTEB(v1 69.32, v2 72.31) | 미확인. 압축(가지치기, 양자화, 증류) 분석을 포함 |
| [M3-Embedding (BGE-M3)](https://aclanthology.org/2024.findings-acl.137/) | 2024 / Findings ACL | 한 모델에서 dense, sparse, multi-vector를 동시 지원. self-knowledge distillation, 100+개 언어, 8192 토큰 | MIRACL, MKQA | 미확인 |
| [Gemini Embedding: Generalizable Embeddings from Gemini](https://arxiv.org/abs/2503.07891) | 2025 / arXiv(tech report) | Gemini LLM에서 초기화한 대조학습 임베더 | MMTEB 1위(Borda), XOR-Retrieve SOTA | 폐쇄 모델이라 재현 불가(관찰). 명시된 한계는 미확인 |
| [Qwen3 Embedding](https://arxiv.org/abs/2506.05176) | 2025 / arXiv(tech report) | Qwen3 0.6B/4B/8B 임베딩과 리랭커. Qwen3로 대규모 합성 쌍 생성(약 1.5억 쌍, 2차 출처). Apache 2.0 | MMTEB multilingual 70.58, MTEB Code 80.68 | 미확인 |
| [MMTEB: Massive Multilingual Text Embedding Benchmark](https://arxiv.org/abs/2502.13595) | 2025 / ICLR | 500개 이상 태스크, 250개 이상 언어. instruction following, 장문서, 코드 검색 태스크 추가 | 자체 | 공개 모델 1위가 560M이라는 결과는 "LLM 임베더 우위"가 언어/태스크에 따라 다르다는 점을 시사 |
| [SPLADE-v3: New baselines for SPLADE](https://arxiv.org/abs/2403.06789) | 2024 / arXiv | SPLADE 학습 개선 baseline 세트(Doc, Lexical 변형 포함) | MS MARCO(MRR@10 40.2), BEIR-13(51.7, 모델카드) | 저자가 "새로움이 부족해 정식 논문 대신 보고서로 낸다"고 밝힘 |
| [Jina-ColBERT-v2](https://arxiv.org/abs/2408.16672) | 2024 / arXiv(EMNLP 워크숍) | 다국어 late interaction. Matryoshka식 projection head로 저장 공간을 최대 50% 절감 | BEIR, MIRACL, mMARCO | 미확인 |
| [MUVERA: Multi-Vector Retrieval via Fixed Dimensional Encodings](https://proceedings.neurips.cc/paper_files/paper/2024/hash/b71cfefae46909178603b5bc6c11d3ae-Abstract.html) | 2024 / NeurIPS | multi-vector 유사도를 근사하는 고정 차원 단일 벡터 인코딩. 이론적 보장 있음 | BEIR(recall +10%, latency −90%) | 미확인 |
| [SMEC: Rethinking Matryoshka Representation Learning for Retrieval Embedding Compression](https://arxiv.org/abs/2510.12474) | 2025 / EMNLP(arXiv 코멘트 기준) | 순차적 MRL, 적응적 차원 선택, cross-batch memory로 LLM 임베딩 차원 압축 | BEIR 등(미확인) | 미확인 |
| [BRIGHT: A Realistic and Challenging Benchmark for Reasoning-Intensive Retrieval](https://arxiv.org/abs/2407.12883) | 2025 / ICLR | 12개 도메인(StackExchange, 수학, 코드 등), 약 1.4k개 질의. 추론이 필요한 관련성 | 자체(SFR-Mistral 18.3) | 쿼리 수가 적음(관찰). 정확한 질의 수는 버전마다 1,384/1,398로 다르게 보고됨 |
| [FollowIR](https://aclanthology.org/2025.naacl-long.597/) | 2025 / NAACL | TREC narrative 기반 instruction 변경 쌍. pairwise p-MRR 평가, FollowIR-7B 공개 | FollowIR | 기존 모델이 instruction을 "키워드"로만 사용하고 긴 instruction에 약하다는 진단 |
| [Promptriever](https://proceedings.iclr.cc/paper_files/paper/2025/hash/2cefdb2c4c3274b78cd450bac35228df-Abstract-Conference.html) | 2025 / ICLR | MS MARCO에 인스턴스 단위 instruction 약 50만 개와 instruction negative를 더한 bi-encoder | FollowIR(+14.3 p-MRR), InstructIR, BEIR | 미확인 |
| [ReasonIR: Training Retrievers for Reasoning Tasks](https://arxiv.org/abs/2504.20595) | 2025 / arXiv(COLM 2025로 알려져 있으나 미확인) | 문서별로 어려운 추론형 질의와 "그럴듯하지만 무용한" hard negative를 합성(ReasonIR-8B) | BRIGHT 29.9(리랭커 포함 36.9), MMLU/GPQA RAG | 미확인 |
| [ReasonEmbed](https://arxiv.org/abs/2510.08252) | 2025 arXiv → 2026 / ACL | ReMixer 합성 82K + Redapter(추론 강도에 따른 샘플 가중) | BRIGHT 38.1(8B), 37.1(4B) | 미확인 |
| [On the Theoretical Limitations of Embedding-Based Retrieval](https://arxiv.org/abs/2508.21038) (Weller et al.) | 2025 / arXiv(v2 2026-03) | 반환 가능한 top-k 부분집합 수가 차원(sign-rank)에 묶인다고 주장. LIMIT 데이터셋 제시 | LIMIT(SOTA 임베더가 매우 낮음, BM25는 높음, 2차 출처) | 이론은 단일 벡터에만 적용. multi-vector와 hybrid는 open question |
| [On Strengths and Limitations of Single-Vector Embeddings](https://arxiv.org/abs/2603.29519) | 2026 / arXiv | Alon et al.(2016)에 따르면 2k+1차원이면 충분(LIMIT은 5차원). 실패 원인을 "drowning"으로 재해석 | LIMIT fine-tune(단일 약 40%, multi 약 98% R@10) | 본문 미열람으로 정리 형태는 미확인 |
| [Optimized Text Embedding Models and Benchmarks for Amharic Passage Retrieval](https://arxiv.org/abs/2505.19356) | 2025 / Findings ACL | 암하라어 검색 벤치마크와 전용 dense retriever. 다국어 모델보다 우수 | 자체(MRR@10 상대 +17.6%, 2차 출처) | 미확인 |

### A-3. 반복되는 오픈 문제, 암묵적 가정, 벤치마크 공백
1. **MTEB/BEIR 점수와 실제 난이도가 어긋남.** MTEB 고득점이 추론형 검색 성능을 보장하지 않습니다. 근거: BRIGHT(59.0 → 18.3), MMTEB(소형 모델 1위), BRIGHT-PRO(2605.04018, 정적 순위와 에이전트 성능의 상관이 느슨함).
2. **"관련성 = 의미 유사도"라는 암묵적 가정.** bi-encoder 대조학습은 쿼리-문서 쌍의 대칭적 유사도를 학습합니다. 반면 추론형 관련성은 비대칭적이고 다단계입니다. 근거: BRIGHT, ReasonIR(학습 데이터가 짧은 사실형 질의에 편중), RaDeR, DIVER(2508.07995. 제목과 BRIGHT 수치는 서베이 경유로만 확인).
3. **합성 데이터 의존과 평가 오염 위험.** E5-Mistral, Qwen3-Embedding, ReasonIR, ReasonEmbed가 모두 LLM 합성 데이터에 크게 기댑니다. 합성기와 평가 벤치마크(StackExchange 기반 BRIGHT)의 분포가 겹칠 수 있는데, 이를 체계적으로 점검한 연구는 찾지 못했습니다(관찰).
4. **표현력 한계의 원인 논쟁이 미결.** 차원 탓인지(Weller 2025), 학습 동역학이나 drowning 탓인지(2603.29519)가 갈립니다. multi-vector와 sparse의 이론적 경계는 공식적으로 제시되지 않았습니다(Weller 2025가 open question으로 명시).
5. **압축(MRL, 양자화)과 추론형 성능의 상호작용 미평가.** SMEC, MRL, Jina-ColBERT-v2, NV-Embed 압축 분석은 모두 BEIR/MTEB 기준입니다. Qwen3-VL-Embedding 보고서(2601.04720)는 int8은 거의 손실이 없지만 binary는 크게 손상된다고 보고했습니다. 그러나 압축이 BRIGHT나 LIMIT형 조합 질의에서 성능을 비대칭적으로 무너뜨리는지는 다루지 않았습니다.
6. **다국어 추론형 검색 벤치마크 부재.** MMTEB는 언어 폭은 넓지만 추론형 태스크가 없습니다. BRIGHT와 BRIGHT-PRO는 영어 중심이고, 저자원 언어 연구(암하라어, IndicRAGSuite 2506.01615)는 사실형 QA 수준입니다.

### A-4. 덜 탐구된 연구 공백
1. **추론 집약 검색에서 압축(MRL 절단, int8, binary)의 비대칭적 손실 측정과 압축 인지 학습.**
   가장 가까운 작업은 SMEC(2025)와 Qwen3-VL-Embedding 보고서입니다. 둘 다 사실형 또는 MTEB 지표만 봅니다. ReasonEmbed와 ReasonIR은 full-precision 4B/8B 모델만 보고합니다. 추론 신호가 하위 차원이나 저비트에서 먼저 사라지는지는 아무도 측정하지 않았습니다. 배포 비용이 큰 8B 추론형 임베더에서 실용적 가치가 큰 질문입니다.
2. **LIMIT 논쟁을 실제 코퍼스로 옮기기: "조합적 관련성"을 측정하는 자연 벤치마크와 진단 도구.**
   가장 가까운 작업은 LIMIT(합성)과 2603.29519(LIMIT fine-tune)입니다. 실제 RAG 질의 중 몇 %가 sign-rank 또는 drowning 병목을 겪는지 정량화한 연구는 없습니다. 쿼리별로 "단일 벡터로 표현 불가" 여부를 판정해 multi-vector나 sparse로 라우팅하는 방법도 비어 있습니다.
3. **다국어·저자원 추론 집약 검색.**
   가장 가까운 작업은 BRIGHT(영어)와 MMTEB(사실형 중심)입니다. ReasonIR/ReasonEmbed식 합성 파이프라인이 저자원 언어로 옮겨지는지, 아니면 번역 노이즈 때문에 "그럴듯한 hard negative"가 무너지는지 검증되지 않았습니다.
4. **추론형 late interaction과 learned sparse.**
   추론형 검색기는 거의 전부 단일 벡터 LLM 임베더입니다(ReasonIR, ReasonEmbed, RTriever). 서베이 2605.00063의 BRIGHT 표도 "Single Vector"로 분류합니다. LIMIT 연구는 multi-vector의 우위를 시사하지만, 추론형 합성 데이터로 학습한 ColBERT/SPLADE를 BRIGHT에서 체계적으로 비교한 연구는 찾지 못했습니다(미확인일 수 있음).
5. **합성 데이터와 벤치마크 오염 감사.** ReasonIR, ReasonEmbed, DIVER가 BRIGHT 점수를 빠르게 올렸지만 합성 원천과 BRIGHT StackExchange 도메인이 겹치는지 점검한 작업은 찾지 못했습니다. held-out 도메인 BRIGHT 변형이 필요합니다.

---

## 축 B. Retriever–LLM 정렬

### B-1. 현황 요약
"LLM 출력으로 retriever를 감독한다"는 기본 패러다임은 2023~2024년에 확립됐습니다. REPLUG LSR은 블랙박스 LM likelihood를 써서 KL을 맞추고, RA-DIT은 retriever와 LM을 따로 튜닝하며, ARL2는 LLM이 관련성 레이블러 역할을 합니다. 2024~2025년에는 축이 두 방향으로 움직였습니다. 하나는 **"관련성 vs 유용성(utility)"을 분리하는 측정과 레이블**입니다(Utility judgments SIGIR'24, eRAG, Power of Noise, Utility-focused annotation EMNLP'25, UDCG EACL'26). 다른 하나는 **선호/이득 신호 기반 정렬**입니다(DPA-RAG, GainRAG). 2026년에는 **에이전트(R1식 검색 학습) 루프 안에서 retriever를 학습**하는 흐름이 나타났습니다(Agentic-R, Critic-R, BRIGHT-PRO/RTriever, CoSearch). 하지만 Search-R1 계열 대부분은 여전히 retriever를 고정된 환경으로 취급합니다. SAGE와 DCI는 에이전트의 서브쿼리 분포가 기존 dense retriever 학습 분포와 맞지 않는다는 증거를 냅니다.

### B-2. 논문 표

| Title | Year/venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [REPLUG: Retrieval-Augmented Black-Box Language Models](https://aclanthology.org/2024.naacl-long.463/) | 2024 / NAACL(arXiv 2023) | 블랙박스 LM의 출력 점수로 retriever를 감독(LSR) | LM perplexity, MMLU(Codex +5.1%) | 미확인 |
| [RA-DIT: Retrieval-Augmented Dual Instruction Tuning](https://arxiv.org/abs/2310.01352) | 2024 / ICLR | LM은 검색 결과 활용을 학습하고, retriever는 LM이 유용하다고 본 결과로 학습(2단계 분리) | KILT 계열 등 지식 집약 0/5-shot(+8.9%/+1.4%) | 미확인 |
| [ARL2](https://aclanthology.org/2024.acl-long.203/) | 2024 / ACL | LLM이 근거 관련성을 레이블링하고 self-training으로 데이터를 큐레이션 | NQ(+5.4%), MMLU(+4.6%) | 미확인 |
| [Are Large Language Models Good at Utility Judgments?](https://arxiv.org/abs/2403.19216) | 2024 / SIGIR | LLM이 관련성과 유용성을 구분할 수 있음을 보임. 반사실 passage에 취약. k-sampling listwise 방법 제안 | NQ 등 QA(미확인) | LLM이 새로 생성된 반사실 passage를 쉽게 받아들임 |
| [Evaluating Retrieval Quality in Retrieval-Augmented Generation (eRAG)](https://arxiv.org/abs/2404.13781) | 2024 / SIGIR(short) | 문서 하나만 넣었을 때의 LLM 성능을 그 문서의 관련성 레이블로 사용 | KILT 계열(Kendall τ +0.168~0.494) | 문서별 독립 평가라 문서 간 상호작용을 반영하지 못함(관찰) |
| [The Power of Noise: Redefining Retrieval for RAG Systems](https://arxiv.org/abs/2401.14887) | 2024 / SIGIR | distracting 문서 하나로 정확도 최대 25% 하락. random 문서는 최대 35% 향상 | NQ-open | random 이득은 문서 수가 적을 때만 나타남(2차 출처) |
| [Understand What LLM Needs: Dual Preference Alignment for RAG (DPA-RAG)](https://arxiv.org/abs/2406.18676) | 2025 / WWW | reranker 외부 정렬(pair/point/contrastive)과 LLM 내부 사전정렬. 5가지 쿼리 증강 | 4개 지식 QA | 선호 데이터가 부족함(증강의 동기) |
| [GainRAG](https://aclanthology.org/2025.acl-long.527/) | 2025 / ACL | 문서가 생성에 주는 "이득(gain)" 신호를 합성해 retriever와 LLM 선호 간극을 정렬 | QA(미확인) | 미확인 |
| [Utility-Focused LLM Annotation for Retrieval and RAG](https://aclanthology.org/2025.emnlp-main.88/) | 2025 / EMNLP | Qwen2.5-32B로 MS MARCO utility 레이블 생성, Disj-InfoNCE | MS MARCO, BEIR, NQ, HotpotQA | in-domain에서는 인간 레이블을 대체하지 못함. 인간 레이블 20%를 섞으면 동등 |
| [Redefining Retrieval Evaluation in the Era of LLMs (UDCG)](https://arxiv.org/abs/2510.21440) | 2026 / EACL | 위치 할인과 relevance-utility 불일치를 지적. 유용성과 방해(distraction)를 반영한 지표 | NQ, TriviaQA, PopQA, BioASQ, NoMIRACL, 6개 LLM(상관 최대 +36%) | γ 하이퍼파라미터 하나(1/3)에 의존 |
| [Search-R1](https://arxiv.org/abs/2503.09516) | 2025 / arXiv(COLM 2025로 알려짐, 미확인) | RL로 LLM이 검색을 교차 호출하도록 학습. retriever는 고정 환경 | 7개 QA | retriever는 최적화 대상이 아님(참고용) |
| [An Empirical Study on RL for Reasoning-Search Interleaved LLM Agents](https://arxiv.org/abs/2505.15117) | 2025 / arXiv | 중간 retrieval 보상의 효과는 제한적이지만, 검색 엔진 선택이 RL 학습 동역학을 크게 좌우 | QA | 미확인 |
| [Agentic-R: Learning to Retrieve for Agentic Search](https://arxiv.org/abs/2601.11888) | 2026 / arXiv | 로컬 관련성과 글로벌 정답 정확도로 passage에 점수. 에이전트와 retriever를 번갈아 양방향 학습 | 7개 단일/멀티홉 QA | 미확인 |
| [Rethinking Reasoning-Intensive Retrieval (BRIGHT-PRO, RTriever)](https://arxiv.org/abs/2605.04018) | 2026 / ACL | 다측면 gold evidence와 정적/에이전트 이중 평가. RTriever-4B(Qwen3-Emb-4B LoRA) | BRIGHT-PRO | 정적 순위와 에이전트 성능의 상관이 느슨함 |
| [Critic-R](https://arxiv.org/abs/2606.00590) | 2026 / Findings EMNLP | critic이 근거 충분성을 판단해 쿼리와 instruction을 재작성. 실패 궤적으로 Critic-Embed 학습 | HotpotQA, 2Wiki, MuSiQue, Bamboogle | 미확인 |
| [SAGE: Benchmarking and Improving Retrieval for Deep Research Agents](https://arxiv.org/abs/2602.05975) | 2026 / arXiv | 1,200개 질의, 20만 편 논문. 에이전트 환경에서 BM25가 LLM retriever보다 약 30% 우세 | 자체 | 에이전트 서브쿼리가 키워드 지향이라는 점을 원인으로 지목 |
| [Beyond Semantic Similarity: ... Direct Corpus Interaction](https://arxiv.org/abs/2605.05242) | 2026 / NeurIPS | 인덱스 없이 grep과 shell 도구로 코퍼스를 직접 탐색(DCI) | BRIGHT, BEIR, BrowseComp-Plus | 대규모 코퍼스 확장성(2차 출처) |

### B-3. 반복되는 오픈 문제, 암묵적 가정, 벤치마크 공백
1. **"유용성 = 단일 문서의 한계 기여"라는 가정.** REPLUG LSR, eRAG, ARL2, Utility annotation은 문서별로 독립적으로 점수를 매깁니다. 하지만 Power of Noise(방해/무작위 효과)와 UDCG(방해 항)는 유용성이 **집합 의존적**임을 보여 줍니다. 집합 단위 유용성으로 retriever를 학습하는 방법은 드뭅니다.
2. **유용성 레이블이 특정 생성기에 묶임(generator-specific).** utility는 특정 LLM 기준으로 정의됩니다(Utility judgments, GainRAG, DPA-RAG). 생성기가 바뀔 때 retriever를 다시 학습해야 하는지, 생성기에 무관한 utility가 존재하는지는 체계적으로 다뤄지지 않았습니다. UDCG가 6개 LLM에 걸친 지표 상관을 보인 것은 평가 차원에 그칩니다.
3. **에이전트 쿼리 분포 이동.** retriever는 사람이 쓴 자연 질의로 학습됩니다. 에이전트는 키워드형이나 CoT 단계형 서브쿼리를 냅니다(SAGE, RaDeR, BRIGHT-PRO). Search-R1 계열은 retriever를 고정하고, 2505.15117은 검색 엔진 선택이 학습 동역학을 좌우한다고 보고합니다. retriever와 정책을 함께 학습할 때의 안정성은 2026년 초기 단계입니다(Agentic-R, CoSearch, Critic-R).
4. **보상 해킹과 신용 할당.** 정답 정확도를 보상으로 retriever를 학습하면 LLM의 파라메트릭 지식으로 맞춘 경우도 양의 신호가 됩니다. 이 혼입을 분리한 연구는 찾지 못했습니다(관찰. Utility judgments 논문은 반사실 passage에 대한 LLM의 취약성을 지적).
5. **평가 공백.** 에이전트 루프 안의 retriever 평가는 BRIGHT-PRO와 SAGE에서 막 시작됐습니다. 다국어 utility 레이블 데이터셋은 NoMIRACL(UDCG가 사용) 외에 찾지 못했습니다.

### B-4. 덜 탐구된 연구 공백
1. **집합 단위(set-level) 유용성으로 dense retriever 학습: 방해 문서를 고려한 listwise 목적함수.**
   가장 가까운 작업은 UDCG(평가 지표만 다룸)와 Power of Noise(현상 관찰)입니다. Utility annotation(Disj-InfoNCE)은 문서별 positive 레이블을 씁니다. UDCG식 utility-distraction 점수를 학습 목표로 미분 가능하게 옮긴 retriever 학습은 확인하지 못했습니다.
2. **생성기 이전 가능성(generator transfer): 유용성 정렬된 retriever가 다른 LLM이나 크기로 옮겨 가는가.**
   가장 가까운 작업은 ARL2(전이 능력 주장)와 DPA-RAG(블랙박스/오픈소스 reader 모두에서 동작)입니다. 그러나 생성기를 체계적으로 교차하는 행렬 실험, 그리고 생성기 무관 utility와 생성기 특화 utility의 분해는 없습니다.
3. **파라메트릭 지식을 통제한 유용성 신호.** 정답 보상에서 "LLM이 원래 아는 것"을 빼는 반사실(closed-book 대비 이득) 보정입니다. GainRAG의 gain 개념이 가장 가깝지만, 구체적 정의는 본문을 열지 못해 **미확인**입니다. 에이전트 RL(Agentic-R, Critic-R)에서는 이런 통제를 확인하지 못했습니다.
4. **에이전트 서브쿼리 분포에 맞춘 retriever 적응과 hybrid 라우팅.**
   SAGE(BM25 우세, 코퍼스 키워드 증강), DCI(인덱스 없는 grep), BRIGHT-PRO(정적 순위와 에이전트 성능의 괴리)가 모두 "dense retriever와 에이전트 쿼리가 맞지 않는다"는 증거를 줍니다. 하지만 쿼리 형태(키워드형인지 추론형인지)에 따라 sparse, dense, DCI를 고르는 학습형 라우터나, 에이전트 궤적으로 retriever를 적응시키는 연구는 Critic-R와 Agentic-R 정도뿐이고 두 연구 모두 QA 멀티홉에 한정됩니다.
5. **축 A×B 교차: LLM 피드백 정렬이 표현력 한계(LIMIT)와 압축에 어떤 영향을 주는가.** utility 정렬이 단일 벡터의 조합적 표현 한계를 악화시키는지 완화시키는지, 그리고 정렬된 retriever가 MRL/양자화에서 더 취약한지 다룬 연구는 찾지 못했습니다.

---

## 참고: 검색 중 확인했지만 표에 넣지 않은 항목
- A Survey of Reasoning-Intensive Retrieval: Progress and Challenges, ACL 2026 ([arXiv 2605.00063](https://arxiv.org/abs/2605.00063))
- RaDeR: Reasoning-aware Dense Retrieval Models, EMNLP 2025 (Anthology 2025.emnlp-main.1011)
- DIVER (arXiv 2508.07995). 제목은 검색 결과 기준 "DIVER: A Multi-Stage Approach for Reasoning-intensive Information Retrieval"
- Matryoshka Representation Learning, NeurIPS 2022. 2D Matryoshka Sentence Embeddings (arXiv 2402.14776)
- IndicRAGSuite (arXiv 2506.01615)
- Qwen3-VL-Embedding (arXiv 2601.04720)
- CoSearch (arXiv 2604.17555). 제목만 확인했고 내용은 미확인
