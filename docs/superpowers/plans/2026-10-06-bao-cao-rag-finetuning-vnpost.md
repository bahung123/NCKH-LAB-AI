# Kế hoạch triển khai: Báo cáo NCKH — Triển khai AI Chatbot/Trợ lý ảo cho Vietnam Post (RAG & Fine-tuning)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Tạo dự án LaTeX (XeLaTeX, tiếng Việt) biên dịch thành PDF ~60 trang cho báo cáo NCKH về trợ lý ảo RAG + fine-tuning tại Tổng công ty Bưu điện Việt Nam.

**Architecture:** Dự án LaTeX phân rã theo chương: `main.tex` nạp `styles/common.tex` (cấu hình chung) và `\input` từng chương trong `chapters/`. Tài liệu tham khảo bằng `references.bib` + `biblatex`/`bibtex`. Hình vẽ bằng TikZ (nhúng trực tiếp, không cần file ngoài). Biên dịch kiểm thử bằng Tectonic (đã cài tại `/tmp/opencode/tectonic`) vì máy chưa có TeX Live; source tương thích cả Overleaf.

**Tech Stack:** LaTeX (XeLaTeX), fontspec + babel vietnamese, report class, amsmath, booktabs, tikz/pgfplots, biblatex, hyperref, glossaries/viết tắt thủ công.

**Spec:** `docs/superpowers/specs/2026-10-06-bao-cao-rag-finetuning-vnpost-design.md`

## Global Constraints

- Ngôn ngữ: **tiếng Việt**; `\usepackage[vietnamese]{babel}`; UTF-8.
- Engine: **XeLaTeX**; `\usepackage{fontspec}` với fallback: `\IfFontExistsTF{DejaVu Serif}{...}{}` (nếu không có DejaVu thì dùng font mặc định Latin Modern — đã kiểm chứng render đúng dấu tiếng Việt).
- Class: `report`, cỡ chữ 12pt, `a4paper`, lề 3cm/2.5cm/3.5cm/2cm (trái/phải/trên/dưới).
- Độ dài mục tiêu: **~60 trang** (chấp nhận 55–68).
- Mọi số liệu thực nghiệm RAG/agent/hiệu năng/hiệu quả kinh tế **phải có nguồn thật** trong `~/Documents` và được ghi nguồn; phần fine-tuning **phải ghi rõ là thiết kế đề xuất + kết quả minh họa**.
- Không dùng placeholder văn bản kiểu "TODO"; chỉ placeholder thông tin bìa theo spec.
- Không sửa/không phụ thuộc file ngoài dự án; sao chép số liệu vào nội dung chương.
- Biên dịch kiểm thử: `cd <project> && /tmp/opencode/tectonic -X compile main.tex` (Tectonic tự tải package). Overleaf: đặt `main.tex` làm root, compiler XeLaTeX.

## Review Focus

- **Font tiếng Việt thiếu ở máy khác:** nếu `DejaVu Serif` không tồn tại, phải rơi về font mặc định mà vẫn hiển thị đủ dấu (kiểm tra bằng render PDF→PNG, không chỉ "compile OK").
- **Trích dẫn/tài liệu tham khảo:** thiếu key trong `.bib` hoặc lệnh `\cite` sai → cảnh báo undefined; phải biên dịch đủ vòng để mục lục/TOC và bibliography ổn định.
- **Tham chiếu hình/bảng:** mọi `\ref`/`\label` phải phân giải được (không hiện `??`).
- **Độ dài:** sau khi viết xong phải đếm số trang thật và điều chỉnh để đạt ~60.
- **Tính trung thực số liệu:** không biến kết quả minh họa fine-tuning thành "kết quả thật"; không gán số liệu RAG của corpus này sang corpus khác.

---

### Task 0: Khởi tạo repo và khung dự án LaTeX

**Files:**
- Create: `.gitignore`
- Create: `main.tex`
- Create: `styles/common.tex`
- Create: `build.sh`
- Create: `chapters/00-frontmatter.tex`, `chapters/01-gioi-thieu.tex`, `chapters/02-co-so-ly-thuyet.tex`, `chapters/03-kien-truc.tex`, `chapters/04-trien-khai.tex`, `chapters/05-ket-qua.tex`, `chapters/06-ket-luan.tex`, `chapters/99-phu-luc.tex` (khung tối thiểu)
- Create: `references.bib` (rỗng, sẽ điền dần)

