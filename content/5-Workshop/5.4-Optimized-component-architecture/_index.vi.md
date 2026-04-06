---
title : "Luồng Nhà tuyển dụng"
date : 2024-01-01 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

#### Hành trình Nhà tuyển dụng: Tạo Việc làm & Xếp hạng Ứng viên

Phần này chi tiết luồng hoàn toàn từ đầu đến cuối khi nhà tuyển dụng tạo hoặc cập nhật bài đăng việc làm, từ lệnh gọi API đến xếp hạng ứng viên thông minh và cập nhật dashboard thời gian thực.

---

#### Bước 1: Tạo Việc làm qua API (React + REST API)

Khi nhà tuyển dụng tạo một công việc mới trong dashboard SmartHire-AI:

```
Nhà tuyển dụng điền form:
  - Tiêu đề công việc
  - Mô tả công việc
  - Kỹ năng bắt buộc
  - Vị trí
  - Mức lương
        ↓
React SPA gửi yêu cầu POST:
  Authorization: JWT (Cognito token)
  Content-Type: application/json
        ↓
API Gateway xác thực yêu cầu
        ↓
Lambda (.NET 8) xác thực & lưu dữ liệu
```

---

#### Bước 2: Lưu trữ Việc làm trong RDS

**RDS (PostgreSQL) Insert:**

Dữ liệu công việc được lưu với các trường:
- ID, recruiter ID, tiêu đề, mô tả
- Kỹ năng bắt buộc, vị trí, mức lương
- Ngày đăng, trạng thái

---

#### Bước 3: Kích hoạt Pipeline Xử lý (Lambda + Step Functions)

Ngay sau khi lưu công việc vào RDS, hàm Lambda:

```python
1. Trích xuất mô tả công việc & kỹ năng bắt buộc
2. Gọi cv_jd_processor để:
   - Chuẩn hóa yêu cầu công việc
   - Tạo embeddings công việc bằng Bedrock
   - Trích xuất kỹ năng kỹ thuật chính
3. Gọi Step Functions với bối cảnh công việc:
   - Phân nhánh tới candidate_ranking_engine
```

---

#### Bước 4: Tìm kiếm & Xếp hạng Ứng viên (candidate_ranking_engine Lambda)

**Xử lý:**
1. Truy vấn RDS cho tất cả CVs ứng viên với kỹ năng được phân tích
2. Truy vấn DynamoDB cho embeddings ứng viên được lưu trữ
3. Cho MỖI ứng viên:
   - Tính điểm kết hợp
   - Xếp hạng các thành phần:
     * Tương tự ngữ nghĩa: 40%
     * Kỹ năng trùng khớp chính xác: 40%
     * Căn chỉnh cấp độ kinh nghiệm: 15%
     * Trùng khớp vị trí: 5%
4. Lọc theo ngưỡng tối thiểu (65%)
5. Sắp xếp theo điểm giảm dần
6. Giới hạn đến 50 ứng viên hàng đầu

---

#### Bước 5: Lưu trữ Xếp hạng trong Tầng Dữ liệu

**RDS Update:**
Lưu trữ xếp hạng với:
- job ID, candidate ID, match score
- rank, skill match %, semantic similarity
- ranked at timestamp

**DynamoDB Record:**
```
PK: "JOB#job-12345"
SK: "RANKING#1#can-555"
{
  "matchScore": 86.45,
  "candidateId": "can-555",
  "skills": [...]
}
```

---

#### Bước 6: Cập nhật Dashboard Thời gian thực (AppSync)

Sau khi xếp hạng hoàn tất:

```
Step Functions → Lambda kết thúc
        ↓
Lambda xuất bản đột biến GraphQL AppSync:
  "CandidateRankingReady"
        ↓
AppSync phát qua subscription
        ↓
Dashboard React của Recruiter nhận cập nhật
        ↓
UI được làm mới bằng danh sách ứng viên được xếp hạng
```

---

#### Bước 7: Trải nghiệm Dashboard Nhà tuyển dụng

**Trang Chi tiết Công việc Hiển thị:**
- Tiêu đề công việc, mô tả, ngày đăng
- **Xếp hạng Ứng viên Trực tiếp** (cập nhật thời gian thực):
  - 🏆 Ứng viên hàng đầu với điểm 86
  - 🥈 Ứng viên thứ hai với điểm 81
  - 🥉 Ứng viên thứ ba với điểm 78

**Hành động Nhà tuyển dụng:**
- Nhấp vào ứng viên → Xem đầy đủ CV + chi tiết kết hợp
- Gửi lời mời phỏng vấn
- Thêm ghi chú / từ chối ứng viên
- Cập nhật yêu cầu công việc → Re-trigger xếp hạng

---

#### Bước 8: Cập nhật Liên tục

**Khi Ứng viên Mới Ứng tuyển:**
1. Ứng viên tải lên CV
2. CV được xử lý
3. Tự động xếp hạng lại chống tất cả các công việc
4. Nếu điểm > ngưỡng:
   - Ứng viên được thêm vào danh sách xếp hạng công việc
   - Nhà tuyển dụng được thông báo ngay lập tức
   - Dashboard cập nhật thời gian thực

---

#### Ưu điểm của Pipeline

✅ Xếp hạng Toàn bộ - Tất cả ứng viên được so sánh với công việc mới

✅ Cập nhật Thời gian thực - Ứng viên mới được xếp hạng ngay khi ứng tuyển

✅ Tự động Retrigger - Khi công việc được cập nhật

✅ Sắp xếp theo Điểm - Ứng viên tốt nhất xuất hiện trước
