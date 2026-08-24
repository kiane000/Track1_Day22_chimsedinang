# Day 22 — Responsible AI in Production — Kế hoạch làm bài (150 phút)

Tài liệu này là **playbook dùng chung**, không gắn với một ngành hay một học viên cụ
thể — bất kỳ ai làm lab này đều có thể copy file này vào `submissions/<MÃ-HỌC-VIÊN>/`
(hoặc giữ trong `docs/`) làm checklist theo dõi tiến độ. Nó tóm tắt lại
[README.md](../README.md), [docs/student-guide.md](student-guide.md) và
[docs/rubric.md](rubric.md) thành một luồng hành động theo thời gian.

Vai trò: bạn là **Product Manager/Product Owner**. Không train model, không viết
app. Câu hỏi cuối: với evidence hiện có, sản phẩm nên `go`, `conditional-go`,
`no-go` hay `research-only`?

## Bước 0 — Setup (trước 0:00)

```bash
mkdir -p submissions/<MÃ-HỌC-VIÊN>
cp templates/* submissions/<MÃ-HỌC-VIÊN>/
python3 scripts/validate-lab.py submissions/<MÃ-HỌC-VIÊN>   # PHẢI fail — đây là baseline đúng
```

Windows PowerShell: dùng `python` thay `python3` nếu máy chưa alias `python3`.

Chọn đúng 1 ngành (`industry` trong `lab.config.json`):

| Ngành | slug |
|---|---|
| HR / tuyển dụng | `hr-recruitment` |
| Giáo dục / AI tutor | `education-ai-tutor` |
| Y tế / health assistant | `healthcare-assistant` |
| Mobility / autonomous driving | `mobility-autonomous-driving` |
| Media/news/social/political | `media-news-social-political-assistant` |
| Content creator | `content-creator` (nhóm chọn chung) |

## Bước 1 (0:00–0:20) — Product Context & AI System Boundary

**File:** `submission.json` → `product_context` + `system_profile`

- Bắt đầu từ vấn đề người dùng/business, không bắt đầu từ "cần dùng AI".
- Chốt: `problem_statement`, `target_users`, `value_hypothesis`, `in_scope`/`non_goals`,
  `user_journey_moment` (AI xuất hiện ở bước nào), `automation_level`,
  `fallback_experience` (luồng vận hành thật, không phải "thử lại sau"),
  `decision_reversibility`, `target_launch_or_review_date`.
- `system_boundary` phải vượt ra ngoài model: `model` + `application` + `people` +
  `vendors` (ghi rõ "none" nếu không có, không để rỗng) + `upstream_inputs` +
  `downstream_decisions`.
- Chỉ dùng dữ liệu giả lập/công khai — không PII, không hồ sơ thật.

✅ Xong khi: mọi field mô tả cụ thể, không còn `<FILL>`/`<CHOOSE>`.

## Bước 2 (0:20–0:35) — Industry Risk Snapshot

**File:** `submission.json` → `industry_risk_snapshot`

- 5 score **integer 1–5**: `harm_severity`, `high_stakes`, `sensitive_data`,
  `affected_scale`, `human_review_need`.
- Anchor: 1 = nhỏ/dễ đảo ngược · 3 = đáng kể hoặc liên quan nhóm dễ tổn thương,
  sửa được nhưng tốn chi phí · 5 = ảnh hưởng quyền/sức khỏe/sinh mạng/sinh kế
  hoặc quy mô lớn và khó đảo ngược.
- `rationale` giải thích từng score gắn với product context cụ thể — **không**
  dùng snapshot này để suy ra legal classification.

## Bước 3 (0:35–1:10) — Case research & Evidence Lineage

**File:** `sources.csv` + `case-studies.csv`
(checklist đọc nguồn: [docs/research-and-evidence-guide.md](research-and-evidence-guide.md))

- Research 2–3 case **có thật** (không hư cấu). ≥2 source, ≥1 source
  `primary-official` hoặc `primary-peer-reviewed`.
- `sources.csv`: `source_id` dạng `SRC-01`; URL HTTPS thật (không placeholder);
  `authority_level` đúng enum; `supports_claim` hẹp, chính xác; `limitations`
  trung thực về giới hạn của nguồn.
- `case-studies.csv`: `CASE-01..03`; `industry` phải khớp `submission.json`.
  Tách 3 lớp không trộn: `verified_facts` (chỉ điều nguồn xác nhận) /
  `reported_harm` (ghi rõ đã xảy ra / allegation / foreseeable hazard) /
  `limitations`. Không biến allegation thành finding.
- Không cite chatbot. Với luật/quy định, dùng nguồn chính thức; snapshot repo
  là **2026-08-24** — luật/guidance có thể đã đổi, phải kiểm tra lại nguồn hiện
  hành khi dùng tài liệu sau ngày này.

## Bước 4 (1:10–1:40) — Harm Map

**File:** `harm-map.csv`