**Interfaces:**
- Produces: `main.tex` là entry point; `styles/common.tex` định nghĩa mọi gói + macro dùng chung (ví dụ `\vnpost`, `\code{}`, môi trường `\begin{ghichu}`); các chương được `\input` theo thứ tự.

- [ ] **Step 1: Khởi tạo git**

```bash
cd /home/bahung/Documents/NCKH-LAB-AI
git init -q
git config user.email "nckh@example.com" 2>/dev/null || true
git config user.name "NCKH LAB AI" 2>/dev/null || true
```

- [ ] **Step 2: Viết `styles/common.tex`**

```latex
% styles/common.tex — cấu hình chung
\usepackage{fontspec}
\IfFontExistsTF{DejaVu Serif}{%
  \setmainfont{DejaVu Serif}%
  \setsansfont{DejaVu Sans}%
  \setmonofont{DejaVu Sans Mono}%
}{}
\usepackage[vietnamese]{babel}
\usepackage{amsmath,amssymb}
\usepackage{booktabs}
\usepackage{longtable}
\usepackage{array}
\usepackage{graphicx}
\usepackage{xcolor}
\usepackage{tikz}
\usetikzlibrary{arrows.meta,positioning,shapes.geometric,fit,backgrounds,calc}
\usepackage{pgfplots}
\pgfplotsset{compat=1.18}
\usepackage{enumitem}
\usepackage{listings}
\usepackage{fancyhdr}
\usepackage{geometry}
\usepackage{titlesec}
\usepackage{caption}
\usepackage[hidelinks]{hyperref}
\geometry{a4paper,top=3.5cm,bottom=2.5cm,left=3cm,right=2cm}
\captionsetup{font=small,labelfont=bf}
\setlist{itemsep=2pt,topsep=3pt}
\definecolor{vnpostgreen}{RGB}{0,120,60}
\definecolor{lightgray}{RGB}{245,245,245}
% Môi trường ghi chú minh bạch
\newenvironment{ghichu}{\begin{quote}\itshape\small}{\end{quote}}
% Macro
\newcommand{\code}[1]{\texttt{#1}}
\newcommand{\vnpost}{Tổng công ty Bưu điện Việt Nam}
\newcommand{\rag}{RAG}
\newcommand{\ft}{fine-tuning}
% listings style
\lstset{basicstyle=\ttfamily\small,backgroundcolor=\color{lightgray},frame=single,
  breaklines=true,columns=fullflexible,showstringspaces=false}
```

- [ ] **Step 3: Viết `main.tex`**

```latex
\documentclass[12pt,a4paper]{report}
\input{styles/common.tex}
\addbibresource{references.bib}
\begin{document}
\input{chapters/00-frontmatter.tex}
\tableofcontents
\clearpage
\listoffigures
\clearpage
\listoftables
\clearpage
\input{chapters/01-gioi-thieu.tex}
\input{chapters/02-co-so-ly-thuyet.tex}
\input{chapters/03-kien-truc.tex}
\input{chapters/04-trien-khai.tex}
\input{chapters/05-ket-qua.tex}
\input{chapters/06-ket-luan.tex}
\input{chapters/99-phu-luc.tex}
\printbibliography[title={Tài liệu tham khảo}]
\end{document}
```

- [ ] **Step 4: Tạo khung rỗng cho các chương**

Mỗi file `chapters/NN-*.tex` tối thiểu chứa `\chapter{Tên chương}` + một câu mô tả. `00-frontmatter.tex` chứa `\titlepage` placeholder.

- [ ] **Step 5: Viết `build.sh` và `.gitignore`**

```bash
#!/usr/bin/env bash
set -euo pipefail
TECTONIC="${TECTONIC:-/tmp/opencode/tectonic}"
if command -v xelatex >/dev/null 2>&1; then
  xelatex -interaction=nonstopmode main.tex
  bibtex main || true
  xelatex -interaction=nonstopmode main.tex
  xelatex -interaction=nonstopmode main.tex
elif [ -x "$TECTONIC" ]; then
  "$TECTONIC" -X compile main.tex
else
  echo "Cần cài TeX Live (xelatex) hoặc đặt biến TECTONIC." >&2; exit 1
fi
```

`.gitignore`: `*.aux`, `*.log`, `*.toc`, `*.out`, `*.bbl`, `*.bcf`, `*.blg`, `*.run.xml`, `*.fls`, `*.fdb_latexmk`, `main.pdf` (cho phép `final/main.pdf`).

