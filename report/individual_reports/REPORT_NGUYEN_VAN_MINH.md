# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Nguyễn Văn Minh
- **Student ID**: 2A202600556
- **Date**: 2026-06-01

---

## I. Technical Contribution (15 Points)

Trong bài Lab này, tôi chịu trách nhiệm chính về mặt **Hiện thực hóa bộ não Agent (Core Agent Logic & Prompt Engineering)**:

1. **Hiện thực hóa Vòng lặp ReAct (`run` method):**
   - Lập trình vòng lặp chính của ReAct trong file [agent.py](file:///Users/a/Documents/research/ai/task_ai/Day3-2A202600631/src/agent/agent.py) để lặp đi lặp lại chu trình sinh suy nghĩ (Thought) -> Thực thi hành động (Action) -> Nhận phản hồi thực tế (Observation).
   - Thiết lập cơ chế ghi nháp liên tục (`scratchpad`) để tích lũy ngữ cảnh suy luận qua từng bước lặp, giúp LLM kiểm soát tốt tiến trình chạy.
   - Viết các biểu thức chính quy (Regex) và logic bóc tách động để lấy ra tên công cụ nghiệp vụ và các tham số truyền vào từ kết quả dạng text của LLM, hỗ trợ phân tích cả định dạng JSON và Keyword Arguments.

2. **Kỹ nghệ Gợi ý & Tối ưu hóa System Prompt (Prompt Engineering):**
   - Xây dựng hàm tạo prompt động `get_system_prompt` nhằm định hình vai trò của Agent là một chuyên gia marketing thông minh.
   - Trực tiếp nâng cấp prompt từ phiên bản v1 lên phiên bản **Prompt v2 (Strict ReAct Prompt)**, bổ sung điều khoản `QUY TẮC BẮT BUỘC (CRITICAL RULES)` cưỡng chế gọi tool cho từng bước nghiệp vụ, loại bỏ lỗi bỏ qua tool (Tool Bypass) khi phát hiện qua logs.

---

## II. Debugging Case Study (10 Points)

### 1. Mô tả lỗi phát hiện (Problem Description)
Trong lượt chạy đầu tiên của Agent v1, khi LLM sinh ra bài viết marketing và lên lịch đăng bài, nó đã tự ý bỏ qua tool `generate_content` và tool `schedule_post`. Nó tự viết bài đăng và tự gõ dòng thông báo xác nhận lịch đăng trong câu trả lời cuối cùng (`Final Answer`) mà không hề thực hiện bất kỳ lệnh gọi Tool vật lý nào (Tool Bypass / Lazy Agent).

### 2. Log minh chứng (Log Source)
Trích xuất từ file log thực tế [2026-06-01.log](file:///Users/a/Documents/research/ai/task_ai/Day3-2A202600631/logs/2026-06-01.log):
```json
{"timestamp": "2026-06-01T08:29:50.627537", "event": "LLM_CALL_START", "data": {"step": 2}}
{"timestamp": "2026-06-01T08:29:59.814849", "event": "LLM_CALL_END", "data": {"step": 2, ...}}
{"timestamp": "2026-06-01T08:29:59.816469", "event": "AGENT_END", "data": {"steps": 2, "success": true}}
```
*Nhận xét:* Agent kết thúc sớm ở Step 2 chỉ với 2 tool được gọi (`analyze_product` và `discover_groups`), bỏ qua hoàn toàn các bước sinh nội dung và lên lịch vật lý.

### 3. Chẩn đoán nguyên nhân (Diagnosis)
Do prompt v1 chỉ hướng dẫn định dạng ReAct chung chung, chưa nhấn mạnh việc **BẮT BUỘC** gọi tool nếu tool đó có tồn tại cho tác vụ. LLM vốn là mô hình sinh ngôn ngữ xuất sắc nên nó ưu tiên việc tự viết văn bản bài đăng hơn là gửi yêu cầu qua một Tool Python trung gian.

### 4. Giải pháp khắc phục (Solution)
Tôi đã viết lại phần System Prompt nâng lên bản **Prompt v2**. Tôi bổ sung phần cưỡng chế gọi Tool cực kỳ nghiêm ngặt:
- *"BẮT BUỘC SỬ DỤNG TOOL: Nếu có công cụ hỗ trợ cho một bước công việc nào đó, bạn BẮT BUỘC phải gọi công cụ đó thông qua Action. Tuyệt đối KHÔNG tự nghĩ ra, KHÔNG tự giả lập..."*
- *"Để tạo nội dung bài viết marketing -> Bắt buộc gọi tool generate_content."*
- *"Để lên lịch đăng bài -> Bắt buộc gọi tool schedule_post."*

Sau khi nâng cấp prompt lên v2, Agent đã thực thi kỷ luật tuyệt đối, gọi đầy đủ cả 4 tool một cách tuần tự.

---

## III. Personal Insights: Chatbot vs ReAct (10 Points)

1. **Reasoning (Khả năng suy luận):**
   Vòng lặp ReAct tạo ra cơ hội để LLM tự sửa sai. Nhờ có bước `Thought` xen kẽ trước mỗi hành động, LLM có thể lập luận một cách tĩnh lặng về kết quả của bước trước rồi mới quyết định bước tiếp theo, thay vì vội vã đưa ra câu trả lời toàn cục thiếu chính xác.

2. **Reliability (Độ tin cậy):**
   Mặc dù ReAct Agent rất mạnh mẽ trong việc giải quyết vấn đề đa bước, nó phụ thuộc cực lớn vào độ chuẩn xác của bộ Parser và tính chặt chẽ của Prompt. Nếu Prompt viết lỏng lẻo, Agent sẽ tự động tìm đường tắt đi qua tool (bypass). Do đó, prompt kỹ thuật của Agent phải mang tính chất như một bản hợp đồng pháp lý chặt chẽ hơn là một đoạn hội thoại gợi ý thông thường.

3. **Observation (Phản hồi môi trường):**
   Dữ liệu Observation phản hồi về trang nháp giúp LLM nhận thức được kết quả khách quan của hành động trước đó, từ đó điều hướng suy nghĩ tiếp theo một cách thực tế và không bị lạc lối trong không gian suy luận.

---

## IV. Future Improvements (5 Points)

1. **Scalability (Khả năng mở rộng):**
   Áp dụng chuẩn trao đổi dữ liệu JSON Schema chuẩn hóa cho phản hồi của Agent (như OpenAI Structured Outputs) thay vì bóc tách chuỗi thô bằng Regex để tăng tính tin cậy tuyệt đối cho bộ Parser trên Production.

2. **Safety (An toàn bảo mật):**
   Tích hợp hệ thống phân tích sắc thái nội dung trực tiếp vào System Prompt để cảnh báo Agent không tạo ra các bài đăng marketing mang tính tiêu cực, lừa đảo hoặc vi phạm luật quảng cáo thương mại.

3. **Performance (Hiệu năng):**
   Tập trung tối giản hóa mô tả tool trong System Prompt để giảm số lượng Prompt Tokens tiêu thụ trong các vòng lặp ReAct lặp đi lặp lại, tối ưu hóa chi phí vận hành hệ thống.
