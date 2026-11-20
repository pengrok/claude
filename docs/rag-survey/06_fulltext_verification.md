# 본문 전수 확인 결과 (2026-11-20)

원자료 01~05에 나온 논문을 모두 원문으로 다시 확인했다. arXiv 194편은 PDF를, ACL Anthology·ICLR·NeurIPS 논문 14편과 블로그 2건은 공식 페이지를 받아 읽었다. 철회된 Re3(2509.01306) 1편은 본문이 없다.

## 집계 (확인 항목 725개)

| 판정 | 건수 | 의미 |
| --- | --- | --- |
| 일치 | 349 | 기록이 원문과 같음 |
| 수정 | 335 | 세부 교정, 또는 "미확인"이던 칸을 원문으로 채움 |
| 오류 | 32 | 기록이 원문과 다름(수치, 제목, 잘못 귀속한 한계·주장) |
| 본문 확인 불가 / 접근 불가 | 9 | 철회 논문, 로컬 본문 없는 참고 문헌(RaDeR, MRL, T-GRAG, Provence 등) |

## 결론에 영향을 준 주요 교정

- C3(집합 utility retriever): OptiSet(2601.05027) §3.2.2가 이미 closed-book 대비 엔트로피 차분(ΔH = H_p − H_o)을 집합 단위 학습 레이블로 쓴다. GainRAG도 contrastive decoding으로 파라메트릭 지식 편향을 뺀다. 남은 공백은 dense bi-encoder 학습과 학습에 쓰지 않은 생성기로의 전이 평가뿐이다.
- C6(멀티벡터 하위호환): 다세대 업그레이드 체인은 단일벡터에서 이미 다뤄졌다(RACT 3세대, QDC 5단계). 멀티벡터·PQ/RaBitQ 인덱스는 여전히 비어 있다.
- C1(요약 계층 삭제): 7편 모두 삭제된 사실이 답변으로 새는 현상을 측정하지 않는다. Stashbird는 삭제 후 요약을 그대로 둔다고 한계에 적었다. 공백이 원문으로 확인됐다.
- C7: C-RAG(2402.03181)가 이미 LTT식 다중검정으로 RAG 설정을 고른다. 차별점은 오류와 실현 비용의 동시 보장, 질의 확장을 포함한 쿼리별 행동 공간으로 좁아졌다.
- 청킹: Zhou 2026이 이미 문서 내 판별력 저하 폭(−3.83% ~ −53.17%)을 측정했다.

## 배치별 상세 표

배치마다 `ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정` 형식이다.


---

## 배치 b01

# b01 검증표: 01_embedding_alignment.md (축 A 임베딩, 축 B Retriever–LLM 정렬, 참고 목록)

근거: 로컬 본문 txt/(pdftotext)와 meta/arxiv_meta.json. 웹 미사용. MUVERA는 txt/2405.19504.txt가 없어서 txt/neurips2024_muvera.txt로 확인함.

판정 범례: 일치 / 수정(서술 보완·정밀화) / 오류(원 기록이 본문과 다름) / 본문 확인 불가 / 본문 접근 불가