- [ ] **Step 6: Biên dịch kiểm thử**

Run: `cd /home/bahung/Documents/NCKH-LAB-AI && /tmp/opencode/tectonic -X compile main.tex`
Expected: tạo `main.pdf`, không lỗi. Nếu biblatex lỗi vì `references.bib` rỗng, thêm `\nocite{*}` tạm hoặc 1 entry mẫu.

- [ ] **Step 7: Commit**

```bash
cd /home/bahung/Documents/NCKH-LAB-AI
git add .gitignore main.tex styles/common.tex build.sh chapters references.bib
git commit -q -m "chore: khung dự án LaTeX báo cáo NCKH"
```

---

### Task 1: Phần đầu — bìa, lời cảm ơn, tóm tắt, danh mục từ viết tắt

**Files:**
- Modify: `chapters/00-frontmatter.tex`

**Interfaces:**
- Consumes: macro trong `styles/common.tex`.
- Produces: `\titlepage` hoàn chỉnh; tóm tắt; bảng từ viết tắt (dùng `longtable`).

- [ ] **Step 1: Viết nội dung frontmatter**

Nội dung bắt buộc:
- Bìa với placeholder: `[TÊN TRƯỜNG/ĐƠN VỊ]`, `[KHOA/BỘ MÔN]`, `[HỌ VÀ TÊN TÁC GIẢ]`, `[MSSV]`, `[GVHD/NGƯỜI HƯỚNG DẪN]`, `[NĂM]`, tên đề tài đầy đủ.
- **Lời cảm ơn** (1 đoạn).
- **Tóm tắt** tiếng Việt (~200 từ) + **Abstract** tiếng Anh (~150 từ): nêu bối cảnh Vietnam Post, mục tiêu, phương pháp RAG hybrid + fine-tuning, kết quả chính (Recall@5 sau rerank 50%, MRR 0.3149, Dense recall 100%; LLM-judge; đề xuất kiến trúc lai).
- **Danh mục từ viết tắt**: LLM, RAG, RAGAS, BM25, MRR, nDCG, PEFT, LoRA, QLoRA, SFT, DPO, RLHF, OCR, MCP, A2A, vLLM, FP8, Qdrant, Vietnam Post, EMS, CSKH, K8s, OTel.

- [ ] **Step 2: Biên dịch và kiểm tra**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS; bìa hiển thị, không lỗi font.

- [ ] **Step 3: Commit**

```bash
git add chapters/00-frontmatter.tex && git commit -q -m "docs: phần đầu báo cáo (bìa, tóm tắt, viết tắt)"
```

---

### Task 2: Chương 1 — Giới thiệu (≈6 trang)

**Files:**
- Modify: `chapters/01-gioi-thieu.tex`

**Interfaces:**
- Produces: `\label{ch:gioithieu}` cho tham chiếu chéo.

- [ ] **Step 1: Viết 6 mục theo spec**

1.1 Bối cảnh & lý do (chuyển đổi số bưu chính; khối lượng văn bản/biểu phí/quy trình lớn; nhu cầu tra cứu nhanh; hạn chế FAQ/tổng đài; dữ liệu nội bộ nhạy cảm → cần on-prem).
1.2 Mục tiêu nghiên cứu (4–5 gạch đầu dòng).
1.3 Đối tượng & phạm vi (corpus văn thư 17 tài liệu, corpus SPDV 119 tài liệu, corpus EMS ~30 tài liệu; nghiệp vụ tra cứu bưu gửi/bưu cục).
1.4 Phương pháp nghiên cứu (nghiên cứu tài liệu; thiết kế kiến trúc; triển khai multi-agent MCP/A2A; thực nghiệm RAG; đánh giá định lượng + định tính; thiết kế fine-tuning minh họa).
1.5 Đóng góp (kiến trúc lai; quy trình đánh giá; bằng chứng thực nghiệm tại Vietnam Post).
1.6 Cấu trúc báo cáo (liệt kê 6 chương).

- [ ] **Step 2: Biên dịch và kiểm tra**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS.

- [ ] **Step 3: Commit**

```bash
git add chapters/01-gioi-thieu.tex && git commit -q -m "docs: chương 1 - giới thiệu"
```

---

