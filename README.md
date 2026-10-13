# FaceCheck — Đặc tả hệ thống điểm danh bằng nhận diện khuôn mặt

- **Tên dự án:** FaceCheck
- **Phiên bản đặc tả:** 1.0
- **Trạng thái:** Draft — đang trong giai đoạn thiết kế
- **Nền tảng:** Web Application + REST API + ESP32-CAM
- **Ngôn ngữ giao diện dự kiến:** Tiếng Việt
- **Mục tiêu:** Xây dựng hệ thống điểm danh bằng nhận diện khuôn mặt, có trang quản trị, lưu kết quả điểm danh trên cloud và tự động xóa ảnh kiểm thử sau tối đa 24 giờ.

---

# 1. Tổng quan dự án

## 1.1. Mục đích

FaceCheck là hệ thống điểm danh sử dụng camera ESP32-CAM để chụp ảnh, backend Python để xử lý và nhận diện khuôn mặt, cơ sở dữ liệu cloud để lưu kết quả, và Next.js để cung cấp giao diện quản trị.

Hệ thống được thiết kế theo hướng tiết kiệm tài nguyên:

- ESP32-CAM chỉ thực hiện nhiệm vụ chụp ảnh và gửi ảnh.
- Python backend xử lý ảnh, thực hiện nhận diện khuôn mặt và quyết định kết quả điểm danh.
- Next.js cung cấp giao diện quản trị và xem lịch sử.
- Supabase lưu dữ liệu nghiệp vụ, không bắt buộc lưu ảnh khuôn mặt.
- Ảnh tạm phục vụ kiểm thử được chuyển sang WebP và tự động xóa sau thời hạn quy định.
- Người dùng thông thường không cần đăng nhập tài khoản web để thực hiện điểm danh tại thiết bị.

## 1.2. Phạm vi phiên bản đầu tiên

Phiên bản MVP (Minimum Viable Product) tập trung vào:

1. Nhận ảnh từ ESP32-CAM qua mạng Internet.
2. Xác thực thiết bị gửi ảnh.
3. Kiểm tra tính hợp lệ của ảnh.
4. Chuyển đổi JPEG sang WebP khi cần lưu ảnh tạm.
5. Nhận diện khuôn mặt bằng Python.
6. Đối chiếu khuôn mặt với dữ liệu đã đăng ký.
7. Ghi nhận kết quả điểm danh vào Supabase.
8. Chống ghi nhận điểm danh trùng trong cùng một phiên.
9. Cung cấp trang quản trị để quản lý người được điểm danh, phiên điểm danh và lịch sử.
10. Tự động xóa ảnh tạm sau tối đa 24 giờ theo chính sách lưu trữ.
11. Ghi log kỹ thuật để hỗ trợ phát hiện và xử lý lỗi.

## 1.3. Ngoài phạm vi MVP

Các chức năng sau chưa thuộc phạm vi bắt buộc của phiên bản đầu tiên:

- Ứng dụng native dành riêng cho Android hoặc iOS.
- Điểm danh ngoại tuyến khi không có Internet.
- Nhận diện đồng thời nhiều khuôn mặt trong một khung hình.
- Chống giả mạo nâng cao bằng camera chiều sâu hoặc cảm biến chuyên dụng.
- Tự động chấm công, tính lương hoặc tính giờ làm.
- Đồng bộ dữ liệu với hệ thống quản lý trường học.
- Nhận diện danh tính chỉ từ một ảnh bất kỳ mà không có bước đăng ký trước.
- Huấn luyện mô hình AI riêng từ đầu.

---

# 2. Đối tượng sử dụng và phân quyền

## 2.1. Quản trị viên (Admin)

Admin là người được phép quản lý hệ thống.

Quyền hạn:

- Đăng nhập vào khu vực quản trị.
- Xem dashboard thống kê.
- Tạo, xem, chỉnh sửa và vô hiệu hóa hồ sơ người tham gia.
- Đăng ký dữ liệu khuôn mặt khi có sự đồng ý hợp lệ.
- Tạo và quản lý phiên điểm danh.
- Xem lịch sử điểm danh.
- Sửa hoặc hủy bản ghi sai theo quy trình được phân quyền.
- Quản lý thiết bị ESP32-CAM.
- Xem trạng thái hoạt động và lỗi thiết bị.
- Quản lý chính sách lưu trữ ảnh tạm.
- Yêu cầu xóa dữ liệu theo chính sách.
- Xem nhật ký hoạt động quản trị.

Không được phép:

- Xem hoặc tải ảnh tạm nếu không có quyền cần thiết.
- Truy cập khóa bí mật từ giao diện trình duyệt.
- Xóa dấu vết kiểm toán của các thao tác quan trọng.
- Thay đổi dữ liệu điểm danh mà không tạo lịch sử điều chỉnh.

## 2.2. Người được điểm danh

Người được điểm danh không bắt buộc có tài khoản web.

Quy trình:

1. Đứng trước camera đã được cấu hình.
2. Camera chụp ảnh theo chu kỳ.
3. Backend xử lý ảnh.
4. Hệ thống trả về trạng thái nhận diện và điểm danh.
5. Kết quả được lưu nếu đáp ứng các điều kiện nghiệp vụ.

Hệ thống không được hiển thị thông tin cá nhân của người khác qua giao diện thiết bị công khai.

## 2.3. Thiết bị camera

Mỗi thiết bị phải có danh tính riêng, bao gồm:

- ID thiết bị.
- Tên thiết bị.
- Địa điểm lắp đặt.
- Trạng thái hoạt động.
- Thời điểm liên lạc gần nhất.
- Thông tin phiên bản firmware.
- Thông tin xác thực riêng của thiết bị.

Một thiết bị bị vô hiệu hóa không được tiếp tục gửi yêu cầu điểm danh hợp lệ.

---

# 3. Kiến trúc tổng thể

## 3.1. Sơ đồ hệ thống

```text
                         INTERNET
                             |
                   +-------------------+
                   |   ESP32-CAM       |
                   | Chụp ảnh JPEG     |
                   +-------------------+
                             |
                         HTTPS POST
                             |
                             v
                   +-------------------+
                   | Python FastAPI    |
                   |                   |
                   | 1. Xác thực thiết bị
                   | 2. Kiểm tra ảnh
                   | 3. Xử lý ảnh trong RAM
                   | 4. Nhận diện khuôn mặt
                   | 5. Quyết định điểm danh
                   +-------------------+
                       |           |
                       |           |
                       v           v
              +---------------+  +----------------+
              | Supabase      |  | Temporary      |
              | PostgreSQL    |  | WebP Storage   |
              |               |  |                |
              | Attendance    |  | Tối đa 24 giờ  |
              | People        |  | Nếu cần lưu    |
              | Sessions      |  +----------------+
              | Devices       |
              +---------------+
                       ^
                       |
                 Server-side API
                       |
                       v
              +-------------------+
              | Next.js Web       |
              | Admin Dashboard   |
              +-------------------+
```

## 3.2. Nguyên tắc kiến trúc

- Backend là nơi duy nhất quyết định một yêu cầu có đủ điều kiện tạo bản ghi điểm danh hay không.
- Trình duyệt không trực tiếp ghi kết quả điểm danh vào cơ sở dữ liệu.
- ESP32-CAM không tự quyết định danh tính người được điểm danh.
- Không đưa khóa bí mật của Supabase hoặc token thiết bị vào mã JavaScript phía trình duyệt.
- API, database và giao diện quản trị được triển khai độc lập về mặt trách nhiệm, nhưng có thể nằm trong cùng một repository.
- Việc triển khai frontend và backend trên cùng hay khác máy chủ không làm thay đổi các quy tắc nghiệp vụ.

---

# 4. Công nghệ dự kiến

| Thành phần | Công nghệ | Trách nhiệm |
|---|---|---|
| Frontend | Next.js, TypeScript | Giao diện quản trị |
| UI | React, CSS/Tailwind CSS | Hiển thị và tương tác |
| Backend | Python, FastAPI | API và nghiệp vụ |
| Xử lý ảnh | Pillow, OpenCV khi cần | Đọc, chuẩn hóa ảnh |
| Nhận diện | Thư viện/mô hình nhận diện khuôn mặt phù hợp | Trích xuất và so sánh embedding |
| Database | Supabase PostgreSQL | Lưu dữ liệu nghiệp vụ |
| Lưu ảnh tạm | Object storage hoặc vùng lưu tạm phù hợp | Ảnh WebP có thời hạn |
| Camera | ESP32-CAM + OV2640 | Chụp ảnh |
| Giao tiếp | HTTPS, REST, JSON, multipart/form-data | Trao đổi dữ liệu |
| Quản lý mã nguồn | Git | Theo dõi thay đổi |

**Lưu ý:** Chưa chốt thư viện AI. Cần đánh giá độ chính xác, hiệu năng, điều kiện cấp phép, khả năng triển khai trên cloud và khả năng tương thích của mô hình trước khi lựa chọn.

---

# 5. Yêu cầu chức năng

## FR-01: Xác thực thiết bị

Mỗi thiết bị phải được xác thực trước khi gửi ảnh.

Điều kiện:

- Yêu cầu gửi kèm thông tin xác thực.
- Backend kiểm tra thiết bị có tồn tại và đang hoạt động.
- Thiết bị không hợp lệ nhận HTTP 401 hoặc 403.
- Thiết bị đã bị vô hiệu hóa không được phép điểm danh.
- Mỗi thiết bị có thể được thu hồi hoặc thay mới thông tin xác thực độc lập.

Không dùng chung một token cố định cho tất cả thiết bị trong môi trường thực tế.

## FR-02: Tiếp nhận ảnh

API tiếp nhận ảnh từ camera bằng phương thức POST.

Yêu cầu:

- Chỉ nhận định dạng đầu vào được hỗ trợ; MVP sử dụng JPEG.
- Kích thước file tối đa mặc định: 5 MB.
- Phải kiểm tra định dạng thực tế của dữ liệu, không chỉ tin vào HTTP Content-Type.
- Từ chối ảnh rỗng, ảnh lỗi hoặc ảnh có độ phân giải vượt giới hạn.
- Có giới hạn thời gian xử lý và kích thước request.
- Không tin tưởng tên file do client gửi.
- Không ghi ảnh JPEG gốc xuống ổ đĩa nếu không cần thiết.

Mã phản hồi dự kiến:

- `201 Created`: đã tiếp nhận thành công trong chế độ kiểm thử.
- `400 Bad Request`: dữ liệu ảnh không hợp lệ.
- `401 Unauthorized`: xác thực thiết bị thất bại.
- `403 Forbidden`: thiết bị không được phép hoạt động.
- `413 Payload Too Large`: vượt giới hạn dữ liệu.
- `415 Unsupported Media Type`: định dạng không được hỗ trợ.
- `429 Too Many Requests`: gửi quá nhiều yêu cầu.
- `500 Internal Server Error`: lỗi xử lý nội bộ.

## FR-03: Xử lý ảnh

Backend phải:

1. Giải mã ảnh trong bộ nhớ.
2. Kiểm tra kích thước, số pixel và khả năng giải mã.
3. Chuẩn hóa màu sắc và hướng ảnh khi cần.
4. Xác định ảnh có đủ chất lượng để nhận diện hay không.
5. Đưa ảnh vào mô hình nhận diện.
6. Giải phóng tài nguyên sau khi xử lý.

Các giới hạn như độ phân giải tối đa, thời gian xử lý và kích thước ảnh phải được cấu hình để tránh tiêu thụ quá nhiều RAM.

## FR-04: Chuyển đổi và lưu ảnh WebP

JPEG là định dạng truyền từ camera trong MVP. WebP được dùng cho ảnh tạm khi cần lưu.

Quy tắc:

- Không bắt buộc lưu mọi ảnh được gửi lên.
- Chỉ tạo bản WebP khi có nhu cầu kiểm thử, điều tra lỗi hoặc được chính sách cho phép.
- Chuyển đổi ảnh trong RAM.
- Có thể giảm kích thước ảnh trước khi lưu.
- Chất lượng WebP mặc định dự kiến: 80; có thể điều chỉnh sau khi kiểm thử.
- Không lưu ảnh WebP lâu dài trong database dạng nhị phân.
- Không dùng thư mục tạm của container làm kho lưu trữ lâu dài.
- Mỗi ảnh phải có thời điểm hết hạn hoặc cơ chế xác định thời điểm hết hạn.

## FR-05: Tự động xóa ảnh tạm

Thời hạn lưu ảnh tạm tối đa: 24 giờ kể từ thời điểm tạo ảnh.

Yêu cầu:

- Ảnh hết hạn phải được xóa tự động.
- Tác vụ xóa phải chạy độc lập với việc có camera gửi ảnh mới hay không.
- Việc server khởi động lại không được làm mất lịch chạy dọn dẹp một cách âm thầm.
- Nếu lưu trên object storage, phải có tác vụ xóa hoặc lifecycle rule phù hợp.
- Có log số lượng ảnh được xóa và lỗi xóa.
- Không ghi nội dung ảnh hoặc embedding vào log ứng dụng.
- Nếu không thể đảm bảo xóa đúng hạn, không được tuyên bố rằng hệ thống đã đảm bảo thời hạn 24 giờ.

Thời hạn này áp dụng cho ảnh tạm do hệ thống quản lý. Bản sao trong backup hoặc log của nhà cung cấp phải được kiểm tra riêng theo chính sách lưu trữ của nền tảng.

## FR-06: Đăng ký người tham gia

Admin có thể tạo hồ sơ người tham gia.

Thông tin cơ bản:

- ID người tham gia.
- Mã định danh nội bộ.
- Họ và tên.
- Trạng thái hoạt động.
- Thời điểm tạo.
- Thời điểm cập nhật.

Dữ liệu khuôn mặt:

- Được tạo từ quy trình đăng ký có sự đồng ý phù hợp.
- Có thể lưu embedding thay vì lưu ảnh gốc lâu dài.
- Phải được bảo vệ như dữ liệu sinh trắc học nhạy cảm.
- Có thể cập nhật hoặc xóa theo quy trình.
- Không được sử dụng dữ liệu khuôn mặt ngoài mục đích đã thông báo.

## FR-07: Nhận diện khuôn mặt

Backend sử dụng mô hình nhận diện đã được lựa chọn để so sánh khuôn mặt từ ảnh mới với dữ liệu đăng ký.

Kết quả nhận diện có thể gồm:

- `MATCHED`: tìm được danh tính phù hợp.
- `UNKNOWN`: không tìm thấy danh tính phù hợp.
- `NO_FACE`: không phát hiện khuôn mặt.
- `MULTIPLE_FACES`: phát hiện nhiều khuôn mặt khi chế độ hiện tại chỉ hỗ trợ một người.
- `LOW_QUALITY`: ảnh không đủ chất lượng.
- `PROCESSING_ERROR`: xảy ra lỗi xử lý.

Ngưỡng so khớp phải được cấu hình và kiểm thử trên dữ liệu phù hợp. Không mặc định rằng một ngưỡng duy nhất có thể phù hợp với mọi camera, ánh sáng và nhóm người.

Nếu độ tin cậy không đạt ngưỡng, hệ thống phải từ chối tự động điểm danh và trả về trạng thái phù hợp.

## FR-08: Quản lý phiên điểm danh

Admin có thể tạo phiên điểm danh với các thông tin:

- Tên phiên.
- Thời gian bắt đầu.
- Thời gian kết thúc.
- Địa điểm hoặc nhóm đối tượng.
- Trạng thái phiên.
- Quy tắc cho phép điểm danh.

Trạng thái phiên:

- `DRAFT`: bản nháp.
- `SCHEDULED`: đã lên lịch.
- `ACTIVE`: đang mở.
- `CLOSED`: đã đóng.
- `CANCELLED`: đã hủy.

Backend phải kiểm tra thời gian và trạng thái phiên trước khi ghi nhận điểm danh.

Không cho phép điểm danh vào phiên đã đóng hoặc đã hủy, trừ khi có nghiệp vụ điều chỉnh thủ công riêng.

## FR-09: Ghi nhận điểm danh

Một bản ghi điểm danh hợp lệ cần đáp ứng:

1. Thiết bị đã được xác thực.
2. Thiết bị đang hoạt động.
3. Ảnh hợp lệ.
4. Nhận diện đạt ngưỡng đã cấu hình.
5. Người tham gia đang hoạt động.
6. Có phiên điểm danh phù hợp.
7. Phiên đang cho phép điểm danh.
8. Chưa có bản ghi cho cùng người tham gia trong cùng phiên, hoặc nghiệp vụ cho phép cập nhật theo quy tắc được xác định trước.

Khi hợp lệ, backend ghi một bản ghi vào database.

Mỗi bản ghi nên có:

- ID bản ghi.
- ID người tham gia.
- ID phiên điểm danh.
- ID thiết bị.
- Thời điểm ghi nhận.
- Phương thức ghi nhận.
- Trạng thái.
- Thông tin kỹ thuật tối thiểu cần thiết để kiểm tra lỗi, nếu được phép.

Không lưu ảnh khuôn mặt trong bản ghi điểm danh.

## FR-10: Chống điểm danh trùng

Hệ thống phải ngăn một người tạo nhiều bản ghi điểm danh trong cùng một phiên.

Cơ chế bắt buộc:

- Có unique constraint ở database trên cặp `(session_id, person_id)`.
- Không chỉ dựa vào kiểm tra trước khi insert ở Python, vì nhiều request có thể đến đồng thời.
- Nếu nhận diện lại cùng người trong phiên, trả về trạng thái `ALREADY_CHECKED_IN`.
- Không tạo thêm bản ghi trùng.
- Nếu cần lưu lịch sử nhiều lần phát hiện, lưu ở bảng sự kiện riêng và không xem mỗi sự kiện là một lần điểm danh mới.

## FR-11: Dashboard quản trị

Dashboard hiển thị tối thiểu:

- Tổng số người tham gia đang hoạt động.
- Tổng số phiên điểm danh.
- Số lượt điểm danh trong phiên hiện tại.
- Danh sách người đã điểm danh.
- Danh sách người chưa điểm danh trong phiên.
- Thời điểm điểm danh gần nhất.
- Trạng thái thiết bị.
- Thống kê lỗi nhận diện hoặc lỗi thiết bị ở mức tổng hợp.

Không hiển thị ảnh khuôn mặt hoặc embedding mặc định trên dashboard.

## FR-12: Quản lý lịch sử

Admin có thể:

- Lọc theo phiên.
- Lọc theo người tham gia.
- Lọc theo ngày.
- Lọc theo thiết bị.
- Xem chi tiết bản ghi.
- Xuất báo cáo CSV khi được cấp quyền.
- Điều chỉnh bản ghi sai theo quy trình kiểm toán.

Mọi chỉnh sửa hoặc hủy bản ghi phải lưu người thực hiện, thời gian và lý do.

## FR-13: Quản lý thiết bị

Admin có thể:

- Tạo thiết bị.
- Đặt tên và địa điểm.
- Xem trạng thái.
- Xem thời điểm liên lạc gần nhất.
- Vô hiệu hóa thiết bị.
- Thu hồi hoặc thay token.
- Xem lỗi kết nối và lỗi gửi ảnh.

Token chỉ được hiển thị theo quy trình cấp phát an toàn và không được trả lại trong các truy vấn thông thường.

## FR-14: Xử lý lỗi và phản hồi

Mọi yêu cầu gửi ảnh phải trả về kết quả có cấu trúc rõ ràng.

Ví dụ:

```json
{
  "success": true,
  "status": "CHECKED_IN",
  "person_id": "uuid",
  "session_id": "uuid",
  "checked_in_at": "2026-10-13T10:00:00Z"
}
```

Đây là định dạng minh họa. API thật phải xác định rõ trường bắt buộc và không được trả thông tin cá nhân cho thiết bị công khai nếu không cần thiết.

Các trạng thái nghiệp vụ nên gồm:

- `CHECKED_IN`
- `ALREADY_CHECKED_IN`
- `UNKNOWN_PERSON`
- `NO_FACE`
- `MULTIPLE_FACES`
- `LOW_QUALITY`
- `NO_ACTIVE_SESSION`
- `DEVICE_DISABLED`
- `PROCESSING_ERROR`

---

# 6. Yêu cầu phi chức năng

## NFR-01: Hiệu năng

Mục tiêu ban đầu để kiểm thử:

- API health phản hồi trong vòng 1 giây trong điều kiện tải nhẹ.
- Một yêu cầu xử lý ảnh có mục tiêu hoàn thành trong vòng 3 giây ở điều kiện bình thường.
- Hệ thống không tải toàn bộ ảnh hoặc embedding của tất cả người tham gia vào RAM nếu dữ liệu tăng lớn.
- Có thể cấu hình số tác vụ xử lý đồng thời.
- Có timeout cho kết nối và xử lý ảnh.

Các mục tiêu trên là tiêu chí kiểm thử ban đầu, không phải cam kết hiệu năng trước khi đo thực tế trên cloud và mô hình AI được lựa chọn.

## NFR-02: Tối ưu chi phí cloud

- Chỉ sử dụng độ phân giải cần thiết cho nhận diện.
- Không lưu JPEG gốc lâu dài.
- Chỉ lưu WebP tạm khi cần.
- Không dùng database để lưu nhị phân ảnh.
- Giới hạn kích thước request.
- Giới hạn tốc độ gửi ảnh.
- Giảm số lần ghi database không cần thiết.
- Không ghi nhận điểm danh lặp lại trong cùng phiên.
- Theo dõi CPU, RAM, dung lượng lưu trữ và số request.
- Không giữ tác vụ xử lý AI chạy liên tục khi không có yêu cầu nếu nền tảng không phù hợp.

## NFR-03: Bảo mật

- Tất cả kết nối qua Internet phải dùng HTTPS.
- Không sử dụng chế độ bỏ qua xác minh chứng chỉ TLS trong môi trường thật.
- Secret chỉ được lưu trong biến môi trường hoặc secret manager.
- Không commit `.env`, token, khóa service role hoặc dữ liệu khuôn mặt.
- Phân quyền admin phía server.
- Bật và kiểm thử RLS cho các bảng Supabase phù hợp.
- Xác thực và phân quyền mọi API nhạy cảm.
- Giới hạn request và chống gửi lặp.
- Không ghi token, ảnh, embedding hoặc thông tin bí mật vào log.
- Có quy trình thay token khi thiết bị bị mất hoặc bị xâm nhập.

## NFR-04: Quyền riêng tư

- Thông báo rõ mục đích thu thập và sử dụng dữ liệu khuôn mặt.
- Có sự đồng ý phù hợp trước khi đăng ký và sử dụng dữ liệu sinh trắc học.
- Chỉ thu thập dữ liệu cần thiết.
- Quy định thời hạn lưu ảnh, embedding và bản ghi điểm danh riêng biệt.
- Có quy trình xem, sửa và xóa dữ liệu theo quyền hợp lệ.
- Giới hạn quyền truy cập dữ liệu sinh trắc học.
- Không sử dụng dữ liệu cho mục đích khác ngoài phạm vi đã thông báo.
- Có phương án điểm danh thay thế khi nhận diện thất bại.
- Kiểm tra nghĩa vụ pháp lý áp dụng cho việc xử lý dữ liệu cá nhân tại nơi triển khai.

## NFR-05: Độ tin cậy

- Database là nguồn dữ liệu chính thức của kết quả điểm danh.
- Không xác nhận thành công trước khi database xác nhận ghi nhận hợp lệ.
- Có unique constraint chống trùng.
- Có log lỗi đủ để chẩn đoán nhưng không chứa dữ liệu nhạy cảm.
- Có kiểm tra sức khỏe API.
- Có cơ chế khởi động lại và giám sát tác vụ dọn dẹp.
- Có quy trình backup và khôi phục dữ liệu nghiệp vụ.

## NFR-06: Khả năng bảo trì

