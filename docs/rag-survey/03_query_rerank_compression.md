# Survey C — RAG 검색 측면: (A) Query-side 처리, (B) Reranking·Filtering·Compression

작성일: 2026-11-19

**검증 방법 메모**: 이 세션에서는 arxiv.org 직접 fetch가 네트워크 정책으로 막혀 있었다(ENOTFOUND). 그래서 2024–2026 논문은 WebSearch 결과(arXiv abs/pdf/html 색인, ACL Anthology, ICLR 가상 페이지, alphaXiv 미러)로 제목·저자·arXiv ID·초록 수준 주장을 교차 확인했다. 2023–2024 고전 논문(HyDE, Self-RAG, RankGPT 등)은 널리 알려진 arXiv ID를 적었다. 이번 세션에서 그 ID를 다시 열어 보지는 못했으므로 "ID 재확인 권장"으로 표시했다. 2차 요약에서만 본 수치는 **미확인**으로 표시했다.

---

## 축 A. Query-side processing (rewriting / expansion / decomposition / routing / when-to-retrieve)

### A.1 분야 현황 요약
- **성숙한 부분**: 생성형 query expansion은 sparse·약한 retriever에서 꾸준히 이득을 준다. HyDE, Query2doc, 그리고 RL로 학습하는 rewriter(Rewrite-Retrieve-Read)가 대표적이다. 다만 2023–2025 분석 연구들은 실패 조건을 분명히 밝혔다. 강한 retriever에서는 오히려 성능이 떨어지고, LLM이 모르는 쿼리나 모호한 쿼리에서도 성능이 떨어진다.
- **when-to-retrieve 계보**: Self-RAG(reflection token), FLARE(다음 문장 확신도), DRAGIN(토큰 중요도·불확실성), SeaKR(내부 상태 불확실성)로 이어진다. 2025년 대규모 비교(Moskvoretskii et al.)는 단순 불확실성 추정치가 복잡한 파이프라인과 대등하거나 더 효율적이라고 보고했다. 이 결과로 정교한 트리거 설계의 가치에 의문이 제기됐다.
- **2025–2026의 흐름**: 첫째, Search-R1, DeepRetrieval, CoRAG처럼 쿼리 생성·분해·검색 시점을 RL이나 rejection sampling으로 end-to-end 학습하는 agentic 검색이 늘었다. 둘째, 2026년에는 "예산(budget)·비용을 고정한 상태에서 평가해야 한다"는 메타 평가 논문들이 나왔다(2607.24010, 2606.29959, 2606.02245). 셋째, reasoning 모델에서는 adaptive retrieval의 역할이 노이즈 필터에서 효율 최적화로 바뀐다는 재검토가 나왔다(2604.15621).
- **아직 움직이는 부분**: 검색 트리거의 calibration과 transfer, 대형 추론 모델(LRM)의 추론 도중 검색(2604.26649), 다국어·대화형 rewriting의 RL 학습(2610.04899, 2610.10955).

