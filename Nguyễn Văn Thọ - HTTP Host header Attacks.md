# Nguyễn Văn Thọ - HTTP Host header Attacks



### Bài 1: Basic password reset poisoning

>![](https://i.imgur.com/G6UJFHh.png)

Sử dụng chức năng reset password, ta thấy nó sẽ gửi 1 post request gồm `csrf token` và `username` đến server. Server sẽ generate 1 đường link về cho ta dạng như sau:

```js!
"https://0a610091037d3404c48c8aa900ba00ca.web-security-academy.net/forgot-password?temp-forgot-password-token=0vn9aj648ptatr4VyksKxoVz4AO7vGoZ"
```

Thử thay đổi host name, ta thấy vẫn gửi request đổi mật khẩu thành công, không những thế, trong email nhận về cũng thay đổi host theo. 

Vậy đường link mà ta nhận được lấy giá trị từ host header trong post request thay đổi mật khẩu.


>![](https://i.imgur.com/zjuzavx.png)

Thay đổi giá trị host header thành tên miền của exploit server và đổi giá trị `username` trong request sang carlos, ta trigger được yêu cầu đổi mật khẩu cho carlos.

Đường link mà carlos nhận được sẽ trỏ tới exploit server của ta.

Khi carlos click vào link trong email sẽ tạo request đến exploit server mang theo token change email.

Ta mở access log trên exploit server và lấy token, sau đó đổi thành công mật khẩu của `carlos`

>![](https://i.imgur.com/3PNCHkg.png)

<hr>

### Bài 2: Host header authentication bypass

>![](https://i.imgur.com/4SAJCPF.png)

Ở bài này, vai trò của người dùng được xác định dựa trên `Host header`

Thử truy cập `/admin` ta thấy nó chỉ availabel với local user. 

Đổi Host header thành `localhost` ta thấy truy cập thành công

>![](https://i.imgur.com/8QxTRWV.png)

Thông qua đó có thể xóa user `carlos` dễ dàng


>![](https://i.imgur.com/koMoPz9.png)


<hr>

### Bài 3: Web cache poisoning via ambiguous requests

>![](https://i.imgur.com/UNkTqxv.png)

Ở bài này, ta sẽ modify host header gửi tới server để khi reflected, nó gây XSS. 

Tuy nhiên để khai thác được ở máy user khác, thì ta cần poison web caching của trang web này. Cho dù user có truy cập với host header là gì thì vẫn bị XSS

Sửa đổi host header ta thấy đã bị validate

>![](https://i.imgur.com/IanThbA.png)

Nhưng khi add 1 host header hợp lệ ngay sau fake host header thì ta có thể bypass được validate, và tiêm được vào.

>![](https://i.imgur.com/nvezzZT.png)

Sửa lại host fake bằng host của exploit server của ta, trên exploit server đặt script độc hại, ta khai thác được lab

>![](https://i.imgur.com/e08TCWk.png)


<hr>

### Bài 4: Routing-based SSRF

>![](https://i.imgur.com/aAXyY6F.png)

Thông qua Host header attack để tấn công SSRF

Thư thay đổi host thành địa chỉ của burp colaborator ta thấy nhận được request, vậy là trang web sẽ gửi request tới host mà nó lấy được từ host header.

>![](https://i.imgur.com/TScn1L0.png)

Brute-force địa chỉ IP nội bộ, ta phát hiện được 1 máy nội bộ có địa chỉ IP `192.168.0.228`

>![](https://i.imgur.com/4yrtzR4.png)

Sau đó ta có thể xóa user `carlos`

>![](https://i.imgur.com/QFj5XFz.png)


<hr>

### Bài 5: SSRF via flawed request parsing

>![](https://i.imgur.com/UVKS6Zw.png)

Kiểm tra ta thấy không thể tiêm vào Host header như bài trước, server trả về 403

>![](https://i.imgur.com/GKzw8Kq.png)

Nếu ta thay đường dẫn từ tương đối `/` thành tuyệt đối `https://0a8200e404237675c083f4a500900081.web-security-academy.net/` thì lại nhận mã 200, truy vấn thành công

>![](https://i.imgur.com/epk9ITz.png)

Từ trên ta suy ra server handle các đường dẫn có vấn đề, nếu đường dẫn là tương đối thì nó sẽ validate host header, nếu đường dẫn là tuyệt đối thì nó không validate mà forward request.

>Nó cắt `/abcxyz` và gắn vào host kia.

Từ đó, ta có thể brute-force các địa chỉ IP trong mạng nội bộ, đường dẫn phải là tuyệt đối để bypass

Cuối cùng tìm được địa chỉ IP `192.168.0.201` nằm trong mạng nội bộ

>![](https://i.imgur.com/MdgBJmA.png)

Thông qua đó ta truy cập được `/admin` của trang web hiện tại và xóa user `Carlos`

>![](https://i.imgur.com/IHUYh5Z.png)


<hr>

### Bài 6: Host validation bypass via connection state attack

>![](https://i.imgur.com/NNtZw6u.png)

Ở bài này, đề miêu tả rằng server xác thực Host header nghiêm ngặt, tuy nhiên nó chỉ xác thực header của request đầu tiên, nếu hợp lệ thì các request sau (của cùng session) nó sẽ không check thêm, do đó ta chỉ cần gửi 1 (Vài) request hợp lệ, sau đó có thể dễ dàng thay đổi host để attack.

Sau khi gửi các request hợp lệ, ta tiến hành tấn công và tìm được địa chỉ IP của máy nội bộ như sau:

>![](https://i.imgur.com/MMyQ2Xx.png)

```!
/admin/delete?username=carlos&csrf=DjGimImUsdHI4ncgNaMB6XZt9adLcBzr
```

Kết quả:

>![](https://i.imgur.com/9IY0aP8.png)

<hr>


### Bài 7: Password reset poisoning via dangling markup

>![](https://i.imgur.com/UiCjnau.png)

Bài này sử dụng kĩ thuật dangling markup để đánh cắp password, bên cạnh đó, hint cho ta biết có thể tận dụng tính năng scan email độc hại của mail client.

Sử dụng `forgot password` ta thấy có email được gửi về mail client với nội dung như sau:

>![](https://i.imgur.com/FqTJDau.png)

Server generate new password và gửi về client mà không cho phép user đặt pass mới.

Đường link ở chỗ `click here` dẫn tới trang đăng nhập

Đoán rằng host header được lấy để đưa vào link đó, ta thử thay đổi giá trị header thì thấy không thành công

>![](https://i.imgur.com/6SqZ0Li.png)

Tuy nhiên khi thêm port vào, server vẫn chấp nhận và đính kèm port trong link `click here`, thậm chí nếu port là chữ cái.

>![](https://i.imgur.com/ja7MvFy.png)

Tới đây ta thấy có thể dùng host header attack kết hợp sử dụng kĩ thuật dangling markup, ta tạo 1 thẻ `<a>` mà không đóng `"` cho `href` để có thể lấy luôn cả password mà server generate ra.

>![](https://i.imgur.com/neTPBUy.png)

Khi các tool trên email client thực hiện quét hòm thư kiểm tra các link độc hại, nó sẽ quét trúng link ta tạo, mang theo password và gửi request đến server của ta.

Kiểm tra access log của server, ta thấy được password.

>![](https://i.imgur.com/gRm7meZ.png)


Kết quả:

>![](https://i.imgur.com/6ONCklk.png)


<hr>


