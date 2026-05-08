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