- Frontend và backend có cấu trúc thư mục rõ ràng.
- Cấu hình tách khỏi mã nguồn.
- API có tài liệu OpenAPI.
- Có kiểm thử đơn vị cho nghiệp vụ quan trọng.
- Có migration cho thay đổi database.
- Có README hướng dẫn cài đặt và triển khai.
- Có log và mã lỗi thống nhất.
- Không trộn logic nhận diện, logic điểm danh và logic giao diện vào cùng một module.

---

# 7. Thiết kế cơ sở dữ liệu

Tên bảng và trường dưới đây là thiết kế đề xuất. Có thể điều chỉnh khi chốt mô hình dữ liệu.

## 7.1. Bảng `people`

Lưu hồ sơ người tham gia.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `external_code` | TEXT | Mã người tham gia |
| `full_name` | TEXT | Họ tên |
| `is_active` | BOOLEAN | Trạng thái hoạt động |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |
| `updated_at` | TIMESTAMPTZ | Thời điểm cập nhật |

`external_code` nên có unique constraint nếu được sử dụng làm mã định danh duy nhất.

## 7.2. Bảng `face_profiles`

Lưu dữ liệu phục vụ nhận diện.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `person_id` | UUID | Tham chiếu `people.id` |
| `embedding` | Dữ liệu số phù hợp | Vector đặc trưng khuôn mặt |
| `model_version` | TEXT | Phiên bản mô hình |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |
| `is_active` | BOOLEAN | Có được sử dụng hay không |

Lưu ý:

- Cần chọn kiểu lưu embedding phù hợp với mô hình và phương pháp tìm kiếm.
- Nếu dùng PostgreSQL, có thể đánh giá extension `pgvector` khi số lượng embedding tăng.
- Embedding là dữ liệu sinh trắc học nhạy cảm và phải được bảo vệ.
- Không lưu ảnh gốc trong bảng này mặc định.
- Quy trình cập nhật hoặc xóa hồ sơ phải đồng bộ với dữ liệu embedding.

## 7.3. Bảng `attendance_sessions`

Lưu phiên điểm danh.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `name` | TEXT | Tên phiên |
| `starts_at` | TIMESTAMPTZ | Bắt đầu |
| `ends_at` | TIMESTAMPTZ | Kết thúc |
| `status` | TEXT | Trạng thái |
| `created_by` | UUID | Admin tạo phiên |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |

Backend phải xác minh trạng thái và thời gian phiên trước khi điểm danh.

## 7.4. Bảng `devices`

Lưu thông tin thiết bị.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `name` | TEXT | Tên thiết bị |
| `location` | TEXT | Địa điểm |
| `is_active` | BOOLEAN | Trạng thái |
| `last_seen_at` | TIMESTAMPTZ | Liên lạc gần nhất |
| `firmware_version` | TEXT | Phiên bản firmware |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |

Token hoặc hash token phải được thiết kế theo cơ chế xác thực phù hợp; không lưu token nguyên văn trong trường dữ liệu có thể đọc tùy ý.

## 7.5. Bảng `attendance_records`

Lưu kết quả điểm danh.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `person_id` | UUID | Người tham gia |
| `session_id` | UUID | Phiên điểm danh |
| `device_id` | UUID | Thiết bị ghi nhận |
| `checked_in_at` | TIMESTAMPTZ | Thời điểm điểm danh |
| `method` | TEXT | Phương thức ghi nhận |
| `status` | TEXT | Trạng thái bản ghi |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |

Ràng buộc bắt buộc:

```sql
UNIQUE (session_id, person_id)
```

Cần thiết lập foreign key tới các bảng tương ứng và quy định hành vi khi hồ sơ bị vô hiệu hóa hoặc xóa.

## 7.6. Bảng `temporary_image_metadata`

Chỉ cần bảng này nếu lưu ảnh tạm trên object storage.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `storage_key` | TEXT | Đường dẫn đối tượng lưu trữ |
| `format` | TEXT | Phải là WebP |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |
| `expires_at` | TIMESTAMPTZ | Thời điểm hết hạn |
| `deleted_at` | TIMESTAMPTZ | Thời điểm xác nhận xóa, nếu cần |

Không lưu ảnh nhị phân trong bảng metadata. Không đưa đường dẫn nội bộ hoặc URL truy cập công khai vào API không cần thiết.

---

# 8. Đặc tả API

Các endpoint dưới đây là đề xuất cho MVP.

## 8.1. Kiểm tra sức khỏe

`GET /api/health`

Mục đích:

- Kiểm tra API hoạt động.
- Phục vụ health check của nền tảng triển khai.

Ví dụ:

```json
{
  "status": "ok",
  "service": "facecheck-api"
}
```

Endpoint này không trả token, thông tin database hoặc dữ liệu nội bộ.

## 8.2. Gửi ảnh từ ESP32-CAM

`POST /api/camera/upload`

Header:

```text
X-Device-Token: <device-token>
Content-Type: multipart/form-data
```

Trường multipart:

- `image`: file JPEG.

Nhiệm vụ:

1. Xác thực thiết bị.
2. Kiểm tra trạng thái thiết bị.
3. Kiểm tra request và ảnh.
4. Xử lý ảnh trong RAM.
5. Chạy nhận diện nếu chức năng đã được bật.
6. Kiểm tra phiên điểm danh.
7. Ghi nhận kết quả nếu hợp lệ.
8. Chỉ lưu WebP tạm nếu được cấu hình.
9. Trả kết quả nghiệp vụ.

API thật không nên mặc định trả `201 Created` chỉ vì nhận được ảnh. Phải phân biệt rõ nhận ảnh thành công và điểm danh thành công.

## 8.3. Danh sách người tham gia

`GET /api/admin/people`

Yêu cầu:

- Admin đã đăng nhập.
- Có phân trang.
- Có thể tìm theo mã hoặc tên.
- Không trả embedding hoặc ảnh khuôn mặt.

## 8.4. Tạo người tham gia

`POST /api/admin/people`

Yêu cầu:

- Admin đã đăng nhập.
- Kiểm tra dữ liệu đầu vào.
- Kiểm tra mã người tham gia trùng.
- Ghi nhận nhật ký thao tác khi cần.

## 8.5. Quản lý phiên điểm danh

- `GET /api/admin/sessions`
- `POST /api/admin/sessions`
- `PATCH /api/admin/sessions/{id}`
- `GET /api/admin/sessions/{id}/attendance`

Tất cả endpoint quản trị phải kiểm tra quyền phía server.

## 8.6. Lịch sử điểm danh

`GET /api/admin/attendance`

Bộ lọc dự kiến:

- `session_id`
- `person_id`
- `device_id`
- `from`
- `to`
- `page`
- `limit`

Phải giới hạn kích thước trang và xác thực quyền truy cập.

## 8.7. Quy tắc API chung

- Dùng JSON cho request/response thông thường.
- Dùng multipart/form-data cho ảnh.
- Có cấu trúc lỗi thống nhất.
- Có timeout và giới hạn request.
- Có phân trang cho danh sách lớn.
- Không trả stack trace cho client.
- Không để client tự chỉ định danh tính được điểm danh.
- Không tin tưởng `person_id` do thiết bị gửi lên; danh tính phải do backend xác định.

---

# 9. Quy trình nghiệp vụ

## 9.1. Quy trình đăng ký người tham gia

1. Admin đăng nhập.
2. Tạo hồ sơ người tham gia.
3. Thông báo mục đích và xin sự đồng ý phù hợp.
4. Thu thập ảnh đăng ký theo quy trình được thiết kế.
5. Kiểm tra chất lượng ảnh.
6. Trích xuất embedding.
7. Lưu embedding và phiên bản mô hình.
8. Kiểm tra khả năng nhận diện.
9. Kích hoạt hồ sơ khi hoàn tất.

Nếu đăng ký thất bại, không được để lại hồ sơ khuôn mặt ở trạng thái hoạt động một phần.

## 9.2. Quy trình điểm danh

1. ESP32-CAM kết nối Wi-Fi.
2. Camera chụp JPEG.
3. Thiết bị gửi ảnh qua HTTPS.
4. Backend xác thực thiết bị.
5. Backend kiểm tra ảnh.
6. Backend phát hiện khuôn mặt.
7. Backend kiểm tra chất lượng.
8. Backend so sánh với dữ liệu đăng ký.
9. Backend xác định phiên điểm danh phù hợp.
10. Backend kiểm tra trùng.
11. Backend ghi nhận điểm danh bằng thao tác database an toàn.
12. Backend trả trạng thái.
13. Thiết bị hoặc dashboard hiển thị phản hồi phù hợp.

Nếu bất kỳ bước bắt buộc nào thất bại, không tạo bản ghi điểm danh thành công.

## 9.3. Quy trình xử lý ảnh tạm

1. Backend nhận JPEG.
2. Ảnh được xử lý trong RAM.
3. Nếu không cần lưu ảnh, giải phóng dữ liệu ảnh sau xử lý.
4. Nếu cần lưu kiểm thử, mã hóa sang WebP.
5. Ghi file vào vùng lưu trữ tạm.
6. Ghi metadata và thời điểm hết hạn nếu dùng object storage.
7. Tác vụ dọn dẹp tìm các ảnh đã hết hạn.
8. Xóa đối tượng ảnh.
9. Ghi nhận kết quả dọn dẹp.
10. Nếu xóa thất bại, thử lại và cảnh báo khi vượt ngưỡng lỗi.

---

# 10. Yêu cầu về giao diện

## 10.1. Trang quản trị `/admin`

Các trang chính:

- `/admin`: Dashboard.
- `/admin/people`: Quản lý người tham gia.
- `/admin/people/[id]`: Chi tiết người tham gia.
- `/admin/sessions`: Quản lý phiên điểm danh.
- `/admin/sessions/[id]`: Chi tiết phiên và danh sách điểm danh.
- `/admin/attendance`: Lịch sử điểm danh.
- `/admin/devices`: Quản lý thiết bị.
- `/admin/settings`: Cấu hình và chính sách lưu trữ.

Việc ẩn hoặc không hiển thị liên kết admin trên giao diện công khai không phải cơ chế bảo mật. Mọi route và API quản trị phải yêu cầu xác thực và kiểm tra quyền.

## 10.2. Trạng thái giao diện

Các màn hình cần xử lý:

- Loading.
- Thành công.
- Không có dữ liệu.
- Lỗi kết nối.
- Không đủ quyền.
- Dữ liệu không hợp lệ.
- Phiên điểm danh đã đóng.
- Thiết bị ngoại tuyến.
- Người chưa đăng ký.
- Điểm danh trùng.

## 10.3. Khả năng sử dụng

- Giao diện tiếng Việt.
- Tương thích máy tính và điện thoại.
- Có thông báo lỗi dễ hiểu.
- Có xác nhận trước thao tác xóa hoặc vô hiệu hóa.
- Có phân trang và bộ lọc khi dữ liệu lớn.
- Không phụ thuộc vào ảnh để truyền đạt trạng thái điểm danh.

---

# 11. Triển khai và cấu hình môi trường

## 11.1. Cấu trúc repository

```text
FaceCheck/
├── frontend/
│   ├── app/
│   ├── components/
│   ├── lib/
│   ├── public/
│   ├── package.json
│   └── .env.example
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   ├── services/
│   │   ├── models/
│   │   ├── repositories/
│   │   └── core/
│   ├── tests/
│   ├── requirements.txt
│   └── .env.example
│
├── firmware/
│   └── esp32-camera/
│       └── esp32-camera.ino
│
├── supabase/
│   └── migrations/
│
├── docs/
│   ├── architecture.md
│   ├── api.md
│   └── privacy.md
│
├── .gitignore
└── README.md
```

## 11.2. Biến môi trường backend

Ví dụ `.env.example`:

```dotenv
APP_ENV=development
DEVICE_API_TOKEN=
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
UPLOAD_DIR=
MAX_UPLOAD_BYTES=5242880
IMAGE_RETENTION_HOURS=24
TEMP_IMAGE_STORAGE_ENABLED=false
```

Quy tắc:

- `.env.example` chỉ chứa tên biến và giá trị mẫu không bí mật.
- `.env` thật không được commit.
- Khóa `SUPABASE_SERVICE_ROLE_KEY` chỉ được sử dụng ở backend đáng tin cậy.
- Không đưa khóa service role vào Next.js client-side code.
- Trong môi trường thực tế nên dùng secret manager của nền tảng.
- Token thiết bị phải được cấp phát riêng và có thể thu hồi.

## 11.3. Biến môi trường frontend

Ví dụ:

```dotenv
NEXT_PUBLIC_APP_NAME=FaceCheck
NEXT_PUBLIC_API_BASE_URL=
```

Chỉ các giá trị thực sự công khai mới được dùng tiền tố `NEXT_PUBLIC_`.

## 11.4. Quy tắc lưu trữ

- Database lưu dữ liệu nghiệp vụ.
- Object storage lưu ảnh WebP tạm nếu được bật.
- Không dùng filesystem tạm của cloud làm kho lưu trữ bền vững.
- Nếu lưu ảnh tạm, phải có chính sách xóa độc lập với vòng đời API.
- Phải kiểm thử cơ chế xóa trên môi trường triển khai thật.

---

# 12. Kiểm thử và tiêu chí nghiệm thu

## 12.1. Kiểm thử tiếp nhận ảnh

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-01 | Gửi JPEG hợp lệ và token hợp lệ | API tiếp nhận thành công |
| T-02 | Thiếu token | HTTP 401 |
| T-03 | Token sai | HTTP 401 |
| T-04 | Thiết bị bị vô hiệu hóa | HTTP 403 |
| T-05 | File rỗng | HTTP 400 |
| T-06 | File vượt giới hạn | HTTP 413 |
| T-07 | Định dạng không hỗ trợ | HTTP 415 |
| T-08 | JPEG bị hỏng | HTTP 400 |
| T-09 | Độ phân giải vượt giới hạn | HTTP 413 |
| T-10 | Server không có cấu hình bắt buộc | HTTP 503 hoặc lỗi cấu hình tương ứng |

## 12.2. Kiểm thử nhận diện

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-11 | Người đã đăng ký, ảnh đạt yêu cầu | Nhận diện đúng trong điều kiện kiểm thử |
| T-12 | Người chưa đăng ký | `UNKNOWN_PERSON` |
| T-13 | Không có khuôn mặt | `NO_FACE` |
| T-14 | Nhiều khuôn mặt khi chỉ hỗ trợ một người | `MULTIPLE_FACES` |
| T-15 | Ảnh quá mờ hoặc quá tối | `LOW_QUALITY` hoặc trạng thái lỗi chất lượng tương ứng |
| T-16 | Lỗi mô hình AI | Không ghi nhận điểm danh thành công |
| T-17 | Ngưỡng nhận diện không đạt | Không tự động điểm danh |

Cần đo tỷ lệ nhận diện sai, từ chối sai và ảnh hưởng của ánh sáng, góc mặt, camera, khoảng cách. Không đánh giá chất lượng mô hình chỉ bằng một vài ảnh thử nghiệm.

## 12.3. Kiểm thử nghiệp vụ điểm danh

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-18 | Người hợp lệ trong phiên đang mở | Tạo bản ghi |
| T-19 | Người đã điểm danh trong cùng phiên | Không tạo bản ghi trùng |
| T-20 | Phiên đã đóng | Không điểm danh |
| T-21 | Không có phiên phù hợp | `NO_ACTIVE_SESSION` |
| T-22 | Hồ sơ người tham gia bị vô hiệu hóa | Không điểm danh |
| T-23 | Hai request đồng thời cho cùng người và phiên | Tối đa một bản ghi |
| T-24 | Database lỗi | Không trả thành công giả |
| T-25 | Thiết bị bị thu hồi | Không ghi nhận |

## 12.4. Kiểm thử lưu trữ

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-26 | Bật lưu ảnh tạm | Chỉ tạo WebP |
| T-27 | Tắt lưu ảnh tạm | Không lưu ảnh xuống kho tạm |
| T-28 | Ảnh chưa hết hạn | Không bị xóa |
| T-29 | Ảnh đã hết hạn | Được xóa theo chính sách |
| T-30 | Tác vụ dọn dẹp gặp lỗi | Có log và cơ chế thử lại |
| T-31 | API restart | Cơ chế dọn dẹp tiếp tục hoạt động |
| T-32 | Ảnh hết hạn nhưng database còn metadata | Metadata được xử lý theo chính sách, không được xem là ảnh còn tồn tại |
| T-33 | Lưu trữ có backup riêng | Kiểm tra thời hạn tồn tại của bản sao |

## 12.5. Tiêu chí nghiệm thu MVP

MVP được coi là đạt khi:

- ESP32-CAM gửi ảnh thành công qua HTTPS.
- Backend từ chối request không hợp lệ.
- Nhận diện hoạt động trên bộ dữ liệu thử nghiệm đã xác định.
- Điểm danh chỉ được tạo khi mọi điều kiện nghiệp vụ được thỏa mãn.
- Không tạo bản ghi trùng trong cùng phiên, kể cả khi có request đồng thời.
- Dashboard hiển thị đúng dữ liệu từ database.
- Không cần lưu ảnh lâu dài để duy trì lịch sử điểm danh.
- Ảnh tạm chỉ được lưu dưới dạng WebP khi chức năng lưu tạm được bật.
- Có cơ chế xóa ảnh theo thời hạn và kiểm thử được trên môi trường cloud.
- Không có khóa bí mật trong repository hoặc frontend bundle.
- Có xác thực và phân quyền cho toàn bộ chức năng quản trị.
- Có tài liệu cài đặt, triển khai và khôi phục lỗi cơ bản.

---

# 13. Kế hoạch phát triển theo giai đoạn

## Phase 1 — Nền tảng API

- Tạo repository.
- Tạo Python FastAPI.
- Tạo endpoint health.
- Xây dựng API tiếp nhận JPEG.
- Xác thực token thiết bị.
- Kiểm tra file và giới hạn request.
- Kiểm thử upload bằng `curl.exe` trước khi mua phần cứng.

**Điều kiện hoàn thành:** Có thể gửi ảnh JPEG từ máy tính tới API và nhận phản hồi chính xác.

## Phase 2 — WebP và quản lý vòng đời ảnh

- Tích hợp Pillow.
- Chuyển JPEG sang WebP trong RAM.
- Bật/tắt lưu ảnh tạm bằng cấu hình.
- Thiết lập thời hạn 24 giờ.
- Xây dựng tác vụ xóa độc lập.
- Kiểm thử trên môi trường cloud.

**Điều kiện hoàn thành:** Ảnh không bị lưu ngoài dự kiến; ảnh tạm được dọn dẹp theo chính sách.

## Phase 3 — Cơ sở dữ liệu

- Thiết kế và tạo migration Supabase.
- Tạo bảng người tham gia, phiên điểm danh và thiết bị.
- Tạo bảng bản ghi điểm danh.
- Thiết lập foreign key, unique constraint và RLS.
- Kiểm thử quyền truy cập.

**Điều kiện hoàn thành:** Dữ liệu nghiệp vụ được lưu và truy vấn đúng.

## Phase 4 — Nhận diện khuôn mặt

- Chọn thư viện và mô hình.
- Xây dựng quy trình đăng ký.
- Trích xuất embedding.
- Xây dựng so khớp.
- Xác định ngưỡng.
- Đánh giá false accept và false reject.
- Kiểm thử chất lượng ảnh.

**Điều kiện hoàn thành:** Mô hình đạt các tiêu chí đã xác định trên bộ kiểm thử đại diện.

## Phase 5 — ESP32-CAM

- Mua và cấu hình phần cứng.
- Kết nối Wi-Fi.
- Chụp JPEG.
- Gửi ảnh qua HTTPS.
- Kiểm thử reconnect, timeout và lỗi mạng.
- Kiểm thử tài nguyên RAM và nguồn cấp.

**Điều kiện hoàn thành:** Camera gửi ảnh ổn định đến cloud trong điều kiện mạng mục tiêu.

## Phase 6 — Dashboard Next.js

- Xây dựng xác thực admin.
- Dashboard.
- Quản lý người tham gia.
- Quản lý phiên.
- Quản lý thiết bị.
- Lịch sử và báo cáo.
- Xử lý lỗi và phân quyền.

**Điều kiện hoàn thành:** Admin có thể vận hành nghiệp vụ từ giao diện web.

## Phase 7 — Bảo mật và triển khai thực tế

- Xác minh TLS.
- Rà soát secrets.
- Kiểm thử quyền truy cập.
- Kiểm thử chống request lặp.
- Kiểm thử xóa ảnh và retention.
- Kiểm thử tải.
- Kiểm tra backup, giám sát và quy trình khôi phục.
- Hoàn thiện chính sách quyền riêng tư.

**Điều kiện hoàn thành:** Hệ thống đáp ứng yêu cầu bảo mật, vận hành và lưu trữ đã xác định.

---

# 14. Các quyết định kiến trúc đã chốt

1. Backend dùng Python FastAPI.
2. Frontend dùng Next.js.
3. ESP32-CAM gửi ảnh trực tiếp tới backend qua Internet.
4. Không cần điện thoại làm trung gian truyền ảnh.
5. Đầu vào camera sử dụng JPEG.
6. WebP là định dạng lưu ảnh tạm.
7. Ảnh gốc không được lưu lâu dài mặc định.
8. Ảnh tạm có thời hạn tối đa 24 giờ và phải có cơ chế xóa tự động.
9. Kết quả điểm danh được lưu trong Supabase.
10. Database là nguồn dữ liệu chính thức.
11. Backend quyết định danh tính và kết quả điểm danh.
12. Có unique constraint chống điểm danh trùng.
13. Trang quản trị chỉ dành cho admin đã xác thực.
14. Không lưu secrets trong Git.
15. Nhận diện khuôn mặt phải được kiểm thử trước khi dùng trong thực tế.

# 15. Các quyết định còn cần xác nhận

- [ ] Chỉ cho phép một khuôn mặt trong mỗi khung hình hay hỗ trợ nhiều người.
- [ ] Một người có thể đăng ký bao nhiêu mẫu khuôn mặt.
- [ ] Cách xác định phiên điểm danh đang hoạt động: tự động theo thời gian hay do admin mở.
- [ ] Có cần lưu ảnh WebP kiểm thử hay mặc định tắt hoàn toàn.
- [ ] Mô hình nhận diện và ngưỡng so khớp.
- [ ] Có yêu cầu chống giả mạo bằng ảnh in hoặc màn hình điện thoại hay không.
- [ ] Nhà cung cấp cloud cho backend.
- [ ] Phương án lưu embedding và kiểm soát quyền truy cập.
- [ ] Quy trình xóa hồ sơ và dữ liệu sinh trắc học.
- [ ] Thời hạn lưu bản ghi điểm danh và thời gian lưu backup.
- [ ] Phương án điểm danh thủ công khi AI hoặc camera lỗi.

# 16. Nguyên tắc phát triển

- Hoàn thành từng giai đoạn và kiểm thử trước khi chuyển sang giai đoạn kế tiếp.
- Không tích hợp AI trước khi API nhận ảnh hoạt động ổn định.
- Không mua phần cứng trước khi xác minh được quy trình API cơ bản, trừ khi cần phần cứng để đánh giá chất lượng ảnh.
- Không đưa toàn bộ nghiệp vụ vào một file Python.
- Không để frontend quyết định kết quả điểm danh.
- Không lưu ảnh chỉ vì có thể lưu.
- Không coi kết quả nhận diện khuôn mặt là bằng chứng tuyệt đối về danh tính.
- Ưu tiên bảo mật, quyền riêng tư, khả năng kiểm thử và chi phí vận hành.
- Mọi thay đổi quan trọng về kiến trúc phải được cập nhật trong tài liệu này.

---

**Kết thúc đặc tả FaceCheck v1.0**# FaceCheck — Đặc tả hệ thống điểm danh bằng nhận diện khuôn mặt

