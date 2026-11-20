# RAG 검색/인덱싱 측 문헌 조사: (A) 표현·임베딩, (B) Retriever–LLM 정렬

작성일: 2026-11-19 (본문 대조 교정: 2026-11-20)

## 검증 방법과 주의 (꼭 읽어 주세요)
- 처음 작성할 때는 서지를 WebSearch 결과로만 확인했고 PDF 본문은 열지 못했습니다. 이번 교정에서는 **표에 나온 논문 전부를 로컬 본문 텍스트(pdftotext)와 arXiv API 메타데이터(제목, 날짜, 저자가 밝힌 comment/journal_ref)에 대조했습니다.** ACL Anthology 논문은 Anthology 본문으로, Promptriever와 MUVERA는 ICLR/NeurIPS 프로시딩 본문으로 확인했습니다.
- venue는 본문 첫 쪽 헤더/각주 또는 arXiv comment로 확인했습니다. 둘 다 없으면 "venue 근거 없음"으로 적었습니다.
- 수치는 본문의 표나 문장에서 찾은 값입니다. 측정 조건(원 질의/재작성 질의 등)이 다르면 함께 적었습니다.
- "Stated limitations" 열은 Limitations, Conclusion, Future work 절에서 저자가 직접 밝힌 한계만 요약했습니다. 그런 절이 없으면 "명시된 한계 없음"으로 적었습니다. 우리 관찰은 "(관찰)"로 구분했습니다.
- 본문이 로컬에 없는 항목(RaDeR, Matryoshka Representation Learning)은 "본문 접근 불가"로 표시하고 다른 논문의 참고문헌으로만 확인했습니다.
- 링크는 arXiv abs 또는 공식 프로시딩/Anthology URL입니다.

---

## 축 A. 표현 / 임베딩

### A-1. 현황 요약
LLM 백본 단일 벡터 임베더가 사실상 표준이 되었습니다. 대표적으로 E5-Mistral, LLM2Vec, NV-Embed, Gemini Embedding, Qwen3-Embedding이 있습니다. 공통 레시피도 굳어졌습니다. 양방향 attention 또는 pooling을 바꾸고, LLM으로 합성 데이터를 만들고, 다단계 대조학습을 하는 방식입니다. 그 결과 MTEB/MMTEB 점수는 포화 단계에 들어섰고, MMTEB에서는 560M 규모 모델(multilingual-e5-large-instruct)이 공개 모델 최고였습니다.

움직임이 가장 큰 곳은 두 곳입니다. 첫째는 **추론 집약 검색(reasoning-intensive retrieval)**입니다. BRIGHT 공개 당시 MTEB 검색 부분집합(BEIR)에서 59.0이던 SFR-Embedding-Mistral이 BRIGHT에서는 nDCG@10 18.3에 그쳤습니다. 2025~2026년 전용 학습으로 이 격차가 빠르게 줄고 있습니다. ReasonIR-8B는 원 질의 24.4, GPT-4 재작성 질의 29.9를 기록했고, ReasonEmbed-8B는 원 질의 38.1을 기록했습니다(두 수치는 질의 조건이 달라 직접 비교할 수 없습니다). 둘째는 **표현력의 이론적 한계**입니다. LIMIT(Weller et al., ICLR 2026)은 단일 벡터가 반환할 수 있는 top-k 부분집합의 수가 임베딩 차원(sign-rank)에 묶인다고 주장했습니다. 2026년 반론(2603.29519)은 LIMIT이 k=2라서 5차원이면 충분하므로 차원만으로는 실패를 설명할 수 없다고 봅니다. 대신 domain shift와 "유사도 ≠ 과제상 관련성" 불일치를 주된 원인으로 들고, 단일 벡터와 multi-vector의 격차는 drowning 효과로 설명합니다. Late interaction(ColBERT 계열, MUVERA)과 learned sparse(SPLADE-v3)는 효율화 단계에 있어 성숙한 편입니다. 다만 LIMIT에서 BM25와 multi-vector가 단일 벡터보다 훨씬 강했고 에이전트 검색에서도 BM25가 강세를 보이면서(SAGE) 다시 주목받고 있습니다.

### A-2. 논문 표

