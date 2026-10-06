# Spec thiết kế: Báo cáo NCKH — Triển khai mô hình AI trong lĩnh vực Chatbot/Trợ lý ảo (RAG & Fine-tuning) cho Tổng công ty Bưu điện Việt Nam

- **Ngày:** 2026-10-06
- **Định dạng:** LaTeX, XeLaTeX + fontspec, class `report`
- **Ngôn ngữ:** Tiếng Việt
- **Quy mô mục tiêu:** ~60 trang
- **Trọng tâm:** RAG và fine-tuning cân bằng; đề xuất kiến trúc lai
- **Nguồn thực nghiệm:** Hệ thống multi-agent + RAG thực tế tại VNPost (trong `~/Documents`), bỏ qua các dự án LangChain/LangGraph
- **Bìa:** dùng placeholder

## 1. Mục tiêu & phạm vi

Xây dựng một báo cáo nghiên cứu khoa học hoàn chỉnh, có thực nghiệm, về việc triển khai trợ lý ảo/chatbot thông minh cho Tổng công ty Bưu điện Việt Nam (VNPost) dựa trên:
1. **RAG** (retrieval-augmented generation) — giải pháp đã triển khai thực tế.
2. **Fine-tuning** (SFT, LoRA/QLoRA) — giải pháp so sánh/đề xuất.
3. **Kiến trúc lai RAG + fine-tuning** — đề xuất của đề tài.

Phạm vi dữ liệu: văn bản nội bộ VNPost (quy trình, quyết định, biểu phí, quy định bưu chính), nghiệp vụ tra cứu bưu gửi và bưu cục, dịch vụ EMS.

## 2. Đối tượng đọc & tiêu chí thành công

- Đối tượng: hội đồng/giảng viên chấm NCKH, kỹ sư AI tại VNPost.
- Thành công khi:
  - Đọc hiểu mạch lạc từ cơ sở lý thuyết → kiến trúc → thực nghiệm → kết quả.
  - Có số liệu thực nghiệm cụ thể, có nguồn, không bịa đặt.
  - Phần fine-tuning minh bạch: thiết kế + kết quả minh họa được ghi rõ.
  - Biên dịch LaTeX thành công bằng XeLaTeX, ra PDF ~60 trang.

## 3. Cấu trúc chi tiết

### Phần đầu (~5 trang)
- Trang bìa (placeholder: `[Tên trường/đơn vị]`, `[Khoa]`, `[Tác giả]`, `[MSSV]`, `[GVHD]`, `[Năm]`).
- Lời cảm ơn.
- Tóm tắt (tiếng Việt + Abstract tiếng Anh).
- Mục lục, Danh mục hình, Danh mục bảng, Danh mục từ viết tắt.

### Chương 1 — Giới thiệu (≈6 tr.)
1.1 Bối cảnh và lý do chọn đề tài (chuyển đổi số VNPost, khối lượng văn bản/tra cứu lớn).
1.2 Mục tiêu nghiên cứu.
1.3 Đối tượng và phạm vi nghiên cứu.
1.4 Phương pháp nghiên cứu (nghiên cứu tài liệu, thiết kế hệ thống, thực nghiệm, đánh giá định lượng/định tính).
1.5 Đóng góp của đề tài.
1.6 Cấu trúc báo cáo.

### Chương 2 — Cơ sở lý thuyết và tổng quan (≈12 tr.)
2.1 Chatbot và trợ lý ảo: phân loại, tiến hóa, kiến trúc hội thoại.
2.2 Mô hình ngôn ngữ lớn: kiến trúc Transformer, tokenization, pre-training, instruction tuning, decoding, prompting (zero/few-shot, CoT).
2.3 Retrieval-Augmented Generation:
   - 2.3.1 Khái niệm và động lực.
   - 2.3.2 Kiến trúc RAG (indexing, retrieval, generation).
   - 2.3.3 Chunking & tiền xử lý tài liệu (OCR, markdown hóa, header-aware/table-safe splitter).
   - 2.3.4 Embedding & vector database (Qdrant).
   - 2.3.5 Retrieval: dense, sparse BM25, hybrid.
   - 2.3.6 Reranking (cross-encoder Qwen3-Reranker).
   - 2.3.7 Các biến thể: naive/advanced/modular RAG, GraphRAG, agentic RAG.
2.4 Fine-tuning mô hình ngôn ngữ lớn:
   - 2.4.1 Full fine-tuning vs PEFT.
   - 2.4.2 SFT, instruction tuning, RLHF, DPO.
   - 2.4.3 LoRA và QLoRA (toán học, tham số, cấu hình).
   - 2.4.4 Khi nào dùng fine-tuning.
