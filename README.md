```mermaid
graph TD
    %% Định nghĩa các Actor
    User((Người dùng))
    Admin((Quản trị viên))

    %% Khối Frontend
    subgraph "Frontend Layer"
        UI["Next.js (Web UI)"]
    end

    %% Khối Backend Cốt lõi
    subgraph "Core Backend (Java)"
        SB["Spring Boot<br/>(Auth, Routing, CRUD)"]
    end

    %% Khối Hạ tầng AWS
    subgraph "AWS Infrastructure"
        RDS[("AWS RDS<br/>(MySQL)")]
        S3[("AWS S3<br/>(Lưu trữ File)")]
        SQS["AWS SQS<br/>(Message Queue)"]
    end

    %% Khối Xử lý AI & Tải nặng
    subgraph "AI & Heavy Workload (Python)"
        PY["Python Worker<br/>(FastAPI / LlamaIndex)"]
        KG[("Neo4j<br/>(Knowledge Graph)")]
        LLM(("Google Gemini API<br/>(LLM)"))
    end

    %% ==========================================
    %% LUỒNG 1: TRUY VẤN CHATBOT (ĐỒNG BỘ)
    %% ==========================================
    User -->|1a. Đặt câu hỏi| UI
    UI -->|1b. Gọi API| SB
    SB <-->|1c. Xác thực JWT & Lưu lịch sử| RDS
    SB -->|1d. HTTP Request (Forward)| PY
    PY <-->|1e. Nhờ LLM tạo câu lệnh Cypher| LLM
    PY <-->|1f. Truy vấn GraphDB| KG
    PY <-->|1g. Tổng hợp câu trả lời| LLM
    PY -->|1h. Trả kết quả (Text)| SB
    SB -->|1i. Hiển thị lên UI| UI

    %% ==========================================
    %% LUỒNG 2: XỬ LÝ TÀI LIỆU (BẤT ĐỒNG BỘ)
    %% ==========================================
    Admin -->|2a. Upload PDF/Doc| UI
    UI -->|2b. Gửi File| SB
    SB -->|2c. Lưu File thô| S3
    SB -->|2d. Lưu trạng thái PENDING| RDS
    SB -->|2e. Đẩy Message (kèm link S3)| SQS
    SQS -->|2f. Worker nhận Job| PY
    PY -->|2g. Tải File về| S3
    PY <-->|2h. Bóc tách Thực thể & Quan hệ| LLM
    PY -->|2i. Build Đồ thị (Insert Node/Edge)| KG
    PY -->|2j. Cập nhật trạng thái COMPLETED| SB

    %% Styling cho đẹp mắt
    classDef frontend fill:#e1f5fe,stroke:#03a9f4,stroke-width:2px;
    classDef springboot fill:#e8f5e9,stroke:#4caf50,stroke-width:2px;
    classDef aws fill:#fff3e0,stroke:#ff9800,stroke-width:2px;
    classDef python fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px;
    classDef database fill:#eceff1,stroke:#607d8b,stroke-width:2px;

    class UI frontend;
    class SB springboot;
    class RDS,S3,SQS aws;
    class PY python;
    class KG database;

```
