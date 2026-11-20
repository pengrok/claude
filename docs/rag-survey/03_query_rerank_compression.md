# Survey C — RAG 검색 측면: (A) Query-side 처리, (B) Reranking·Filtering·Compression

작성일: 2026-11-19 (축 A 본문 대조 교정: 2026-11-20)

**검증 방법 메모**: 처음 작성할 때는 arxiv.org 직접 fetch가 막혀 있어(ENOTFOUND) WebSearch 결과로 제목, 저자, ID, 초록 수준 주장을 교차 확인했다. 이후 축 A에 인용한 arXiv 논문 23편은 모두 arXiv 본문(PDF 텍스트 추출)과 arXiv API 메타데이터로 다시 대조했다. 제목, 연도, 수치, 벤치마크, 저자가 밝힌 한계를 본문 표와 절에서 직접 확인했다. venue는 본문 첫 페이지 헤더·각주와 arXiv comment로 정했다. 둘 다 근거가 없으면 "venue 근거 없음"으로 적었고, 다른 논문의 참고문헌에서만 확인되는 venue는 "(타 논문 인용)"이라고 표시했다. 본문에서 찾지 못한 수치는 "본문 확인 불가"로 남겼다.

---

## 축 A. Query-side processing (rewriting / expansion / decomposition / routing / when-to-retrieve)

### A.1 분야 현황 요약
- **성숙한 부분**: 생성형 query expansion은 sparse 검색이나 약한 retriever에서 꾸준히 이득을 준다. HyDE, Query2doc, 그리고 RL로 학습하는 rewriter(Rewrite-Retrieve-Read)가 대표적이다. 다만 2023–2025 분석 연구들은 실패 조건을 분명히 밝혔다. 강한 retriever와 reranker에서는 오히려 성능이 떨어지고(2309.08541), LLM이 모르는 쿼리나 모호한 쿼리에서도 떨어진다(2505.12694).
- **when-to-retrieve 계보**: Self-RAG(reflection token), FLARE(다음 문장 확신도), DRAGIN(토큰 불확실성·attention 중요도), SeaKR(내부 상태 불확실성)로 이어진다. 2025년 대규모 비교(Moskvoretskii et al., 2501.12835)는 adaptive retrieval 시스템 35개를 비교했다. 그 결과 기존 불확실성 추정(UE) 방법이 최신 방법들과 QA 성능은 비슷하면서 효율과 self-knowledge에서는 더 낫다고 보고했다. 이 결과는 정교한 트리거 설계의 가치에 의문을 던진다. 단, 이 비교에는 Self-RAG가 포함되지 않았다.
- **2025–2026의 흐름**: 첫째, 쿼리 생성, 분해, 검색 시점을 end-to-end로 학습하는 agentic 검색이 늘었다. Search-R1과 DeepRetrieval은 RL로, CoRAG는 rejection sampling과 SFT로 학습한다. 둘째, 2026년에는 "예산과 비용을 고정한 상태에서 평가해야 한다"는 메타 평가 논문들이 나왔다(2607.24010, 2606.29959, 2606.02245). 셋째, adaptive retrieval의 역할이 모델 능력에 따라 달라진다는 재검토가 나왔다. 약한 모델에서는 노이즈 필터, 강한 추론 모델에서는 효율 최적화 역할을 한다(2604.15621).
- **아직 움직이는 부분**: 검색 트리거의 calibration과 transfer, 대형 추론 모델(LRM)의 추론 도중 검색(2604.26649, SIGIR 2026), 대화형 검색 rewriting을 corpus 피드백 기반 RL로 학습하는 연구(2610.10955), 검색 corpus에 맞춘 재작성 전략 라우팅(2610.33768)이 여기에 속한다. 인접 분야에서는 다국어 추론 입력을 RL로 적응적으로 재작성하는 연구가 있다(2610.04899). 다만 이 논문은 검색을 포함하지 않아 RAG query 처리로 직접 분류하지 않는다.