2.5 So sánh RAG vs fine-tuning và kiến trúc lai.
2.6 Thước đo đánh giá:
   - Retrieval: Recall@k, Precision@k, MRR, nDCG.
   - Generation/RAG: Faithfulness, Answer Relevancy, Context Precision, Context Recall (RAGAS), LLM-as-judge, BLEU/ROUGE.
2.7 Nền tảng triển khai: vLLM, quantization (FP8), serving on-prem; giao thức MCP, A2A; observability (OpenTelemetry, Langfuse, Grafana); MLflow Prompt Registry.

### Chương 3 — Kiến trúc hệ thống đề xuất (≈12 tr.)
3.1 Đặc thù bài toán tại VNPost (dữ liệu hành chính, biểu phí, quy trình ISO, tra cứu bưu gửi/bưu cục, EMS).
3.2 Yêu cầu chức năng và phi chức năng (chính xác, ngữ cảnh hóa, bảo mật dữ liệu nội bộ, on-prem, độ trễ).
3.3 Kiến trúc tổng thể multi-agent: host orchestrator + specialist agents (RAG sản phẩm/dịch vụ, tra cứu bưu gửi, tra cứu bưu cục, chất lượng phát).
3.4 Pipeline RAG chi tiết (indexing → retrieval hybrid → rerank → generation) với sơ đồ.
3.5 Pipeline fine-tuning đề xuất: thu thập dữ liệu hội thoại → tạo dataset SFT → LoRA/QLoRA → serving adapter (vLLM).
3.6 Kiến trúc lai RAG + fine-tuning: router quyết định, khi nào dùng RAG, khi nào dùng model đã fine-tune, khi nào kết hợp.
3.7 Hạ tầng và bảo mật: Kubernetes, vLLM, mô hình on-prem (Qwen3.6-35B-A3B-FP8, Qwen3.8-27B-FP8, GPT-OSS-20B), embedding/reranker Qwen3, không để dữ liệu ra ngoài.

### Chương 4 — Triển khai thực nghiệm (≈14 tr.)
4.1 Môi trường và công cụ (Python, Docker Compose, K8s, Qdrant, vLLM, MLflow, Langfuse).
4.2 Dữ liệu:
   - Corpus văn thư (17 tài liệu), corpus sản phẩm/dịch vụ tra cứu (119 tài liệu), corpus EMS (~30 tài liệu).
   - Bộ đánh giá EMS 99 câu hỏi/đáp (từ `test ems.xlsx`).
   - Tiền xử lý: OCR (olmocr-2-7b/lightonocr2-1b), markdown hóa, metadata hóa (title/ref_no/date/summary).
4.3 Cài đặt RAG: chunking, embedding Qwen3 (4096-d), hybrid BM25+dense, reranker, tham số top-k (20/20/5), ngưỡng điểm, prompt hệ thống.
4.4 Cài đặt fine-tuning: xây dựng dataset SFT từ log hội thoại + dữ liệu tổng hợp; cấu hình LoRA (r, alpha, target modules); quy trình huấn luyện; serving adapter.
4.5 Cài đặt multi-agent + MCP/A2A, rule định tuyến.
4.6 Observability và prompt management (MLflow prompt `@production`, Langfuse, Grafana).
4.7 Khó khăn và giải pháp (dữ liệu tiếng Việt, BM25 tiếng Việt, reranker, chi phí hạ tầng).

### Chương 5 — Kết quả và đánh giá (≈12 tr.)
5.1 Phương pháp đánh giá (harness retrieval, LLM-as-judge, RAGAS-4).
5.2 Kết quả retrieval:
   - Recall@20 BM25 = 50%, Dense = 100%, trước rerank = 100%, Recall@5 sau rerank = 50%, MRR = 0.3149.
   - Phân tích lỗi: 50% lỗi do reranker xếp hạng, 0% do recall.
5.3 Kết quả generation/agent: LLM-as-judge 0–10; RAGAS-4 trên 99 câu (thiết kế + kết quả minh họa).
5.4 So sánh RAG vs fine-tuning vs kiến trúc lai (bảng + phân tích).
5.5 Hiệu năng hệ thống: throughput vLLM (single-stream ~180 tok/s Qwen3.8-27B; batch ~3.137 tok/s @1024 req; các mô hình khác), độ trễ.
5.6 Case study hội thoại định tính.
5.7 Hiệu quả kinh tế – xã hội (tiết kiệm ~2.167 giờ công/ngày, ~32,5 tỷ VNĐ/năm theo sáng kiến ĐKCL; giảm thời gian tra cứu).
5.8 Thảo luận (ưu điểm, hạn chế, tính khái quát).

