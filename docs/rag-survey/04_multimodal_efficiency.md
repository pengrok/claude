# Survey D: RAG 검색/인덱싱 측 — (A) 멀티모달·시각 문서 검색, (B) 효율·스케일·동적 인덱스

작성일: 2026-11-19

**검증 방법:** 이 세션에서 WebFetch는 arxiv.org에 접근할 수 없었습니다(DNS 차단). 그래서 모든 논문은 WebSearch로 확인했습니다. 확인한 것은 arXiv abs/HTML 페이지, ACL Anthology, NeurIPS/ICLR proceedings에 노출된 제목·저자·날짜·초록 단편입니다. 본문 표의 수치는 초록이나 검색 요약에 나온 것만 적었습니다. 확인하지 못한 항목은 **미확인**으로 표시했습니다. "Stated limitations" 칸은 초록·요약에서 확인된 경우에만 적고, 나머지는 "미확인" 또는 "(관찰)"로 구분했습니다. "(관찰)"은 저자의 진술이 아니라 필자의 해석입니다.

---

## 축 A. 멀티모달 / 시각 문서 검색 (Visual Document Retrieval, VDR)

### A-1. 분야 현황 요약
- **성숙한 부분: 파싱 없는 검색**
  - ColPali(ICLR 2025)와 DSE(2024) 이후, 페이지 스크린샷을 VLM으로 직접 임베딩하는 방식이 OCR/파싱 파이프라인을 대체하는 사실상 표준이 되었습니다.
  - 표현 방식은 두 계열입니다. late-interaction 멀티벡터(ColPali/ColQwen/ColNomic/Jina-v4)와 단일벡터(DSE)입니다.
  - 모델 소형화(ModernVBERT, 250M)도 빠르게 진행되었습니다.
- **벤치마크 세대교체**
  - ViDoRe v1은 nDCG@5 90% 이상으로 포화되었습니다.
  - 그 뒤 v2(2025.05, 블라인드·교차문서·다국어 질의)와 v3(2026.01, ACL 2026, end-to-end RAG, 26k 페이지, 6개 언어)가 나왔습니다.
  - MMDocIR(EMNLP 2025, 페이지/레이아웃 단위), VisDoM(NAACL 2025, 다문서·차트·표·슬라이드), ViDoSeek 등도 등장했습니다.
- **지금 가장 활발한 축: 멀티벡터 시각 인덱스의 저장비용**
  - 머지 계열: Light-ColPali(97% 메모리 절감)
  - 문서 적응형 프루닝: DocPruner(50–60%)
  - 양자화+프루닝: HPC-ColPali(32×)
  - Prune-then-Merge(ACL Findings 2026)
- **아직 초기 단계: 비디오·오디오 RAG의 검색 측**
  - 비디오: VideoRAG(2종), CFVBench, WikiVideo/MIRAGE
  - 오디오: WavRAG(ACL 2025)
  - 대부분 소규모 코퍼스에서 정확도만 다루고, 인덱스 규모·비용·갱신은 거의 다루지 않습니다.

