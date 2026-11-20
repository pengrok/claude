# Survey D: RAG 검색/인덱싱 측 — (A) 멀티모달·시각 문서 검색, (B) 효율·스케일·동적 인덱스

작성일: 2026-11-19

**검증 방법:** 처음 작성할 때는 arxiv.org에 직접 접근할 수 없어 WebSearch로 제목·저자·날짜·초록 단편만 확인했습니다. 이후 축 A의 19편 전부를 논문 본문 텍스트(arXiv PDF, ACL Anthology PDF를 텍스트로 변환한 것)와 arXiv 메타데이터로 다시 대조했습니다. 확인 대상은 서지, 핵심 기여, 수치, 벤치마크, 저자가 밝힌 한계입니다. venue는 본문 첫 페이지 헤더·각주나 arXiv comment에 근거가 있을 때만 적었고, 근거가 없으면 "venue 근거 없음"으로 표시했습니다. "Stated limitations" 칸에는 Limitations / Conclusion / Future work 절에서 저자가 직접 밝힌 내용만 요약했습니다. 그런 절이 없으면 "명시된 한계 없음"이라고 적었습니다. "(관찰)"은 저자의 진술이 아니라 필자의 해석입니다.

---

## 축 A. 멀티모달 / 시각 문서 검색 (Visual Document Retrieval, VDR)

### A-1. 분야 현황 요약
- **성숙한 부분: 파싱 없는 검색**
  - ColPali(ICLR 2025)와 DSE(EMNLP 2024) 이후, 페이지 스크린샷을 VLM으로 직접 임베딩하는 방식이 OCR/파싱 파이프라인을 대체하는 사실상 표준이 되었습니다.
  - 표현 방식은 두 계열입니다. late-interaction 멀티벡터(ColPali/ColQwen/ColNomic/Jina-v4)와 단일벡터(DSE)입니다.
  - 모델 소형화(ModernVBERT, 250M)도 빠르게 진행되었습니다.
- **벤치마크 세대교체**
  - ViDoRe v1은 상위 모델이 nDCG@5 90을 넘으면서 포화되었습니다.
  - 그 뒤 v2(2025.03 블로그, 2025.05 arXiv. 블라인드 맥락 질의, 교차문서 질의, 다국어 질의)와 v3(2026.01 arXiv)가 나왔습니다. v3는 end-to-end RAG 평가, 26k 페이지, 3,099 질의, 6개 언어 규모이며 venue 근거는 없습니다.
  - MMDocIR(EMNLP 2025, 페이지/레이아웃 단위), VisDoM(NAACL 2025, 다문서·차트·표·슬라이드), ViDoSeek(EMNLP 2025, 1,142 질문) 등도 등장했습니다.
- **지금 가장 활발한 축: 멀티벡터 시각 인덱스의 저장비용**
  - 머지 계열: Light-ColPali(Findings ACL 2025). 메모리 2.8%만으로 성능 94.6%를 유지합니다.
  - 문서 적응형 프루닝: DocPruner(50–60% 절감)
  - 양자화+프루닝: HPC-ColPali(최대 32×)
  - Prune-then-Merge(Findings ACL 2026): near-lossless 범위를 60–70%로 확장했습니다.
- **아직 초기 단계: 비디오·오디오 RAG의 검색 측**
  - 비디오: VideoRAG 2종(그중 Jeong et al.은 Findings ACL 2025), CFVBench, WikiVideo/MIRAGE
  - 오디오: WavRAG(ACL 2025)
  - 대부분 소규모 코퍼스(예: 164편, 599편)에서 정확도만 다룹니다. 인덱스 규모·비용·갱신은 거의 다루지 않습니다.