### A.2 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Precise Zero-Shot Dense Retrieval without Relevance Labels (HyDE)](https://arxiv.org/abs/2212.10496) | 2023 / ACL (ID 재확인 권장) | LLM이 가상 답변 문서를 생성하고, 그 임베딩으로 검색한다. | TREC DL19/20, BEIR 일부, Mr.TyDi | 생성 문서의 hallucination에 의존함. LLM 지식 밖 쿼리에서 취약(이후 2505.12694가 실증). |
| [Query2doc: Query Expansion with Large Language Models](https://arxiv.org/abs/2303.07678) | 2023 / EMNLP (ID 재확인 권장) | few-shot으로 pseudo-doc을 생성한 뒤 쿼리에 이어 붙인다(BM25와 dense 모두 적용). | MS MARCO, TREC DL | LLM 호출 latency. 강한 dense retriever에서는 이득이 줄어듦. |
| [Query Rewriting for Retrieval-Augmented LLMs (Rewrite-Retrieve-Read)](https://arxiv.org/abs/2305.14283) | 2023 / EMNLP (ID 재확인 권장) | 소형 rewriter를 reader 피드백 기반 RL(PPO)로 학습한다. | HotpotQA, AmbigNQ, PopQA, MMLU | 보상이 reader에 종속되어 reader를 바꾸면 재학습이 필요함. |
| [When do Generative Query and Document Expansions Fail?](https://arxiv.org/abs/2309.08541) | 2023→EACL 2024 Findings (venue 미확인) | expansion 기법 11종 × 데이터셋 12개 × retriever 24개를 체계적으로 비교. 약한 ranker에는 도움이 되고 강한 ranker에는 해가 된다는 결론. | BEIR 등 12개 | 원인은 분석했지만 "언제 확장할지"를 정하는 판정기는 제시하지 않음. |
| [LLM-based Query Expansion Fails for Unfamiliar and Ambiguous Queries](https://arxiv.org/abs/2505.12694) | 2025 / SIGIR short | 실패 조건 두 가지를 통제 실험으로 확인: LLM 지식 부족(존재하지 않는 엔티티 생성), 쿼리 모호성(인기 해석 쪽으로 편향). | sparse/dense retriever, 다수 데이터셋 | 분석 논문이라 해법은 제시하지 않음. |
| [ThinkQE: Query Expansion via an Evolving Thinking Process](https://arxiv.org/abs/2506.09260) | 2025 / arXiv | 학습 없이 thinking 기반 확장과 corpus 피드백(top-5)을 반복한다. | DL19/20, BRIGHT StackExchange(평균 nDCG@10 34.9, 논문 표 기준) | 여러 라운드의 LLM 호출로 비용이 큼. reranker(Rank-K-32B 37.9)보다는 낮음. |
| [Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection](https://arxiv.org/abs/2310.11511) | 2024 / ICLR (ID 재확인 권장) | reflection token으로 검색 여부와 근거성을 스스로 판단한다. | PopQA, TriviaQA, PubHealth, ARC, ASQA 등 | critic 라벨링 비용이 큼. 이후 재현 연구들에서 효율 대비 이득이 제한적이라는 지적(2501.12835). |
| [Active Retrieval Augmented Generation (FLARE)](https://arxiv.org/abs/2305.06983) | 2023 / EMNLP (ID 재확인 권장) | 다음 문장의 토큰 확신도가 낮으면 그 문장을 쿼리로 삼아 검색한다. | 2Wiki, StrategyQA, ASQA, WikiAsp | 임계값이 수작업. 반복 생성 비용. |
| [DRAGIN: Dynamic RAG based on the Real-time Information Needs of LLMs](https://arxiv.org/abs/2403.10081) | 2024 / ACL | RIND(불확실성×토큰 중요도×의미성)로 검색 시점을 정하고, QFS(전체 문맥 attention)로 쿼리를 만든다. | 2Wiki, HotpotQA, StrategyQA, IIRC | attention 접근이 필요해 white-box 모델에서만 쓸 수 있음. |
| [SeaKR: Self-aware Knowledge Retrieval for Adaptive RAG](https://arxiv.org/abs/2406.19215) | 2024 arXiv → ACL 2025 | 내부 상태 기반 self-aware 불확실성으로 검색을 트리거한다. 불확실성을 가장 많이 줄이는 스니펫 순으로 재정렬한다. | 복합·단순 QA(2Wiki, HotpotQA, IIRC, NQ 등; 세부 미확인) | 내부 상태 접근이 필요하고, 여러 번 샘플링하는 비용이 있음. |
| [Adaptive-RAG: Learning to Adapt RAG through Question Complexity](https://arxiv.org/abs/2403.14403) | 2024 / NAACL (ID 재확인 권장) | 쿼리 복잡도 분류기로 no-retrieval, single-step, multi-step 경로 중 하나를 고른다. | SQuAD, NQ, TriviaQA, MuSiQue, HotpotQA, 2Wiki | 분류기 라벨이 silver label(모델 성공 여부)에 의존함. 3단계로 거칠게 나눔. |
| [Adaptive Retrieval Without Self-Knowledge? Bringing Uncertainty Back Home](https://arxiv.org/abs/2501.12835) | 2025 / ACL | adaptive retrieval 시스템 35개(최신 방법 8개 + UE 방법 27개)를 QA 데이터셋 6개에서 비교. 단순 UE가 효율과 self-knowledge 측면에서 우세. | 6 QA(단일·멀티홉) | oracle과의 격차가 큼. 지식 경계 식별은 여전히 미해결. |
| [Chain-of-Retrieval Augmented Generation (CoRAG)](https://arxiv.org/abs/2501.14342) | 2025 / arXiv (v3 2025-10) | rejection sampling으로 중간 검색 체인을 합성하고 SFT한다. 테스트 시 greedy, best-of-N, tree search로 컴퓨트를 조절. | 멀티홉 QA(EM 10점 이상 향상 주장), KILT SOTA | 테스트 타임 컴퓨트 비용. 체인 합성이 기존 QA 정답에 의존함. |
| [Search-R1: Training LLMs to Reason and Leverage Search Engines with RL](https://arxiv.org/abs/2503.09516) | 2025 / arXiv (venue 미확인) | `<search>` 토큰으로 추론과 검색을 교차하는 RL(outcome reward). | NQ, TriviaQA, PopQA, HotpotQA, 2Wiki, MuSiQue, Bamboogle | 버전에 따라 수치가 다름(v2 26%, v3 41%). 검색 횟수나 비용을 제약하지 않음. |
| [DeepRetrieval: Hacking Real Search Engines and Retrievers with LLMs via RL](https://arxiv.org/abs/2503.00223) | 2025 / arXiv | retrieval 지표(recall 등)를 직접 보상으로 쓰는 RL query rewriting. | 문헌 검색, BEIR 등(세부 미확인) | 세부 미확인. |
| [Rethinking the Necessity of Adaptive RAG through the Lens of Adaptive Listwise Ranking (AdaRankLLM)](https://arxiv.org/abs/2604.15621) | 2026 / arXiv | listwise prompt에 passage dropout을 더해 k를 적응적으로 정한다. 약한 모델에서는 노이즈 필터, 강한 추론 모델에서는 효율 최적화 역할. | 세부 미확인 | 최적 k가 작업과 모델에 따라 크게 달라 동적 보정이 필요함. |
| [When to Retrieve During Reasoning: Adaptive Retrieval for Large Reasoning Models (ReaLM-Retrieve)](https://arxiv.org/abs/2604.26649) | 2026 / arXiv | LRM의 추론 도중 검색 시점을 REINFORCE로 학습한다(보상 = 정확도 − 호출 수·latency 비용). | 미확인 | 미확인. |
| [Know Before You Fetch: Calibrated Retrieval-Budget Allocation for RAG](https://arxiv.org/abs/2606.29959) | 2026 / arXiv | 쿼리마다 closed-book, k=1, k=5, abstain 중 하나를 고른다. 불확실성을 정답 확률로 보정해 사용. | 미확인 | retrieval-call AUC는 binary gating과 비슷함. gating이 Qwen3-8B에서는 latency를 늘림(약 27%, 2차 요약 기준 **미확인**). |
| [When Should Active RAG Retrieve? A Budget-Aware Evaluation of Utility, Calibration, and Cost](https://arxiv.org/abs/2607.24010) | 2026 / arXiv (CMU) | active retrieval을 "budgeted marginal utility 추정"으로 정식화. 단순 UE·retrieval score baseline이 학습된 router와 대등함. 임계값이 held-out 데이터로 transfer되지 않는 문제를 보임. | 2Wiki 등(검색 시 오답이 되는 비율 8.8%는 **미확인**) | 저자 명시: long-form, multi-step retrieval, 공식 Self-RAG/FLARE 재현은 다루지 않음. latency, 달러, 에너지 비용은 제외. |
| [When Knowledge Is Not Free: Cost-Aware Evidence Selection in RAG](https://arxiv.org/abs/2606.02245) | 2026 / arXiv | evidence에 접근 비용 tier를 부여한다. LLM이 검색 여부, tier, 중단 시점을 결정. | 미확인 | 모든 경우에 우세한 고정 selector는 없음. |

### A.3 반복되는 open problem / 암묵적 가정 / 벤치마크 공백
1. **"확장·재작성은 항상 이득"이라는 가정이 깨짐**: 강한 retriever(2309.08541), 모르는 쿼리와 모호한 쿼리(2505.12694)에서 성능이 떨어진다. 그런데 대부분의 방법 논문(HyDE, Query2doc, ThinkQE)은 쿼리별로 확장을 할지 말지 정하는 게이트 없이 모든 쿼리에 일괄 적용한다.
2. **트리거 신호의 calibration과 transfer 문제**: 임계값이 데이터셋과 모델에 따라 달라진다(2607.24010, 2604.15621). 단순 UE가 정교한 방법과 대등하다(2501.12835). 신호 자체보다 "보정과 운영점 선택"이 병목이다.
3. **비용 회계가 불완전함**: 검색 호출 수만 세고 트리거 계산(다중 샘플링, 내부 상태 probing)과 rewriting LLM 호출 비용은 빼는 경우가 많다. 2607.24010과 2606.29959가 이 문제를 지적했지만, 둘 다 latency, 달러, 에너지는 제외한다.
4. **white-box 의존성**: DRAGIN(attention)과 SeaKR(hidden state)는 API 모델에 쓸 수 없다. black-box 환경에서 쓸 수 있는 트리거는 Self-RAG류 학습 토큰이나 샘플링 기반 UE로 제한된다.
5. **벤치마크 편향**: 대부분 Wikipedia 기반 단답 QA(NQ, TriviaQA, HotpotQA, 2Wiki, MuSiQue)에서 평가한다. long-form, 대화형, 시간 민감, 도메인 특화 코퍼스에서의 평가는 드물다(2607.24010이 long-form 부재를 명시). 쿼리 모호성 벤치(AmbigNQ)도 일부 논문에서만 쓴다.
6. **RL 기반 검색 agent의 보상 설계**: Search-R1과 CoRAG는 outcome reward를 쓰고 검색 비용을 제약하지 않는다. 비용 항을 넣은 시도(ReaLM-Retrieve, Dynamic Search-R1 — 후자는 **미확인**)는 이제 막 시작됐다.

### A.4 구체적 연구 공백 (under-explored)
1. **쿼리 단위 "확장 여부와 방식" 게이트를 retriever 강도에 맞춰 학습**
   - 문제: 확장이 해가 되는 조건은 이미 밝혀졌다(2309.08541, 2505.12694). 그런데 쿼리마다 {원 쿼리, HyDE, Q2D, 분해} 중 하나를 고르는 "retriever-aware expansion router"는 확인되지 않았다.
   - 가장 가까운 작업: Adaptive-RAG는 경로를 고르지만 확장 방식은 고르지 않는다. 2026년 SkillQR(arXiv 2610.33768, 세부 **미확인**)은 재작성 전략으로 라우팅한다.
   - 미해결 이유: router 라벨이 retriever마다 다르다. "LLM이 이 쿼리를 아는가"라는 신호(지식 경계 추정)와 연결되지 않았다.
2. **retriever·reranker 교체에 강건한 트리거 calibration (conformal 또는 risk-control 기반)**
   - 2607.24010은 임계값 transfer 실패를 보였고, 2606.29959는 보정된 확률을 인터페이스로 제안했다. 하지만 분포 이동(도메인, 모델 교체) 아래에서 예산 보장을 하는 방법은 없다.
   - 연구 방향: conformal risk control로 "검색 사용률 ≤ B, 정확도 손실 ≤ ε"를 보장하는 게이트.
3. **multi-hop 분해와 검색 트리거의 공동 최적화 및 단계별 비용 회계**
   - CoRAG와 Search-R1은 체인을 길게 늘려 성능을 얻는다. 2607.24010은 multi-step을 다루지 않는다고 명시했다. 하위 질문마다 "검색할지, 파라메트릭 지식으로 답할지, 중단할지"를 정하는 budget-aware 정책과 그 평가 프로토콜(비용-정확도 frontier)이 비어 있다.
4. **black-box LLM용 저비용 지식 경계 신호**
   - SeaKR과 DRAGIN은 white-box 전용이다. 2501.12835도 oracle과의 격차를 보였다. API 모델에서 단일 호출로 얻을 수 있는 신호(logprob top-k, verbalized confidence, 짧은 probe 생성)를 검색 트리거로 체계적으로 비교하고 보정한 연구는 부족하다(비용 포함 평가가 거의 없음).
5. **모호한 쿼리에서 다중 해석 확장과 coverage 평가**
   - 2505.12694가 보인 "인기 해석 편향"을 해결하는 방법이 없다. 해석별 확장을 만든 뒤 retrieval 단계에서 coverage를 최적화하는 방식, 그리고 해석별 recall을 재는 벤치마크가 필요하다. AmbigNQ와 ASQA는 생성 쪽 평가에 치우쳐 있다.

---

## 축 B. Reranking · Filtering · Compression

### B.1 분야 현황 요약
- **성숙한 부분**: cross-encoder와 LLM 기반 listwise·setwise reranker(RankGPT, RankZephyr, Setwise, FIRST)는 MS MARCO, TREC DL, BEIR에서 이미 표준이다. FIRST처럼 단일 토큰 디코딩으로 latency를 줄이는 효율화도 확립됐다.
- **2025년의 큰 흐름**: R1 계열 추론을 reranking에 넣은 Rank1(distillation), Rank-R1(GRPO, setwise), ReasonRank(listwise + 다중 관점 보상)가 BRIGHT 같은 reasoning-intensive 벤치에서 SOTA를 경쟁했다. 동시에 "Don't Overthink"(2505.16886)는 같은 학습 조건에서는 비추론 reranker가 대체로 더 낫다는 반론을 냈다. 2026년에는 추론 reranker의 처리량 문제(TALRanker가 보고한 "Rank-R1-14B 약 30 q/h"; 수치는 해당 논문 주장)와 도구 호출형 reranker가 이슈다.
- **compression**: 추출형(RECOMP extractive, Provence pruning), 생성 요약형(RECOMP abstractive), 토큰 삭제형(LongLLMLingua), soft·embedding형(xRAG, PISCO, 2026년 SeleCom, AdaMem, DEX-Comp)으로 분화했다. 2026년 soft compression 논문들은 "압축했는데 비압축보다 같거나 더 낫다"고 주장한다. 하지만 BenchPress(2510.20797)는 단순 baseline(mean pooling)이 널리 쓰는 압축 토큰 방식보다 낫다고 보고해 평가의 엄밀성에 의문을 제기했다.
- **filtering과 set selection**: "relevance ≠ utility"라는 인식이 퍼졌다(Power of Noise, utility selector 2507.19102, LLM-specific utility 2510.11358). 개별 점수 대신 집합을 선택하는 SetR(ACL 2025)이 나왔다.

### B.2 논문 표

| Title | Year/Venue | Core contribution | Benchmarks | Stated limitations |
|---|---|---|---|---|
| [Is ChatGPT Good at Search? Investigating LLMs as Re-Ranking Agents (RankGPT)](https://arxiv.org/abs/2304.09542) | 2023 / EMNLP (ID 재확인 권장) | sliding-window listwise permutation 생성. NovelEval로 데이터 오염을 점검. | TREC DL19/20, BEIR, Mr.TyDi | 독점 API 비용과 위치 편향. 윈도우 순차 호출로 latency가 큼. |
| [A Setwise Approach for Effective and Highly Efficient Zero-shot Ranking with LLMs](https://arxiv.org/abs/2310.09497) | 2024 / SIGIR (ID 재확인 권장) | pointwise, pairwise, listwise의 효율과 효과 trade-off를 분석. setwise 비교 + heap/bubble sort. | TREC DL, BEIR | 정렬 알고리즘에 따라 추론 횟수가 달라짐. |
| [RankZephyr: Effective and Robust Zero-Shot Listwise Reranking is a Breeze!](https://arxiv.org/abs/2312.02724) | 2023 / arXiv (ID 재확인 권장) | 7B 오픈 모델 listwise reranker(GPT-4 distill). | TREC DL19–23, BEIR 일부 | teacher(GPT-4)에 의존. 윈도우 길이 제약. |
| [FIRST: Faster Improved Listwise Reranking with Single Token Decoding](https://arxiv.org/abs/2406.15657) | 2024 / EMNLP | 첫 토큰 logits로 순위를 매겨 순열 전체 생성을 피함. latency 약 50% 감소(저자 주장). | BEIR, TREC DL | 첫 토큰 순위와 전체 순열이 일치하지 않을 수 있음. 재현 연구 2411.05508이 개선점을 보고. |
| [BRIGHT: A Realistic and Challenging Benchmark for Reasoning-Intensive Retrieval](https://arxiv.org/abs/2407.12883) | 2024 arXiv → ICLR 2025 (venue 재확인 권장) | StackExchange, 코딩, 수학 등 추론 집약 검색 벤치. | 자체 | 문서 수와 도메인 범위. RAG 생성 품질과의 연결은 약함. |
| [Rank1: Test-Time Compute for Reranking in Information Retrieval](https://arxiv.org/abs/2502.18418) | 2025 / COLM | R1의 MS MARCO 추론 체인 63.5만 개로 pointwise 추론 reranker를 distill. 양자화해도 성능 유지. | BRIGHT, 지시 따르기 검색(FollowIR 등), BEIR | pointwise 추론으로 latency가 큼(ReasonRank는 자사 7B가 Rank1-7B보다 2–2.7배 빠르다고 주장, alphaXiv 요약 기준 **미확인**). |
| [Rank-R1: Enhancing Reasoning in LLM-based Document Rerankers via RL](https://arxiv.org/abs/2503.06034) | 2025 arXiv → SIGIR 2026 | setwise prompt + R1-Zero식 추론 + GRPO(relevance label만 사용). SFT 대비 18% 데이터로 비슷한 성능(2차 출처). | TREC DL19/20, BRIGHT | 추론 길이로 처리량이 낮음. in-domain 이득은 제한적. |
| [ReasonRank: Empowering Passage Ranking with Strong Reasoning Ability](https://arxiv.org/abs/2508.07050) | 2025 / arXiv | R1으로 reasoning-intensive 학습 데이터를 자동 합성. SFT 후 다중 관점 ranking reward로 RL. listwise. | BRIGHT(32B 40.6, v2 기준), R2MED | listwise 윈도우 제약. 합성 데이터가 teacher에 의존. |
| [Don't "Overthink" Passage Reranking: Is Reasoning Truly Necessary?](https://arxiv.org/abs/2505.16886) | 2025 / arXiv (venue 미확인) | 같은 조건에서 ReasonRR과 StandardRR을 비교. 비추론 reranker가 대체로 더 우수. 추론이 점수를 극단값으로 몰아 부분 관련성 표현을 해친다는 분석. | TREC DL, BEIR, BRIGHT(세부 미확인) | 특정 모델 크기와 학습 설정에 한정. |
| [Tool-Adaptive LLM Reranker (TALRanker)](https://arxiv.org/abs/2607.10555) | 2026 / arXiv | pointwise 점수화를 MDP로 정식화. 비대칭 비용 보상 RL로 확신이 있을 때 도구(웹 검색) 호출을 생략. | DL19/20, BEIR 9종, reasoning 벤치 | v1 preprint. 처리량 수치는 2차 요약 기준 **미확인**. |
| [Shifting from Ranking to Set Selection for RAG (SetR)](https://arxiv.org/abs/2507.06838) | 2025 / ACL (oral) | CoT로 정보 요구를 식별한 뒤 그 요구를 함께 커버하는 passage 집합을 선택. | HotpotQA, 2Wiki, MuSiQue, MultiHopRAG | 멀티홉 중심. 선택 LLM 비용. |
| [The Power of Noise: Redefining Retrieval for RAG Systems](https://arxiv.org/abs/2401.14887) | 2024 / SIGIR | "관련은 있어 보이지만 정답이 없는" distractor가 가장 해롭고, 무작위 문서는 정확도를 최대 35% 올린다는 반직관적 결과. | NQ-open 기반 | 메커니즘 설명이 부족함. 모델과 설정 의존성이 큼. |
| [Distilling a Small Utility-Based Passage Selector to Enhance RAG](https://arxiv.org/abs/2507.19102) | 2025 / SIGIR-AP | Qwen3-32B의 utility 판단을 1.7B 모델로 distill. passage 수를 적응적으로 선택. 멀티홉에서 이득이 큼. | MS MARCO, HotpotQA 등(세부 미확인) | utility가 generator에 종속됨(2510.11358). |
| [LLM-Specific Utility: A New Perspective for RAG](https://arxiv.org/abs/2510.11358) | 2025 / arXiv | utility가 LLM마다 다르다는 결과: 한 모델에 유용한 passage가 다른 모델에는 유용하지 않음. attention 기반 utility 추정이 가장 나쁨. | 미확인(2차 요약) | 미확인. |
| [RECOMP: Improving RAG with Compression and Selective Augmentation](https://arxiv.org/abs/2310.04408) | 2024 / ICLR (ID 재확인 권장) | 추출형과 생성형 compressor. 문서가 필요 없으면 빈 요약을 내는 selective augmentation. | NQ, TriviaQA, HotpotQA, WikiText | compressor가 target LM에 맞춰 학습됨. 멀티홉에서 정보 손실. |
| [LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression](https://arxiv.org/abs/2310.06839) | 2024 / ACL (ID 재확인 권장) | 질문 인지형 coarse-to-fine 토큰 압축과 문서 재정렬(lost-in-the-middle 대응). | NaturalQuestions 멀티문서, LongBench, ZeroSCROLLS | 소형 LM perplexity에 의존. 높은 압축률에서 가독성과 사실성 저하. |
| [xRAG: Extreme Context Compression for RAG with One Token](https://arxiv.org/abs/2405.13792) | 2024 / NeurIPS (ID 재확인 권장) | retriever 임베딩을 projector로 LLM 공간에 투영해 문서를 토큰 1개로 압축. | NQ, TriviaQA, WebQ, HotpotQA 등 | 멀티홉과 세부 사실에서 성능 하락. |
| [Provence: Efficient and Robust Context Pruning for RAG](https://arxiv.org/abs/2501.16214) | 2025 / ICLR | DeBERTa 기반 sequence labeling pruner. 문맥에 따라 pruning 양을 정하고 reranker와 통합. | 다도메인 QA(세부 미확인) | 문장 단위 pruning이라 문장 간 의존성 손실 가능. |
| [PISCO: Pretty Simple Compression for RAG](https://arxiv.org/abs/2501.16075) | 2025 / Findings of ACL | LoRA compressor와 decoder. 문서 기반 질문으로 distillation. 16배 압축에서 정확도 손실 0–3%, 5.7배 속도 향상. | RAG QA 다수 | 문서 오프라인 사전 압축이 필요함(인덱스 저장 비용). 백본마다 따로 학습해야 함. |
| [Rethinking Soft Compression in RAG: A Query-Conditioned Selector Perspective (SeleCom)](https://arxiv.org/abs/2602.15856) | 2026 / WWW '26 | 압축 encoder를 query-conditioned selector로 재해석. 연산과 latency 33.8–84.6% 감소. | 미확인 | 쿼리 조건부라서 오프라인 사전 압축의 이점이 사라짐(추론). |
| [Compression Beyond the Uncompressed: Two-Stage Training for Soft Context Compression in RAG (DEX-Comp)](https://arxiv.org/abs/2609.05152) | 2026 / arXiv | distillation 뒤에 비압축 RAG가 틀리는 쿼리로 RL. 16배 압축, 4–24배 가속. | QA 5종 | 미확인. |
| [AdaMem: Adaptive Memory Token Allocation for Soft Compression in RAG](https://arxiv.org/abs/2609.22100) | 2026 / arXiv | passage마다 memory token 수를 적응적으로 배분(OSCAR 대비). KILT-SCR 리소스 공개. | open-domain QA 6종 | 제출일 메타데이터 불일치(검색 결과 기준 **미확인**). |
| [No Mean Feat: Simple, Strong Baselines for Context Compression (BenchPress)](https://arxiv.org/abs/2510.20797) | 2025 / arXiv | 압축 평가 스위트. mean pooling과 양방향 압축 토큰이 causal 압축 토큰보다 우수. | 자체 suite | RAG 전용은 아님. |

### B.3 반복되는 open problem / 암묵적 가정 / 벤치마크 공백
1. **"relevance ranking = RAG에 좋은 context"라는 가정**: Power of Noise, 2507.19102, 2510.11358, SetR이 이 가정을 반박한다. 그런데도 reranker 논문 대부분(RankGPT부터 ReasonRank까지)은 여전히 nDCG@10(TREC DL, BEIR, BRIGHT)으로만 평가하고, 다운스트림 생성 품질은 보고하지 않는다.
2. **추론 reranker의 효과-비용 trade-off가 정리되지 않음**: Rank1, Rank-R1, ReasonRank는 BRIGHT에서 이득을 보이지만, 2505.16886은 같은 조건에서 비추론이 낫다고 보고한다. TALRanker는 처리량 문제를 제기했다. 쿼리별로 "추론이 필요한지"를 판단하는 연구는 거의 없다.
3. **reranking·filtering·compression이 분리된 파이프라인으로 설계됨**: Provence(rerank + prune 통합)와 SeaKR(불확실성 기반 재정렬)이 예외다. 대부분은 단계별로 독립 최적화되어 있어 단계 사이의 오류 전파와 중복 비용이 측정되지 않는다.
4. **compression 평가의 취약성**: soft compression 논문은 단답 QA(NQ, TriviaQA, HotpotQA) 위주다. 압축률과 속도의 기준이 서로 다르다(오프라인 사전 압축 vs 쿼리 시 압축). BenchPress는 단순 baseline이 더 강하다고 보고했다. 압축된 문맥으로 출처 인용(attribution)이 가능한지는 거의 다뤄지지 않는다.
5. **utility의 generator 종속성**: 2510.11358에 따르면 utility는 LLM마다 다르다. 하지만 reranker와 compressor는 특정 generator 하나에 맞춰 학습된다(RECOMP, PISCO, xRAG). generator를 바꾸면 재학습이 필요하다.
6. **집합 수준 목표의 부재**: SetR을 빼면 대부분 점수를 독립적으로 매긴 뒤 top-k를 자른다. 중복, 상호보완성, 충돌(서로 모순되는 증거)을 고려한 선택과 이를 재는 지표(정보 요구 coverage)가 표준화되어 있지 않다.

### B.4 구체적 연구 공백 (under-explored)
1. **쿼리 적응형 "추론 예산" reranking**
   - 문제: 쿼리마다 비추론 점수, 짧은 추론, 긴 추론 또는 도구 호출 중 하나를 고르는 라우팅.
   - 가장 가까운 작업: TALRanker는 도구 호출 여부만 적응적으로 정한다. 2505.16886은 "추론이 필요 없다"는 평균적 결론만 낸다.
   - 미해결 이유: 추론이 도움이 되는 쿼리 유형을 판별하는 신호가 없다. BRIGHT와 DL 사이의 혼합 트래픽에서 비용-효과 frontier를 측정한 연구도 없다.
2. **generator-agnostic utility와 집합 선택**
   - 문제: 2510.11358은 utility가 LLM마다 다르다고 보였고, 2507.19102는 단일 generator로 distill한다. SetR은 coverage를 쓰지만 선택 비용이 큰 LLM에 의존한다.
   - 비어 있는 부분: 여러 generator에 걸쳐 transfer되는 utility 표현(또는 generator 특성을 조건으로 받는 소형 selector)과 집합 수준 목표(coverage − 중복 − 충돌)를 결합한 경량 모델.
3. **reranking과 compression의 단일 패스 통합 + budget 제약 최적화**
   - Provence는 문장 pruning과 reranking을 통합했다. AdaMem은 passage별 토큰 수를 배분한다. 하지만 "총 토큰 예산 B 안에서 어떤 passage를 얼마나 압축해 넣을지"를 함께 정하는 방법은 확인되지 않았다(knapsack 형태).
   - AdaRankLLM(2604.15621)이 보인 "최적 k는 동적"이라는 결과와 연결할 수 있다.
4. **압축과 pruning 이후의 attribution·faithfulness 평가**
   - xRAG, PISCO, DEX-Comp는 정답 정확도만 보고한다. 압축 후에 답변이 어느 원문 문장에 근거하는지 추적할 수 있는지, 그리고 압축이 distractor의 영향을 줄이는지 키우는지(Power of Noise 관점)는 측정되지 않았다. 출처 인용이 필요한 enterprise RAG 환경에서 중요한 공백이다.
5. **"downstream-aware" reranker 벤치마크**
   - 현재 reranker 벤치(BRIGHT, BEIR)는 nDCG만 잰다. SciRerankBench(2508.08742)는 RAG와 연결했지만 과학 도메인에 한정된다.
   - 비어 있는 부분: 같은 후보 풀에서 nDCG, 생성 정확도, 토큰 비용, distractor 민감도를 동시에 재는 공개 벤치. AdaMem의 KILT-SCR(대규모 후보 풀)이 출발점이 될 수 있지만 범위는 **미확인**이다.

---

## 교차 축 관찰
- **두 축의 공통 병목은 "운영점(budget)과 calibration"이다.** query 측(검색할지, 확장할지)과 rerank 측(추론할지, 몇 개 넣을지, 얼마나 압축할지)이 각각 따로 예산을 쓰고, 이를 함께 최적화하거나 평가한 연구는 확인되지 않았다. 2026년 budget-aware 평가 논문(2607.24010, 2606.29959, 2606.02245)은 모두 검색 트리거 하나에만 집중한다.
- **SeaKR(불확실성 기반 재정렬)과 AdaRankLLM(listwise dropout을 adaptive retrieval로 사용)은 두 축을 잇는 드문 사례다.** "trigger와 selection을 하나의 정책으로 통합"하는 방향의 근거가 된다.
