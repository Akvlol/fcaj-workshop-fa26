---
title : "Tạo và Cấu hình các AWS Lambda Functions"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

Trong phần này, chúng ta sẽ triển khai các **AWS Lambda Functions** của SecureTable bằng cách tạo function trên AWS Console, tải mã nguồn có sẵn và cấp quyền **IAM Execution Role** để Lambda tương tác với Amazon DynamoDB và các dịch vụ liên quan.

Tất cả các function được triển khai tại **Asia Pacific (Singapore) – `ap-southeast-1`**, sử dụng **Runtime Python 3.15** và hình thức **Lambda Event Function** để kết nối với Amazon API Gateway hoặc nhận sự kiện từ EventBridge.

## 1. Tạo AWS Lambda Function

Trước tiên, chúng ta tạo function `SecureTable-GuestServices`:

1. Truy cập **AWS Lambda → Functions → Create function**.
2. Chọn **Event Function** (nếu Console yêu cầu chọn loại function), sau đó chọn **Author from scratch**.
3. Nhập **Function name:** `SecureTable-GuestServices`.
4. Chọn **Runtime:** `Python 3.15` (`python3.15`).
5. Giữ **Architecture:** `x86_64`.
6. Chọn **Create function** và đợi function được tạo thành công.

![Create Lambda Function](../../../images/5-Workshop/5.6-Backend/create-lambda.png)

**Lưu ý:** AWS sẽ tự tạo **IAM Execution Role** cho Lambda và gắn quyền cơ bản để ghi log lên Amazon CloudWatch thông qua policy `AWSLambdaBasicExecutionRole`. Chúng ta **không cần tạo IAM Role thủ công trước**. Tuy nhiên, role này chưa có quyền đọc/ghi các bảng DynamoDB.

## 2. Lấy và Upload Source Code

Mã nguồn backend SecureTable đã được chuẩn bị sẵn. Thay vì viết lại mã nguồn cho từng Lambda, chúng ta sẽ tải các gói triển khai từ liên kết sau:

**[Lấy source code ở đây](https://github.com/Akvlol/fcaj-workshop-fa26/tree/main/src_code/Lambda)**

Sau khi tải về, chọn đúng gói `.zip` tương ứng với function cần triển khai, rồi thực hiện:

1. Mở Lambda `SecureTable-GuestServices` vừa tạo.
2. Vào tab **Code**.
3. Chọn **Upload from → .zip file**.
4. Chọn tệp ZIP của `SecureTable-GuestServices` và xác nhận **Save**.
5. Kiểm tra **Runtime settings → Handler** là `lambda_function.lambda_handler`. Nếu khác, chọn **Edit** để sửa.
6. Nếu thay đổi code bằng trình chỉnh sửa trực tiếp trên Console, chọn **Deploy** để áp dụng.

![Upload Lambda Source Code](../../../images/5-Workshop/5.6-Backend/lambda-upload.png)

**Lưu ý:** File `lambda_function.py` và các module được import phải nằm ở đúng vị trí trong ZIP để Python có thể tải. Không upload nguyên thư mục chứa ZIP vào Lambda. Mã nguồn từng function phải tương thích với **Python 3.15**; nếu gói có thư viện ngoài hoặc Lambda Layer, cần kiểm tra phiên bản hỗ trợ trước khi triển khai.

## 3. Cấu hình Memory, Timeout và Environment Variables

Sau khi upload mã nguồn, chúng ta tiếp tục kiểm tra cấu hình thực thi:

1. Vào **Configuration → General configuration → Edit**.
2. Điều chỉnh **Memory** và **Timeout** phù hợp với từng function.
3. Chọn **Save**.
4. Nếu mã nguồn có sử dụng biến môi trường, mở **Configuration → Environment variables → Edit** và nhập các giá trị cần thiết.
5. Kiểm tra lại **Runtime settings** để xác nhận runtime là **Python 3.15** và Handler là `lambda_function.lambda_handler`.

![Lambda General Configuration](../../../images/5-Workshop/5.6-Backend/lambda-general.png)

Có thể sử dụng cấu hình khởi đầu sau cho môi trường demo:

| Lambda Function | Memory tham khảo | Timeout tham khảo |
|---|---:|---:|
| `SecureTable-GuestServices` | 256 MB | 15 giây |
| `SecureTable-Ordering` | 256 MB | 15 giây |
| `SecureTable-Reception` | 512 MB | 30 giây |
| `SecureTable-Kitchen` | 256 MB | 15 giây |
| `SecureTable-Inventory` | 512 MB | 30 giây |
| `SecureTable-AdminDashboard` | 512 MB | 30 giây |
| `SecureTable-Report` | Theo tác vụ | Cấu hình tại mục [5.7](../../5.7-Automation/) |
| `SecureTable-Backup` | Theo tác vụ | Cấu hình tại mục [5.7](../../5.7-Automation/) |

**Lưu ý:** Đây là các giá trị **tham khảo cho demo**, không phải cấu hình bắt buộc. Memory và Timeout cần được điều chỉnh theo thời gian chạy thực tế trong CloudWatch. Với API đồng bộ, thời gian xử lý còn bị giới hạn bởi timeout của API Gateway.

## 4. Bổ sung quyền IAM cho Execution Role mặc định

Execution Role do AWS tự tạo ban đầu chỉ có quyền ghi CloudWatch Logs. Để function đọc/ghi dữ liệu DynamoDB, chúng ta phải bổ sung quyền cần thiết vào role này.

Ví dụ với `SecureTable-GuestServices`:

1. Mở Lambda → **Configuration → Permissions**.
2. Tại **Execution role**, nhấn vào tên role để chuyển sang **IAM Console**.
3. Chọn **Add permissions → Create inline policy**.
4. Chuyển sang chế độ **JSON** và nhập policy phù hợp với function.
5. Thay `<ACCOUNT_ID>` bằng AWS Account ID của tài khoản đang triển khai.
6. Chọn **Next**, đặt tên policy (ví dụ `SecureTable-GuestServices-DynamoDB`), rồi chọn **Create policy**.

Ví dụ IAM Policy cho `SecureTable-GuestServices`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadSessionTableAndBill",
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem"],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/SecureTable-Tables",
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/SecureTable-Bills",
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/SecureTable-GuestSessions",
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/SecureTable-Ingredients"
      ]
    },
    {
      "Sid": "ReadMenu",
      "Effect": "Allow",
      "Action": ["dynamodb:Scan"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/SecureTable-MenuItems"
    },
    {
      "Sid": "CreateGuestSession",
      "Effect": "Allow",
      "Action": ["dynamodb:PutItem"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/SecureTable-GuestSessions"
    }
  ]
}
```

![Lambda Execution Role Permissions](../../../images/5-Workshop/5.6-Backend/lambda-iam.png)

**Lưu ý:** Mỗi Lambda cần **IAM Policy riêng**, chỉ cấp quyền đối với bảng và hành động mà mã nguồn thực sự sử dụng. Không gắn `AmazonDynamoDBFullAccess` cho tất cả các Lambda. Đối với transaction, role phải có các quyền DynamoDB tương ứng với thao tác bên trong transaction; nếu truy vấn qua GSI, cần xét thêm ARN của index.

## 5. Triển khai các Lambda còn lại

Lặp lại quy trình **Create function → Upload ZIP → Cấu hình → Bổ sung IAM Policy** cho các Lambda còn lại:

| Lambda Function | Chức năng |
|---|---|
| `SecureTable-GuestServices` | Tạo Guest Session và cung cấp thực đơn |
| `SecureTable-Ordering` | Nhận đơn và cho Customer xem lại món đã đặt |
| `SecureTable-Reception` | Quản lý bàn, mở và đóng bill |
| `SecureTable-Kitchen` | Xem hàng đợi, hoàn tất hoặc từ chối món |
| `SecureTable-Inventory` | Nhập kho, xem lịch sử và điều chỉnh phiếu |
| `SecureTable-AdminDashboard` | Truy xuất dữ liệu tài chính và Audit Logs |
| `SecureTable-Report` | Tổng hợp và gửi báo cáo tự động |
| `SecureTable-Backup` | Khởi tạo sao lưu DynamoDB sang S3 |

**Lưu ý:** Một Lambda có thể xử lý **nhiều API endpoints**. Ví dụ, `SecureTable-GuestServices` dùng chung cho `POST /v1/guest/sessions` và `GET /v1/guest/menu`; `SecureTable-Ordering` xử lý cả `POST` và `GET /v1/guest/orders`. Không cần tạo riêng một Lambda cho từng HTTP Method.

Các Lambda `SecureTable-Report` và `SecureTable-Backup` **không cần gắn API Gateway**, vì chúng nhận sự kiện từ lịch tự động; phần kích hoạt và quyền liên quan tới SNS/S3 được trình bày ở mục **5.7 – Tự động hóa Báo cáo & Sao lưu dữ liệu**.

## 6. Kiểm tra kết quả

1. Truy cập **AWS Lambda → Functions**, xác nhận đã tạo đủ các function cần thiết.
2. Mở từng function, kiểm tra **Runtime: Python 3.15** và **Handler: `lambda_function.lambda_handler`**.
3. Vào **Configuration → Permissions**, xác nhận Execution Role đã được bổ sung quyền truy cập đúng các DynamoDB Tables cần dùng.
4. Vào **Monitor → View CloudWatch logs** để theo dõi lỗi khi Lambda được gọi.
5. Có thể sử dụng tab **Test** với event mẫu để kiểm tra lỗi cú pháp, import và cấu hình. Những API yêu cầu session/token phải có dữ liệu xác thực hợp lệ mới kiểm tra được đầy đủ.

![Lambda Functions List](../../../images/5-Workshop/5.6-Backend/lambda-list.png)

**Lưu ý:** Tạo event Test giả lập trên Lambda Console **không thay thế** kiểm thử Cognito Authorizer hoặc API Gateway thực tế. Chúng ta sẽ kết nối endpoint và kiểm thử request qua API Gateway trong **5.6.3 – Tích hợp Lambda với Amazon API Gateway**.

Sau khi hoàn thành, các AWS Lambda Functions đã được triển khai với **Python 3.15**, sử dụng **Execution Role mặc định được bổ sung IAM Policy theo chức năng**, và sẵn sàng để tích hợp với Amazon API Gateway.
