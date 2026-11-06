# MiniDistributedSystem – EduSubmit, lát cắt 1

Hai tiến trình độc lập giao tiếp qua HTTP (MEP yêu cầu–phản hồi):

- `src/Submission.Api` – máy chủ, Minimal API trên ASP.NET Core (.NET 10)
- `src/Submission.Client` – máy khách dòng lệnh dùng `HttpClient`

## Tự tạo lại từ đầu bằng .NET CLI

```bash
mkdir MiniDistributedSystem && cd MiniDistributedSystem
dotnet new sln -n MiniDistributedSystem
dotnet new web -n Submission.Api -o src/Submission.Api
dotnet new console -n Submission.Client -o src/Submission.Client
dotnet sln add src/Submission.Api/Submission.Api.csproj
dotnet sln add src/Submission.Client/Submission.Client.csproj
```

## Chạy

Terminal 1 (API):

```bash
dotnet run --project src/Submission.Api --urls http://localhost:5000
```

Terminal 2 (Client):

```bash
dotnet run --project src/Submission.Client -- http://localhost:5000
```

Kết quả mong đợi:

```
Danh sach bai tap:
- 1: Bai tap 1
- 2: Bai tap 2
Da nop bai. Ma luot nop: <guid>
Trang thai: received
```

Có thể thử từng endpoint bằng `requests.http` (VS Code REST Client / Visual Studio) hoặc `curl`.

## Bài tập

1. Thêm bài tập mẫu thứ 3 và kiểm tra máy khách in đúng.
2. Tắt API rồi chạy máy khách: ghi lại lỗi và giải thích.
3. Gửi thiếu `StudentCode`/`RepositoryUrl`: quan sát 400 Bad Request.
4. (Mở rộng) Thêm `GET /api/submissions/{id}` và cho máy khách tra cứu lại lượt nộp.

## Bước 12 – đọc lại lượt nộp

API lưu các `SubmissionResult` trong danh sách `submissions` ở bộ nhớ. Sau khi POST tạo lượt nộp thành công, client lấy `Id` từ phản hồi rồi gọi GET `/api/submissions/{id}` để đọc lại. GET trả `200 OK` nếu tìm thấy và `404 Not Found` nếu không có. Danh sách này chỉ tồn tại trong thời gian tiến trình API đang chạy.

Luồng thông điệp:

```text
Client --(HTTP GET /api/assignments)--> Api : request-response, 200 OK
Client <--(danh sách bài tập)------------- Api

Client --(HTTP POST /api/submissions)----> Api : request-response, 201 Created
Client <--(SubmissionResult có Id)-------- Api

Client --(HTTP GET /api/submissions/{id})-> Api : request-response, 200 OK
Client <--(SubmissionResult đã lưu)-------- Api

Client --(HTTP GET /api/submissions/{id} không tồn tại)--> Api : request-response, 404 Not Found
```

Trong `requests.http`, gửi POST hợp lệ trước; request GET ngay sau đó dùng biến `createSubmission.response.body.$.id` để truy vấn đúng bản ghi vừa tạo. GET với GUID cố định khác sẽ minh họa phản hồi 404.
