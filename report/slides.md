---
marp: true
theme: default
class: lead
paginate: true
backgroundColor: #ffffff
color: #333333
style: |
  section {
    font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
    padding: 40px;
  }
  h1 {
    color: #0f172a;
  }
  h2 {
    color: #1e293b;
    border-bottom: 2px solid #3b82f6;
  }
  footer {
    font-size: 0.5em;
    color: #64748b;
  }
---

# Dự Án Nhóm: AI Marketing Agent
## Chuyển đổi từ Chatbot thông thường sang ReAct Agent kiểm soát lỗi

**Thành viên thực hiện:**
- Phạm Hoàng Anh (2A202600631)
- Nguyễn Văn Minh (2A202600556)

---

# 1. Ý Tưởng Dự Án: AI Marketing Agent

* **Vấn đề của Chatbot thường**: Chỉ biết trả lời lý thuyết, dễ nói dối (hallucinate) đã đăng bài, đã lên lịch nhưng thực tế không làm gì.
* **Mục tiêu của nhóm**: Tạo ra một Agent thực sự có tay chân (các hàm Python) để:
  1. Tự phân tích sản phẩm.
  2. Tìm kiếm nhóm Facebook phù hợp nhất.
  3. Viết bài tiếp thị riêng cho nhóm đó.
  4. Lên lịch đăng bài thật thông qua code Python.

---

# 2. Quy Trình Hoạt Động (ReAct Loop)

Agent hoạt động nhịp nhàng theo 3 bước lặp đi lặp lại:

1. **Nghĩ (Thought)**: Xem yêu cầu của user và lịch sử nháp để phân tích bước tiếp theo cần làm gì.
2. **Gọi Tool (Action)**: Ra lệnh cho Python chạy một công cụ cụ thể (ví dụ: `schedule_post`).
3. **Nhận kết quả thật (Observation)**: Python trả dữ liệu thực tế về cho Agent để suy nghĩ tiếp bước sau.

*Ngăn chặn Agent tự tiện bịa kết quả bằng kỹ thuật **Stop Sequences**.*

---

# 3. Danh Sách 5 Công Cụ (Tools) Đã Xây Dựng

Chúng tôi lập trình 5 công cụ Python thực tế:

* **analyze_product**: Phân tích sản phẩm ra từ khóa và khách hàng.
* **discover_groups**: Tìm nhóm Facebook phù hợp nhất theo từ khóa.
* **generate_content**: Tự viết nội dung bài đăng quảng cáo phù hợp với nhóm.
* **schedule_post**: Gọi code Python lên lịch đăng bài thật.
* **get_analytics**: Lấy chỉ số tương tác thực tế (views, clicks, CTR).

---

# 4. Phân Tích Lỗi Thực Tế Trên Bản Agent v1

* **Hiện tượng lỗi**: Agent v1 chạy tìm sản phẩm và group xong thì **tự ý bỏ qua tool viết bài và đặt lịch**. Nó tự viết bài và tự kết luận Final Answer trong một lượt suy nghĩ thô của nó.
* **Tại sao lỗi**:
  - Prompt v1 viết lỏng lẻo, không bắt buộc AI dùng tool.
  - Luồng code Python cũ check từ khóa kết thúc (`Final Answer`) trước khi check lệnh gọi tool (`Action`), làm ngắt vòng lặp ReAct sớm.

---

# 5. Bản Vá Tối Ưu Lỗi Trên Bản Agent v2

Chúng tôi đã bắt tay sửa lỗi hoàn chỉnh:

* **Vá Prompt v2**: Viết thêm quy tắc nghiêm khắc cấm AI tự ý làm thay tool.
* **Vá logic Python**: Đảo thứ tự check, **bắt buộc chạy Action trước**, chỉ kết thúc khi không còn Action nào.
* **Áp dụng Stop Sequences**: Cấu hình API dừng tạo chữ ngay khi viết xong lệnh `Action:`, ép con AI phải nhường sân cho code Python chạy thật.

---

# 6. So Sánh Hiệu Năng Đạt Được

Bảng đo đạc thực tế từ script `parse_logs.py`:

| Chỉ số đo lường | Chatbot Thường | Agent v1 (Lười biếng) | Agent v2 (Chuẩn xác) |
| :--- | :--- | :--- | :--- |
| **Độ trễ trung bình** | ~1.5 giây | ~3.2 giây | ~12.4 giây (Full 5 steps) |
| **Tokens tiêu thụ** | ~1,100 tokens | ~2,500 tokens | ~6,200 tokens |
| **Số Tool gọi thật** | 0 tool | 2 tools (Bỏ qua 2) | **5 tools tuần tự** |
| **Tỷ lệ thành công** | 0% (Hallucinate) | 0% (Không chạy tool) | **100% (Hoàn thành thật)** |

---

# 7. Hướng Phát Triển Cho Bản Thực Tế (Go-to-Market)

* **Bảo mật**: Dùng Pydantic kiểm tra tham số bóc tách từ LLM để tránh bị hack qua câu lệnh.
* **Mở rộng**: Kết nối API Facebook/LinkedIn thật, dùng Celery chạy ngầm các tác vụ cào dữ liệu và đăng bài để không treo Agent.
* **Tối ưu**: Dùng Vector DB (ChromaDB) lưu nhúng mô tả tool để chỉ lấy ra 5 tool phù hợp nhất cho prompt, giúp tiết kiệm tối đa tiền API.

---

# CẢM ƠN THẦY CÔ VÀ CÁC BẠN ĐÃ LẮNG NGHE!
## Q&A - Team AI Marketing Agent
