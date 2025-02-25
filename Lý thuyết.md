# File-upload-vulnerabilities 
Hiểu đơn giản, thông qua việc trang web cho chúng ta upload file lên, hacker sẽ sử dụng các thủ thuật nằm trong file upload nhằm khiến hệ thống thực hiện các hành động không mong muốn. Thậm chí, ta có thể lấy được quyền kiểm soát hệ thống!

# How do web servers handle requests for static files?
- Theo truyền thống, các đường dẫn của request thường được ánh xạ 1-1 với hệ thống tệp của máy chủ mà không có sự kiểm duyệt. Ngày nay, hệ thống hiện đại hơn, thường các đường dẫn được đổi tên, lưu trữ cục bộ, kiểm duyệt,... trước khi được upload lên. 

- Quy trình xử lý các tệp tĩnh đa số giống nhau khi máy chủ sẽ phân tích cú pháp đường dẫn rồi yêu cầu xác định extension của tệp - thường là MIME type hoặc accept list
  + Nếu tệp không thể thực thi, máy chủ sẽ chỉ gửi nội dung tệp đến client trong Response.
  + Nếu tệp có thể thực thi ví dụ như .php và máy chủ được cấu hình để chạy loại tệp này, chúng sẽ được thực thi và kết quả sẽ được trả về client trong Response.
  + Nếu tệp có thể thực thi nhưng máy chủ không được cấu hình để chạy loại tệp này, thông thường sẽ trả về lỗi nhưng trong một số trường hợp, kết quả thực thi tệp vẫn được cung cấp cho máy khách dưới dạng văn bản thuần tuý.
**Tip**: `Content-Type` cung cấp loại tệp mà máy chủ có thể thực thi. Ngoài ra, có thể tham khảo thêm cả `MIME type`.

# Exploiting file upload by web shell
Nếu một trang web cho phép bạn tải lên một tệp php, java hay python... và cũng được cấu hình để chạy, bạn có thể tạo web shell và tiến hành khai thác. 
- Vậy web shell là gì?
  Web shell là tập lệnh độc hại cho phép attacker kiểm soát máy chủ hoặc khiến máy chủ thực hiện những hành động không mong muốn bằng cách gửi request HTTP đến endpoint của api.
  VD với php: `<?php echo file_get_contents('/path/to/target/file'); ?>`
  web shell này sẽ trả về nội dung của file đường dẫn trong response.
- Lab: <https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/exploiting-unrestricted-file-uploads-to-deploy-a-web-shell/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload>

  # Flawed file type validation
  Khi gửi một HTML forms, trình duyệt sẽ cung cấp data trong một `POST` request với dạng `Content-Type` là `application/x-www-form-url-encoded`. Nếu gửi một ảnh hay một lượng lớn binary data hoặc PDF document,... thì `Content-Type` sẽ là `multipart/form-data`. Ví dụ, khi ta gửi một ảnh, browser sẽ gửi một request như sau:
  ```
  POST /images HTTP/1.1
    Host: normal-website.com
    Content-Length: 12345
    Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="image"; filename="example.jpg"
    Content-Type: image/jpeg

    [...binary content of example.jpg...]

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="description"

    This is an interesting description of my image.

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="username"

    Ceokon
    ---------------------------012345678901234567890123456--
  ```
Ta có thể thấy mỗi phần sẽ có một `Content-Disposition` header và mỗi phần cũng sẽ có một `Content-Type` riêng. Cách duy nhất để website có thể kiểm tra file upload là kiểm tra định dạng `Content-Type` có trùng với `MIME type` không. Ví dụ, server chỉ cho phép ảnh thì nó cũng chỉ cho phép `Content-Type` dạng `image/jpg` và `image/png`.
Lỗi ở đây sẽ xảy ra khi máy chủ ngầm định xác nhận trong các request. Nếu không xác thực lại sau mỗi lần gửi, thì biện pháp này có thể bị bỏ qua dễ dàng.
- Lab1:<https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/exploiting-flawed-validation-of-file-uploads/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass>

Tuy nhiên, với lỗi này website có thể phòng ngừa bằng cách chỉ chạy tập lệnh có loại `MIME type` được cấu hình rõ ràng để thực thi. Nếu không chúng chỉ báo lỗi hoặc trả về nội dung của tệp dưới dạng văn bản thuần tuý. Nhưng loại cấu hình này thường khác nhau giữa các thư mục. Một thư mục cho phép người dùng tải lên sẽ có nhiều biện pháp hơn các vị trí khác. Bạn có thể thử upload file script lên một vị trí mà hệ trống không cho phép upload file người dùng. Có thể hệ thống sẽ thực thi file script đó
- Lab2:<https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/preventing-file-execution-in-user-accessible-directories/file-upload/lab-file-upload-web-shell-upload-via-path-traversal>
#