### Task 3: Chương 2 — Cơ sở lý thuyết (≈12 trang)

**Files:**
- Modify: `chapters/02-co-so-ly-thuyet.tex`
- Modify: `references.bib` (thêm các entry nền tảng)

**Interfaces:**
- Produces: `\label{ch:coso}`; định nghĩa thuật ngữ dùng lại ở chương 3–5.

- [ ] **Step 1: Viết mục 2.1–2.2 (chatbot; LLM/Transformer)**

Gồm: phân loại chatbot; kiến trúc Transformer (self-attention, encoder-decoder, causal LM); tokenization; pre-training; instruction tuning; decoding (temperature, top-p, top-k); prompting (zero/few-shot, CoT). Chèn Hình TikZ minh họa kiến trúc Transformer.

- [ ] **Step 2: Viết mục 2.3 (RAG)**

2.3.1–2.3.7 như spec. Chèn Hình TikZ: (a) RAG tổng quát; (b) pipeline indexing; (c) hybrid retrieval + rerank. Nêu công thức BM25 và cosine similarity.

- [ ] **Step 3: Viết mục 2.4 (Fine-tuning)**

Full FT vs PEFT; SFT/instruction tuning/RLHF/DPO; công thức LoRA ($W = W_0 + BA$, hạng $r$, $\alpha$); QLoRA (4-bit NF4 + double quant + paged optimizers). Chèn Hình TikZ minh họa LoRA.

- [ ] **Step 4: Viết mục 2.5–2.7**

2.5 So sánh RAG vs fine-tuning (bảng `booktabs`) + kiến trúc lai.
2.6 Thước đo: Recall@k, Precision@k, MRR, nDCG; RAGAS (Faithfulness, Answer Relevancy, Context Precision, Context Recall); LLM-as-judge; BLEU/ROUGE.
2.7 Nền tảng: vLLM + PagedAttention; FP8 quantization; MCP; A2A; OpenTelemetry/Langfuse/Grafana; MLflow Prompt Registry.

- [ ] **Step 5: Điền `references.bib`**

Thêm entry cho: Vaswani 2017 (Transformer), Lewis 2020 (RAG), Gao 2023 (RAG survey), Hu 2021 (LoRA), Dettmers 2023 (QLoRA), Robertson 2009 (BM25), Es 2023 (RAGAS), Karpukhin 2020 (DPR), Ouyang 2022 (InstructGPT), Rafailov 2023 (DPO), Kwon 2023 (vLLM/PagedAttention). Dùng `@article`/`@inproceedings` với đầy đủ title/author/year.

- [ ] **Step 6: Biên dịch và kiểm tra tham chiếu**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS, không còn "undefined citation"; TOC có chương 2.

- [ ] **Step 7: Commit**

```bash
git add chapters/02-co-so-ly-thuyet.tex references.bib && git commit -q -m "docs: chương 2 - cơ sở lý thuyết"
```

---

### Task 4: Chương 3 — Kiến trúc hệ thống đề xuất (≈12 trang)

**Files:**
- Modify: `chapters/03-kien-truc.tex`

**Interfaces:**
- Consumes: thuật ngữ chương 2; `\label{ch:coso}`.
- Produces: `\label{ch:kientruc}`; các sơ đồ kiến trúc được chương 4–5 tham chiếu.

- [ ] **Step 1: Viết mục 3.1–3.2**

Đặc thù bài toán Vietnam Post (văn bản hành chính, quyết định, biểu phí EMS, quy định hàng cấm, chính sách bồi thường; nghiệp vụ tra cứu bưu gửi, bưu cục). Yêu cầu chức năng (hỏi–đáp ngữ cảnh, tra cứu, định tuyến agent, tạo ticket) và phi chức năng (chính xác, độ trễ, bảo mật on-prem, khả mở rộng).

- [ ] **Step 2: Viết mục 3.3 (multi-agent) + Hình TikZ**

Mô tả host orchestrator + các specialist agent (RAG sản phẩm/dịch vụ, tra cứu bưu gửi, tra cứu bưu cục, chất lượng phát). Sơ đồ TikZ thể hiện host ↔ agents qua A2A, tools qua MCP, LLM/embedding/reranker qua vLLM.

- [ ] **Step 3: Viết mục 3.4 (pipeline RAG chi tiết) + Hình TikZ**