| Title | Year/venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Improving Text Embeddings with Large Language Models](https://aclanthology.org/2024.acl-long.642/) (Wang et al.) | 2024 / ACL | Mistral-7B + LoRA(rank 16). 독점 LLM으로 93개 언어, 수십만 태스크의 합성 데이터를 만들고 1k step 미만으로 학습(E5-Mistral). | MTEB(합성만 63.1, +레이블 66.6), BEIR(56.9) | LLM이라 BERT형보다 추론 비용이 크고, 4096차원이라 저장 비용이 큼. 독점 LLM에서 좋은 합성 데이터를 얻으려고 수동 프롬프트 엔지니어링에 의존(Limitations 절) |
| [LLM2Vec: Large Language Models Are Secretly Powerful Text Encoders](https://arxiv.org/abs/2404.05961) | 2024 / COLM | 양방향 attention + MNTP + 비지도 SimCSE로 decoder LLM을 인코더로 전환 | 단어 수준 과제(chunking, NER, POS), MTEB(비지도 SOTA 56.8) | 모델 크기와 4096차원 때문에 인덱싱 비용이 큼, LLaMA-2/Mistral 사전학습 데이터의 MTEB 오염 가능성, 영어로만 평가(부록 A Limitations) |
| [NV-Embed: Improved Techniques for Training LLMs as Generalist Embedding Models](https://arxiv.org/abs/2405.17428) | 2025 / ICLR Spotlight | latent-attention pooling, causal mask 제거, 2단계 instruction 대조학습. 공개 데이터 + 합성 데이터(Mixtral-8x22B) + hard-negative mining. 압축(가지치기, 양자화, 증류) 분석 포함 | MTEB 56개 태스크(NV-Embed-v1 69.32, NV-Embed-v2 72.31), AIR-Bench | 명시된 한계 없음 |
| [M3-Embedding (BGE-M3)](https://aclanthology.org/2024.findings-acl.137/) | 2024 / Findings ACL | 한 모델에서 dense, sparse, multi-vector를 동시 지원. self-knowledge distillation, 100+개 언어, 8192 토큰 | MIRACL, MKQA, MLDR, NarrativeQA | 다양한 실제 데이터셋으로의 일반화는 추가 검증이 필요하고, 8192 토큰을 넘는 초장문서는 효율 문제가 있으며, 언어별 성능 편차는 충분히 분석하지 못함(Limitations 절) |
| [Gemini Embedding: Generalizable Embeddings from Gemini](https://arxiv.org/abs/2503.07891) | 2025 / arXiv(tech report) | Gemini LLM에서 초기화한 대조학습 임베더. Gemini로 학습 데이터 생성과 hard negative 선별 | MTEB(Multilingual) Borda 1위(task mean 68.32), MTEB(Eng v2), MTEB(Code), XOR-Retrieve SOTA, XTREME-UP | 별도 Limitations 절 없음. hard negative가 과하면 과적합된다고 보고(§6)했고, 멀티모달 확장은 향후 과제로 둠(§7). 폐쇄 모델이라 재현할 수 없음(관찰) |
| [Qwen3 Embedding](https://arxiv.org/abs/2506.05176) | 2025 / arXiv(tech report) | Qwen3 0.6B/4B/8B 임베딩과 리랭커. Qwen3로 약한 감독 합성 쌍 약 1.5억 개를 만들고 그중 약 1,200만 개를 고품질로 선별. Apache 2.0 | MMTEB multilingual 70.58, MTEB Code 80.68(8B) | 명시된 한계 없음 |
| [MMTEB: Massive Multilingual Text Embedding Benchmark](https://arxiv.org/abs/2502.13595) | 2025 / ICLR | 500개 이상 태스크, 250개 이상 언어. instruction following, 장문서, 코드 검색, 추론 과제(RAR-b, BRIGHT) 통합. 상관 기반 다운샘플링으로 비용 절감 | 자체(공개 모델 1위는 560M multilingual-e5-large-instruct) | 사람 번역을 허용해서 생기는 English leakage, 기여 점수 배분의 불완전함, 고자원 언어 편중(Limitations 절) |
| [SPLADE-v3: New baselines for SPLADE](https://arxiv.org/abs/2403.06789) | 2024 / arXiv(tech report) | SPLADE 학습 개선 baseline 세트(다중 hard negative, cross-encoder 앙상블 증류. Doc, Lexical 변형 포함) | MS MARCO(MRR@10 40.2), TREC DL19/20, BEIR-13(51.7), LoTTE 등 최대 44개 query set | 저자가 "정식 논문에 필요한 새로움은 부족하지만" 모델을 공개한다고 밝힘(§1). Lexical 변형은 BEIR에서 약하고 Doc 변형은 zero-shot에서 약함 |
| [Jina-ColBERT-v2](https://arxiv.org/abs/2408.16672) | 2024 / arXiv(EMNLP 워크숍 제출본, 채택 근거 없음) | 다국어 late interaction. MRL로 64~768차원 projection head 6개를 함께 학습해 저장 공간과 성능을 맞바꿀 수 있음(절감률 "50%"는 본문 확인 불가) | BEIR, LoTTE, MIRACL, mMARCO | 별도 Limitations 절 없음. ArguAna에서 ColBERTv2보다 크게 낮음, 다국어 학습의 curse of multilinguality 가능성을 결과 절에서 언급 |
| [MUVERA: Multi-Vector Retrieval via Fixed Dimensional Encodings](https://proceedings.neurips.cc/paper_files/paper/2024/hash/b71cfefae46909178603b5bc6c11d3ae-Abstract.html) | 2024 / NeurIPS | multi-vector(Chamfer) 유사도를 ε-근사하는 고정 차원 단일 벡터 인코딩(FDE). 이론적 보장 있음 | BEIR 6개 데이터셋(PLAID 대비 평균 recall +10%, latency −90%) | MS MARCO에서는 PLAID를 이기지 못했고, 문서당 임베딩 수가 FDE 품질에 주는 영향은 연구하지 않음(Broader Impacts and Limitations) |
| [SMEC: Rethinking Matryoshka Representation Learning for Retrieval Embedding Compression](https://arxiv.org/abs/2510.12474) | 2025 / EMNLP(arXiv 코멘트 기준) | 순차적 MRL(SMRL), 적응적 차원 선택(ADS), selectable cross-batch memory(S-XBM)로 LLM 임베딩 차원 압축 | BEIR(256차원에서 Matryoshka-Adaptor 대비 +1.1, Search-Adaptor 대비 +2.7), Products-10K, Fashion-200K | 특정 도메인 레이블 데이터로 소수 파라미터만 학습해서 범용성이 제한됨. 전체 파라미터 학습과 다양한 데이터 학습은 향후 과제(Limitation 절) |
| [BRIGHT: A Realistic and Challenging Benchmark for Reasoning-Intensive Retrieval](https://arxiv.org/abs/2407.12883) | 2025 / ICLR | 12개 데이터셋(StackExchange 7개, 코딩·수학 5개), 1,384개 질의. 추론이 필요한 관련성 | 자체(SFR-Embedding-Mistral 18.3. 이 모델의 MTEB 검색 부분집합 점수는 59.0) | 관련성 판단이 주관적임(다수 주석자 합의로 완화). 클러스터링 등 다른 임베딩 과제와 멀티모달은 다루지 않음(부록 G). 질의 수 1,398은 본문 확인 불가(이번 본문 버전은 1,384) |
| [FollowIR](https://aclanthology.org/2025.naacl-long.597/) | 2025 / NAACL | TREC narrative 기반 instruction 변경 쌍. pairwise p-MRR 평가, FollowIR-7B 공개. 기존 모델은 instruction을 키워드처럼만 쓰고 긴 instruction에 약하다고 진단 | FollowIR(Robust04, News21, Core17. FollowIR-7B 평균 p-MRR +12.2), InstructIR | 전체 코퍼스 검색이 아니라 reranking 설정으로만 평가함(라이선스 때문에 passage만 배포). TREC 판정과 새 주석에 오류가 있을 수 있음(§7 Limitations) |
| [Promptriever](https://proceedings.iclr.cc/paper_files/paper/2025/hash/2cefdb2c4c3274b78cd450bac35228df-Abstract-Conference.html) | 2025 / ICLR | MS MARCO에 인스턴스 단위 instruction 약 50만 개와 instruction negative(GPT-4o 생성)를 더한 bi-encoder | FollowIR(RepLLaMA 대비 +14.3 p-MRR), InstructIR(Robustness@10 +12.9), BEIR(프롬프트 선택 시 평균 +1.4), MS MARCO, TREC DL | in-context 예시 같은 다른 프롬프팅 기법은 탐색하지 않음, 어떤 프롬프트가 잘 되는지 이유가 불분명함, LLM 생성 데이터의 오류와 편향 가능성(§8 Limitations) |
| [ReasonIR: Training Retrievers for Reasoning Tasks](https://arxiv.org/abs/2504.20595) | 2025 / arXiv(venue 근거 없음) | BRIGHT 문서 풀을 seed로 삼아 문서별로 어려운 추론형 질의와 "그럴듯하지만 무용한" hard negative를 합성(ReasonIR-8B) | BRIGHT(원 질의 24.4, GPT-4 재작성 질의 29.9, 리랭커 포함 36.9), MMLU/GPQA RAG(closed-book 대비 +6.4%/+22.6%) | 별도 Limitations 절 없음. 향후 과제로 합성 데이터 스케일링, seed 문서 선택, 다국어·멀티모달 확장, 다중 턴 추론 결합을 제시(§7 Discussion) |
| [ReasonEmbed](https://arxiv.org/abs/2510.08252) | 2025 arXiv → 2026 / ACL | ReMixer(BRIGHT 코퍼스에서 합성 82K, 평가 positive 제외) + Redapter(추론 강도에 따른 샘플 가중). 질의 중복 분석으로 누출 없음 보고 | BRIGHT 원 질의 38.1(8B), 37.1(4B), R2MED(OOD) | 합성은 LLM 추론 능력에 묶이고, 추론형 검색에만 집중했으며, BRIGHT가 다루는 도메인에 한정됨(Limitations 절) |
| [On the Theoretical Limitations of Embedding-Based Retrieval](https://arxiv.org/abs/2508.21038) (Weller et al.) | 2025 arXiv → 2026 / ICLR | 반환 가능한 top-k 부분집합 수가 차원(sign-rank)에 묶인다고 주장. LIMIT 데이터셋 제시 | LIMIT(5만 문서 전체판에서 SOTA 임베더가 recall@100 20% 미만, BM25는 거의 완벽, GTE-ModernColBERT는 단일 벡터보다 훨씬 나음. 동의어판에서는 BM25가 89% 넘게 하락) | 이론은 단일 벡터에만 적용되고 multi-vector로 확장하지 않음, 일부 오류를 허용하는 설정의 경계는 제시하지 않음. multi-vector의 추론 과제 전이는 open question, sparse/hybrid는 future work(Limitations, §5.3) |
| [On Strengths and Limitations of Single-Vector Embeddings](https://arxiv.org/abs/2603.29519) | 2026 / arXiv | Alon et al.(2016)에 따르면 2k+1차원이면 충분(LIMIT은 5차원). 주된 실패 원인을 domain shift와 유사도–관련성 불일치로 보고, 단일 대 다중 벡터 격차는 drowning으로 설명 | LIMIT fine-tune(Recall@10 단일 약 1%→40%, multi 40%→98%). 단일 벡터 fine-tune 후 MS MARCO Recall@100이 40% 넘게 하락 | 명시된 한계 없음 |
| [Optimized Text Embedding Models and Benchmarks for Amharic Passage Retrieval](https://arxiv.org/abs/2505.19356) | 2025 / Findings ACL | 암하라어 검색 벤치마크와 전용 dense retriever, ColBERT 변형 | AMNEWS 기반 Amharic Passage Retrieval(Arctic Embed 2.0 대비 MRR@10 상대 +17.6%, Recall@10 +9.86%. ColBERT MRR@10 0.843), 2AIRTC | 사람 관련성 판정이 없어서 헤드라인 기반 약한 감독 노이즈가 있고 규모가 작음. 사전학습 코퍼스가 3억 토큰으로 작음(§8 Limitations) |

### A-3. 반복되는 오픈 문제, 암묵적 가정, 벤치마크 공백
1. **MTEB/BEIR 점수와 실제 난이도가 어긋남.** MTEB 고득점이 추론형 검색 성능을 보장하지 않습니다. 근거: BRIGHT(MTEB 검색 부분집합 59.0인 모델이 BRIGHT 18.3), MMTEB(소형 모델 1위), BRIGHT-PRO(2605.04018, "정적 순위가 에이전트 루프 성능으로 느슨하게만 이어진다").
2. **"관련성 = 의미 유사도"라는 암묵적 가정.** bi-encoder 대조학습은 쿼리-문서 쌍의 대칭적 유사도를 학습합니다. 반면 추론형 관련성은 비대칭적이고 다단계입니다. 근거: BRIGHT, ReasonIR(기존 학습 데이터가 짧은 사실형 질의에 편중), RaDeR(본문 접근 불가), DIVER(2508.07995. BRIGHT 전체 46.8, 원 질의 31.9). 2603.29519도 LIMIT 실패의 주원인 중 하나로 유사도와 과제상 관련성의 불일치를 듭니다.
3. **합성 데이터 의존과 평가 오염 위험.** E5-Mistral, Qwen3-Embedding, ReasonIR, ReasonEmbed가 모두 LLM 합성 데이터에 크게 기댑니다. 특히 ReasonIR과 ReasonEmbed는 **BRIGHT 자체의 문서 풀을 합성 seed로 씁니다.** 그래서 합성 데이터와 평가 벤치마크는 설계상 도메인과 코퍼스를 공유합니다. ReasonEmbed는 BRIGHT 평가 positive 문서를 빼고, BM25와 weighted Jaccard로 질의 문자열 중복을 분석해 누출이 없다고 보고했으며, OOD 평가로 R2MED를 씁니다. 그러나 문자열 중복 분석은 의미 수준 중복을 잡지 못하고, ReasonIR에서는 같은 점검을 찾지 못했습니다. BRIGHT 도메인과 코퍼스를 학습에서 완전히 뺀 held-out 평가도 찾지 못했습니다(관찰).
4. **표현력 한계의 원인 논쟁이 미결.** 차원 탓인지(Weller 2025), domain shift와 유사도–관련성 불일치, drowning 탓인지(2603.29519)가 갈립니다. 두 논문은 domain shift에 대해서도 충돌합니다. Weller는 LIMIT train으로 학습해도 거의 도움이 안 된다는 이유로 domain shift를 배제했고, 2603.29519는 fine-tune으로 크게 개선된다는 이유로 domain shift를 주요 요인으로 봅니다. multi-vector와 sparse의 이론적 경계는 공식적으로 제시되지 않았습니다(Weller 2025가 Limitations에서 명시).
5. **압축(MRL, 양자화)과 추론형 성능의 상호작용 미평가.** SMEC(BEIR와 이미지 검색), Jina-ColBERT-v2(BEIR, LoTTE, MIRACL, mMARCO), NV-Embed 압축 분석, Qwen3-VL-Embedding(MS MARCO, VL3-Syn)은 모두 사실형 또는 일반 벤치마크 기준입니다. Qwen3-VL-Embedding 보고서(2601.04720)는 int8은 손실이 무시할 만하지만 binary는 크게 손상되고, 차원이 낮을수록 손실이 커진다고 보고했습니다. 그러나 압축이 BRIGHT나 LIMIT형 조합 질의에서 성능을 비대칭적으로 무너뜨리는지는 다루지 않았습니다.
6. **다국어 추론형 검색 벤치마크가 드묾.** MMTEB는 언어 폭이 넓고 추론 과제(RAR-b, BRIGHT)도 포함하지만, 포함된 추론 과제는 영어 데이터셋입니다. BRIGHT와 BRIGHT-PRO는 영어 중심입니다. 저자원 언어 연구(암하라어, IndicRAGSuite 2506.01615)는 사실형 QA 수준입니다. 서베이(2605.00063)가 정리한 벤치마크 중 다국어를 다루는 것은 수학 동치 문제 검색인 MathNet-Retrieve 정도입니다(서베이 경유, 본문 미확인).

### A-4. 덜 탐구된 연구 공백
1. **추론 집약 검색에서 압축(MRL 절단, int8, binary)의 비대칭적 손실 측정과 압축 인지 학습.**
   가장 가까운 작업은 SMEC(2025)와 Qwen3-VL-Embedding 보고서입니다. 둘 다 사실형 또는 일반 검색 지표만 봅니다. ReasonEmbed와 ReasonIR은 full-precision 4B/8B 모델만 보고합니다. 추론 신호가 하위 차원이나 저비트에서 먼저 사라지는지는 아무도 측정하지 않았습니다. 배포 비용이 큰 8B 추론형 임베더에서 실용적 가치가 큰 질문입니다.
2. **LIMIT 논쟁을 실제 코퍼스로 옮기기: "조합적 관련성"을 측정하는 자연 벤치마크와 진단 도구.**
   가장 가까운 작업은 LIMIT(합성)과 2603.29519(LIMIT fine-tune)입니다. 실제 RAG 질의 중 몇 %가 sign-rank 또는 drowning 병목을 겪는지 정량화한 연구는 없습니다. 쿼리별로 "단일 벡터로 표현 불가" 여부를 판정해 multi-vector나 sparse로 라우팅하는 방법도 비어 있습니다. 서베이(2605.00063)도 adaptive routing과 multi-vector를 향후 방향으로만 제시합니다.
3. **다국어·저자원 추론 집약 검색.**
   가장 가까운 작업은 BRIGHT(영어)와 MMTEB(추론 과제는 영어)입니다. ReasonIR 저자는 다국어 확장을 향후 과제로 명시했지만, ReasonIR/ReasonEmbed식 합성 파이프라인이 저자원 언어로 옮겨지는지, 번역 노이즈 때문에 "그럴듯한 hard negative"가 무너지는지는 검증되지 않았습니다.
4. **추론형 late interaction과 learned sparse.**
   추론형 검색기는 거의 전부 단일 벡터 LLM 임베더입니다(ReasonIR, ReasonEmbed, RTriever). 서베이 2605.00063의 BRIGHT 표도 이들을 모두 "Single Vector"로 분류합니다. LIMIT 연구는 multi-vector의 우위를 보였지만, multi-vector를 추론·instruction 과제에 쓴 사례가 드물다고(Chaffin 2025b가 예외) 밝히고 이를 open question으로 남겼습니다. 추론형 합성 데이터로 학습한 ColBERT/SPLADE를 BRIGHT에서 체계적으로 비교한 연구는 찾지 못했습니다.
5. **합성 데이터와 벤치마크 오염 감사.** ReasonIR, ReasonEmbed, DIVER가 BRIGHT 점수를 빠르게 올렸습니다. 이 가운데 ReasonIR과 ReasonEmbed는 BRIGHT 코퍼스에서 직접 합성합니다. ReasonEmbed의 질의 문자열 중복 분석이 유일한 점검이고, 의미 수준 중복과 코퍼스 공유 효과를 분리한 감사는 찾지 못했습니다. BRIGHT 도메인을 학습에서 완전히 뺀 held-out 도메인 변형이 필요합니다.

---

## 축 B. Retriever–LLM 정렬

### B-1. 현황 요약
"LLM 출력으로 retriever를 감독한다"는 기본 패러다임은 2023~2024년에 확립됐습니다. REPLUG LSR은 블랙박스 LM likelihood 분포와 retriever 분포 사이의 KL을 최소화하고, RA-DIT은 LM과 retriever를 따로 튜닝하며, ARL2는 LLM이 관련성 레이블러 역할을 합니다. 2024~2025년에는 축이 두 방향으로 움직였습니다. 하나는 **"관련성 vs 유용성(utility)"을 분리하는 측정과 레이블**입니다(Utility judgments SIGIR'24, eRAG SIGIR'24, Power of Noise SIGIR'24, Utility-focused annotation EMNLP'25, UDCG(venue 근거 없음)). 다른 하나는 **선호/이득 신호 기반 정렬**입니다(DPA-RAG, GainRAG ACL'25). 2026년에는 **에이전트(R1식 검색 학습) 루프 안에서 retriever를 학습**하는 흐름이 나타났습니다(Agentic-R, Critic-R, CoSearch). BRIGHT-PRO의 RTriever는 오프라인 합성 데이터로 학습하고 에이전트 프로토콜로 평가하는 경우입니다. Search-R1 계열 대부분은 여전히 retriever를 고정된 환경으로 취급합니다. SAGE는 에이전트의 키워드형 서브쿼리 때문에 BM25가 LLM 기반 retriever보다 약 30% 앞선다는 증거를 냅니다. DCI는 원인을 고정 top-k 유사도 인터페이스의 해상도에서 찾습니다.

### B-2. 논문 표

| Title | Year/venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [REPLUG: Retrieval-Augmented Black-Box Language Models](https://aclanthology.org/2024.naacl-long.463/) | 2024 / NAACL(arXiv 2023) | 블랙박스 LM의 출력 점수로 retriever를 감독(LSR, KL 최소화) | Pile LM(GPT-3 +6.3%), MMLU(Codex 5-shot +5.1%), NQ, TriviaQA | 모델이 검색 지식과 파라메트릭 지식 중 무엇에 의존하는지 불명확함, 항상 검색하므로 방해 문서 위험과 비용이 있음, DB가 Wikipedia/Pile에 한정됨(§9 Limitations) |
| [RA-DIT: Retrieval-Augmented Dual Instruction Tuning](https://arxiv.org/abs/2310.01352) | 2024 / ICLR | LM은 검색 결과 활용을 학습하고, retriever는 LM이 선호한 결과로 학습(2단계 분리) | MMLU, NQ, TriviaQA, KILT 부분집합(REPLUG 대비 0-shot +8.9%, 5-shot +1.4%, MMLU·NQ·TQA·ELI5 평균) | 별도 Limitations 절 없음. 반복(multi-step) RA-DIT은 추가 이득이 없어 향후 과제로 둠 |
| [ARL2](https://aclanthology.org/2024.acl-long.203/) | 2024 / ACL | LLM(ChatGPT)이 근거 관련성을 레이블링하고 적응적 self-training으로 데이터를 큐레이션. 과제 간 전이와 zero-shot 일반화 보고 | NQ(+5.4%), TriviaQA, MMLU(+4.6%) | ChatGPT API 주석 비용이 큼, 생의학 등 특정 도메인으로 확장하고 평가해야 함(Limitations 절) |
| [Are Large Language Models Good at Utility Judgments?](https://arxiv.org/abs/2403.19216) | 2024 / SIGIR | LLM 5개로 관련성과 유용성 구분 능력을 평가. 반사실 passage를 쉽게 받아들임을 발견. k-sampling listwise 방법 제안 | NQ, HotpotQA, MSMARCO-QA | 작은 후보 집합만 평가해서 대규모 후보로 확장이 필요하고, zero-shot만 탐색함(§7 결론). LLM 답에 기반한 utility는 LLM 내부 지식의 영향을 받는다고 지적 |
| [Evaluating Retrieval Quality in Retrieval-Augmented Generation (eRAG)](https://arxiv.org/abs/2404.13781) | 2024 / SIGIR('short' 표기는 본문 확인 불가) | 문서 하나만 넣었을 때의 LLM 성능을 그 문서의 관련성 레이블로 사용 | NQ, TriviaQA, HotpotQA, FEVER, WoW(KILT. Kendall τ +0.168~0.494, GPU 메모리 최대 50배 절감) | 별도 Limitations 절 없음. 검색 문서 수가 늘면 상관이 떨어지는데, 지표가 문서별 레이블을 독립적으로 쓰는 반면 LLM은 문서 전체를 쓰기 때문이라고 저자가 설명(§3) |
| [The Power of Noise: Redefining Retrieval for RAG Systems](https://arxiv.org/abs/2401.14887) | 2024 / SIGIR | distracting 문서 하나로 정확도 최대 25% 하락. random 문서를 채우면 최대 35% 향상(검색 문서 4개일 때) | NQ-open(random 문서는 Wikipedia, Reddit, 무작위 단어) | 정확 일치 기반 정답 판정의 한계를 인정하고, random 이득의 메커니즘은 미해명이라고 밝힘. random 이득은 검색 문서 수와 무관하게 거의 항상 나타났고, 최적은 최소한의 문서를 검색한 뒤 random을 채울 때 |
| [Understand What LLM Needs: Dual Preference Alignment for RAG (DPA-RAG)](https://arxiv.org/abs/2406.18676) | arXiv 2024(venue 근거 없음, comment "Work in progress") | reranker 외부 정렬(pair/point/contrastive)과 LLM 내부 사전정렬. 5가지 쿼리 증강. 블랙박스와 오픈소스 reader 모두에 적용 | NQ, TriviaQA, HotpotQA, WebQSP | 명시된 한계 없음(선호 데이터 부족은 증강의 동기로 언급) |
| [GainRAG](https://aclanthology.org/2025.acl-long.527/) | 2025 / ACL | passage의 "이득(gain)"을 contrastive decoding을 적용한 perplexity로 정의해 LLM 내부 지식 편향을 뺌. 소량 데이터로 BGE 기반 selector(middleware)를 학습해 retriever와 LLM 선호를 정렬. pseudo-passage 전략 | HotpotQA, 2WikiMultiHopQA, WebQuestions, SQuAD, NQ, TriviaQA | passage 단위 선택이 최적이 아닐 수 있고 passage 조합의 gain은 미검증. 학습 데이터가 매우 적음. 소형 모델로 신호를 생성할 수 있는지 미검증(Limitations 절) |
| [Utility-Focused LLM Annotation for Retrieval and RAG](https://aclanthology.org/2025.emnlp-main.88/) | 2025 / EMNLP | Qwen2.5-32B로 MS MARCO 약 50만 질의에 listwise utility 레이블(RelSel/UtilSel/UtilRank) 생성. 다중 positive용 SumMargLH 손실, 인간 레이블과의 curriculum learning | MS MARCO, BEIR, MS MARCO QA, NQ, HotpotQA | 주석 후보 풀이 인간 positive와 다른 모델의 hard negative로 구성돼 현실과 다르고, 더 강한 LLM은 쓰지 않았으며, MS MARCO만 주석함(§8). 발견: LLM 레이블은 in-domain에서 인간 레이블보다 낮고 OOD에서 높으며, 인간 레이블 20%를 섞으면 동등 |
| [Redefining Retrieval Evaluation in the Era of LLMs (UDCG)](https://arxiv.org/abs/2510.21440) | arXiv 2025(venue 근거 없음) | 위치 할인과 relevance–utility 불일치를 지적. passage별 utility(비기권 확률, 방해 passage는 음수)를 따로 구한 뒤 γ로 가중해 선형 결합한 지표 | NQ, TriviaQA, PopQA, BioASQ, NoMIRACL(영어 부분), 6개 LLM(상관 최대 +36%) | QA에만 실험해서 멀티홉과 사실검증은 미검증, logit 접근이 필요해서 블랙박스 API에는 못 씀, 영어 데이터만 평가(Limitations 절). γ=1/3은 여러 설정에서 잘 된다고 보고(한계로 쓰지 않음) |
| [Search-R1](https://arxiv.org/abs/2503.09516) | 2025 / COLM | RL로 LLM이 검색을 교차 호출하도록 학습. retriever(E5)는 고정 환경 | 7개 QA(NQ, TriviaQA, PopQA, HotpotQA, 2Wiki, MuSiQue, Bamboogle. Qwen2.5-7B +24%, 3B +20%) | 별도 Limitations 절 없음. 향후 과제: 보상 설계, 불확실성 기반 검색 조절, 다양한 도구와 멀티모달 |
| [An Empirical Study on RL for Reasoning-Search Interleaved LLM Agents](https://arxiv.org/abs/2505.15117) | 2025 / arXiv | format 보상은 효과가 있지만 중간 retrieval 보상의 효과는 제한적임. LLM 규모와 초기화, 검색 엔진 선택이 RL 학습 동역학과 추론 시 견고성을 크게 좌우 | Search-R1과 같은 7개 QA | 검색 에이전트 시나리오에 한정되어 데이터 과학, SWE, 다중 도구 에이전트로 일반화되지 않음(부록 A) |
| [Agentic-R: Learning to Retrieve for Agentic Search](https://arxiv.org/abs/2601.11888) | 2026 / arXiv | 로컬 관련성과 글로벌 정답 정확도(passage를 넣고 이어서 생성한 답의 EM)로 passage에 점수. 에이전트와 retriever를 번갈아 반복 학습 | 7개 단일/멀티홉 QA | 단일/멀티홉 QA만 평가했고 GPQA 같은 전문가 추론은 평가하지 않음, 중간 크기 에이전트와 retriever만 실험(Limitations 절) |
| [Rethinking Reasoning-Intensive Retrieval (BRIGHT-PRO, RTriever)](https://arxiv.org/abs/2605.04018) | 2026 / ACL | BRIGHT StackExchange 질의에 다측면 gold evidence를 붙이고 정적/에이전트 이중 평가. RTriever-4B(Qwen3-Emb-4B LoRA, 오프라인 합성 RTriever-Synth로 학습) | BRIGHT-PRO(발견: 정적 순위가 에이전트 성능으로 느슨하게만 이어지고, BM25는 에이전트 루프에서 크게 오름) | StackExchange 7개 도메인만 다루고, 전문가 주석 비용 때문에 예시 수가 적으며, 학습은 1-positive-1-negative 부분집합만 사용(Limitations and Future Work) |
| [Critic-R](https://arxiv.org/abs/2606.00590) | 2026 / arXiv(venue 근거 없음) | critic이 근거 충분성을 판단해 쿼리와 instruction을 재작성(Critic-R-Zero). 성공·실패 궤적으로 Critic-Embed 학습 | HotpotQA, 2Wiki, MuSiQue, Bamboogle | 추론 에이전트의 자기성찰 trace 품질에 의존해서 약한 모델에서는 신호가 나빠질 수 있음. 정적 Wikipedia 코퍼스만 평가했고 실시간 웹이나 기업 문서는 평가하지 않음(Limitations 절) |
| [SAGE: Benchmarking and Improving Retrieval for Deep Research Agents](https://arxiv.org/abs/2602.05975) | 2026 / arXiv | 4개 과학 도메인 1,200개 질의, 20만 편 논문. DR Tulu 백본에서 BM25가 LLM 기반 retriever(ReasonIR, gte-Qwen2-7B)보다 약 30% 우세하고, 원인을 키워드 지향 서브쿼리로 분석. 코퍼스 수준 메타데이터·키워드 증강(short-form +8%, open-ended +2%) | 자체 | 에이전트를 retriever 유형에 맞춰 학습하는 실험은 하지 않았고, 행동 분석이 DR Tulu에 치우쳐 일반화가 제한됨(Limitations and Future Work) |
| [Beyond Semantic Similarity: ... Direct Corpus Interaction](https://arxiv.org/abs/2605.05242) | 2026 / arXiv(venue 근거 없음) | 인덱스 없이 grep과 shell 도구로 코퍼스를 직접 탐색(DCI). 원인을 고정 유사도 인터페이스의 해상도로 봄 | BRIGHT 4개, BEIR 2개(ArguAna, SciFact), BrowseComp-Plus, 6개 QA | 별도 Limitations 절 없음. 코퍼스를 100K→200K로 늘리면 정확도 −13.6pt, 도구 호출 38.5→86.9. 400K에서는 정확도 37.5%(RQ4) |

### B-3. 반복되는 오픈 문제, 암묵적 가정, 벤치마크 공백
1. **"유용성 = 단일 문서의 한계 기여"라는 가정.** REPLUG LSR, eRAG, ARL2, UDCG는 문서별 유용성을 따로 구한 뒤 합치거나 비교합니다. UDCG도 passage별 utility를 독립적으로 구해 선형 결합하므로 문서 간 상호작용은 모델링하지 않습니다. Utility annotation은 주석을 listwise로 하지만 학습 레이블은 문서별 positive입니다. 반면 Power of Noise(방해/무작위 문서 효과)와 eRAG(문서 수가 늘면 문서별 레이블과 성능의 상관이 떨어짐)는 유용성이 **집합 의존적**임을 보여 줍니다. GainRAG도 passage 조합의 gain을 한계로 남겼습니다. 집합 단위 유용성으로 retriever를 학습하는 방법은 드뭅니다.
2. **유용성 레이블이 특정 생성기에 묶임(generator-specific).** utility는 특정 LLM 기준으로 정의됩니다(Utility judgments, GainRAG, DPA-RAG). 생성기가 바뀔 때 retriever를 다시 학습해야 하는지, 생성기에 무관한 utility가 존재하는지는 체계적으로 다뤄지지 않았습니다. UDCG가 6개 LLM에 걸친 지표 상관을 보인 것은 평가 차원에 그치고, logit 접근이 필요해 블랙박스 생성기에는 적용하지 못합니다.
3. **에이전트 쿼리 분포 이동.** retriever는 사람이 쓴 자연 질의로 학습됩니다. 에이전트는 키워드형 서브쿼리를 냅니다(SAGE. BRIGHT-PRO도 LLM 후속 질의의 구체적 키워드가 BM25의 어휘 불일치를 메운다고 관찰. RaDeR 관련 서술은 본문 접근 불가). Search-R1 계열은 retriever를 고정하고, 2505.15117은 검색 엔진 선택이 학습 동역학을 좌우한다고 보고합니다. CoSearch는 고정 retrieval과 oracle의 격차가 7개 QA에서 최대 +26.8% 상대 F1이라고 보고합니다. retriever와 정책을 함께 학습할 때의 안정성은 2026년 초기 단계입니다(Agentic-R, CoSearch, Critic-R).
4. **보상 해킹과 신용 할당.** 정답 정확도를 보상으로 retriever를 학습하면 LLM의 파라메트릭 지식으로 맞춘 경우도 양의 신호가 됩니다. 이 문제는 여러 논문이 인정합니다. REPLUG는 Limitations에서 검색 지식과 파라메트릭 지식 중 무엇에 의존하는지 불명확하다고 썼고, Utility judgments 논문은 LLM 답 기반 utility가 내부 지식의 영향을 받는다고 지적했습니다. 비에이전트 설정에서는 GainRAG가 contrastive decoding으로 내부 지식 편향을 빼는 gain을 정의했습니다. 그러나 에이전트 RL에서 retriever를 학습하는 연구(Agentic-R은 EM 기반 글로벌 정답 정확도, CoSearch는 궤적 결과 보상)에서는 이런 보정을 찾지 못했습니다(관찰).
5. **평가 공백.** 에이전트 루프 안의 retriever 평가는 BRIGHT-PRO와 SAGE에서 막 시작됐습니다. 다국어 utility 레이블 데이터셋은 찾지 못했습니다. UDCG는 NoMIRACL의 영어 부분만 사용했고, 다국어·교차언어 설정을 한계로 명시했습니다.

### B-4. 덜 탐구된 연구 공백
1. **집합 단위(set-level) 유용성으로 dense retriever 학습: 방해 문서를 고려한 listwise 목적함수.**
   가장 가까운 작업은 UDCG(평가 지표만 다루고, 그마저 passage별 utility의 가산 결합)와 Power of Noise(현상 관찰)입니다. Utility annotation(SumMargLH)은 문서별 다중 positive 레이블을 씁니다. GainRAG는 passage 조합의 gain을 미검증 한계로 남겼습니다. 집합 상호작용을 반영한 utility-distraction 점수를 미분 가능한 학습 목표로 옮긴 retriever 학습은 확인하지 못했습니다.
2. **생성기 이전 가능성(generator transfer): 유용성 정렬된 retriever가 다른 LLM이나 크기로 옮겨 가는가.**
   가장 가까운 작업은 DPA-RAG(블랙박스와 오픈소스 reader 모두에서 동작)입니다. ARL2가 보고한 전이는 과제·데이터셋 전이(NQ→TQA, MMLU few-shot)이고 레이블러는 ChatGPT 하나여서 생성기 전이 근거가 되지 않습니다. 생성기를 체계적으로 교차하는 행렬 실험, 그리고 생성기 무관 utility와 생성기 특화 utility의 분해는 없습니다.
3. **에이전트 RL에서 파라메트릭 지식을 통제한 유용성 신호.** 정답 보상에서 "LLM이 원래 아는 것"을 빼는 반사실(closed-book 대비 이득) 보정입니다. 단일 턴 RAG에서는 GainRAG가 contrastive perplexity로 이를 이미 다룹니다. 남은 공백은 이 보정을 다중 턴 에이전트 RL로 옮기는 것입니다. Agentic-R(EM 기반 글로벌 정답 정확도), Critic-R, CoSearch에서는 이런 통제를 확인하지 못했습니다.
4. **에이전트 서브쿼리 분포에 맞춘 retriever 적응과 hybrid 라우팅.**
   SAGE(BM25 우세, 코퍼스 키워드 증강), DCI(인덱스 없는 grep, 원인은 인터페이스 해상도), BRIGHT-PRO(정적 순위와 에이전트 성능의 괴리, 루프 안 BM25 상승)가 모두 "고정된 dense retriever는 에이전트 검색에 잘 맞지 않는다"는 증거를 줍니다. SAGE는 retriever 유형에 맞춘 에이전트 학습을 향후 과제로 명시했습니다. 하지만 쿼리 형태(키워드형인지 추론형인지)에 따라 sparse, dense, DCI를 고르는 학습형 라우터는 찾지 못했습니다. 에이전트 궤적으로 retriever를 적응시키는 연구는 Agentic-R, Critic-R, CoSearch 정도이고, 모두 Wikipedia 기반 단일/멀티홉 QA에 한정됩니다.
5. **축 A×B 교차: LLM 피드백 정렬이 표현력 한계(LIMIT)와 압축에 어떤 영향을 주는가.** utility 정렬이 단일 벡터의 조합적 표현 한계를 악화시키는지 완화시키는지, 그리고 정렬된 retriever가 MRL/양자화에서 더 취약한지 다룬 연구는 찾지 못했습니다. 2603.29519가 보고한 단일 벡터 fine-tune의 catastrophic forgetting(MS MARCO Recall@100 40% 넘게 하락)은 좁은 정렬 신호로 미세조정할 때의 위험을 시사합니다(관찰).

---

## 참고: 검색 중 확인했지만 표에 넣지 않은 항목
- A Survey of Reasoning-Intensive Retrieval: Progress and Challenges, ACL 2026 ([arXiv 2605.00063](https://arxiv.org/abs/2605.00063)). BRIGHT 표에서 ReasonEmbed, DIVER-Retriever, RaDeR, ReasonIR을 모두 Single Vector로 분류
- RaDeR: Reasoning-aware Dense Retrieval Models, EMNLP 2025 (Anthology 2025.emnlp-main.1011). 본문 접근 불가. venue와 쪽수(pp. 19981–20008)는 서베이 2605.00063 참고문헌으로 확인했고, Anthology ID는 본문 확인 불가
- DIVER: A Multi-Stage Approach for Reasoning-intensive Information Retrieval (arXiv 2508.07995). BRIGHT nDCG@10 전체 46.8, 원 질의 31.9(파이프라인 전체). 서베이 표의 retriever 단독(4B)은 28.9
- Matryoshka Representation Learning, NeurIPS 2022(본문 접근 불가. Jina-ColBERT-v2 참고문헌으로 서지 확인). 2D Matryoshka Sentence Embeddings (arXiv 2402.14776)
- IndicRAGSuite: Large-Scale Datasets and a Benchmark for Indian Language RAG Systems (arXiv 2506.01615). 19개 인도 언어, Wikipedia와 MS MARCO 기반
- Qwen3-VL-Embedding and Qwen3-VL-Reranker (arXiv 2601.04720). MS MARCO와 VL3-Syn에서 int8은 손실이 무시할 만하고 binary는 크게 손상
- CoSearch: Joint Training of Reasoning and Document Ranking via Reinforcement Learning for Agentic Search (arXiv 2604.17555). 추론 에이전트와 생성형 document ranker를 GRPO로 공동 학습, 7개 QA