- **Tên dự án:** FaceCheck
- **Phiên bản đặc tả:** 1.0
- **Trạng thái:** Draft — đang trong giai đoạn thiết kế
- **Nền tảng:** Web Application + REST API + ESP32-CAM
- **Ngôn ngữ giao diện dự kiến:** Tiếng Việt
- **Mục tiêu:** Xây dựng hệ thống điểm danh bằng nhận diện khuôn mặt, có trang quản trị, lưu kết quả điểm danh trên cloud và tự động xóa ảnh kiểm thử sau tối đa 24 giờ.

---

# 1. Tổng quan dự án

## 1.1. Mục đích

FaceCheck là hệ thống điểm danh sử dụng camera ESP32-CAM để chụp ảnh, backend Python để xử lý và nhận diện khuôn mặt, cơ sở dữ liệu cloud để lưu kết quả, và Next.js để cung cấp giao diện quản trị.

Hệ thống được thiết kế theo hướng tiết kiệm tài nguyên:

- ESP32-CAM chỉ thực hiện nhiệm vụ chụp ảnh và gửi ảnh.
- Python backend xử lý ảnh, thực hiện nhận diện khuôn mặt và quyết định kết quả điểm danh.
- Next.js cung cấp giao diện quản trị và xem lịch sử.
- Supabase lưu dữ liệu nghiệp vụ, không bắt buộc lưu ảnh khuôn mặt.
- Ảnh tạm phục vụ kiểm thử được chuyển sang WebP và tự động xóa sau thời hạn quy định.
- Người dùng thông thường không cần đăng nhập tài khoản web để thực hiện điểm danh tại thiết bị.

## 1.2. Phạm vi phiên bản đầu tiên

Phiên bản MVP (Minimum Viable Product) tập trung vào:

1. Nhận ảnh từ ESP32-CAM qua mạng Internet.
2. Xác thực thiết bị gửi ảnh.
3. Kiểm tra tính hợp lệ của ảnh.
4. Chuyển đổi JPEG sang WebP khi cần lưu ảnh tạm.
5. Nhận diện khuôn mặt bằng Python.
6. Đối chiếu khuôn mặt với dữ liệu đã đăng ký.
7. Ghi nhận kết quả điểm danh vào Supabase.
8. Chống ghi nhận điểm danh trùng trong cùng một phiên.
9. Cung cấp trang quản trị để quản lý người được điểm danh, phiên điểm danh và lịch sử.
10. Tự động xóa ảnh tạm sau tối đa 24 giờ theo chính sách lưu trữ.
11. Ghi log kỹ thuật để hỗ trợ phát hiện và xử lý lỗi.

## 1.3. Ngoài phạm vi MVP

Các chức năng sau chưa thuộc phạm vi bắt buộc của phiên bản đầu tiên:

- Ứng dụng native dành riêng cho Android hoặc iOS.
- Điểm danh ngoại tuyến khi không có Internet.
- Nhận diện đồng thời nhiều khuôn mặt trong một khung hình.
- Chống giả mạo nâng cao bằng camera chiều sâu hoặc cảm biến chuyên dụng.
- Tự động chấm công, tính lương hoặc tính giờ làm.
- Đồng bộ dữ liệu với hệ thống quản lý trường học.
- Nhận diện danh tính chỉ từ một ảnh bất kỳ mà không có bước đăng ký trước.
- Huấn luyện mô hình AI riêng từ đầu.

---

# 2. Đối tượng sử dụng và phân quyền

## 2.1. Quản trị viên (Admin)

Admin là người được phép quản lý hệ thống.

Quyền hạn:

- Đăng nhập vào khu vực quản trị.
- Xem dashboard thống kê.
- Tạo, xem, chỉnh sửa và vô hiệu hóa hồ sơ người tham gia.
- Đăng ký dữ liệu khuôn mặt khi có sự đồng ý hợp lệ.
- Tạo và quản lý phiên điểm danh.
- Xem lịch sử điểm danh.
- Sửa hoặc hủy bản ghi sai theo quy trình được phân quyền.
- Quản lý thiết bị ESP32-CAM.
- Xem trạng thái hoạt động và lỗi thiết bị.
- Quản lý chính sách lưu trữ ảnh tạm.
- Yêu cầu xóa dữ liệu theo chính sách.
- Xem nhật ký hoạt động quản trị.

Không được phép:

- Xem hoặc tải ảnh tạm nếu không có quyền cần thiết.
- Truy cập khóa bí mật từ giao diện trình duyệt.
- Xóa dấu vết kiểm toán của các thao tác quan trọng.
- Thay đổi dữ liệu điểm danh mà không tạo lịch sử điều chỉnh.

## 2.2. Người được điểm danh

Người được điểm danh không bắt buộc có tài khoản web.

Quy trình:

1. Đứng trước camera đã được cấu hình.
2. Camera chụp ảnh theo chu kỳ.
3. Backend xử lý ảnh.
4. Hệ thống trả về trạng thái nhận diện và điểm danh.
5. Kết quả được lưu nếu đáp ứng các điều kiện nghiệp vụ.

Hệ thống không được hiển thị thông tin cá nhân của người khác qua giao diện thiết bị công khai.

## 2.3. Thiết bị camera

Mỗi thiết bị phải có danh tính riêng, bao gồm:

- ID thiết bị.
- Tên thiết bị.
- Địa điểm lắp đặt.
- Trạng thái hoạt động.
- Thời điểm liên lạc gần nhất.
- Thông tin phiên bản firmware.
- Thông tin xác thực riêng của thiết bị.

Một thiết bị bị vô hiệu hóa không được tiếp tục gửi yêu cầu điểm danh hợp lệ.

---

# 3. Kiến trúc tổng thể

## 3.1. Sơ đồ hệ thống

```text
                         INTERNET
                             |
                   +-------------------+
                   |   ESP32-CAM       |
                   | Chụp ảnh JPEG     |
                   +-------------------+
                             |
                         HTTPS POST
                             |
                             v
                   +-------------------+
                   | Python FastAPI    |
                   |                   |
                   | 1. Xác thực thiết bị
                   | 2. Kiểm tra ảnh
                   | 3. Xử lý ảnh trong RAM
                   | 4. Nhận diện khuôn mặt
                   | 5. Quyết định điểm danh
                   +-------------------+
                       |           |
                       |           |
                       v           v
              +---------------+  +----------------+
              | Supabase      |  | Temporary      |
              | PostgreSQL    |  | WebP Storage   |
              |               |  |                |
              | Attendance    |  | Tối đa 24 giờ  |
              | People        |  | Nếu cần lưu    |
              | Sessions      |  +----------------+
              | Devices       |
              +---------------+
                       ^
                       |
                 Server-side API
                       |
                       v
              +-------------------+
              | Next.js Web       |
              | Admin Dashboard   |
              +-------------------+
```

## 3.2. Nguyên tắc kiến trúc

- Backend là nơi duy nhất quyết định một yêu cầu có đủ điều kiện tạo bản ghi điểm danh hay không.
- Trình duyệt không trực tiếp ghi kết quả điểm danh vào cơ sở dữ liệu.
- ESP32-CAM không tự quyết định danh tính người được điểm danh.
- Không đưa khóa bí mật của Supabase hoặc token thiết bị vào mã JavaScript phía trình duyệt.
- API, database và giao diện quản trị được triển khai độc lập về mặt trách nhiệm, nhưng có thể nằm trong cùng một repository.
- Việc triển khai frontend và backend trên cùng hay khác máy chủ không làm thay đổi các quy tắc nghiệp vụ.

---

# 4. Công nghệ dự kiến

| Thành phần | Công nghệ | Trách nhiệm |
|---|---|---|
| Frontend | Next.js, TypeScript | Giao diện quản trị |
| UI | React, CSS/Tailwind CSS | Hiển thị và tương tác |
| Backend | Python, FastAPI | API và nghiệp vụ |
| Xử lý ảnh | Pillow, OpenCV khi cần | Đọc, chuẩn hóa ảnh |
| Nhận diện | Thư viện/mô hình nhận diện khuôn mặt phù hợp | Trích xuất và so sánh embedding |
| Database | Supabase PostgreSQL | Lưu dữ liệu nghiệp vụ |
| Lưu ảnh tạm | Object storage hoặc vùng lưu tạm phù hợp | Ảnh WebP có thời hạn |
| Camera | ESP32-CAM + OV2640 | Chụp ảnh |
| Giao tiếp | HTTPS, REST, JSON, multipart/form-data | Trao đổi dữ liệu |
| Quản lý mã nguồn | Git | Theo dõi thay đổi |

**Lưu ý:** Chưa chốt thư viện AI. Cần đánh giá độ chính xác, hiệu năng, điều kiện cấp phép, khả năng triển khai trên cloud và khả năng tương thích của mô hình trước khi lựa chọn.

---

# 5. Yêu cầu chức năng

## FR-01: Xác thực thiết bị

Mỗi thiết bị phải được xác thực trước khi gửi ảnh.

Điều kiện:

- Yêu cầu gửi kèm thông tin xác thực.
- Backend kiểm tra thiết bị có tồn tại và đang hoạt động.
- Thiết bị không hợp lệ nhận HTTP 401 hoặc 403.
- Thiết bị đã bị vô hiệu hóa không được phép điểm danh.
- Mỗi thiết bị có thể được thu hồi hoặc thay mới thông tin xác thực độc lập.

Không dùng chung một token cố định cho tất cả thiết bị trong môi trường thực tế.

## FR-02: Tiếp nhận ảnh

API tiếp nhận ảnh từ camera bằng phương thức POST.

Yêu cầu:

- Chỉ nhận định dạng đầu vào được hỗ trợ; MVP sử dụng JPEG.
- Kích thước file tối đa mặc định: 5 MB.
- Phải kiểm tra định dạng thực tế của dữ liệu, không chỉ tin vào HTTP Content-Type.
- Từ chối ảnh rỗng, ảnh lỗi hoặc ảnh có độ phân giải vượt giới hạn.
- Có giới hạn thời gian xử lý và kích thước request.
- Không tin tưởng tên file do client gửi.
- Không ghi ảnh JPEG gốc xuống ổ đĩa nếu không cần thiết.

Mã phản hồi dự kiến:

- `201 Created`: đã tiếp nhận thành công trong chế độ kiểm thử.
- `400 Bad Request`: dữ liệu ảnh không hợp lệ.
- `401 Unauthorized`: xác thực thiết bị thất bại.
- `403 Forbidden`: thiết bị không được phép hoạt động.
- `413 Payload Too Large`: vượt giới hạn dữ liệu.
- `415 Unsupported Media Type`: định dạng không được hỗ trợ.
- `429 Too Many Requests`: gửi quá nhiều yêu cầu.
- `500 Internal Server Error`: lỗi xử lý nội bộ.

## FR-03: Xử lý ảnh

Backend phải:

1. Giải mã ảnh trong bộ nhớ.
2. Kiểm tra kích thước, số pixel và khả năng giải mã.
3. Chuẩn hóa màu sắc và hướng ảnh khi cần.
4. Xác định ảnh có đủ chất lượng để nhận diện hay không.
5. Đưa ảnh vào mô hình nhận diện.
6. Giải phóng tài nguyên sau khi xử lý.

Các giới hạn như độ phân giải tối đa, thời gian xử lý và kích thước ảnh phải được cấu hình để tránh tiêu thụ quá nhiều RAM.

## FR-04: Chuyển đổi và lưu ảnh WebP

JPEG là định dạng truyền từ camera trong MVP. WebP được dùng cho ảnh tạm khi cần lưu.

Quy tắc:

- Không bắt buộc lưu mọi ảnh được gửi lên.
- Chỉ tạo bản WebP khi có nhu cầu kiểm thử, điều tra lỗi hoặc được chính sách cho phép.
- Chuyển đổi ảnh trong RAM.
- Có thể giảm kích thước ảnh trước khi lưu.
- Chất lượng WebP mặc định dự kiến: 80; có thể điều chỉnh sau khi kiểm thử.
- Không lưu ảnh WebP lâu dài trong database dạng nhị phân.
- Không dùng thư mục tạm của container làm kho lưu trữ lâu dài.
- Mỗi ảnh phải có thời điểm hết hạn hoặc cơ chế xác định thời điểm hết hạn.

