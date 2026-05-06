# **BÁO CÁO TIẾN ĐỘ DỰ ÁN GAKU AI — STEM STEP VERIFIER**
Nhóm A20-App-022

## **1. Tổng quan dự án**

Dự án **Gaku AI — STEM Step Verifier** xây dựng nền tảng gia sư AI cho sinh viên STEM theo hướng **xác minh từng bước giải** thay vì chỉ trả lời đúng/sai ở đáp án cuối. Hệ thống hỗ trợ sinh viên nhận phản hồi nhanh, gợi ý kiểu Socratic, và cho phép giảng viên theo dõi hàng đợi các bài cần review khi độ tin cậy thấp.

Mục tiêu hiện tại:
- Phản hồi gần thời gian thực cho từng bài làm.
- Tăng độ chính xác xác minh bước giải.
- Giảm tải chấm thủ công cho giảng viên thông qua cơ chế escalation.

## Link sản phẩm
- Demo local: `http://localhost:5173`
- API local: `http://localhost:8000`
- Production URL: *(bổ sung sau)*

Tài khoản test:
- Teacher
```
teacher@demo.com
```
- Student
```
student@demo.com
```
- Password
```
demo123
```

## **2. Các hạng mục đã hoàn thành**

### **2.1. Frontend – Backend (Core Features)**

Trong giai đoạn hiện tại, hệ thống đã hoàn thiện các chức năng cốt lõi cho cả sinh viên và giảng viên:

- Xây dựng **xác thực người dùng** (đăng ký, đăng nhập, lấy thông tin user) với JWT.
- Hoàn thiện **role-based app** cho 2 vai trò: `student` và `teacher`.
- Triển khai trang **Dashboard sinh viên**: theo dõi tiến độ, bài tập, lịch sử học.
- Triển khai trang **Dashboard giảng viên**: lớp học, danh sách bài tập, review queue, insights.
- Xây dựng luồng **giải bài theo từng problem** với giao diện nhập lời giải và kiểm tra trực tiếp.
- Tích hợp API **`/api/verify`** để chấm theo pipeline nhiều bước:
  - normalize input
  - route case
  - phân tích cú pháp/tĩnh
  - sandbox/test execution
  - tổng hợp confidence + evidence
  - phát hiện trường hợp cần escalation
- Hỗ trợ **2 luồng xác minh**:
  - `math`: kiểm tra tương đương biểu thức và logic biến đổi bằng SymPy-first pipeline.
  - `python/cs`: kiểm tra code qua static analysis + sandbox + test runner.
- Tích hợp **hint thích ứng** qua `/api/hint/start` và `/api/hint/next`:
  - gợi ý mở đầu
  - gợi ý theo tiến độ
  - gợi ý debug theo lỗi
  - quota/lock cho hint thường
- Trả về **evidence chain** để minh bạch quá trình xác minh từng bước/tool.
- Tích hợp dữ liệu môn học, đề bài, bài nộp mẫu và seed demo để chạy end-to-end.

> TODO: Thêm ảnh giao diện Landing/Login/Dashboard/Solve/Review Queue tại đây.

---

### **2.2. Hạ tầng (Infrastructure & DevOps)**

#### **Môi trường triển khai hiện tại**

- Đóng gói dịch vụ bằng **Docker + docker-compose** với 4 thành phần:
  - `frontend` (Vite build + Nginx)
  - `backend` (FastAPI)
  - `db` (PostgreSQL 16)
  - `adminer` (quản trị DB)
- Backend có endpoint **`/health`** và healthcheck cho container.
- Backend dùng **AsyncOpenAI client** lifecycle theo FastAPI lifespan.

#### **Database & Data Layer**

- Sử dụng **PostgreSQL** cho dữ liệu nghiệp vụ (user, lớp học, bài tập, lịch sử giải).
- ORM: **SQLModel + SQLAlchemy async**.
- Có cơ chế **seed dữ liệu demo** tự động khi khởi chạy backend.

#### **AI & Knowledge Services**

- Tích hợp mô hình OpenAI cho các bước cần LLM (contract extraction, hint generation, routing fallback).
- Có pipeline ingest kiến thức vào **Pinecone** (phục vụ mở rộng RAG cho gợi ý theo giáo trình).

> TODO: Thêm ảnh kiến trúc hệ thống / docker services / database schema tại đây.

---

### **2.3. Quy trình chất lượng & vận hành hiện tại**

Quy trình hiện tại tập trung vào kiểm soát chất lượng code và logging AI trước khi đẩy code:

```text
Developer code change
        ↓
Chạy kiểm tra cục bộ (pytest / lint / build)
        ↓
Git pre-push hook kích hoạt
        ↓
Tự động submit .ai-log/session.jsonl lên AI log server
        ↓
Push code
```

Các điểm đã có:
- Test suite backend (unit + integration).
- Frontend lint/build scripts.
- Cơ chế hook tự động gửi AI prompt log (không chặn push nếu server log lỗi).

Các điểm đang hoàn thiện tiếp:
- Chuẩn hóa CI pipeline tập trung (test/lint/build tự động trên remote).
- Tự động hóa deploy production theo branch/release.

---

## **3. Kế hoạch giai đoạn tiếp theo**

### **3.1. Frontend**

- Tối ưu UX cho màn **Solve**:
  - hiển thị evidence rõ hơn theo từng bước sai
  - cải thiện flow sửa bài và nộp lại nhiều vòng
- Nâng cấp dashboard sinh viên/giảng viên với biểu đồ tiến độ theo kỹ năng.
- Bổ sung khả năng quản lý lịch sử hội thoại/hint trực quan hơn.

### **3.2. Backend/AI**

- Tăng độ an toàn thực thi code bằng sandbox cô lập cao hơn (container-level isolation).
- Mở rộng pipeline verify cho nhiều dạng bài STEM hơn.
- Tích hợp **RAG textbook retrieval** vào luồng gợi ý để hint bám sát tài liệu môn học.
- Tối ưu confidence calibration và ngưỡng escalation cho từng môn.

### **3.3. Hạ tầng & Giám sát hệ thống**

- Thiết lập pipeline CI/CD đầy đủ: test gating, build image, deploy tự động.
- Bổ sung logging/monitoring tập trung cho backend và agent pipeline.
- Chuẩn hóa môi trường staging/production và tài liệu vận hành.

> TODO: Thêm ảnh roadmap/kanban/monitoring dashboard tại đây.