Indexing: tài liệu → OCR (olmocr-2-7b / lightonocr2-1b) → markdown → chunking (header-aware, table-safe, max_length≈1000) → embedding Qwen3 (4096-d) → Qdrant (dense COSINE + BM25 sparse). Retrieval: dense top-k 20 + BM25 top-k 10–20 → union → rerank toàn bộ ứng viên bằng Qwen3-Reranker → top-k 5–10 + ngưỡng điểm. Nêu rõ tham số thực tế.

- [ ] **Step 4: Viết mục 3.5 (pipeline fine-tuning đề xuất)**

Thu thập log hội thoại + dữ liệu tổng hợp → làm sạch/ẩn PII → định dạng instruction/chat → chia train/val → LoRA/QLoRA (r=16, alpha=32, target q/k/v/o/gate/up/down) → huấn luyện SFT → merge/đóng gói adapter → phục vụ bằng vLLM. Ghi rõ đây là thiết kế đề xuất.

- [ ] **Step 5: Viết mục 3.6 (kiến trúc lai) + Hình TikZ**

Router quyết định: câu hỏi tri thức nội bộ → RAG; câu hỏi phong cách/định dạng/nghiệp vụ lặp → model fine-tuned; kết hợp (fine-tuned sinh khung + RAG bổ sung bằng chứng). Tiêu chí định tuyến, fallback khi thiếu bằng chứng.

- [ ] **Step 6: Viết mục 3.7 (hạ tầng & bảo mật)**

K8s; vLLM; mô hình on-prem: `vietnampost-llm` (gateway), Qwen3.6-35B-A3B-FP8, Qwen3.8-27B-FP8, GPT-OSS-20B; embedding Qwen3-Embedding-8B; reranker Qwen3-Reranker-8B; OCR olmocr-2-7b. Không để dữ liệu ra ngoài; SSO/RBAC.

- [ ] **Step 7: Biên dịch và kiểm tra hình**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS; các hình TikZ hiển thị; danh mục hình có mục.

- [ ] **Step 8: Commit**

```bash
git add chapters/03-kien-truc.tex && git commit -q -m "docs: chương 3 - kiến trúc đề xuất"
```

---

### Task 5: Chương 4 — Triển khai thực nghiệm (≈14 trang)

**Files:**
- Modify: `chapters/04-trien-khai.tex`

**Interfaces:**
- Consumes: kiến trúc chương 3.
- Produces: `\label{ch:trienkhai}`; bảng tham số được chương 5 tham chiếu.

- [ ] **Step 1: Viết mục 4.1–4.2 (môi trường; dữ liệu)**

Môi trường: Python, Docker Compose, K8s, Qdrant v1.16.1, vLLM, MLflow, Langfuse, Grafana, Airflow. Dữ liệu: corpus văn thư 17 tài liệu (`agent.template.rag/rag-data/unique_documents/`), corpus SPDV 119 tài liệu (114 txt, 3 pdf, 2 docx) (`chatbottct.deployment/tracuu-spdv-rag-data/`), corpus EMS ~30 docx. Bộ đánh giá EMS 99 câu hỏi/đáp (`tool-test-ems-rag/test ems.xlsx`, Sheet2). Bảng thống kê dữ liệu.

- [ ] **Step 2: Viết mục 4.3 (cài đặt RAG) + bảng tham số**

Chunking, embedding Qwen3-Embedding-8B, hybrid BM25+dense, reranker Qwen3-Reranker-8B, `TOP_K_DENSE=20`, `TOP_K_BM25=10–20`, `TOP_K_RERANKER=5–10`, ngưỡng điểm, prompt hệ thống (trích `system_prompt.md`). Bảng tham số `booktabs`.

- [ ] **Step 3: Viết mục 4.4 (cài đặt fine-tuning đề xuất)**

Quy trình tạo dataset SFT, cấu hình LoRA (bảng: r, alpha, dropout, target modules, lr, epochs), phần cứng giả định, tiêu chí dừng. Ghi rõ là đề xuất.

- [ ] **Step 4: Viết mục 4.5 (multi-agent + MCP/A2A) + 4.6 (observability)**

Mô tả triển khai host + specialist agents; rule định tuyến (ví dụ 26 rule CSKH EMS, taxonomy ticket); MCP tools; A2A. Observability: OTel → collector → Grafana Tempo/Langfuse; MLflow prompt `@production` (`tct-host-agent`, `tct-rag-agent`, `mipo-doc-rag-agent`...).