### Chương 6 — Kết luận và hướng phát triển (≈4 tr.)
6.1 Kết luận.
6.2 Hạn chế.
6.3 Hướng phát triển (GraphRAG, agentic RAG, continual learning, fine-tuning định kỳ, mở rộng đa ngôn ngữ).

### Phần cuối (~5 tr.)
- Tài liệu tham khảo (IEEE/numeric hoặc APA; gồm bài báo RAG, LoRA, Transformer, BM25, Qdrant, vLLM...).
- Phụ lục: prompt hệ thống mẫu, cấu hình tham số, bảng biểu bổ sung.

## 4. Danh mục hình & bảng dự kiến

**Hình:**
- Kiến trúc Transformer; minh họa RAG tổng quát; pipeline indexing; hybrid retrieval + rerank; LoRA; kiến trúc multi-agent VNPost; kiến trúc lai; luồng observability; biểu đồ Recall/MRR; biểu đồ throughput.

**Bảng:**
- So sánh RAG vs fine-tuning; bảng tham số RAG; cấu hình LoRA; kết quả retrieval; kết quả LLM-judge/RAGAS; benchmark throughput; tham số mô hình.

## 5. Ánh xạ dữ liệu thực nghiệm (nguồn trong `~/Documents`)

| Nội dung | Nguồn |
|---|---|
| Kiến trúc RAG hybrid + tham số top-k | `agent.template.rag/.../ARCHITECTURE.md`, `mcp_server/tools.py`, `.env` |
| Kết quả retrieval (Recall/MRR) | `agent.template.rag/.../eval/retrieval/SAMPLE_eval_retrieval_input_results.xlsx` |
| Kết quả agent eval | `.../eval/agent/SAMPLE_eval_agent_input_results.xlsx` |
| Multi-agent TCT | `chatbottct.deployment/docker-compose.yml`, các `ARCHITECTURE.md`, `system_prompt.md` |
| Corpus & metadata | `chatbottct.deployment/tracuu-spdv-rag-data/`, `agent.template.rag/rag-data/` |
| Bộ 99 câu hỏi EMS | `tool-test-ems-rag/test ems.xlsx`, `results/ragas4_responses.jsonl` |
| RAGAS-4 & LLM-judge | `tool-test-ems-rag/evaluate_ragas4.py`, `ragas_eval.py`, `llm_eval.py` |
| Benchmark vLLM | `dockers/test_vllm_speed/benchmark_results_*.json` |
| Hiệu quả kinh tế | `sk/02-dkcl-bao-cao-phan-quyen-airflow/sections/*.tex` |
| Observability / MLflow prompt | `mipong.hostagent`, `prompt-mlfow-test/t.py`, `mlflow/` |

Ghi chú minh bạch: codebase hiện **không có pipeline fine-tuning**; phần fine-tuning trong báo cáo là **thiết kế đề xuất + kết quả mô phỏng minh họa**, được ghi rõ trong báo cáo.

## 6. Cấu trúc file dự án LaTeX

```
NCKH-LAB-AI/
├── main.tex                 % file gốc
├── references.bib           % tài liệu tham khảo
├── chapters/
│   ├── 00-frontmatter.tex   % bìa, lời cảm ơn, tóm tắt
│   ├── 01-gioi-thieu.tex
│   ├── 02-co-so-ly-thuyet.tex
│   ├── 03-kien-truc.tex
│   ├── 04-trien-khai.tex
│   ├── 05-ket-qua.tex
│   ├── 06-ket-luan.tex
│   └── 99-phu-luc.tex
├── figures/                 % hình (TikZ hoặc PDF)
├── styles/                  % gói, cấu hình
├── Makefile / build.sh
└── docs/superpowers/specs/...
```

## 7. Rủi ro & giảm thiểu

- **Không biên dịch được XeLaTeX do thiếu font tiếng Việt:** dùng font an toàn (Latin Modern / TeX Gyre) + kiểm tra `xelatex` có sẵn; nếu thiếu, cung cấp hướng dẫn cài.
- **Số liệu fine-tuning không có thật:** ghi rõ là minh họa, tách khỏi số liệu RAG thật.
- **Độ dài ~60 trang:** phân bổ theo chương như trên; nếu thiếu trang, mở rộng phần thảo luận và phụ lục.
- **Trùng lặp thông tin giữa các chương:** kiểm tra chéo khi viết.

## 8. Ngoài phạm vi (YAGNI)

- Không viết code huấn luyện fine-tuning chạy thật.
- Không cào thêm dữ liệu mới.
- Không xây dựng/demo web.
- Không dịch báo cáo sang tiếng Anh (chỉ có Abstract tiếng Anh).
