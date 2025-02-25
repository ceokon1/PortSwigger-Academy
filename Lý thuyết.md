# File-upload-vulnerabilities 
Hiểu đơn giản, thông qua việc trang web cho chúng ta upload file lên, hacker sẽ sử dụng các thủ thuật nằm trong file upload nhằm khiến hệ thống thực hiện các hành động không mong muốn. Thậm chí, ta có thể lấy được quyền kiểm soát hệ thống!

# How do web servers handle requests for static files?
- Theo truyền thống, các đường dẫn của request thường được ánh xạ 1-1 với hệ thống tệp của máy chủ mà không có sự kiểm duyệt. Ngày nay, hệ thống hiện đại hơn, thường các đường dẫn được đổi tên, lưu trữ cục bộ, kiểm duyệt,... trước khi được upload lên. 

- Quy trình xử lý các tệp tĩnh đa số giống nhau khi máy chủ sẽ phân tích cú pháp đường dẫn rồi yêu cầu xác định extension của tệp - thường là MIME type hoặc accept list
  + Nếu tệp không thể thực thi, máy chủ sẽ chỉ gửi nội dung tệp đến client trong Response.
  + Nếu tệp có thể thực thi ví dụ như .php và máy chủ được cấu hình để chạy loại tệp này, chúng sẽ được thực thi và kết quả sẽ được trả về client trong Response.
  + Nếu tệp có thể thực thi nhưng máy chủ không được cấu hình để chạy loại tệp này, thông thường sẽ trả về lỗi nhưng trong một số trường hợp, kết quả thực thi tệp vẫn được cung cấp cho máy khách dưới dạng văn bản thuần tuý.
**Tip**: Content-Type cung cấp loại tệp mà máy chủ có thể thực thi. Ngoài ra, có thể tham khảo thêm cả MIME type.

# Exploiting file upload by web shell
Nếu một trang web cho phép bạn tải lên một tệp php, java hay python... và cũng được cấu hình để chạy, bạn có thể tạo web shell và tiến hành khai thác. 
- Vậy web shell là gì?
  Web shell là tập lệnh độc hại cho phép attacker kiểm soát máy chủ hoặc khiến máy chủ thực hiện những hành động không mong muốn bằng cách gửi request HTTP đến endpoint của api.
  VD với php: <?php echo file_get_contents('/path/to/target/file'); ?>
  web shell này sẽ trả về nội dung của file đường dẫn trong response.
- Lab: <https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/exploiting-unrestricted-file-uploads-to-deploy-a-web-shell/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload>
