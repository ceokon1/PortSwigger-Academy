# Endpoint
Ví dụ, một GET request như sau: 
```
Get /api/Dat HTTP/1.1 
Host: datdeptrai.com
```
Điểm cuối của api này sẽ là `/api/Dat`. Ta có thể tương tác với api bằng các endpoint khác nhau như `/api/Ceo` để truy xuất thông tin.
Sau khi đã xác định được các **endpoint**, ta có thể kiểm tra các response tại endpoint đó khi thay đổi HTTP method hoặc media type. 
Mỗi HTTP method chỉ định một hành động như sau:
```
GET - truy xuất dữ liệu
PATCH - Thay đổi dữ liệu
OPTIONS - kiểm tra các loại method có thể sử dụng
```
Ta có thể thay đổi các method để mở ra nhiều hướng tấn công hơn. Ví dụ:
`/api/Dat` sau khi thay đổi method từ `GET /api/Dat HTTP/1.1` thành `OPTIONS /api/Dat`, ta biết được **endpoint** này hỗ trợ các method sau:
```
GET /api/Dat - truy xuất dữ liệu
POST /api/Dat - tạo một task mới
DELETE /api/Dat - xoá một task
```
***NOTE***: ta có thể dùng Burp Intruder để tìm các endpoint ẩn bằng Param miner BApp

# Server-side param pollution
nếu API truyền trực tiếp dữ liệu người dùng vào server-side mà không qua mã hoá, điều này có thể dẫn tới các hành động sau:
- Ghi đè các param
- Sửa đổi các ứng dụng
- Truy cập dữ liệu trái phép
Bạn có thể kiểm tra các loại param pollution thông qua query param, form fields, headers, url path,...
=> Test server-side pollution:
  - Đặt các ký tự như *#*, *&* và *=* vào input và xem xét các respones
  - Truncating query bằng *#*. Ví dụ: `GET /users/search?name=peter#foo&publicProfile=true`
    + Nếu respone trả về user peter, truy vấn có thể đã bị cắt bớt.
    + Nếu thông báo lỗi tên thì có thể application có thể coi 'foo' là một phần của username. Điều này cho thấy truy vấn không bị cắt bớt. 
  - Chèn tham số không hợp lệ. Ví dụ: `GET /userSearch?name=peter%26foo=xyz&back=/home` sẽ dẫn tới `GET /users/search?name=peter&foo=xyz&publicProfile=true`. Nếu respone không thay đổi, có thể `foo=xyz` đã chèn thành công nhưng được hệ thống bỏ qua.
  - Chèn tham số hợp lệ. Ví dụ: `GET /userSearch?name=peter%26email=foo&back=/home` sẽ dẫn tới `GET /users/search?name=peter&email=foo&publicProfile=true`. Tương tự với chèn tham số không hợp lệ, ta có thể tìm thêm thông tin để khai thác.
  - Ghi đè tham số. Ví dụ: `GET /userSearch?name=peter%26name=carlos&back=/home` dẫn tới API nội bộ `GET /users/search?name=peter&name=carlos&publicProfile=true`. Tuỳ vào các hệ thống sẽ xử lý truy vấn trên theo những các khác nhau. Nếu bạn có thể ghi đè `&name=Administrator` thì bạn có thể có quyền của quản trị viên.