## FR-05: Tự động xóa ảnh tạm

Thời hạn lưu ảnh tạm tối đa: 24 giờ kể từ thời điểm tạo ảnh.

Yêu cầu:

- Ảnh hết hạn phải được xóa tự động.
- Tác vụ xóa phải chạy độc lập với việc có camera gửi ảnh mới hay không.
- Việc server khởi động lại không được làm mất lịch chạy dọn dẹp một cách âm thầm.
- Nếu lưu trên object storage, phải có tác vụ xóa hoặc lifecycle rule phù hợp.
- Có log số lượng ảnh được xóa và lỗi xóa.
- Không ghi nội dung ảnh hoặc embedding vào log ứng dụng.
- Nếu không thể đảm bảo xóa đúng hạn, không được tuyên bố rằng hệ thống đã đảm bảo thời hạn 24 giờ.

Thời hạn này áp dụng cho ảnh tạm do hệ thống quản lý. Bản sao trong backup hoặc log của nhà cung cấp phải được kiểm tra riêng theo chính sách lưu trữ của nền tảng.

## FR-06: Đăng ký người tham gia

Admin có thể tạo hồ sơ người tham gia.

Thông tin cơ bản:

- ID người tham gia.
- Mã định danh nội bộ.
- Họ và tên.
- Trạng thái hoạt động.
- Thời điểm tạo.
- Thời điểm cập nhật.

Dữ liệu khuôn mặt:

- Được tạo từ quy trình đăng ký có sự đồng ý phù hợp.
- Có thể lưu embedding thay vì lưu ảnh gốc lâu dài.
- Phải được bảo vệ như dữ liệu sinh trắc học nhạy cảm.
- Có thể cập nhật hoặc xóa theo quy trình.
- Không được sử dụng dữ liệu khuôn mặt ngoài mục đích đã thông báo.

## FR-07: Nhận diện khuôn mặt

Backend sử dụng mô hình nhận diện đã được lựa chọn để so sánh khuôn mặt từ ảnh mới với dữ liệu đăng ký.

Kết quả nhận diện có thể gồm:

- `MATCHED`: tìm được danh tính phù hợp.
- `UNKNOWN`: không tìm thấy danh tính phù hợp.
- `NO_FACE`: không phát hiện khuôn mặt.
- `MULTIPLE_FACES`: phát hiện nhiều khuôn mặt khi chế độ hiện tại chỉ hỗ trợ một người.
- `LOW_QUALITY`: ảnh không đủ chất lượng.
- `PROCESSING_ERROR`: xảy ra lỗi xử lý.

Ngưỡng so khớp phải được cấu hình và kiểm thử trên dữ liệu phù hợp. Không mặc định rằng một ngưỡng duy nhất có thể phù hợp với mọi camera, ánh sáng và nhóm người.

Nếu độ tin cậy không đạt ngưỡng, hệ thống phải từ chối tự động điểm danh và trả về trạng thái phù hợp.

## FR-08: Quản lý phiên điểm danh

Admin có thể tạo phiên điểm danh với các thông tin:

- Tên phiên.
- Thời gian bắt đầu.
- Thời gian kết thúc.
- Địa điểm hoặc nhóm đối tượng.
- Trạng thái phiên.
- Quy tắc cho phép điểm danh.

Trạng thái phiên:

- `DRAFT`: bản nháp.
- `SCHEDULED`: đã lên lịch.
- `ACTIVE`: đang mở.
- `CLOSED`: đã đóng.
- `CANCELLED`: đã hủy.

Backend phải kiểm tra thời gian và trạng thái phiên trước khi ghi nhận điểm danh.

Không cho phép điểm danh vào phiên đã đóng hoặc đã hủy, trừ khi có nghiệp vụ điều chỉnh thủ công riêng.

## FR-09: Ghi nhận điểm danh

Một bản ghi điểm danh hợp lệ cần đáp ứng:

1. Thiết bị đã được xác thực.
2. Thiết bị đang hoạt động.
3. Ảnh hợp lệ.
4. Nhận diện đạt ngưỡng đã cấu hình.
5. Người tham gia đang hoạt động.
6. Có phiên điểm danh phù hợp.
7. Phiên đang cho phép điểm danh.
8. Chưa có bản ghi cho cùng người tham gia trong cùng phiên, hoặc nghiệp vụ cho phép cập nhật theo quy tắc được xác định trước.

Khi hợp lệ, backend ghi một bản ghi vào database.

Mỗi bản ghi nên có:

- ID bản ghi.
- ID người tham gia.
- ID phiên điểm danh.
- ID thiết bị.
- Thời điểm ghi nhận.
- Phương thức ghi nhận.
- Trạng thái.
- Thông tin kỹ thuật tối thiểu cần thiết để kiểm tra lỗi, nếu được phép.

Không lưu ảnh khuôn mặt trong bản ghi điểm danh.

## FR-10: Chống điểm danh trùng

Hệ thống phải ngăn một người tạo nhiều bản ghi điểm danh trong cùng một phiên.

Cơ chế bắt buộc:

- Có unique constraint ở database trên cặp `(session_id, person_id)`.
- Không chỉ dựa vào kiểm tra trước khi insert ở Python, vì nhiều request có thể đến đồng thời.
- Nếu nhận diện lại cùng người trong phiên, trả về trạng thái `ALREADY_CHECKED_IN`.
- Không tạo thêm bản ghi trùng.
- Nếu cần lưu lịch sử nhiều lần phát hiện, lưu ở bảng sự kiện riêng và không xem mỗi sự kiện là một lần điểm danh mới.

## FR-11: Dashboard quản trị

Dashboard hiển thị tối thiểu:

- Tổng số người tham gia đang hoạt động.
- Tổng số phiên điểm danh.
- Số lượt điểm danh trong phiên hiện tại.
- Danh sách người đã điểm danh.
- Danh sách người chưa điểm danh trong phiên.
- Thời điểm điểm danh gần nhất.
- Trạng thái thiết bị.
- Thống kê lỗi nhận diện hoặc lỗi thiết bị ở mức tổng hợp.

Không hiển thị ảnh khuôn mặt hoặc embedding mặc định trên dashboard.

## FR-12: Quản lý lịch sử

Admin có thể:

- Lọc theo phiên.
- Lọc theo người tham gia.
- Lọc theo ngày.
- Lọc theo thiết bị.
- Xem chi tiết bản ghi.
- Xuất báo cáo CSV khi được cấp quyền.
- Điều chỉnh bản ghi sai theo quy trình kiểm toán.

Mọi chỉnh sửa hoặc hủy bản ghi phải lưu người thực hiện, thời gian và lý do.

## FR-13: Quản lý thiết bị

Admin có thể:

- Tạo thiết bị.
- Đặt tên và địa điểm.
- Xem trạng thái.
- Xem thời điểm liên lạc gần nhất.
- Vô hiệu hóa thiết bị.
- Thu hồi hoặc thay token.
- Xem lỗi kết nối và lỗi gửi ảnh.

Token chỉ được hiển thị theo quy trình cấp phát an toàn và không được trả lại trong các truy vấn thông thường.

## FR-14: Xử lý lỗi và phản hồi

Mọi yêu cầu gửi ảnh phải trả về kết quả có cấu trúc rõ ràng.

Ví dụ:

```json
{
  "success": true,
  "status": "CHECKED_IN",
  "person_id": "uuid",
  "session_id": "uuid",
  "checked_in_at": "2026-10-13T10:00:00Z"
}
```

Đây là định dạng minh họa. API thật phải xác định rõ trường bắt buộc và không được trả thông tin cá nhân cho thiết bị công khai nếu không cần thiết.

Các trạng thái nghiệp vụ nên gồm:

- `CHECKED_IN`
- `ALREADY_CHECKED_IN`
- `UNKNOWN_PERSON`
- `NO_FACE`
- `MULTIPLE_FACES`
- `LOW_QUALITY`
- `NO_ACTIVE_SESSION`
- `DEVICE_DISABLED`
- `PROCESSING_ERROR`

---

# 6. Yêu cầu phi chức năng

## NFR-01: Hiệu năng

Mục tiêu ban đầu để kiểm thử:

- API health phản hồi trong vòng 1 giây trong điều kiện tải nhẹ.
- Một yêu cầu xử lý ảnh có mục tiêu hoàn thành trong vòng 3 giây ở điều kiện bình thường.
- Hệ thống không tải toàn bộ ảnh hoặc embedding của tất cả người tham gia vào RAM nếu dữ liệu tăng lớn.
- Có thể cấu hình số tác vụ xử lý đồng thời.
- Có timeout cho kết nối và xử lý ảnh.

Các mục tiêu trên là tiêu chí kiểm thử ban đầu, không phải cam kết hiệu năng trước khi đo thực tế trên cloud và mô hình AI được lựa chọn.

## NFR-02: Tối ưu chi phí cloud

- Chỉ sử dụng độ phân giải cần thiết cho nhận diện.
- Không lưu JPEG gốc lâu dài.
- Chỉ lưu WebP tạm khi cần.
- Không dùng database để lưu nhị phân ảnh.
- Giới hạn kích thước request.
- Giới hạn tốc độ gửi ảnh.
- Giảm số lần ghi database không cần thiết.
- Không ghi nhận điểm danh lặp lại trong cùng phiên.
- Theo dõi CPU, RAM, dung lượng lưu trữ và số request.
- Không giữ tác vụ xử lý AI chạy liên tục khi không có yêu cầu nếu nền tảng không phù hợp.

## NFR-03: Bảo mật

- Tất cả kết nối qua Internet phải dùng HTTPS.
- Không sử dụng chế độ bỏ qua xác minh chứng chỉ TLS trong môi trường thật.
- Secret chỉ được lưu trong biến môi trường hoặc secret manager.
- Không commit `.env`, token, khóa service role hoặc dữ liệu khuôn mặt.
- Phân quyền admin phía server.
- Bật và kiểm thử RLS cho các bảng Supabase phù hợp.
- Xác thực và phân quyền mọi API nhạy cảm.
- Giới hạn request và chống gửi lặp.
- Không ghi token, ảnh, embedding hoặc thông tin bí mật vào log.
- Có quy trình thay token khi thiết bị bị mất hoặc bị xâm nhập.

## NFR-04: Quyền riêng tư

- Thông báo rõ mục đích thu thập và sử dụng dữ liệu khuôn mặt.
- Có sự đồng ý phù hợp trước khi đăng ký và sử dụng dữ liệu sinh trắc học.
- Chỉ thu thập dữ liệu cần thiết.
- Quy định thời hạn lưu ảnh, embedding và bản ghi điểm danh riêng biệt.
- Có quy trình xem, sửa và xóa dữ liệu theo quyền hợp lệ.
- Giới hạn quyền truy cập dữ liệu sinh trắc học.
- Không sử dụng dữ liệu cho mục đích khác ngoài phạm vi đã thông báo.
- Có phương án điểm danh thay thế khi nhận diện thất bại.
- Kiểm tra nghĩa vụ pháp lý áp dụng cho việc xử lý dữ liệu cá nhân tại nơi triển khai.

## NFR-05: Độ tin cậy

- Database là nguồn dữ liệu chính thức của kết quả điểm danh.
- Không xác nhận thành công trước khi database xác nhận ghi nhận hợp lệ.
- Có unique constraint chống trùng.
- Có log lỗi đủ để chẩn đoán nhưng không chứa dữ liệu nhạy cảm.
- Có kiểm tra sức khỏe API.
- Có cơ chế khởi động lại và giám sát tác vụ dọn dẹp.
- Có quy trình backup và khôi phục dữ liệu nghiệp vụ.

## NFR-06: Khả năng bảo trì