- [ ] **Step 5: Viết mục 4.7 (khó khăn & giải pháp)**

Dữ liệu tiếng Việt (BM25 `disable_stemmer=True`, stopwords cleared); chunking bảng biểu; reranker; chi phí GPU; prompt management.

- [ ] **Step 6: Biên dịch và kiểm tra**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add chapters/04-trien-khai.tex && git commit -q -m "docs: chương 4 - triển khai thực nghiệm"
```

---

### Task 6: Chương 5 — Kết quả và đánh giá (≈12 trang)

**Files:**
- Modify: `chapters/05-ket-qua.tex`

**Interfaces:**
- Consumes: `\label{ch:trienkhai}`; tham số chương 4.
- Produces: `\label{ch:ketqua}`; các bảng/hình kết quả.

- [ ] **Step 1: Viết mục 5.1–5.2 (phương pháp; kết quả retrieval)**

Phương pháp: harness `eval/retrieval/eval_retrieval.py`, LLM-as-judge `eval/agent/eval_agent.py`, RAGAS-4 `tool-test-ems-rag/evaluate_ragas4.py`. Kết quả retrieval (nguồn `SAMPLE_eval_retrieval_input_results.xlsx`): Recall@20 BM25 = 3/6 (50%), Dense = 6/6 (100%), trước rerank = 6/6 (100%), Recall@5 sau rerank = 3/6 (50%), MRR = 0.3149; lỗi RERANK 3/6 (50%), lỗi RECALL 0/6 (0%). Bảng + biểu đồ pgfplots. Phân tích vì sao reranker làm giảm recall.

- [ ] **Step 2: Viết mục 5.3 (generation/agent)**

LLM-as-judge 0–10 (mẫu 3 câu, mean/median 10.00 — ghi rõ cỡ mẫu nhỏ). RAGAS-4 trên 99 câu: mô tả harness đã thu 99/99 phản hồi trong 249.4 s (10 workers, `results/ragas4_collect.md`); trình bày **kết quả minh họa** cho Faithfulness/Answer Relevancy/Context Precision/Context Recall và ghi rõ là minh họa do chưa tổng hợp điểm thật.

- [ ] **Step 3: Viết mục 5.4 (so sánh RAG vs fine-tuning vs lai)**

Bảng so sánh theo tiêu chí: độ chính xác tri thức động, chi phí cập nhật, chi phí huấn luyện, độ trễ, khả năng trích nguồn, bảo mật. Phân tích định tính + dựa số liệu RAG thật; phần fine-tuning ghi rõ là đánh giá đề xuất/minh họa.

- [ ] **Step 4: Viết mục 5.5 (hiệu năng hệ thống)**

Bảng benchmark vLLM (`dockers/test_vllm_speed/benchmark_results_*.json`): Qwen3.8-27B-FP8 single ~180 tok/s, batch ~3,137 tok/s @1024 req; Qwen3.6 (k8s) 232.9/3,976; Qwen3.6 7g 236.4/7,337; GPT-OSS 242.8/4,484; v.v. Biểu đồ cột pgfplots. Thảo luận độ trễ end-to-end.

- [ ] **Step 5: Viết mục 5.6–5.8 (case study; hiệu quả kinh tế; thảo luận)**

Case study: ví dụ hội thoại hỏi biểu phí EMS / tra cứu bưu gửi / tra cứu bưu cục. Hiệu quả kinh tế (nguồn `sk/02-dkcl.../sections/`): tiết kiệm 30–60 phút/ngày; ~13.000 người dùng × ≥10 phút/ngày → ~2.167 giờ công/ngày ≈ 32,5 tỷ VNĐ/năm (56.818 VNĐ/giờ, 22 ngày/tháng, 264 ngày/năm). Thảo luận ưu điểm/hạn chế/tính khái quát.

- [ ] **Step 6: Biên dịch và kiểm tra**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS; biểu đồ hiển thị.

- [ ] **Step 7: Commit**

```bash
git add chapters/05-ket-qua.tex && git commit -q -m "docs: chương 5 - kết quả và đánh giá"
```

---

### Task 7: Chương 6 — Kết luận và hướng phát triển (≈4 trang)

**Files:**
- Modify: `chapters/06-ket-luan.tex`

- [ ] **Step 1: Viết nội dung**

6.1 Kết luận (tóm tắt đóng góp + kết quả định lượng chính).
6.2 Hạn chế (corpus/cỡ mẫu nhỏ; reranker giảm recall; fine-tuning chưa huấn luyện thật; chưa đánh giá người dùng diện rộng).
6.3 Hướng phát triển (GraphRAG, agentic RAG, continual learning, fine-tuning định kỳ, mở rộng đa ngôn ngữ, đánh giá A/B với người dùng).

- [ ] **Step 2: Biên dịch, commit**

Run: `/tmp/opencode/tectonic -X compile main.tex` → PASS.
```bash
git add chapters/06-ket-luan.tex && git commit -q -m "docs: chương 6 - kết luận và hướng phát triển"
```

---

### Task 8: Tài liệu tham khảo và Phụ lục (≈5 trang)

**Files:**
- Modify: `references.bib`
- Modify: `chapters/99-phu-luc.tex`

- [ ] **Step 1: Bổ sung tài liệu tham khảo**

Đảm bảo mọi `\cite` trong chương 2–6 có entry; thêm tài liệu kỹ thuật (Qdrant docs, vLLM, RAGAS, MCP spec, A2A, MLflow, Langfuse, OpenTelemetry) dạng `@misc` với `howpublished`/`url`.

- [ ] **Step 2: Viết phụ lục**

Phụ lục A: prompt hệ thống mẫu (host + RAG agent). Phụ lục B: bảng cấu hình tham số RAG/agent. Phụ lục C: danh sách mô hình và endpoint. Phụ lục D (tùy chọn): mẫu hội thoại.

- [ ] **Step 3: Biên dịch và kiểm tra bibliography**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS; "Tài liệu tham khảo" hiển thị; không còn undefined citation.

- [ ] **Step 4: Commit**

```bash
git add references.bib chapters/99-phu-luc.tex && git commit -q -m "docs: tài liệu tham khảo và phụ lục"
```

---

### Task 9: Kiểm tra cuối, cân chỉnh độ dài, xuất PDF

**Files:**
- Modify: các file `.tex` nếu cần
- Create: `final/bao-cao-rag-finetuning-vnpost.pdf`

- [ ] **Step 1: Biên dịch bản cuối**

Run: `/tmp/opencode/tectonic -X compile main.tex`
Expected: PASS, `main.pdf` tạo thành công.

- [ ] **Step 2: Đếm số trang**

Run: `pdfinfo main.pdf | grep -i pages` (hoặc `python3 -c "import re;print(open('main.pdf','rb').read())"`).
Expected: 55–68 trang. Nếu thiếu: mở rộng thảo luận chương 5, thêm ví dụ/case study, phụ lục. Nếu thừa: rút gọn.

- [ ] **Step 3: Kiểm tra tham chiếu và mục lục**

Run: `/tmp/opencode/tectonic -X compile main.tex` (vòng 2) và render vài trang quan trọng sang PNG để xem:
`pdftoppm -png -r 90 -f 1 -l 3 main.pdf check && pdftoppm -png -r 90 -f <trang chương 5> main.pdf kq`
Expected: không có `??`; tiếng Việt hiển thị đúng; hình/bảng đánh số đúng.

- [ ] **Step 4: Xuất bản PDF cuối**

```bash
mkdir -p final && cp main.pdf final/bao-cao-rag-finetuning-vnpost.pdf
```

- [ ] **Step 5: Commit**

```bash
git add -A && git commit -q -m "docs: hoàn thiện báo cáo (~60 trang) và xuất PDF"
```

---

## Self-Review (đã chạy)

- **Spec coverage:** Phần đầu (Task 1), C1 (Task 2), C2 (Task 3), C3 (Task 4), C4 (Task 5), C5 (Task 6), C6 (Task 7), TLTK+Phụ lục (Task 8), độ dài/PDF (Task 9). Đủ.
- **Placeholder scan:** Không có TODO/TBD ngoài placeholder bìa (cố ý theo spec).
- **Type consistency:** Tên file/macro thống nhất (`styles/common.tex`, `\vnpost`, `\code`). Nhãn chương: `ch:gioithieu`, `ch:coso`, `ch:kientruc`, `ch:trienkhai`, `ch:ketqua`.
- **Review Focus:** đã gắn kiểm tra font (Task 0/1), bibliography (Task 3/8), tham chiếu (Task 9), độ dài (Task 9), trung thực số liệu (Global Constraints + Task 5/6).
