Để cài đặt cấu hình **Multi-Agent** (Đa tác nhân) trong OpenClaw, bạn cần tập trung vào việc định nghĩa các "Skill" và cấu hình cách thức các Agent này giao tiếp với nhau thông qua file cấu hình trung tâm.

Dưới đây là các bước chi tiết để thiết lập:

### 1. Cấu trúc thư mục Agentic

Trong OpenClaw, mỗi Agent thường được đại diện bởi một bộ kỹ năng (Skills) hoặc các cấu hình Prompt khác nhau. Bạn nên tổ chức theo cấu trúc:

* `/skills`: Chứa các file định nghĩa công cụ cho từng Agent.
* `openclaw.json`: File cấu hình chính để điều phối.

---

### 2. Định nghĩa các "Chuyên gia" (Expert Skills)

Để các Agent hoạt động đa nhiệm, bạn cần tạo các file JSON trong thư mục `skills`. Ví dụ, bạn muốn có một Agent chuyên về **Lập trình** và một Agent chuyên về **Kiểm thử**:

* **Coder Agent (`coder.json`):** Định nghĩa các hàm liên quan đến viết code, sửa lỗi.
* **Reviewer Agent (`reviewer.json`):** Định nghĩa các hàm kiểm tra tiêu chuẩn code, bảo mật.

---

### 3. Cấu hình Multi-Agent trong `openclaw.json`

Theo schema mới nhất của năm 2026, bạn cần thiết lập chế độ liên kết giữa các Model hoặc các Agent thông qua các nhãn (Labels).

```json
{
  "agents": [
    {
      "name": "Manager_Agent",
      "model": "gpt-4-turbo",
      "mode": "orchestrator",
      "skills": ["routing_skill"]
    },
    {
      "name": "Worker_Agent",
      "model": "llama-3-70b",
      "mode": "executor",
      "skills": ["coding_skill", "search_skill"]
    }
  ],
  "network_binding": "Mode-based"
}

```

> **Lưu ý quan trọng:** Đảm bảo `network_binding` được đặt là `Mode-based` để phù hợp với kiến trúc phân lớp, giúp các Agent nhận diện đúng vai trò (Executor hoặc Orchestrator) của mình.

---

### 4. Thiết lập quy trình giao tiếp (Orchestration)

Để các Agent làm việc cùng nhau, bạn cần một **Agent Điều phối (Orchestrator)**. Quy trình cài đặt bao gồm:

1. **System Prompt:** Thiết lập cho Agent chính quyền hạn điều hướng yêu cầu đến các "sub-agents".
2. **Shared Memory (RAG):** Cài đặt một Vector Database chung (như Milvus hoặc ChromaDB) để các Agent có thể truy cập cùng một nguồn ngữ cảnh, tránh việc mỗi Agent hiểu một kiểu.
3. **Communication Protocol:** Sử dụng cơ chế "Agent-to-Agent" gọi thông qua API nội bộ của OpenClaw.

---

### 5. Triển khai với Docker (Khuyên dùng)

Để quản lý nhiều tác nhân mà không gây xung đột tài nguyên (đặc biệt khi dùng GPU), bạn nên sử dụng **Bind Mounts** để quản lý file cấu hình dễ dàng hơn thay vì dùng Named Volumes.

```bash
docker run -d \
  --name openclaw-multi-agent \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/skills:/app/skills \
  --gpus all \
  openclaw-image:latest

```

Sau khi cài đặt xong, bạn có thể kiểm tra luồng công việc bằng cách gửi một yêu cầu phức tạp (ví dụ: "Hãy viết một hàm Python và nhờ một Agent khác kiểm tra tính bảo mật"). Nếu cấu hình đúng, OpenClaw sẽ tự động luân chuyển tác vụ giữa các Agent đã định nghĩa.

Bạn định dùng mô hình Multi-Agent này cho việc chấm điểm tự động hay để hỗ trợ phát triển phần mềm?

----------------------