- Frontend và backend có cấu trúc thư mục rõ ràng.
- Cấu hình tách khỏi mã nguồn.
- API có tài liệu OpenAPI.
- Có kiểm thử đơn vị cho nghiệp vụ quan trọng.
- Có migration cho thay đổi database.
- Có README hướng dẫn cài đặt và triển khai.
- Có log và mã lỗi thống nhất.
- Không trộn logic nhận diện, logic điểm danh và logic giao diện vào cùng một module.

---

# 7. Thiết kế cơ sở dữ liệu

Tên bảng và trường dưới đây là thiết kế đề xuất. Có thể điều chỉnh khi chốt mô hình dữ liệu.

## 7.1. Bảng `people`

Lưu hồ sơ người tham gia.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `external_code` | TEXT | Mã người tham gia |
| `full_name` | TEXT | Họ tên |
| `is_active` | BOOLEAN | Trạng thái hoạt động |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |
| `updated_at` | TIMESTAMPTZ | Thời điểm cập nhật |

`external_code` nên có unique constraint nếu được sử dụng làm mã định danh duy nhất.

## 7.2. Bảng `face_profiles`

Lưu dữ liệu phục vụ nhận diện.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `person_id` | UUID | Tham chiếu `people.id` |
| `embedding` | Dữ liệu số phù hợp | Vector đặc trưng khuôn mặt |
| `model_version` | TEXT | Phiên bản mô hình |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |
| `is_active` | BOOLEAN | Có được sử dụng hay không |

Lưu ý:

- Cần chọn kiểu lưu embedding phù hợp với mô hình và phương pháp tìm kiếm.
- Nếu dùng PostgreSQL, có thể đánh giá extension `pgvector` khi số lượng embedding tăng.
- Embedding là dữ liệu sinh trắc học nhạy cảm và phải được bảo vệ.
- Không lưu ảnh gốc trong bảng này mặc định.
- Quy trình cập nhật hoặc xóa hồ sơ phải đồng bộ với dữ liệu embedding.

## 7.3. Bảng `attendance_sessions`

Lưu phiên điểm danh.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `name` | TEXT | Tên phiên |
| `starts_at` | TIMESTAMPTZ | Bắt đầu |
| `ends_at` | TIMESTAMPTZ | Kết thúc |
| `status` | TEXT | Trạng thái |
| `created_by` | UUID | Admin tạo phiên |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |

Backend phải xác minh trạng thái và thời gian phiên trước khi điểm danh.

## 7.4. Bảng `devices`

Lưu thông tin thiết bị.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `name` | TEXT | Tên thiết bị |
| `location` | TEXT | Địa điểm |
| `is_active` | BOOLEAN | Trạng thái |
| `last_seen_at` | TIMESTAMPTZ | Liên lạc gần nhất |
| `firmware_version` | TEXT | Phiên bản firmware |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |

Token hoặc hash token phải được thiết kế theo cơ chế xác thực phù hợp; không lưu token nguyên văn trong trường dữ liệu có thể đọc tùy ý.

## 7.5. Bảng `attendance_records`

Lưu kết quả điểm danh.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `person_id` | UUID | Người tham gia |
| `session_id` | UUID | Phiên điểm danh |
| `device_id` | UUID | Thiết bị ghi nhận |
| `checked_in_at` | TIMESTAMPTZ | Thời điểm điểm danh |
| `method` | TEXT | Phương thức ghi nhận |
| `status` | TEXT | Trạng thái bản ghi |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |

Ràng buộc bắt buộc:

```sql
UNIQUE (session_id, person_id)
```

Cần thiết lập foreign key tới các bảng tương ứng và quy định hành vi khi hồ sơ bị vô hiệu hóa hoặc xóa.

## 7.6. Bảng `temporary_image_metadata`

Chỉ cần bảng này nếu lưu ảnh tạm trên object storage.

| Trường | Kiểu | Mô tả |
|---|---|---|
| `id` | UUID | Khóa chính |
| `storage_key` | TEXT | Đường dẫn đối tượng lưu trữ |
| `format` | TEXT | Phải là WebP |
| `created_at` | TIMESTAMPTZ | Thời điểm tạo |
| `expires_at` | TIMESTAMPTZ | Thời điểm hết hạn |
| `deleted_at` | TIMESTAMPTZ | Thời điểm xác nhận xóa, nếu cần |

Không lưu ảnh nhị phân trong bảng metadata. Không đưa đường dẫn nội bộ hoặc URL truy cập công khai vào API không cần thiết.

---

# 8. Đặc tả API

Các endpoint dưới đây là đề xuất cho MVP.

## 8.1. Kiểm tra sức khỏe

`GET /api/health`

Mục đích:

- Kiểm tra API hoạt động.
- Phục vụ health check của nền tảng triển khai.

Ví dụ:

```json
{
  "status": "ok",
  "service": "facecheck-api"
}
```

Endpoint này không trả token, thông tin database hoặc dữ liệu nội bộ.

## 8.2. Gửi ảnh từ ESP32-CAM

`POST /api/camera/upload`

Header:

```text
X-Device-Token: <device-token>
Content-Type: multipart/form-data
```

Trường multipart:

- `image`: file JPEG.

Nhiệm vụ:

1. Xác thực thiết bị.
2. Kiểm tra trạng thái thiết bị.
3. Kiểm tra request và ảnh.
4. Xử lý ảnh trong RAM.
5. Chạy nhận diện nếu chức năng đã được bật.
6. Kiểm tra phiên điểm danh.
7. Ghi nhận kết quả nếu hợp lệ.
8. Chỉ lưu WebP tạm nếu được cấu hình.
9. Trả kết quả nghiệp vụ.

API thật không nên mặc định trả `201 Created` chỉ vì nhận được ảnh. Phải phân biệt rõ nhận ảnh thành công và điểm danh thành công.

## 8.3. Danh sách người tham gia

`GET /api/admin/people`

Yêu cầu:

- Admin đã đăng nhập.
- Có phân trang.
- Có thể tìm theo mã hoặc tên.
- Không trả embedding hoặc ảnh khuôn mặt.

## 8.4. Tạo người tham gia

`POST /api/admin/people`

Yêu cầu:

- Admin đã đăng nhập.
- Kiểm tra dữ liệu đầu vào.
- Kiểm tra mã người tham gia trùng.
- Ghi nhận nhật ký thao tác khi cần.

## 8.5. Quản lý phiên điểm danh

- `GET /api/admin/sessions`
- `POST /api/admin/sessions`
- `PATCH /api/admin/sessions/{id}`
- `GET /api/admin/sessions/{id}/attendance`

Tất cả endpoint quản trị phải kiểm tra quyền phía server.

## 8.6. Lịch sử điểm danh

`GET /api/admin/attendance`

Bộ lọc dự kiến:

- `session_id`
- `person_id`
- `device_id`
- `from`
- `to`
- `page`
- `limit`

Phải giới hạn kích thước trang và xác thực quyền truy cập.

## 8.7. Quy tắc API chung

- Dùng JSON cho request/response thông thường.
- Dùng multipart/form-data cho ảnh.
- Có cấu trúc lỗi thống nhất.
- Có timeout và giới hạn request.
- Có phân trang cho danh sách lớn.
- Không trả stack trace cho client.
- Không để client tự chỉ định danh tính được điểm danh.
- Không tin tưởng `person_id` do thiết bị gửi lên; danh tính phải do backend xác định.

---

# 9. Quy trình nghiệp vụ

## 9.1. Quy trình đăng ký người tham gia

1. Admin đăng nhập.
2. Tạo hồ sơ người tham gia.
3. Thông báo mục đích và xin sự đồng ý phù hợp.
4. Thu thập ảnh đăng ký theo quy trình được thiết kế.
5. Kiểm tra chất lượng ảnh.
6. Trích xuất embedding.
7. Lưu embedding và phiên bản mô hình.
8. Kiểm tra khả năng nhận diện.
9. Kích hoạt hồ sơ khi hoàn tất.

Nếu đăng ký thất bại, không được để lại hồ sơ khuôn mặt ở trạng thái hoạt động một phần.

## 9.2. Quy trình điểm danh

1. ESP32-CAM kết nối Wi-Fi.
2. Camera chụp JPEG.
3. Thiết bị gửi ảnh qua HTTPS.
4. Backend xác thực thiết bị.
5. Backend kiểm tra ảnh.
6. Backend phát hiện khuôn mặt.
7. Backend kiểm tra chất lượng.
8. Backend so sánh với dữ liệu đăng ký.
9. Backend xác định phiên điểm danh phù hợp.
10. Backend kiểm tra trùng.
11. Backend ghi nhận điểm danh bằng thao tác database an toàn.
12. Backend trả trạng thái.
13. Thiết bị hoặc dashboard hiển thị phản hồi phù hợp.

Nếu bất kỳ bước bắt buộc nào thất bại, không tạo bản ghi điểm danh thành công.

## 9.3. Quy trình xử lý ảnh tạm

1. Backend nhận JPEG.
2. Ảnh được xử lý trong RAM.
3. Nếu không cần lưu ảnh, giải phóng dữ liệu ảnh sau xử lý.
4. Nếu cần lưu kiểm thử, mã hóa sang WebP.
5. Ghi file vào vùng lưu trữ tạm.
6. Ghi metadata và thời điểm hết hạn nếu dùng object storage.
7. Tác vụ dọn dẹp tìm các ảnh đã hết hạn.
8. Xóa đối tượng ảnh.
9. Ghi nhận kết quả dọn dẹp.
10. Nếu xóa thất bại, thử lại và cảnh báo khi vượt ngưỡng lỗi.

---

# 10. Yêu cầu về giao diện

## 10.1. Trang quản trị `/admin`

Các trang chính:

- `/admin`: Dashboard.
- `/admin/people`: Quản lý người tham gia.
- `/admin/people/[id]`: Chi tiết người tham gia.
- `/admin/sessions`: Quản lý phiên điểm danh.
- `/admin/sessions/[id]`: Chi tiết phiên và danh sách điểm danh.
- `/admin/attendance`: Lịch sử điểm danh.
- `/admin/devices`: Quản lý thiết bị.
- `/admin/settings`: Cấu hình và chính sách lưu trữ.

Việc ẩn hoặc không hiển thị liên kết admin trên giao diện công khai không phải cơ chế bảo mật. Mọi route và API quản trị phải yêu cầu xác thực và kiểm tra quyền.

## 10.2. Trạng thái giao diện

Các màn hình cần xử lý:

- Loading.
- Thành công.
- Không có dữ liệu.
- Lỗi kết nối.
- Không đủ quyền.
- Dữ liệu không hợp lệ.
- Phiên điểm danh đã đóng.
- Thiết bị ngoại tuyến.
- Người chưa đăng ký.
- Điểm danh trùng.

## 10.3. Khả năng sử dụng

- Giao diện tiếng Việt.
- Tương thích máy tính và điện thoại.
- Có thông báo lỗi dễ hiểu.
- Có xác nhận trước thao tác xóa hoặc vô hiệu hóa.
- Có phân trang và bộ lọc khi dữ liệu lớn.
- Không phụ thuộc vào ảnh để truyền đạt trạng thái điểm danh.

---

# 11. Triển khai và cấu hình môi trường

## 11.1. Cấu trúc repository

```text
FaceCheck/
├── frontend/
│   ├── app/
│   ├── components/
│   ├── lib/
│   ├── public/
│   ├── package.json
│   └── .env.example
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   ├── services/
│   │   ├── models/
│   │   ├── repositories/
│   │   └── core/
│   ├── tests/
│   ├── requirements.txt
│   └── .env.example
│
├── firmware/
│   └── esp32-camera/
│       └── esp32-camera.ino
│
├── supabase/
│   └── migrations/
│
├── docs/
│   ├── architecture.md
│   ├── api.md
│   └── privacy.md
│
├── .gitignore
└── README.md
```

## 11.2. Biến môi trường backend

Ví dụ `.env.example`:

```dotenv
APP_ENV=development
DEVICE_API_TOKEN=
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
UPLOAD_DIR=
MAX_UPLOAD_BYTES=5242880
IMAGE_RETENTION_HOURS=24
TEMP_IMAGE_STORAGE_ENABLED=false
```

Quy tắc:

