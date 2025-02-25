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

# Overriding the server configuration
Ví dụ, một Apache server sẽ thực thi một file php theo dạng: 
```
LoadModule php_module /usr/lib/apache2/modules/libphp.so
    AddType application/x-httpd-php .php
```
Mỗi server thường có `web.congif` chứa danh sách các tệp cho phép và không cho phép. Nếu bạn có thể upload file script chứa web shell thay đổi danh sách các tệp không cho phép hoặc cho phép, ta có thể đánh lừa máy chủ, đổi extension của file sang dạng `MIME type` có thể thực thi.
- Lab: <https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/insufficient-blacklisting-of-dangerous-file-types/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass>
# Obfuscating file extensions
Vì `MIME type` không thể nhận diện ký tự hoa thường nên ta có thể làm nhiều file extension để bypass qua nó.
- Tạo nhiều file extension. VD: `exploit.php.jpg`
- Thêm các ký tự vào cuối. VD: `exploit.php.`
- Mã hoá URl hoặc thêm byte null. VD: `expoilt%2Ephp` hay `exploit.php;.jpg` hoặc `exploit.php%00.jpg`
- Sử dụng các chuỗi như xC0 x2E, xC4 xAE hoặc Xc) xAE để dịch thành x2E nếu tên tệp dịch dưới dạng UTF-8 rồi sau đó chuyển thành mã ASCII được sử dụng trong file path.
- Đánh lừa hệ thống tự động xoá. VD; Với `exploit.p.phphp` hệ thống sẽ xoá phần `.php` ở giữa và file sẽ thành `exploit.php`.
- etc..

- Lab: <https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/insufficient-blacklisting-of-dangerous-file-types/file-upload/lab-file-upload-web-shell-upload-via-obfuscated-file-extension>

# Flawed validation file contents
- Thay vì tin tưởng `Content-Type` header, các máy chủ sẽ kiểm tra lại file có được cho phép hay không. Trong trường hợp upload function, server sẽ thử kiểm tra các thuộc tính của ảnh như kích thước. Khi bạn upload file script lên, ví dụ như php, nó sẽ coi là không có kích thước và bị từ chối. Ta có thể bypass bằng cách bắt đầu file bằng signature byte của file ảnh. Ví dụ: JPEG sẽ bắt đầu bằng `FF D8 FF`.
- Có một cách nữa là xử dụng các công cụ đặc biệt như ExifTool, nó có thể tạo ra các file ảnh chứa script.
- Lab: <https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/flawed-validation-of-the-file-s-contents/file-upload/lab-file-upload-remote-code-execution-via-polyglot-web-shell-upload>