Dựa trên yêu cầu của bạn, tôi sẽ thiết lập cấu hình **Multi-Agent** cho hệ thống chấm điểm tự động trong OpenClaw. Hệ thống này sẽ bao gồm một **Chairman Agent** điều phối và các **Specialist Agents** thực thi các nhiệm vụ chuyên biệt.

Dưới đây là cách cài đặt chi tiết:

### 1. Cấu hình file `openclaw.json` (Hệ thống điều phối)

Chúng ta sẽ thiết lập cơ chế **Mode-based** để Chairman có thể quản lý các Specialist.

```json
{
  "system_name": "Agentic_Grading_System",
  "network_binding": "Mode-based",
  "agents": [
    {
      "agent_id": "chairman_agent",
      "role": "Chairman",
      "mode": "orchestrator",
      "model": "gpt-4-turbo",
      "system_prompt": "Bạn là Chairman Agent. Nhiệm vụ của bạn là tổng hợp báo cáo từ các Specialist, giải quyết các xung đột về điểm số và đưa ra kết luận điểm cuối cùng dựa trên barem.",
      "skills": ["grading_aggregator", "conflict_resolver"]
    },
    {
      "agent_id": "specialist_content",
      "role": "Content Specialist",
      "mode": "executor",
      "model": "gpt-4o",
      "system_prompt": "Bạn chuyên phân tích tính chính xác và độ sâu của nội dung bài làm.",
      "skills": ["content_analyzer", "data_extraction"]
    },
    {
      "agent_id": "specialist_structure",
      "role": "Structure Specialist",
      "mode": "executor",
      "model": "llama-3-70b",
      "system_prompt": "Bạn chuyên đánh giá bố cục, tính logic và sự liên kết giữa các phần trong bài.",
      "skills": ["structure_checker"]
    }
  ]
}

```

### 2. Định nghĩa các Skills (Tools) chuyên dụng

Bạn cần tạo các file JSON trong thư mục `/skills` để gán cho các Agent:

* **Tool Trích xuất & Định dạng (`data_tools.json`):**
* **OCR:** Chuyển đổi file bài làm (ảnh/PDF) thành văn bản.
* **JSON Formatter:** Ép kết quả chấm điểm về định dạng JSON để Chairman dễ xử lý.


* **Tool Lưu trữ (`storage_tools.json`):**
* **Data Saver:** Ghi điểm số vào cơ sở dữ liệu hoặc file hệ thống sau khi Chairman đã chốt.



### 3. Quy trình vận hành (Workflow)

Quy trình sẽ diễn ra theo mô hình phân lớp như sau:

1. **Lớp Tích hợp (Integration):** Công cụ **OCR** trích xuất dữ liệu từ bài nộp.
2. **Lớp Điều phối (Orchestration):**
* **Chairman** nhận dữ liệu thô và phân phối cho các **Specialist Agents**.
* **Specialist Content** và **Specialist Structure** trả về kết quả phân tích riêng lẻ.


3. **Lớp Mô hình (Model):**
* **Chairman** so sánh các kết quả, nếu có sự chênh lệch lớn về điểm số, nó sẽ yêu cầu các Specialist giải trình thêm trước khi chốt điểm.


4. **Kết quả:** Dữ liệu được định dạng JSON và lưu trữ thông qua các **Skills** đã cài đặt.

### 4. Triển khai Docker với Bind Mounts

Để quản lý các file cấu hình và dữ liệu bài làm một cách linh hoạt, bạn hãy dùng lệnh sau:

```bash
docker run -d \
  --name openclaw-grading \
  -v $(pwd)/openclaw.json:/app/openclaw.json \
  -v $(pwd)/skills:/app/skills \
  -v $(pwd)/grading_data:/app/data \
  --gpus all \
  openclaw-core:latest

```

Hệ thống này sẽ giúp bạn tự động hóa hoàn toàn quy trình chấm điểm, đảm bảo tính khách quan nhờ sự phân tách vai trò rõ rệt giữa các Agent. Bạn có muốn điều chỉnh thêm tiêu chí chấm điểm cho Specialist nào không?
