# Day 13 Observability Lab Report

> **Instruction**: Fill in all sections below. This report is designed to be parsed by an automated grading assistant. Ensure all tags (e.g., `[GROUP_NAME]`) are preserved.

## 1. Report Metadata
- Họ và tên: Hoàng Trung Quân - 2A202600720
- Role: All roles (Logging, PII, Tracing, SLO, Alerts, Dashboard, Incident Response)
---

## 2. Performance (Auto-Verified)
- [VALIDATE_LOGS_FINAL_SCORE]: 100/100
- [TOTAL_TRACES_COUNT]: > 20 traces
- [PII_LEAKS_FOUND]: 0 (PII data scrubbed successfully via structlog processors)

---

## 3. Technical Evidence (Group)

### 3.1 Logging & Tracing
- [EVIDENCE_CORRELATION_ID_SCREENSHOT]: Xem file `data/logs.jsonl` (các trường `correlation_id` đã được gắn thành công).
- [EVIDENCE_PII_REDACTION_SCREENSHOT]: Xem file `data/logs.jsonl` (thông tin nhạy cảm đã bị thay thành `[REDACTED_EMAIL]`, `[REDACTED_PHONE_VN]`).
- [EVIDENCE_TRACE_WATERFALL_SCREENSHOT]: Tham khảo thư mục `screenshots/` (Langfuse Waterfall đã hoạt động và hiển thị thời gian chính xác).
- [TRACE_WATERFALL_EXPLANATION]: Trong quá trình kiểm thử tải sự cố (Load Test with Incident), span `retrieve` trong Trace Waterfall đã giúp chỉ đích danh nguyên nhân gây chậm hệ thống (ngốn tới 2.5s) khi chạy script Chaos Engineering `inject_incident.py`.

### 3.2 Dashboard & SLOs
- [DASHBOARD_6_PANELS_SCREENSHOT]: 
  - ![Cost Dashboard](../screenshots/LLMs%20cost.png)
  - ![Latency Dashboard](../screenshots/Latency%20dashboard.png)
  - ![Usage Management](../screenshots/Usage%20management.png)
- [SLO_TABLE]:
| SLI | Target | Window | Current Value |
|---|---:|---|---:|
| Latency P95 | < 3000ms | 28d | ~150ms (Normal) / ~2660ms (Incident) |
| Error Rate | < 2% | 28d | 0% |
| Cost Budget | < $2.5/day | 1d | Nằm trong ngưỡng an toàn (See Dashboard) |

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: Xem `config/alert_rules.yaml`
- [SAMPLE_RUNBOOK_LINK]: [docs/alerts.md#L1-L20]

---

## 4. Incident Response
- [SCENARIO_NAME]: `rag_slow`
- [SYMPTOMS_OBSERVED]: Hệ thống có độ trễ tăng vọt từ mức bình thường ~150ms lên hơn 2660ms (2.6 giây) khi kiểm thử tải bằng `load_test.py`.
- [ROOT_CAUSE_PROVED_BY]: Dữ liệu Trace trên giao diện Langfuse (Waterfall) chỉ ra hàm `retrieve` trong `app/agent.py` bị nghẽn và ngốn 2.5 giây. Log Terminal cũng in ra `latency_ms` = 2666ms.
- [FIX_ACTION]: Chạy lệnh `python scripts/inject_incident.py --scenario rag_slow --disable` để vô hiệu hóa sự cố do Chaos Engineering gây ra, đưa server về trạng thái phản hồi 150ms.
- [PREVENTIVE_MEASURE]: Cấu hình luật cảnh báo Symptom-based `latency_p95_ms > 5000 for 30m` trong `config/alert_rules.yaml` để đảm bảo On-call Engineer nhận được tin nhắn gọi báo động nếu hệ thống duy trì tình trạng tải chậm trong 30 phút, chống lại hiệu ứng Alert Fatigue (cảnh báo giả).

---

## 5. Individual Contributions & Evidence

### Hoàng Trung Quân
- [TASKS_COMPLETED]: 
  - Đạt điểm 100/100 trên hệ thống Validate Log tự động.
  - Thiết lập Correlation ID để xâu chuỗi Request, Enrich logs với các thông tin Contextvars.
  - Cấu hình Processor để Sanitize PII Data (bảo vệ quyền riêng tư).
  - Khắc phục lỗi tương thích phiên bản của Langfuse (downgrade v3 -> v2) để tích hợp thành công `@observe` Tracing.
  - Cấu hình thông số `cost_details` và `model` để theo dõi ngân sách (Cost).
  - Vận hành giả lập sự cố (Chaos Engineering) bằng `inject_incident.py`.
  - Hoàn thiện toàn bộ các nhiệm vụ tùy chọn (Bonus Items) nâng cao hệ thống.
  - [EVIDENCE_LINK]: Xem toàn bộ file `data/logs.jsonl` và `data/audit.jsonl` cùng cấu trúc dự án.
  - Xem chi tiết tại `docs/session_learning_checklist.md` 
---

## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: Đã thêm cấu hình `cost_details` và truyền tên `model` vào `langfuse_context` để hệ thống tự động tính toán chi phí chính xác trên Langfuse Dashboard (khắc phục lỗi tính chi phí bằng $0.00).
- [BONUS_AUDIT_LOGS]: Đã lập trình cấu trúc chia tách Log Kiểm toán (Audit Logs) độc lập. Mở rộng `JsonlFileProcessor` trong `logging_config.py` để tự động lọc và ghi các log có gắn cờ `audit=True` (như hành động bật/tắt sự cố) sang một file lưu trữ bảo mật riêng mang tên `data/audit.jsonl`.
- [BONUS_CUSTOM_METRIC]: Đã tự định nghĩa thêm một chỉ số giám sát đặc thù (Custom Metric) mang tên `slow_requests` trong `app/metrics.py`. Hệ thống tự động đếm số lượng các request bị phản hồi chậm hơn 1000ms và hiển thị tại endpoint `/metrics`.