### A.2 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Precise Zero-Shot Dense Retrieval without Relevance Labels (HyDE)](https://arxiv.org/abs/2212.10496) | arXiv 2022-12 / venue는 본문·comment 근거 없음(타 논문 인용: ACL 2023) | instruction LLM(InstructGPT)이 가상 답변 문서를 생성한다. 비지도 대조학습 인코더(Contriever)로 그 문서를 임베딩해 검색한다. 인코더의 dense bottleneck이 잘못된 세부를 걸러낸다고 주장한다. | TREC DL19/20, BEIR 저자원 6개, Mr.TyDi(sw, ko, ja, bn) | 명시된 Limitations 절 없음. 결론(6절)에서는 검색 로그가 쌓이면 지도학습 dense retriever로 점진 전환하라고 제언한다. (후속 연구 지적: LLM이 모르는 쿼리에서 생성 기반 확장이 해가 됨, 2505.12694) |
| [Query2doc: Query Expansion with Large Language Models](https://arxiv.org/abs/2303.07678) | 2023 / EMNLP 2023 | few-shot 프롬프트로 pseudo-doc을 생성한 뒤 쿼리에 이어 붙인다. BM25와 dense 모두에 적용하며, BM25 성능을 3–15% 높인다. | MS MARCO dev, TREC DL19/20, BEIR 5개(DBpedia, NFCorpus, SciFact, TREC-COVID, Touché2020) | Limitations: LLM 호출 지연(>2000ms)이 있고, 확장된 쿼리로 역색인 검색도 느려진다. 3.2절: 중간 사전학습이나 cross-encoder 증류를 거친 dense(SimLM, E5)에서는 이득이 줄고, NFCorpus·SciFact에서는 소폭 하락한다. |
| [Query Rewriting for Retrieval-Augmented Large Language Models (Rewrite-Retrieve-Read)](https://arxiv.org/abs/2305.14283) | 2023 / EMNLP 2023 | 소형 rewriter(T5-large)를 warm-up한 뒤, black-box LLM reader의 성능을 보상으로 PPO로 학습한다. | HotpotQA, AmbigNQ, PopQA, MMLU (검색기는 Bing) | Limitations: 학습 단계를 추가하므로 일반화와 특화 사이 trade-off가 생기고, few-shot 대비 직접 transfer가 약해진다. 단일 턴 retrieve-then-read 설계다. 웹 검색엔진에 의존한다. |
| [When do Generative Query and Document Expansions Fail? A Comprehensive Study Across Methods, Retrievers, and Datasets](https://arxiv.org/abs/2309.08541) | 2023 arXiv → EACL 2024 (arXiv comment) | 확장 기법 11종, 데이터셋 12개, retrieval 모델 24개를 체계적으로 비교했다. retriever 성능과 확장 이득 사이에 강한 음의 상관이 있었다. 약한 모델에는 도움이 되지만 강한 모델(대형 reranker 포함)에는 해가 된다. | ArguAna, FiQA, GooAQ Tech, NFCorpus, Quora, SciFact-Refute, TREC-CT, TREC-DL19/20, ToT, Touché, WikiQA | Limitations: ranker를 zero-shot으로만 평가했고, 확장에 맞춘 학습은 하지 않았다. 확장이 필요한지 정하려면 대상 도메인의 라벨된 평가 데이터가 필요하다. |
| [LLM-based Query Expansion Fails for Unfamiliar and Ambiguous Queries](https://arxiv.org/abs/2505.12694) | 2025 / SIGIR 2025 short | 실패 조건 두 가지를 통제 실험으로 확인했다. 첫째, LLM 지식이 부족하면 존재하지 않는 엔티티를 생성한다. 둘째, 쿼리가 모호하면 인기 있는 해석 쪽으로 편향된다. | 지식 축: NQ, TriviaQA, MS MARCO, BioASQ. 모호성 축: AmbigDocs, AmbigQA, TREC Web 09–12. Retriever: BM25, Contriever, E5-base-v2. QE: Q2D, Q2E, GaQR | 분석 논문이라 해법은 없다. 결론의 future work: (i) PRF, CoT, 다중 확장 파이프라인에서 재검증 (ii) LLM의 쿼리 지식과 모호성을 평가해 adaptive QE에 쓰는 모듈 개발. |
| [ThinkQE: Query Expansion via an Evolving Thinking Process](https://arxiv.org/abs/2506.09260) | 2025 / EMNLP 2025 Findings | 학습 없이 thinking 기반 확장과 corpus 피드백(top-5 문서)을 여러 라운드 반복한다. | DL19/20, BRIGHT StackExchange 7개(평균 nDCG@10: R1-14B 백본 34.9, 최고 36.0[Phi-4-Reasoning-14B], 표 4) | Limitations: 단일 확장보다 추론 지연과 비용이 크다. 영어 웹 검색만 실험해 다국어는 검증하지 않았다. (우리 관찰: 표 4에서 listwise reranker Rank-K-32B가 37.9로 더 높다.) |
| [Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection](https://arxiv.org/abs/2310.11511) | arXiv 2023-10 (Preprint) / venue는 본문·comment 근거 없음(타 논문 인용: ICLR 2024) | reflection token으로 검색 여부, 관련성, 근거성을 스스로 판단한다. critic 라벨은 GPT-4로 만든 뒤 증류한다. | PopQA, TriviaQA, PubHealth, ARC-Challenge, Biography(FactScore), ASQA | 명시된 Limitations 절 없음. Ethical Concerns: 인용으로 완전히 뒷받침되지 않는 출력이 여전히 나올 수 있다. |
| [Active Retrieval Augmented Generation (FLARE)](https://arxiv.org/abs/2305.06983) | 2023 / EMNLP 2023 | 다음 문장을 미리 생성해 보고, 확신도가 낮은 토큰이 있으면 그 문장(저확신 토큰은 마스킹)을 쿼리로 삼아 검색한다. | 2WikiMultihopQA, StrategyQA, ASQA, WikiAsp | 9절 Limitations: Wizard of Wikipedia와 ELI5에서는 유의미한 이득이 없었다. 생성과 검색을 교차하면 LM을 여러 번 활성화해야 해서 오버헤드와 생성 비용이 늘어난다. |
| [DRAGIN: Dynamic Retrieval Augmented Generation based on the Information Needs of Large Language Models](https://arxiv.org/abs/2403.10081) | arXiv 2024-03 / venue는 본문·comment 근거 없음(타 논문 인용: ACL 2024) | RIND(토큰 entropy × 최대 attention × stopword 기반 의미성)로 검색 시점을 정한다. QFS(전체 문맥 self-attention)로 쿼리를 만든다. | 2WikiMultihopQA, HotpotQA, StrategyQA, IIRC | 7절 Limitations: self-attention 점수를 제공하지 않는 API 모델에는 적용할 수 없다. |
| [SeaKR: Self-aware Knowledge Retrieval for Adaptive Retrieval Augmented Generation](https://arxiv.org/abs/2406.19215) | arXiv 2024-06 / venue는 본문·comment 근거 없음(타 논문 인용: ACL 2025) | 내부 상태로 계산한 self-aware 불확실성으로 검색을 트리거한다. 불확실성을 가장 많이 줄이는 스니펫 순으로 재정렬한다. | 복합 QA: 2WikiMultiHopQA, HotpotQA, IIRC. 단순 QA: NQ, TriviaQA, SQuAD | Limitations: (1) 내부 상태가 필요해 오픈 모델에서만 쓸 수 있다. (2) 단답 QA 위주라 long-form 등은 평가하지 않았다. (3) Gram determinant를 계산하려면 pseudo-generation이 20회 필요하다. |
| [Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity](https://arxiv.org/abs/2403.14403) | 2024 / NAACL 2024 | 소형 LM 쿼리 복잡도 분류기로 no-retrieval, single-step, multi-step 경로 중 하나를 고른다. | SQuAD, NQ, TriviaQA, MuSiQue, HotpotQA, 2WikiMultiHopQA | Limitations: 모델 예측 결과와 데이터셋 편향으로 자동 라벨링하므로 라벨이 틀릴 수 있다. ideal 분류기와의 격차가 있다. 3단계보다 세분된 복잡도는 future work로 남겼다. |
| [Adaptive Retrieval Without Self-Knowledge? Bringing Uncertainty Back Home](https://arxiv.org/abs/2501.12835) | arXiv 2025-01 / venue는 본문·comment 근거 없음(타 논문 인용: ACL 2025) | adaptive retrieval 시스템 35개(최신 방법 8개 + UE 방법 27개)를 QA 데이터셋 6개와 지표 10개로 비교했다. 기존 UE가 QA 성능은 비슷하면서 효율과 self-knowledge에서 우세했다. 비교한 최신 방법은 IRCoT, AdaptiveRAG, DRAGIN, FLARE, Rowen×3, SeaKR이다. | SQuAD v1.1, NQ, TriviaQA, MuSiQue, HotpotQA, 2WikiMultiHopQA | Limitations: LLaMA3.1-8B-instruct 한 모델만 썼고, 표준 QA 6개만 썼다(도메인 특화 없음). (결과: oracle "Ideal"과의 격차가 크고, 예컨대 첫 데이터셋에서 Best UE 0.512 vs Ideal 0.608이다. OOD에서는 self-knowledge가 크게 떨어진다.) |
| [Chain-of-Retrieval Augmented Generation (CoRAG)](https://arxiv.org/abs/2501.14342) | 2025 / NeurIPS 2025 | rejection sampling으로 중간 검색 체인을 합성해 SFT한다(정답만 있는 기존 QA 데이터를 활용). 테스트 시 greedy, best-of-N, tree search로 컴퓨트를 조절한다. | 2WikiMultihopQA, HotpotQA, Bamboogle, MuSiQue(EM 10점 이상 향상), KILT(SOTA) | 7절 Limitations: 짧고 검증하기 쉬운 답이 있는 과제(멀티홉 QA, entity linking)만 다뤘다. long-form 생성은 신뢰할 만한 평가 지표가 없어 다루지 않았다. |
| [Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning](https://arxiv.org/abs/2503.09516) | 2025 / COLM 2025 | `<search>` 토큰으로 추론과 검색을 교차하는 RL이다. 규칙 기반 outcome reward만 쓰고 format reward는 없다. | NQ, TriviaQA, PopQA, HotpotQA, 2WikiMultiHopQA, MuSiQue, Bamboogle | Limitations 절 없음. 결론의 future work: 더 정교한 보상, 불확실성 기반 동적 검색 조정. 보상에 검색 횟수나 비용 항이 없다(3절). 향상 폭은 v5 안에서도 다르게 적혀 있다(초록 24%/20%, 1절 41%/20%). 다른 버전의 수치는 본문 확인 불가. |
| [DeepRetrieval: Hacking Real Search Engines and Retrievers with Large Language Models via Reinforcement Learning](https://arxiv.org/abs/2503.00223) | 2025 / arXiv(Preprint), venue 근거 없음 | 참조 쿼리 없이 retrieval 지표(recall, nDCG 등)를 직접 보상으로 쓰는 RL query generation이다. 문헌 검색 Recall@3K가 65.07%(기존 SOTA 24.68%), 임상시험 검색이 63.18%(32.11%)다. | PubMed, ClinicalTrials.gov(실제 검색엔진), NQ, TriviaQA, SQuAD, HotpotQA, FEVER, NFCorpus, SciFact, MS MARCO 도메인 subset, Spider, BIRD | Limitations 절 없음. MS-H subset에서는 이득이 작았다(기반 모델의 도메인 지식 부족이나 retriever 천장 때문으로 추정). 도메인 적응은 future work로 남겼다. |
| [Rethinking the Necessity of Adaptive Retrieval-Augmented Generation through the Lens of Adaptive Listwise Ranking (AdaRankLLM)](https://arxiv.org/abs/2604.15621) | 2026 / arXiv, venue 근거 없음 | zero-shot listwise prompt에 passage dropout을 더해 쿼리마다 k를 적응적으로 정한다. 2단계 progressive distillation으로 소형 모델에 이전한다. 약한 모델에서는 노이즈 필터, 강한 추론 모델에서는 효율 최적화 역할을 한다. | ASQA, QAMPARI, ELI5(ALCE), LLM 8종 | 별도 Limitations 절 없음. 결론: 이론적 Oracle과의 격차가 남아 최적 검색 문제는 미해결이고, agentic 패러다임은 future work다. (최적 고정 k가 작업·데이터셋·모델마다 크게 달라진다는 점은 이 방법의 동기다.) |
| [When to Retrieve During Reasoning: Adaptive Retrieval for Large Reasoning Models (ReaLM-Retrieve)](https://arxiv.org/abs/2604.26649) | 2026 / SIGIR 2026 | 추론 step 단위 불확실성(RSUS)과 contextual-bandit 개입 정책을 쓴다. 정책은 QueryGen과 함께 REINFORCE로 학습하며, 보상은 F1 − λ1·검색 호출 수 − λ2·latency다. | MuSiQue, HotpotQA, 2WikiMultiHopQA. 표준 RAG 대비 F1 +10.1%p, IRCoT 대비 검색 호출 −47%, MuSiQue F1 71.2%, 질문당 1.8회 | 7.5절: completion-only(API) 모델에서는 F1이 2–3% 낮다. 관련 정보가 없으면 검색이 낭비된다. 학습 예제 약 2만 개가 필요하다. 오버헤드가 8% 있다. 30K 토큰을 넘으면 KV-cache 문제가 생긴다. 검색 주입이 3회 이상이면 성능이 떨어진다. |
| [Know Before You Fetch: When Does Confidence-Based Retrieval-Budget Allocation Pay Off?](https://arxiv.org/abs/2606.29959) | 2026 / arXiv(v2 2026-11), venue 근거 없음 | reader의 closed-book confidence로 쿼리마다 passage 0, 1, 5개를 배분한다. 같은 평균 passage cap에서 최선의 단순 정책(고정 깊이 또는 고정 깊이의 랜덤 혼합)과 비교한다. 결과: cap이 0.5일 때는 7개 설정 모두에서 이긴다(+0.010~0.075). cap이 1 이상이면 주로 Wikipedia 색인에서만 이득이 있다. depth 선택(1 vs 5)에는 거의 쓸모가 없다. 보정(calibration)해도 결정은 그대로였다(5.5절). | TriviaQA, NQ, MS MARCO. Qwen3-8B/32B. passage pool과 3.5M Wikipedia 색인 | 7.3절: 단답 factoid QA와 frozen 오픈 reader만 다뤘다. long-form, 대화형, multi-step 검색은 다를 수 있다. Qwen3 한 계열, 설정마다 retriever 1개를 썼다. 6절: probe latency를 포함하면 pool 설정에서는 이득이 사라지고, Wikipedia 4개 설정 중 3개에서만 남는다. |
| [When Should Active RAG Retrieve? A Budget-Aware Evaluation of Utility, Calibration, and Cost](https://arxiv.org/abs/2607.24010) | 2026 / KDD 2026 Workshop on Evaluation and Trustworthiness of Agentic AI | active retrieval을 "budgeted marginal utility 추정"으로 정식화했다. 단순 UE와 retrieval score baseline이 학습된 router와 대등한 경우가 많았다. calibration split에서 고른 임계값이 held-out에서 목표 사용률을 지키지 못했다(HotpotQA에서 split의 40–100%가 초과). | HotpotQA distractor, 2WikiMultiHopQA, MuSiQue. retrieval harm 비율(Qwen2.5-1.5B): 2Wiki 8.8%, HotpotQA 4.0%, MuSiQue 1.5% | Limitations: long-form, multi-step retrieval, memory agent, API 규모 모델, 공식 Self-RAG/FLARE 재현은 다루지 않았다. 전체 Wikipedia 색인이 아니다. 배포 측 latency, 달러, 에너지 비용은 뺐다. |
| [When Knowledge Is Not Free: Cost-Aware Evidence Selection in Retrieval-Augmented Generation](https://arxiv.org/abs/2606.02245) | 2026 / arXiv, venue 근거 없음 | MS MARCO v2.1 evidence에 접근 비용 tier를 부여하고 예산 아래에서 evidence를 고른다. 정적 selector 중 모든 경우에 우세한 것은 없었다. agentic 설정에서는 LLM이 검색 여부, tier, 중단 시점을 정한다. | HotpotQA, NQ, TriviaQA, MedQA-US, MMLU-Med | 8절: 비용 tier는 금액, 라이선스, latency를 실측한 값이 아닌 대리값이다. 도메인 자동 분류가 틀릴 수 있다. QA 5개와 표본 subset만 평가했다. |

### A.3 반복되는 open problem / 암묵적 가정 / 벤치마크 공백
1. **"확장·재작성은 항상 이득"이라는 가정이 깨짐**: 강한 retriever와 reranker(2309.08541), 그리고 LLM이 모르는 쿼리와 모호한 쿼리(2505.12694)에서 성능이 떨어진다. 그런데 대부분의 방법 논문(HyDE, Query2doc, ThinkQE)은 쿼리별로 확장할지 말지를 정하는 게이트 없이 모든 쿼리에 일괄 적용한다. 2309.08541도 확장 여부를 정하려면 대상 도메인의 라벨 데이터가 필요하다고 밝혔다.
2. **트리거 신호의 calibration과 transfer 문제**: calibration split에서 고른 임계값이 held-out에서 목표 사용률을 지키지 못한다(2607.24010). 최적 고정 검색 깊이 k도 작업, 데이터셋, 모델마다 크게 달라진다(2604.15621). 단순 UE가 정교한 방법과 대등하다(2501.12835, 2607.24010). 따라서 신호 자체보다 "운영점 선택과 그 transfer"가 병목이다. 다만 2606.29959에 따르면 분위수 임계값 방식에서는 점수를 보정해도 결정이 바뀌지 않는다. 문제는 점수의 보정 오차보다 분포 이동 아래에서 임계값이 유지되느냐에 있다.
3. **비용 회계가 불완전함**: 검색 호출 수만 세고 트리거 계산(다중 샘플링, 내부 상태 probing)과 rewriting LLM 호출 비용은 빼는 경우가 많다. 2607.24010은 trigger 쪽 계산을 분리해 회계하지만 latency, 달러, 에너지는 제외한다. 2606.29959는 closed-book probe의 latency를 실측해 포함시켰다. 그 결과 pool 설정에서는 이득이 사라졌다. 달러와 에너지 비용을 넣은 평가는 여전히 없다.
4. **white-box 의존성**: DRAGIN(attention)과 SeaKR(hidden state)은 API 모델에 쓸 수 없다(두 논문 모두 Limitations에 명시). black-box 환경에서 쓸 수 있는 트리거는 Self-RAG류 학습 토큰이나 샘플링 기반 UE로 제한된다. ReaLM-Retrieve도 completion-only 모델에서는 F1이 2–3% 낮다.
5. **벤치마크 편향**: 대부분 Wikipedia 기반 단답 QA(NQ, TriviaQA, HotpotQA, 2Wiki, MuSiQue)에서 평가한다. long-form을 다루지 않았다고 명시한 논문이 여럿이다(2607.24010, 2606.29959, SeaKR, CoRAG). 예외는 일부뿐이다(Self-RAG의 ASQA·Biography, AdaRankLLM의 ASQA·QAMPARI·ELI5, FLARE의 ASQA·WikiAsp). 대화형, 시간 민감, 도메인 특화 코퍼스에서의 평가는 드물다(도메인 예: DeepRetrieval의 PubMed, 2606.02245의 MedQA). 쿼리 모호성 벤치(AmbigNQ/AmbigQA, AmbigDocs)는 일부 논문에서만 쓴다.
6. **학습형 검색 agent의 보상 설계**: Search-R1은 outcome reward만 쓰고 검색 비용을 제약하지 않는다. CoRAG는 RL이 아니라 rejection sampling + SFT로 학습하며, 테스트 시 체인 길이로 컴퓨트를 늘린다. 비용 항을 넣은 시도는 이제 막 시작됐다. ReaLM-Retrieve는 보상에 호출 수와 latency 벌점을 넣었다. Dynamic Search-R1(Hashemi et al., WWW 2026)은 token-cost 벌점으로 검색 깊이를 조절한다(2604.26649의 인용으로만 확인, 본문 확인 불가). 2610.10955도 step·latency 벌점을 future work로 남겼다.

### A.4 구체적 연구 공백 (under-explored)
1. **쿼리 단위 "확장 여부와 방식" 게이트를 retriever 강도에 맞춰 학습**
   - 문제: 확장이 해가 되는 조건은 이미 밝혀졌다(2309.08541, 2505.12694). 그런데 쿼리마다 {원 쿼리, HyDE, Q2D, 분해} 중 하나를 고르는 "retriever-aware expansion router"는 확인되지 않았다. 2505.12694는 LLM 지식과 쿼리 모호성을 평가해 adaptive QE에 쓰는 모듈을 future work로 제시했다.
   - 가장 가까운 작업: Adaptive-RAG는 검색 경로를 고르지만 확장 방식은 고르지 않는다. SkillQR(2610.33768)은 검색 corpus에서 재작성 skill을 자동으로 발견하고 쿼리별로 라우팅한다. 하지만 항상 원 쿼리에 확장을 붙이고 "확장하지 않음" 선택지가 없다. retriever 강도도 고려하지 않는다. LLM router(42.9 nDCG@10)와 쿼리별 최선 skill 상한(53.4) 사이에 격차도 크다.
   - 미해결 이유: router 라벨이 retriever마다 다르다. "LLM이 이 쿼리를 아는가"라는 신호(지식 경계 추정)와 연결되지 않았다.
2. **retriever·reranker 교체에 강건한 트리거 calibration (conformal 또는 risk-control 기반)**
   - 2607.24010은 임계값 transfer 실패를 보였다. 2606.29959는 분위수 임계값에서는 점수 보정이 결정을 바꾸지 않으며, 보정이 의미를 갖는 것은 확률을 고정 비용비와 비교하는 규칙뿐이라고 보였다. 2501.12835는 OOD에서 self-knowledge가 크게 떨어진다고 보고했다. 하지만 분포 이동(도메인 변경, 모델·retriever 교체) 아래에서 예산을 보장하는 방법은 없다.
   - 연구 방향: conformal risk control로 "검색 사용률 ≤ B, 정확도 손실 ≤ ε"를 held-out에서 보장하는 게이트.
3. **multi-hop 분해와 검색 트리거의 공동 최적화 및 단계별 비용 회계**
   - CoRAG와 Search-R1은 체인을 길게 늘려 성능을 얻는다. 2607.24010과 2606.29959는 multi-step 검색을 다루지 않는다고 명시했다. 가장 가까운 작업인 ReaLM-Retrieve(2604.26649)는 추론 step 단위로 검색 여부를 정하고 보상에 호출 수와 latency를 넣었다. 하지만 검색 주입이 3회 이상이면 성능이 떨어지고, 하위 질문마다 "검색 / 파라메트릭 답변 / 중단"을 고르지는 않는다. 비용-정확도 frontier 평가 프로토콜도 비어 있다.
4. **black-box LLM용 저비용 지식 경계 신호**
   - SeaKR과 DRAGIN은 white-box 전용이다. 2501.12835는 27개 UE를 비교했지만 오픈 모델 한 개만 썼고 oracle과의 격차를 보였다. ReaLM-Retrieve는 completion-only 모델에서 성능이 떨어졌다. API 모델에서 단일 호출로 얻을 수 있는 신호(logprob top-k, verbalized confidence, 짧은 probe 생성)를 검색 트리거로 체계적으로 비교하고 보정한 연구는 부족하다. 특히 probe 비용까지 포함한 평가는 2606.29959 정도뿐이다.
5. **모호한 쿼리에서 다중 해석 확장과 coverage 평가**
   - 2505.12694가 보인 "인기 해석 편향"을 해결한 방법은 없다. 해석별 확장을 만든 뒤 retrieval 단계에서 coverage를 최적화하는 방식이 필요하고, 해석별 recall을 재는 벤치마크도 필요하다. AmbigNQ와 ASQA는 생성 쪽 평가에 치우쳐 있다.

---


---

## 축 B. Reranking · Filtering · Compression

### B.1 분야 현황 요약
- **성숙한 부분**: cross-encoder와 LLM 기반 listwise·setwise reranker(RankGPT, RankZephyr, Setwise, FIRST)는 TREC DL과 BEIR에서 이미 표준이다. 효율화도 확립됐다. FIRST는 첫 토큰 logits로 순위를 매겨 latency를 50% 줄였다고 주장한다. 재현 연구(2411.05508)가 직접 측정한 이득은 21–42%였다.
- **2025년의 큰 흐름**: R1 계열 추론을 reranking에 넣은 세 계열이 BRIGHT 같은 reasoning-intensive 벤치에서 SOTA를 다퉜다. Rank1은 pointwise distillation, Rank-R1은 setwise + GRPO, ReasonRank는 listwise + 다중 관점 보상이다. 같은 시기에 "Don't Overthink"(2505.16886)가 반론을 냈다. **pointwise reranker를 Qwen2.5 ≤7B에서 같은 학습 조건으로 비교하면** 비추론 쪽이 대체로 더 낫다는 것이다. 저자들도 listwise·setwise에 같은 결론이 적용되는지는 열린 문제라고 밝혔다.
- **2026년의 이슈**: 추론 reranker의 처리량 문제가 부각됐다. TALRanker 본문은 Rank-R1-14B가 BRIGHT에서 단일 GPU로 시간당 약 30개 질의만 처리한다고 보고한다. 도구 호출형 reranker도 새 이슈다. 비용은 랭킹 방식에 따라 달라진다. ReasonRank(listwise, 7B)는 추론 체인을 윈도우당 하나만 만들기 때문에 Rank1(pointwise, 7B)보다 2–2.7배 빠르다고 보고한다.
- **compression**: 네 갈래로 분화했다.
  - 추출형: RECOMP extractive, Provence pruning
  - 생성 요약형: RECOMP abstractive
  - 토큰 삭제형: LongLLMLingua
  - soft·embedding형: xRAG, PISCO, 그리고 2026년의 SeleCom, AdaMem, DEX-Comp

  2026년 soft compression 논문들은 "압축했는데 비압축과 같거나 더 낫다"고 주장한다. DEX-Comp의 비교 대상은 튜닝하지 않은 full-context RAG다. 한편 BenchPress(2510.20797)는 단순 baseline(mean pooling, 양방향 압축 토큰)이 널리 쓰는 causal 압축 토큰 방식보다 낫다고 보고해 평가가 엄밀한지 의문을 제기했다.
- **filtering과 set selection**: "relevance ≠ utility"라는 인식이 퍼졌다. 근거는 Power of Noise, utility selector 2507.19102, LLM-specific utility 2510.11358이다. 개별 점수 대신 passage 집합을 고르는 SetR(ACL 2025)도 나왔다.

### B.2 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Is ChatGPT Good at Search? Investigating LLMs as Re-Ranking Agents (RankGPT)](https://arxiv.org/abs/2304.09542) | 2023 / EMNLP (arXiv comment) | sliding-window listwise permutation 생성. NovelEval로 데이터 오염을 점검. permutation distillation으로 만든 440M 모델이 BEIR에서 3B supervised 모델보다 우수. | TREC DL19/20, BEIR, Mr.TyDi | (Limitations 절) 분석이 독점 모델(ChatGPT/GPT-4) 중심. 성능 상한이 1단계 검색의 recall에 묶임. 초기 passage 순서에 매우 민감함. |
| [A Setwise Approach for Effective and Highly Efficient Zero-shot Ranking with LLMs](https://arxiv.org/abs/2310.09497) | 2024 / SIGIR | pointwise, pairwise, listwise의 효율과 효과 trade-off를 같은 조건에서 비교. setwise 비교 + heap/bubble sort. | TREC DL19/20, BEIR 8종 | 한계 절 없음. 비교 문서 수 c가 크면 프롬프트 길이 제한 때문에 효과가 떨어질 수 있다고 봄(§5). 더 많은 LLM에서 평가하는 일을 future work로 둠(Conclusion). |
| [RankZephyr: Effective and Robust Zero-Shot Listwise Reranking is a Breeze!](https://arxiv.org/abs/2312.02724) | 2023 / arXiv (venue 근거 없음) | 7B 오픈 모델 listwise reranker(RankGPT3.5/4 distill). | TREC DL19–22, BEIR NEWS·COVID, NovelEval | 명시된 한계 없음. |
| [FIRST: Faster Improved Listwise Reranking with Single Token Decoding](https://arxiv.org/abs/2406.15657) | 2024 / arXiv preprint (venue 근거 없음) | 첫 토큰 logits로 순위를 매겨 순열 전체 생성을 피함. learning-to-rank loss를 추가. latency 50% 감소(저자 주장). 재현 연구 2411.05508의 측정값은 21–42%. | BEIR(TREC DL 확장은 2411.05508) | (Limitations 절) GPT-4 라벨 데이터만으로 학습함. 영어만 다룸. 알파벳 식별자를 써서 윈도우가 20 이하로 제한됨. |
| [BRIGHT: A Realistic and Challenging Benchmark for Reasoning-Intensive Retrieval](https://arxiv.org/abs/2407.12883) | 2024 arXiv → ICLR 2025 | StackExchange, 코딩, 수학 등 12개 데이터셋, 1,384개 질의의 추론 집약 검색 벤치. SFR-Embedding-Mistral의 nDCG@10이 MTEB 59.0에서 BRIGHT 18.3으로 떨어짐. | 자체 | (App. G) 관련성 판단이 주관적임(다중 annotator 합의로 완화). 멀티모달은 다루지 않음. |
| [Rank1: Test-Time Compute for Reranking in Information Retrieval](https://arxiv.org/abs/2502.18418) | 2025 / COLM | R1이 MS MARCO에서 만든 추론 체인 635,264개로 pointwise 추론 reranker를 distill. 양자화해도 성능이 유지됨. | BRIGHT, NevIR, mFollowIR, DL19, BEIR | (§6) "overthinking"으로 관련성 판정이 지나치게 엄격해짐. 분류 헤드 모델보다 추론이 느림. ReasonRank 본문은 자사 7B가 Rank1-7B보다 2–2.7배 빠르다고 보고함(BRIGHT 8개 데이터셋). |
| [Rank-R1: Enhancing Reasoning in LLM-based Document Rerankers via RL](https://arxiv.org/abs/2503.06034) | 2025 / arXiv (v1 "Work in progress", venue 근거 없음) | setwise prompt + R1-Zero식 추론 + GRPO(relevance label만 사용). in-domain에서 SFT의 18% 데이터로 대등한 성능. BRIGHT에서는 14B가 RankGPT4를 넘음. | TREC DL19/20, BRIGHT | 명시된 한계 없음. 처리량 30 q/h는 TALRanker가 보고한 값. |
| [ReasonRank: Empowering Passage Ranking with Strong Reasoning Ability](https://arxiv.org/abs/2508.07050) | 2025 arXiv → ACL 2026 main (arXiv comment) | R1으로 reasoning-intensive 학습 데이터를 자동 합성. SFT 후 다중 관점 ranking reward로 RL. listwise. | BRIGHT(32B 평균 38.03, v3 Table 1), R2MED(32B 42.85), BEIR 7종 | (Limitations 절) 비추론 데이터가 없어 난이도에 따라 추론과 비추론 모드를 전환하지 못함. sliding window에 의존함. |
| [Don't "Overthink" Passage Reranking: Is Reasoning Truly Necessary?](https://arxiv.org/abs/2505.16886) | 2025 / arXiv (venue 근거 없음) | 같은 조건에서 pointwise ReasonRR(Rank1 재현)과 StandardRR을 비교. 비추론이 대체로 우수함. 추론을 끈 ReasonRR-NoReason도 ReasonRR보다 나음. 추론이 점수를 양극화해 부분 관련성 표현을 해친다는 분석. | TREC DL19–23, BRIGHT | (Limitations 절) pointwise만 다뤄 listwise·setwise 결과는 열린 문제. Qwen2.5 계열 ≤7B로 한정됨. |
| [Tool-Adaptive LLM Reranker (TALRanker)](https://arxiv.org/abs/2607.10555) | 2026 / arXiv | pointwise 점수화를 agentic MDP로 정식화. 비대칭 비용 보상 RL로 확신이 있을 때 도구(외부 검색) 호출을 생략. 4B와 8B의 처리량은 각각 977, 778 q/h. | DL19/20, BEIR 9종, BRIGHT | (Conclusion) base LLM 고유의 추론 능력, 특히 도구 사용 궤적에 강하게 의존함. |
| [Shifting from Ranking to Set Selection for RAG (SetR)](https://arxiv.org/abs/2507.06838) | 2025 / ACL (oral) | CoT로 정보 요구를 식별한 뒤 그 요구를 함께 커버하는 passage 집합을 선택. Llama-3.1-8B로 distill. | HotpotQA, 2Wiki, MuSiQue, MultiHopRAG | (Limitations 절) 1단계 검색에 의존함. 멀티홉 외 도메인(코드, 대화)은 검증하지 않음. 추론 LLM의 품질에 의존함. |
| [The Power of Noise: Redefining Retrieval for RAG Systems](https://arxiv.org/abs/2401.14887) | 2024 / SIGIR | "관련은 있어 보이지만 정답이 없는" 상위 문서가 해롭고, 적절한 위치에 넣은 무작위 문서가 정확도를 최대 35% 올린다는 반직관적 결과. | NQ-open (무작위 문서는 Reddit으로도 실험) | (Conclusion) 내부 메커니즘 규명은 추가 연구가 필요함. 정답 문자열 기반 이진 평가의 한계(§4). |
| [Distilling a Small Utility-Based Passage Selector to Enhance RAG](https://arxiv.org/abs/2507.19102) | 2025 / SIGIR-AP | Qwen3-32B의 utility 판단을 1.7B 모델로 distill. passage 수를 적응적으로 선택. 복잡한 질문(HotpotQA)에서 이득이 큼. | 학습 MS MARCO. ranking은 DL19/20과 BEIR 8종. RAG는 NQ, HotpotQA(generator: Llama-3.1-8B, Qwen2.5-7B) | 명시된 한계 없음. 저자는 두 generator에서 robust하다고 주장함. generator 종속성은 2510.11358에 근거한 우리 해석. |
| [LLM-Specific Utility for Retrieval-Augmented Generation](https://arxiv.org/abs/2510.11358) | 2025 arXiv → CIKM 2026 | utility가 LLM마다 다르다는 결과: 각 LLM은 자기에게 맞춘 utilitarian evidence에서 가장 잘하고, 다른 LLM용 evidence는 일관되게 차선임. attention 기반 utility 추정이 가장 나쁨(초기 검색보다도 나쁨). SpecUBench 공개. | SpecUBench: NQ, TriviaQA, MS MARCO-FQA, HotpotQA, FEVER, 2WikiQA × LLM 4종 | (§3.3, §6) 단일 passage(pointwise) utility만 다룸. passage 간 보완·중복, 확률적 utility는 future work. |
| [RECOMP: Improving Retrieval-Augmented LMs with Compression and Selective Augmentation](https://arxiv.org/abs/2310.04408) | 2023 / arXiv v1 (ICLR은 로컬 본문에서 venue 근거 없음) | 추출형과 생성형 compressor. 문서가 필요 없으면 빈 요약을 내는 selective augmentation. NQ에서 압축률 5%에 EM −2. | NQ, TriviaQA, HotpotQA, WikiText-103 | 한계 절 없음. HotpotQA용 생성형 compressor는 추가 연구가 필요함(§5). LM 과제에서는 다른 LM으로 transfer되지만 QA에서는 성능이 떨어짐(§6). |
| [LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression](https://arxiv.org/abs/2310.06839) | 2024 / ACL (arXiv comment) | 질문 인지형 coarse-to-fine 토큰 압축과 문서 재정렬(lost-in-the-middle 대응). NQ에서 토큰 약 4배 감소에 최대 +21.4%. 1.4–2.6배 가속. | NaturalQuestions 멀티문서, LongBench, ZeroSCROLLS, MuSiQue, LooGLE | (Limitation 절) 질문마다 다시 압축해야 해 문맥 캐싱이 불가함. 연산량이 LLMLingua의 2배. 문맥과 질문의 관계가 복잡하면 효과가 줄 수 있음. |
| [xRAG: Extreme Context Compression for RAG with One Token](https://arxiv.org/abs/2405.13792) | 2024 / NeurIPS | retriever 임베딩을 projector로 LLM 공간에 투영해 문서를 토큰 1개로 압축. retriever와 LLM은 고정. FLOPs 3.53배 감소. | NQ, TriviaQA, WebQ, HotpotQA, TruthfulQA, FactKG | (App. G) 단일 dense 벡터 검색만 다룸. 추론 과제(HotpotQA, FactKG)에서 RAG에 크게 뒤짐. Top-1 문서만 다룸. |
| [Provence: Efficient and Robust Context Pruning for RAG](https://arxiv.org/abs/2501.16214) | 2025 / ICLR (arXiv comment) | DeBERTa-v3 기반 sequence labeling pruner. 문맥에 따라 pruning 양을 정하고 reranker와 통합. | NQ, TyDi QA, PopQA, HotpotQA, BioASQ, SyllabusQA, RGB. reranking은 MS MARCO, DL19, BEIR | (Conclusion 내 Limitations) QA에만 초점. passage를 한 번에 하나씩 처리. 영어 데이터만 사용. |
| [PISCO: Pretty Simple Compression for RAG](https://arxiv.org/abs/2501.16075) | 2025 / arXiv (Findings of ACL은 로컬 본문에서 venue 근거 없음) | LoRA compressor와 decoder. 문서 기반 질문으로 sequence-level distillation(사전학습 불필요). 16배 압축에서 정확도 손실 0–3%, 5.7배 속도 향상. | ASQA, HotpotQA, NQ, TriviaQA, PopQA + 도메인 밖·다국어. backbone 3종 | 명시된 한계 없음. 문서를 오프라인에서 미리 압축해야 하고 백본마다 따로 학습해야 한다는 점은 우리 추론. |
| [Rethinking Soft Compression in RAG: A Query-Conditioned Selector Perspective (SeleCom)](https://arxiv.org/abs/2602.15856) | 2026 / WWW '26 | 압축 encoder를 query-conditioned selector로 재해석. 연산과 latency 33.8–84.6% 감소. | NQ, TriviaQA, WebQ, PopQA, HotpotQA, FactKG | 명시된 한계 없음. 쿼리 조건부라 오프라인 사전 압축의 이점이 사라진다는 점은 우리 추론. |
| [Compression Beyond the Uncompressed: A Two-Stage Training Recipe for Soft Context Compression in RAG (DEX-Comp)](https://arxiv.org/abs/2609.05152) | 2026 / arXiv (under review) | teacher(full-context)가 맞힌 질문만 distill하고, teacher가 틀린 질문에는 결과 기반 RL을 적용. 16배 압축. TTFT 4.4–23.7배 감소. 튜닝하지 않은 full-context RAG보다 평균 정확도가 높음. | NQ, TriviaQA, HotpotQA, ASQA, PopQA | 명시된 한계 없음. |
| [AdaMem: Adaptive Memory Token Allocation for Soft Compression in RAG](https://arxiv.org/abs/2609.22100) | 2026 / arXiv (v1 날짜 2026-08-12와 ID 2609 불일치) | 고정 memory-token 예산을 relevance에 따라 쿼리별로 passage에 배분하고 점수가 낮은 passage는 생략. 한 번의 패스로 memory와 relevance 점수를 함께 냄. 16배에서 OSCAR 대비 최대 +3.2점, 64배에서 평균 상대 이득 14.6%. KILT-SCR 공개. | TriviaQA, HotpotQA, NQ, PopQA, ASQA, BioASQ | 명시된 한계 없음. future work: 배분 온도를 쿼리별로 학습하는 것, KV-cache 압축 등으로 확장하는 것. |
| [No Mean Feat: Simple, Strong Baselines for Context Compression (BenchPress)](https://arxiv.org/abs/2510.20797) | 2025 / arXiv | 압축 평가 스위트. mean pooling과 양방향 압축 토큰이 causal 압축 토큰보다 우수. | BenchPress: SQuAD, NarrativeQA, HotpotQA, AdversarialQA, TriviaQA, ParaphraseRC, LongBench-E | 명시된 한계 없음. 128배에서는 모든 방법이 teacher와 지속적인 격차를 보임(§7). RAG 전용이 아니라 영어 독해 평가임. |

### B.3 반복되는 open problem / 암묵적 가정 / 벤치마크 공백
1. **"relevance ranking = RAG에 좋은 context"라는 가정**
   - 반박하는 연구: Power of Noise, 2507.19102, 2510.11358, SetR.
   - 그런데도 reranker 논문 대부분(RankGPT부터 ReasonRank까지)은 여전히 nDCG@10(TREC DL, BEIR, BRIGHT)으로만 평가하고, 다운스트림 생성 품질은 보고하지 않는다.
   - 예외: BRIGHT는 QA 성능 향상을 일부 보고한다. SciRerankBench는 생성 품질을 함께 잰다.
2. **추론 reranker의 효과-비용 trade-off가 정리되지 않음**
   - Rank1, Rank-R1, ReasonRank는 BRIGHT에서 이득을 보인다.
   - 2505.16886은 반대 결과를 낸다. 단, 이 결과는 pointwise에 한정된다. 같은 조건의 pointwise reranker(≤7B)에서는 비추론이 낫다.
   - 비용은 랭킹 방식에 따라 달라진다. listwise ReasonRank는 pointwise Rank1보다 2–2.7배 빠르다. TALRanker는 Rank-R1-14B의 처리량이 약 30 q/h라고 보고한다.
   - ReasonRank는 추론과 비추론 모드를 전환하지 못한다는 점을 스스로 한계로 밝혔다.
   - 쿼리별로 "추론이 필요한지"를 판단하는 연구는 거의 없다.
3. **reranking·filtering·compression이 분리된 파이프라인으로 설계됨**
   - 예외: Provence(rerank + 문장 pruning 통합), AdaMem(relevance 점수와 soft memory를 한 번에 내고 예산을 배분), SeaKR(불확실성 기반 재정렬. 이 배치에서는 검증하지 않음).
   - 대부분은 단계별로 독립 최적화되어 있다. 그래서 단계 사이의 오류 전파와 중복 비용이 측정되지 않는다.
4. **compression 평가의 취약성**
   - soft compression 논문은 단답·사실형 QA 위주다(NQ, TriviaQA, HotpotQA, PopQA, ASQA. xRAG·SeleCom은 FactKG 포함).
   - 압축률과 속도의 기준이 논문마다 다르다(오프라인 사전 압축 vs 쿼리 시 압축, FLOPs vs TTFT).
   - BenchPress는 단순 baseline이 더 강하다고 보고했다. AdaMem도 기존 평가가 작은 top-k 풀만 써서 reranking의 효과를 재기 어렵다고 지적한다.
   - 압축된 문맥으로 출처 인용(attribution)이 가능한지는 거의 다뤄지지 않는다.
5. **utility의 generator 종속성**
   - 2510.11358에 따르면 utility는 LLM마다 다르고 transfer되지 않는다.
   - 그런데 compressor는 대체로 특정 decoder 하나에 맞춰 학습된다. PISCO는 backbone마다 별도 모델을 쓰고, xRAG는 backbone마다 projector를 학습한다.
   - RECOMP의 텍스트 요약은 LM 과제에서는 다른 LM으로 transfer됐지만 QA에서는 성능이 떨어졌다.
   - 2507.19102는 두 generator에서 robust하다고 주장해 2510.11358의 결과와 긴장 관계에 있다. 두 논문은 저자진이 같다.
6. **집합 수준 목표의 부재**
   - SetR을 빼면 대부분 점수를 독립적으로 매긴 뒤 top-k를 자른다.
   - 2510.11358도 passage 간 보완·중복을 다루지 못한 것을 한계로 밝혔다.
   - 중복, 상호보완성, 충돌(서로 모순되는 증거)을 고려한 선택 방법과 이를 재는 지표(정보 요구 coverage)는 표준화되어 있지 않다.

### B.4 구체적 연구 공백 (under-explored)
1. **쿼리 적응형 "추론 예산" reranking**
   - 문제: 쿼리마다 비추론 점수, 짧은 추론, 긴 추론, 도구 호출 중 하나를 고르는 라우팅.
   - 가장 가까운 작업: TALRanker는 도구 호출 여부만 적응적으로 정한다. 2505.16886은 pointwise에 대해 "평균적으로 추론이 필요 없다"는 결론만 낸다. ReasonRank는 모드 전환을 future work로 남겼다.
   - 미해결 이유: 추론이 도움이 되는 쿼리 유형을 판별하는 신호가 없다. BRIGHT와 DL이 섞인 트래픽에서 비용-효과 frontier를 측정한 연구도 없다.
2. **generator-agnostic utility와 집합 선택**
   - 문제: 2510.11358은 utility가 LLM마다 다르다는 것을 보였다. 2507.19102는 단일 teacher(Qwen3-32B)의 판단으로 distill한다(평가는 generator 2종). SetR은 coverage를 쓰지만 선택 단계에서 8B LLM이 CoT를 생성해야 한다.
   - 비어 있는 부분: 여러 generator에 걸쳐 transfer되는 utility 표현, 또는 generator 특성을 조건으로 받는 소형 selector. 그리고 이를 집합 수준 목표(coverage − 중복 − 충돌)와 결합한 경량 모델.
3. **reranking과 compression의 단일 패스 통합 + budget 제약 최적화** (공백 축소)
   - 이미 나온 것: Provence는 문장 pruning과 reranking을 통합했다. AdaMem은 고정 memory-token 예산을 relevance 점수에 따라 passage별로 배분하고 점수가 낮은 passage는 생략한다. relevance 점수와 압축 memory도 한 번의 패스로 낸다. 그래서 "총 예산 안에서 어떤 passage를 얼마나 압축해 넣을지"는 soft compression 안에서 이미 다뤄졌다.
   - 남은 공백 (1) 배분 방식: AdaMem의 배분은 압축률별로 온도를 정해 두는 결정적 규칙이다. 쿼리별로 학습하는 배분은 저자들도 future work로 남겼다.
   - 남은 공백 (2) 압축 방식의 혼합: 이질적인 압축 수단(hard pruning, 요약, soft memory, 생략)을 함께 고르는 knapsack형 최적화는 확인되지 않았다.
   - 남은 공백 (3) 집합 효과: passage 사이의 중복과 보완을 반영한 배분도 확인되지 않았다.
   - AdaRankLLM(2604.15621)의 결과와 연결할 수 있다. passage dropout 기반 적응형 listwise ranking이 고정 깊이 검색보다 대체로 나았다.
4. **압축과 pruning 이후의 attribution·faithfulness 평가**
   - xRAG, PISCO, DEX-Comp는 이번에 확인한 범위에서 정답 정확도 계열 지표(EM, match, accuracy)만 보고한다.
   - 측정되지 않은 것: 압축 후 답변이 어느 원문 문장에 근거하는지 추적할 수 있는지. 압축이 distractor의 영향을 줄이는지 키우는지(Power of Noise 관점).
   - 출처 인용이 필요한 enterprise RAG 환경에서 중요한 공백이다.
5. **"downstream-aware" reranker 벤치마크**
   - 현재 reranker 벤치(BRIGHT, BEIR)는 nDCG만 잰다.
   - SciRerankBench(2508.08742)는 reranker를 RAG와 연결해 평가한다. Recall@k, 답변 품질, 실행 시간을 재고, 잡음 문맥(NC)·의미는 비슷하지만 논리적으로 무관한 문맥(SSLI)·반사실 문맥(CC) 같은 distractor 유형별로도 잰다. 다만 과학 5개 분야에 한정되고 오픈소스 LLM만 다룬다.
   - AdaMem의 KILT-SCR은 쿼리당 25개 passage 고정 풀(KILT-100w)과 6개 QA 평가 세트(총 약 1.2만 개)로 이루어진다. reranking과 압축을 함께 연구하기 위한 자원이다. 그러나 passage별 relevance qrels가 없어 nDCG를 재지 않는다.
   - 비어 있는 부분: 범용 도메인에서 같은 후보 풀로 nDCG, 생성 정확도, 토큰 비용, distractor 민감도를 동시에 재는 공개 벤치.

---

## 교차 축 관찰
- **두 축의 공통 병목은 "운영점(budget)과 calibration"이다.** query 측은 검색할지, 확장할지를 정한다. rerank 측은 추론할지, 몇 개 넣을지, 얼마나 압축할지를 정한다. 두 쪽이 각각 따로 예산을 쓰고, 이를 함께 최적화하거나 평가한 연구는 확인되지 않았다. 2026년 budget-aware 평가 논문(2607.24010, 2606.29959, 2606.02245; 이 배치에서는 검증하지 않음)은 모두 검색 트리거 하나에만 집중한다. rerank 측 안에서는 AdaMem이 선택과 압축 예산을 하나로 묶었지만, query 측 예산과는 분리되어 있다.
- **SeaKR(불확실성 기반 재정렬)과 AdaRankLLM(listwise passage dropout을 adaptive retrieval 판단에 사용)은 두 축을 잇는 드문 사례다.** "trigger와 selection을 하나의 정책으로 통합"하는 방향의 근거가 된다. SeaKR은 이 배치에서 검증하지 않았다.