### A-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [ColPali: Efficient Document Retrieval with Vision Language Models](https://arxiv.org/abs/2407.01449) | 2024 arXiv / ICLR 2025 | 페이지 이미지를 VLM(PaliGemma)로 패치 단위 멀티벡터로 임베딩하고 late interaction으로 매칭. 파싱 파이프라인보다 단순하면서 성능이 높음. ViDoRe 벤치마크 공개 | ViDoRe v1 | 명시된 한계 절 없음. Future Work: 벤치마크를 더 많은 언어·모달리티·태스크로 확장, 신뢰도 추정. 저장: 페이지당 1024 패치+6 토큰(D=128), 257.5 KB이며 압축·클러스터링·token pooling으로 개선 가능하다고 언급(§6) |
| [Unifying Multimodal Retrieval via Document Screenshot Embedding (DSE)](https://arxiv.org/abs/2406.11251) | 2024 arXiv (v2 2024.12) / EMNLP 2024 | 스크린샷을 단일 dense 벡터로 임베딩. Wiki-SS(130만 스크린샷) 공개. BM25 대비 top-1 +17pt, 슬라이드 검색에서 OCR 대비 nDCG@10 +15pt 이상 | Wiki-SS(NQ, TriviaQA), SlideVQA-open(50,714 슬라이드) | Limitations: PDF 등 다양한 구조의 일반 문서에서는 미검증, 대조 사전학습 미적용, 저화질 이미지에서는 성능이 떨어지고 고해상도 이미지에서는 효율이 떨어짐 |
| [VisRAG: Vision-based Retrieval-augmented Generation on Multi-modality Documents](https://arxiv.org/abs/2410.10594) | 2024 arXiv / ICLR 2025 | VLM 기반 검색기(VisRAG-Ret)와 생성기(VisRAG-Gen). 텍스트 RAG 대비 end-to-end 20–40% 향상. 단일 이미지만 받는 VLM용으로 page concatenation과 weighted selection 제안 | ArxivQA, ChartQA, MP-DocVQA, InfoVQA, PlotQA, SlideVQA(+합성 학습 데이터) | 명시된 한계 없음 |
| [M3DocRAG: Multi-modal Retrieval is What You Need for Multi-page Multi-document Understanding](https://arxiv.org/abs/2411.04952) | 2024.11 arXiv (venue 근거 없음) | ColPali 검색과 Qwen2-VL 생성을 결합한 open-domain 다문서 DocVQA. M3DocVQA(3,368 PDF, 41,005 페이지, 2,441 질문) 공개. IVFFlat/IVFPQ 근사 인덱스로 검색 지연을 21s에서 1.8s/0.2s로 줄이면서 정확도 대부분 유지(표4) | M3DocVQA, MP-DocVQA, MMLongBench-Doc | 명시된 한계 없음 |
| [ViDoRe Benchmark V2: Raising the Bar for Visual Retrieval](https://arxiv.org/abs/2505.17166) | 2025.05 arXiv (원래 2025.03 블로그 공개, venue 근거 없음) | v1 포화(>90 nDCG@5)에 대응. 블라인드 맥락 질의, 장문·교차문서 질의, 합성+HITL 생성 | 4개 데이터셋(Insurance, Biomedical, Economics, ESG) + 다국어 번역판(fr/en/es/de). Insurance는 저작권 문제로 이후 제거 | 명시된 한계 없음. (관찰) 저자 스스로 v1과 순위 상관이 강하다고 보고하므로, 변별력 증가는 주로 절대 점수 여유에서 옴 |
| [ViDoRe V3: A Comprehensive Evaluation of RAG in Complex Real-World Scenarios](https://arxiv.org/abs/2601.08620) | 2026.01 arXiv (v2 2026.04, venue 근거 없음) | 10개 전문 도메인 데이터셋(8개 공개, 2개 비공개 hold-out), 26,000 페이지, 3,099 질의 × 6개 언어. 어노테이션 12,000시간. 검색·생성·bounding-box grounding을 함께 평가. 결과: 모델들이 비텍스트 요소, 열린 질의, 세밀한 시각 grounding에서 고전 | ViDoRe v3 | Limitations: 원문은 영어·프랑스어뿐이고 질의는 서유럽 고자원 6개 언어에 한정. 공개된 장문 문서 위주라 이메일·티켓·손글씨 등 기업 문서 분포를 반영하지 못함 |
| [MMDocIR: Benchmarking Multimodal Retrieval for Long Documents](https://arxiv.org/abs/2501.08828) | 2025.01 arXiv / EMNLP 2025 | 장문서 페이지 단위와 레이아웃(표·차트·문단) 단위 검색. 평가셋 313 문서, 1,658 질의. 훈련셋 6,878 문서, 73,843 질의(본문 §4·표4 기준. 초록의 "173,843"과 "1,685"는 오기) | MMDocIR 평가셋(MMLongBench-Doc, DocBench 기반). 훈련셋은 7개 DocQA 데이터셋에서 구성 | Limitations: 7개 훈련 서브셋 중 3개에는 레이아웃 라벨이 없음. 텍스트·시각 공동 학습 부재(시각 검색기가 텍스트 passage 모델링에 약함) |
| [VisDoM: Multi-Document QA with Visually Rich Elements Using Multimodal RAG](https://aclanthology.org/2025.naacl-long.310) | 2024.12 arXiv(날짜는 본문 확인 불가) / NAACL 2025 | 표·차트·슬라이드 다문서 QA 벤치마크 VisDoMBench(2,271 질의, 1,277 문서). VisDoMRAG는 시각·텍스트 RAG를 병렬로 돌리고 consistency-constrained modality fusion으로 합침. 기준선 대비 12–20% 향상. 검색에서는 ColQwen2가 텍스트 검색기보다 우수 | VisDoMBench(PaperTab, FetaTab, SciGraphQA, SPIQA, SlideVQA) | Limitations: 텍스트 추출·파싱 오버헤드가 남아 있음. 질의당 LLM 3회 호출. 환각 위험 |
| [ViDoRAG: Visual Document RAG via Dynamic Iterative Reasoning Agents](https://aclanthology.org/2025.emnlp-main.464) | 2025.02 arXiv(날짜는 본문 확인 불가) / EMNLP 2025 | GMM 기반 시각·텍스트 하이브리드 검색(K 자동 결정)과 Seeker/Inspector/Answer 다중 에이전트 반복 추론. 기존 대비 10% 이상 향상. ViDoSeek 공개: 1,142 질문, 25–50쪽 영어 슬라이드 300개 | ViDoSeek, SlideVQA-Refined(2,020 질문) | Limitations: 전문가가 작성한 질의의 편향. 다중 에이전트 반복으로 계산 오버헤드와 지연이 커짐. 환각 |
| [jina-embeddings-v4: Universal Embeddings for Multimodal Multilingual Retrieval](https://arxiv.org/abs/2506.18902) | 2025.06 arXiv (venue 근거 없음) | 3.8B 모델. 단일벡터와 멀티벡터(late interaction)를 동시 지원. 태스크별 LoRA(각 60M). Jina-VDR 벤치마크 공개 | Jina-VDR, ViDoRe, CLIP Benchmark, MMTEB, MTEB-en, CoIR, LongEmbed, STS | 명시된 한계 절 없음. Future work: 다국어 능력 강화, 더 작은 효율 변형 |
| [ModernVBERT: Towards Smaller Visual Document Retrievers](https://arxiv.org/abs/2510.01149) | 2025.10 arXiv (venue 근거 없음) | 250M 인코더 ColModernVBERT. ViDoRe v1 81.2(ColPali 2.92B는 81.6). CPU 질의 인코딩 지연 20ms(ColPali 175ms) | ViDoRe v1, ViDoRe v2(영어 split) | Future Work & Limitations: 소형 모델에 한정, 영어 전용(다국어 확장 시 trade-off 미확인) |
| [Towards Storage-Efficient Visual Document Retrieval: An Empirical Study on Reducing Patch-Level Embeddings (Light-ColPali)](https://arxiv.org/abs/2506.04997) | 2025.06 arXiv / Findings ACL 2025 | 프루닝과 머징 실증 비교. 랜덤 프루닝이 정교한 프루닝보다 나음. 다만 질의 정보 없이 제거해야 하므로 프루닝은 VDR에 본질적으로 부적합. 후단 머지와 파인튜닝: 메모리 11.8%에서 성능 98.2%, 2.8%에서 94.6% 유지. 극단 압축에서는 파인튜닝이 손실의 61–67% 회복 | 3개 벤치마크 9개 데이터셋(ViDoRe 6개, VisRAG, MMLongBench-Doc) | Limitations: 토큰 감소만 다룸. 차원 축소, 벡터 양자화, 데이터 정제, 증류는 미탐색(직교적이라 결합 가능하다고 언급) |
| [DocPruner: A Storage-Efficient Framework for Multi-Vector VDR via Adaptive Patch-Level Embedding Pruning](https://arxiv.org/abs/2509.23883) | 2025.09 arXiv (Under review, venue 근거 없음) | 문서 내 패치 어텐션 분포로 문서별 임계값을 정하는 질의 독립(오프라인) 프루닝. 저장 50–60% 절감, 오프라인 인코딩 지연은 60–66% 증가 | ViDoRe-V2, JinaVDR(de/ru/zh/ja). 모델 ColQwen2.5, ColNomic, Jina-v4 | 명시된 한계 절 없음. Future work: 학습 과정에 프루닝 통합, 다른 모달리티로 확장 |
| [Hierarchical Patch Compression for ColPali (HPC-ColPali)](https://arxiv.org/abs/2506.21601) | 2025.06 arXiv (단일 저자, venue 근거 없음) | K-Means 양자화(1바이트 인덱스), 어텐션 기반 질의 시점 프루닝, 이진 인코딩. 최대 32× 압축(nDCG@10 하락 2% 미만). HNSW에서 질의 지연 30–50% 감소. RAG 환각 30% 감소 | ViDoRe, SEC-Filings | 명시된 한계 절 없음. Future Work: PQ 확장, 적응형 프루닝 정책, 동적 코퍼스용 스트리밍 코드북 갱신 |
| [Sculpting the Vector Space: Towards Efficient Multi-Vector VDR via Prune-then-Merge](https://arxiv.org/abs/2602.19549) | 2026.02 arXiv / Findings ACL 2026 | 저정보 패치를 먼저 적응형 프루닝(DocPruner와 동일)한 뒤 계층적으로 머지. near-lossless 압축 범위를 DocPruner의 50–60%에서 60–70%로 확장 | 6개 벤치마크 29개 데이터셋(ViDoRe-V1/V2, JinaVDR, REAL-MM-RAG, ViDoSeek, MMLongBench-Doc). ColQwen2.5, ColNomic, Jina-v4 | Limitations: 프루닝 효과가 base LVLM 어텐션의 신뢰도에 의존. adaptation/merging factor 등 사전 정의 하이퍼파라미터에 의존 |
| [VideoRAG: Retrieval-Augmented Generation over Video Corpus](https://arxiv.org/abs/2501.05874) | 2025.01 arXiv (Jeong et al.) / Findings ACL 2025 | 비디오 코퍼스에서 비디오를 검색하고 LVLM으로 생성. 적응형 프레임 선택은 검색(질의 독립)과 생성(질의 유도) 양쪽에 적용. 자막이 없으면 ASR 보조 텍스트 사용 | WikiHowQA × HowTo100M, Synthetic QA × HowTo100M | Limitations: 질의-비디오-정답 triple 데이터셋이 부족해 자체 구성으로 대체. 프레임 선택의 효과·효율은 추가 개선이 필요 |
| [VideoRAG: Retrieval-Augmented Generation with Extreme Long-Context Videos](https://arxiv.org/abs/2502.01549) | 2025.02 arXiv (HKU, venue 근거 없음) | 극장문 비디오용 RAG. 그래프 기반 텍스트 지식과 멀티모달 컨텍스트 인코딩을 결합한 dual-channel 구조 | LongerVideos(22 컬렉션, 164편, 602 질의, 약 134.6시간) | 명시된 한계 없음 |
| [CFVBench: A Comprehensive Video Benchmark for Fine-grained Multimodal RAG](https://arxiv.org/abs/2510.09266) | 2025.10 arXiv (venue 근거 없음) | 599개 비디오, 5,360 QA. 7개 검색법(Recall@1/3/5/10)과 14개 MLLM 평가. 적응형 프레임 밀도 증가와 도구 호출을 결합한 AVR 제안 | CFVBench | 명시된 한계 없음 |
| [WavRAG: Audio-Integrated RAG for Spoken Dialogue Models](https://aclanthology.org/2025.acl-long.613) | 2025.02 arXiv(날짜는 본문 확인 불가) / ACL 2025 | ASR 없이 원시 오디오를 임베딩·검색하는 텍스트+오디오 혼합 KB(WavRetriever). ASR+텍스트 RAG와 비슷한 검색 성능에 약 10× 가속(Whisper 크기별 4.8–14.4×) | HotpotQA(음성), Spoken-SQuAD, SLUE-SQA-5, 자체 혼합 모달리티 데이터셋 | Limitations: RAG가 응답의 음향 측면(억양, 표현성, 화자 스타일)에 얼마나 기여하는지는 미해결 |

### A-3. 반복되는 미해결 문제 / 암묵적 가정 / 벤치마크 공백
1. **"코퍼스는 작고 정적이다"라는 가정**
   - VDR 벤치마크 코퍼스는 대부분 수백~수만 페이지 규모입니다. ViDoRe v3는 26,000 페이지, MMDocIR 평가셋은 313문서(평균 65.1쪽), M3DocVQA는 3,368 PDF·41,005 페이지, ViDoSeek은 슬라이드 300개입니다.
   - 수백만 페이지 규모에서 멀티벡터 VDR의 recall·지연·저장을 함께 측정한 공개 벤치마크는 찾지 못했습니다.
   - 근거: ViDoRe v2/v3, MMDocIR, M3DocRAG, ViDoRAG.
2. **압축 연구가 "정확도 대비 바이트"만 본다는 점**
   - Light-ColPali, DocPruner, HPC-ColPali, Prune-then-Merge는 nDCG 대비 저장률을 보고합니다. DocPruner와 Prune-then-Merge는 오프라인 인코딩 지연 증가(각각 60–66%, 47–56%)도 보고합니다.
   - 압축된 패치 벡터를 ANN 인덱스(HNSW/IVF) 위에 올렸을 때의 후보 생성 recall, 인덱스 갱신 비용, 문서 추가 시 재클러스터링 비용은 거의 다루지 않습니다.
   - 부분 예외는 두 편입니다. HPC-ColPali는 HNSW 질의 지연을 보고하고, 동적 코퍼스용 스트리밍 코드북 갱신을 future work로 남겼습니다. M3DocRAG는 압축 없는 ColPali 페이지 인덱스에서 IVFFlat/IVFPQ의 속도-정확도 trade-off를 보고했습니다.
3. **압축이 모델에 종속된다는 점**
   - Light-ColPali는 극단 압축에서 파인튜닝이 필요하다고 보고합니다. 파인튜닝은 merge factor 25/49에서 손실의 61–67%를 회복합니다.
   - 압축 결과는 특정 백본(ColQwen2.5 등)에 묶이므로, 백본이 바뀌면 압축 인덱스 전체를 재생성해야 합니다. Prune-then-Merge도 프루닝이 base LVLM 어텐션에 의존한다는 점을 한계로 명시합니다. 이는 축 B의 모델 업그레이드 문제와 연결됩니다.
4. **페이지 단위 granularity 가정**
   - 대부분 페이지 단위 검색입니다. 레이아웃·요소 단위(MMDocIR)나 교차 페이지·교차 문서 근거(ViDoRe v2/v3, VisDoM)는 여전히 성능이 낮습니다.
   - ViDoRe v3는 "fine-grained visual grounding" 약점을 명시합니다.
5. **비텍스트 요소(차트·표)의 약점**
   - ViDoRe v3는 비텍스트 요소에서 성능이 떨어진다고 보고합니다.
   - 차트 수치 질의처럼 수치 추론이 필요한 질의를 검색 단계에서 처리하는 연구는 적습니다(VisDoM, MMDocIR). VisDoM도 시각 RAG와 함께 텍스트 추출·파싱 파이프라인을 유지합니다.
6. **비디오·오디오 검색에서 코퍼스 규모와 인덱스 설계 공백**
   - 비디오 RAG 논문들은 프레임 선택과 생성 품질에 집중합니다(VideoRAG ×2, CFVBench). 코퍼스 규모는 164편(LongerVideos)과 599편(CFVBench) 수준입니다.
   - 시간축 멀티벡터(프레임/클립 × 패치)의 저장 폭증을 다룬 연구는 이번 검토 범위에서 찾지 못했습니다.
7. **다국어 편향**
   - ModernVBERT는 영어 전용임을 명시합니다.
   - ViDoRe v3는 다국어 평가를 포함하지만 서유럽 고자원 6개 언어에 한정된다고 스스로 밝힙니다. DocPruner의 다국어 평가는 de/ru/zh/ja입니다. 저자원 언어는 둘 다 포함하지 않습니다.

### A-4. 구체적 연구 공백 (A)
1. **ANN 인덱스와 함께 설계하는 시각 멀티벡터 압축**
   - 현재 압축은 brute-force MaxSim 기준 nDCG로 평가됩니다.
   - 제안: 프루닝·머지된 패치 벡터가 IVF/HNSW/MUVERA-FDE 후보 생성에서 어떻게 동작하는지 공동 최적화합니다. 압축이 centroid 분포를 바꿔 PLAID식 centroid pruning의 recall을 떨어뜨리는지 측정합니다.
   - 가장 가까운 연구: HPC-ColPali(HNSW 지연만 보고), M3DocRAG(압축 없는 페이지 인덱스에서 IVF/PQ 속도-정확도), Prune-then-Merge(29개 데이터셋, 인덱스 미고려), MUVERA(텍스트 ColBERT만).
   - 미해결 이유: 시각 패치의 높은 중복과 공간 구조를 FDE/centroid 인덱스와 함께 평가한 작업이 없습니다.
2. **백만 페이지 이상 규모의 VDR 스케일 벤치마크(정확도·지연·저장·갱신)**
   - ViDoRe v3의 26k 페이지보다 1–2 자릿수 큰 코퍼스가 필요합니다. distractor로는 실제 기업 PDF 덤프를 사용합니다. ViDoRe v3도 기업 문서 분포를 반영하지 못한다는 점을 한계로 밝혔습니다.
   - 저장 바이트/페이지, p95 지연, 증분 삽입 비용을 공식 지표로 둡니다.
   - 가장 가까운 연구: ViDoRe v3, M3DocVQA(3,368 PDF, 41k 페이지), ViDoSeek(본문은 "large-scale"을 내세우지만 슬라이드 300개 × 25–50쪽, 약 7.5k–15k 페이지).
3. **압축 인덱스의 모델 업그레이드 호환성**
   - 백본 교체(ColQwen2 → 2.5 → 차세대) 시 페이지 이미지를 전부 재인코딩하지 않고, 질의 측 어댑터로 이전 패치 인덱스를 재사용하는 방법입니다.
   - 가장 가까운 연구: Drift-Adapter(EMNLP 2025, 단일벡터 텍스트/CLIP), Query Drift Compensation(CoLLAs 2025).
   - 미해결 이유: 멀티벡터·late-interaction 공간에서 패치 수가 다르거나 머지된 경우의 정렬 문제를 다룬 작업이 없습니다.
4. **차트·표 요소 단위 하이브리드 인덱스**
   - 페이지 스크린샷 멀티벡터와 요소별(차트 데이터 테이블) 구조화 표현을 함께 색인하고, 질의 유형에 따라 라우팅하는 방식입니다.
   - 가장 가까운 연구: MMDocIR(레이아웃 단위 평가와 훈련셋. 단 훈련 서브셋 3개는 레이아웃 라벨 없음), ViDoRAG(GMM 하이브리드 융합), VisDoM(시각·텍스트 병렬 RAG 융합).
   - 미해결 이유: ViDoRe v3가 비텍스트 요소 약점을 명시했지만 인덱스 granularity 설계로 접근한 작업이 없습니다.
5. **비디오 RAG용 시간축 토큰 예산 인덱싱**
   - DocPruner식 질의 독립 적응형 프루닝을 프레임·클립 시간축으로 확장하여, 시간당 저장 예산 하에서 검색 recall을 최대화합니다. DocPruner도 다른 모달리티로의 확장을 future work로 언급합니다.
   - 가장 가까운 연구: VideoRAG(Jeong). 검색 단계에서도 질의 독립 프레임 선택을 쓰지만, 저장 예산이나 인덱스 규모를 목표로 한 설계는 아닙니다. CFVBench(평가, AVR은 생성 단계의 프레임 밀도 조정).

---


---

## 축 B. 효율 · 스케일 · 동적 인덱스

### B-1. 분야 현황 요약
- **성숙한 부분: 정적 단일벡터 ANN**
  - HNSW, DiskANN, IVF-PQ는 성숙했습니다.
  - 양자화 쪽에서는 RaBitQ(SIGMOD 2024)와 그 확장판(2409.09913, arXiv preprint, venue 근거 없음)이 표준 후보가 되었습니다. 둘 다 이론 오차 한계를 갖춘 이진·저비트 압축입니다.
- **멀티벡터 효율화: 실용 단계**
  - PLAID(2022, arXiv preprint): vanilla ColBERTv2 대비 GPU 최대 7×, CPU 최대 45× 빠름
  - MUVERA(NeurIPS 2024): FDE로 MIPS에 환원
  - WARP(SIGIR 2025): XTR 대비 41×, PLAID 대비 약 3× 빠름
- **빠르게 움직이는 축 ①: 필터 검색**
  - 2025년에 통합 벤치마크가 3편 연달아 나왔습니다(2507.21989, 2508.16263[SIGMOD 2026], 2509.07789).
  - 세 편 모두 최적 방법이 필터 유형, 선택도, 규모에 따라 달라지므로 용도별로 골라야 한다고 결론짓습니다.
  - 기존 비교의 편향도 지적합니다(2509.07789 초록, 2508.16263 O2).
  - 2508.16263은 동적 필터 ANN 인덱스가 아직 거의 연구되지 않았다고 명시합니다(O4).
- **빠르게 움직이는 축 ②: 동적성**
  - 스트리밍 삭제·삽입: IP-DiskANN(2025, arXiv)
  - 임베딩 모델 업그레이드를 재색인 없이: Drift-Adapter(EMNLP 2025), Query Drift Compensation(CoLLAs 2025)
  - 갱신 시점 판단: GradNormIR(ACL Findings 2025)
- **시간 민감 RAG**
  - HoH, DailyQA, ChronoQA, TempRAGEval(본문 확인 불가) 등 벤치마크는 늘었습니다.
  - 그러나 인덱스 계층에서 시간을 1급 시민으로 다루는 방법은 드뭅니다.
- **생성형 검색(GR)**
  - 스케일에서의 이론적 장점이 주장되었습니다(2509.22116).
  - 그러나 동적 코퍼스에서는 망각과 재학습 의존이 남아 있습니다(DSI++, IncDSI, 2509.22116 §5).
  - 2504.17519 재현연구는 텍스트 docid가 새 문서에 잘 일반화되고, multi-docid 설계로 재학습 없이 개선된다고 보고합니다.

### B-2. 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [MUVERA: Multi-Vector Retrieval via Fixed Dimensional Encodings](https://arxiv.org/abs/2405.19504) | 2024 / NeurIPS 2024 (arXiv v2(2026)에서 Theorem 2.1의 FDE 차원 한계 정정) | 멀티벡터 유사도를 근사하는 고정차원 인코딩(FDE)으로 단일벡터 MIPS에 환원. SV heuristic과 같은 recall에서 후보 2–5× 감소. PLAID 대비 평균 recall 10% 향상, 지연 90% 감소 | BEIR 6개(MS MARCO, NQ, HotpotQA, ArguAna, SciDocs, Quora) | MS MARCO에서는 PLAID를 넘지 못함. 문서당 임베딩 수(m_avg)의 영향은 미연구(Broader Impacts and Limitations) |
| [WARP: An Efficient Engine for Multi-Vector Retrieval](https://arxiv.org/abs/2501.17788) | 2025.01 / SIGIR 2025 | XTR용 엔진: WARPSELECT, 암묵적 decompression, 2단계 reduction. XTR 참조 구현 대비 41×(LoTTE Pooled 6초 초과 → 171ms), PLAID 대비 약 3× | BEIR 6개, LoTTE 6개 | 별도 한계 절 없음. Future work로 SIMD/GPU 가속, 경량 query encoding, end-to-end 학습을 듦(§6) |
| [RaBitQ: Quantizing High-Dimensional Vectors with a Theoretical Error Bound for ANN Search](https://arxiv.org/abs/2405.12497) | 2024 / SIGMOD 2024 | D차원을 D비트로 무작위 회전 양자화. 불편 추정과 이론 오차 한계. 확장판([2409.09913](https://arxiv.org/abs/2409.09913), "Practical and Asymptotically Optimal Quantization…", arXiv preprint, venue 근거 없음)은 중간 압축률(B비트/차원)을 지원하고 공간-오차의 점근 최적성을 증명 | MSong, SIFT, DEEP, Word2Vec, GIST, Image | 그래프 인덱스와 다른 저장장치로의 확장, 하드웨어 최적화는 향후 과제(§6–7) |
| [LEANN: A Low-Storage Vector Index](https://arxiv.org/abs/2506.08276) | 2025.06 arXiv (v2 2025.11, Under Review) | 임베딩을 저장하지 않고 질의 시 재계산. hub 보존 그래프 프루닝. 인덱스를 원본 데이터의 5% 미만으로 줄이고(기존 대비 최대 50×), 90% top-3 recall을 1초 이내에 달성(v2). 삽입, 버퍼를 둔 배치 추가, soft delete로 갱신 지원(v2 §6, 부록 B) | NQ, TriviaQA, GPQA, HotpotQA(RPJ-Wiki), FinanceBench, Enron Email, LAION | 별도 한계 절 없음. 더 정교한 그래프 병합(RNG pruning 등)은 향후 과제 |
| [In-Place Updates of a Graph Index for Streaming ANN Search (IP-DiskANN)](https://arxiv.org/abs/2502.13826) | 2025.02 arXiv (venue 근거 없음) | 배치 consolidation 없이 삽입·삭제를 in-place로 처리. 장기 갱신 중에도 recall이 안정적이고, FreshDiskANN·HNSW보다 처리량과 갱신 속도가 좋음 | SlidingWindow·ExpirationTime·Clustered runbook × MSTuring-10M, Wikipedia(Cohere) 10M/35M | replacement 등 다른 adversarial workload, IVF·LSH 스트리밍 방법과의 비교는 향후 과제(§7) |
| [Benchmarking Filtered ANN Search Algorithms on Transformer-based Embedding Vectors](https://arxiv.org/abs/2507.21989) | 2025.07 arXiv (ETH, venue 근거 없음) | 필터 ANN 서베이·분류체계. arXiv 초록 270만 건 이상(4096차원) + 11개 속성의 arxiv-for-fanns 데이터셋. 11개 방법 벤치마크와 8개 관찰 | arxiv-for-fanns(small/medium/large) | 자체 한계 절 없음. 관찰: 특정 필터 전용 방법이 범용 방법보다 최대 10× 빠름(Obs.3). 대규모에서는 recall이 0.2 미만으로 떨어지는 방법도 있음(Obs.5) |
| [Attribute Filtering in ANN Search: An In-depth Experimental Study](https://arxiv.org/abs/2508.16263) | 2025.08 / SIGMOD 2026 | 속성 유형(수치·범주·임의)과 전략(pre/post/joint)별 분류와 실험 비교. 10개 알고리즘, 12개 방법, 선택도 0.1–100% | SIFT, SpaceV(각 1,000만), Redcaps, Youtube-RGB(코드: FANNBench) | §6.3 Open Problems: 임의 필터는 미해결, 하이퍼파라미터 민감, 벡터 DB 통합이 제한적, **동적 필터 ANN 인덱스가 미개척**(DSG만 갱신 지원), 속성을 uniform 분포로 가정 |
| [Filtered ANN Search: A Unified Benchmark and Systematic Experimental Study](https://arxiv.org/abs/2509.07789) | 2025.09 arXiv (venue 근거 없음) | filter-then-search / search-then-filter / hybrid 분류. 통합 파라미터 튜닝을 쓰는 표준 평가 프레임워크. 기존 비교가 데이터셋·워크로드 편향 때문에 신뢰하기 어렵다고 지적 | arXiv, TripClick, LAION1M, YFCC, YouTube Audio, YouTube Video | 명시된 한계 없음 |
| [Drift-Adapter: A Practical Approach to Near Zero-Downtime Embedding Model Upgrades in Vector Databases](https://aclanthology.org/2025.emnlp-main.805) | 2025 / EMNLP 2025 (Main) | 새 모델 질의를 구 임베딩 공간으로 매핑(Orthogonal Procrustes / 저랭크 affine / residual MLP). 전체 재임베딩 recall의 95–99% 회복, 추가 지연 10μs 미만, 재계산 비용 100× 이상 절감 | MTEB 텍스트(AG-News, DBpedia-14, Emotion, 각 1M), LAION 1M(CLIP ViT-B/32 → L/14) | Limitations 절: drift의 크기·유형에 의존, 구 모델 접근 필요, 전역 adapter 하나뿐(이질적 drift), 재계산을 미룰 뿐 없애지는 않음, 10억 규모 검증 없음, **연쇄 적응의 누적 오차는 미연구** |
| [Query Drift Compensation: Enabling Compatibility in Continual Learning of Retrieval Embedding Models](https://arxiv.org/abs/2506.00037) | 2025 / CoLLAs 2025 | 코퍼스 재인코딩 없이 구·신 공간 사이의 drift를 추정해 질의 임베딩에 보정하고, 임베딩 distillation을 병행. 문서(특히 긴 문서)가 질의보다 drift가 큼. 비싼 재인덱싱도 성능이 좋지 않음 | BEIR 5개 순차 task(MS MARCO → NQ → HotpotQA → FEVER → FiQA-2018) | 한계 절 없음. task-id를 모르는 CIL 설정, 다른 retriever 구조(ColBERT 등)는 향후 과제(부록 A, C) |
| [When Should Dense Retrievers Be Updated in Evolving Corpora? Detecting OOD Corpora Using GradNormIR](https://arxiv.org/abs/2506.01877) | 2025 / ACL Findings 2025 | gradient norm으로 색인 전에 코퍼스의 OOD 여부를 예측하여 갱신 시점 결정 | BEIR 10개(순차 session) | 문서 수준에서만 판단하고 미래 질의를 고려하지 않음(Limitations). 연속 도착 문서에 대한 online OOD 탐지는 향후 과제 |
| [HoH: A Dynamic Benchmark for Evaluating the Impact of Outdated Information on RAG](https://arxiv.org/abs/2503.04800) | 2025.03 arXiv (venue 근거 없음) | Wikipedia 스냅샷 diff로 만든 QA 96,124개, 문서 219,463개. 현재·과거 문서를 함께 보유한 모의 검색엔진. 최신 정보가 있어도 outdated 정보가 정확도를 낮춤 | HoH | Wikipedia 스냅샷의 고정 주기, 단일 문서 기반 QA, 모의 검색엔진의 outdated 문서가 동질적(§7) |
| [DailyQA: A Benchmark to Evaluate Web Retrieval Augmented LLMs Based on Capturing Real-World Changes](https://arxiv.org/abs/2505.17162) | 2025.05 arXiv | Wikipedia revision log 기반으로 질문은 매주, 답은 매일 갱신. 상위 랭크 문서가 시간 제약을 충족하지 않아 오도할 수 있음. 웹 결과 rerank가 중요 | DailyQA | one-hop만 다룸, 추론형 LLM 미평가, 일부 페이지의 갱신 시각 미확보로 Rerank-T 성능이 떨어질 수 있음(Limitations) |
| [A Question Answering Dataset for Temporal-Sensitive RAG (ChronoQA)](https://arxiv.org/abs/2508.12282) | 2025.08 arXiv | 중국어 뉴스 30만 건 이상(2019–2024)으로 만든 5,176개 QA. 시간 유형은 absolute/aggregate/relative, 시간 표현은 explicit/implicit. 단일·다중 문서 | ChronoQA. 검색 baseline은 Native RAG, Temporal Filter, Query Rewrite, Query Decomposition | 명시된 한계 없음. (관찰) 중국어 한정 |
| [DSI++: Updating Transformer Memory with New Documents](https://arxiv.org/abs/2212.09744) | 2022 arXiv / EMNLP 2023 | GR 연속 색인. 새 문서 추가 시 망각을 정식화. SAM(flat minima)과 생성 메모리(pseudo-query)로 완화(NQ Hits@10 +21.1%, 재학습 대비 업데이트 6× 감소) | NQ, MS MARCO | 새 문서가 기존 문서를 반박·수정하면 동작을 예측할 수 없음. full MS MARCO에서는 망각이 여전히 큼(Limitations) |
| [IncDSI: Incrementally Updatable Document Retrieval](https://arxiv.org/abs/2307.10323) | 2023 / ICML 2023 | 제약 최적화로 문서를 빠르게 추가(문서당 약 20–50ms) | NQ320K, MS MARCO | 문서가 늘수록 기존 성능이 저하되어 결국 재학습 필요. 추가만 고려하며 편집은 향후 과제, 삭제는 언급 없음(§7) |
| [Replication and Exploration of Generative Retrieval over Dynamic Corpora](https://arxiv.org/abs/2504.17519) | 2025.04 / SIGIR 2025 | 동적 코퍼스(문서 추가)에서 GR 재현. 숫자 docid는 초기 문서 쪽으로 편향되어 약하고, 텍스트 docid는 일반화에 유리. multi-docid(MDGR)는 재학습 없이 개선 | NQ, MS MARCO | 별도 한계 절 없음. 숫자 docid 방법은 추가 학습 없이 일반화하기 어려움(§7) |
| [Does Generative Retrieval Overcome the Limitations of Dense Retrieval?](https://arxiv.org/abs/2509.22116) | 2025.09 arXiv (venue 근거 없음) | 이론: DR은 local normalization에 의한 최적화 drift와 전역 저랭크 제약을 받고, GR은 스케일에서 유리 | NQ, MS MARCO | 실제로는 GR이 DR을 보편적으로 능가하지 못함. 이상화한 이론, 대규모·SOTA 변형 비교 미실시(§6). 연속 corpus drift에서 GR은 재학습이 필요(§5) |
| [On the Theoretical Limitations of Embedding-Based Retrieval (LIMIT)](https://arxiv.org/abs/2508.21038) | 2025.08 / ICLR 2026 | sign-rank로 임베딩 차원별로 표현할 수 없는 top-k 조합을 증명. full LIMIT Recall@100에서 단일벡터 SOTA 최고 18.9%, GTE-ModernColBERT 54.8%, BM25 93.6% | LIMIT(full, 46문서 small) | 이론은 단일벡터에 한정(멀티벡터 확장은 향후 과제). 오류를 허용하는 설정의 bound 없음. 실패할 조합을 사전에 특정할 수 없음(Limitations) |
| [RAGO: Systematic Performance Optimization for RAG Serving](https://arxiv.org/abs/2503.14649) | 2025.03 arXiv (ISCA 2025 표기는 본문 근거 없음) | RAGSchema 추상화와 배치·자원·배치정책 탐색. QPS/chip 최대 2×, TTFT 55% 감소. 병목이 검색과 추론 사이를 이동(hyperscale에서 검색이 80% 초과, long-context에서 1% 미만) | 4개 RAG 패러다임의 시뮬레이션·프로파일링(TPU급 XPU) | 에너지·비용 효율은 future work(§9) |

참고(배경, 2024 이전):
- PLAID([arXiv 2205.09707](https://arxiv.org/abs/2205.09707)): 로컬 본문은 arXiv preprint이며 CIKM 2022 표기는 근거가 없습니다.
- HNSW, DiskANN/FreshDiskANN, IVF-PQ: 개별 재검증하지 않았습니다. FreshDiskANN이 삭제를 배치로 모아 consolidation한다는 점은 IP-DiskANN 초록과 §3.2에서 확인됩니다.

### B-3. 반복되는 미해결 문제 / 암묵적 가정 / 벤치마크 공백
1. **"인덱스는 한 번 만들면 끝"이라는 가정**
   - MUVERA, WARP, RaBitQ는 정적 코퍼스로 평가됩니다.
   - LEANN은 v2에서 삽입·배치 추가·soft delete를 지원합니다. IP-DiskANN과 마찬가지로 그래프 기반 단일벡터 인덱스입니다.
   - 따라서 갱신 지원은 그래프 단일벡터 쪽에 국한됩니다.
   - 필터 ANN에서도 동적 인덱스는 미개척으로 명시되어 있습니다(2508.16263 O4).
   - 멀티벡터(PLAID centroid, MUVERA FDE)의 스트리밍 갱신과 centroid drift는 공백입니다.
2. **모델 업그레이드 = 전체 재임베딩이라는 관행**
   - Drift-Adapter는 단일벡터에서 모델 교체를 한 번만 평가했고, 연쇄 적응의 누적 오차는 미연구라고 스스로 밝힙니다.
   - QDC는 5개 task를 순차로 갱신하는 다세대 설정을 다룹니다. 다만 같은 모델의 연속 fine-tuning이며 task-id를 안다고 가정합니다.
   - 서로 다른 모델 교체를 여러 세대 거치는 경우, 멀티벡터, 양자화(RaBitQ 코드) 인덱스와의 결합은 미평가입니다.
3. **세 축이 서로 독립적으로 평가됨**
   - 필터 ANN(2507.21989, 2508.16263, 2509.07789), 시간 민감 RAG(HoH, DailyQA, ChronoQA), 스트리밍 ANN(IP-DiskANN)은 각자 따로 평가됩니다.
   - 실제 RAG에서는 "최신성 필터 + 연속 삽입·삭제 + 시간 제약 질의"가 동시에 일어나는데, 이를 결합한 벤치마크는 없습니다.
4. **시간 민감 RAG 벤치마크가 생성기·리랭커 쪽 해법에 치중**
   - DailyQA는 리랭킹의 중요성을, HoH는 outdated 정보의 해악을 보여줍니다.
   - ChronoQA는 단순 Temporal Filter 검색 baseline을 평가했는데 Native RAG보다 낮았습니다.
   - 인덱스 계층의 시간 메타데이터 활용(시간 필터 ANN, 버전 관리 인덱스)은 드뭅니다.
5. **GR의 동적성 문제는 대체로 재학습으로 귀결**
   - DSI++와 IncDSI는 기존 문서 망각과 결국 재학습이 필요하다는 점을 보고합니다.
   - 2509.22116은 이론적 우위를 주장하면서도, 실전에서는 DR을 보편적으로 능가하지 못하고 corpus drift 하에서 재학습이 필요하다고 인정합니다(§5).
   - 반면 2504.17519는 텍스트 docid와 multi-docid로 재학습 없이 개선된다고 보고합니다. 다만 문서 추가 시나리오에 한정됩니다.
   - 삭제·수정을 다룬 연구는 드뭅니다. IncDSI는 추가만 고려하고, DSI++는 수정·반박 문서에서의 동작이 예측 불가하다고 명시합니다.
6. **비용 지표의 부재**
   - RAGO는 에너지·비용을 future work로 남겼습니다.
   - 대부분 논문이 recall-QPS만 보고합니다. 재임베딩 GPU 시간이나 월별 저장 비용 같은 총소유비용(TCO) 지표는 없습니다.
7. **단일벡터 표현 한계 vs 효율의 긴장**
   - LIMIT은 단일벡터의 이론 한계를 보이고, full LIMIT에서 멀티벡터(54.8%)와 BM25(93.6%)가 단일벡터(최고 18.9%, Recall@100)보다 낫다는 결과를 냅니다.
   - 다만 저자는 이론을 멀티벡터로 확장하지 않았다고 밝힙니다.
   - 그런데 효율 연구(MUVERA, 축 A의 압축)는 멀티벡터를 단일벡터에 가깝게 압축하는 방향입니다. 압축률과 표현 가능한 top-k 조합 사이의 관계는 측정되지 않았습니다.

### B-4. 구체적 연구 공백 (B)
1. **스트리밍 멀티벡터 인덱스(centroid/FDE drift 대응)**
   - PLAID/WARP의 centroid 코드북과 MUVERA FDE가 문서 연속 삽입·삭제와 분포 이동 하에서 얼마나 열화되는지 측정합니다.
   - 그리고 부분 재클러스터링 정책을 설계합니다.
   - 가장 가까운 연구: IP-DiskANN과 LEANN(단일벡터 그래프), WARP/MUVERA(정적).
   - 미해결 이유: 멀티벡터는 문서당 수백 토큰이라 갱신 단위와 비용 구조가 다르지만, 평가가 전무합니다.
2. **다세대·멀티벡터·양자화 인덱스에서의 backward-compatible 업그레이드**
   - Drift-Adapter와 QDC를 ColBERT/ColPali 토큰 공간과 RaBitQ·PQ 코드 인덱스로 확장합니다.
   - 서로 다른 모델로 N세대 교체할 때의 누적 오차를 측정합니다. Drift-Adapter는 이를 미연구로 명시했고, QDC는 같은 모델의 연속 fine-tuning만 다뤘습니다.
   - "언제 실제로 재임베딩할지" 정하는 규칙을 GradNormIR 같은 OOD 신호와 결합합니다.
   - 가장 가까운 연구: Drift-Adapter(단일벡터, 1회 교체), QDC(5-task 연속 학습), GradNormIR(시점 판단만, online 탐지는 향후 과제).
3. **시간 인식 필터 ANN + 버전 인덱스 벤치마크**
   - 문서마다 유효 기간(valid-from/to)을 두고, 질의 시점과 암시적 시간 표현을 범위 필터로 변환하는 retrieval 계층입니다.
   - 스트리밍 갱신 하에서 정확도와 지연을 측정하는 통합 벤치마크를 함께 제안합니다.
   - 가장 가까운 연구:
     - 필터 ANN 벤치마크 3편: 시간을 다루지 않습니다. 2508.16263은 동적 필터 인덱스를 미개척으로 명시합니다.
     - HoH·DailyQA: 인덱스를 다루지 않습니다.
     - ChronoQA: 단순 Temporal Filter baseline만 있습니다.
     - Re3([2509.01306](https://arxiv.org/abs/2509.01306)): relevance와 recency의 균형을 다룹니다. 저자가 arXiv에서 철회했으며(문제 설정과 방법을 크게 바꾼 개정 원고 준비 중), 본문 접근 불가입니다.
4. **GR의 삭제·수정을 포함한 완전 동적 평가 프로토콜**
   - 추가·삭제·내용 수정·docid 재할당을 섞은 장기 스트림에서, GR과 DR+ANN(IP-DiskANN)을 같은 계산 예산으로 비교합니다.
   - 가장 가까운 연구: 2504.17519(문서 추가 위주 재현), IncDSI(추가만, 편집은 향후 과제), DSI++(수정 문서에서의 동작 예측 불가를 한계로 명시), MixLoRA-DSI(본문 확인 불가).
   - 미해결 이유: 기존 평가는 망각만 측정하고, 삭제 정합성(지워진 문서가 생성되지 않음)은 측정하지 않습니다. RAG 거버넌스와 "잊힐 권리" 측면에서 중요합니다.
5. **RAG 인덱스 TCO 모델**
   - 재임베딩 GPU 시간, 저장, 질의 지연, 갱신 빈도를 하나의 비용 함수로 두고, 압축률·업그레이드 정책·인덱스 종류를 공동 최적화합니다.
   - 가장 가까운 연구: RAGO(서빙 시스템, 비용은 future work), LEANN(저장과 재계산의 trade-off, 개인 디바이스 중심), Drift-Adapter(재계산 비용 >100× 절감 추정).

---

## 축 교차 관찰
- 축 A의 저장 압축과 축 B의 동적·업그레이드 문제가 만나는 지점, 즉 **압축된 시각 멀티벡터 인덱스의 갱신과 모델 교체**는 양쪽 문헌 모두에서 비어 있습니다.
- LIMIT의 결과는 멀티벡터의 가치를 뒷받침합니다(full LIMIT에서 ColBERT 54.8% vs 단일벡터 최고 18.9%, Recall@100). 다만 이론적 정당화는 단일벡터에 한정됩니다.
- 그러나 압축 연구는 그 가치를 얼마나 깎는지를 nDCG로만 측정합니다.