### A-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [ColPali: Efficient Document Retrieval with Vision Language Models](https://arxiv.org/abs/2407.01449) | 2024 arXiv / ICLR 2025 | 페이지 이미지를 VLM(PaliGemma)로 패치 단위 멀티벡터로 임베딩하고 late interaction으로 매칭. 파싱 파이프라인 대비 단순·고성능. ViDoRe 벤치마크 공개 | ViDoRe v1 | 미확인. (관찰) 페이지당 수백~천 개 벡터로 저장비용이 크며, 후속 압축 연구의 출발점 |
| [Unifying Multimodal Retrieval via Document Screenshot Embedding (DSE)](https://arxiv.org/abs/2406.11251) | 2024 arXiv (v2 2024.12) | 스크린샷을 단일 dense 벡터로 임베딩. Wiki-SS 컬렉션 공개 | Wiki-SS(NQ), 슬라이드 검색 | 미확인. (관찰) 단일벡터라 저장은 싸지만 세밀한 매칭에 한계 가능 |
| [VisRAG: Vision-based Retrieval-augmented Generation on Multi-modality Documents](https://arxiv.org/abs/2410.10594) | 2024 arXiv / ICLR 2025 | VLM 기반 검색기(VisRAG-Ret)와 생성기(VisRAG-Gen). 텍스트 RAG 대비 end-to-end 20–40% 향상 | 자체 구성 다중 데이터셋 | 단일 이미지만 받는 VLM을 위해 페이지 연결·가중 선택이 필요함 |
| [M3DocRAG: Multi-modal Retrieval is What You Need for Multi-page Multi-document Understanding](https://arxiv.org/abs/2411.04952) | 2024.11 arXiv | ColPali 검색과 Qwen2-VL 생성을 결합한 open-domain 다문서 DocVQA. M3DocVQA(3,368 PDF, 2,441 질문) 공개 | M3DocVQA, MP-DocVQA, MMLongBench-Doc | 미확인 |
| [ViDoRe Benchmark V2: Raising the Bar for Visual Retrieval](https://arxiv.org/abs/2505.17166) | 2025.05 arXiv | v1 포화(>90 nDCG@5)에 대응. 블라인드 맥락 질의, 장문·교차문서 질의, 합성+HITL 생성 | 4개 다국어 데이터셋 | v1과 순위 상관이 높음(변별력 증가는 절대값 중심). 4개 데이터셋 규모 |
| [ViDoRe V3: A Comprehensive Evaluation of RAG in Complex Real-World Scenarios](https://arxiv.org/abs/2601.08620) | 2026.01 arXiv / ACL 2026 | 10개 전문 도메인, 약 26k 페이지, 3,099 질의 × 6개 언어. 검색과 생성을 함께 평가 | ViDoRe v3 | 모델들이 비텍스트 요소, 열린 질의, 세밀한 시각 grounding에서 고전함. 어노테이션 시간 수치가 블로그(9k h)와 논문(12k h) 간 불일치 |
| [MMDocIR: Benchmarking Multimodal Retrieval for Long Documents](https://arxiv.org/abs/2501.08828) | 2025.01 arXiv / EMNLP 2025 | 장문서 페이지 단위와 레이아웃(표·차트·문단) 단위 검색. 313 문서, 1,658 질의 | MMDocIR(MMLongBench-Doc, DocBench 기반) | 미확인. 훈련셋 크기 표기가 판본마다 다름(73.8k vs 173.8k) |
| [VisDoM: Multi-Document QA with Visually Rich Elements Using Multimodal RAG](https://aclanthology.org/2025.naacl-long.310) | 2024.12 arXiv / NAACL 2025 | 표·차트·슬라이드 다문서 QA 벤치마크(2,271 질의, 1,277 문서). 시각 검색기가 텍스트 검색기를 능가함 | VisDoMBench(PaperTab, FetaTab, SciGraphQA, SPIQA, SlideVQA 통합) | 미확인 |
| [ViDoRAG: Visual Document RAG via Dynamic Iterative Reasoning Agents](https://aclanthology.org/2025.emnlp-main.464) | 2025.02 arXiv / EMNLP 2025 | GMM 기반 시각·텍스트 하이브리드 검색과 다중 에이전트 반복 추론. ViDoSeek 공개 | ViDoSeek, SlideVQA | 미확인. ViDoSeek 규모 기술이 출처마다 다름 |
| [jina-embeddings-v4: Universal Embeddings for Multimodal Multilingual Retrieval](https://arxiv.org/abs/2506.18902) | 2025.06 arXiv | 3.8B, 단일벡터와 멀티벡터(late interaction)를 동시 지원. 태스크별 LoRA. Jina-VDR 벤치마크 | Jina-VDR, ViDoRe | 미확인 |
| [ModernVBERT: Towards Smaller Visual Document Retrievers](https://arxiv.org/abs/2510.01149) | 2025.10 arXiv | 250M 인코더 ColModernVBERT. ColPali(약 3B)와 비슷한 ViDoRe 성능에 지연은 약 20ms vs 175ms | ViDoRe v1/v2 | 영어 중심, 소형 모델에 한정(다국어·대형은 미탐색) |
| [Towards Storage-Efficient Visual Document Retrieval: An Empirical Study on Reducing Patch-Level Embeddings (Light-ColPali)](https://arxiv.org/abs/2506.04997) | 2025.06 arXiv | 프루닝 vs 머징 실증. 랜덤 프루닝이 정교한 프루닝보다 나음. 후단 semantic clustering 머지와 파인튜닝으로 메모리를 97% 이상 줄이면서 성능 94% 이상 유지 | ViDoRe 등 | 프루닝은 여전히 불만족스러움. 극단 압축에는 재학습 필수 |
| [DocPruner: A Storage-Efficient Framework for Multi-Vector VDR via Adaptive Patch-Level Embedding Pruning](https://arxiv.org/abs/2509.23883) | 2025.09 arXiv | 문서 내 패치 어텐션 분포로 문서별 임계값을 정하는 질의 독립(오프라인) 프루닝. 저장 50–60% 절감 | ViDoRe, 다국어(de/ru/zh/ja) | 미확인 |
| [Hierarchical Patch Compression for ColPali (HPC-ColPali)](https://arxiv.org/abs/2506.21601) | 2025.06 arXiv | K-Means 양자화, 어텐션 기반 질의 시점 프루닝, 이진 인코딩. 최대 32× 압축(nDCG@10 하락 2% 미만), HNSW에서 지연 30–50% 감소 | ViDoRe, SEC-Filings | 미확인(단일 저자, 소규모 평가) |
| [Sculpting the Vector Space: Towards Efficient Multi-Vector VDR via Prune-then-Merge](https://arxiv.org/abs/2602.19549) | 2026.02 arXiv / Findings ACL 2026 | 저정보 패치를 먼저 프루닝하고 계층적으로 머지. near-lossless 압축 범위를 확장 | 29개 VDR 데이터셋. ColQwen2.5, ColNomic, Jina-v4 | 미확인 |
| [VideoRAG: Retrieval-Augmented Generation over Video Corpus](https://arxiv.org/abs/2501.05874) | 2025.01 arXiv (Jeong et al.) | 비디오 코퍼스에서 비디오를 검색하고 LVLM으로 생성. 프레임 선택 모듈 | WikiHowQA × HowTo100M | 미확인 |
| [VideoRAG: Retrieval-Augmented Generation with Extreme Long-Context Videos](https://arxiv.org/abs/2502.01549) | 2025.02 arXiv (HKU) | 극장문 비디오용 RAG(그래프 기반 지식과 멀티모달 컨텍스트) | LongerVideos(약 160편; 시간 수치는 출처마다 다름) | 미확인 |
| [CFVBench: A Comprehensive Video Benchmark for Fine-grained Multimodal RAG](https://arxiv.org/abs/2510.09266) | 2025.10 arXiv | 599개 비디오, 5,360 QA. 검색(Recall@K)과 생성을 2단계로 평가 | CFVBench | 미확인 |
| [WavRAG: Audio-Integrated RAG for Spoken Dialogue Models](https://aclanthology.org/2025.acl-long.613) | 2025.02 arXiv / ACL 2025 | ASR 없이 원시 오디오를 임베딩·검색하는 텍스트+오디오 혼합 KB. ASR+텍스트 RAG와 동등한 검색 성능에 약 10× 속도 | 자체 구성 | ASR 오류와 정보 손실 문제를 제기(저자 동기). 그 외 미확인 |

### A-3. 반복되는 미해결 문제 / 암묵적 가정 / 벤치마크 공백
1. **"코퍼스는 작고 정적이다"라는 가정**
   - VDR 벤치마크 코퍼스는 대부분 수백~수만 페이지 규모입니다. ViDoRe v3는 약 26k, MMDocIR은 313문서, M3DocVQA는 3,368 PDF입니다.
   - 수백만 페이지 규모에서 멀티벡터 VDR의 recall·지연·저장을 함께 측정한 공개 벤치마크는 찾지 못했습니다.
   - 근거: ViDoRe v2/v3, MMDocIR, M3DocRAG.
2. **압축 연구가 "정확도 대비 바이트"만 본다는 점**
   - Light-ColPali, DocPruner, HPC-ColPali, Prune-then-Merge는 nDCG 대비 저장률을 보고합니다.
   - ANN 인덱스(HNSW/IVF) 위에서의 후보 생성 recall, 인덱스 갱신 비용, 문서 추가 시 재클러스터링 비용은 거의 다루지 않습니다. HPC-ColPali만 HNSW 지연을 일부 보고합니다.
3. **압축이 모델에 종속된다는 점**
   - 극단 압축은 재학습이 필요합니다(Light-ColPali).
   - 압축 결과는 특정 백본(ColQwen2.5 등)에 묶이므로, 백본이 바뀌면 압축 인덱스 전체를 재생성해야 합니다. 이는 축 B의 모델 업그레이드 문제와 연결됩니다.
4. **페이지 단위 granularity 가정**
   - 대부분 페이지 단위 검색입니다. 레이아웃·요소 단위(MMDocIR)나 교차 페이지·교차 문서 근거(ViDoRe v2/v3, VisDoM)는 여전히 성능이 낮습니다.
   - ViDoRe v3는 "fine-grained visual grounding" 약점을 명시합니다.
5. **비텍스트 요소(차트·표)의 약점**
   - ViDoRe v3는 비텍스트 요소에서 성능이 떨어진다고 보고합니다.
   - 차트 수치 질의처럼 수치 추론이 필요한 질의를 검색 단계에서 처리하는 연구는 적습니다(VisDoM, MMDocIR).
6. **비디오·오디오 검색에서 코퍼스 규모와 인덱스 설계 공백**
   - 비디오 RAG 논문들은 프레임 선택과 생성 품질에 집중합니다(VideoRAG ×2, CFVBench).
   - 시간축 멀티벡터(프레임/클립 × 패치)의 저장 폭증을 다룬 연구는 찾지 못했습니다(미확인).
7. **다국어 편향**
   - ModernVBERT는 영어 한정을 명시합니다. ViDoRe v3와 DocPruner는 다국어 평가를 포함하지만 저자원 언어는 미포함입니다.

### A-4. 구체적 연구 공백 (A)
1. **ANN 인덱스와 함께 설계하는 시각 멀티벡터 압축**
   - 현재 압축은 brute-force MaxSim 기준 nDCG로 평가됩니다.
   - 제안: 프루닝·머지된 패치 벡터가 IVF/HNSW/MUVERA-FDE 후보 생성에서 어떻게 동작하는지 공동 최적화합니다. 압축이 centroid 분포를 바꿔 PLAID식 centroid pruning의 recall을 떨어뜨리는지 측정합니다.
   - 가장 가까운 연구: HPC-ColPali(HNSW 지연만 보고), Prune-then-Merge(29개 데이터셋, 인덱스 미고려), MUVERA(텍스트 ColBERT만).
   - 미해결 이유: 시각 패치의 높은 중복과 공간 구조를 FDE/centroid 인덱스와 함께 평가한 작업이 없습니다.
2. **백만 페이지 이상 규모의 VDR 스케일 벤치마크(정확도·지연·저장·갱신)**
   - ViDoRe v3 26k 페이지를 1–2 자릿수 넘어서는 코퍼스가 필요합니다. distractor로 실제 기업 PDF 덤프를 사용합니다.
   - 저장 바이트/페이지, p95 지연, 증분 삽입 비용을 공식 지표로 둡니다.
   - 가장 가까운 연구: ViDoRe v3, M3DocVQA(3k PDF), ViDoSeek("대규모" 주장이나 수천 페이지 수준).
3. **압축 인덱스의 모델 업그레이드 호환성**
   - 백본 교체(ColQwen2 → 2.5 → 차세대) 시 페이지 이미지를 전부 재인코딩하지 않고, 질의 측 어댑터로 이전 패치 인덱스를 재사용하는 방법입니다.
   - 가장 가까운 연구: Drift-Adapter(EMNLP 2025, 단일벡터 텍스트/CLIP), Query Drift Compensation(2025).
   - 미해결 이유: 멀티벡터·late-interaction 공간에서 패치 수가 다르거나 머지된 경우의 정렬 문제를 아무도 다루지 않았습니다.
4. **차트·표 요소 단위 하이브리드 인덱스**
   - 페이지 스크린샷 멀티벡터와 요소별(차트 데이터 테이블) 구조화 표현을 함께 색인하고, 질의 유형에 따라 라우팅하는 방식입니다.
   - 가장 가까운 연구: MMDocIR(레이아웃 단위 평가만), ViDoRAG(GMM 하이브리드 융합).
   - 미해결 이유: ViDoRe v3가 비텍스트 요소 약점을 명시했지만 인덱스 granularity 설계로 접근한 작업이 없습니다.
5. **비디오 RAG용 시간축 토큰 예산 인덱싱**
   - DocPruner식 질의 독립 적응형 프루닝을 프레임·클립 시간축으로 확장하여, 시간당 저장 예산 하에서 검색 recall을 최대화합니다.
   - 가장 가까운 연구: VideoRAG(Jeong, 프레임 선택은 생성용), CFVBench(평가만).

---

## 축 B. 효율 · 스케일 · 동적 인덱스

### B-1. 분야 현황 요약
- **성숙한 부분: 정적 단일벡터 ANN**
  - HNSW, DiskANN, IVF-PQ는 성숙했습니다.
  - 양자화는 RaBitQ(SIGMOD 2024)와 Extended RaBitQ(SIGMOD 2025)가 이론 오차 한계를 갖춘 이진·저비트 압축으로 표준 후보가 되었습니다.
- **멀티벡터 효율화: 실용 단계**
  - PLAID(2022)
  - MUVERA(NeurIPS 2024): FDE로 MIPS 환원
  - WARP(SIGIR 2025): XTR 대비 41×, PLAID 대비 약 3×
- **빠르게 움직이는 축 ①: 필터 검색**
  - 2025년에 통합 벤치마크가 3편 연달아 나왔습니다(2507.21989, 2508.16263, 2509.07789).
  - 공통 결론은 "단일 최강 방법 없음"과 "기존 비교의 편향"입니다.
- **빠르게 움직이는 축 ②: 동적성**
  - 스트리밍 삭제·삽입: IP-DiskANN(2025)
  - 임베딩 모델 업그레이드 무재색인: Drift-Adapter(EMNLP 2025), Query Drift Compensation(CoLLAs 2025)
  - 갱신 시점 판단: GradNormIR(ACL Findings 2025)
- **시간 민감 RAG**
  - HoH, DailyQA, ChronoQA, TempRAGEval 등 벤치마크는 늘었습니다.
  - 그러나 인덱스 계층에서 시간을 1급 시민으로 다루는 방법은 드뭅니다.
- **생성형 검색(GR)**
  - 이론적 장점이 주장되었습니다(2509.22116).
  - 그러나 동적 코퍼스에서 망각과 재학습 의존이 미해결입니다(DSI++, IncDSI, 2504.17519 재현연구).

### B-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [MUVERA: Multi-Vector Retrieval via Fixed Dimensional Encodings](https://arxiv.org/abs/2405.19504) | 2024 / NeurIPS 2024 | 멀티벡터 유사도를 근사하는 고정차원 인코딩(FDE)으로 단일벡터 MIPS에 환원. 같은 recall에서 후보 2–5× 감소, 지연 약 90% 감소 | BEIR | 미확인 |
| [WARP: An Efficient Engine for Multi-Vector Retrieval](https://arxiv.org/abs/2501.17788) | 2025.01 / SIGIR 2025 | XTR용 엔진: WARPSELECT, 암묵적 decompression, 2단계 reduction. XTR 대비 41×, PLAID 대비 약 3× | BEIR, LoTTE 등(세부 미확인) | 미확인 |
| [RaBitQ: Quantizing High-Dimensional Vectors with a Theoretical Error Bound for ANN Search](https://arxiv.org/abs/2405.12497) | 2024 / SIGMOD 2024 | D차원을 D비트로 무작위 회전 양자화. 불편 추정과 이론 오차 한계. 확장판(2409.09913, SIGMOD 2025)은 B비트/차원 | 표준 ANN 데이터셋 | 미확인 |
| [LEANN: A Low-Storage Vector Index](https://arxiv.org/abs/2506.08276) | 2025.06 arXiv (v2 2025.11) | 임베딩을 저장하지 않고 질의 시 재계산. hub 보존 그래프 프루닝. 인덱스를 원본 데이터의 5% 미만으로 줄이고, 90% top-3 recall을 2초 미만에 달성(로컬 디바이스) | RAG 벤치마크(세부 미확인) | 온라인 증분 갱신 미지원(2차 요약 출처, 미확인) |
| [In-Place Updates of a Graph Index for Streaming ANN Search (IP-DiskANN)](https://arxiv.org/abs/2502.13826) | 2025.02 arXiv | 배치 consolidation 없이 삽입·삭제를 in-place로 처리. 장기 갱신 중에도 recall 안정 | 스트리밍 갱신 시나리오 | 미확인. 동료심사 venue 미확인 |
| [Benchmarking Filtered ANN Search Algorithms on Transformer-based Embedding Vectors](https://arxiv.org/abs/2507.21989) | 2025.07 arXiv (ETH) | 필터 ANN 서베이·분류체계. arXiv 초록 270만 건 + 11개 속성 벤치마크 | 자체 | 모든 시나리오에서 최선인 단일 방법 없음 |
| [Attribute Filtering in ANN Search: An In-depth Experimental Study](https://arxiv.org/abs/2508.16263) | 2025.08 arXiv | 속성 유형(수치·범주·임의)과 전략(pre/post/joint)별 실험 비교 | FANNBench | 미확인 |
| [Filtered ANN Search: A Unified Benchmark and Systematic Experimental Study](https://arxiv.org/abs/2509.07789) | 2025.09 arXiv | filter-then-search / search-then-filter / hybrid 분류와 표준 평가 프레임워크 | 통합 벤치마크 | 기존 데이터셋·워크로드 편향으로 비교가 불신뢰하다고 지적 |
| [Drift-Adapter: Near Zero-Downtime Embedding Model Upgrades in Vector Databases](https://aclanthology.org/2025.emnlp-main.805) | 2025.09 / EMNLP 2025 | 새 모델 질의를 구 임베딩 공간으로 매핑(Procrustes / 저랭크 affine / residual MLP). 전체 재임베딩 recall의 95–99% 회복, 추가 지연 10μs 미만, 재계산 비용 100× 이상 절감 주장 | MTEB 텍스트, 1M CLIP 이미지 | 미확인. (관찰) 단일벡터만 다루며, 장기 다세대 업그레이드 누적은 미평가 |
| [Query Drift Compensation: Enabling Compatibility in Continual Learning of Retrieval Embedding Models](https://arxiv.org/abs/2506.00037) | 2025 / CoLLAs 2025 | 코퍼스 재인코딩 없이 구·신 공간의 drift를 추정하여 질의 임베딩에 보정. 문서가 질의보다 drift가 큼 | 미확인 | 미확인. 정확한 제목은 검색 요약 기반 |
| [When Should Dense Retrievers Be Updated in Evolving Corpora? Detecting OOD Corpora Using GradNormIR](https://arxiv.org/abs/2506.01877) | 2025 / ACL Findings 2025 | gradient norm으로 색인 전 코퍼스 OOD 여부를 예측하여 갱신 시점 결정 | BEIR | 새 문서를 색인하면 미래 질의 성능이 저하될 수 있다는 문제의식 |
| [HoH: A Dynamic Benchmark for Evaluating the Impact of Outdated Information on RAG](https://arxiv.org/abs/2503.04800) | 2025.03 arXiv | 현재·과거 문서를 함께 보유한 모의 검색엔진. 최신 정보가 있어도 outdated 정보가 정확도를 낮춤 | HoH | 미확인 |
| [DailyQA](https://arxiv.org/abs/2505.17162) (정식 제목 미확인) | 2025.05 arXiv | 매일 갱신되는 시간 제약 QA. 상위 랭크 문서가 시간 제약 불충족으로 오도할 수 있음 | DailyQA | 미확인 |
| [A Question Answering Dataset for Temporal-Sensitive RAG (ChronoQA)](https://arxiv.org/abs/2508.12282) | 2025.08 arXiv | 중국어 시간 민감 RAG QA. 명시·암시·상대 시간 표현 | ChronoQA | 중국어 한정(관찰) |
| [DSI++: Updating Transformer Memory with New Documents](https://arxiv.org/abs/2212.09744) | 2022 arXiv / EMNLP 2023 | GR 연속 색인. 새 문서 추가 시 망각 문제를 정식화. 생성 메모리(pseudo-query)로 완화 | NQ, MS MARCO | 새 문서 추가 시 기존 문서의 상당한 망각 |
| [IncDSI: Incrementally Updatable Document Retrieval](https://arxiv.org/abs/2307.10323) | 2023 / ICML 2023 | 제약 최적화로 문서를 빠르게 추가 | NQ320k, MS MARCO | 문서가 늘수록 기존 성능이 저하되어 결국 재학습 필요. 추가만 고려(삭제·수정 미고려) |
| [Replication and Exploration of Generative Retrieval over Dynamic Corpora](https://arxiv.org/abs/2504.17519) | 2025.04 / SIGIR 2025 | 동적 코퍼스 GR 재현. 숫자 docid는 새 문서에 약하고 텍스트 docid가 일반화에 유리. multi-docid 제안 | NQ, MS MARCO | 숫자 docid는 재학습 없이 적응 불가 |
| [Does Generative Retrieval Overcome the Limitations of Dense Retrieval?](https://arxiv.org/abs/2509.22116) | 2025.09 arXiv | 이론: DR은 local normalization 드리프트와 전역 저랭크 제약을 받고, GR은 스케일에서 유리 | NQ, MS MARCO | 실제로는 GR이 DR을 보편적으로 능가하지 못함 |
| [On the Theoretical Limitations of Embedding-Based Retrieval (LIMIT)](https://arxiv.org/abs/2508.21038) | 2025.08 / ICLR 2026 | sign-rank로 임베딩 차원별 표현 불가능한 top-k 조합을 증명. LIMIT 데이터셋에서 SOTA 임베딩 Recall@20 60% 미만, BM25 93.6% | LIMIT | 정확 표현에 대한 정리이며, 단일 합성 벤치마크 |
| [RAGO: Systematic Performance Optimization for RAG Serving](https://arxiv.org/abs/2503.14649) | 2025.03 / ISCA 2025 | RAGSchema 추상화와 배치·자원·배치정책 탐색. QPS/chip 최대 2×, TTFT 55% 감소. 병목이 검색과 추론 사이를 이동 | 시뮬레이션·프로파일링 | 에너지·비용 효율은 future work |

참고(배경, 2024 이전): PLAID([arXiv 2205.09707](https://arxiv.org/abs/2205.09707), CIKM 2022), HNSW, DiskANN/FreshDiskANN, IVF-PQ. 이 항목들은 이번 검색에서 개별 재검증하지 않았습니다. 다만 IP-DiskANN 초록에 FreshDiskANN의 배치 consolidation 언급이 있습니다.

### B-3. 반복되는 미해결 문제 / 암묵적 가정 / 벤치마크 공백
1. **"인덱스는 한 번 만들면 끝"이라는 가정**
   - MUVERA, WARP, RaBitQ, LEANN은 정적 코퍼스로 평가됩니다. LEANN은 증분 갱신 미지원이 지적됩니다(미확인 2차 출처).
   - 갱신 지원은 IP-DiskANN 등 그래프 단일벡터 쪽에 국한됩니다.
   - 멀티벡터(PLAID centroid, MUVERA FDE)의 스트리밍 갱신이나 centroid drift는 공백입니다.
2. **모델 업그레이드 = 전체 재임베딩이라는 관행**
   - Drift-Adapter와 QDC가 대안을 제시했지만, 단일벡터에서 한 세대 업그레이드만 다룹니다.
   - 다세대 누적 오차, 멀티벡터, 양자화(RaBitQ 코드) 인덱스와의 결합은 미평가입니다.
3. **세 축이 서로 독립적으로 평가됨**
   - 필터 ANN(2507.21989, 2508.16263, 2509.07789), 시간 민감 RAG(HoH, DailyQA, ChronoQA), 스트리밍 ANN(IP-DiskANN)은 각자 따로 평가됩니다.
   - 실제 RAG에서는 "최신성 필터 + 연속 삽입·삭제 + 시간 제약 질의"가 동시에 일어나는데, 이를 결합한 벤치마크는 없습니다.
4. **시간 민감 RAG 벤치마크가 생성기·리랭커 쪽 해법에 치중**
   - DailyQA는 리랭킹의 중요성을, HoH는 outdated 정보의 해악을 보여줍니다.
   - 인덱스 계층의 시간 메타데이터 활용(시간 필터 ANN, 버전 관리 인덱스)은 드뭅니다.
5. **GR의 동적성 문제는 재학습으로 귀결**
   - DSI++, IncDSI, 2504.17519 모두 기존 문서 망각과 재학습 필요를 보고합니다.
   - 2509.22116은 이론적 우위를 주장하면서도 실전에서는 DR을 보편적으로 능가하지 못한다고 인정합니다.
   - 삭제·수정을 다룬 연구는 드뭅니다(IncDSI는 추가만 고려).
6. **비용 지표의 부재**
   - RAGO는 에너지·비용을 future work로 남겼습니다.
   - 대부분 논문이 recall-QPS만 보고하고, 재임베딩 GPU 시간이나 저장 $/월 같은 총소유비용(TCO) 지표가 없습니다.
7. **단일벡터 표현 한계 vs 효율의 긴장**
   - LIMIT은 단일벡터의 이론 한계와 멀티벡터·희소의 우위를 보여줍니다.
   - 그런데 효율 연구(MUVERA, 축 A의 압축)는 멀티벡터를 단일벡터에 가깝게 압축하는 방향입니다. 압축률과 표현 가능 top-k 조합 사이의 관계는 측정되지 않았습니다.

### B-4. 구체적 연구 공백 (B)
1. **스트리밍 멀티벡터 인덱스(centroid/FDE drift 대응)**
   - PLAID/WARP의 centroid 코드북과 MUVERA FDE가 문서 연속 삽입·삭제와 분포 이동 하에서 얼마나 열화되는지 측정합니다.
   - 그리고 부분 재클러스터링 정책을 설계합니다.
   - 가장 가까운 연구: IP-DiskANN(단일벡터 그래프), WARP/MUVERA(정적).
   - 미해결 이유: 멀티벡터는 문서당 수백 토큰이라 갱신 단위·비용 구조가 다르지만 평가가 전무합니다.
2. **다세대·멀티벡터·양자화 인덱스에서의 backward-compatible 업그레이드**
   - Drift-Adapter와 QDC를 ColBERT/ColPali 토큰 공간과 RaBitQ·PQ 코드 인덱스로 확장합니다.
   - N세대 업그레이드 누적 오차를 측정하고, "언제 실제 재임베딩할지" 결정 규칙을 GradNormIR 같은 OOD 신호와 결합합니다.
   - 가장 가까운 연구: Drift-Adapter(단일벡터 1세대), QDC, GradNormIR(시점 판단만).
3. **시간 인식 필터 ANN + 버전 인덱스 벤치마크**
   - 문서마다 유효 기간(valid-from/to)을 두고, 질의 시점·암시적 시간 표현을 범위 필터로 변환하는 retrieval 계층입니다.
   - 스트리밍 갱신 하에서 정확도·지연을 측정하는 통합 벤치마크를 함께 제안합니다.
   - 가장 가까운 연구: 필터 ANN 벤치마크 3편(시간 없음), HoH·DailyQA(인덱스 없음), Re3([2509.01306](https://arxiv.org/abs/2509.01306), 검색 방법이나 세부 미확인).
4. **GR의 삭제·수정을 포함한 완전 동적 평가 프로토콜**
   - 추가·삭제·내용 수정·docid 재할당을 섞은 장기 스트림에서 GR과 DR+ANN(IP-DiskANN)을 같은 계산 예산으로 비교합니다.
   - 가장 가까운 연구: 2504.17519(추가 위주 재현), IncDSI(추가만), MixLoRA-DSI(세부 미확인).
   - 미해결 이유: 기존 평가가 망각만 측정하고 삭제 정합성(지워진 문서가 생성되지 않음)은 측정하지 않습니다. RAG 거버넌스와 "잊힐 권리" 측면에서 중요합니다.
5. **RAG 인덱스 TCO 모델**
   - 재임베딩 GPU 시간, 저장, 질의 지연, 갱신 빈도를 하나의 비용 함수로 두고, 압축률·업그레이드 정책·인덱스 종류를 공동 최적화합니다.
   - 가장 가까운 연구: RAGO(서빙 시스템, 비용은 future work), LEANN(저장↔재계산 trade-off, 로컬 한정).

---

## 축 교차 관찰
- 축 A의 저장 압축과 축 B의 동적·업그레이드 문제가 만나는 지점, 즉 **압축된 시각 멀티벡터 인덱스의 갱신과 모델 교체**는 양쪽 문헌 모두에서 비어 있습니다.
- LIMIT의 이론은 멀티벡터의 가치를 정당화합니다. 그러나 압축 연구는 그 가치를 얼마나 깎는지를 nDCG로만 측정합니다.
