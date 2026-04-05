---
title: "Đề xuất"
date: 2026-01-05
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SmartHire AI
## Giải pháp Serverless trên AWS để Trích xuất CV tự động và Đánh giá Ứng viên

### 1. Tóm tắt Điều hành
SmartHire AI là nền tảng hỗ trợ tuyển dụng được xây dựng trên kiến trúc Serverless 100% trên AWS, nhằm tự động hóa hai quy trình cốt lõi tốn thời gian nhất trong hoạt động HR: nhận và phân tích CV của ứng viên, cũng như tập hợp và đánh giá kết quả phỏng vấn. Hệ thống tận dụng kiến trúc hướng sự kiện kết hợp với các dịch vụ AI sinh tạo (Amazon Bedrock) để cung cấp hiệu suất cao, giảm chi phí băng thông và đánh giá ứng viên công bằng, nhất quán.

### 2. Phát biểu Vấn đề
### Vấn đề là gì?
Tải lên các tệp CV lớn (PDF, DOCX) thông qua máy chủ trung gian thường gây nghẽn băng thông, làm giảm trải nghiệm ứng viên và tăng chi phí cơ sở hạ tầng. Các đội ngũ HR và Lãnh đạo Kỹ thuật phải đọc thủ công từng CV để lọc kỹ năng, tiêu tốn nhiều nhân lực và có nguy cơ bỏ sót những ứng viên tiềm năng. Sau các cuộc phỏng vấn, việc tập hợp phản hồi và so sánh điểm số giữa các ứng viên thường thiếu sự nhất quán và phụ thuộc nhiều vào phán đoán cá nhân của người phỏng vấn.

### Giải pháp
Nền tảng sử dụng S3 Presigned URLs để ứng viên có thể tải CV trực tiếp lên Amazon S3 mà không cần thông qua Backend, hoàn toàn loại bỏ tắc nghẽn băng thông. Pipeline S3 Event Notification → SQS → Lambda → Amazon Textract tự động trích xuất văn bản thô một cách không đồng bộ ngay khi CV được tải lên. Amazon Bedrock (Claude 3.5 Sonnet) phân tích văn bản thô thành JSON có cấu trúc lưu trữ trong DynamoDB. Trong khi đó, AWS Step Functions điều phối luồng công việc đánh giá phỏng vấn song song dựa trên phương pháp STAR, tự động tạo báo cáo cho HR.

### Lợi ích và Lợi nhuận Đầu tư
Giải pháp cho phép HR tập trung vào các nhiệm vụ có giá trị cao thay vì đọc thủ công từng hồ sơ, giảm thời gian sàng lọc CV đi 80%. Báo cáo sau phỏng vấn được tạo tự động trong chưa đến 90 giây thông qua xử lý song song, đảm bảo tiêu chí đánh giá nhất quán và khách quan cho tất cả ứng viên. Kiến trúc Serverless Trả tiền theo cách sử dụng tối ưu hóa chi phí hoạt động và mở rộng quy mô vô hạn để hỗ trợ các chiến dịch tuyển dụng quy mô lớn.

### 3. Kiến trúc Giải pháp
Nền tảng sử dụng kiến trúc Serverless với sự phân tách rõ ràng giữa lớp API và các pipeline xử lý nền nặng. Tất cả xử lý dữ liệu được thực hiện không đồng bộ để đảm bảo trải nghiệm người dùng liền mạch:

![Kiến trúc SmartHire](https://res.cloudinary.com/diahbkjog/image/upload/v1775413223/smarthire_architecture_cugoh4.png)

### Các Dịch vụ AWS được Sử dụng
- **Amazon S3**: Lưu trữ CV gốc, tạo Presigned URLs cho tải lên trực tiếp của ứng viên, và lưu trữ báo cáo PDF/HTML cuối cùng.
- **Amazon SQS**: Hàng đợi tin nhắn ngăn chặn hệ thống bị quá tải khi hàng ngàn CV được tải lên cùng một lúc.
- **AWS Lambda**: Các hàm worker thực thi logic gọi AI, xử lý dữ liệu và lưu trữ kết quả.
- **Amazon Textract**: Công cụ Nhận dạng ký tự quang học (OCR) để trích xuất văn bản từ các định dạng tệp PDF và DOCX.
- **Amazon Bedrock**: Cung cấp LLM (Claude 3.5 Sonnet) để phân tích CV ngữ nghĩa và đánh giá câu trả lời phỏng vấn.
- **AWS Step Functions**: Điều phối các luồng công việc nhiều bước để đánh giá ứng viên song song.
- **Amazon DynamoDB**: Lưu trữ siêu dữ liệu ứng viên, cấu trúc kỹ năng được trích xuất và trạng thái xử lý.
- **Amazon API Gateway**: Cổng giao tiếp giữa Frontend và các hàm Lambda Backend.
- **Amazon Cognito**: Xác thực và phân quyền cho người dùng (ứng viên, HR, nhà tuyển dụng).

### Thiết kế Thành phần
- **Lớp Tải lên CV**: Frontend gọi API Gateway → Lambda → tạo S3 Presigned URL; ứng viên PUT tệp trực tiếp lên S3.
- **Pipeline Hướng sự kiện**: S3 Event Notification kích hoạt SQS → Lambda Worker gọi Textract và Bedrock → lưu JSON trong DynamoDB.
- **Luồng công việc Đánh giá AI**: Step Functions Express Workflow điều phối Parse → ScoreParallel → Aggregate → GenerateReport.
- **Lớp Thông báo**: Lambda cập nhật trạng thái xử lý và cung cấp kết quả cho Frontend thông qua AppSync (GraphQL Subscription).
- **Xác thực & Bảo mật**: Cognito User Pool phát hành mã thông báo JWT; API Gateway sử dụng Cognito Authorizer để xác thực mọi yêu cầu được bảo vệ.

### 4. Triển khai Kỹ thuật
**Các Giai đoạn Triển khai**
Dự án được cung cấp trong 4 giai đoạn liên tiếp trong thời kỳ thực tập:
- **Giai đoạn 1 — Cơ sở hạ tầng Lưu trữ & API Tải lên (Tuần 1):** Thiết lập nhóm S3, định cấu hình IAM Roles, và xây dựng Lambda API để tạo Presigned URLs cho Frontend. Ứng viên tải CV trực tiếp lên S3 mà không cần qua Backend.
- **Giai đoạn 2 — Pipeline Hướng sự kiện & Textract (Tuần 2):** Định cấu hình S3 Event Notification → SQS trigger → Lambda Worker được tích hợp với Amazon Textract để trích xuất văn bản thô từ các tệp PDF/DOCX.
- **Giai đoạn 3 — Tích hợp Bedrock & DynamoDB (Tuần 3):** Thiết kế Kỹ thuật Kỹ dẫn lời nhắc, gọi Amazon Bedrock (Claude 3.5 Sonnet) để phân tích văn bản thô thành JSON (`skills`, `yearsOfExperience`, `education`), và lưu trữ kết quả trong DynamoDB.
- **Giai đoạn 4 — Step Functions & Hoàn thành (Tuần 4):** Xây dựng Step Functions Express Workflow với Parallel State cho đánh giá phỏng vấn đồng thời, tạo báo cáo PDF/HTML được lưu trở lại S3. Tiến hành kiểm thử bảo mật, tối ưu hóa và tài liệu.

**Yêu cầu Kỹ thuật**
- **Pipeline Phân tích CV:** Nhóm S3 (tiền tố `cvs/`), Hàng đợi SQS Tiêu chuẩn với DLQ, Lambda runtime Python 3.12, Amazon Textract AnalyzeDocument API, Amazon Bedrock InvokeModel API (claude-3-5-sonnet), Bảng DynamoDB với Single Table Design.
- **Luồng công việc Đánh giá AI:** Step Functions Express Workflow, Parallel State với tối đa 10 nhánh đồng thời, Lambda được tích hợp với Bedrock để tạo tóm tắt toàn bộ, S3 cho lưu trữ đầu ra báo cáo.
- **API & Xác thực:** API Gateway REST API, Cognito User Pool + App Client, JWT Authorizer trên tất cả các điểm cuối được bảo vệ.

### 5. Lịch trình & Mốc quan trọng
**Lịch trình Dự án**
- **Tuần 1:** Thiết lập nhóm S3, xây dựng API Presigned URL; định cấu hình IAM Roles cho Lambda; kiểm thử luồng tải lên trực tiếp từ Frontend.
- **Tuần 2:** Xây dựng Pipeline Hướng sự kiện: S3 Event Notification → SQS → Lambda; tích hợp và kiểm thử Amazon Textract trên các định dạng PDF và DOCX khác nhau.
- **Tuần 3:** Thiết kế Kỹ thuật Kỹ dẫn lời nhắc cho Amazon Bedrock; chuẩn hóa đầu ra JSON; lưu trữ siêu dữ liệu trong DynamoDB; đánh giá độ chính xác của trích xuất.
- **Tuần 4:** Xây dựng State Machine Step Functions với Parallel scoring; tạo báo cáo PDF/HTML; tiến hành kiểm thử bảo mật (IAM, Cognito, S3 Bucket Policy) và hoàn thành tài liệu.

### 6. Ước tính Ngân sách
Bạn có thể khám phá ước tính chi phí trên [AWS Pricing Calculator](https://calculator.aws/).

### Chi phí Cơ sở hạ tầng
- **AWS Lambda:** $0.00/tháng (nằm trong Free Tier — 1 triệu request đầu tiên miễn phí).
- **Amazon S3 Standard:** ~$0.05/tháng (lưu trữ CV và báo cáo, ~2 GB).
- **Amazon SQS:** $0.00/tháng (dưới 1 triệu request/tháng — Free Tier).
- **Amazon Textract:** ~$0.015/trang (AnalyzeDocument API).
- **Amazon Bedrock (Claude 3.5 Sonnet):** ~$0.10–$0.30/tháng tùy theo khối lượng xử lý CV.
- **AWS Step Functions:** $0.00/tháng (4.000 chuyển tiếp trạng thái đầu tiên miễn phí).
- **Amazon DynamoDB:** $0.00/tháng (25 GB lưu trữ + 25 WCU/RCU — Free Tier).
- **Amazon API Gateway:** ~$0.01/tháng (dưới 1 triệu lệnh gọi).

**Tổng Ước tính Chi phí: ~$0.50–$1.00/tháng** (tùy thuộc vào khối lượng lưu thông CV thực tế)

### 7. Đánh giá Rủi ro
#### Ma trận Rủi ro
- **Kỹ thuật Kỹ dẫn lời nhắc Không chính xác:** Ảnh hưởng cao, xác suất trung bình — trích xuất CV tạo ra các giá trị trường không chính xác.
- **Lambda Hết thời gian chờ:** Ảnh hưởng trung bình, xác suất thấp — tệp CV quá lớn hoặc Textract/Bedrock phản hồi chậm.
- **Bedrock Chi phí Vượt ngân sách:** Ảnh hưởng trung bình, xác suất thấp — nếu khối lượng tải CV tăng đột ngột.
- **Tích Lũy Tin nhắn DLQ:** Ảnh hưởng thấp, xác suất trung bình — Lambda Worker liên tục không thể xử lý tin nhắn.

#### Chiến lược Giảm thiểu
- **Kỹ thuật Kỹ dẫn lời nhắc:** Xây dựng bộ kiểm thử bằng các CV đa dạng (tiếng Việt, tiếng Anh, nhiều định dạng); đánh giá độ chính xác mỗi lần lặp lại trước khi triển khai.
- **Lambda Hết thời gian chờ:** Tăng timeout khi cần (tối đa 15 phút); chia các bước Textract và Bedrock thành hai hàm Lambda riêng biệt nếu cần thiết.
- **Chi phí:** Thiết lập Cảnh báo AWS Budget để thông báo khi chi tiêu vượt $5/tháng; sử dụng giá Bedrock On-Demand thay vì Provisioned.
- **DLQ:** Cấu hình CloudWatch Alarm để theo dõi `ApproximateNumberOfMessagesNotVisible`; tạo runbook để xử lý lại DLQ định kỳ.

#### Kế hoạch Dự phòng
- Nếu Bedrock không khả dụng: quay lại phân tích dựa trên Regex để giữ pipeline hoạt động.
- Nếu Step Functions gặp sự cố: thực hiện lại các lần thực thi thủ công qua AWS Console hoặc CLI.

### 8. Kết quả Kỳ vọng
#### Cải tiến Kỹ thuật: 
Tải lên và xử lý CV nhanh chóng, liền mạch mà không có gián đoạn, nhờ kiến trúc hoàn toàn không đồng bộ. Hệ thống có thể mở rộng vô hạn để xử lý các chiến dịch tuyển dụng quy mô lớn mà không cần nâng cấp máy chủ.
#### Giá trị Dài hạn
Giảm **80%** thời gian đọc và phân loại CV thủ công. Báo cáo sau phỏng vấn được tạo tự động trong **chưa đến 90 giây** thông qua xử lý song song. Tiêu chí đánh giá nhất quán, khách quan có thể tái sử dụng trên nhiều vị trí công việc. Mô hình Serverless Trả tiền theo cách sử dụng tối ưu hóa chi phí hoạt động bằng cách chỉ tính phí cho mức sử dụng thực tế.
