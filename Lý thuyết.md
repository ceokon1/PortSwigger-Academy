# Khái niệm
  SQLi là một lỗ hổng bảo mật mà hacker sẽ dùng các truy vấn để tìm những data mà chúng ta không muốn public phục vụ mục đích của chúng. Trong nhiều trường hợp, một attacker có thể sửa hoặc xoá data, dẫn tới thiệt hại cho hệ thống.  
  Ngoài ra, attacker còn có thể dùng SQL injection để khai thác những thứ mà server ẩn đi. Nó cũng có thể dẫn tới DOS attack

# SQL injection có thể lấy những gì
  Một cuộc tấn công SQLi thành công có thể lấy được những data nhạy cảm như:
- Password  
- Thông tin thẻ Credit  
- Thông tin cá nhân
- Tìm được cách xâm nhập vào hệ thống tổ chức

# Làm thế nào để nhận dạng SQLi
  Bạn có thể nhận dạng thông qua những kỹ thuật kiểm thử ở đầu vào của hệ thống. Nó thường hoạt động như sau:  
  - Một dấu '
  - Một số các syntax đặc biệt của SQL như --, #, ...
  - Biểu thức so sánh như `or 1 = 1` hay `or 1 = 2`, và tìm những khác biệt trong responses của hệ thống
  v.v...