- `.env.example` chỉ chứa tên biến và giá trị mẫu không bí mật.
- `.env` thật không được commit.
- Khóa `SUPABASE_SERVICE_ROLE_KEY` chỉ được sử dụng ở backend đáng tin cậy.
- Không đưa khóa service role vào Next.js client-side code.
- Trong môi trường thực tế nên dùng secret manager của nền tảng.
- Token thiết bị phải được cấp phát riêng và có thể thu hồi.

## 11.3. Biến môi trường frontend

Ví dụ:

```dotenv
NEXT_PUBLIC_APP_NAME=FaceCheck
NEXT_PUBLIC_API_BASE_URL=
```

Chỉ các giá trị thực sự công khai mới được dùng tiền tố `NEXT_PUBLIC_`.

## 11.4. Quy tắc lưu trữ

- Database lưu dữ liệu nghiệp vụ.
- Object storage lưu ảnh WebP tạm nếu được bật.
- Không dùng filesystem tạm của cloud làm kho lưu trữ bền vững.
- Nếu lưu ảnh tạm, phải có chính sách xóa độc lập với vòng đời API.
- Phải kiểm thử cơ chế xóa trên môi trường triển khai thật.

---

# 12. Kiểm thử và tiêu chí nghiệm thu

## 12.1. Kiểm thử tiếp nhận ảnh

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-01 | Gửi JPEG hợp lệ và token hợp lệ | API tiếp nhận thành công |
| T-02 | Thiếu token | HTTP 401 |
| T-03 | Token sai | HTTP 401 |
| T-04 | Thiết bị bị vô hiệu hóa | HTTP 403 |
| T-05 | File rỗng | HTTP 400 |
| T-06 | File vượt giới hạn | HTTP 413 |
| T-07 | Định dạng không hỗ trợ | HTTP 415 |
| T-08 | JPEG bị hỏng | HTTP 400 |
| T-09 | Độ phân giải vượt giới hạn | HTTP 413 |
| T-10 | Server không có cấu hình bắt buộc | HTTP 503 hoặc lỗi cấu hình tương ứng |

## 12.2. Kiểm thử nhận diện

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-11 | Người đã đăng ký, ảnh đạt yêu cầu | Nhận diện đúng trong điều kiện kiểm thử |
| T-12 | Người chưa đăng ký | `UNKNOWN_PERSON` |
| T-13 | Không có khuôn mặt | `NO_FACE` |
| T-14 | Nhiều khuôn mặt khi chỉ hỗ trợ một người | `MULTIPLE_FACES` |
| T-15 | Ảnh quá mờ hoặc quá tối | `LOW_QUALITY` hoặc trạng thái lỗi chất lượng tương ứng |
| T-16 | Lỗi mô hình AI | Không ghi nhận điểm danh thành công |
| T-17 | Ngưỡng nhận diện không đạt | Không tự động điểm danh |

Cần đo tỷ lệ nhận diện sai, từ chối sai và ảnh hưởng của ánh sáng, góc mặt, camera, khoảng cách. Không đánh giá chất lượng mô hình chỉ bằng một vài ảnh thử nghiệm.

## 12.3. Kiểm thử nghiệp vụ điểm danh

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-18 | Người hợp lệ trong phiên đang mở | Tạo bản ghi |
| T-19 | Người đã điểm danh trong cùng phiên | Không tạo bản ghi trùng |
| T-20 | Phiên đã đóng | Không điểm danh |
| T-21 | Không có phiên phù hợp | `NO_ACTIVE_SESSION` |
| T-22 | Hồ sơ người tham gia bị vô hiệu hóa | Không điểm danh |
| T-23 | Hai request đồng thời cho cùng người và phiên | Tối đa một bản ghi |
| T-24 | Database lỗi | Không trả thành công giả |
| T-25 | Thiết bị bị thu hồi | Không ghi nhận |

## 12.4. Kiểm thử lưu trữ

| Mã | Tình huống | Kết quả mong đợi |
|---|---|---|
| T-26 | Bật lưu ảnh tạm | Chỉ tạo WebP |
| T-27 | Tắt lưu ảnh tạm | Không lưu ảnh xuống kho tạm |
| T-28 | Ảnh chưa hết hạn | Không bị xóa |
| T-29 | Ảnh đã hết hạn | Được xóa theo chính sách |
| T-30 | Tác vụ dọn dẹp gặp lỗi | Có log và cơ chế thử lại |
| T-31 | API restart | Cơ chế dọn dẹp tiếp tục hoạt động |
| T-32 | Ảnh hết hạn nhưng database còn metadata | Metadata được xử lý theo chính sách, không được xem là ảnh còn tồn tại |
| T-33 | Lưu trữ có backup riêng | Kiểm tra thời hạn tồn tại của bản sao |

## 12.5. Tiêu chí nghiệm thu MVP

MVP được coi là đạt khi:

- ESP32-CAM gửi ảnh thành công qua HTTPS.
- Backend từ chối request không hợp lệ.
- Nhận diện hoạt động trên bộ dữ liệu thử nghiệm đã xác định.
- Điểm danh chỉ được tạo khi mọi điều kiện nghiệp vụ được thỏa mãn.
- Không tạo bản ghi trùng trong cùng phiên, kể cả khi có request đồng thời.
- Dashboard hiển thị đúng dữ liệu từ database.
- Không cần lưu ảnh lâu dài để duy trì lịch sử điểm danh.
- Ảnh tạm chỉ được lưu dưới dạng WebP khi chức năng lưu tạm được bật.
- Có cơ chế xóa ảnh theo thời hạn và kiểm thử được trên môi trường cloud.
- Không có khóa bí mật trong repository hoặc frontend bundle.
- Có xác thực và phân quyền cho toàn bộ chức năng quản trị.
- Có tài liệu cài đặt, triển khai và khôi phục lỗi cơ bản.

---

# 13. Kế hoạch phát triển theo giai đoạn

## Phase 1 — Nền tảng API

- Tạo repository.
- Tạo Python FastAPI.
- Tạo endpoint health.
- Xây dựng API tiếp nhận JPEG.
- Xác thực token thiết bị.
- Kiểm tra file và giới hạn request.
- Kiểm thử upload bằng `curl.exe` trước khi mua phần cứng.

**Điều kiện hoàn thành:** Có thể gửi ảnh JPEG từ máy tính tới API và nhận phản hồi chính xác.

## Phase 2 — WebP và quản lý vòng đời ảnh

- Tích hợp Pillow.
- Chuyển JPEG sang WebP trong RAM.
- Bật/tắt lưu ảnh tạm bằng cấu hình.
- Thiết lập thời hạn 24 giờ.
- Xây dựng tác vụ xóa độc lập.
- Kiểm thử trên môi trường cloud.

**Điều kiện hoàn thành:** Ảnh không bị lưu ngoài dự kiến; ảnh tạm được dọn dẹp theo chính sách.

## Phase 3 — Cơ sở dữ liệu

- Thiết kế và tạo migration Supabase.
- Tạo bảng người tham gia, phiên điểm danh và thiết bị.
- Tạo bảng bản ghi điểm danh.
- Thiết lập foreign key, unique constraint và RLS.
- Kiểm thử quyền truy cập.

**Điều kiện hoàn thành:** Dữ liệu nghiệp vụ được lưu và truy vấn đúng.

## Phase 4 — Nhận diện khuôn mặt

- Chọn thư viện và mô hình.
- Xây dựng quy trình đăng ký.
- Trích xuất embedding.
- Xây dựng so khớp.
- Xác định ngưỡng.
- Đánh giá false accept và false reject.
- Kiểm thử chất lượng ảnh.

**Điều kiện hoàn thành:** Mô hình đạt các tiêu chí đã xác định trên bộ kiểm thử đại diện.

## Phase 5 — ESP32-CAM

- Mua và cấu hình phần cứng.
- Kết nối Wi-Fi.
- Chụp JPEG.
- Gửi ảnh qua HTTPS.
- Kiểm thử reconnect, timeout và lỗi mạng.
- Kiểm thử tài nguyên RAM và nguồn cấp.

**Điều kiện hoàn thành:** Camera gửi ảnh ổn định đến cloud trong điều kiện mạng mục tiêu.

## Phase 6 — Dashboard Next.js

- Xây dựng xác thực admin.
- Dashboard.
- Quản lý người tham gia.
- Quản lý phiên.
- Quản lý thiết bị.
- Lịch sử và báo cáo.
- Xử lý lỗi và phân quyền.

**Điều kiện hoàn thành:** Admin có thể vận hành nghiệp vụ từ giao diện web.

## Phase 7 — Bảo mật và triển khai thực tế

- Xác minh TLS.
- Rà soát secrets.
- Kiểm thử quyền truy cập.
- Kiểm thử chống request lặp.
- Kiểm thử xóa ảnh và retention.
- Kiểm thử tải.
- Kiểm tra backup, giám sát và quy trình khôi phục.
- Hoàn thiện chính sách quyền riêng tư.

**Điều kiện hoàn thành:** Hệ thống đáp ứng yêu cầu bảo mật, vận hành và lưu trữ đã xác định.

---

# 14. Các quyết định kiến trúc đã chốt

1. Backend dùng Python FastAPI.
2. Frontend dùng Next.js.
3. ESP32-CAM gửi ảnh trực tiếp tới backend qua Internet.
4. Không cần điện thoại làm trung gian truyền ảnh.
5. Đầu vào camera sử dụng JPEG.
6. WebP là định dạng lưu ảnh tạm.
7. Ảnh gốc không được lưu lâu dài mặc định.
8. Ảnh tạm có thời hạn tối đa 24 giờ và phải có cơ chế xóa tự động.
9. Kết quả điểm danh được lưu trong Supabase.
10. Database là nguồn dữ liệu chính thức.
11. Backend quyết định danh tính và kết quả điểm danh.
12. Có unique constraint chống điểm danh trùng.
13. Trang quản trị chỉ dành cho admin đã xác thực.
14. Không lưu secrets trong Git.
15. Nhận diện khuôn mặt phải được kiểm thử trước khi dùng trong thực tế.

# 15. Các quyết định còn cần xác nhận

- [ ] Chỉ cho phép một khuôn mặt trong mỗi khung hình hay hỗ trợ nhiều người.
- [ ] Một người có thể đăng ký bao nhiêu mẫu khuôn mặt.
- [ ] Cách xác định phiên điểm danh đang hoạt động: tự động theo thời gian hay do admin mở.
- [ ] Có cần lưu ảnh WebP kiểm thử hay mặc định tắt hoàn toàn.
- [ ] Mô hình nhận diện và ngưỡng so khớp.
- [ ] Có yêu cầu chống giả mạo bằng ảnh in hoặc màn hình điện thoại hay không.
- [ ] Nhà cung cấp cloud cho backend.
- [ ] Phương án lưu embedding và kiểm soát quyền truy cập.
- [ ] Quy trình xóa hồ sơ và dữ liệu sinh trắc học.
- [ ] Thời hạn lưu bản ghi điểm danh và thời gian lưu backup.
- [ ] Phương án điểm danh thủ công khi AI hoặc camera lỗi.

# 16. Nguyên tắc phát triển

- Hoàn thành từng giai đoạn và kiểm thử trước khi chuyển sang giai đoạn kế tiếp.
- Không tích hợp AI trước khi API nhận ảnh hoạt động ổn định.
- Không mua phần cứng trước khi xác minh được quy trình API cơ bản, trừ khi cần phần cứng để đánh giá chất lượng ảnh.
- Không đưa toàn bộ nghiệp vụ vào một file Python.
- Không để frontend quyết định kết quả điểm danh.
- Không lưu ảnh chỉ vì có thể lưu.
- Không coi kết quả nhận diện khuôn mặt là bằng chứng tuyệt đối về danh tính.
- Ưu tiên bảo mật, quyền riêng tư, khả năng kiểm thử và chi phí vận hành.
- Mọi thay đổi quan trọng về kiến trúc phải được cập nhật trong tài liệu này.

---

**Kết thúc đặc tả FaceCheck v1.0**