- Mỗi case ≥1 dòng harm. Toàn bộ harm map phải có **≥1 dòng
  `stakeholder_type = affected-non-customer`**.
- Mỗi dòng: `case_id` → `high_risk_moment` → `stakeholder`/`stakeholder_type` →
  `failure_mode`/`failure_layer`/`harm_lens` (đúng enum) → `harm_description`
  (hậu quả với stakeholder, không chỉ "model sai") → 4 score 1–5 (`severity`,
  `likelihood`, `scale`, `frequency`) → `evidence_source_ids` (dùng `;`) →
  `existing_controls` tách khỏi `proposed_controls` → `human_oversight` (ai
  review, khi nào, có quyền override/pause/escalate không) →
  `residual_severity_1_5`/`residual_likelihood_1_5` →
  `owner`/`monitoring_metric`/`trigger_threshold`/`response_action`.
- Xét đủ 6 lớp failure layer (UX, data, people, vendor/tooling, governance) —
  không đổ hết cho model.

## Bước 5 (1:40–2:10) — Product Backlog, Legal Gap, KPI/KRI

**File:** `compliance-gap-analysis.csv` + `submission.json`
(`legal_classification`, `product_metrics`)

- Mỗi risk/control quan trọng → 1 dòng `GAP-XX`: mô tả outcome/rule (không phải
  task kỹ thuật); `acceptance_criteria` phải có đủ **Given/When/Then**;
  `priority`; `release_blocking` (yes/no); `owner`; `target_milestone`;
  `deadline_or_trigger`; `status`; `evidence_needed`; `source_ids` hợp lệ.
- `legal_classification`: đánh giá Việt Nam và EU **độc lập** (không copy
  status qua lại, không suy từ risk snapshot). Thiếu căn cứ → dùng
  `uncertain-requires-legal-review`. `other_markets` cần mô tả có nghĩa.
- `product_metrics`: 4 string cụ thể — `success_kpi`, `risk_kri`,
  `guardrail_metric`, `review_cadence`. Tự hỏi: "KPI tăng mà harm cũng tăng thì
  sao?" — KRI/guardrail phải bắt được tình huống đó.

## Bước 6 — Release Decision

**File:** `submission.json` → `release_decision`

- 3 role (không phải tên người): `accountable_product_owner`,
  `risk_acceptance_owner`, `independent_reviewer`.
- `decision` ∈ {go, conditional-go, no-go, research-only}. **Không chọn `go`**
  nếu còn gap `release_blocking=yes` mà `status ≠ done` — mọi GAP ID đó phải
  có mặt trong `release_blockers`.
- `release_blockers`/`release_conditions` là array không rỗng.
- `residual_risk_rationale` + `next_review_trigger` phải cụ thể.

## Bước 7 (2:10–2:25) — Cross-industry Synthesis (làm cùng nhóm)

**File:** `group-synthesis.csv`

Đúng **1 dòng duy nhất** cho cả nhóm. So sánh product context, reversibility,
affected stakeholder, evidence thật giữa các hồ sơ trong nhóm — chốt
`highest_stakes_industry`, `recurring_harms`, `common_failure_layers`,
`human_in_loop_decisions`, `strongest_guardrails`, một `cross_industry_pattern`
có tác động thật đến roadmap/gate, và `evidence_source_ids` (phải tồn tại
trong `sources.csv` của người validate).

## Bước 8 (2:25–2:30) — Validate, tự review, nộp bài

```bash
python3 scripts/validate-lab.py submissions/<MÃ-HỌC-VIÊN>
python3 -m unittest discover -s tests -v
git status
git diff --check
```

Checklist trước khi commit:

- [ ] Đúng 6 artifact, không thêm executive readout làm file thứ 7.
- [ ] 2–3 case, ≥2 sources, ≥1 nguồn primary-official/peer-reviewed.
- [ ] Mọi ID (SRC/CASE/GAP) đúng format, tồn tại; multi-ID dùng `;` không `,`.
- [ ] Mỗi case có harm row; có ≥1 `affected-non-customer`.
- [ ] Mọi score integer 1–5; mọi acceptance criteria có Given/When/Then.
- [ ] Mọi blocker mở có mặt trong `release_blockers`; không chọn `go` nếu còn
      blocker.
- [ ] `group-synthesis.csv` đúng 1 dòng.
- [ ] Không còn `<FILL>`/`<CHOOSE>`/TODO/TBD/URL placeholder.
- [ ] Không có PII/CV thật/hồ sơ bệnh án/API key/secret.
- [ ] validator PASS, unit tests pass, `git diff --check` sạch.

Tạo repo riêng `Day22-ResponsibleAI-<MãHV>-<HọVàTên>`, commit, push.

## Gỡ lỗi nhanh

Sửa theo đúng thứ tự **source → case → harm → gap → release** — một source ID
sai có thể làm nhiều chỗ khác cùng báo lỗi. Bảng lỗi thường gặp nằm trong
[docs/student-guide.md](student-guide.md) (mục "Gỡ lỗi validator theo thứ tự
traceability").