## 축 A

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2024.acl-long.642 | Improving Text Embeddings with LLMs (E5-Mistral) | 서지 | 2024 / ACL | 첫 쪽 각주 "Proceedings of the 62nd Annual Meeting of the ACL (Vol 1: Long Papers), pp. 11897–11916" | 일치 |
| 〃 | 〃 | Core | Mistral-7B+LoRA, 독점 LLM 합성, 93개 언어, 수십만 태스크, 1k step 미만 | 초록, §4.1(LoRA rank 16) | 일치 |
| 〃 | 〃 | 수치 | MTEB 합성만 63.1, +레이블 66.6 | Table 1(합성만 63.1 / full 66.6), BEIR 56.9 | 일치(BEIR 56.9 추가) |
| 〃 | 〃 | Limitations | "독점 LLM 합성 데이터에 의존. 그 밖은 미확인" | Limitations 절: LLM 기반이라 추론 비용 큼, 4096차원이라 저장 비용 큼, 독점 LLM용 수동 프롬프트 엔지니어링 | 수정 |
| 2404.05961 | LLM2Vec | 서지 | 2024 / COLM | 첫 쪽 헤더 "Published as a conference paper at COLM 2024", arXiv comment | 일치 |
| 〃 | 〃 | Core/수치 | 양방향 attention+MNTP+SimCSE, MTEB 비지도 SOTA | 초록, Table 1(비지도 56.80) | 일치(56.8 추가) |
| 〃 | 〃 | Benchmarks | MTEB | 단어 수준 과제(chunking/NER/POS) + MTEB | 수정 |
| 〃 | 〃 | Limitations | 미확인 | 부록 A Limitations: LLM 크기와 4096차원 때문에 비용 큼, 사전학습 데이터 오염 가능성, 영어로만 평가 | 수정 |
| 2405.17428 | NV-Embed | 서지 | 2025 / ICLR Spotlight | 헤더 "ICLR 2025", arXiv comment "ICLR 2025 (Spotlight)" | 일치 |
| 〃 | 〃 | Core | "공개 데이터만 사용" | 초록·§4: 공개 데이터 + 합성 데이터 생성(Mixtral-8x22B, 12만 예시) + hard-negative mining | 오류 |
| 〃 | 〃 | 수치 | MTEB(v1 69.32, v2 72.31) | Table 1: NV-Embed-v1 69.32, NV-Embed-v2 72.31. 둘 다 MTEB 56개 태스크 점수이고 "v1/v2"는 모델 버전 | 수정(표기 명확화) |
| 〃 | 〃 | Benchmarks | MTEB | MTEB 56개 태스크 + AIR-Bench | 수정 |
| 〃 | 〃 | Limitations | 미확인 | Limitations 절 없음. 결론에도 한계 언급 없음 | 수정(명시된 한계 없음) |
| 2024.findings-acl.137 | M3-Embedding (BGE-M3) | 서지 | 2024 / Findings ACL | 첫 쪽 각주 "Findings of the ACL: ACL 2024, pp. 2318–2335" | 일치 |
| 〃 | 〃 | Core | dense/sparse/multi-vector, self-KD, 100+개 언어, 8192 | 초록 | 일치 |
| 〃 | 〃 | Benchmarks | MIRACL, MKQA | MIRACL, MKQA, MLDR, NarrativeQA(Table 1–4) | 수정 |
| 〃 | 〃 | Limitations | 미확인 | Limitations: 다양한 실제 데이터셋으로의 일반화는 추가 검증 필요, 8192 초과 초장문서 효율 문제, 언어별 성능 편차는 충분히 논의하지 못함 | 수정 |
| 2503.07891 | Gemini Embedding | 서지 | 2025 / tech report | arXiv comment "19 pages", 본문 "In this report" | 일치(venue 없음) |
| 〃 | 〃 | 수치 | MMTEB 1위(Borda), XOR-Retrieve SOTA | §4: MTEB(Multilingual) Borda 1위, task mean 68.32(2위 대비 +5.09), XOR-Retrieve 90.42 | 일치(수치 추가) |
| 〃 | 〃 | Limitations | 폐쇄 모델이라 재현 불가(관찰) | 명시적 Limitations 절 없음. §6에서 hard negative가 과하면 과적합된다고 하고, §7 Future Work에서 멀티모달 확장을 제시 | 수정 |
| 2506.05176 | Qwen3 Embedding | 서지 | 2025 / tech report | 헤더 "Technical Report" | 일치 |
| 〃 | 〃 | 수치 | 약 1.5억 쌍(2차 출처), MMTEB 70.58, MTEB Code 80.68 | §3: "approximately 150 million pairs"(약한 감독 합성), 그 뒤 약 1,200만 쌍 고품질 선별. 70.58 / 80.68은 Table 2/3과 본문 | 일치(2차 출처 표시 삭제) |
| 〃 | 〃 | Limitations | 미확인 | Limitations 절 없음. 결론에도 한계 없음 | 수정(명시된 한계 없음) |
| 2502.13595 | MMTEB | 서지 | 2025 / ICLR | 헤더 "ICLR 2025", arXiv comment | 일치 |
| 〃 | 〃 | Core | 500+ 태스크, 250+ 언어, instruction following/장문서/코드 | 초록 | 일치 |
| 〃 | 〃 | Core(누락) | — | §1: 추론 과제(RAR-b, BRIGHT)도 통합. 다운샘플링으로 계산 비용 절감 | 수정 |
| 〃 | 〃 | Limitations | "공개 1위가 560M → LLM 우위가 상황에 따라 다르다는 시사"(우리 해석) | Limitations 절: 사람 번역 허용에 따른 English leakage, 기여 점수 배분의 불완전함, 고자원 언어 편중 | 오류(저자 한계로 교체) |
| 2403.06789 | SPLADE-v3 | 서지 | 2024 / arXiv | arXiv comment "Technical report" | 일치 |
| 〃 | 〃 | 수치 | MRR@10 40.2, BEIR-13 51.7(모델카드) | Table 1: MSM 40.2, BEIR13 51.7(본문 표에 있음) | 일치(출처를 본문 Table 1로) |
| 〃 | 〃 | Benchmarks | MS MARCO, BEIR-13 | MS MARCO, TREC DL19/20, BEIR-13, LoTTE 등 최대 44개 query set(meta-analysis) | 수정 |
| 〃 | 〃 | Limitations | "새로움 부족으로 보고서로 냄" | §1 원문 "despite the lack of novelty required for a proper publication". 결과 절: Lexical 변형은 BEIR에 약하고 Doc 변형은 zero-shot에 약함 | 일치(보완) |
| 2408.16672 | Jina-ColBERT-v2 | 서지 | 2024 / arXiv(EMNLP 워크숍) | arXiv comment "EMNLP workshop submission". 채택 근거 없음 | 수정 |
| 〃 | 〃 | 수치 | 저장 공간 최대 50% 절감 | §(MRL): 64~768차원 linear head 6개를 MRL로 학습. "50%"는 본문에 없음 | 본문 확인 불가 |
| 〃 | 〃 | Benchmarks | BEIR, MIRACL, mMARCO | BEIR, LoTTE, MIRACL, mMARCO | 수정 |
| 〃 | 〃 | Limitations | 미확인 | Limitations 절 없음. 결과 절에서 ArguAna 약세, 다국어 학습의 curse of multilinguality 가능성 언급 | 수정 |
| neurips2024_muvera (2405.19504) | MUVERA | 서지 | 2024 / NeurIPS | 첫 쪽 "38th Conference on NeurIPS (NeurIPS 2024)". txt/2405.19504.txt는 없음(NeurIPS판으로 확인) | 일치 |
| 〃 | 〃 | 수치 | recall +10%, latency −90% | 초록·§4: PLAID 대비 평균 recall 10%↑, latency 90%↓. BEIR 6개 데이터셋 | 일치 |
| 〃 | 〃 | Limitations | 미확인 | "Broader Impacts and Limitations": MS MARCO에서는 PLAID를 이기지 못함, 문서당 임베딩 수(m_avg)의 영향은 연구하지 않음 | 수정 |
| 2510.12474 | SMEC | 서지 | 2025 / EMNLP(arXiv 코멘트 기준) | arXiv comment "Accepted by EMNLP2025". 본문에 venue 헤더 없음 | 일치 |
| 〃 | 〃 | Core | 순차 MRL, 적응적 차원 선택, cross-batch memory | 초록: SMRL, ADS, Selectable Cross-batch Memory(S-XBM) | 일치 |
| 〃 | 〃 | Benchmarks | BEIR 등(미확인) | BEIR(13개), Products-10K(이미지), Fashion-200K(교차 모달). BEIR에서 256차원 LLM2Vec 기준 Matryoshka-Adaptor 대비 +1.1, Search-Adaptor 대비 +2.7 | 수정 |
| 〃 | 〃 | Limitations | 미확인 | Limitation 절: 특정 도메인 레이블 데이터로 소수 파라미터만 학습해서 범용성이 제한됨. 전체 파라미터 학습과 다양한 데이터 학습은 향후 과제 | 수정 |
| 2407.12883 | BRIGHT | 서지 | 2025 / ICLR | 헤더 "ICLR 2025" | 일치 |
| 〃 | 〃 | 수치 | 12개 도메인, 약 1.4k 질의, SFR 18.3 | 초록: 1,384개 질의, 12개 데이터셋(StackExchange 7개 + 코딩·수학 5개). SFR-Embedding-Mistral은 MTEB **검색 부분집합(BEIR)** 59.0, BRIGHT 18.3 | 수정(59.0의 의미 명확화) |
| 〃 | 〃 | 수치 | 1,384/1,398로 버전마다 다름 | 이 본문 버전에는 1,384만 있음. 1,398은 본문에 없음 | 본문 확인 불가(1,398) |
| 〃 | 〃 | Limitations | 쿼리 수가 적음(관찰) | 부록 G: 관련성 판단이 주관적(다수 주석자 합의로 완화), 클러스터링 등 다른 임베딩 과제와 멀티모달은 다루지 않음 | 수정 |
| 2025.naacl-long.597 | FollowIR | 서지 | 2025 / NAACL | 첫 쪽 "Proceedings of the 2025 Conference of NAACL" | 일치 |
| 〃 | 〃 | Core | TREC narrative 기반, p-MRR, FollowIR-7B | 초록, §3 | 일치 |
| 〃 | 〃 | Benchmarks | FollowIR | FollowIR(Robust04, News21, Core17) + InstructIR. FollowIR-7B 평균 p-MRR +12.2(Table 2) | 수정 |
| 〃 | 〃 | Limitations | "instruction을 키워드로만 사용" 진단 | 이것은 발견(§4.3)이고 한계가 아님. §7 Limitations: 전체 코퍼스 검색이 아닌 reranking 설정만 평가(라이선스 문제로 passage만 배포), TREC 판정과 새 주석의 오류 가능성 | 수정 |
| iclr2025_promptriever | Promptriever | 서지 | 2025 / ICLR | 헤더 "ICLR 2025" | 일치 |
| 〃 | 〃 | 수치 | 약 50만 instruction, FollowIR +14.3 p-MRR | 초록: ~500k instances, FollowIR +14.3 p-MRR / +3.1 nDCG(RepLLaMA 대비), InstructIR Robustness@10 +12.9, BEIR 평균 +1.4(프롬프트 선택 시) | 일치 |
| 〃 | 〃 | Benchmarks | FollowIR, InstructIR, BEIR | + MS MARCO dev, TREC DL19/20 | 수정 |
| 〃 | 〃 | Limitations | 미확인 | §8 Limitations: in-context 예시 등 다른 프롬프팅 기법은 탐색하지 않음, 어떤 프롬프트가 잘 되는지 이유가 불분명, LLM 생성 데이터의 오류·편향 가능성 | 수정 |
| 2504.20595 | ReasonIR | 서지 | 2025 / arXiv(COLM 2025로 알려짐, 미확인) | 본문 헤더와 arXiv comment 모두 venue 없음 | 수정(venue 근거 없음) |
| 〃 | 〃 | 수치 | BRIGHT 29.9(리랭커 36.9) | 초록·§4: 29.9는 **GPT-4 재작성 질의** 기준이고 원 질의는 24.4. 리랭커 포함 36.9. MMLU +6.4%, GPQA +22.6%(closed-book 대비) | 수정(조건 명시) |
| 〃 | 〃 | Core(누락) | — | §3: seed 문서로 BRIGHT 저자(Su et al., 2024)가 모은 문서 풀을 FineWeb-Edu로 필터해 사용 | 수정(오염 논의에 영향) |
| 〃 | 〃 | Limitations | 미확인 | Limitations 절 없음. §7 Discussion의 향후 과제: 합성 데이터 스케일링, seed 선택, 다국어·멀티모달 확장, 다중 턴 추론 결합 | 수정 |
| 2510.08252 | ReasonEmbed | 서지 | 2025 arXiv → 2026 / ACL | arXiv comment "Accepted to ACL 2026 Main". 본문 헤더 없음 | 일치 |
| 〃 | 〃 | 수치 | ReMixer 82K, BRIGHT 38.1(8B), 37.1(4B) | 초록, Table 2(원 질의 기준, Redapter: 4B 37.1 / 8B 38.1) | 일치 |
| 〃 | 〃 | Benchmarks | BRIGHT | BRIGHT + R2MED(OOD 의료) | 수정 |
| 〃 | 〃 | Core(누락) | — | §3: BRIGHT 12개 데이터셋 코퍼스를 합성 원천으로 직접 사용, BRIGHT 평가 positive 문서는 제외. 부록: BRIGHT·R2MED 대비 BM25+weighted Jaccard 질의 중복 분석으로 "누출 없음" 보고 | 수정(A-3 #3, A-4 #5에 영향) |
| 〃 | 〃 | Limitations | 미확인 | Limitations: 합성은 LLM 추론 능력에 묶임, 추론형에만 집중(일반 검색 데이터와 혼합 필요), BRIGHT가 다루는 도메인에 한정 | 수정 |
| 2508.21038 | On the Theoretical Limitations of Embedding-Based Retrieval (LIMIT) | 서지 | 2025 / arXiv(v2 2026-03) | 헤더 "ICLR 2026", arXiv comment "Accepted to ICLR'26" | 오류(venue) |
| 〃 | 〃 | 수치 | SOTA 임베더 매우 낮음, BM25 높음(2차 출처) | §4: 5만 문서·1,000 질의 전체판에서 모델들이 recall@100 20%에도 못 미침. BM25는 거의 완벽. GTE-ModernColBERT는 단일 벡터보다 훨씬 나음. 동의어판(LIMIT-small synonyms)에서는 BM25가 89% 넘게 떨어져 신경 모델보다 낮아짐 | 수정(2차 출처 삭제, 보완) |
| 〃 | 〃 | Limitations | 이론은 단일 벡터만, multi-vector/hybrid는 open question | Limitations 절과 §5.3: multi-vector에는 이론을 확장하지 않음, 오류 허용 설정의 경계는 없음, multi-vector 전이는 open question, sparse/hybrid는 future work | 일치 |
| 2603.29519 | On Strengths and Limitations of Single-Vector Embeddings | 서지 | 2026 / arXiv | arXiv 2026-03-31, venue 없음 | 일치 |
| 〃 | 〃 | Core | 2k+1차원 충분(LIMIT 5차원), drowning으로 재해석 | §1.1: Alon et al.(2016)에 따라 k=2이므로 5차원이면 충분. 초록: 차원만으로는 설명되지 않음. **domain shift와 유사도–관련성 불일치가 주된 요인**이고 fine-tune으로 완화됨. 단일 대 다중 벡터의 격차는 drowning으로 설명 | 수정(원인 서술 정밀화) |
| 〃 | 〃 | 수치 | 단일 약 40%, multi 약 98% R@10 | §1: Recall@10이 단일 벡터는 약 1%→40%, multi-vector는 40%→98%(fine-tune). 추가로 단일 벡터 fine-tune 후 MS MARCO Recall@100이 40% 넘게 하락(catastrophic forgetting) | 일치(보완) |
| 〃 | 〃 | Limitations | 본문 미열람 | Limitations 절 없음(Impact Statement만 있음) | 수정(명시된 한계 없음) |
| 2505.19356 | Amharic Passage Retrieval | 서지 | 2025 / Findings ACL | arXiv comment "Accepted to ACL 2025 Findings" | 일치 |
| 〃 | 〃 | 수치 | MRR@10 상대 +17.6%(2차 출처) | 초록: RoBERTa-Base-Amharic-Embed(110M)가 Arctic Embed 2.0(568M) 대비 MRR@10 +17.6%(0.659→0.775), Recall@10 +9.86%. ColBERT 변형 MRR@10 0.843 | 일치(2차 출처 삭제) |
| 〃 | 〃 | Benchmarks | 자체 | AMNEWS 기반 Amharic Passage Retrieval Dataset + 2AIRTC(부록) | 수정 |
| 〃 | 〃 | Limitations | 미확인 | §8: 사람 관련성 판정이 없음(헤드라인=관련 문서라는 약한 감독), 규모가 작음, 사전학습 코퍼스가 3억 토큰으로 작음 | 수정 |

## 축 B

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2024.naacl-long.463 | REPLUG | 서지 | 2024 / NAACL | 첫 쪽 "Proceedings of the 2024 Conference of NAACL" | 일치 |
| 〃 | 〃 | Core | LM 출력 점수로 retriever 감독(LSR), KL 맞춤 | §4: KL(Q_LM ∥ P_R) 최소화 | 일치 |
| 〃 | 〃 | 수치 | MMLU Codex +5.1% | 초록: GPT-3 LM 6.3%, Codex 5-shot MMLU +5.1%(LSR) | 일치 |
| 〃 | 〃 | Benchmarks | LM perplexity, MMLU | Pile(BPB), MMLU, NQ, TriviaQA | 수정 |
| 〃 | 〃 | Limitations | 미확인 | §9: 검색 지식과 파라메트릭 지식 중 무엇에 의존하는지 불명확(해석 가능성), 항상 검색해서 방해 문서 위험이 있음, DB가 Wikipedia/Pile로 한정 | 수정(B-3 #4 근거로 사용 가능) |
| 2310.01352 | RA-DIT | 서지 | 2024 / ICLR | 헤더 "ICLR 2024", arXiv comment "v4: ICLR 2024 camera-ready" | 일치 |
| 〃 | 〃 | 수치 | 0/5-shot +8.9%/+1.4% | 초록·§5.1: REPLUG(in-context RALM) 대비, MMLU·NQ·TQA·ELI5 평균 | 일치 |
| 〃 | 〃 | Benchmarks | KILT 계열 등 | MMLU, NQ, TriviaQA, KILT 부분집합 | 수정 |
| 〃 | 〃 | Limitations | 미확인 | Limitations 절 없음. 향후 과제로 multi-step RA-DIT(반복 학습에서 추가 이득 없음), 과제별 검색 질의 자동 생성 | 수정 |
| 2024.acl-long.203 | ARL2 | 서지 | 2024 / ACL | 첫 쪽 "Proceedings of the 62nd Annual Meeting of the ACL (Vol 1)" | 일치 |
| 〃 | 〃 | 수치 | NQ +5.4%, MMLU +4.6% | 초록 | 일치 |
| 〃 | 〃 | Core(전이) | B-4 #2에서 "전이 능력 주장"을 생성기 전이 근거로 사용 | §4.3: 전이는 **과제/데이터셋 전이**(NQ로 학습→TQA zero-shot, MMLU few-shot). 레이블러는 ChatGPT 하나이고 생성기 교차 실험은 없음 | 오류(B-4 #2 서술 교정) |
| 〃 | 〃 | Limitations | 미확인 | Limitations: ChatGPT API 주석 비용, 생의학 등 특정 도메인 확장과 평가 필요 | 수정 |
| 2403.19216 | Are LLMs Good at Utility Judgments? | 서지 | 2024 / SIGIR | 첫 쪽 ACM Reference "SIGIR '24", arXiv comment | 일치 |
| 〃 | 〃 | Benchmarks | NQ 등 QA(미확인) | NQ, HotpotQA, MSMARCO-QA. LLM 5개 | 수정 |
| 〃 | 〃 | Limitations | 반사실 passage를 쉽게 받아들임 | 이것은 발견(초록). §7 결론의 한계: 작은 후보 집합만 평가(대규모 후보로 확장 필요), zero-shot만 탐색(few-shot 등 필요). 관련 연구 절: LLM 답 기반 utility는 LLM 내부 지식의 영향을 받음 | 수정 |
| 2404.13781 | eRAG | 서지 | 2024 / SIGIR(short) | 첫 쪽 "SIGIR '24". "short" 표기는 본문에 없음(4쪽 분량) | 수정(short 여부 본문 확인 불가) |
| 〃 | 〃 | 수치 | Kendall τ +0.168~0.494 | 초록. 그리고 GPU 메모리 최대 50배 절감 | 일치 |
| 〃 | 〃 | Benchmarks | KILT 계열 | NQ, TriviaQA, HotpotQA, FEVER, WoW(KILT) | 수정 |
| 〃 | 〃 | Limitations | 문서 간 상호작용 미반영(관찰) | §3.2: 검색 문서 수가 늘면 상관이 떨어지고, 원인은 모든 지표가 문서별 레이블을 독립적으로 쓰는 반면 LLM은 문서 전체를 쓰기 때문이라고 저자가 직접 설명. 별도 Limitations 절은 없음 | 수정(관찰→저자 서술) |
| 2401.14887 | The Power of Noise | 서지 | 2024 / SIGIR | 첫 쪽 ACM Reference "SIGIR '24" | 일치 |
| 〃 | 〃 | 수치 | distracting 1개로 최대 25% 하락, random 최대 35% 향상 | §5: distracting 1개로 최대 −25%(0.24). random 추가는 검색 문서 4개일 때 최대 +35%(0.07) | 일치 |
| 〃 | 〃 | Limitations | random 이득은 문서 수가 적을 때만(2차 출처) | §5.4: "검색 문서 수와 무관하게" random을 채우면 거의 항상 이득이라고 씀. 최적은 최소한의 문서를 검색한 뒤 random을 채울 때. 저자 한계: 정확 일치 기반 정답 판정의 한계, 메커니즘 미해명(결론) | 오류 |
| 2406.18676 | DPA-RAG | 서지 | 2025 / WWW | 본문에 venue 없음. arXiv comment "Work in progress" | 오류(venue 근거 없음) |
| 〃 | 〃 | Core | pair/point/contrastive reranker 정렬 + LLM 사전정렬, 5가지 쿼리 증강 | 초록 | 일치 |
| 〃 | 〃 | Benchmarks | 4개 지식 QA | NQ, TriviaQA, HotpotQA, WebQSP | 수정 |
| 〃 | 〃 | Limitations | 선호 데이터 부족(증강의 동기) | 이것은 동기이지 한계가 아님. Limitations 절 없음 | 수정(명시된 한계 없음) |
| 2025.acl-long.527 | GainRAG | 서지 | 2025 / ACL | 첫 쪽 "Proceedings of the 63rd Annual Meeting of the ACL (Vol 1)" | 일치 |
| 〃 | 〃 | Core | gain 신호를 합성해 retriever–LLM 선호 정렬 | §3: gain = contrastive decoding을 적용한 perplexity("contrastive perplexity"). **LLM 내부 지식 편향을 빼려고**(CAD 착안) 도입. BGE 기반 selector(middleware)로 증류. pseudo-passage 전략 | 수정(B-4 #3 "미확인" 해소) |
| 〃 | 〃 | Benchmarks | QA(미확인) | 6개: HotpotQA, 2WikiMultiHopQA, WebQuestions, SQuAD, NQ, TriviaQA | 수정 |
| 〃 | 〃 | Limitations | 미확인 | Limitations: passage 조합으로 gain이 더 커지는지 미검증, 학습 데이터가 매우 적음, 소형 모델로 신호를 생성할 수 있는지 미검증 | 수정(B-4 #1 근거) |
| 2025.emnlp-main.88 | Utility-Focused LLM Annotation | 서지 | 2025 / EMNLP | 첫 쪽 "Proceedings of the 2025 Conference on EMNLP, pp. 1683–1702" | 일치 |
| 〃 | 〃 | Core | Qwen2.5-32B MS MARCO utility 레이블, Disj-InfoNCE | 손실 이름은 **SumMargLH**(여러 positive의 marginal likelihood 합 최대화). 주석은 RelSel/UtilSel/UtilRank의 listwise 방식 | 오류(손실 이름) |
| 〃 | 〃 | 수치 | 인간 레이블 20% 혼합 시 동등 | 초록 | 일치 |
| 〃 | 〃 | Limitations | in-domain에서 인간 레이블 대체 불가 | 이것은 발견(LLM 레이블이 in-domain에서 낮고 OOD에서 높음). §8 Limitations: 주석 후보 풀이 인간 positive + 다른 모델의 hard negative로 구성돼 현실과 다름, 더 강한 LLM 미사용, MS MARCO만 주석(NQ는 부록에서 Qwen3-32B로 확인) | 수정 |
| 2510.21440 | Redefining Retrieval Evaluation in the Era of LLMs (UDCG) | 서지 | 2026 / EACL | 본문 헤더와 arXiv comment 모두 venue 없음 | 오류(venue 근거 없음) |
| 〃 | 〃 | Core | 위치 할인과 relevance–utility 불일치 지적, utility+distraction 지표 | §4–5: passage별 utility u(q,p) = R(q,p)·(1 − P_LLM(NO-RESPONSE)). 각 passage를 **따로** LLM에 넣어 구하고, UDCG는 이를 선형 결합(γ로 방해 항 가중) | 수정(집합 의존성 해석에 영향) |
| 〃 | 〃 | 수치 | 상관 최대 +36%, 6개 LLM | 초록·§7 | 일치 |
| 〃 | 〃 | Benchmarks | NQ, TriviaQA, PopQA, BioASQ, NoMIRACL | §6: NoMIRACL은 **영어 부분집합**만 사용 | 수정 |
| 〃 | 〃 | Limitations | γ(1/3) 하나에 의존 | 저자는 γ를 시나리오별로 조정 가능하다고 했고 1/3이 여러 설정에서 잘 된다고 보고(한계로 쓰지 않음). Limitations: QA에만 실험(멀티홉·사실검증 미검증), logit 접근이 필요해 블랙박스 API 불가, 영어 데이터만 | 오류(저자 한계로 교체) |
| 2503.09516 | Search-R1 | 서지 | 2025 / arXiv(COLM 2025로 알려짐, 미확인) | 헤더 "Published as a conference paper at COLM 2025" | 수정(COLM 2025 확인) |
| 〃 | 〃 | Core | RL 검색 교차 호출, retriever 고정 | 초록, §4(E5 retriever, 2018 Wikipedia) | 일치 |
| 〃 | 〃 | 수치/Benchmarks | 7개 QA | NQ, TriviaQA, PopQA, HotpotQA, 2Wiki, MuSiQue, Bamboogle. Qwen2.5-7B +24%, 3B +20% | 일치 |
| 〃 | 〃 | Limitations | retriever는 최적화 대상 아님(참고용) | Limitations 절 없음. 결론의 향후 과제: 보상 설계, 불확실성 기반 검색 조절, 다양한 도구, 멀티모달 | 수정 |
| 2505.15117 | Empirical Study on RL for Reasoning-Search Agents | 서지 | 2025 / arXiv | arXiv comment "22 pages" | 일치 |
| 〃 | 〃 | Core | 중간 retrieval 보상 효과 제한, 검색 엔진 선택이 학습 동역학 좌우 | 초록(format 보상은 효과 있음, LLM 규모와 초기화도 중요) | 일치 |
| 〃 | 〃 | Benchmarks | QA | Search-R1과 같은 7개 QA | 수정 |
| 〃 | 〃 | Limitations | 미확인 | 부록 A: 검색 에이전트 시나리오에 한정되어 데이터 과학·SWE 등 다른 에이전트로 일반화되지 않음 | 수정 |
| 2601.11888 | Agentic-R | 서지 | 2026 / arXiv | arXiv 2026-01-17, venue 없음 | 일치 |
| 〃 | 〃 | Core | 로컬 관련성 + 글로벌 정답 정확도, 에이전트–retriever 반복 학습 | §3: 글로벌 정답 정확도 = passage를 넣고 에이전트가 이어서 생성한 답의 EM(closed-book 보정 없음) | 일치(보완) |
| 〃 | 〃 | Limitations | 미확인 | Limitations: 단일/멀티홉 QA만 평가(GPQA 같은 전문가 추론 미평가), 중간 크기 에이전트와 retriever만 실험 | 수정 |
| 2605.04018 | Rethinking Reasoning-Intensive Retrieval (BRIGHT-PRO, RTriever) | 서지 | 2026 / ACL | arXiv comment "ACL 2026" | 일치 |
| 〃 | 〃 | Core | 다측면 gold evidence, 정적/에이전트 이중 평가, RTriever-4B(Qwen3-Emb-4B LoRA) | 초록. RTriever는 RTriever-Synth(MS MARCO seed 기반 합성)로 **오프라인 학습**했고, 에이전트 루프 안에서 학습하지 않음 | 수정(B-1 분류 교정) |
| 〃 | 〃 | Limitations | 정적 순위와 에이전트 성능의 상관이 느슨함 | 이것은 발견(§5 "Static rankings translate only loosely"). Limitations: BRIGHT StackExchange 7개 도메인만, 전문가 주석 비용 때문에 예시 수가 적음, 학습은 1-positive-1-negative 부분집합만 사용 | 수정 |
| 2606.00590 | Critic-R | 서지 | 2026 / Findings EMNLP | 본문 헤더와 arXiv comment 모두 venue 없음(2026-05-30 arXiv) | 오류(venue 근거 없음) |
| 〃 | 〃 | Core/Benchmarks | critic 기반 쿼리·instruction 재작성, Critic-Embed / HotpotQA, 2Wiki, MuSiQue, Bamboogle | 초록 | 일치 |
| 〃 | 〃 | Limitations | 미확인 | Limitations: 추론 에이전트의 자기성찰 trace 품질에 의존(약한 모델은 어려움), 정적 Wikipedia 코퍼스만 평가(실시간 웹·기업 문서 미평가) | 수정 |
| 2602.05975 | SAGE | 서지 | 2026 / arXiv | arXiv 2026-02-05, venue 없음 | 일치 |
| 〃 | 〃 | 수치 | 1,200 질의, 20만 논문, BM25 약 30% 우세 | 초록(4개 과학 도메인, DR Tulu 백본, ReasonIR·gte-Qwen2-7B 대비). corpus 수준 증강으로 short-form +8%, open-ended +2% | 일치 |
| 〃 | 〃 | Limitations | 에이전트 서브쿼리가 키워드 지향이라는 원인 지목 | 이것은 원인 분석. Limitations: 에이전트를 retriever에 맞춰 학습하는 실험은 하지 않음(retriever-aware agent training은 향후 과제), 행동 분석이 DR Tulu에 치우쳐 일반화가 제한됨 | 수정 |
| 2605.05242 | Beyond Semantic Similarity … DCI | 서지 | 2026 / NeurIPS | 본문과 arXiv comment에 venue 없음 | 오류(venue 근거 없음) |
| 〃 | 〃 | Core | grep/shell로 코퍼스 직접 탐색 | 초록. 원인은 쿼리 분포가 아니라 고정 top-k 유사도 **인터페이스의 해상도**라고 주장(결론) | 수정(B-1 서술 교정) |
| 〃 | 〃 | Benchmarks | BRIGHT, BEIR, BrowseComp-Plus | BRIGHT 4개(Biology, Earth Sci, Econ, Robotics), BEIR 2개(ArguAna, SciFact), BrowseComp-Plus, 6개 QA(NQ, TQA, Bamboogle, HotpotQA, 2Wiki, MuSiQue) | 수정 |
| 〃 | 〃 | Limitations | 대규모 코퍼스 확장성(2차 출처) | RQ4(§4): 코퍼스를 100K→200K로 늘리면 도구 호출 38.5→86.9, 정확도 −13.6pt. 400K에서는 37.5%. 별도 Limitations 절은 없음 | 일치(2차 출처 삭제, 수치 보완) |

## 참고 목록과 본문 내 인용

| ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정 |
|---|---|---|---|---|---|
| 2605.00063 | A Survey of Reasoning-Intensive Retrieval | 서지 | ACL 2026 | arXiv comment "Accepted to the ACL 2026 Main Conference" | 일치 |
| 〃 | 〃 | A-4 #4 근거 | BRIGHT 표가 "Single Vector"로 분류 | Table(Group 1): ReasonEmbed 8B/4B, DIVER-Retriever 4B(28.9), RaDeR 7B(25.5), ReasonIR 8B(24.4) 모두 Single Vector. 본문에서 multi-vector와 adaptive routing을 향후 방향으로 제시 | 일치 |
| 〃 | 〃 | A-3 #6 관련 | 다국어 추론형 벤치마크 부재 | 서베이 Table에 MathNet-Retrieve(수학, 다국어·멀티모달 동치 문제 검색)가 있음 | 수정(완전 부재 → 드묾) |
| 2025.emnlp-main.1011 | RaDeR | 서지 | EMNLP 2025 | 본문 txt 없음. 서베이(2605.00063) 참고문헌에 "Proceedings of EMNLP 2025, pp. 19981–20008". Anthology ID 1011은 확인 불가 | 본문 접근 불가(venue는 서베이 경유 일치, ID 본문 확인 불가) |
| 2508.07995 | DIVER | 서지/수치 | 제목 검색 결과 기준, BRIGHT 수치 서베이 경유 | 본문 제목 "DIVER: A Multi-Stage Approach for Reasoning-intensive Information Retrieval". 초록: BRIGHT 전체 46.8, 원 질의 31.9. 표: v1 41.6, v2 45.8. 서베이 표의 retriever 단독 4B는 28.9 | 일치(수치 직접 확인) |
| 2402.14776 | 2D Matryoshka Sentence Embeddings | 서지 | 제목 | 본문 제목 일치 | 일치 |
| (MRL) | Matryoshka Representation Learning | 서지 | NeurIPS 2022 | 본문 txt 없음. Jina-ColBERT-v2 참고문헌에 "NeurIPS vol. 35, pp. 30233–30249(2022)" | 본문 접근 불가(서지는 인용 경유 일치) |
| 2506.01615 | IndicRAGSuite | 서지/내용 | 제목만, "사실형 QA 수준" | 본문 제목 일치. Wiki + MS MARCO 기반 19개 언어, 사실형 QA/검색 | 일치 |
| 2601.04720 | Qwen3-VL-Embedding | A-3 #5 근거 | int8 거의 무손실, binary 크게 손상 | §7.1: MS MARCO passage와 VL3-Syn(text→image)에서 int8은 손실이 무시할 만하고 binary는 크게 손상, 저차원일수록 손실이 커짐. 1024→512는 −1.4% | 일치(벤치마크는 MS MARCO/VL3-Syn) |
| 2604.17555 | CoSearch | 내용 | 제목만 확인, 내용 미확인 | 초록: 추론 에이전트와 생성형 document ranker를 GRPO로 공동 학습, sub-query 의미 그룹핑, 랭킹+궤적 결과 복합 보상, 7개 QA. oracle 대비 고정 retrieval 격차는 최대 +26.8% 상대 F1 | 수정(내용 확인) |

## 오픈 문제·연구 공백 서술 점검(본문과의 모순 여부)

| 위치 | 우리 서술 | 본문 근거 | 판정 |
|---|---|---|---|
| A-1 | "MTEB 59.0 모델이 BRIGHT 18.3" | 59.0은 MTEB 검색 부분집합(BEIR) 점수(BRIGHT §1) | 수정 |
| A-1 | ReasonIR 29.9와 ReasonEmbed 38.1을 나란히 비교 | ReasonIR 29.9는 GPT-4 재작성 질의, 원 질의는 24.4. ReasonEmbed 38.1은 원 질의 | 수정(조건 불일치 명시) |
| A-1, A-3 #4 | 2026 반론 = "차원이 아니라 drowning" | 2603.29519: 주원인은 domain shift와 유사도–관련성 불일치, drowning은 단일 대 다중 벡터 격차 설명. Weller는 LIMIT train으로 학습해도 도움이 거의 없어 domain shift가 아니라고 함 → 두 논문이 domain shift에서 충돌 | 수정 |
| A-3 #3, A-4 #5 | 합성 원천과 BRIGHT 중복을 점검한 연구 없음 | ReasonIR과 ReasonEmbed 모두 BRIGHT 문서 풀을 합성 seed로 직접 씀(설계상 도메인 중복). ReasonEmbed는 평가 positive를 빼고 질의 문자열 중복(BM25+weighted Jaccard)을 분석해 누출 없음이라고 보고, OOD로 R2MED 평가. ReasonIR에서는 같은 점검을 찾지 못함 | 오류(공백을 "코퍼스/도메인 수준 held-out 평가 부재"로 좁혀 교정) |
| A-3 #5 | 압축 분석은 모두 BEIR/MTEB | SMEC은 이미지 데이터셋도, Qwen3-VL은 MS MARCO/VL3-Syn | 수정(사실형·일반 벤치마크로) |
| A-3 #6, A-4 #3 | MMTEB에 추론형 태스크 없음, MMTEB 사실형 중심 | MMTEB §1: RAR-b와 BRIGHT 추론 과제를 통합(영어 데이터) | 오류 |
| A-4 #4 | 추론형 ColBERT/SPLADE 체계적 비교 없음 | Weller §5.3: multi-vector의 추론·instruction 과제 적용은 드물고(Chaffin 2025b가 예외) open question. 서베이도 multi-vector를 향후 방향으로 제시 | 일치(보완) |
| B-1 | BRIGHT-PRO/RTriever를 "에이전트 루프 안에서 retriever 학습"으로 분류 | RTriever는 오프라인 합성 데이터로 학습하고 에이전트 프로토콜로 평가만 함 | 오류 |
| B-1, B-4 #4 | DCI가 서브쿼리 분포 불일치의 증거 | DCI는 고정 유사도 인터페이스의 해상도를 원인으로 봄 | 수정 |
| B-3 #1 | UDCG가 유용성의 집합 의존성을 보여 줌, Utility annotation은 문서별 독립 점수 | UDCG utility는 passage별로 따로 구한 뒤 가산 결합(집합 상호작용은 모델링하지 않음). Utility annotation 주석은 listwise(후보 비교)이고 학습 레이블은 문서별 다중 positive. 집합 의존성 근거는 Power of Noise, eRAG(문서 수↑ → 상관↓), GainRAG Limitations | 오류 |
| B-3 #4, B-4 #3 | 파라메트릭 지식 혼입을 분리한 연구 없음, GainRAG gain 정의 미확인 | GainRAG는 contrastive perplexity로 내부 지식 편향을 명시적으로 제거. REPLUG Limitations와 Utility judgments 관련 연구 절도 이 문제를 언급. 에이전트 RL(Agentic-R: EM 기반 GAC)에서는 보정 없음 | 오류(공백을 에이전트 RL로 좁힘) |
| B-3 #5 | 다국어 utility 데이터셋은 NoMIRACL(UDCG 사용) 외에 없음 | UDCG는 NoMIRACL 영어 부분만 썼고 다국어는 한계로 명시 | 수정 |
| B-4 #2 | ARL2가 전이 능력 주장(생성기 전이 근거) | ARL2 전이는 과제/데이터셋 전이 | 오류 |
| B-4 #4 | Critic-R, Agentic-R 모두 QA 멀티홉에 한정 | Agentic-R은 단일+멀티홉 7개, Critic-R은 멀티홉 4개, CoSearch도 7개 QA | 수정 |


---

## 배치 b02a

# b02a 검증표 — 02_chunking_graph.md 머리말 + 축 A (A-1~A-4) + 부록 A-Mem

근거: verify/txt/ 의 본문 텍스트(pdftotext), meta/arxiv_meta.json(arXiv comment). 쪽 번호 대신 섹션/표 번호를 적었다.

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2312.06648 | Dense X Retrieval | venue | 2023 arXiv → EMNLP 2024 Main | 첫 쪽 헤더 "EMNLP 2024 Main Conference" | 일치 |
| 2312.06648 | Dense X Retrieval | Core contribution | 명제 단위 검색, FactoidWiki, passage/sentence/proposition 비교 | 초록, §3–4 | 일치 |
| 2312.06648 | Dense X Retrieval | Benchmarks | 5개 open-domain QA, 4개 dual-encoder | §4.1: NQ, TriviaQA, WebQ, SQuAD, EntityQuestions. §4.2: 4종 retriever. 단 Limitations 절에는 "6 types of dense retrievers"로 적혀 본문 내 불일치 | 수정(데이터셋명 명시) |
| 2312.06648 | Dense X Retrieval | Stated limitations | 미확인 + [관찰] | Limitations 절: Wikipedia 코퍼스만, bi/dual-encoder 위주, 영어만 | 수정 |
| 2401.18059 | RAPTOR | venue | ICLR 2024 | 헤더 "Published as a conference paper at ICLR 2024" | 일치 |
| 2401.18059 | RAPTOR | 수치 | QuALITY GPT-4 결합 시 +20%p 절대 | 초록 "by 20% in absolute accuracy"; 표 7: RAPTOR+GPT-4 82.6 (test)/76.2 (hard) vs CoLISA 62.3/54.7 | 일치(보강) |
| 2401.18059 | RAPTOR | Benchmarks | QuALITY, NarrativeQA, QASPER | §4 | 일치 |
| 2401.18059 | RAPTOR | Stated limitations | 미확인 + [관찰] 정적 코퍼스 | Limitations 절 없음. §3: 요약의 약 4%에 경미한 환각(부록 E, 150노드 중 6), QA 영향은 미미하다고 보고. 업데이트/증분 언급 없음 | 수정(명시된 한계 없음 + 환각 4% 보고) |
| 2402.05131 | Financial Report Chunking | venue | 2024 arXiv (venue 미확인) | 본문·arXiv comment 모두 venue 표기 없음 | venue 근거 없음 |
| 2402.05131 | Financial Report Chunking | Core contribution | element 기반 청킹, chunk size 튜닝 불필요 | 초록 "best chunk size without tuning"; §5 | 일치 |
| 2402.05131 | Financial Report Chunking | 수치 | FinanceBench 검색 정확도 최대 84.4% | 표 4: "Chipper Aggregation"(여러 청킹 결합) page accuracy 84.40. 단일 element 청킹 최고는 Prefix & Table Description 67.38. Q&A 수동 정확도 53.19%(표 5, §5: 기존 50% 대비) | 수정(84.4%는 결합 전략 값) |
| 2402.05131 | Financial Report Chunking | Stated limitations | 회사명·연도 메타데이터 소실 언급 | 해당 서술 본문에서 찾을 수 없음. §5: 결합(aggregation)은 GPT-4 토큰 한도 초과로 Q&A에 쓸 수 없음. §6: 다른 도메인(바이오 등) 검증은 future work | 오류 |
| 2406.17526 | LumberChunker | venue | Findings of EMNLP 2024 | 본문 헤더·arXiv comment에 venue 표기 없음 (Zhou et al. 2026 참고문헌은 Findings EMNLP 2024로 인용) | venue 근거 없음(2차 인용만) |
| 2406.17526 | LumberChunker | 수치 | GutenQA 100권·3,000쌍, DCG@20 +7.37% | 초록 | 일치 |
| 2406.17526 | LumberChunker | Stated limitations | LLM 비용·지연(2차 요약 기준) | §7 Limitations: LLM 필요로 비싸고 느림; 구조화된 텍스트(법률)에서는 불필요하게 복잡; 문서 길이·문서 수에 대한 확장성 문제 | 수정(원문 확인) |
| 2406.15319 | LongRAG | venue | 2024 arXiv (venue 미확인) | arXiv comment "Technical Report" | 수정(Technical Report) |
| 2406.15319 | LongRAG | 수치 | 22M→700K | 초록(v3): "from 22M to 600K"; §3 표: 600K | 수정(600K) |
| 2406.15319 | LongRAG | 수치 | NQ EM 62.7, HotpotQA 64.3, Qasper F1 25.9, MultiFieldQA-en 57.5 | 초록, 표 5 | 일치 |
| 2406.15319 | LongRAG | Stated limitations | 미확인. 버전마다 수치 다름 | Limitations 절 없음. §3.2/§4: 장문 임베딩 모델이 약해 최대 점수 근사로 대체, 향후 장문 임베딩 발전에 기대 | 수정 |
| 2407.16833 | Self-Route | venue | EMNLP 2024 Industry | arXiv comment "Accepted to EMNLP 2024 industry track" | 일치 |
| 2407.16833 | Self-Route | Benchmarks | LongBench, ∞Bench 계열(세부 미확인) | §3/부록 A: LongBench 7종(NarrativeQA, Qasper, MultiFieldQA, HotpotQA, 2WikiMultihopQA, MuSiQue, QMSum) + ∞Bench 2종(En.QA, EN.MC); §5.4 PassKey 합성 | 수정(세부 확정) |
| 2407.16833 | Self-Route | 수치 | 비용 Gemini-1.5-Pro −65%, GPT-4o −39% | §1 | 일치 |
| 2407.16833 | Self-Route | Stated limitations | 미확인 | Limitations 절 없음. §5.4: 합성 데이터셋은 구성 artifact에 크게 좌우됨; §5.5: LLM 내부 지식(누출) 배제는 future work | 수정 |
| 2409.04701 | Late Chunking | venue | 2024 arXiv (Jina AI. venue 미확인) | arXiv comment "11 pages, 3rd draft"; Jina AI + Weaviate | venue 근거 없음 |
| 2409.04701 | Late Chunking | Benchmarks | BEIR 일부(세부 미확인) | §4.1: BeIR 소형 과제(SciFact, NFCorpus, FiQA, TREC-COVID); §4.2–4.3: LongEmbed(NarrativeQA 등, Needle/Passkey 합성) | 수정 |
| 2409.04701 | Late Chunking | Stated limitations | 입력 길이 초과 문서는 long late chunking 필요 | Limitations 절 없음. §4.2: 일부 독해 과제(특히 Needle/Passkey)에서 큰 청크일 때 naive가 더 나음; 부록 A.1: 장문맥 임베딩 모델 자체의 한계(긴 입력 직접 인코딩 성능 저하) | 수정 |
| 2409.05591 | MemoRAG | venue | WWW 2025 | ACM 각주 "WWW '25"; arXiv comment "theWebConf 2025" | 일치 |
| 2409.05591 | MemoRAG | Benchmarks | 세부 미확인(UltraDomain 등) | §4: LongBench, InfiniteBench, 그리고 저자들이 새로 만든 UltraDomain(20개 데이터셋) | 수정 |
| 2409.05591 | MemoRAG | Stated limitations | 미확인 | Limitations 절 없음; §5 Conclusion에 한계 언급 없음 | 명시된 한계 없음 |
| blog_315cd385 | Anthropic Contextual Retrieval | 서지 | 2024 산업 블로그 | 본문 "Published Sep 19, 2024" | 일치 |
| blog_315cd385 | Anthropic Contextual Retrieval | 수치 | 50–100토큰 문맥, 5.7%→2.9%(−49%), 리랭킹 1.9%(−67%) | 본문 일치. 추가: Contextual Embeddings 단독 5.7→3.7%(−35%); 1회성 비용 $1.02/백만 문서 토큰 | 일치 |
| blog_315cd385 | Anthropic Contextual Retrieval | Benchmarks | 자체 데이터 | "codebases, fiction, ArXiv papers, Science Papers" 등 도메인 | 일치(보강) |
| blog_315cd385 | Anthropic Contextual Retrieval | Stated limitations | 독립 재현 제한적; 일반 문서 요약 부착은 이득 미미 | "adding generic document summaries ... very limited gains" 확인; "독립 재현 제한적"은 조사자 해석; 리랭킹은 지연·비용 증가(Cost and latency considerations) | 일치(앞부분은 [관찰]로 표시) |
| 2410.13070 | Is Semantic Chunking Worth...? | venue | NAACL 2025 Findings라는 2차 정보, 미확인 | 본문·comment에 venue 없음 | venue 근거 없음 |
| 2410.13070 | Is Semantic Chunking Worth...? | Benchmarks | stitched + 실제 데이터 | §3: BEIR 등 10개 데이터셋(짧은 문서 6종은 stitch), 증거 검색은 RAGBench 5종 | 수정(세부) |
| 2410.13070 | Is Semantic Chunking Worth...? | Stated limitations | 이득 불일정, stitched에서만 유리, 임베딩 영향 큼 | 이는 §5 결론(결과). Limitations 절: 문장 단위 청킹이라 임베딩이 문맥 없음; 청크 품질 직접 지표 부재; 적합한 데이터셋 부재(stitch 인공성, RAGBench 답이 LLM 생성) | 수정 |
| 2410.05983 | Long-Context LLMs Meet RAG | venue | ICLR 2025 | 본문 헤더 없음, comment "34 pages" | venue 근거 없음 |
| 2410.05983 | Long-Context LLMs Meet RAG | Core contribution | 역U자, hard negative, reordering + robustness FT | 초록: reordering, RAG-specific implicit FT, intermediate reasoning FT 3가지 | 수정(3가지 방법) |
| 2410.05983 | Long-Context LLMs Meet RAG | Benchmarks | NQ, TriviaQA, HotpotQA 등 | 분석: NQ, PopQA. FT 학습: NQ, WoW, FEVER, MMLU; 평가: TriviaQA, PopQA, HotpotQA, 2Wiki, WebQ, Bamboogle, ASQA, T-REx, zsRE | 수정 |
| 2410.05983 | Long-Context LLMs Meet RAG | Stated limitations | 강한 검색기 hard negative↑, precision 부족 | 이는 §3 발견. Limitations 절 없음. §7: 자동 위치 최적화, 다단계 추론 FT는 future work | 수정 |
| 2412.06206 | SiReRAG | venue | ICLR 2025 | 헤더 "Published as a conference paper at ICLR 2025" | 일치 |
| 2412.06206 | SiReRAG | 수치 | 평균 F1 +1.9%, 리랭킹 최대 +7.8% | 초록 | 일치 |
| 2412.06206 | SiReRAG | Stated limitations | 미확인. [관찰] 인덱싱 비용 | Limitations 절 없음. §6.4: RAPTOR보다 질의 시간 김(TPER<1이라 합리적이라 주장). §6: 요약 LLM을 GPT-3.5로 바꿔도 성능 차 미미 → 비용 절감 가능 | 수정(명시된 한계 없음) |
| 2502.09977 | LaRA | venue | ICML 2025 | 본문 헤더·comment("22 pages")에 venue 없음. ICML 양식의 Impact Statement만 있음 | venue 근거 없음 |
| 2502.09977 | LaRA | 수치 | 2,326 케이스, 소설·논문·재무제표, LLM 11종 | 초록: 2326 cases, 4개 과제 유형, 3종 문서, 오픈 7 + 상용 4 | 일치 |
| 2502.09977 | LaRA | Stated limitations | 단일 라우팅 규칙 없음 | 이는 결론(발견). Limitations 절 없음 | 수정(명시된 한계 없음) |
| 2503.09600 | MoC | venue | ACL 2025 Long | 본문·comment에 venue 없음 | venue 근거 없음 |
| 2503.09600 | MoC | Benchmarks | 다수 QA(세부 미확인) | §4.1: CRUD(single/two-hop), DuReader(LongBench), WebCPM — 모두 중국어 | 수정 |
| 2503.09600 | MoC | Stated limitations | 미확인 | §6 Limitations: 학습 데이터 약 2만 건으로 제한적, 다국어 적응·검증 미흡 | 수정 |
| 2504.19754 | Reconstructing Context | venue | ECIR 2025 Workshop (KEIR) | arXiv comment "Second Workshop on Knowledge-Enhanced Information Retrieval, ECIR 2025" | 일치 |
| 2504.19754 | Reconstructing Context | Benchmarks | NFCorpus 등 | §4: 검색 NFCorpus, 생성 MS MARCO. RQ2(CR)은 NFCorpus 20%, 50 질의·약 300 문서 | 수정(규모 명시) |
| 2504.19754 | Reconstructing Context | Stated limitations | CR 응집성↑ 비용↑, LC 효율↑ 관련성·완결성 희생 | 초록 결론과 일치. 추가로 §4: GPU(24GB) 제약으로 소규모 부분집합만 사용 | 일치(보강) |
| 2509.11552 | HiChunk | venue | 2025 arXiv (venue 미확인) | 본문·comment에 venue 없음 | venue 근거 없음 |
| 2509.11552 | HiChunk | Core contribution | fine-tune Qwen3-4B로 경계+계층 레벨 예측, HiCBench | §4: Qwen3-4B 기반; Auto-Merge 검색 알고리즘도 제안(누락) | 수정(Auto-Merge 추가) |
| 2509.11552 | HiChunk | Benchmarks | HiCBench 외(미확인) | §5: 청킹 정확도 Gov-report, Qasper; RAG 전체 HiCBench, LongBench, Qasper, GutenQA, OHRBench | 수정 |
| 2509.11552 | HiChunk | Stated limitations | 기존 청커가 선형 구조만 다룬다고 지적 | 이는 기존 연구 비판(§1). Limitations 절 없음 | 수정(명시된 한계 없음) |
| 2510.04293 | RDR2 | venue | Findings of EMNLP 2025 (검색 결과 기준) | arXiv comment "EMNLP2025 Findings" | 일치 |
| 2510.04293 | RDR2 | 제목 | "RDR2 (구조 라우팅)" | 정식 제목 "Equipping Retrieval-Augmented Large Language Models with Document Structure Awareness" | 수정 |
| 2510.04293 | RDR2 | Core contribution | 헤딩 트리에서 Answer/Expand/Refuse 라우팅 | §3: [ANS]/[EXP]/[REF] 액션, 라우팅을 학습 과제로 정식화 | 일치 |
| 2510.04293 | RDR2 | Benchmarks | 미확인 | §4.1: TriviaQA, HotpotQA, ASQA, QAMPARI, ELI5(각 최대 1,000개 샘플), Wikipedia 데이터스토어 | 수정 |
| 2510.04293 | RDR2 | Stated limitations | 미확인 | Limitations 절: 문서 간 통합 제한(top-k 문서 독립 처리), 전체 데이터스토어의 DST 오프라인 구축 필요, 반복 라우팅의 계산 오버헤드 | 수정 |
| 2025.emnlp-main.1062 | MultiDocFusion | venue | EMNLP 2025 Main | 첫 쪽 각주 "Proceedings of the 2025 Conference on EMNLP, pages 20985–21004", Anthology ID emnlp-main | 일치 |
| 2025.emnlp-main.1062 | MultiDocFusion | Benchmarks/수치 | 산업 문서(미확인) | §4: DSHP-LLM은 DocHieNet+HRDH; RAG-VQA는 DUDE, MPVQA, CUAD, MOAMOB. 초록: 검색 precision +8–15%, ANLS +2–3% | 수정 |
| 2025.emnlp-main.1062 | MultiDocFusion | Stated limitations | 미확인 | Limitations 절: DSHP-LLM이 세밀한 레이아웃 신호(폰트·여백·열 구조)를 활용하지 못함; 그래프 기반 검색은 미검증; 부모 문맥 중복으로 인덱스·지연·저장 비용 증가 | 수정 |
| 2025.emnlp-main.1062 | MultiDocFusion | 오픈문제 관련 | (A-4 #5) 파싱 오류 전파 분석은 2604.12047 정도 | §5.4: DP 모델(DETR/DiT/VGT)·OCR 모델(EasyOCR/Tesseract/TrOCR) 교체에 대한 강건성 실험 수행 | 수정(A-4 #5에 반영) |
| 2602.16974 | Beyond Chunk-Then-Embed | 서지 | 2026 arXiv (Zuccon 그룹) | UQ/CSIRO/Google; comment에 venue 없음(익명 심사 중) | 일치(venue 근거 없음) |
| 2602.16974 | Beyond Chunk-Then-Embed | Benchmarks | BEIR 6종, GutenQA, 임베딩 4종 | §3: FiQA, ArguAna, SciDocs, TREC-COVID, SciFact, NFCorpus + GutenQA; Jina-v2, Jina-v3, Nomic, E5-large | 일치 |
| 2602.16974 | Beyond Chunk-Then-Embed | 핵심 결론 | in-corpus 구조 기반, in-document LumberChunker, 문맥화는 in-document 저하 | 초록, §5. 여기서 "contextualized chunking" = late chunking(분할 전 임베딩). 저하 폭: Nomic −53.17%, Jina-v3 −29.82%, Jina-v2 −5.16%, E5-large −3.83%(Finding 3) | 일치(정의 명확화) |
| 2602.16974 | Beyond Chunk-Then-Embed | Stated limitations | 문맥화 청킹은 in-corpus↑ in-document↓ | 이는 발견. Limitations 절 없음 | 수정(명시된 한계 없음) |
| 2602.16974 | Beyond Chunk-Then-Embed | 오픈문제 관련 | (A-3 #4) 같은 문서 청크가 서로 가까워질 것이라는 [관찰]; trade-off 정량화 연구 없음 | Finding 3: 저자들이 같은 가설을 직접 제시("chunks within the same document more similar"), 저하를 정량화함. 단 가설 검증과 접두사형 CR 실험은 없음 | 수정(A-3 #4, A-4 #2 반영) |
| 2603.06976 | Systematic Investigation (Shaukat) | 수치 | 36 전략 × 6 도메인 × 5 임베딩 = 1,080 | 초록, §1 | 일치 |
| 2603.06976 | Systematic Investigation (Shaukat) | 결론 | (A-1) "만능 청커 없음", semantic/LLM 이득 불안정 | 초록: content-aware 청킹이 naive 고정 길이보다 유의하게 우수, 최고는 Paragraph Group Chunking(nDCG@5≈0.459); LLM 기반 분할도 상위권이나 전처리 메모리 부담 큼(표 3); 도메인별 최적 상이 | 수정(A-1 반영) |
| 2603.06976 | Systematic Investigation (Shaukat) | Stated limitations | 잘게 나누면 인덱스·지연 증가 | 이는 효율 분석 결과. Limitations and Future Work 절: 인덱스 크기·청크 수가 교란 요인, LLM 판정자 사용(편향 가능), UltraDomain 일부 도메인만 | 수정 |
| 2604.12047 | Empirical Evaluation of PDF Parsing and Chunking for Financial QA | 서지/내용 | 세부 미확인 | 첫 쪽 헤더 ICSE-SEIP 2026; PDF 파서(PyMuPDF, Unstructured 등)×청킹(overlap 변화)를 FinanceBench와 신규 TableQuest에서 비교 | 수정 |
| 2502.12110 | A-Mem: Agentic Memory for LLM Agents | venue | NeurIPS 2025 | arXiv comment "NeurIPS 2025" (본문 헤더엔 없음) | 일치 |
| 2502.12110 | A-Mem | Core contribution | Zettelkasten식 노트 연결과 memory evolution | 초록, §3.3 | 일치 |
| 2502.12110 | A-Mem | 추가 | — | 평가: LoCoMo, DialSim, 기반 모델 6종. §6 Limitations: 조직 품질이 기반 LLM에 의존, 텍스트 전용 | 보강 |
| 머리말 | 검증 방법 주의 | 서술 | WebSearch로만 확인, 본문 미열람 | 이번 검증은 본문 텍스트 전수 확인 | 수정(머리말 교체) |
| A-4 #3 등 | StructRAG, EraRAG, LightRAG, "BM25 Wins at Scale", HippoRAG 2 | — | 축 A 본문에서 언급 | b02a 담당 본문 파일 범위 밖(다른 배치에서 확인) | 본 배치 미검증 |


---

## 배치 b02b

# b02b 검증 표 — 02_chunking_graph.md 축 B (B-1~B-4, 부록)

본문 출처: txt/<arXivID>.txt (pdftotext), LazyGraphRAG는 txt/blog_51652eef.txt. 메타데이터: meta/arxiv_meta.json. 모든 대상 본문 접근 가능(본문 접근 불가 0건).

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2404.16130 | From Local to Global: A GraphRAG Approach to Query-Focused Summarization | 서지 | 2024 arXiv (v2 2025-02) | 첫 쪽 "arXiv:2404.16130v2 19 Feb 2025", 각주 "Preprint. Under review."; arXiv comment 없음 | 일치 |
| 2404.16130 | 〃 | Core contribution | 엔티티 KG + 커뮤니티 요약 사전 생성, map-reduce로 글로벌 질의 응답 | 초록 | 일치 |
| 2404.16130 | 〃 | Benchmarks | 약 1M 토큰 코퍼스 2종(팟캐스트, 뉴스), LLM-judge 포괄성·다양성 | §4 데이터셋: Podcast ~1M, News ~1.7M 토큰; 지표 Comprehensiveness/Diversity/Empowerment/Directness(LLM 쌍대 비교) + claim 기반 지표(Table 3·4) | 수정 (News는 약 1.7M, empowerment·claim 지표 추가) |
| 2404.16130 | 〃 | 수치 | (없음) 승률 | §5: 포괄성 승률 72–83%(Podcast)/72–80%(News), 다양성 75–82%/62–71% | 보강 |
| 2404.16130 | 〃 | Stated limitations | directness에서 vector RAG 우세; [관찰] 인덱싱 비용 큼 | §5 "vector RAG produces the most direct responses"(directness는 통제 기준); §6.1 평가가 약 1M 토큰 코퍼스 2개에 한정, 일반화·fabrication rate 비교 필요 | 수정 (저자 명시 한계는 §6.1의 평가 범위) |
| 2402.07630 | G-Retriever | 서지 | NeurIPS 2024 | 본문(v3, 2024-05-27)·arXiv comment 모두 venue 표기 없음 | venue 근거 없음 |
| 2402.07630 | 〃 | Core / Benchmarks | PCST 부분그래프 검색 + GNN soft prompt, GraphQA; ExplaGraphs, SceneGraphs, WebQSP | 초록, §4 "GraphQA benchmark integrates ExplaGraphs, SceneGraphs, and WebQSP" | 일치 |
| 2402.07630 | 〃 | 수치 | 환각 −54%(2차 요약 출처) | §6.4: 응답 100개 수작업 검토, "reduces hallucinations by 54% compared to the baseline" | 일치 (본문 확인) |
| 2402.07630 | 〃 | Stated limitations | 미확인 | §7 뒤 "Limitations and Future Work": 검색 구성요소가 정적(학습 불가), 학습 가능한 검색은 향후 과제 | 수정 |
| 2405.14831 | HippoRAG | 서지 | NeurIPS 2024 | 첫 쪽 각주 "38th Conference on Neural Information Processing Systems (NeurIPS 2024)", arXiv comment | 일치 |
| 2405.14831 | 〃 | 수치 | IRCoT 대비 10–20배 저렴, 6–13배 빠름, 최대 +20% | 초록: "up to 20%", "10-20 times cheaper and 6-13 times faster"; 본문 §1·§5는 "10-30 times cheaper" | 일치 (본문 내 10–30배 표기도 병존) |
| 2405.14831 | 〃 | Benchmarks | MuSiQue, 2Wiki, HotpotQA | §4·§5 | 일치 |
| 2405.14831 | 〃 | Stated limitations | 미확인; [관찰] OpenIE 품질 의존 | §7 Conclusions & Limitations: 구성요소 무학습 사용, 오류 대부분 NER·OpenIE(App. F), PPR 개선 여지, 긴 문서 OpenIE 일관성, 스케일 검증 미흡 | 수정 (관찰이 저자 명시와 일치) |
| 2405.16506 | GRAG | 서지 | Findings of NAACL 2025 | 본문(v3 2025-07)에 venue 표기 없음, arXiv comment "13 pages, 5 figures" | venue 근거 없음 |
| 2405.16506 | 〃 | Benchmarks | WebQSP, ExplaGraphs 등 (미확인) | §5 Table 1·2: WebQSP, ExplaGraphs 두 개뿐 | 수정 |
| 2405.16506 | 〃 | Stated limitations | 무관 노드·엣지 문제를 soft pruning으로 완화한다고 명시 | soft pruning은 방법(§3). §7 Limitations: 부분그래프 검색 효율이 초기 노드 랭킹·pruning 품질에 의존, 구조·노드 중요도 추정이 어려우면 성능 저하 | 오류 → 수정 |
| 2408.08921 | Graph Retrieval-Augmented Generation: A Survey | 서지·핵심 | 2024 arXiv survey; G-Indexing/G-Retrieval/G-Generation 3단계 | arXiv comment "Ongoing work"; §2 3단계 정의(ACM 템플릿 표기는 자리표시) | 일치 |
| 2408.08921 | 〃 | Stated limitations | — | §10 Future Prospects(동적 그래프, 멀티모달, 확장성, 표준 벤치마크 등) | 보강 |
| 2410.05779 | LightRAG | 서지 | 2024 arXiv (v3 2025-04, EMNLP 2025 Findings 미확인) | 본문 v3 28 Apr 2025, venue 표기 없음, arXiv comment 없음 | venue 근거 없음 |
| 2410.05779 | 〃 | Core / Benchmarks | 이중 수준 검색, 증분 삽입; UltraDomain(Agriculture/CS/Legal/Mix) LLM-judge 승률 | §3, §4.1 데이터셋, Table 1 승률 | 일치 |
| 2410.05779 | 〃 | Stated limitations | 미확인; [관찰] 삭제·수정 미지원(Preprints.org 지적) | Limitations 절 없음(§6 Conclusion만). 증분 갱신(§3.3)은 새 문서 D′ 추가만 서술. Preprints.org 논문은 로컬 본문 없음 | 명시된 한계 없음; 관찰은 본문 서술과 모순 없음, 외부 지적은 본문 확인 불가 |
| 2410.14684 | RepoGraph | 서지 | ICLR 2025 | 첫 쪽 "Published as a conference paper at ICLR 2025", comment | 일치 |
| 2410.14684 | 〃 | Core / Benchmarks | 줄 단위 노드 + 정의-참조 엣지; SWE-bench(4개 프레임워크 향상), CrossCodeEval | 초록, §1(평균 상대 향상 32.8%), §5 Table 5 | 일치 (SWE-bench Lite임을 보강) |
| 2410.14684 | 〃 | Stated limitations | 미확인 | App. E: GPT-4 계열만 사용, SWE-bench Lite만 실험(비용), Python만 | 수정 |
| 2410.08815 | StructRAG | 서지 | ICLR 2025 | 본문 v2(2024-10-25)에 venue 표기 없음, comment 없음 | venue 근거 없음 |
| 2410.08815 | 〃 | Core | DPO 학습 router가 표·그래프·알고리즘·카탈로그·청크 중 선택 | §3, §3.1 DPO | 일치 |
| 2410.08815 | 〃 | Benchmarks | Loong 등 (미확인) | §4.1: Loong(4개 과제, LLM 점수·EM), Podcast Transcripts(승률) | 수정 |
| 2410.08815 | 〃 | Stated limitations | 미확인; [관찰] 질의별 구조화 비용 | §6 Conclusion만, 한계 절 없음 | 명시된 한계 없음 (관찰 유지) |
| 2407.02883 | CoIR | 서지 | ACL 2025 Long | arXiv comment "ACL 2025 Main"; 본문 헤더 없음 | 수정 (Main) |
| 2407.02883 | 〃 | Core | 10개 코드 검색 데이터셋, MTEB/BEIR 호환 | §1: 10개(기존 8 + 신규 2), 4개 주 과제·8개 하위 과제, 14개 언어, 검색 모델 10종 평가 | 일치 (보강) |
| 2407.02883 | 〃 | Stated limitations | SOTA 모델도 고전 | "struggle"은 결과(§6). §7 Limitations: 모든 데이터셋이 영어, 텍스트 정보만 쓰는 질의(메타데이터·버전 정보 미반영), 이진 관련성 라벨 | 수정 |
| 2406.14497 | CodeRAG-Bench | 서지 | Findings of NAACL 2025 | 본문 v2(2025-02) 헤더 없음, comment 없음 | venue 근거 없음 |
| 2406.14497 | 〃 | 수치·Core | 약 9k 과제, 25M 문서(튜토리얼, 문서, SO, GitHub), 검색기 10 × LM 10 | 초록·§1: 9k 과제, 25M 문서, 5개 출처(대회 풀이, 튜토리얼, 라이브러리 문서, SO, GitHub), 검색기 10·LM 10 | 수정 (출처에 대회 풀이 누락) |
| 2406.14497 | 〃 | Benchmarks | 기초, open-domain, 저장소 수준 | §2: 기초 프로그래밍, open-domain, 저장소 수준, 코드 검색 4범주 | 수정 |
| 2406.14497 | 〃 | Stated limitations | 검색기가 문맥 놓침, 생성기가 문맥 활용 못함 | 이는 초록의 결과. Limitations 절: 과제·언어 다양성 부족(Python 중심), vanilla 검색·리랭킹·생성 위주 | 수정 |
| 2502.09304 | KET-RAG | 서지 | 2025 arXiv (KDD 2025 여부 미확인) | 첫 쪽 ACM 표기 "KDD '25, August 3–7, 2025, Toronto" | 수정 → KDD 2025 |
| 2502.09304 | 〃 | Benchmarks | multi-hop QA (미확인) | §6.1: MuSiQue, HotpotQA, RAG-QA Arena(각 500문항) | 수정 |
| 2502.09304 | 〃 | 수치 | 인덱싱 비용 10배 이상 절감, 생성 품질 최대 +32.4% | 초록: Graph-RAG 대비 검색 품질 동등 이상 + 인덱싱 비용 "over an order of magnitude" 절감; 생성 품질 +32.4%는 "approximately 20%" 낮은 인덱싱 비용 조건(§1, §7) | 수정 (두 수치는 다른 설정) |
| 2502.09304 | 〃 | Stated limitations | 미확인 | §7: 향후 글로벌 검색 시나리오로 확장, 적응형 패러다임 탐색 (=로컬 검색만 다룸) | 수정 |
| 2502.09891 | ArchRAG | 서지 | 2025 arXiv (v3 2025-08) | 본문 v4 11 May 2026, "Copyright © 2026 AAAI"; journal_ref AAAI 40(19) 15868–15876 | 수정 → AAAI 2026 |
| 2502.09891 | 〃 | 수치 | 토큰 최대 250배 절감, 정확도 +10% | §1: specific 질문에서 SOTA 그래프 RAG 대비 정확도 10% 높음, GraphRAG 대비 토큰 최대 250배 절감 | 일치 |
| 2502.09891 | 〃 | Benchmarks | specific/abstract QA | §실험: Multihop-RAG, HotpotQA, NarrativeQA(specific), Multihop-RAG 코퍼스 기반 생성 질문(abstract) | 보강 |
| 2502.09891 | 〃 | Stated limitations | 미확인 | Conclusion: 대규모 코퍼스 처리로 확장을 향후 과제로 언급, 한계 절 없음 | 명시된 한계 없음 |
| 2502.11371 | RAG vs. GraphRAG | 서지 | 2025 arXiv (v3 2026-03) | 본문 v4 7 Nov 2026, ACM 표기 "KDD '26, August 09–13, 2026, Jeju" | 수정 → KDD 2026 (v4) |
| 2502.11371 | 〃 | Benchmarks | NQ, HotpotQA, MultiHop-RAG, 요약 과제 | §4: NQ, HotPotQA, MultiHop-RAG, NovelQA; §5: SQuALITY, QMSum, ODSum-story, ODSum-meeting | 수정 (NovelQA·요약 데이터셋 명시) |
| 2502.11371 | 〃 | 수치 | 인덱싱 KG-GraphRAG 약 7,702s vs RAG 135s (2차 요약) | Table 4(MultiHop-RAG): 구축 시간 RAG 135s, KG-GraphRAG 7,702s, Community-GraphRAG 5,560s | 일치 (본문 확인) |
| 2502.11371 | 〃 | Stated limitations | LLM-judge 순서에 따라 결과 바뀜 | §1 기여: 요약 LLM-judge가 제시 순서에 민감(위치 효과); GraphRAG 구축 비용·지연·저장 비용, 그래프 구축 품질 민감 | 일치 (보강) |
| 2502.14802 | HippoRAG 2 | 서지 | ICML 2025 | 첫 쪽 "Proceedings of the 42nd International Conference on Machine Learning", comment | 일치 |
| 2502.14802 | 〃 | Core / Benchmarks / 수치 | passage 노드 통합, recognition memory; NQ, PopQA, MuSiQue, 2Wiki, HotpotQA, LV-Eval, NarrativeQA; 연상 +7% | 초록, Fig. 1, Table 1, §3.4 | 일치 |
| 2502.14802 | 〃 | Stated limitations | 미확인 | §7: 한계 절 없음, 향후 긴 대화의 일화 기억으로 확장 | 명시된 한계 없음 |
| 2502.14902 | PathRAG | 서지 | AAAI 2026 | 첫 쪽 "Copyright © 2026, AAAI" | 일치 |
| 2502.14902 | 〃 | Core | 중복 진단, flow 기반 pruning, 신뢰도 오름차순 배치 | 초록, §3 | 일치 |
| 2502.14902 | 〃 | Benchmarks | 6개 데이터셋 × 5개 차원 (UltraDomain 계열, LLM-judge) | Table 1: Legal, History, Biology, Mix(UltraDomain), SQuALITY, SummScreen; 18만–500만 토큰; 승률 | 수정 (요약 데이터셋 2개 포함) |
| 2502.14902 | 〃 | Stated limitations | 그래프 구축 자체는 다루지 않음; 경로 검색에 학습이 없어 성능 제한 가능 | Conclusion: 향후 인덱스 그래프 구축 최적화, 사람 주석 데이터셋 수집, 경로 외 부분구조 탐색. "학습 없음"은 장점으로 서술(§1)되며 한계로 명시되지 않음 | 수정 (후반부 오류) |
| 2503.04338 | In-depth Analysis of Graph-based RAG in a Unified Framework | 서지 | PVLDB 18(13), 2025 | "PVLDB, 18(13): 5623–5637, 2025" | 일치 |
| 2503.04338 | 〃 | Core / Benchmarks | 4단계 통합 프레임워크로 12개 방법 비교 | §1·§3, Table: specific QA 6개(MultihopQA, Quality, PopQA, MusiqueQA, HotpotQA, ALCE), abstract QA 5개(Mix, MultihopSum, Agriculture, CS, Legal); 61만–1,349만 토큰 | 일치 (보강) |
| 2503.04338 | 〃 | Stated limitations | 미확인 | §8 Lessons & Opportunities: O1 모든 방법이 정적 코퍼스 가정, O2 질의 전 그래프 품질 평가가 미해결·구축 토큰 비용 큼, L6 대규모에서 GraphRAG 토큰 비용 과다 | 수정 |
| 2506.06331 | Unbiased Evaluation Framework for GraphRAG | 서지 | 2025 arXiv | 본문 v2 13 Aug 2026, venue 표기 없음 | 일치 (venue 근거 없음) |
| 2506.06331 | 〃 | Core | 위치·길이·시행 편향 제거 후 재평가, 이득이 크게 줄거나 사라짐 | 초록: 두 결함(무관 질문, 평가 편향) 지적, 그래프-텍스트 기반 질문 생성 + 편향 제거 절차; 이득이 "much more moderate", LightRAG는 편향 제거 후 NaiveRAG가 근소 우세(§1) | 수정 (질문 생성 기여 누락, "사라짐"은 일부 사례) |
| 2506.06331 | 〃 | Benchmarks | 3개 GraphRAG(LightRAG 포함) vs NaiveRAG | MGRAG, LightRAG, FGRAG vs NaiveRAG; Harry Potter 소설, Mix·Agriculture·Music | 보강 |
| 2506.06331 | 〃 | Stated limitations | 자신의 프레임워크도 결함 가능 | 초록·§VII "may still have flaws" | 일치 |
| 2506.05690 | GraphRAG-Bench | 서지 | ICLR 2026 (arXiv v3 기준) | 각 쪽 헤더 "Published as a conference paper at ICLR 2026" | 일치 |
| 2506.05690 | 〃 | Benchmarks | GraphRAG-Bench | §3: Novel·Medical(NCCN) 데이터셋, 4단계 과제 | 보강 |
| 2506.05690 | 〃 | Stated limitations | 출발점: GraphRAG가 vanilla RAG보다 못함 (NQ −13.4%, Han et al.) | §1: Han et al.(2025) 인용 NQ 정확도 13.4% 낮음 확인. 그러나 이는 한계가 아닌 동기. App. J Limitation: 텍스트 단일 모달만 다룸 | 수정 (수치는 일치) |
| 2506.05690 | 〃 | 오픈 문제 근거 | "그래프 밀도-검색 품질 상관" (2차 정보) | Obs.7·Table 5: HippoRAG2의 더 조밀한 그래프가 연결성·커버리지를 높여 성능에 기여한다는 정성적 관찰. 정량 상관 분석은 아님 | 수정 |
| 2505.11545 | TARGET | 서지 | 2025 arXiv (NeurIPS 2024 워크숍 페이지 존재, 세부 미확인) | 본문 v1 2025-05-14, venue 표기 없음, comment 없음 | venue 근거 없음 |
| 2505.11545 | 〃 | Benchmarks | 여러 표 데이터셋 | Table 2: OTTQA, FeTaQA(QA), TabFact(사실 검증), Spider, BIRD(text-to-SQL) | 수정 |
| 2505.11545 | 〃 | Stated limitations | dense > BM25, 과제 간 편차, 메타데이터 없으면 민감 | 이는 초록의 결과. §7 Limitations: 소스 코드가 없는 기존 방법 일부 미포함, 열·행 단위 관련성 라벨 없음 | 수정 |
| 2506.20963 | EraRAG | 서지 | 2025 arXiv | comment "Under review" | 일치 |
| 2506.20963 | 〃 | 수치 | 5%씩 10라운드, 토큰 −57.6%, 구축 시간 −77.5% | §1: 5%씩 10라운드; §6: 토큰 −57.6%(PopQA), 재구축 시간 −77.5%(QuALITY), 차선 기준선(RAPTOR) 대비 | 일치 (비교 대상 보강) |
| 2506.20963 | 〃 | Benchmarks | (기록 없음) | §6: PopQA, MultiHopQA, HotpotQA, QuALITY, MuSiQue | 보강 |
| 2506.20963 | 〃 | Stated limitations | 정적 코퍼스 가정 비판; 삭제·수정 미다룸(관찰) | 초록: 기존 방법의 정적 코퍼스 가정 비판. 한계 절 없음. 삽입만 다룸 | 명시된 한계 없음 (관찰은 본문과 모순 없음) |
| 2510.10114 | LinearRAG | 서지 | ICLR 2026 (검색 결과 기준) | 본문 v4(2025-11) 헤더 없음, Ethics Statement에 "ICLR Code of Ethics"만 언급, comment 없음 | venue 근거 없음 |
| 2510.10114 | 〃 | Benchmarks / 수치 | multi-hop QA (미확인), 인덱싱 시간 77% 이상 절감 | §4: HotpotQA, 2Wiki, MuSiQue, Medical(GraphRAG-Bench); §1 "reduces indexing time by over 77%" | 수정 (벤치마크) / 일치 (수치) |
| 2510.10114 | 〃 | Stated limitations | 구축 오류가 노이즈·모호성 유발 문제의식 | 초록·§1 동기와 일치. 한계 절 없음 | 명시된 한계 없음 (동기 서술은 일치) |
| 2510.13590 | Temporal GraphRAG (TG-RAG) | 서지 | 제목 "Temporal GraphRAG (TG-RAG)", 2025 arXiv | 정식 제목 "RAG Meets Temporal Graphs: Time-Sensitive Modeling and Retrieval for Evolving Knowledge", v1 2025-10-15, venue 표기 없음 | 수정 (제목) |
| 2510.13590 | 〃 | Core | 이중 수준 시간 그래프, 갱신 친화 | 초록: 타임스탬프 관계 KG + 계층 시간 그래프, 새 사실을 추출·병합하는 증분 갱신, 같은 사실의 시점별 값을 별도 엣지로 유지 | 일치 (삭제·무효화는 다루지 않음) |
| 2510.13590 | 〃 | Benchmarks | 미확인 | ECT-QA(자체, 실적 발표 녹취 2020–2024) + 증분 평가 프로토콜; 일반 QA NQ, 2Wiki, HotpotQA (§VI-C) | 수정 |
| 2510.13590 | 〃 | Stated limitations | 기존 평가가 갱신 비용·검색 안정성을 안 본다고 지적 | 초록 일치. 한계 절 없음 | 일치 / 명시된 한계 없음 |
| 2602.02053 | WildGraphBench | 서지 | Findings of ACL 2026 | 본문 v2(2026-02-03) "Preprint. Work in progress.", comment는 GitHub 링크만 | 수정 → venue 근거 없음 |
| 2602.02053 | 〃 | 수치 | 질문 1,100개 | 초록 1,100; 통계표 합계 1,197(단일 667, 다중 191, 요약 339) | 수정 (병기) |
| 2602.02053 | 〃 | Core / 결론 | 단일 사실에서 NaiveRAG가 대부분 GraphRAG보다 우세, 집계 패러다임이 세부 희생 | §4.2: NaiveRAG가 대부분 그래프 변형보다 우세(HippoRAG2만 예외), 다중 사실은 MS GraphRAG(global) 최고, 요약은 NaiveRAG가 최고 F1 | 일치 |
| 2602.02053 | 〃 | Stated limitations | (위 결과를 한계로 기록) | Limitations: 정답 문장이 Wikipedia 편집 합의에 의존, LLM 판정·문장 매칭의 체계적 편향 가능 | 수정 |
| 2607.26497 | BM25 Wins at Scale (v1: Which RAG Paradigm Wins at Scale?) | 서지 | 2026 arXiv | 본문 v3 31 Jul 2026, venue 없음. v1 제목은 로컬 자료에 없음 | 일치 / v1 제목 본문 확인 불가 |
| 2607.26497 | 〃 | Core / 수치 | 약 10M 토큰 이후 BM25가 정확도 주도, 저비용 Pareto 끝점 | 초록·§4: 28단계(약 450배) 코퍼스, 약 10M 토큰에서 BM25가 File-System Agent 추월 후 모든 공유 단계 선두, 전체 규모 격차 약 20점, Pareto 저비용 끝 | 일치 |
| 2607.26497 | 〃 | Benchmarks | 다층 규모 코퍼스 | §3.1: EnterpriseRAG-Bench(511,957 문서, 600.8M 토큰, 500문항) | 수정 |
| 2607.26497 | 〃 | 수치 | 인덱싱 토큰당 최대 24.6 LLM 토큰; 처음 2% 안에서 구축 멈춤 | v3 본문에 24.6·2% 표기 없음. Table 2 기준 LightRAG 73M/3.0M ≈ 24배; 구축 한계 LightRAG 2,254문서(3.0M 토큰, 약 0.5%), MS-GraphRAG 8,750문서(10.6M, 약 1.8%), HippoRAG 2·LinearRAG는 131,876문서(약 155M, 약 26%)까지 | 수정 |
| 2607.26497 | 〃 | Stated limitations | (위 수치를 한계로 기록) | Limitations 절 없음 | 명시된 한계 없음 |
| LazyGraphRAG blog | LazyGraphRAG: Setting a new standard for quality and cost | 서지·수치 | 2024 산업 블로그; 인덱싱 비용 vector RAG와 같고 full GraphRAG의 0.1%; 700배 등은 2차 출처 | 블로그 2024-11-25 게시; "indexing costs are identical to vector RAG and 0.1% of the costs of full GraphRAG"; 글로벌 질의에서 GraphRAG Global과 비슷한 품질에 "more than 700 times lower query cost"; C2 질의 비용 4%에서 전 조건 능가 | 일치 (700배는 본문 확인) |
| LazyGraphRAG blog | 〃 | Benchmarks | 자체 평가 | AP 뉴스 5,590건, 합성 질의 100개(로컬 50·글로벌 50), LLM 쌍대 비교(포괄성·다양성·empowerment) | 보강 |
| LazyGraphRAG blog | 〃 | Stated limitations | 질의 시 비용 발생 | "Looking forward": 모든 그래프 RAG가 lazy여야 하는 것은 아님(요약 인덱스의 QA 외 활용 가치, 요약 인덱스 + lazy 검색 결합이 더 나을 것) | 수정 |
| 2502.12110 | A-Mem (부록) | 서지·핵심 | NeurIPS 2025, Zettelkasten식 노트 연결·memory evolution | arXiv comment "NeurIPS 2025"; §3 Link Generation·Memory Evolution, Zettelkasten 언급 | 일치 |

## 오픈 문제·연구 공백 서술 점검
| 위치 | 우리 서술 | 본문 대조 | 판정 |
|---|---|---|---|
| B-3 #2 | GraphRAG-Bench가 "그래프 밀도-검색 품질 상관"을 보고했다는 2차 정보 | Obs.7은 정성적 관찰(HippoRAG2의 조밀한 그래프가 성능에 기여) | 수정 |
| B-3 #3 | 대부분의 그래프 RAG 실험 코퍼스는 1M 토큰 안팎 | GraphRAG 1–1.7M, KET-RAG 0.6–1.4M, PathRAG 0.18–5M, Unified Framework 0.6–13.5M | 수정 (수십만~수백만 토큰, 최대 약 13.5M) |
| B-3 #4 | 삭제·수정·시간적 무효화는 TG-RAG와 Preprints.org 2025 논문 정도가 다룸 | TG-RAG는 시점별 사실을 별도 엣지로 유지하고 증분 병합만 함. 삭제·무효화는 다루지 않음. Preprints.org는 본문 확인 불가. Unified Framework O1이 정적 코퍼스 가정을 오픈 문제로 명시 | 수정 |
| B-3 #5 | 구축 품질-하류 성능 관계 분해 연구 부족 | 부분 분석 존재: HippoRAG App. D(OpenIE 내재 평가)·App. F(오류 분석), RAG vs GraphRAG §4.7(구축 LLM 교체). Unified Framework O2가 질의 전 그래프 품질 평가를 미해결로 명시 | 수정 (부분 존재 명시, 결론 유지) |
| B-4 #1 | 인덱싱 전 의사결정 연구 없음 | Unified Framework O2가 같은 문제를 오픈 문제로 명시 → 공백 주장과 일치, 근거 보강 | 일치 |
| B-4 #2 | WildGraphBench는 Wikipedia 문장을 정답으로 사용 | 정답 문장은 있으나 판정은 LLM·문장 매칭(저자 명시 한계) | 보강 |
| B-4 #4 | 질의 분포 적응형 부분 그래프 구축 연구 미확인 | GraphRAG §6.2와 LazyGraphRAG 블로그가 just-in-time 요약·하이브리드 인덱스를 향후 방향으로만 제시, KET-RAG §7도 적응형 패러다임을 향후 과제로 언급 → 공백 주장과 모순 없음 | 일치 |


---

## 배치 b03a

# b03a 검증표 — 03_query_rerank_compression.md 헤더·검증 메모 + 축 A (A.1–A.4)

근거: 로컬 txt/<arXivID>.txt(pdftotext)와 meta/arxiv_meta.json만 사용(웹 미사용). 23편 모두 본문 파일이 있었다. "타 논문 인용"은 로컬에 있는 다른 논문의 참고문헌 항목을 뜻하며, 1차 근거(본문 첫 페이지, arXiv comment)가 아니라 보조 근거다.

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 헤더 | — | 검증 방법 메모 | WebSearch 교차 확인, 고전 논문은 "ID 재확인 권장", 2차 수치는 **미확인** | 이번 배치에서 arXiv 본문(pdftotext)과 arXiv API 메타데이터로 전수 확인함 | 수정(메모 교체) |
| 2212.10496 | Precise Zero-Shot Dense Retrieval without Relevance Labels | 서지/ID | 2023 / ACL (ID 재확인 권장) | meta: 2022-12-20 v1, comment·journal_ref 없음. 본문 첫 페이지에 venue 표기 없음. 타 논문 인용(2304.09542 참고문헌): "In ACL 2023" | 수정(ID 일치. arXiv 2022, venue는 본문·comment 근거 없음, 타 논문 인용상 ACL 2023) |
| 2212.10496 | 〃 | Core contribution | 가상 문서를 생성한 뒤 그 임베딩으로 검색 | 초록: InstructGPT가 가상 문서를 생성하고 Contriever가 인코딩, "dense bottleneck"이 잘못된 세부를 걸러냄 | 일치 |
| 2212.10496 | 〃 | Benchmarks | TREC DL19/20, BEIR 일부, Mr.TyDi | 4절 Setup: DL19/20, BEIR 저자원 6개, Mr.TyDi(sw, ko, ja, bn) | 일치(세부 보강) |
| 2212.10496 | 〃 | Stated limitations | hallucination 의존, 지식 밖 쿼리 취약 | Limitations 절 없음. 6절 Conclusion: 검색 로그가 쌓이면 지도학습 retriever로 점진 전환하라는 실용 제언. 본문은 가짜 세부를 인코더가 걸러낸다고 주장함. "지식 밖 쿼리 취약"은 2505.12694가 Q2D 계열 QE에서 실증한 것이며 HyDE 저자가 밝힌 한계가 아님 | 수정(명시된 한계 없음. 해당 내용은 후속 연구의 지적으로 분리) |
| 2303.07678 | Query2doc: Query Expansion with Large Language Models | 서지 | 2023 / EMNLP (ID 재확인 권장) | meta comment "Accepted to EMNLP 2023" | 일치(ID 확인됨) |
| 2303.07678 | 〃 | Core contribution | few-shot pseudo-doc을 쿼리에 이어 붙임(BM25, dense) | 초록: few-shot 프롬프트로 pseudo-doc 생성 후 쿼리 확장. BM25 3–15% 향상 | 일치 |
| 2303.07678 | 〃 | Benchmarks | MS MARCO, TREC DL | 3.1절: MS MARCO dev, DL19/20, BEIR 저자원 5개(DBpedia, NFCorpus, SciFact, TREC-COVID, Touché2020) | 수정(BEIR 5개 추가) |
| 2303.07678 | 〃 | Stated limitations | latency. 강한 dense에서는 이득 감소 | Limitations 절: LLM 추론 지연 >2000ms, 확장 후 역색인 검색도 느려짐(표 6). 3.2절: SimLM/E5(KD) 조합에서는 이득 감소, NFCorpus·SciFact에서 소폭 하락 | 일치 |
| 2305.14283 | Query Rewriting for Retrieval-Augmented Large Language Models | 서지 | 2023 / EMNLP (ID 재확인 권장) | meta comment "EMNLP2023". 정식 제목은 "...Large Language Models" | 일치(ID 확인됨, 제목 표기 보정) |
| 2305.14283 | 〃 | Core contribution | 소형 rewriter를 reader 피드백 RL(PPO)로 학습 | 3.2절: T5-large rewriter, warm-up 후 PPO, LLM reader 성능을 보상으로 사용 | 일치 |
| 2305.14283 | 〃 | Benchmarks | HotpotQA, AmbigNQ, PopQA, MMLU | 1절·5절: HotpotQA, AmbigNQ, PopQA, MMLU. 검색기는 Bing | 일치 |
| 2305.14283 | 〃 | Stated limitations | 보상이 reader에 종속되어 재학습 필요 | Limitations: (i) 학습을 추가하므로 일반화와 특화 사이 trade-off, few-shot 대비 직접 transfer 약화. (ii) 다중 호출 agent가 아닌 단일 턴 설계. (iii) 웹 검색엔진 의존 | 수정(저자 서술로 교체) |
| 2309.08541 | When do Generative Query and Document Expansions Fail? A Comprehensive Study Across Methods, Retrievers, and Datasets | 서지 | 2023→EACL 2024 Findings (venue 미확인) | meta comment "EACL 2024 camera ready". Findings 여부는 로컬 근거 없음 | 수정(EACL 2024. Findings 표기 삭제) |
| 2309.08541 | 〃 | 수치 | 기법 11 × 데이터셋 12 × retriever 24 | 초록: eleven techniques, twelve datasets, twenty-four retrieval models | 일치 |
| 2309.08541 | 〃 | Core contribution | 약한 ranker에는 도움, 강한 ranker에는 해 | 초록, 8절 Conclusion | 일치 |
| 2309.08541 | 〃 | Benchmarks | BEIR 등 12개 | 그림 2: ArguAna, FiQA, GooAQ Tech, NFCorpus, Quora, SciFact-Refute, TREC-CT, TREC-DL19, TREC-DL20, ToT, Touché, WikiQA | 수정(목록 구체화) |
| 2309.08541 | 〃 | Stated limitations | "언제 확장할지" 판정기 부재 | Limitations: zero-shot ranker만 평가(확장 대응 학습 안 함). 확장 필요 여부를 정하려면 대상 도메인의 라벨 평가 데이터가 필요함 | 수정(저자 서술 반영. 판정기 부재 서술과는 모순 없음) |
| 2505.12694 | LLM-based Query Expansion Fails for Unfamiliar and Ambiguous Queries | 서지 | 2025 / SIGIR short | meta comment "SIGIR 2025 short paper track". 본문 ACM 서식에 SIGIR '25 | 일치 |
| 2505.12694 | 〃 | Core contribution | 지식 부족(존재하지 않는 엔티티), 모호성(인기 해석 편향) | 초록, 3절, 그림 2 popularity bias | 일치 |
| 2505.12694 | 〃 | Benchmarks | sparse/dense, 다수 데이터셋 | 3.1절: RQ1은 NQ, TriviaQA, MS MARCO, BioASQ. RQ2는 AmbigDocs, AmbigQA, TREC Web 09–12. Retriever: BM25, Contriever, E5-base-v2. QE: Q2D, Q2E, GaQR | 수정(구체화) |
| 2505.12694 | 〃 | Stated limitations | 해법 없음 | 5절 Conclusion: 향후 (i) PRF, CoT, 다중 확장 파이프라인에서 재검증 (ii) LLM 지식과 쿼리 모호성을 평가해 adaptive QE에 쓰는 모듈 개발 | 수정(저자 future work 반영) |
| 2506.09260 | ThinkQE: Query Expansion via an Evolving Thinking Process | 서지 | 2025 / arXiv | meta comment "EMNLP 2025 Findings"(v2) | 수정(EMNLP 2025 Findings) |
| 2506.09260 | 〃 | 수치 | BRIGHT StackExchange 평균 nDCG@10 34.9, Rank-K-32B 37.9 | 표 4: ThinkQE(R1-14B) 34.9, Rank-K-32B 37.9. 다른 백본은 최고 36.0(Phi-4-Reasoning-14B) | 일치(백본 명시) |
| 2506.09260 | 〃 | Core contribution / Benchmarks | thinking 확장 + corpus 피드백(top-5), DL19/20, BRIGHT SE | 3절: top-5 문서로 프롬프트, 중복 필터·누적. 4절: DL19, DL20, BRIGHT StackExchange 7개 | 일치 |
| 2506.09260 | 〃 | Stated limitations | 다중 라운드 비용, reranker보다 낮음 | Limitations: 단일 확장 대비 높은 추론 지연·비용, 영어 웹 검색만 실험(다국어 미검증). "reranker보다 낮음"은 우리 관찰(표 4) | 수정(저자 서술 + 우리 관찰 구분) |
| 2310.11511 | Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection | 서지 | 2024 / ICLR (ID 재확인 권장) | meta: v1 2023-10, comment에 venue 없음. 본문 헤더 "Preprint." 타 논문 인용(2501.12835, 2501.14342, 2403.14403 참고문헌): ICLR 2024 | 수정(ID 일치. venue는 본문 근거 없음, 타 논문 인용상 ICLR 2024) |
| 2310.11511 | 〃 | Benchmarks | PopQA, TriviaQA, PubHealth, ARC, ASQA 등 | 4절: PopQA, TriviaQA, PubHealth, ARC-Challenge, Biography(FactScore), ASQA(6개 과제) | 일치(보강) |
| 2310.11511 | 〃 | Stated limitations | critic 라벨링 비용 큼, 2501.12835가 효율 대비 이득이 제한적이라고 지적 | Limitations 절 없음. Ethical Concerns: 인용으로 완전히 뒷받침되지 않는 출력이 여전히 나올 수 있음. critic 라벨은 GPT-4로 만든 뒤 증류하며, 저자는 오히려 RLHF 대비 학습 비용이 낮다고 주장(2절). **2501.12835는 Self-RAG를 평가 대상 8개 방법에 넣지 않음**(표 1: IRCoT, AdaptiveRAG, DRAGIN, FLARE, Rowen×3, SeaKR) | 오류(근거 인용 잘못. 삭제·교체) |
| 2305.06983 | Active Retrieval Augmented Generation | 서지 | 2023 / EMNLP (ID 재확인 권장) | meta comment "EMNLP 2023" | 일치(ID 확인됨) |
| 2305.06983 | 〃 | Benchmarks | 2Wiki, StrategyQA, ASQA, WikiAsp | 4절: 2WikiMultihopQA, StrategyQA, ASQA, WikiAsp | 일치 |
| 2305.06983 | 〃 | Stated limitations | 임계값 수작업, 반복 생성 비용 | 9절 Limitations: Wizard of Wikipedia·ELI5에서 유의미한 이득 없음. 생성과 검색을 교차하면 LM을 여러 번 활성화해야 하고 캐시 없이 재계산하므로 오버헤드 증가. 임계값은 6절에서 민감도 분석만 함(한계로 명시하지 않음) | 수정 |
| 2403.10081 | DRAGIN: Dynamic Retrieval Augmented Generation based on the Information Needs of Large Language Models | 제목 | "...based on the Real-time Information Needs of LLMs" | meta·본문 제목: "...based on the Information Needs of Large Language Models" | 수정 |
| 2403.10081 | 〃 | 서지 | 2024 / ACL | meta comment 없음, 본문에 venue 표기 없음. 타 논문 인용(2606.29959, 2410.08815): 62nd ACL(2024) | 수정(본문 근거 없음, 타 논문 인용상 ACL 2024) |
| 2403.10081 | 〃 | Core / Benchmarks | RIND(불확실성×attention 중요도×의미성), QFS. 2Wiki, HotpotQA, StrategyQA, IIRC | 3절: entropy, max attention, stopword 기반 semantic 지시자. 4절 데이터셋 4개 | 일치 |
| 2403.10081 | 〃 | Stated limitations | white-box 전용 | 7절 Limitations: self-attention 점수를 주지 않는 API에는 적용 불가 | 일치 |
| 2406.19215 | SeaKR: Self-aware Knowledge Retrieval for Adaptive Retrieval Augmented Generation | 서지 | 2024 arXiv → ACL 2025 | meta comment 없음. 타 논문 인용(2606.29959 [34]): ACL 2025 long, 2025.acl-long.1312 | 수정(본문 근거 없음, 타 논문 인용상 ACL 2025) |
| 2406.19215 | 〃 | Benchmarks | 2Wiki, HotpotQA, IIRC, NQ 등(세부 미확인) | 4절: 복합 QA는 2WikiMultiHopQA, HotpotQA, IIRC. 단순 QA는 NQ, TriviaQA, SQuAD | 수정(확인 완료) |
| 2406.19215 | 〃 | Stated limitations | 내부 상태 필요, 다중 샘플링 비용 | Limitations: (1) 내부 상태 접근이 필요해 오픈 모델만 가능 (2) 단답 QA 위주, long-form 등 미평가 (3) Gram determinant 계산에 pseudo-generation 20회 필요 | 일치(보강) |
| 2403.14403 | Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity | 서지 | 2024 / NAACL (ID 재확인 권장) | meta comment "NAACL 2024" | 일치(ID 확인됨) |
| 2403.14403 | 〃 | Benchmarks | SQuAD, NQ, TriviaQA, MuSiQue, HotpotQA, 2Wiki | 4절 | 일치 |
| 2403.14403 | 〃 | Stated limitations | silver label 의존, 3단계로 거칠게 나눔 | Limitations: 자동 라벨링이 틀릴 수 있어 복잡도 주석 데이터셋 필요, ideal 분류기(표 1)와 격차. 3.2절: 더 세분된 복잡도는 future work | 일치 |
| 2501.12835 | Adaptive Retrieval Without Self-Knowledge? Bringing Uncertainty Back Home | 서지 | 2025 / ACL | meta comment에 venue 없음, 본문에 venue 표기 없음. 타 논문 인용(2604.26649 참고문헌): ACL 2025 long | 수정(본문 근거 없음, 타 논문 인용상 ACL 2025) |
| 2501.12835 | 〃 | 수치 | 35개(최신 8 + UE 27), QA 6개 | 초록·1절: 35 = 8 + 27, 데이터셋 6개, 지표 10개 | 일치 |
| 2501.12835 | 〃 | Benchmarks | 6 QA | 4절: SQuAD v1.1, NQ, TriviaQA, MuSiQue, HotpotQA, 2WikiMultiHopQA | 일치(구체화) |
| 2501.12835 | 〃 | Stated limitations | oracle 격차, 지식 경계 미해결 | Limitations: LLaMA3.1-8B-instruct 한 모델만 사용, 6개 표준 QA(도메인 특화 없음). oracle(Ideal) 격차는 표 1에 나타남(예: 첫 데이터셋 Best UE 0.512 vs Ideal 0.608). OOD에서 self-knowledge 크게 저하(8절) | 수정(저자 한계와 결과 해석 구분) |
| 2501.14342 | Chain-of-Retrieval Augmented Generation | 서지 | 2025 / arXiv (v3 2025-10) | meta comment "Accepted by NeurIPS 2025". 본문 첫 페이지 각주 "39th Conference on NeurIPS 2025" | 수정(NeurIPS 2025) |
| 2501.14342 | 〃 | Core / 수치 | rejection sampling 체인 합성 + SFT. greedy/best-of-N/tree search. EM 10점 이상, KILT SOTA | 초록, 3절 | 일치 |
| 2501.14342 | 〃 | Benchmarks | 멀티홉 QA, KILT | 4절: 2WikiMultihopQA, HotpotQA, Bamboogle, MuSiQue, KILT | 일치(구체화) |
| 2501.14342 | 〃 | Stated limitations | 테스트 타임 컴퓨트 비용, 체인 합성이 기존 정답에 의존 | 7절 Limitations: 짧고 검증하기 쉬운 답 과제(멀티홉 QA, entity linking)만 다룸. long-form은 평가 지표 부재로 미해결. 정답 의존은 초록에서 확인되나 한계로 명시하지는 않음 | 수정 |
| 2503.09516 | Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning | 서지 | 2025 / arXiv (venue 미확인) | 본문 페이지 헤더 "Published as a conference paper at COLM 2025"(v5) | 수정(COLM 2025) |
| 2503.09516 | 〃 | Benchmarks | NQ, TriviaQA, PopQA, HotpotQA, 2Wiki, MuSiQue, Bamboogle | 4절 데이터셋 7개 | 일치 |
| 2503.09516 | 〃 | 수치 | 버전별로 다름(v2 26%, v3 41%) | 로컬 v5만 있음. 초록·5절은 24%(Qwen2.5-7B)/20%(3B), 1절 기여 목록은 41%/20%로 같은 버전 안에서도 다름. v2/v3 수치는 로컬에 없음 | 수정(v5 내부 불일치로 교체. v2/v3 수치는 본문 확인 불가) |
| 2503.09516 | 〃 | Stated limitations | 검색 횟수·비용 제약 없음 | Limitations 절 없음. 3절: outcome 기반 규칙 보상만 쓰고 format reward 없음(보상에 비용 항 없음은 본문에서 확인). 6절 future work: 더 정교한 보상, 불확실성 기반 동적 검색 조정 | 일치(근거 보강) |
| 2503.00223 | DeepRetrieval: Hacking Real Search Engines and Retrievers with Large Language Models via Reinforcement Learning | 서지 | 2025 / arXiv | meta comment 없음, 본문 헤더 "Preprint." | 일치(venue 근거 없음) |
| 2503.00223 | 〃 | Core | retrieval 지표를 보상으로 쓰는 RL query rewriting | 초록: 참조 쿼리 없이 retrieval metric 보상. 문헌 검색 recall 65.07%(기존 24.68%), 임상시험 검색 63.18%(32.11%) | 일치(수치 추가) |
| 2503.00223 | 〃 | Benchmarks | 문헌 검색, BEIR 등(세부 미확인) | 그림 1, 4절: PubMed·ClinicalTrials.gov(실제 검색엔진, Recall@3K), evidence-seeking NQ/TriviaQA/SQuAD, classic IR HotpotQA/FEVER/NFCorpus/SciFact/MS MARCO 도메인 subset, SQL Spider/BIRD | 수정(확인 완료) |
| 2503.00223 | 〃 | Stated limitations | 세부 미확인 | Limitations 절 없음. 4절: MS-H subset에서는 이득이 작음(기반 모델의 도메인 지식 부족 또는 retriever 천장으로 추정). 도메인 적응은 future work | 수정 |
| 2604.15621 | Rethinking the Necessity of Adaptive Retrieval-Augmented Generation through the Lens of Adaptive Listwise Ranking | 서지 | 2026 / arXiv | meta: 7쪽, venue 없음 | 일치(venue 근거 없음) |
| 2604.15621 | 〃 | Core | passage dropout listwise로 k를 적응. 약한 모델은 노이즈 필터, 강한 모델은 효율 최적화 | 초록, 6절. 2단계 progressive distillation도 포함 | 일치(보강) |
| 2604.15621 | 〃 | Benchmarks | 세부 미확인 | 표 I: ASQA(GTR), QAMPARI(GTR), ELI5(Sphere, BM25), LLM 8종. EM/F1/Claim Recall | 수정(확인 완료) |
| 2604.15621 | 〃 | Stated limitations | 최적 k가 작업·모델마다 달라 동적 보정 필요 | 5.B절의 "fixed-k의 오류"는 AdaRankLLM이 해결하려는 동기이지 이 방법의 한계가 아님. 별도 Limitations 절 없음. 6절 Conclusion: 이론적 Oracle과의 격차가 남아 최적 검색은 미해결, agentic 패러다임은 future work | 오류(한계 서술 교체) |
| 2604.26649 | When to Retrieve During Reasoning: Adaptive Retrieval for Large Reasoning Models | 서지 | 2026 / arXiv | meta comment·본문: SIGIR 2026 (Melbourne) | 수정(SIGIR 2026) |
| 2604.26649 | 〃 | Core | REINFORCE, 보상 = 정확도 − 호출 수·latency | 4.3절: contextual bandit 정책을 QueryGen과 함께 REINFORCE로 학습, R = F1 − λ1·n_ret − λ2·t_latency. step 단위 불확실성 RSUS | 일치(보상은 정확도가 아니라 F1) |
| 2604.26649 | 〃 | Benchmarks/수치 | 미확인 | 초록: MuSiQue, HotpotQA, 2WikiMultiHopQA. 표준 RAG 대비 F1 +10.1%p(9.0–11.8), IRCoT 대비 검색 호출 −47%, MuSiQue F1 71.2%, 질문당 1.8회 | 수정(확인 완료) |
| 2604.26649 | 〃 | Stated limitations | 미확인 | 7.5절 Limitations: completion-only(API) 모델은 F1 2–3% 낮음(o1 68.9 vs R1 71.2), 관련 정보가 없으면 검색 낭비, 학습 예제 19,938개, RSUS 오버헤드 8%, 30K 토큰 초과 시 KV-cache 문제, 주입 3회 이상이면 성능 저하 | 수정(확인 완료) |
| 2606.29959 | Know Before You Fetch: When Does Confidence-Based Retrieval-Budget Allocation Pay Off? | 제목 | "Know Before You Fetch: Calibrated Retrieval-Budget Allocation for RAG" | meta·본문 v2(2026-11-18) 제목 | 수정 |
| 2606.29959 | 〃 | 서지 | 2026 / arXiv | meta comment: 25쪽, venue 없음 | 일치(venue 근거 없음) |
| 2606.29959 | 〃 | Core | closed-book, k=1, k=5, abstain 중 선택, 보정 확률 사용 | v2 초록·1절: closed-book confidence로 {0, 1, 5} passage를 배분하고, 같은 평균 passage cap의 최선 단순 정책(고정 깊이 또는 고정 깊이의 랜덤 혼합)과 비교. **abstain 옵션 없음**. 5.5절: 보정해도 결과 불변(순서 보존이라 선택된 쿼리가 같음, 50개 조합 모두 동일). 보정은 고정 비용비와 비교하는 규칙에서만 의미 | 오류(v2 기준 서술 교체) |
| 2606.29959 | 〃 | Benchmarks | 미확인 | 초록: TriviaQA, NQ, MS MARCO. Qwen3-8B/32B reader. per-question passage pool과 3.5M passage Wikipedia 색인. 약 2,000문항 설정 7개 + 추가 15개 | 수정(확인 완료) |
| 2606.29959 | 〃 | 수치 | retrieval-call AUC는 binary gating과 비슷, Qwen3-8B에서 latency 약 27% 증가(**미확인**) | v2 본문에서 AUC 비교나 27% 수치를 찾지 못함. 대신 6절 표 4: probe latency를 포함하면 pool 설정에서는 이득이 사라지고(예: TriviaQA 8B pool, B=1에서 −0.040), Wikipedia 4개 설정 중 3개에서만 이득 유지. {0,1} gate의 손익분기 skip 비율 0.73–0.96 | 수정(v2 수치로 교체. 옛 수치는 본문 확인 불가) |
| 2606.29959 | 〃 | Stated limitations | (위 수치 칸에 섞여 있음) | 7.3절: 단답 factoid QA와 frozen 오픈 reader만 다룸. long-form, 대화형, multi-step 검색은 다를 수 있음. Qwen3 한 계열, 설정마다 retriever 1개. MS MARCO 라벨 오류율 12.5% | 수정 |
| 2607.24010 | When Should Active RAG Retrieve? A Budget-Aware Evaluation of Utility, Calibration, and Cost | 서지 | 2026 / arXiv (CMU) | meta comment: KDD 2026 Workshop on Evaluation and Trustworthiness of Agentic AI. 저자 소속 CMU 외 | 수정(KDD 2026 워크숍) |
| 2607.24010 | 〃 | Core | budgeted marginal utility 정식화, 단순 UE·retrieval score가 학습 router와 대등, 임계값 transfer 실패 | 초록, 1절: HotpotQA에서 흔한 trigger가 50% 목표를 split의 40–100%에서 초과 | 일치 |
| 2607.24010 | 〃 | 수치 | 2Wiki 검색 시 오답 비율 8.8%(**미확인**) | 표 1: Qwen2.5-1.5B, 2Wiki의 retrieval harm 8.8%(no-retrieval 정답이 retrieval로 오답이 되는 비율). HotpotQA 4.0, MuSiQue 1.5 | 일치(미확인 해제) |
| 2607.24010 | 〃 | Benchmarks | 2Wiki 등 | 3절: HotpotQA distractor, 2WikiMultiHopQA, MuSiQue(제공된 후보 문단, BM25). Qwen2.5 위주, SmolLM2·Granite 일부 | 수정(구체화) |
| 2607.24010 | 〃 | Stated limitations | long-form, multi-step, 공식 Self-RAG/FLARE 재현 없음. latency·달러·에너지 제외 | Limitations 절(같은 내용 + 전체 Wikipedia 색인 아님, 소형 모델) | 일치 |
| 2606.02245 | When Knowledge Is Not Free: Cost-Aware Evidence Selection in Retrieval-Augmented Generation | 서지 | 2026 / arXiv | meta comment 없음 | 일치(venue 근거 없음) |
| 2606.02245 | 〃 | Core | 접근 비용 tier 부여, LLM이 검색 여부·tier·중단 결정 | 초록: MS MARCO v2.1에 access-friction tier 부여, 정적 selector와 agentic 설정 | 일치 |
| 2606.02245 | 〃 | Benchmarks | 미확인 | 1절·3절: HotpotQA, NQ, TriviaQA, MedQA-US, MMLU-Med(5개) | 수정(확인 완료) |
| 2606.02245 | 〃 | Stated limitations | 모든 경우에 우세한 고정 selector 없음 | 이 내용은 초록의 결과(한계 아님). 8절 Limitations: 비용 tier는 금액·라이선스·latency의 실측이 아닌 대리값, 도메인 분류 자동 주석 오류 가능, QA 5개와 표본 subset만 평가 | 수정 |
| 2610.04899 | Rewrite What Matters: Adaptive Multilingual Query Rewriting for Reasoning via Agentic Reinforcement Learning | 주제 적합성 | "다국어 rewriting의 RL 학습"을 query-side RAG 흐름으로 인용 | 초록·5절: mRewriter-R1은 **검색 없이** 다국어 추론 입력을 다중 턴 operator 선택(번역, 구조, 의미)으로 재작성. 벤치마크 MGSM, BELEBELE, XCOPA(정확도). 본문에서 "retriev"는 관련연구 1곳과 참고문헌 2곳에만 나옴 | 오류(RAG 검색용 rewriting 아님. 인접 연구로 격하) |
| 2610.04899 | 〃 | 서지/한계 | — | 헤더 "Preprint", meta comment 없음(venue 근거 없음). Limitations 절 없음(7절 Conclusion에도 한계 서술 없음) | 명시된 한계 없음 |
| 2610.10955 | Learning Multi-Step Query Rewriting via Corpus Feedback for Conversational Search | 주제/Core | 대화형 rewriting의 RL 학습 | 초록: CQR을 순차 검색 문제로 재정의. 재작성 operator 3종 + stop, SFT 후 retrieval-quality 단일 보상 RL, 인간 재작성 라벨 없음. TopiOCQA, QReCC, zero-shot CAsT | 일치 |
| 2610.10955 | 〃 | 서지/한계 | — | meta comment 없음(venue 근거 없음). 5절 Conclusions and Future Work: step penalty나 latency 비용을 보상에 넣으면 cost-aware 조기 종료 정책이 가능하다고 future work로 제시(현재 보상에 비용 항 없음) | 확인(한계는 future work 형태) |
| 2610.33768 | SkillQR: Agentic Skill Discovery for Reason-Intensive Query Rewriting | Core | 재작성 전략으로 라우팅(세부 **미확인**) | 초록·4절: 검색 대상 corpus(retriever 임베딩의 KMeans 클러스터)에서 재작성 skill 세트(최대 5개)를 오프라인으로 발견하고, 추론 시 LLM router가 쿼리별 skill을 고름. 학습 데이터 없음. 최종 쿼리는 원 쿼리 + skill 기반 확장을 concat(PRF top 문서 사용). "확장 안 함" 선택지는 없음 | 수정(미확인 해제, 세부 확인) |
| 2610.33768 | 〃 | Benchmarks/수치 | — | BRIGHT, R2MED. Retriever ReasonIR, Reason-Embed, Diver-Retriever. Router 분석: LLM routing 42.9 nDCG@10 vs 쿼리별 최선 skill 상한 53.4 | 확인 |
| 2610.33768 | 〃 | 서지/한계 | — | meta comment 없음(venue 근거 없음). Limitations: LLM proposer/judge/rewriter의 편향·도메인 지식·hallucination, router와 상한의 격차, corpus 변화에 맞춘 skill 갱신 필요 | 확인 |
| (A.3.6) Dynamic Search-R1 | — | 존재 | **미확인** | 로컬 본문 없음. 2604.26649 2.2절과 참고문헌 [7]: Hashemi, Rühle, Rajmohan 2026, "Cost-Aware Retrieval-Augmentation Reasoning Models with Adaptive Retrieval Depth", WWW 2026. PPO/GRPO에 token-cost penalty를 넣어 검색 깊이를 조절 | 수정(타 논문 인용으로 존재 확인, 본문 확인 불가) |

## 오픈 문제·연구 공백 서술 점검
| 위치 | 우리 서술 | 원문 대조 | 판정 |
|---|---|---|---|
| A.1 마지막 줄 | 다국어·대화형 rewriting의 RL 학습(2610.04899, 2610.10955) | 2610.04899는 검색 없는 다국어 추론용 rewriting | 수정 |
| A.3.2 | 임계값이 데이터셋·모델에 따라 달라짐(2607.24010, 2604.15621) | 2607.24010은 임계값 transfer 실패를 보임(일치). 2604.15621이 보인 것은 "최적 고정 k"의 변동 | 수정(표현 정밀화) |
| A.3.3 | 2607.24010과 2606.29959 둘 다 latency·달러·에너지 제외 | 2606.29959 v2는 6절에서 probe latency를 H100에서 실측해 비교에 포함. 달러·에너지 언급은 없음 | 수정 |
| A.3.5 | long-form 등 평가 드묾(2607.24010 명시) | 2606.29959 7.3, SeaKR, CoRAG 7절도 long-form 미평가를 명시. 예외: 2604.15621(ASQA, QAMPARI, ELI5), Self-RAG(ASQA, Bio) | 수정(근거 보강, 예외 명시) |
| A.3.6 | Search-R1과 CoRAG는 outcome reward | CoRAG는 RL이 아니라 rejection sampling + SFT. Search-R1은 outcome reward(일치). 2610.10955도 비용 항이 없고 future work로 제시 | 수정 |
| A.4.1 | 가장 가까운 작업은 SkillQR(세부 미확인) | SkillQR은 corpus 기반 skill로 라우팅하지만 retriever 강도에 따른 "확장 여부" 판단은 없음. 2505.12694도 이런 모듈을 future work로 제시 | 공백 유지(근거 강화) |
| A.4.2 | 2606.29959는 보정된 확률을 인터페이스로 제안 | v2 5.5절: 분위수 임계값에서는 보정해도 결정이 바뀌지 않음. 보정은 고정 비용비 규칙에서만 의미가 있음 | 오류(서술 교체. 공백 자체는 유지) |
| A.4.3 | multi-hop 단계별 budget-aware 정책이 비어 있음 | 2604.26649가 step 단위 when-to-retrieve에 호출 수·latency 비용을 보상에 넣었음. 다만 주입 3회 이상에서 저하되고 frontier 평가 프로토콜은 없음 | 수정(가장 가까운 작업 추가, 공백 범위 축소) |
| A.4.4 | black-box 신호 비교 부족 | 2604.26649: completion-only 모델에서 F1 2–3% 하락. 2501.12835는 logit/consistency UE 27종을 비교했지만 open 모델(LLaMA3.1-8B)만 썼음 | 유지(근거 보강) |
| A.4.5 | 다중 해석 확장 부재 | 2505.12694 결론과 일치 | 일치 |


---

## 배치 b03b

# b03b 검증표 — 03_query_rerank_compression.md, 축 B (B.1–B.4, 교차 축 관찰)

근거는 모두 로컬 txt/<arXivID>.txt (pdftotext)와 meta/arxiv_meta.json. 웹은 쓰지 않음.
범위 밖이라 검증하지 않은 인용: SeaKR, 2607.24010, 2606.29959, 2606.02245 (교차 축 관찰과 B.3.3). AdaRankLLM(2604.15621)은 B.4.3에 쓰인 서술만 초록으로 확인함.

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2304.09542 | Is ChatGPT Good at Search? Investigating Large Language Models as Re-Ranking Agents | 서지 | 2023 / EMNLP (ID 재확인 권장) | arXiv comment "EMNLP 2023". 본문 헤더에는 venue 표기 없음. 제목 확인 | 일치 (ID 확인) |
| 2304.09542 | 〃 | Core | sliding-window permutation 생성, NovelEval | 초록과 서론: permutation generation, NovelEval, permutation distillation(440M 모델이 BEIR에서 3B supervised보다 우수) | 일치 (distillation 보강) |
| 2304.09542 | 〃 | Benchmarks | TREC DL19/20, BEIR, Mr.TyDi | Fig.1 캡션 "TREC, BEIR, and Mr.TyDi" | 일치 |
| 2304.09542 | 〃 | Limitations | 독점 API 비용, 위치 편향, latency | Limitations 절: 분석이 독점 모델(ChatGPT/GPT-4) 중심. 성능 상한이 1단계 recall에 묶임. 초기 passage 순서에 매우 민감. latency는 서술되지 않음 | 수정 |
| 2310.09497 | A Setwise Approach for Effective and Highly Efficient Zero-shot Ranking with Large Language Models | 서지 | 2024 / SIGIR (ID 재확인 권장) | 첫 쪽 ACM 각주 "SIGIR '24, July 14–18, 2024". comment "SIGIR2024 full paper" | 일치 (ID 확인) |
| 2310.09497 | 〃 | Benchmarks | TREC DL, BEIR | §4: TREC DL 2019/2020 + BEIR 8종(Table 3: Covid, NFCorpus, Touche, DBPedia, SciFact, Signal, News, Robust04) | 일치 (세부 보강) |
| 2310.09497 | 〃 | Limitations | 정렬 알고리즘에 따라 추론 횟수가 달라짐 | 한계 절은 없음. §5: 비교 문서 수 c가 커지면 프롬프트 길이 제한(512 토큰) 때문에 효과가 떨어질 수 있음. logits 방식은 오픈 모델에서만 가능. Conclusion: 더 많은 LLM에서 평가하는 일을 future work로 둠 | 수정 |
| 2312.02724 | RankZephyr: Effective and Robust Zero-Shot Listwise Reranking is a Breeze! | 서지 | 2023 / arXiv (ID 재확인 권장) | v1 2023-12-05, comment 없음, 본문 헤더 없음 | 일치 (ID 확인, venue 근거 없음) |
| 2312.02724 | 〃 | Benchmarks | TREC DL19–23, BEIR 일부 | 초록과 §5: DL19–DL22, BEIR NEWS·COVID, NovelEval. DL23은 본문에 없음 | 수정 |
| 2312.02724 | 〃 | Limitations | teacher(GPT-4) 의존, 윈도우 제약 | Limitations 절 없음. Conclusion에도 한계 서술 없음 | 수정 (명시된 한계 없음) |
| 2406.15657 | FIRST: Faster Improved Listwise Reranking with Single Token Decoding | 서지 | 2024 / EMNLP | arXiv comment "Preprint". 본문에 EMNLP 표기 없음 | 수정 (venue 근거 없음) |
| 2406.15657 | 〃 | 수치 | latency 약 50% 감소 | 초록과 서론: "lowers the latency of LLM rerankers by 50%" | 일치 |
| 2406.15657 | 〃 | Benchmarks | BEIR, TREC DL | Table 1·2는 BEIR만 사용. TREC DL 평가는 FIRST 본문에 없음(재현 연구 2411.05508이 DL19–22로 확장) | 수정 |
| 2406.15657 | 〃 | Limitations | 첫 토큰 순위와 전체 순열 불일치 가능 | Limitations 절: 학습 데이터가 GPT-4 라벨뿐(MS MARCO 사람 라벨은 실험하지 않음). 영어만 다룸. 식별자를 알파벳으로 써서 윈도우가 20 이하로 제한됨 | 수정 |
| 2411.05508 | An Early FIRST Reproduction and Improvements to Single-Token Decoding for Fast Listwise Reranking | 서지 | (FIRST 행에서 언급) | v2 2024-11, comment 없음 | 확인 (venue 근거 없음) |
| 2411.05508 | 〃 | 내용 | "개선점을 보고" | 초록: DL19–22로 확장, 측정한 latency 이득은 21–42%(FIRST가 주장한 50%보다 작음). backbone을 바꿔 원 구현보다 높은 효과. LM pre-training이 FIRST fine-tuning을 방해할 수 있다는 의문 제기 | 일치 (수치 보강) |
| 2407.12883 | BRIGHT: A Realistic and Challenging Benchmark for Reasoning-Intensive Retrieval | 서지 | 2024 arXiv → ICLR 2025 (venue 재확인 권장) | 첫 쪽 헤더 "Published as a conference paper at ICLR 2025" | 일치 (확인) |
| 2407.12883 | 〃 | Core/수치 | StackExchange, 코딩, 수학 | 초록: 1,384개 질의, 12개 데이터셋(§2). SFR-Embedding-Mistral이 MTEB 59.0에서 BRIGHT 18.3으로 떨어짐 | 일치 (보강) |
| 2407.12883 | 〃 | Limitations | 문서 수와 도메인 범위. RAG 연결이 약함 | App. G: 관련성 판단이 주관적임(다중 annotator 합의로 완화). 멀티모달은 다루지 않음. 저자는 검색 결과가 QA 성능을 올린다고도 보고함 | 수정 |
| 2502.18418 | Rank1: Test-Time Compute for Reranking in Information Retrieval | 서지 | 2025 / COLM | 첫 쪽 "Published as a conference paper at COLM 2025" | 일치 |
| 2502.18418 | 〃 | 수치 | 추론 체인 63.5만 개 | §1 "635,000 examples", §2 "635,264" (초록은 "600,000 이상") | 일치 |
| 2502.18418 | 〃 | Benchmarks | BRIGHT, FollowIR 등, BEIR | BRIGHT, NevIR, mFollowIR, DL19, BEIR(Table 6) | 수정 (FollowIR → mFollowIR·NevIR) |
| 2502.18418 | 〃 | ReasonRank가 2–2.7배 빠르다는 주장 | alphaXiv 요약 기준 **미확인** | 2508.07050 §5.5: BRIGHT 8개 데이터셋에서 ReasonRank(7B)가 Rank1(7B)보다 2–2.7배 빠름 | 일치 (본문 확인) |
| 2502.18418 | 〃 | Limitations | pointwise 추론이라 latency 큼 | §6: "overthinking"(관련성 판정이 지나치게 엄격함). 추론 속도가 분류 헤드 모델보다 느림(vLLM과 양자화로 일부 완화) | 일치 (보강) |
| 2503.06034 | Rank-R1: Enhancing Reasoning in LLM-based Document Rerankers via Reinforcement Learning | 서지 | 2025 arXiv → SIGIR 2026 | v1, comment 없음, 각주 "Work in progress". SIGIR 2026 근거 없음 | 수정 (venue 근거 없음) |
| 2503.06034 | 〃 | 수치 | SFT 대비 18% 데이터 (2차 출처) | 초록: in-domain에서 SFT와 대등한 효과를 학습 데이터 18%로 얻음 | 일치 (본문 확인) |
| 2503.06034 | 〃 | Benchmarks | TREC DL19/20, BRIGHT | Table 1(DL19/20), Table 2(BRIGHT) | 일치 |
| 2503.06034 | 〃 | Limitations | 처리량 낮음, in-domain 이득 제한적 | 한계 절 없음. in-domain은 "SFT와 대등"이라는 결과이지 한계로 서술되지 않음. 처리량 30 q/h는 TALRanker가 보고한 값 | 수정 (명시된 한계 없음) |
| 2508.07050 | ReasonRank: Empowering Passage Ranking with Strong Reasoning Ability | 서지 | 2025 / arXiv | comment "accepted by ACL2026 main conference" | 수정 |
| 2508.07050 | 〃 | 수치 | BRIGHT 32B 40.6 (v2 기준) | 로컬 v3 Table 1: ReasonRank(32B) BRIGHT 평균 38.03(ReasonIR top-100), 7B 35.74. R2MED 32B 42.85. 40.6은 v3 본문에 없음 | 수정 (v3 값으로 교체, 40.6은 본문 확인 불가) |
| 2508.07050 | 〃 | Benchmarks | BRIGHT, R2MED | BRIGHT, R2MED, BEIR 7종(Table 4) | 수정 (BEIR 추가) |
| 2508.07050 | 〃 | Limitations | listwise 윈도우 제약, 합성 데이터의 teacher 의존 | Limitations 절: (1) 비추론 데이터가 없어 난이도에 따라 추론과 비추론 모드를 전환하지 못함. (2) sliding window에 의존함(full-list ranking은 future work) | 수정 (teacher 의존은 우리 추론) |
| 2505.16886 | Don't "Overthink" Passage Reranking: Is Reasoning Truly Necessary? | 서지 | 2025 / arXiv (venue 미확인) | v1, comment 없음, 헤더 없음 | 일치 (venue 근거 없음) |
| 2505.16886 | 〃 | Core | 같은 조건에서 비추론이 대체로 우수, 점수 양극화 | 초록과 §1: 이 결론은 pointwise reranker(ReasonRR = Rank1 재현)에 한정됨. ReasonRR-NoReason이 ReasonRR보다 나음. 추론이 점수를 양극화해 부분 관련성을 반영하지 못함 | 일치 (pointwise 한정 명시 필요) |
| 2505.16886 | 〃 | Benchmarks | TREC DL, BEIR, BRIGHT(세부 미확인) | §3: DL19–DL23(MS MARCO v1/v2)과 BRIGHT. BEIR는 쓰지 않음 | 수정 |
| 2505.16886 | 〃 | Limitations | 특정 모델 크기와 학습 설정에 한정 | Limitations 절: listwise·setwise에는 적용되지 않을 수 있음. Qwen2.5 계열 ≤7B로 한정됨 | 일치 (구체화) |
| 2607.10555 | Tool-Adaptive LLM Reranker (TALRanker) | 서지 | 2026 / arXiv | v1 2026-07, "Preprint" | 일치 |
| 2607.10555 | 〃 | 수치 | Rank-R1-14B 약 30 q/h (**미확인**) | Results "Throughput Superiority": Rank-R1-14B 30 queries/hour. TALRanker-4B 977, 8B 778 q/h. direct-scoring 모델 500–1000 q/h(BRIGHT, 단일 GPU) | 일치 (본문 확인) |
| 2607.10555 | 〃 | Benchmarks | DL19/20, BEIR 9종, reasoning 벤치 | DL19/20, BEIR 9종, BRIGHT | 일치 (BRIGHT 명시) |
| 2607.10555 | 〃 | Limitations | v1 preprint, 수치 미확인 | Conclusion: base LLM 고유의 추론 능력(특히 도구 사용 궤적)에 강하게 의존함 | 수정 |
| 2507.06838 | Shifting from Ranking to Set Selection for Retrieval Augmented Generation (SetR) | 서지 | 2025 / ACL (oral) | comment "Accepted to ACL 2025 main (Oral Presentation)" | 일치 |
| 2507.06838 | 〃 | Benchmarks | HotpotQA, 2Wiki, MuSiQue, MultiHopRAG | §4.1 동일. Llama-3.1-8B-Instruct로 distill, generator는 Llama-3.1-8B | 일치 |
| 2507.06838 | 〃 | Limitations | 멀티홉 중심, 선택 LLM 비용 | Limitations 절: 1단계 검색에 의존함. 멀티홉 외 도메인(코드, 대화)은 검증하지 않음. 추론 LLM의 품질에 의존함. 비용은 한계로 쓰지 않았고, 오히려 효율(평균 passage 수 약 50% 감소)을 주장함 | 수정 |
| 2401.14887 | The Power of Noise: Redefining Retrieval for RAG Systems | 서지 | 2024 / SIGIR | 첫 쪽 "SIGIR '24" | 일치 |
| 2401.14887 | 〃 | 수치 | 무작위 문서가 정확도를 최대 35% 올림 | 초록 "up to 35%". §5: 0.07(+35%), 검색 문서 4개 설정. 적절한 위치에 배치했을 때의 결과 | 일치 |
| 2401.14887 | 〃 | Benchmarks | NQ-open 기반 | NQ-open. 무작위 문서는 Reddit Webis-TLDR-17로도 실험 | 일치 (보강) |
| 2401.14887 | 〃 | Limitations | 메커니즘 설명 부족, 설정 의존성 | Conclusion: 내부 메커니즘 규명은 추가 연구가 필요함. §4: 정답 문자열 포함 여부로 하는 이진 평가의 한계 | 일치 (보강) |
| 2507.19102 | Distilling a Small Utility-Based Passage Selector to Enhance Retrieval-Augmented Generation | 서지 | 2025 / SIGIR-AP | 첫 쪽 "SIGIR-AP '25, Xi'an" | 일치 |
| 2507.19102 | 〃 | Core | Qwen3-32B → 1.7B, 적응적 passage 수 | 초록 일치. 복잡한 질문(HotpotQA)에서 utility selection이 relevance ranking보다 나음 | 일치 |
| 2507.19102 | 〃 | Benchmarks | MS MARCO, HotpotQA 등(세부 미확인) | 학습: MS MARCO. ranking 평가: DL19/20 + BEIR 8종. RAG 평가: NQ, HotpotQA(BEIR 부분집합). generator 2종(Llama-3.1-8B, Qwen2.5-7B) | 수정 (세부 확정) |
| 2507.19102 | 〃 | Limitations | utility가 generator에 종속(2510.11358) | 한계 절 없음. Conclusion에도 한계 서술 없음. 저자는 오히려 두 generator에서 robust하다고 주장함. 종속성은 2510.11358에 근거한 우리 해석 | 수정 (명시된 한계 없음) |
| 2510.11358 | LLM-Specific Utility for Retrieval-Augmented Generation | 서지 | "LLM-Specific Utility: A New Perspective for RAG", 2025 / arXiv | 실제 제목은 "LLM-Specific Utility for Retrieval-Augmented Generation". 첫 쪽 "CIKM '26, Rome". comment "Accepted to CIKM 2026" | 오류 (제목·venue 수정) |
| 2510.11358 | 〃 | Core | utility가 LLM마다 다름, attention 기반 추정이 가장 나쁨 | 초록: utilitarian passage는 모델에 종속되고 transfer되지 않음. §5 Table 4: attention 가중치가 utility와 가장 안 맞음(초기 검색보다도 나쁨) | 일치 |
| 2510.11358 | 〃 | Benchmarks | 미확인(2차 요약) | SpecUBench: NQ, TriviaQA, MS MARCO-FQA, HotpotQA, FEVER, 2WikiQA × LLM 4종(Qwen3-8B/14B/32B, Llama3.1-8B) | 수정 |
| 2510.11358 | 〃 | Limitations | 미확인 | §3.3과 §6: 단일 passage(pointwise) utility만 다루고 passage 간 보완·중복은 다루지 않음. 이진·정확도 중심 정의 | 수정 |
| 2310.04408 | RECOMP: Improving Retrieval-Augmented LMs with Compression and Selective Augmentation | 서지 | "…Improving RAG with…", 2024 / ICLR (ID 재확인 권장) | 정식 제목은 "Improving Retrieval-Augmented LMs with…". 로컬 v1에는 헤더도 comment도 없음 | 수정 (ID 확인, ICLR은 venue 근거 없음) |
| 2310.04408 | 〃 | Benchmarks | NQ, TriviaQA, HotpotQA, WikiText | §4: WikiText-103(LM), NQ, TriviaQA, HotpotQA | 일치 |
| 2310.04408 | 〃 | 수치(추가) | — | §5: NQ에서 압축률 5%에 EM −2. HotpotQA에서 추출형 11%에 EM −2.4 | 추가 |
| 2310.04408 | 〃 | Limitations | target LM에 맞춰 학습, 멀티홉 정보 손실 | 한계 절 없음. §5: HotpotQA용 생성형 compressor는 추가 연구가 필요함. §6: LM 과제에서는 다른 LM으로 잘 transfer되지만 QA(LLaMA-13B)에서는 성능이 떨어짐 | 수정 |
| 2310.06839 | LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression | 서지 | 2024 / ACL (ID 재확인 권장) | comment "Accepted at ACL 2024". 본문 헤더 없음 | 일치 (ID 확인) |
| 2310.06839 | 〃 | 수치(추가) | — | 초록: NQ에서 토큰 약 4배 감소에 최대 +21.4%. LooGLE 비용 94.0% 감소. 약 10k 토큰을 2–6배 압축하면 1.4–2.6배 가속 | 추가 |
| 2310.06839 | 〃 | Benchmarks | NQ 멀티문서, LongBench, ZeroSCROLLS | + MuSiQue, LooGLE | 수정 |
| 2310.06839 | 〃 | Limitations | 소형 LM perplexity 의존, 고압축 시 가독성 저하 | Limitation 절: 질문 인지형이라 같은 문맥도 질문마다 다시 압축해야 해 캐싱이 불가함. 연산량이 LLMLingua의 2배. 문맥과 질문의 관계가 복잡하면 효과가 줄 수 있음 | 수정 |
| 2405.13792 | xRAG: Extreme Context Compression for Retrieval-augmented Generation with One Token | 서지 | 2024 / NeurIPS (ID 재확인 권장) | 첫 쪽 각주 "38th Conference on Neural Information Processing Systems (NeurIPS 2024)" | 일치 (ID 확인) |
| 2405.13792 | 〃 | Benchmarks | NQ, TriviaQA, WebQ, HotpotQA 등 | 6종: NQ, TriviaQA, WebQ, HotpotQA, TruthfulQA, FactKG. FLOPs 3.53배 감소 | 일치 (보강) |
| 2405.13792 | 〃 | Limitations | 멀티홉·세부 사실에서 하락 | App. G: 단일 dense 벡터 검색만 다룸. 추론이 필요한 과제(HotpotQA, FactKG)에서 RAG에 크게 뒤짐. Top-1 문서만 다룸 | 일치 (보강) |
| 2501.16214 | Provence: efficient and robust context pruning for retrieval-augmented generation | 서지 | 2025 / ICLR | comment "Accepted to ICLR 2025" | 일치 |
| 2501.16214 | 〃 | Benchmarks | 다도메인 QA(세부 미확인) | NQ, TyDi QA, PopQA, HotpotQA(Wikipedia), BioASQ(PubMed), SyllabusQA, RGB(news). reranking은 MS MARCO, DL19, BEIR | 수정 (세부 확정) |
| 2501.16214 | 〃 | Limitations | 문장 단위 pruning으로 의존성 손실 가능 | Conclusion의 Limitations: QA에만 초점, passage를 한 번에 하나씩 처리, 영어 데이터만 사용 | 수정 |
| 2501.16075 | PISCO: Pretty Simple Compression for Retrieval-Augmented Generation | 서지 | 2025 / Findings of ACL | 로컬 v1에 comment도 헤더도 없음 | 수정 (venue 근거 없음) |
| 2501.16075 | 〃 | 수치 | 16배 압축, 손실 0–3%, 5.7배 속도 | 초록과 §1: 16배, 0–3%, 5.7배(Mistral-7B 대비) | 일치 |
| 2501.16075 | 〃 | Benchmarks | RAG QA 다수 | Table 2: ASQA, HotpotQA, NQ, TriviaQA, PopQA + 도메인 밖·다국어. backbone 3종(Mistral-7B, Llama-3.1-8B, Solar-10.7B) | 수정 (세부 확정) |
| 2501.16075 | 〃 | Limitations | 오프라인 사전 압축 필요, 백본별 학습 | 한계 절 없음 | 수정 (명시된 한계 없음, 기존 서술은 우리 추론) |
| 2602.15856 | Rethinking Soft Compression in Retrieval-Augmented Generation: A Query-Conditioned Selector Perspective (SeleCom) | 서지 | 2026 / WWW '26 | 첫 쪽 "WWW '26, Dubai". comment "Accepted by WWW 2026". 메타데이터 게시일(2026-01-25)이 ID(2602)와 맞지 않음 | 일치 |
| 2602.15856 | 〃 | 수치 | 연산·latency 33.8–84.6% 감소 | 초록 일치 | 일치 |
| 2602.15856 | 〃 | Benchmarks | 미확인 | NQ, TriviaQA, WebQ, PopQA, HotpotQA, FactKG | 수정 |
| 2602.15856 | 〃 | Limitations | 쿼리 조건부라 오프라인 이점 사라짐(추론) | 한계 절 없음 | 수정 (명시된 한계 없음, 추론 표시 유지) |
| 2609.05152 | Compression Beyond the Uncompressed: A Two-Stage Training Recipe for Soft Context Compression in RAG (DEX-Comp) | 서지 | 제목 "Two-Stage Training for…" | 실제 제목은 "A Two-Stage Training Recipe for…". v2 "Preprint", comment "Under Review" | 수정 (제목) |
| 2609.05152 | 〃 | Core/수치 | distill 후 비압축 RAG가 틀리는 쿼리로 RL, 16배, 4–24배 가속 | 초록: teacher가 맞힌 질문만 distill(Pure Distillation), teacher가 틀린 질문에 결과 기반 RL(Hard Exploration). 16배 압축. TTFT 4.4–23.7배 감소. 튜닝하지 않은 full-context RAG보다 평균 정확도가 높음 | 일치 (수치 정밀화) |
| 2609.05152 | 〃 | Benchmarks | QA 5종 | NQ, TriviaQA, HotpotQA, ASQA, PopQA | 일치 (세부) |
| 2609.05152 | 〃 | Limitations | 미확인 | 한계 절 없음 | 수정 (명시된 한계 없음) |
| 2609.22100 | AdaMem: Adaptive Memory Token Allocation for Soft Compression in Retrieval-Augmented Generation | 서지 | 2026 / arXiv, 제출일 메타데이터 불일치(**미확인**) | v1 표기 "12 Aug 2026"인데 ID는 2609. 메타데이터 게시일도 2026-08-12. 불일치는 사실이고 원인은 확인 불가 | 확인 |
| 2609.22100 | 〃 | Core | passage마다 memory token 수를 적응적으로 배분, KILT-SCR | 초록: 고정된 memory-token 예산을 relevance에 따라 쿼리별로 배분하고 점수가 낮은 passage는 생략함. 하나의 compressor가 한 번의 패스로 memory와 relevance 점수를 함께 냄. 16배에서 OSCAR 대비 최대 +3.2점, 64배에서 평균 상대 이득 14.6%. full context 대비 latency 최대 4배 감소 | 일치 (보강: 예산 배분과 단일 패스 통합) |
| 2609.22100 | 〃 | Benchmarks | open-domain QA 6종 | TriviaQA, HotpotQA, NQ, PopQA, ASQA, BioASQ | 일치 |
| 2609.22100 | 〃 | KILT-SCR 범위 | **미확인** | §KILT-SCR: 쿼리당 KILT-100w에서 25개 passage 고정 풀(bge-large). 학습 196,916개(NQ, HotpotQA, TriviaQA). 평가는 NQ·HotpotQA·TriviaQA·PopQA 각 2,000개, ASQA 948개, BioASQ 1,609개 | 수정 (범위 확정) |
| 2609.22100 | 〃 | Limitations | 제출일 불일치 | 한계 절 없음. Conclusion의 future work: 온도를 쿼리별로 학습·예측하는 것, KV-cache 압축 등으로 확장하는 것 | 수정 |
| 2510.20797 | No Mean Feat: Simple, Strong Baselines for Context Compression (BenchPress) | 서지 | 2025 / arXiv | v2 "Preprint. Under review." | 일치 |
| 2510.20797 | 〃 | Core | mean pooling과 양방향 압축 토큰이 causal 압축 토큰보다 우수 | 초록 일치 | 일치 |
| 2510.20797 | 〃 | Benchmarks | 자체 suite | BenchPress: SQuAD, NarrativeQA, HotpotQA(in-domain), AdversarialQA, TriviaQA, ParaphraseRC(OOD), 중간 길이는 LongBench-E | 일치 (세부) |
| 2510.20797 | 〃 | Limitations | RAG 전용 아님 | 한계 절 없음. §7: 128배에서는 모든 방법이 teacher와 지속적인 격차를 보임. 이번 실험은 soft compression과 영어 독해에 한정됨 | 수정 |
| 2508.08742 | SciRerankBench: Benchmarking Rerankers Towards Scientific Retrieval-Augmented Generation with LLMs | 서지/내용 | 과학 도메인 RAG 연결 reranker 벤치 | 초록: 과학 5개 분야, NC·SSLI·CC 3종 Q-C-A, reranker 13종. 평가는 Recall@10/5, 답변 품질, 실행 효율(§4.6). LLM 수는 초록 "5 families", 결론 "11 LLMs". venue 근거 없음(v3, comment 없음) | 일치 (보강) |
| 2508.08742 | 〃 | Limitations | — | Limitations: 오픈소스 LLM만 다루고 GPT-4·Claude 같은 독점 모델은 포함하지 않음 | 추가 |
| 2604.15621 | Rethinking the Necessity of Adaptive RAG through the Lens of Adaptive Listwise Ranking (AdaRankLLM) | B.4.3 서술 | "최적 k는 동적" | 초록: passage dropout 기반 적응형 listwise ranker가 고정 깊이 검색보다 대체로 우수함. 약한 모델에는 noise filter, 강한 모델에는 효율화 수단 | 일치 (표현 정밀화) |


---

## 배치 b04a

# b04a 검증표 — 04_multimodal_efficiency.md, 헤더 + 축 A (A-1~A-4)

근거: 로컬 본문 txt/ 및 meta/arxiv_meta.json만 사용(웹 미사용). VisDoM=txt/2025.naacl-long.310.txt, ViDoRAG=txt/2025.emnlp-main.464.txt, WavRAG=txt/2025.acl-long.613.txt, 나머지 txt/<arXivID>.txt. 19편 모두 본문 접근 가능.

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2407.01449 | ColPali | 서지 | 2024 arXiv / ICLR 2025 | 1쪽 헤더 "Published as a conference paper at ICLR 2025"; arXiv comment 동일 | 일치 |
| 2407.01449 | ColPali | Core | PaliGemma 패치 멀티벡터 + late interaction, ViDoRe 공개 | 초록, §1 Contribution 1 | 일치 |
| 2407.01449 | ColPali | 수치(관찰) | 페이지당 수백~천 개 벡터 | §6 Storage Footprint: 1024 패치 + 6 텍스트 토큰, D=128, 257.5 KB/페이지 | 수정(구체 수치로 교체) |
| 2407.01449 | ColPali | Benchmarks | ViDoRe v1 | 초록, §3 | 일치 |
| 2407.01449 | ColPali | Limitations | 미확인 | Limitations 절 없음. §7 Future Work: 언어·모달리티·태스크로 벤치마크 확장, 신뢰도 추정; §6: 저장비용은 압축/클러스터링·token pooling으로 개선 가능 | 수정 |
| 2406.11251 | DSE | 서지 | 2024 arXiv (v2 2024.12), venue 미기재 | arXiv comment "EMNLP2024 main"; 본문 Ethics에 ACL Ethics Policy | 수정(EMNLP 2024 추가) |
| 2406.11251 | DSE | Core/수치 | 단일 dense 벡터, Wiki-SS 공개 | 초록: Wiki-SS 1.3M 스크린샷, BM25 대비 top-1 +17pt, 슬라이드 검색 OCR 대비 nDCG@10 +15pt 이상 | 일치(수치 보강) |
| 2406.11251 | DSE | Benchmarks | Wiki-SS(NQ), 슬라이드 검색 | §4: Wiki-SS(NQ, TriviaQA zero-shot), SlideVQA-open(50,714 슬라이드) | 수정(명칭 구체화) |
| 2406.11251 | DSE | Limitations | 미확인 | §7 Limitations: PDF 등 일반 문서 미검증, 대조 사전학습 미적용, 저화질/고해상도 이미지 시 효율·성능 문제 | 수정 |
| 2410.10594 | VisRAG | 서지 | ICLR 2025 | 1쪽 헤더 ICLR 2025 | 일치 |
| 2410.10594 | VisRAG | 수치 | end-to-end 20–40% 향상 | 초록 "20–40% end-to-end performance gain"; §5: 평균 40% 상대 정확도 향상 | 일치 |
| 2410.10594 | VisRAG | Benchmarks | 자체 구성 다중 데이터셋 | §4 표1: ArxivQA, ChartQA, MP-DocVQA, InfoVQA, PlotQA, SlideVQA(+합성 학습데이터) | 수정 |
| 2410.10594 | VisRAG | Limitations | 단일 이미지 VLM 위해 페이지 연결·가중 선택 필요 | 이는 §3 방법(page concatenation / weighted selection) 설명이지 한계 진술이 아님. Limitations 절 없음, §6 Conclusion에 한계 없음 | 오류 → "명시된 한계 없음" |
| 2411.04952 | M3DocRAG | 서지 | 2024.11 arXiv | arXiv v1 2024-11-07, comment에 venue 없음 | 일치(venue 근거 없음) |
| 2411.04952 | M3DocRAG | 수치 | 3,368 PDF, 2,441 질문 | §3: 2,441 multi-hop 질문, 3,368 PDF, 41,005 페이지 | 일치(페이지 수 보강) |
| 2411.04952 | M3DocRAG | Benchmarks | M3DocVQA, MP-DocVQA, MMLongBench-Doc | 초록, §5 | 일치 |
| 2411.04952 | M3DocRAG | 추가 발견 | (없음) | §5.3 표4: FlatIP 21s/질의 → IVFFlat 1.8s, IVFPQ 0.2s, 정확도 대부분 유지 | 추가(A-3 #2 영향) |
| 2411.04952 | M3DocRAG | Limitations | 미확인 | Limitations 절 없음, §7 Conclusion에도 한계 없음 | 수정 → "명시된 한계 없음" |
| 2505.17166 | ViDoRe v2 | 서지 | 2025.05 arXiv | arXiv 2025-05-22; comment "Benchmark Release (Published as a blogpost originally)"; 본문 Date: March 18, 2025 | 일치(블로그 원출처 명시) |
| 2505.17166 | ViDoRe v2 | 수치 | v1 >90 nDCG@5 포화 | 서론 "surpass a nDCG@5 of 90" | 일치 |
| 2505.17166 | ViDoRe v2 | Benchmarks | 4개 다국어 데이터셋 | §3 표1: Insurance(Fr), Biomedical, Economics, ESG(+Human); 각 다국어 번역판(fr/en/es/de). 각주: Insurance는 저작권 문제로 이후 제거 | 수정(Insurance 제거 사실 추가) |
| 2505.17166 | ViDoRe v2 | Limitations | v1과 순위 상관 높음, 4개 데이터셋 규모 | Limitations 절 없음. "strong correlation with v1"은 §5 관찰 결과로 제시(한계로 서술 안 함) | 수정 → 명시된 한계 없음 + (관찰) |
| 2601.08620 | ViDoRe v3 | 서지 | 2026.01 arXiv / ACL 2026 | arXiv 2601.08620v2(2026-04-21), comment 비어 있음; 본문 헤더에 venue 표기 없음 | 수정 → "venue 근거 없음" |
| 2601.08620 | ViDoRe v3 | 수치 | 10 도메인, ~26k 페이지, 3,099 질의 × 6개 언어 | 초록: 10 datasets, 26,000 pages, 3,099 queries, 6 languages; §3: 2개 데이터셋은 private hold-out | 일치(+ 8 공개/2 비공개) |
| 2601.08620 | ViDoRe v3 | 어노테이션 시간(충돌) | 블로그 9k h vs 논문 12k h | 초록·결론·Environmental Impact 모두 12,000 hours. 블로그 수치는 로컬 자료 없어 확인 불가 | 수정 → 논문 기준 12,000시간 |
| 2601.08620 | ViDoRe v3 | Limitations | 비텍스트 요소·열린 질의·세밀한 grounding에서 고전 | 그 내용은 초록·결론의 '결과'. 실제 Limitations 절: 영어·프랑스어 원문 + 서유럽 고자원 6개 언어 한정, 공개 장문 문서 위주(이메일·티켓·손글씨 등 미포함) | 수정 |
| 2501.08828 | MMDocIR | 서지 | EMNLP 2025 | arXiv comment "Paper accepted to EMNLP-2025(Main)" | 일치 |
| 2501.08828 | MMDocIR | 수치 | 313 문서, 1,658 질의 | 그림1·§1·§3·표1: 313 docs, 1,658 질의(초록·결론은 1,685로 오기) | 일치(본문 기준) |
| 2501.08828 | MMDocIR | 훈련셋(충돌) | 73.8k vs 173.8k | §1 기여, §4, 표1(73.8k), 표4(6,878 docs, 73,843 QA; 7개 서브셋 합), 결론 모두 73,843. 초록만 "173,843" — 초록 오기 | 수정 → 73,843 질의 / 6,878 문서 |
| 2501.08828 | MMDocIR | Benchmarks | MMLongBench-Doc, DocBench 기반 | §3.1 | 일치 |
| 2501.08828 | MMDocIR | Limitations | 미확인 | Limitations 절: 7개 훈련 서브셋 중 3개는 레이아웃 라벨 미추출, 텍스트·시각 공동 학습 부재 | 수정 |
| naacl-long.310 | VisDoM | 서지 | 2024.12 arXiv / NAACL 2025 | 1쪽 각주 "Proceedings of the 2025 Conference of NAACL" — NAACL 2025 일치. arXiv 날짜는 로컬 메타 없음 | 일치(arXiv 날짜 본문 확인 불가) |
| naacl-long.310 | VisDoM | 수치 | 2,271 질의, 1,277 문서 | 표1 합계 377+350+407+586+551=2,271, 297+300+319+117+244=1,277 | 일치 |
| naacl-long.310 | VisDoM | Core | 시각 검색기가 텍스트 검색기를 능가 | §7.1 표: ColQwen2 평균 96.94로 최고(BGE1.5 대비 +4.5%). 핵심 기여에 VisDoMRAG(시각·텍스트 병렬 RAG + consistency-constrained modality fusion, 12–20% 향상) 누락 | 수정(보강) |
| naacl-long.310 | VisDoM | Benchmarks | PaperTab, FetaTab, SciGraphQA, SPIQA, SlideVQA | 표1 | 일치 |
| naacl-long.310 | VisDoM | Limitations | 미확인 | §10 Limitations: 텍스트 추출·파싱 오버헤드 유지, 질의당 LLM 3회 호출, 환각 | 수정 |
| emnlp-main.464 | ViDoRAG | 서지 | 2025.02 arXiv / EMNLP 2025 | 1쪽 각주 "Proceedings of the 2025 Conference on EMNLP, pages 9113–9134" | 일치(arXiv 날짜 본문 확인 불가) |
| emnlp-main.464 | ViDoRAG | Core | GMM 하이브리드 검색 + 다중 에이전트 반복 추론 | 초록·§5(Seeker/Inspector/Answer), 기존 대비 10% 이상 향상 | 일치 |
| emnlp-main.464 | ViDoRAG | ViDoSeek 규모(충돌) | 출처마다 다름 | §4.1 "approximately ~1.2k questions"; 부록 표11: 총 1,142 질문(single-hop 645 / multi-hop 497); 코퍼스는 25–50쪽 영어 슬라이드 300개(12 도메인) | 수정 → 1,142 질문, 300 슬라이드 덱 |
| emnlp-main.464 | ViDoRAG | Benchmarks | ViDoSeek, SlideVQA | §6, 부록 표12: SlideVQA-Refined(2,020 질문) | 수정(Refined 명시) |
| emnlp-main.464 | ViDoRAG | Limitations | 미확인 | Limitations: 전문가 작성 질의의 편향, 다중 에이전트 반복으로 계산 오버헤드·지연, 환각 | 수정 |
| 2506.18902 | jina-v4 | 서지 | 2025.06 arXiv | arXiv 2025-06-23, venue 없음 | 일치(venue 근거 없음) |
| 2506.18902 | jina-v4 | 수치 | 3.8B, 단일+멀티벡터, 태스크별 LoRA | 초록·표: 3.8B + LoRA 어댑터 각 60M(3개) | 일치 |
| 2506.18902 | jina-v4 | Benchmarks | Jina-VDR, ViDoRe | 표3: J-VDR, ViDoRe, CLIP Benchmark, MMTEB, MTEB-en, CoIR, LongEmbed, STS | 수정(보강) |
| 2506.18902 | jina-v4 | Limitations | 미확인 | Limitations 절 없음. §9 Future work: 다국어 능력 강화, 더 작은 효율 변형 | 수정 |
| 2510.01149 | ModernVBERT | 서지 | 2025.10 arXiv | arXiv 2025-10-01, venue 없음 | 일치(venue 근거 없음) |
| 2510.01149 | ModernVBERT | 수치 | 250M, ColPali(~3B)와 비슷한 ViDoRe, 지연 ~20ms vs 175ms | 표3: ColModernVBERT 0.25B, ViDoRe v1 81.2 / v2(eng) 56.0, CPU 질의 인코딩 20ms; ColPali 2.92B, 81.6/56.8, 175ms | 일치(지연은 CPU 질의 인코딩 기준임을 명시) |
| 2510.01149 | ModernVBERT | Benchmarks | ViDoRe v1/v2 | 표3(v2는 영어 split만) | 일치 |
| 2510.01149 | ModernVBERT | Limitations | 영어 중심, 소형 모델 한정 | §6 "Future Work & Limitations" | 일치 |
| 2506.04997 | Light-ColPali | 서지 | 2025.06 arXiv | arXiv comment "Accepted by ACL 2025 findings" | 수정(Findings ACL 2025 추가) |
| 2506.04997 | Light-ColPali | 수치 | 메모리 97% 이상 절감, 성능 94% 이상 | 초록: 11.8% 메모리로 98.2% 유지, 2.8% 메모리로 94.6% 유지 | 일치(정확 수치로 교체) |
| 2506.04997 | Light-ColPali | Benchmarks | ViDoRe 등 | §6/부록 B.1: 3개 벤치마크 9개 데이터셋(ViDoRe 6개, VisRAG, MMLongBench-Doc) | 수정 |
| 2506.04997 | Light-ColPali | Limitations | 프루닝 불만족, 극단 압축엔 재학습 필수 | 이는 본문 결과(§4, §5.2: 파인튜닝이 merge factor 25/49에서 손실의 61%/67% 회복). 실제 Limitations 절: 토큰 감소만 다룸 — 차원 축소·벡터 양자화·데이터 정제·증류 미탐색 | 수정 |
| 2509.23883 | DocPruner | 서지 | 2025.09 arXiv | 1쪽 "Preprint", comment "Under review" | 일치(venue 근거 없음) |
| 2509.23883 | DocPruner | 수치 | 질의 독립 오프라인, 저장 50–60% 절감 | 초록·§3·§5; 오프라인 인코딩 지연 60–66% 증가 | 일치(+오버헤드) |
| 2509.23883 | DocPruner | Benchmarks | ViDoRe, 다국어(de/ru/zh/ja) | §4: ViDoRe-V2, JinaVDR-Bench(de/ru/zh/ja); 모델 ColQwen2.5, ColNomic, Jina-v4 | 수정(V2·JinaVDR 명시) |
| 2509.23883 | DocPruner | Limitations | 미확인 | Limitations 절 없음. §6 Future work: 학습 과정에 프루닝 통합, 다른 모달리티로 확장 | 수정 |
| 2506.21601 | HPC-ColPali | 서지 | 2025.06 arXiv | arXiv comment "9 pages", 단일 저자 | 일치(venue 근거 없음) |
| 2506.21601 | HPC-ColPali | 수치 | 최대 32×(nDCG@10 하락 <2%), HNSW 지연 30–50% 감소 | 초록·§VI: 32×, <2%, 30–50%, 질의시 프루닝으로 late-interaction 연산 최대 60% 절감, RAG 환각 30% 감소·지연 절반 | 일치 |
| 2506.21601 | HPC-ColPali | Benchmarks | ViDoRe, SEC-Filings | §IV.A | 일치 |
| 2506.21601 | HPC-ColPali | Limitations | 미확인(단일 저자, 소규모 평가) | Limitations 절 없음. §VII Future Work: PQ 확장, 적응형 프루닝, **동적 코퍼스용 스트리밍 코드북 갱신** | 수정 |
| 2602.19549 | Prune-then-Merge | 서지 | Findings ACL 2026 | arXiv comment "Accepted by ... ACL 2026, Findings" | 일치 |
| 2602.19549 | Prune-then-Merge | 수치 | near-lossless 범위 확장 | §1: DocPruner 대비 평균 10pt, [50–60%]→[60–70%]; 오프라인 지연 47–56% 증가 | 일치(수치 보강) |
| 2602.19549 | Prune-then-Merge | Benchmarks | 29개 데이터셋, ColQwen2.5/ColNomic/Jina-v4 | §4.1: 6개 벤치마크 29개 데이터셋(ViDoRe-V1, V2, JinaVDR, REAL-MM-RAG, ViDoSeek, MMLongBench-Doc) | 일치(구성 보강) |
| 2602.19549 | Prune-then-Merge | Limitations | 미확인 | Limitations: 프루닝이 base LVLM 어텐션 신뢰도에 의존, 사전 정의 하이퍼파라미터(adaptation/merging factor) 의존 | 수정 |
| 2501.05874 | VideoRAG (Jeong) | 서지 | 2025.01 arXiv | arXiv comment "ACL Findings 2025" | 수정(Findings ACL 2025 추가) |
| 2501.05874 | VideoRAG (Jeong) | Core | 비디오 검색 + LVLM 생성, 프레임 선택 모듈 | §2.3: 프레임 선택은 **검색(질의 독립)과 생성(질의 유도) 양쪽**에 적용; §2.4 자막 없을 때 ASR 보조 텍스트 | 수정(검색에도 적용) |
| 2501.05874 | VideoRAG (Jeong) | Benchmarks | WikiHowQA × HowTo100M | §3: WikiHowQA×HowTo100M, Synthetic QA×HowTo100M | 수정(보강) |
| 2501.05874 | VideoRAG (Jeong) | Limitations | 미확인 | Limitations: 질의-비디오-정답 triple 데이터셋 부족(자체 구성으로 대체), 프레임 선택의 효과·효율 추가 개선 필요 | 수정 |
| 2502.01549 | VideoRAG (HKU) | 서지 | 2025.02 arXiv | arXiv 2025-02-03, venue 없음 | 일치(venue 근거 없음) |
| 2502.01549 | VideoRAG (HKU) | 수치(충돌) | 약 160편, 시간 수치 출처마다 다름 | 초록 "over 160 videos totaling 134+ hours"; 표1: 22 컬렉션, 164편, 602 질의, ~134.6시간 | 수정 → 164편 / ~134.6시간 |
| 2502.01549 | VideoRAG (HKU) | Limitations | 미확인 | Limitations 절 없음, §6 Conclusion에 한계 없음 | 수정 → 명시된 한계 없음 |
| 2510.09266 | CFVBench | 서지 | 2025.10 arXiv | arXiv 2025-10-10, venue 없음(ACM 템플릿 placeholder뿐) | 일치(venue 근거 없음) |
| 2510.09266 | CFVBench | 수치 | 599 비디오, 5,360 QA | 초록·§7 | 일치 |
| 2510.09266 | CFVBench | Core | 검색(Recall@K)+생성 2단계 평가 | §4: 7개 검색법·14개 MLLM 평가, R@1/3/5/10; AVR(적응형 프레임 밀도 증가 + 도구 호출) 제안 누락 | 수정(보강) |
| 2510.09266 | CFVBench | Limitations | 미확인 | Limitations 절 없음 | 수정 → 명시된 한계 없음 |
| acl-long.613 | WavRAG | 서지 | 2025.02 arXiv / ACL 2025 | 1쪽 각주 "Proceedings of the 63rd Annual Meeting of the ACL (Volume 1: Long Papers)" | 일치(arXiv 날짜 본문 확인 불가) |
| acl-long.613 | WavRAG | 수치 | ASR+텍스트 RAG와 동등 검색, ~10× 가속 | 초록 "comparable retrieval ... 10x acceleration"; 표1 Whisper 크기별 4.84–14.38× | 일치 |
| acl-long.613 | WavRAG | Benchmarks | 자체 구성 | §4: HotpotQA(음성), Spoken-SQuAD, SLUE-SQA-5, 자체 혼합 모달리티 데이터셋 | 수정 |
| acl-long.613 | WavRAG | Limitations | ASR 오류·정보 손실 제기(동기), 그 외 미확인 | Limitations: RAG가 응답의 음향 측면(억양·표현성·화자 스타일)에 기여하는 정도는 미해결 | 수정 |
| — | 헤더 | 검증 방법 | WebSearch 기반, 초록 단편만 | 이번 검증은 로컬 본문 전수 대조 | 수정 |
| — | A-3 #2 | 서술 | ANN 위 recall·갱신비용 거의 안 다룸, HPC-ColPali만 HNSW 지연 | M3DocRAG가 IVFFlat/IVFPQ 속도-정확도(표4) 보고; DocPruner·PtM은 오프라인 인코딩 지연 증가 보고; HPC-ColPali는 스트리밍 코드북 갱신을 future work로 명시 | 수정(주장 약화·정밀화) |
| — | A-4 #5 | 서술 | VideoRAG(Jeong) 프레임 선택은 생성용 | §2.3: 검색에도 질의 독립 프레임 선택 적용 | 오류 → 수정 |
| — | A-4 #2 | 서술 | ViDoSeek "대규모" 주장이나 수천 페이지 | 300 덱 × 25–50쪽 ≈ 7.5k–15k 페이지, 본문 "hundreds of documents, thousands of pages" | 일치(수치 보강) |


---

## 배치 b04b

# b04b 검증표: 04_multimodal_efficiency.md, 축 B (B-1–B-4, 참고 배경, 축 교차 관찰)

근거: verify/txt/*.txt(pdftotext 본문), verify/meta/arxiv_meta.json. 웹 검색은 쓰지 않았다.

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2405.19504 | MUVERA | 서지 | 2024 / NeurIPS 2024 | neurips2024_muvera.txt 첫 쪽 각주 "38th Conference on Neural Information Processing Systems (NeurIPS 2024)". arXiv v2(2026-06) comment: Theorem 2.1의 차원 한계를 정정 | 일치(v2 정정 사실을 덧붙임) |
| 2405.19504 | MUVERA | Core | FDE로 MIPS 환원, 후보 2–5× 감소, 지연 약 90% 감소 | 초록: "same recall … retrieving 2-5× fewer candidates", PLAID 대비 "average of 10% improved recall with 90% lower latency" | 일치(+10% recall 추가) |
| 2405.19504 | MUVERA | Benchmarks | BEIR | §3, 부록 Table 9: BEIR 6개(MS MARCO, NQ, HotpotQA, ArguAna, SciDocs, Quora) | 수정(세부 기입) |
| 2405.19504 | MUVERA | Limitations | 미확인 | §Broader Impacts and Limitations: MS MARCO에서는 PLAID를 넘지 못함. 문서당 임베딩 수 m_avg의 영향은 연구하지 않음 | 수정 |
| 2501.17788 | WARP | 서지 | 2025.01 / SIGIR 2025 | 첫 쪽 ACM Reference "SIGIR '25", arXiv comment "Accepted at SIGIR 2025" | 일치 |
| 2501.17788 | WARP | Core/수치 | XTR 대비 41×, PLAID 대비 약 3× | 초록: "reduces end-to-end latency … by 41x, … 3x speedup over ColBERTv2/PLAID". §6: LoTTE Pooled 기준 6초 초과 → 171ms | 일치 |
| 2501.17788 | WARP | Benchmarks | BEIR, LoTTE 등(세부 미확인) | Table 1: BEIR 6개, LoTTE 6개 | 수정 |
| 2501.17788 | WARP | Limitations | 미확인 | 별도 한계 절 없음. §6 Future work: SIMD/GPU 가속, 더 가벼운 query encoding, WARP와의 end-to-end 학습 | 수정 |
| 2405.12497 | RaBitQ | 서지 | 2024 / SIGMOD 2024 | 첫 쪽 "SIGMOD '24, Santiago", arXiv comment "accepted by SIGMOD 2024" | 일치 |
| 2405.12497 | RaBitQ | Core | D차원 → D비트, 불편 추정, 이론 오차 한계 | 초록 "quantizes D-dimensional vectors into D-bit strings … sharp theoretical error bound", §3 unbiased estimator | 일치 |
| 2405.12497 | RaBitQ | Benchmarks | 표준 ANN 데이터셋 | Table 3: MSong, SIFT, DEEP, Word2Vec, GIST, Image(6개) | 수정(세부 기입) |
| 2405.12497 | RaBitQ | Limitations | 미확인 | §7: 그래프 인덱스와 다른 저장장치로 확장하는 것은 향후 과제. §6: 전용 하드웨어 최적화는 열린 질문 | 수정 |
| 2409.09913 | Extended RaBitQ | 서지 | SIGMOD 2025 | 본문 헤더는 ACM 템플릿 placeholder("Conference'17"). arXiv comment "Preprint". 정식 제목 "Practical and Asymptotically Optimal Quantization of High-Dimensional Vectors in Euclidean Space for ANN Search" | 수정(venue 근거 없음) |
| 2409.09913 | Extended RaBitQ | Core | B비트/차원 | 초록: RaBitQ는 32× 압축만 지원 → 확장판은 중간 압축률(예: 8×, 4×), 공간-오차 trade-off의 점근 최적성 증명 | 일치 |
| 2506.08276 | LEANN | 서지 | 2025.06 arXiv (v2 2025.11) | arXiv v2 2025-11-25. 각주 "Under Review" | 일치(venue 근거 없음) |
| 2506.08276 | LEANN | 수치 | 원본의 5% 미만, 90% top-3 recall을 2초 미만 | v2 §1 기여: "over 90% top-3 recall within one second while using less than 5%". 초록: 기존 인덱스 대비 최대 50× 축소 | 수정(2초 → 1초, v2 기준) |
| 2506.08276 | LEANN | Benchmarks | RAG 벤치마크(세부 미확인) | §7.1: NQ, TriviaQA, GPQA, HotpotQA(RPJ-Wiki 데이터스토어) + FinanceBench, Enron Email, LAION | 수정 |
| 2506.08276 | LEANN | Limitations | 온라인 증분 갱신 미지원(2차 출처) | v2 초록 "supporting storage-efficient index construction and updates". §6 "Efficient Index Update"와 부록 B: 단일 삽입, 버퍼를 둔 배치 추가, soft deletion. §7.3: add 연산이 naive 대비 최대 63.3× 빠름. 별도 한계 절 없음 | 오류(v2는 갱신을 지원) |
| 2502.13826 | IP-DiskANN | 서지 | 2025.02 arXiv, venue 미확인 | arXiv comment 없음. 본문 "after the anonymous review" | 일치(venue 근거 없음) |
| 2502.13826 | IP-DiskANN | Core | 배치 consolidation 없이 in-place 삽입·삭제, recall 안정 | 초록: "first algorithm to avoid batch consolidation … stable recall". FreshDiskANN과 HNSW보다 처리량과 갱신 속도가 좋음 | 일치 |
| 2502.13826 | IP-DiskANN | Benchmarks | 스트리밍 갱신 시나리오 | §5: SlidingWindow·ExpirationTime·Clustered runbook × MSTuring-10M, Wikipedia(Cohere 768d) 10M/35M | 수정 |
| 2502.13826 | IP-DiskANN | Limitations | 미확인 | §7: replacement 같은 다른 adversarial workload 탐색, IVF·LSH 스트리밍 방법과의 비교는 향후 과제. §6: SPFresh·Ada-IVF와의 정규화 비교도 미실시 | 수정 |
| 2507.21989 | FANNS benchmark (ETH) | 서지/수치 | 2025.07, 270만 건 + 11개 속성 | 초록: "over 2.7 million arXiv papers … 11 real-world attributes", 11개 방법. §7: 4096차원. v3는 2026-04, venue 근거 없음 | 일치 |
| 2507.21989 | FANNS benchmark | Benchmarks | 자체 | arxiv-for-fanns(small/medium/large). 임베딩 특성 분석에는 SIFT, UQ-V 등도 사용 | 일치(보충) |
| 2507.21989 | FANNS benchmark | Limitations | 모든 시나리오에서 최선인 단일 방법 없음 | 그런 문장은 없음. 관련 서술은 Obs.3(특정 필터 전용 방법이 범용 방법보다 최대 10× 빠름), Obs.5(대규모에서 recall이 0.2 미만으로 떨어지는 방법 있음). §7의 "key limitation"은 기존 문헌에 transformer 임베딩 데이터셋이 없다는 지적. 자체 한계 절 없음 | 수정 |
| 2508.16263 | Attribute Filtering study | 서지 | 2025.08 arXiv | 본문 "SIGMOD 26, Bangalore", arXiv comment "Accepted at SIGMOD 2026" | 수정(venue 추가) |
| 2508.16263 | Attribute Filtering study | Core | 속성 유형·전략별 비교 | 초록: 10개 알고리즘, 12개 방법, 4개 데이터셋(최대 1,000만), 선택도 0.1–100% | 일치 |
| 2508.16263 | Attribute Filtering study | Benchmarks | FANNBench | FANNBench는 코드 저장소 이름. 데이터는 SIFT, SpaceV(각 1,000만), Redcaps, Youtube-RGB(각 100만)(Table 4) | 수정 |
| 2508.16263 | Attribute Filtering study | Limitations | 미확인 | §6.3 Open Problems O1–O5. O4: "Dynamic Filtering ANN index is still underexplored"(동적 갱신은 DSG만 지원). O5: 속성 분포를 uniform으로 가정 | 수정 |
| 2509.07789 | FANNS unified benchmark | 서지 | 2025.09 arXiv | PVLDB 템플릿(Vol.14 No.1 placeholder)만 있고 arXiv comment 없음 | 일치(venue 근거 없음) |
| 2509.07789 | FANNS unified benchmark | Benchmarks | 통합 벤치마크 | Table 3: arXiv, TripClick, LAION1M, YFCC, YouTube Audio, YouTube Video | 수정 |
| 2509.07789 | FANNS unified benchmark | Limitations | 기존 비교의 편향 지적 | 이 지적은 초록의 기존 연구 비판이며, 저자의 자기 한계가 아님. 자체 한계 절 없음 | 수정(항목 재분류) |
| 2025.emnlp-main.805 | Drift-Adapter | 서지 | 2025.09 / EMNLP 2025 | 첫 쪽 "Proceedings of the 2025 Conference on EMNLP, pages 15938–15949". 정식 제목 "Drift-Adapter: A Practical Approach to Near Zero-Downtime Embedding Model Upgrades in Vector Databases". "2025.09" 날짜 근거는 없음 | 수정(제목, 날짜) |
| 2025.emnlp-main.805 | Drift-Adapter | 수치 | 95–99%, <10μs, >100× | 초록 그대로 일치. §5.1: 텍스트 R@10 ARR 98–99%, CLIP 약 97.8% | 일치 |
| 2025.emnlp-main.805 | Drift-Adapter | Benchmarks | MTEB 텍스트, 1M CLIP 이미지 | §4: AG-News, DBpedia-14, Emotion(MTEB, 각 1M, MiniLM-L6 → mpnet-base), LAION-400M 1M(CLIP ViT-B/32 → L/14) | 일치(보충) |
| 2025.emnlp-main.805 | Drift-Adapter | Limitations | 미확인. (관찰) 단일벡터, 다세대 미평가 | Limitations 절: drift의 크기·유형에 의존, f_old에 접근할 수 있어야 함, 전역 adapter 하나뿐(local drift 미대응), 재계산을 없애는 것이 아니라 미룸, 10억 규모 검증 없음, 연쇄 적응의 "Cumulative Error … was not studied" | 수정(관찰 → 저자 명시) |
| 2506.00037 | QDC | 서지/제목 | CoLLAs 2025, 제목은 검색 요약 기반 | 첫 쪽 헤더 "Published at 4th Conference on Lifelong Learning Agents (CoLLAs), 2025". 제목은 meta와 일치 | 일치(표시 제거) |
| 2506.00037 | QDC | Core | 재인코딩 없이 질의 drift 보정, 문서 drift가 더 큼 | 초록, §5(그림 3): 문서 임베딩 drift가 질의보다 큼. 긴 문서일수록 drift가 큼. 재인덱싱은 비용이 크고 성능도 좋지 않음 | 일치 |
| 2506.00037 | QDC | Benchmarks | 미확인 | §4: BEIR 5개를 순차 task로(MS MARCO → NQ → HotpotQA → FEVER → FiQA-2018) | 수정 |
| 2506.00037 | QDC | Limitations | 미확인 | 한계 절 없음. 부록 A: task-id를 모르는 CIL 설정은 향후 과제. 부록 C: 다른 retriever 구조(ColBERT 등)의 연속학습은 향후 과제 | 수정 |
| 2506.01877 | GradNormIR | 서지 | ACL Findings 2025 | arXiv comment "ACL 2025 Findings" | 일치 |
| 2506.01877 | GradNormIR | Benchmarks | BEIR | BEIR 10개 데이터셋을 순차 session으로 투입 | 일치 |
| 2506.01877 | GradNormIR | Limitations | 새 문서 색인이 미래 질의를 저하시킬 수 있다는 문제의식 | 이것은 동기(초록)다. Limitations 절: 문서 수준에서만 판단하고 미래 질의를 고려하지 않음. Conclusion: 문서가 연속으로 도착하는 online OOD 탐지는 향후 과제 | 수정 |
| 2503.04800 | HoH | 서지/Core | 2025.03 arXiv, 모의 검색엔진, outdated가 정확도 저하 | arXiv v3, comment 없음(venue 근거 없음). 96,124개 질문, 219,463개 문서. outdated 정보가 정확도를 낮추고 유해한 출력을 유도 | 일치 |
| 2503.04800 | HoH | Limitations | 미확인 | §7 Limitation: Wikipedia 스냅샷의 고정 주기, 단일 문서 기반 QA, 모의 검색엔진의 outdated 문서가 동질적(같은 문서의 이전 버전) | 수정 |
| 2505.17162 | DailyQA | 제목 | 정식 제목 미확인 | "DailyQA: A Benchmark to Evaluate Web Retrieval Augmented LLMs Based on Capturing Real-World Changes" | 수정 |
| 2505.17162 | DailyQA | Core | 매일 갱신, 상위 문서가 시간 제약 불충족으로 오도 | 초록: 질문은 매주, 답은 매일 갱신(Wikipedia revision log 기반). 웹 검색 결과 rerank가 중요. §1: 상위 문서가 시간 제약을 충족하지 않아 오도할 수 있음 | 수정(질문은 주 단위) |
| 2505.17162 | DailyQA | Limitations | 미확인 | Limitations: one-hop 질의만 포함(multi-hop·false premise 없음), 추론형 LLM 미평가, 일부 웹 페이지의 갱신 시각을 얻지 못해 Rerank-T 성능이 떨어졌을 수 있음 | 수정 |
| 2508.12282 | ChronoQA | Core | 중국어, 명시·암시·상대 시간 표현 | 초록: 2019–2024 뉴스 30만 건 이상, 5,176개 질문. 시간 유형은 absolute/aggregate/relative, 시간 표현은 explicit/implicit. 단일·다중 문서 | 수정(분류 축 교정) |
| 2508.12282 | ChronoQA | Benchmarks | ChronoQA | Table 4: Native RAG, Temporal Filter, Query Rewrite, Query Decomposition(K=5/10). Temporal Filter가 Native RAG보다 낮음 | 일치(보충) |
| 2508.12282 | ChronoQA | Limitations | 중국어 한정(관찰) | 한계 절 없음. 중국어 한정은 우리 관찰 | 일치(명시된 한계 없음) |
| 2212.09744 | DSI++ | 서지/Core | 2022 arXiv / EMNLP 2023, 망각 정식화, pseudo-query 생성 메모리 | arXiv comment "EMNLP 2023 main". 초록: SAM으로 flatter minima(+12% 암기) + generative memory. NQ에서 Hits@10 +21.1%, 재학습 대비 모델 업데이트 6× 적음 | 일치(SAM 추가) |
| 2212.09744 | DSI++ | Limitations | 새 문서 추가 시 기존 문서의 상당한 망각 | 초록은 이와 일치. Limitations 절: 새 문서가 기존 문서를 반박·수정하면 동작을 예측할 수 없음. full MS MARCO에서는 망각이 여전히 큼 | 수정(저자 명시 한계로 보강) |
| 2307.10323 | IncDSI | 서지 | ICML 2023 | 첫 쪽 "Proceedings of the 40th ICML, PMLR 202, 2023" | 일치 |
| 2307.10323 | IncDSI | Core/수치 | 제약 최적화로 빠른 추가 | 초록: 문서당 약 20–50ms | 일치 |
| 2307.10323 | IncDSI | Limitations | 성능 저하 → 재학습, 추가만 고려 | §7: 문서가 늘면 원 문서 성능이 조금씩 떨어져 결국 재학습이 필요. frozen query encoder. "only consider the setting of adding new documents". 편집은 향후 과제(삭제 언급 없음) | 일치 |
| 2504.17519 | GR over Dynamic Corpora | 서지 | SIGIR 2025 | ACM Reference "SIGIR '25", arXiv comment | 일치 |
| 2504.17519 | GR over Dynamic Corpora | Core | 텍스트 docid가 유리, multi-docid 제안 | 초록 일치. MDGR은 "improved performance in dynamic corpus without requiring additional retraining" | 일치 |
| 2504.17519 | GR over Dynamic Corpora | Limitations | 숫자 docid는 재학습 없이 적응 불가 | §7: 숫자 docid 방법은 추가 학습 없이 일반화하기 어려움(challenges). 텍스트 docid는 강함. 별도 한계 절 없음 | 수정(완화) |
| 2509.22116 | Does GR overcome DR? | 서지 | 2025.09 arXiv | 헤더 "Manuscript", comment 없음 | 일치(venue 근거 없음) |
| 2509.22116 | Does GR overcome DR? | Limitations | GR이 DR을 보편적으로 능가하지 못함 | 초록 일치. §6: 이상화한 이론, 대규모 비교 미실시, SOTA 변형 미포함, 제안 확장은 실험하지 않음. §5: 연속 corpus drift에서 GR은 재학습이나 국소 fine-tuning이 필요하고 DR은 index-only 갱신 가능 | 일치(보충) |
| 2508.21038 | LIMIT | 서지 | ICLR 2026 | 헤더 "Published as a conference paper at ICLR 2026" | 일치 |
| 2508.21038 | LIMIT | 수치 | SOTA 임베딩 Recall@20 60% 미만, BM25 93.6% | full LIMIT(Table 5, Recall@100): 단일벡터 최고 18.9(Promptriever 4096d), GTE-ModernColBERT 54.8, BM25 93.6. LIMIT-small(46문서, Recall@20): Promptriever 97.7, GritLM 90.5 등이며 60% 미만이 아님. 본문 §5: "struggle to reach even 20% recall@100" | 오류 → 수정 |
| 2508.21038 | LIMIT | Limitations | 정확 표현 정리, 단일 합성 벤치 | Limitations 절: 이론은 단일벡터에 한정되고 multi-vector에는 반드시 성립하지 않음. 일부 오류를 허용하는 설정의 bound는 없음. 어떤 조합이 실패할지는 사전에 증명할 수 없음 | 수정(저자 명시로 교체) |
| 2503.14649 | RAGO | 서지 | ISCA 2025 | 본문에 ISCA 헤더 없음. arXiv comment는 쪽수와 그림 수뿐 | 수정(venue 근거 없음) |
| 2503.14649 | RAGO | 수치 | QPS/chip 최대 2×, TTFT 55% 감소, 병목 이동 | 초록 일치. §1: hyperscale 검색은 지연의 80% 이상, long-context에서는 1% 미만 | 일치 |
| 2503.14649 | RAGO | Limitations | 에너지·비용은 future work | §9: "additional efficiency metrics, such as energy and cost efficiency. We leave these … to future work" | 일치 |
| 2205.09707 | PLAID | 서지 | CIKM 2022 | arXiv v1 comment "Preprint". 본문에 CIKM 표기 없음. GPU 최대 7×, CPU 최대 45×(vs vanilla ColBERTv2), 1.4억 passage | 수정(venue 근거 없음) |
| 2509.01306 | Re3 | 전체 | 검색 방법이나 세부 미확인 | 본문 파일 없음. arXiv meta: 제목 "Re3: Learning to Balance Relevance & Recency for Temporal Information Retrieval", v2(2026-01-06) comment는 저자가 문제 설정과 방법이 크게 다른 원고를 준비 중이라 철회(withdrawn)했다고 밝힘 | 본문 접근 불가(철회됨) |
| — | TempRAGEval, MixLoRA-DSI | 언급 | B-1, B-4에서 이름만 언급 | 배정된 로컬 본문 없음 | 본문 확인 불가 |
| — | HNSW, DiskANN/FreshDiskANN, IVF-PQ | 참고 배경 | 개별 재검증 안 함 | FreshDiskANN의 배치 consolidation은 IP-DiskANN 초록·§3.2에서 확인 | 일치 |

## 오픈 문제·연구 공백 서술 점검
| 위치 | 서술 | 근거 | 판정 |
|---|---|---|---|
| B-1 필터 검색 | 공통 결론은 "단일 최강 방법 없음"과 "기존 비교의 편향" | 편향은 2509.07789 초록이 명시. 2508.16263 O2는 "universal tuning impractical". 2507.21989는 용도별 선택 지침을 제시. "단일 최강 없음"이라는 문장 자체는 세 편 어디에도 없음 | 수정(표현 완화) |
| B-3 #1 | LEANN은 증분 갱신 미지원 | LEANN v2 §6·부록 B는 삽입, 배치 추가, soft delete를 지원 | 오류 → 수정. 결론("멀티벡터 스트리밍 공백")은 유지되고, 2508.16263 O4가 이를 보강 |
| B-3 #2 | Drift-Adapter/QDC는 한 세대만 다룸 | QDC는 5개 task 순차 갱신(다세대 연속 fine-tuning)을 평가. 다만 같은 모델을 연속 fine-tuning한 것이며 task-id를 안다고 가정. Drift-Adapter는 연쇄 적응의 누적 오차를 미연구로 명시 | 수정 |
| B-3 #4 / B-4 #3 | 인덱스 계층의 시간 활용 드묾 | ChronoQA는 Temporal Filter 검색 baseline을 평가했고 Native RAG보다 낮았음. 인덱스 수준 연구는 여전히 없음 | 일치(근거 보강) |
| B-3 #5 | DSI++, IncDSI, 2504.17519 모두 망각과 재학습 필요를 보고 | DSI++와 IncDSI는 맞음. 2504.17519는 텍스트 docid의 일반화와 MDGR의 재학습 없는 개선을 보고함. 2509.22116 §5가 GR의 재학습 필요를 명시 | 수정 |
| B-3 #7 | LIMIT은 멀티벡터·희소의 우위를 보여줌 | full LIMIT에서 ColBERT 54.8, BM25 93.6(R@100). 저자는 이론이 멀티벡터에 성립하는지는 미확장이라고 밝힘 | 일치(단서 추가) |
| B-4 #3 | 가장 가까운 연구로 Re3 언급 | Re3는 철회됨 | 수정(철회 표기) |


---

## 배치 b05a

# b05a 검증표 — 05_prior_art_check.md (머리말 + C1~C4)

근거: txt/<ID>.txt (pdftotext 본문), meta/arxiv_meta.json. 웹 미사용.

## C1. 계층 요약 / GraphRAG 인덱스의 삭제·수정

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2506.20963 | EraRAG: Efficient and Incremental Retrieval Augmented Generation for Growing Corpora | 서지 | 2025 | arXiv v2 2025-07-04, comment "Under review" → venue 근거 없음 | 일치 |
| 2506.20963 | EraRAG | 핵심 | 삽입 시 영향 세그먼트·조상만 재요약, 삭제는 다루지 않는 것으로 보임 | 초록 "localized insertions of new data"; §IV 업데이트식 ∀Sj∈A(Si) ReSummarize; 실험은 50% 초기 + 5%씩 10회 삽입. "Delete the original chunk node"는 버킷 분할 연산이지 문서 삭제가 아님. 삭제·수정 실험 없음 | 일치(확정) |
| 2506.20963 | EraRAG | 한계 | — | Limitations 절 없음. 결론(VII)은 성장 코퍼스만 언급 | 명시된 한계 없음 |
| 2410.01736 | Recursive Abstractive Processing for Retrieval in Dynamic Datasets (adRAP) | 서지 | 2024 | arXiv 2024-10-02, comment 없음 → venue 근거 없음 | 일치 |
| 2410.01736 | adRAP | 핵심 | 추가/삭제 환경에서 RAPTOR 트리 유지 (adaptive UMAP + online GMM). 삭제 요약 무효화는 미확인 | §4.2 Adaptive UMAP, §4.3 Adaptive GMM 확인. 삭제는 §4.4 끝 한 단락뿐: "Though we focused on adding documents, the algorithm easily handles deletions by removing the chunk from the tree and recomputing summaries for its ancestors." 삭제 실험 없음(§6.4: 70% 구축 후 30% 추가). 잔류 사실 측정 없음 | 수정(삭제는 주장만, 미평가) |
| 2410.01736 | adRAP | 벤치 | — | MultiHop(RAG), QASPER(300문항), QuALITY(300문항) (§6.1) | 보완 |
| 2410.01736 | adRAP | 한계 | — | §7: UMAP/GMM 모델 저장 메모리 부담, 대량 추가 시 전체 재계산 필요; postQFRAP 요약은 출처 추적을 어렵게 함 | 보완 |
| 2609.34242 | Stashbird: Episode-Grounded Lifecycle Reconciliation for Conversational Agent Memory | 서지 | 2026 | arXiv v2 2026-10-17, Microsoft, "Preprint" → venue 근거 없음 | 일치 |
| 2609.34242 | Stashbird | 핵심·수치 | provenance 추적, 삭제 시 stale 커뮤니티 문서를 검색에서 제외; 업데이트 20 시퀀스, 삭제 개입 263건 | 초록·§4: 20 sequences 전부 통과(재조정 없으면 8), 263 = LoCoMo 203 + LongMemEval 60 개입. 평가는 **레코드 수준 final-state check**(관계·인덱스 문서 제거/보존)이며 **답변에 삭제 사실이 새는지는 측정하지 않음** | 일치(+보완) |
| 2609.34242 | Stashbird | 한계 | — | §5 Limitations: "Episode deletion leaves conversation summaries and rolling checkpoints unchanged, while affected community summaries remain stored pending regeneration." 엔티티 매칭 임계값 미탐색, LLM judge 무검증 | 보완(C1 공백 직접 지지) |
| 2607.11683 | RAGU: A Multi-Step GraphRAG Engine with a Compact Domain-Adapted LLM | 서지 | 2026 | arXiv 2026-07-13, comment 없음 → venue 근거 없음 | 일치 |
| 2607.11683 | RAGU | 핵심 | upsert/update/delete 해시 기반 증분 유지(계층 Leiden). 삭제 뒤 커뮤니티 요약 갱신 미확인 | §2: "Incremental upsert/update/delete with deterministic hash-based IDs and merge policies, plus a consistency auditor"; Steps 4–6 Hierarchical Leiden + LLM 커뮤니티 리포트. 삭제 후 커뮤니티 리포트 처리 방식·평가는 본문에 없음(엔지니어링 기능 목록, Table 6) | 일치(미확인 → 본문에 기술 없음) |
| 2607.11683 | RAGU | 한계 | — | Limitations: 스케일링 근거가 Qwen2.5 한 계열, Meno-Lite는 32K 넘으면 다중홉 열화, NetworkX 백엔드 확장성 | 보완 |
| 2606.18075 | A Unified Framework for Context-Aware and Relation-Aware Graph Retrieval-Augmented Generation (HyGRAG) | 서지 | 2026 | WWW '26 (본문 헤더 "WWW '26, April 13–17, 2026, Dubai", arXiv comment) | 보완(venue) |
| 2606.18075 | HyGRAG | 핵심 | 동적 업데이트 시 국소 재요약; 삭제/수정 미확인 | §3.4 Dynamic Knowledge Update: 새 내용의 계층 attachment + 조상 커뮤니티 재요약(식 30). RQ3 실험은 증분 **삽입**만, 정적 대비 1–2% 하락. 삭제·수정 언급 없음 | 수정(삽입 전용으로 확정) |
| 2606.18075 | HyGRAG | 한계 | — | 부록 C: 커뮤니티 요약 환각률 17%(Musique 100개 수작업, gpt-4o-mini로 7%) | 보완 |
| 2603.10765 | RAGPerf: An End-to-End Benchmarking Framework for Retrieval-Augmented Generation Systems | 서지 | 2026 | arXiv 2026-03-11, venue 근거 없음 | 일치 |
| 2603.10765 | RAGPerf | 핵심 | insert/update/delete 워크로드에서 갱신 사실 검색 여부 확인, GraphRAG 전용 아님 | §3: Insert/Update/Removal 연산 정의; update에는 LLM으로 합성한 사실 변경 + QA를 만들어 "retrieves the updated fact rather than stale data" 확인. **Removal은 연산만 있고 삭제 사실 누출 QA는 없음**. §5.5는 LanceDB 벡터 인덱스 대상(임시 flat index 유무) | 일치(+보완) |
| 2601.05254 | TagRAG: Tag-guided Hierarchical Knowledge Graph Retrieval-Augmented Generation | 서지 | 2026 | arXiv comment "Accepted by ACL 2026 Findings"(meta published 2025-10-18) | 보완(venue) |
| 2601.05254 | TagRAG | 핵심 | 증분 삽입 전용 | §4.1.4 Knowledge Incremental Insertion(tag/knowledge increment), §6.4·6.6 증분 실험. 삭제 언급 없음 | 일치 |
| 2601.05254 | TagRAG | 한계 | — | Limitations: 색인이 LLM 호출에 의존(비용·재현성), 멀티미디어 미지원 | 보완 |
| ACM MM 2025 | T-GRAG / Time-LongQA | 전체 | 시간 충돌 다루는 동적 GraphRAG | 로컬 본문 없음 | 본문 접근 불가 |
| — | FalkorDB, LightRAG | 전체 | orphan 정리, provenance 기반 엣지 철회 | 논문 아님, 로컬 근거 없음 | 본문 확인 불가 |

## C2. 임베딩 압축 × 추론 집약 검색

| ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정 |
|---|---|---|---|---|---|
| 2510.12474 | SMEC: Rethinking Matryoshka Representation Learning for Retrieval Embedding Compression | 서지 | 2025 | EMNLP 2025 (arXiv comment) | 보완(venue) |
| 2510.12474 | SMEC | 벤치 | BEIR류 | §4.1: BEIR(텍스트), Products-10K(이미지), Fashion-200K(멀티모달). 양자화·BRIGHT 없음 | 보완 |
| 2510.12474 | SMEC | 한계 | — | Limitations: 특정 도메인 라벨 데이터로 소수 파라미터만 학습해 일반화 제한 | 보완 |
| 2606.01074 | When Is 0.1% Enough? Analyzing the Combined Effects of Dimensionality Reduction and Quantization on Text Embedding Compression | 수치 | MTEB 4개 task family, **4개 모델** | 초록 "four task families ... using **five** pretrained embedding models" | 수정(4→5 모델) |
| 2606.01074 | 동 | 벤치 | BRIGHT/LIMIT 미포함(스니펫 기준) | Retrieval = MIRACL, Quora, HotpotQA, DBpedia-Entity, NQ, MS MARCO 6개(hard-negative 버전). BRIGHT/LIMIT 없음 | 일치(확정) |
| 2606.01074 | 동 | 핵심 | 최적 압축은 태스크마다 다름 | 결론: classification은 0.1% 저장량으로 99% 유지 가능, retrieval이 가장 민감 | 일치 |
| 2606.01074 | 동 | 한계 | — | Limitations: MTEB 일부·영어만; 다른 도메인·태스크 유형으로 일반화 불명 | 보완 |
| 2508.21038 | On the Theoretical Limitations of Embedding-Based Retrieval (LIMIT) | 서지 | 2025 | ICLR 2026 (본문 헤더 + comment) | 보완(venue) |
| 2508.21038 | LIMIT | 핵심 | MRL truncation 시 큰 열화, 양자화 다루지 않음 | §5: 32차원까지 truncation sweep, 차원↑ 성능↑. 양자화는 §3 논의에서 언급("quantized or truncated")만, 실험 없음 | 일치 |
| 2506.20923 | KaLM-Embedding-V2 | 서지·핵심 | 부록에서 차원 축소 시 retrieval/reranking 하락 최대 | ICLR 2026. 부록 D Table 11: "retrieval tasks exhibit the largest performance drops" (MTEB cmn/eng) | 일치 |
| 2412.04506 | Arctic-Embed 2.0: Multilingual Retrieval Without Compromise | 핵심 | MRL+스칼라 양자화 결합해도 열화 작음(BEIR/MTEB) | §2.5: MRL을 256차원에서 학습해 "facilitates aggressive quantization" — 양자화는 Merrick(2024) 인용일 뿐 **본문 실험 없음**. truncation 256에서 MTEB-R 원점수 99%(m)/98%(l) 유지. 벤치 MTEB-R, CLEF, MIRACL | 수정 |
| 2412.04506 | Arctic-Embed 2.0 | 서지 | 2024 | comment "10 pages" → venue 근거 없음 | 일치 |
| 2608.21494 | Retrieval Needs Multivectors: An Exponential Separation | 핵심 | 멀티벡터 필요성 이론 | 초록: 단일벡터는 지수 크기 필요, 멀티벡터는 다항 크기로 충분; ANDOR 벤치 신규. 압축 실험 없음 | 일치(+ANDOR 보완) |
| 2609.29652 | SmallReason-ColBERT | 서지 | — | EMNLP 2026 Main (arXiv comment) | 보완 |
| 2609.29652 | SmallReason-ColBERT | 핵심 | 초소형 late-interaction 추론 검색기(BRIGHT), 압축과 인접 | 32M, BRIGHT 평균 nDCG@10 21.41 (Reason-ModernColBERT 22.62). **질의 토큰 gate를 0.95 미만에서 0으로 자르는 hard pruning은 BRIGHT 16.02로 하락**(Table 4, 10/12 split), Top-K MaxSim(K=4) 21.66. 투영 차원 64→128 확장. 문서측 인덱스 압축·양자화 실험은 없음 | 보완(C2 관련 단서) |
| 2510.08252 | ReasonEmbed | 핵심 | 압축 분석 없음 | 압축/양자화/MRL 실험 없음. ACL 2026 Main (comment) | 일치 |
| 2608.13200 | GEM: A Generative Embedding Model Bridging Reasoning and Retrieval | 핵심 | 압축 분석 없음 | 압축/양자화/MRL 언급 없음(truncation은 입력 길이) | 일치 |

## C3. Set-level utility / closed-book 반사실 / 생성기 전이

| ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정 |
|---|---|---|---|---|---|
| 2601.05027 | OptiSet: Unified Optimizing Set Selection and Ranking for Retrieval-Augmented Generation | 서지 | Findings EMNLP 2026 | 본문 헤더·arXiv comment(코드 링크뿐) 어디에도 venue 없음 | 수정(venue 근거 없음) |
| 2601.05027 | OptiSet | utility 정의 | 집합 조건부 perplexity 변화 | §3.2.2 식(5)–(7): PPL = exp(−1/N Σ log p(a_j\|q,S,a<j)), H = log PPL, "**We therefore compute the baseline entropy (w/o passages) H_o and define ∆H = H_p − H_o**", ∆H≤0이면 positive. 즉 **closed-book 대비 차분 utility를 이미 사용** | 수정(중요) |
| 2601.05027 | OptiSet | 학습 대상 | 검색기 학습처럼 서술 | LLaMA3.1-8B LoRA 기반 **LLM 집합 선택기/리랭커**(Contriever top-20 후보에서 집합 생성), dense bi-encoder 아님 | 수정 |
| 2601.05027 | OptiSet | 생성기 전이 | — | 단일 생성기. §4.9: "different generators may have different document combination preferences, and improving generalization is a future direction." held-out 생성기 평가 없음 | 보완 |
| 2601.05027 | OptiSet | 벤치 | — | HotpotQA, Bamboogle, MuSiQue, TriviaQA (EM/F1), 21M Wikipedia | 보완 |
| 2601.05027 | OptiSet | 한계 | — | Limitations: LLM 컨텍스트 길이로 후보 수 제한, RL 미사용, 동적 top-k라 증거 예산 통제 어려움 | 보완 |
| 2504.00573 | Training a Utility-based Retriever Through Shared Context Attribution for Retrieval-Augmented Language Models (SCARLet) | utility 정의 | perturbation 기반 utility 귀속 | §3.4 식(4)–(5): 섭동 벡터 v∈{0,1}^k로 passage를 제거하고, 정답 토큰 logit 합 z_i = Σ_t logit y_t 를 ridge 회귀(α̂)로 적합, α(i)=passage i utility. closed-book 차분을 명시적으로 정의하진 않음(빈 집합은 섭동의 한 경우일 뿐) | 일치(+정의 보완) |
| 2504.00573 | SCARLet | 전이 | — | 학습 신호 생성은 LLaMA-3-8B-Instruct(§5.3). 평가는 LLaMA-3-8B와 Qwen2.5-3B 두 생성기, 저자 표현 "preliminarily indicating its adaptability across generators"(§5.1). Qwen용을 따로 학습했는지는 본문에서 명확하지 않음 | 보완 |
| 2504.00573 | SCARLet | 서지·한계 | EMNLP 2025 | comment "EMNLP 2025 Main". Limitations: 소수 데이터셋, 코드 도메인 일반화 하락, 대형 검색기·생성기 미평가 | 일치 |
| 2025.acl-long.527 | GainRAG: Preference Alignment in Retrieval-Augmented Generation through Gain Signal Synthesis | utility 정의 | closed-book 대비 차분인지 미확인 | §3.2 식(4)–(5): contrastive decoding으로 logit을 (1+α)logit(a_t\|c,q,a<t) − α logit(a_t\|q,a<t)로 보정한 뒤 perplexity M(c,a\|q) 계산. "cannot distinguish whether the answer generation is driven by context c or the LLM's internal knowledge" → **closed-book 사전확률을 토큰 수준에서 빼는 반사실 보정**. 단일 passage, BGE-reranker 선택기, LLama3-8B | 수정(미확인 → 사실상 closed-book 보정) |
| 2025.acl-long.527 | GainRAG | 한계 | — | Limitations: passage 조합의 gain 미검증, 소량 학습 데이터, 작은 모델로 신호 생성 가능성 미검증 | 보완 |
| 2604.22722 | Aligning Dense Retrievers with LLM Utility via Distillation (UAE) | utility 정의 | perplexity 감소 기반 utility를 dense retriever로 증류 | §2.1 식(1): U(q,d,a) = exp(1/\|a\| Σ log p(x_t\|x<t,q,d)) — 본문 수식은 **절대 우도(역 perplexity)**이고 closed-book 항이 없음(서론은 "perplexity reduction"이라 표현). 단일 문서 pointwise, BERT reward model → bi-encoder 증류. 생성기 Llama-3-8B-Instruct 고정("we prioritize generator-specific alignment"), QASPER 학습, Token F1 +17.3% vs BGE-Base | 수정(정의 정밀화) |
| 2510.11358 | LLM-Specific Utility for Retrieval-Augmented Generation | utility 정의·핵심 | gold utility 집합이 LLM 간 전이되지 않음 | §3.1: **u(d;L,q) = Acc(L(q,d)) − Acc(L(q,∅))** (closed-book 대비 정확도 차). 4개 LLM(Qwen3-8B/14B/32B, Llama3.1-8B), 6개 QA. "model-dependent and non-transferable". 검색기 학습은 하지 않음(벤치·판단 과제 SpecUBench) | 일치(+정의 보완) |
| 2510.11358 | 동 | 서지·한계 | CIKM '26 | 본문 ACM Reference Format "CIKM '26" 확인. §3.3: pointwise utility라 passage 조합 효과 미반영 | 일치 |
| 2507.06838 | Shifting from Ranking to Set Selection for RAG (SetR) | 서지 | 2025 | ACL 2025 main (comment) | 보완 |
| 2510.21440 | Redefining Retrieval Evaluation in the Era of LLMs (UDCG) | 핵심 | 방해 문서 반영 집합 메트릭 | 초록: UDCG 제안, 상관 최대 36% 향상. venue 근거 없음 | 일치 |
| 2606.29959 | Know Before You Fetch: When Does Confidence-Based Retrieval-Budget Allocation Pay Off? | 핵심 | closed-book 충분성을 보정 확률로 예측, 게이팅용 | 초록: closed-book confidence로 질의별 0/1/5 passage를 할당하는 정책을 같은 평균 예산의 최적 고정 깊이와 비교; 희소 예산에서 +0.010~0.075. 검색기 학습 아님 | 일치(서술 정밀화) |

## C4. 인덱스 구축 전 예측기

| ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정 |
|---|---|---|---|---|---|
| 2506.05690 | When to use Graphs in RAG: A Comprehensive Analysis for Graph Retrieval-Augmented Generation (GraphRAG-Bench) | 서지 | ICLR 2026으로 인용됨, 미확인 | 본문 매 쪽 헤더 "Published as a conference paper at ICLR 2026" | 수정(확인됨) |
| 2506.05690 | 동 | 핵심 | 정성적·벤치마크 분석, 예측기 없음 | 의료 가이드라인(NCCN)+소설 코퍼스, 4종 과제. 부록 B "Takeaway findings"는 정성 지침("tightly connected communities"). 사전 예측 모델 없음 | 일치 |
| 2506.05690 | 동 | 한계 | — | 부록 J: 텍스트 단일 모달만 | 보완 |
| 2502.11371 | RAG vs. GraphRAG: A Systematic Evaluation and Key Insights | 서지 | 2025 | KDD '26 (본문 ACM Reference Format) | 보완(venue) |
| 2602.03578 | Use Graph When It Needs (EA-GraphRAG) | 핵심 | 질의 단위 MLP로 라우팅, 구축된 인덱스 전제 | §3: 구문 특징 → MLP adapter → 복잡도 점수 s(q), 임계값으로 RAG/GraphRAG/융합 라우팅; "a pre-built graph G=(V,E) over C", §3.1 Offline Indexing. 200개 샘플로 adapter 학습. 코퍼스 단위·구축 전 예측 없음. 본문 헤더 "Conference, Under Review" | 일치(확정) |
| 2503.09600 | MoC: Mixtures of Text Chunking Learners for RAG | 핵심 | 청킹 품질 평가 + mixture-of-chunkers | 초록: Boundary Clarity·Chunk Stickiness 두 지표, granularity-aware MoC | 일치 |
| 2607.26497 | BM25 Wins at Scale: A Scaling Study of Retrieval-Augmented Generation Paradigms | 핵심 | 규모에 따른 역전(재확인 못함) | 초록: 28개 중첩 규모(약 450배), 약 1천만 코퍼스 토큰에서 BM25가 File-System Agent를 추월, 그래프 RAG는 배포 규모 전에 "construction walls" | 수정(확인됨, 수치 보완) |
| 2610.10170 | Does Document Structure Help Dense Retrieval? A Placebo-Controlled Ablation of Four Mechanisms Across Two Corpora | 핵심 | 문서 구조 메커니즘 placebo 통제 ablation | 초록·§7: Wikipedia FA 200편(951 질의), QASPER 1,585편(4,303 질의). 구조 정렬 청킹 +0.022/+0.012, 계층 2단계 검색 −0.033/−0.015. §7에서 "section inventory" 크기가 계층 검색 성패를 설명한다고 사후 해석. 사전 예측기 없음. comment "Initial draft" | 일치(+보완) |
| 2610.10170 | 동 | 한계 | — | §8: 효과 작음(dz≤0.11), 구조 유도 모델 1개, 영어 2개 코퍼스 | 보완 |


---

## 배치 b05b

# b05b 검증표: 05_prior_art_check.md — C5, C6, C7, 요약표

출처: 로컬 본문 txt/<ID>.txt, meta/arxiv_meta.json. 웹 미사용. 모든 대상 논문 본문 접근 가능.

## C5. 문맥화 청킹의 문서 내 판별력 붕괴

| ID | 제목 | 항목 | 우리 기록 | 원문 근거(섹션/표/쪽) | 판정 |
|---|---|---|---|---|---|
| 2602.16974 | Beyond Chunk-Then-Embed: A Comprehensive Taxonomy and Evaluation of Document Chunking Strategies for Information Retrieval | venue | SIGIR 2026으로 표기(미확인) | 1쪽 머리말에 venue 없음, "Anonymoused for review"; arXiv comment는 익명 심사 중이라는 말뿐 | 수정: venue 근거 없음(익명 심사 중) |
| 2602.16974 | 〃 | 핵심 기여 | 문맥화 청킹이 in-corpus는 높이고 in-document는 떨어뜨림 | 초록; §4.4 Finding 3: "Contextualized chunking degrades in-document retrieval across all configurations" | 일치 |
| 2602.16974 | 〃 | 수치(추가) | — | §4.4: GutenQA에서 평균 하락 Nomic −53.17%, Jina-v3 −29.82%, Jina-v2 −5.16%, E5-large −3.83% | 추가 |
| 2602.16974 | 〃 | 벤치마크 | — | §3.3: in-document는 GutenQA, in-corpus는 BEIR 6종(SciFact, NFCorpus, FiQA, TREC-COVID 등) | 추가 |
| 2602.16974 | 〃 | 메커니즘 | 붕괴 현상 보고 | §4.4/§5: "같은 문서 청크끼리 더 비슷해진다"는 것은 **가설**일 뿐 측정하지 않았다. Late Chunking만 다루고 Anthropic 방식 LLM 접두사는 다루지 않음 | 보완(C5 진단 부분의 여지를 뒷받침) |
| 2602.16974 | 〃 | 한계 | — | Limitations 절 없음. §5 Conclusion은 "contextualized chunking은 in-corpus에만 써라"고 권고 | 명시된 한계 없음 |
| 2505.24782 | Context is Gold to find the Gold Passage: Evaluating and Training Contextual Document Embeddings | venue | EMNLP 2025 | 본문 머리말에 venue 없음, arXiv v2 comment "Under Review" | 수정: 로컬 venue 근거 없음 |
| 2505.24782 | 〃 | 핵심 기여 | 같은 문서 청크를 negative로 쓰는 대조 학습 | §4.2: InfoNCE에 in-sequence 대조 손실(같은 문서 청크 = hard negative)을 더하고, 목적을 "prevent embedding collapse"로 명시 | 일치 |
| 2505.24782 | 〃 | Anthropic CR | (C5는 "CR에서 붕괴를 정량화한 연구 없음"이라고 씀) | Table 2: "Anthropic Contextual"을 ConTEB 평균 72.4, NanoBEIR 63.2로 평가. "120x slower"라고 언급. 다만 집계 nDCG만 보고하고 문서 내 판별 지표는 없음 | 보완: CR을 평가하긴 했으나 붕괴는 정량화하지 않음 |
| 2505.24782 | 〃 | 수치 | — | §5: 순진한 late chunking은 −21 nDCG@10, 학습하면 +16 회복 | 추가 |
| 2505.24782 | 〃 | 벤치마크 | — | ConTEB(MLDR, SQuAD, NarrativeQA, COVID, ESG, Football, Cities, Insurance), NanoBEIR | 추가 |
| 2505.24782 | 〃 | 한계 | — | Limitations: 컨텍스트 길이(8k, 32k까지 외삽, 1M+ 미해결), 데이터 생성 자동화가 미해결 | 확인 |
| 2610.38854 | Contextual Embeddings Beyond the Gold Chunk | ID/제목 | "Contextual Embeddings for Document Retrieval"(ID 검증 불가) | 본문과 메타데이터 모두 제목은 "Contextual Embeddings Beyond the Gold Chunk", 2026-10-28, Perplexity AI + turbopuffer | 오류: 제목 수정, ID는 실재 |
| 2610.38854 | 〃 | 핵심 기여 | KL 목적 + 문서 수준 대조 손실, gold 라벨 없이 형제 청크 처리 | §2: query-aware context compression 교사의 청크 단위 graded relevance를 forward KL로 증류하고 문서 수준 InfoNCE를 더함. gold-chunk 라벨이 필요 없음. pplx-embed-v2-context 9B/0.6B 공개 | 일치(보강) |
| 2610.38854 | 〃 | 표의 "Perplexity 업계 발표(2026-10)" 행 | 별도 행 | 같은 작업물(pplx-embed-v2-context)의 논문이다 | 수정: 2610.38854와 같은 행으로 병합 |
| 2610.38854 | 〃 | training-free/디바이어싱 여부 | (확인 요청) | 학습 기반(증류) 방법이다. 평균 제거, PC 제거, 센터링, 디바이어싱 언급 0건. Anthropic CR은 baseline에 없음 | C5 여지 유지 |
| 2610.38854 | 〃 | 벤치마크 | — | context-bench, Q2D-Chunk(신규, in-document 변형 evidence recall 포함), ConTEB, ViDoRe-V3, MIRACL-Vision | 추가 |
| 2610.38854 | 〃 | 한계 | — | 별도 Limitations 절 없음. §6: "Evidence retrieval remains challenging" | 확인 |
| 2611.12097 | Calibrating Document-Local Boundaries for Contextual Chunk Retrieval | 존재/서지 | 스니펫 1건에서만 확인(미확인) | 본문 있음. Qwen Business Unit, Alibaba. 2026-11-09. 11쪽. venue 근거 없음 | 수정: 확인됨 |
| 2611.12097 | 〃 | 핵심 기여 | gold와 가장 어려운 형제 사이 경계 보정. "fix와 직접 겹칠 가능성 높음" | 초록/§3: DcLBC = AMS(gold와 가장 어려운 형제 margin으로 인스턴스별 scale을 정해 InSeNT 문서 내 손실에 적용) + CKR(post-RoPE key에 청크 파티션 offset 주입). **둘 다 학습(post-training) 방법**이고 post-hoc 공통 성분 제거는 아님 | 수정: 겹침은 "학습 기반 fix" 수준이고 training-free fix와는 겹치지 않음 |
| 2611.12097 | 〃 | 수치 | — | §1: InSeNT 평가 질의 19,974개 중 6,976개가 gold를 1위로 못 올림. 그중 82.21%는 가장 높은 형제 점수가 gold보다 높음. §5: InSeNT 대비 최대 +2.45 NDCG@10(ModernColBERT). Table 1: ModernBERT-large ConTEB 평균 78.61 vs InSeNT 76.70 | 추가 |
| 2611.12097 | 〃 | Anthropic CR | — | Table 1에 "Anthropic Contextual" 72.40 행이 있음(ConTEB 값 재인용). 분석 대상은 아님 | 보완 |
| 2611.12097 | 〃 | 벤치마크 | — | ConTEB, NanoBEIR. 백본 ModernBERT-base/large, ModernColBERT | 추가 |
| 2611.12097 | 〃 | 한계 | — | Limitations 절 없음(§5 Conclusion만 있음) | 명시된 한계 없음 |
| 2604.10167 | Visual Late Chunking: An Empirical Study of Contextual Chunking for Efficient Visual Document Retrieval | 핵심 기여 | 시각 문서에서의 문맥화 청킹 실증 연구 | 초록: ColChunk는 patch 임베딩을 2D 위치 prior와 함께 계층 군집해 페이지당 약 40개 멀티벡터로 만든다. 학습 없음. 저장 90% 이상 절감, 단일 벡터 대비 nDCG@5 평균 +9. 문서 내 판별 문제는 다루지 않음 | 수정(구체화) |
| 2604.10167 | 〃 | venue/벤치마크 | — | ACM 템플릿 자리표시자만 있고 comment는 "Preprint". 즉 venue 근거 없음. ViDoRe-V1/V2, VisRAG, ViDoSeek 등 24개 데이터셋 | 추가 |
| 2604.10167 | 〃 | 한계 | — | Limitations 절 없음. §4에 청크 수가 너무 적으면 해롭다는 언급 | 명시된 한계 없음 |
| 1702.01417 | All-but-the-Top: Simple and Effective Postprocessing for Word Representations | 서지/기여 | 공통 성분 제거, 고전 | 1쪽 "Published as a conference paper at ICLR 2018". 공통 평균 벡터와 상위 우세 방향 몇 개를 제거하는 단어 벡터 후처리. 문장 수준 STS에도 적용. 청크/문서 수준 적용은 없음 | 일치(venue ICLR 2018 추가) |

## C6. 멀티벡터·양자화 인덱스의 다세대 하위호환 업그레이드

| ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정 |
|---|---|---|---|---|---|
| 2025.emnlp-main.805 | Drift-Adapter: A Practical Approach to Near Zero-Downtime Embedding Model Upgrades in Vector Databases | 서지 | EMNLP 2025 | 1쪽 각주 "Proceedings of the 2025 Conference on EMNLP, pages 15938–15949" | 일치 |
| 〃 | 〃 | 핵심 기여 | 새 쿼리를 레거시 공간으로 매핑, 단일 벡터 | 초록: OP/Low-Rank Affine/Residual MLP adapter. 백엔드는 FAISS HNSW 단일 벡터. ColBERT, PQ, 양자화 언급 0건 | 일치 |
| 〃 | 〃 | 수치 | — | 초록: full re-embedding 대비 recall 95–99% 회복, 질의 지연 10µs 미만 증가, 재계산 비용 100배 이상 절감. MTEB 텍스트 + CLIP ViT-B→ViT-L(1M 항목) | 추가 |
| 〃 | 〃 | 한계 | — | Limitations: "chained or cascaded adaptations (A→B→C)의 오차 누적은 연구하지 않았다". 불균일 드리프트에는 단일 global adapter가 부적합 | 확인. C6의 (c) 다세대 체인 공백을 저자가 직접 확인 |
| 2506.00037 | Query Drift Compensation: Enabling Compatibility in Continual Learning of Retrieval Embedding Models | 서지/기여 | 쿼리 측 드리프트 보정 | 1쪽 "Published at CoLLAs 2025". §3: **태스크 전환마다 drift 벡터를 저장하고 누적합으로 옛 공간에 투영**. MS MARCO→NQ→HotpotQA→FEVER→FiQA, 5단계 연속 학습(nomic-embed, 768d, 단일 벡터) | 수정: **다세대 체인(5단계)을 이미 다룸**. 단, 같은 모델의 연속 미세조정이고 단일 벡터임 |
| 2506.00037 | 〃 | 한계 | — | 별도 Limitations 절 없음. 결론에서 "더 넓은 연속 검색 벤치마크"를 요청 | 확인 |
| 2609.16875 | Multi-modal Knowledge Preserving Adapter for Embedding Backward Compatibility | 서지/기여 | MLLM용 adapter-only BCT, 단일 벡터 | comment "ECCV 2026 camera ready". 초록: backbone 갱신 없는 adapter-only BCT, multi-level preservation loss + focal re-weighting. ViDoRe에서도 "각 문서를 단일 이미지로 취급"(CLIP→GME) | 일치(venue 추가) |
| 2609.16875 | 〃 | 한계 | — | Limitations 절 없음 | 명시된 한계 없음 |
| 2610.07266 | RAGFlip: Measuring Query-Level Negative Flips in Retriever Upgrades | 핵심 기여 | query 단위 negative flip 측정(BM25→dense), 호환성 학습 없음 | 초록: BM25→BGE-large/E5-large-v2/SPLADE, NQ/HotpotQA/FiQA, k∈{1,5,10,20,50}. k=1에서 BM25 성공의 8.6–37.5%를 잃음. 완화책은 고정 예산 hybrid(alternation, RRF, union)뿐 | 일치(SPLADE 포함으로 보완) |
| 2610.07266 | 〃 | 한계 | — | §6: BEIR qrel이 불완전, reader 실험은 7B 1개·HotpotQA·k=10으로 제한, FiQA 표본이 작음 | 확인 |
| 2201.09724 | Hot-Refresh Model Upgrades with Regression-Alleviating Compatible Training in Image Retrieval | 서지/기여 | RACT, 단일 벡터 | ICLR 2022(1쪽). §5.4 "Sequential model upgrades": old→1세대→2세대→3세대 연속 업그레이드를 실험(GLDv2) | 수정: **다세대(3세대) 연속 업그레이드 실험이 있음**(이미지, 단일 벡터) |
| 2303.04766 | FastFill: Efficient Compatible Model Update | 서지/기여 | backfilling 고전 | ICLR 2023. 불확실성 기반 feature alignment와 정책 기반 partial backfilling. ImageNet/Places-365/VGG-Face2. 2세대만 다룸 | 일치 |
| 2301.03767 | Metric Compatible Training for Online Backfilling in Large-Scale Retrieval | 서지/기여 | online backfilling | 본문/메타에 venue 근거 없음. distance rank merge + reverse transformation + metric-compatible contrastive. 이미지 4개 벤치마크 | 일치(venue 근거 없음) |
| 2302.08714 | Binary Embedding-based Retrieval at Tencent | 핵심 기여 | 이진 임베딩 + 하위호환 학습, 양자화와 호환성을 함께 다룬 드문 사례 | 초록/§3.2.3: recurrent binarization과 backward-compatible training(BEBR-BC)으로 "multiple embedding versions" 색인을 지원. 인덱스 비용 30–50% 절감. 단일 벡터이고, 양자화는 PQ/RaBitQ가 아닌 학습형 이진화 | 일치(구체화). venue 근거 없음 |
| 2606.05568 | ColBERTSaR: Sparsified ColBERT Index via Product Quantization | 서지/기여 | PQ로 ColBERT 인덱스 희소화, 업그레이드 없음 | comment "accepted at SIGIR 2026 as a short paper". 잔차 없는 PQ로 ColBERT를 inverted index로 변환. 1-bit PLAID보다 50–70% 작음. BEIR, NeuCLIRBench. 업그레이드·호환성 언급 없음 | 일치(venue·수치 추가) |

## C7. 검색 여부·확장·k·압축을 한 예산 아래 결합하고 conformal 보장

| ID | 제목 | 항목 | 우리 기록 | 원문 근거 | 판정 |
|---|---|---|---|---|---|
| 2606.29959 | Know Before You Fetch: When Does Confidence-Based Retrieval-Budget Allocation Pay Off? | 핵심 기여 | closed-book/k=1/k=5/abstain 중 **보정된 확률**로 선택 | 초록/§1: 행동은 0/1/5 passage, **abstain 없음**. closed-book 신뢰도 threshold를 쓰고, §5.5에서 보정을 해도 결정이 바뀌지 않음(50/50 조합). 예산은 질의당 평균 passage cap. probe 지연까지 계산 | 오류: abstain 삭제, "보정된 확률" 대신 "신뢰도 threshold" |
| 2606.29959 | 〃 | 수치 | — | 초록: cap 0.5에서 7개 설정 모두 best simple policy 대비 +0.010~0.075, Wikipedia Qwen3-8B TriviaQA +0.051. 1 vs 5 passage 선택은 우연 수준 | 추가 |
| 2606.29959 | 〃 | 보장 | — | §7.1: 400개 pilot의 conservative 규칙도 "5% 위반률을 보장하지 않는다"고 명시 | 보완: 분포 무관 보장 없음 |
| 2606.29959 | 〃 | 한계 | — | §7.3: 짧은 답 factoid QA, 동결 reader(Qwen3), 설정당 retriever 1개 | 확인 |
| 2607.24010 | When Should Active RAG Retrieve? A Budget-Aware Evaluation of Utility, Calibration, and Cost | 서지 | — | KDD 2026 Workshop on Evaluation and Trustworthiness of Agentic AI(comment) | 추가 |
| 2607.24010 | 〃 | 핵심 기여 | 예산 인지 평가 프로토콜(실현 사용률, threshold 전이 오차), 단순 불확실성 ≈ 학습 라우터 | 초록 일치. 추가로 §3 "Budget-safe calibration": DKW형 slack ε=√(log(2/δ)/2n)으로 보수적 threshold를 정해 **실현 사용률 위반을 줄임**(경험적으로 다루고, 보장으로 주장하지는 않음). 행동은 이진(검색/비검색) | 수정: 실현 예산 보정은 이진 gating에서 이미 시도됨 |
| 2607.24010 | 〃 | 수치/벤치 | — | HotpotQA/2Wiki/MuSiQue 각 2,000개, Qwen2.5-1.5B. 명목 50% 목표에서 HotpotQA는 40–100% split이 초과 | 추가 |
| 2607.24010 | 〃 | 한계/향후 | — | §6: 후보 문단 검색, 소형 모델. 향후 과제로 "retrieval timing과 context compression이 함께 예산을 결정하는 long-context 과제" | 확인. C7 공백을 직접 지지 |
| 2606.02245 | When Knowledge Is Not Free: Cost-Aware Evidence Selection in Retrieval-Augmented Generation | 핵심 기여 | 비용이 다른 증거를 예산 아래 선택, LLM 컨트롤러 불안정 | 초록: MS MARCO v2.1에 접근 비용 tier를 부여. 고정 selector 중 우월한 것 없음. 에이전트는 "highly model- and task-dependent". HotpotQA/NQ/TriviaQA/MedQA-US/MMLU-Med | 일치 |
| 2606.02245 | 〃 | 한계 | — | §8: 비용 tier는 시뮬레이션, 자동 tier 주석 오류, 5개 QA·부분 표본 | 확인 |
| 2610.34872 | Adaptive Retrieval, Selective Answers: Evaluating Conformal Factuality in Agentic RAG | 핵심 기여 | 에이전트형 RAG에 conformal factuality | 초록: 생성 **후** claim 필터링에만 conformal을 적용. 검색 비용(최대 3회)은 고정이고 보장 대상이 아님. HotpotQA/NQ/TriviaQA 각 60문항 | 일치(범위 명시) |
| 2610.34872 | 〃 | 한계 | — | §7: 데이터셋당 60문항, 고정된 bounded policy, 모델 2종 | 확인 |
| 2511.17908 | Principled Context Engineering for RAG: Statistical Guarantees via Conformal Prediction | 서지/기여 | conformal 필터로 컨텍스트 크기 제어, coverage 보장 | ECIR 2026(comment). 보장 대상은 **관련 snippet coverage**뿐이고, 컨텍스트 크기(2–3배 감소)는 결과로 나오는 것이지 보장되지 않음. NeuCLIR, RAGTIME | 수정: "크기 제어 보장" 아님 |
| 2511.17908 | 〃 | 한계 | — | §5: exchangeability 가정, 분포 이동 하 재보정은 향후 과제 | 확인 |
| 2307.04642 | TRAQ: Trustworthy Retrieval Augmented Question Answering via Conformal Prediction | 서지/기여 | 검색 집합과 생성 집합 각각 conformal, NAACL 2024 | comment NAACL 2024. 두 집합을 union bound로 결합해 end-to-end 보장. Bayesian optimization으로 α 배분을 골라 집합 크기를 평균 16.2% 줄임(크기는 보장 대상 아님). NQ/TriviaQA/SQuAD-1/BioASQ | 일치(보완) |
| 2402.03181 | C-RAG: Certified Generation Risks for Retrieval-Augmented Language Models | 제목/기여 | "Conformal risk control for RAG configs"(제목 미확인, C-RAG 계열로 보임) | 실제 제목 C-RAG, ICML 2024(comment, PMLR 235). 설정 λ=[N_rag, λ_g, λ_s]에 대해 conformal generation risk 상한을 주고, FWER(Bonferroni 등, LTT류) 다중검정으로 위험 ≤ α인 **valid configuration set**을 산출. 비용/예산 제약은 없고 전역 설정이며 질의별 정책은 아님. AESLC/CommonGen/DART/E2E | 수정: 제목 확정. LTT식 설정 선택은 이미 존재 |
| 2609.22100 | AdaMem: Adaptive Memory Token Allocation for Soft Compression in Retrieval-Augmented Generation | 핵심 기여 | soft compression 토큰 예산을 passage별 할당 | 초록 일치. 6개 ODQA. 16배 압축에서 OSCAR 대비 최대 +3.2점, 64배 압축에서 평균 상대 +14.6%, 지연은 full context 대비 최대 4배 낮음. venue 근거 없음 | 일치 |
| 2511.09803 | Retrieval as a Decision: Training-Free Adaptive Gating for Efficient RAG | 서지/기여 | TARG, 학습 없는 게이팅 | "Published in TMLR (04/2026)". 짧은 no-context draft의 prefix logit 불확실성(entropy/margin/variance)으로 gating. 검색 70–90% 감소. NQ-Open/TriviaQA/PopQA/MuSiQue/ASQA. 보장 없음 | 일치(venue 추가) |
| 2608.19535 | From Retrieved Context to Runtime Control: Adaptive Compression for Edge-based RAG | 핵심 기여 | 런타임 압축률 제어 | 초록/§5: **비전 논문**이다. Jetson AGX Thor에서 LLMLingua-2 압축 trade-off를 측정(GPU 에너지 최대 −53.2%)하고 telemetry 기반 런타임 controller를 "주장"하지만 controller는 구현하지 않음. ACM AI Leadership Summit 2026 | 수정: "제어 방법" 대신 "필요성 실측과 비전 제시" |
| 2508.06165 | UR²: Unify RAG and Reasoning through Reinforcement Learning | 서지/기여 | ACL 2026, RL로 검색·추론 조정, 요약 압축 포함 | 1쪽 "ACL 2026 (Oral Presentation)". difficulty-aware curriculum과 LLM 요약 corpus를 결합한 hybrid knowledge access. 예산 제약이나 보장은 없음 | 일치 |
| 2508.06165 | 〃 | 한계 | — | Limitations: 8B까지만 실험, LLM 요약 corpus가 실제 웹 복잡성을 반영하지 못할 수 있음 | 확인 |
| 2604.15621 | Rethinking the Necessity of Adaptive Retrieval-Augmented Generation through the Lens of Adaptive Listwise Ranking | 서지/기여 | AdaRankLLM, k/pruning 적응(호출자 제공) | 제목 확인. zero-shot listwise ranker + passage dropout(종료 토큰 포함)과 2단계 증류. ASQA/QAMPARI/ELI5, LLM 8종. 보장 없음. venue 근거 없음 | 일치(제목 보완) |
| Provence | — | — | (호출자 제공) | 본문 파일 없음(2608.19535 참고문헌에만 있음, arXiv 2501.16214) | 본문 접근 불가 |
| (12편 공통) | — | 질의 확장 유형 축 | "질의 확장은 없음" | 12편 모두 'query expansion'/HyDE를 행동으로 다루지 않음(grep) | 일치 |
