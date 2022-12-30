# ***Nguyễn Văn Thọ - Information Disclosure - Business logic vulnerabilities***

<hr>

## Information Disclosure
### Bài 1. Information disclosure in error messages
>![](https://i.imgur.com/Ndf1A4b.png)

Mô tả: Trang web hiển thị error message khi có các truy vấn sai logic, kết quả được trả về người dùng. Mục tiêu của ta là khai thác để trigger error message và lấy version number của framework

Truy cập vào 1 sản phẩm bất kì, ta thấy đường dẫn có chứa tham số `productId` như sau
https://blabla.web-security-academy.net/product?productId=1
Thử thay giá trị của nó bằng chuỗi bất kì, ví dụ `abc`. Ta thấy trả về trang báo lỗi như sau
>![](https://i.imgur.com/c5vujpm.png)

Ở dòng cuối có phiên bản framework, ta có thể dùng nó để submit và solve bài lab
>![](https://i.imgur.com/BGXtNLK.png)

<hr>

### Bài 2. Information disclosure on debug page
>![](https://i.imgur.com/qqtMZ2L.png)

Trang web bật debug, làm lộ nội dung nhạy cảm có chứa secret key, mục tiêu của ta là tìm ra nó và khai thác.

Xem mã HTML của trang web, ta thấy có dòng này
>![](https://i.imgur.com/SY865LA.png)

Truy cập vào đường dẫn đó ta thấy trả về trang web debug
>![](https://i.imgur.com/BxcWlqQ.png)

Và có secret key
>![](https://i.imgur.com/hy3FIhd.png)

submit nó là solve bài lab
>![](https://i.imgur.com/KZGn1ON.png)

<hr>

### Bài 3. Source code disclosure via backup files
>![](https://i.imgur.com/Gvk0RLJ.png)

Trang web lộ source code thông qua backup file trong thư mục ẩn. Mục tiêu là tìm ra thư mục đó và submit database password

Vào đường dẫn `/backup` ta thấy trả về như sau:
>![](https://i.imgur.com/3KB3u3s.png)

Click vào đường link trên, ta thấy lộ source như sau
>![](https://i.imgur.com/jhANdaR.png)

Chú ý ta thấy có phần connect SQL như sau
>![](https://i.imgur.com/CJdsZus.png)

Lấy được password, ta submit là xong
>![](https://i.imgur.com/79INNKI.png)

<hr>

### Bài 4. Authentication bypass via information disclosure
>![](https://i.imgur.com/Yz8cAaz.png)

Bài lab cho biết rằng trang web chứa lỗ hổng xác thực (authen) nhưng mà để khai thác cần phải biết 1 HTTP header mà tác giả custom.
Truy cập trang web và dirsearch, ta thấy có đường dẫn `/admin`. Tuy nhiên đã bị từ chối truy cập 
>![](https://i.imgur.com/Rb3sF9H.png)

Em thử dùng `X-Forwarded-For` header với giá trị là `127.0.0.1` để giả truy cập từ local. Nhưng thất bại
Theo như đề, trang web xác thực bằng 1 trường HTTP header custom, để biết được trường đó, ta cần xem tại server, gói tin của ta được nhận như thế nào, điều này có thể thực hiện với phương thức `TRACE`
Dùng Burp Proxy để thay đổi phương thức request, ta có kết quả như sau:
>![](https://i.imgur.com/zwKVA0P.png)

Chú ý ta thấy rằng có trường `X-Custom-IP-Authorization` với giá trị là địa chỉ IP của ta, vậy là server lấy địa chỉ IP của ta, thêm vào http header 1 trường custom như vậy và xử lý dựa theo trường này.
Sửa lại request GET của ta, ta thêm trường đó vào, với giá trị là 127.0.0.1 để giả mạo đang ở local. Kết quả server đã nhận nhầm IP của ta (khi trường này đã có giá trị thì không set lại)
>![](https://i.imgur.com/fvXbIx4.png)

>![](https://i.imgur.com/p9fq3iF.png)

Delete user `carlos` (trong http request header phải có trường này và giá trị là địa chỉ IP local) là ta solve được bài lab
>![](https://i.imgur.com/MVVTy0u.png)

<hr>

### Bài 5. Information disclosure in version control history
>![](https://i.imgur.com/oPMM0Fq.png)

Mô tả: trang web lộ thông tin nhạy cảm trong version history mà ta có thể thấy ở thư mục `/.git/`
Dùng `wget -r https://blabla/.git` em download được source của trang `/.git` về máy local.
Dùng lệnh `git log` để xem lịch sử chỉnh sửa của git
>![](https://i.imgur.com/cu6ZX9e.png)

Ta thấy có commit `Remove admin password from config`
Dùng lệnh `git show <commit ID>` ta thấy có 1 dòng admin password đã bị xóa đi ở phiên bản trước
>![](https://i.imgur.com/MwCDCa6.png)

Dùng nó để login ta thấy vào tài khoản admin thành công, và solve bài lab
>![](https://i.imgur.com/28Icc5H.png)

<hr>

<hr>

## Business logic vulnerabilities

### Bài 1: Excessive trust in client-side controls
>![](https://i.imgur.com/lpd9Aa9.png)

Mô tả: trang web chứa lỗ hổng logic cho phép người dùng mua các sản phẩm với giá trị vượt quá số dư. Để solve bài lab, mua sản phẩm "Lightweight l33t leather jacket"

Tiến hành mua hàng như một người bình thường, sau đó kiểm tra phần history các request bắt được trên burp proxy, em thấy có một request khi thêm sản phẩm vào giỏ hàng sẽ kèm theo giá sản phầm
>![](https://i.imgur.com/D7ngbDj.png)

Thay đổi giá trị của nó thành số 1 và gửi lại request đó, em thấy kết quả sản phẩm đã nằm trong giỏ hàng với giá 0.01$
>![](https://i.imgur.com/Nl8r00J.png)

Giờ thì ta có thể tiến hành thanh toán sản phẩm khi giá của nó bé hơn số dư hiện có của ta
>![](https://i.imgur.com/cHy28WP.png)

<hr>

### Bài 2: High-level logic vulnerability
>![](https://i.imgur.com/J1hieYP.png)

Bài lab này cũng giống bài trên, tuy nhiên, qua tiêu đề của nó, ta biết để khai thác logic của nó cần nhiều kĩ thuật hơn.
Đầu tiên, ta sẽ mua hàng như user bình thường và ghi lại các request thông qua burp proxy

Kiểm tra, ta thấy giá của sản phẩm không còn được post lên server nữa, chỉ có `productId` và `quatity`
>![](https://i.imgur.com/uNkNlLo.png)

Thử sửa số lượng từ 1 thành -1, ta thấy trong giỏ hàng xuất hiện sản phẩm như sau:
>![](https://i.imgur.com/jTpFRGG.png)

Giá sản phẩm được tính bằng `quantity`*`Giá`, nên đã để xuất hiện trường hợp giá là số âm, bé hơn số dư của ta và ta có thể mua.
Tuy nhiên, server check và biết rằng giá nhỏ hơn 0 nên ta không thể mua, và thêm điều nữa là nếu số lượng sản phẩm bé hơn 0 nó cũng sẽ không được mua, vì thế cần thêm 1 số sản phẩm vào để giá về số lớn hơn 0 và có đúng 1 cái Jacket như hình sau
>![](https://i.imgur.com/riFytRj.png)

>![](https://i.imgur.com/MdTbLUQ.png)

Giờ thì ta có thể thanh toán và solve bài lab.
>![](https://i.imgur.com/1gJjd7v.png)

<hr>

### Bài 3: Inconsistent security controls
>![](https://i.imgur.com/xP08LQu.png)

Bài lab chứa lỗ hổng logic cho phép người dùng truy cập vào site admin mà đáng nhẽ chỉ có nhân viên mới truy cập được. Mục tiêu là khai thác và xóa user `carlos`
Xem tổng thể các request thì em không tìm được cách để làm giả việc đăng ký thành user với email `@dontwannacry.com`
Tuy nhiên trang web có chức năng update email cho phép thay đổi bản ghi email trong database mà không xác thực ở chức năng này. Vì thế sau  khi đăng nhập thành công, em update email thành `test@dontwannacry.com` và biến thành employee của công ty. 
>![](https://i.imgur.com/oyRKOTi.png)

Và do đó có thể truy cập admin panel thành công.
>![](https://i.imgur.com/AbDncVH.png)

<hr>

### Bài 4: Flawed enforcement of business rules
>![](https://i.imgur.com/MODoSDo.png)

Trang web chứa lỗ hổng trong khâu thanh toán, mục tiêu của ta là exploit để mua sản phẩm `Lightweight l33t leather jacket`

Đầu tiên, khi vào trang web, ta sẽ thấy thông báo về Coupon giảm giá `NEWCUST5`
>![](https://i.imgur.com/R5JejFd.png)

Thử khai thác với coupon này không được, ta tìm kiếm thêm và thấy ở cuối trang có chức năng `Sign up to our newsletter!`, sau khi signup sẽ được coupon giảm hơn 400$
>![](https://i.imgur.com/75Y3UQ2.png)

Khai thác với coupon thứ 2 này ta thấy vẫn không được, server check xem ta đã sử dụng coupon hay chưa trước khi giảm giá. 
Tuy nhiên nếu dùng xen kẽ như hình sau thì lại bypass được chức năng kiểm tra của server, server chỉ check coupon cuối cùng mà ta nhập vào.
>![](https://i.imgur.com/cZqMzQh.png)

Kết quả là ta có thể mua sản phẩm với giá 0$
>![](https://i.imgur.com/ycpMPAq.png)

<hr>

### Bài 5: Low-level logic flaw
>![](https://i.imgur.com/LR20nvk.png)

Đề bài giống các bài trên, nên ta thực hiện kiểm tra để xem trang web có những chức năng nào.
Các kiểu tấn công trong các bài lab trên đều không thực hiện được ở đây.
Tấn công integer overflow ta thấy tối đa chỉ có thể gửi tham số `quantity` có giá trị là 99, nếu lớn hơn sẽ không hợp lệ, vì vậy ta cần gửi mỗi request tối đa 99 sản phẩm áo Jacket. Dùng burp intruder để tự động hóa quá trình đó đến khi ta thấy số tiền bắt đầu tràn về số âm.
>![](https://i.imgur.com/Z6b5nVY.png)

Thêm lại 1 số sản phẩm để số tiền trở lại số dương
>![](https://i.imgur.com/ePKtvFR.png)

Số tiền giờ đã bé hơn 100 và vẫn là số dương, ta có thể mua sản phẩm như yêu cầu và solve bài lab.
>![](https://i.imgur.com/Mtc6BIS.png)

<hr>

### Bài 6: Inconsistent handling of exceptional input
>![](https://i.imgur.com/HtgReKg.png)

Bài lab miêu tả rằng các chức năng đang bị lỗi trong việc handle input không thống  nhất với nhau, công việc của ta là tìm cách bypass để đăng nhập với tài khoản có email `@dontwannacry.com`

Trang web chỉ có duy nhất một chức năng liên quan đến việc xử lý email input là chức năng `Register`
Và Email Client của ta sẽ tổng hợp tất cả các mail gửi đến subdomain của nó
>![](https://i.imgur.com/MhjfKdn.png)

Ta thử các payload để server xử lý lỗi như `test@dontwannacry.com%00.<email-client-id>.web-security-academy.net` nhưng không thành công.
Thử đoán rằng giá trị email trên server có giới hạn số ký tự nhất định, em thử nhập payload dài nhất có thể vào thì thấy kết quả như sau
>![](https://i.imgur.com/GQrwGfJ.png)

Phần email domain đã bị cắt mất. Tính toán lại vài lần em biết trường này có giá trị là 256 ký tự, và server sẽ thực hiện gửi link xác thực trước khi nó lưu vào database, nên ta hoàn toàn có thể nhập một email giả nhưng Email Client vẫn nhận được link xác thực

Em thử lại với email như sau `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@dontwannacry.com.exploit-0a1d006d041ee5bac093995801eb00ea.exploit-server.net`

>![](https://i.imgur.com/AN6EQSZ.png)

>![](https://i.imgur.com/k01G3Hn.png)

Vậy là email của ta đã bị cắt thành 1 email thuộc domain `@dontwannacry.com`

>![](https://i.imgur.com/bIyoxgk.png)

<hr>

### Bài 7: Weak isolation on dual-use endpoint
>![](https://i.imgur.com/zIYXhZt.png)

Bài lab xác thực quyền người dùng thông qua input của chính họ, dẫn đến user có thể có quyền của admin. Mục tiêu là truy cập vào `administrator` và xóa user `carlos`

Kiểm tra ta thấy trang web có chức năng thay đổi password, thử sử dụng nó và bắt các request, ta thấy tham số username được gửi lên server và server dùng nó để biết cần thay đổi password của người dùng nào. Vậy ta có thể giả mạo giá trị của tham số này để thay đổi password của người dùng khác.
`csrf=HHw00ayFTefb3Jx4Da5T25DpLO7g3Qt6&username=wiener&current-password=peter&new-password-1=peter&new-password-2=peter`

Để thay đổi password của user khác (`administrator`) ta buộc cần phải biết `current-password`. Thử để trống giá trị này vẫn thấy server báo lỗi.

Tuy nhiên nếu không set giá trị này thì, server sẽ không trải qua bước xác thực mà thay đổi password thẳng luôn.
>![](https://i.imgur.com/NRX5U3E.png)

Thành công thay đổi password của admin, ta đăng nhập và xóa user như yêu cầu.
>![](https://i.imgur.com/VW7PugR.png)

<hr>

### Bài 8: Insufficient workflow validation
>![](https://i.imgur.com/WM5q3fh.png)

Theo mô tả, tragn web chứa lỗ hổng trong chức năng thanh toán, để solve bài lab, ta sẽ mua `Lightweight l33t leather jacket` với giá tiền nhiều hơn budget của ta

Sau khi test mua số lượng sản phẩm lớn (~16k sản phẩm), ta thấy nó bắt đầu reset số sản phẩm về lại 0. Nhưng chẳng khai thác được gì thêm.

Sau khi không khai thác được bằng các cách cũ, em tiến hành mua hàng như một user bình thường, và phát hiện thấy khi thanh toán sẽ được trả về một mã redirect 303 đến đường dẫn `/cart/order-confirmation?order-confirmed=true`
>![](https://i.imgur.com/BAf8y2A.png)

Ở đây có tham số `order-confirmed` được đính kèm, ta thử đổi giá trị của nó từ `true` thành `false` thì thấy sản phẩm vẫn `on the way` nhưng tiền không bị trừ.
>![](https://i.imgur.com/osNBIID.png)

Vậy là sản phẩm vẫn được mua nhưng tiền không được tính, ta thay sản phẩm bằng `Lightweight l33t leather jacket` trong cart. Và truy vấn GET tới đường dẫn order-confirmation như trên, sản phẩm sẽ được mua, và tiền không ảnh hưởng.
>![](https://i.imgur.com/86MlY3P.png)

<hr>

### Bài 9: Authentication bypass via flawed state machine
>![](https://i.imgur.com/T1rnELj.png)

Trang web chứa lỗ hổng xác thực trong quá trình đăng nhập, ta cần khai thác và đăng nhập vào trang admin, xóa user `carlos`

Sau khi đăng nhập, ta thấy được redirect tới trang `role-selector`
>![](https://i.imgur.com/tODmvRX.png)

Thử inspect và đổi giá trị của role, ta thấy bị server bỏ qua request.
Thử bỏ qua quy trình này, đổi đường dẫn từ `/role-selector` thành `/`, ta thấy vẫn không có kết quả mong muốn.

Tuy nhiên nếu drop request redirect đó thì ta sẽ được chuyển về role mặc định là `admin` và có thể solve bài lab
>![](https://i.imgur.com/xGFUoMa.png)

<hr>

### Bài 10: Infinite money logic flaw
>![](https://i.imgur.com/ZMNnB5b.png)

Trang web chứa lỗ hổng trong chức năng thanh toán, ta cần exploit và mua sản phẩm `Lightweight l33t leather jacket`

Ở cuối trang ta thấy sau khi đăng ký sẽ được mã giảm giá `SINGUP30` giảm 30% đơn hàng của ta
>![](https://i.imgur.com/UL74ayQ.png)

Bên cạnh đó, ta thấy có thể mua gift-card giá 10$ và đổi thưởng ở trang `my-account` để lấy lại 10$
Vậy cứ mỗi đơn hàng `gift card` ta tốn 7$ (discount 30%) và thu về 10$ sau khi `redeem`. Vậy chỉ cần ta tự động hóa quá trình này thì tiền trong tài khoản sẽ tăng lên không ngừng.
Sử dụng `macro` trong burpsuit để gửi các request theo từng bộ `mua gift-card` -> `sử dụng counpon` -> `GET gift-card` -> `redeem gift-card`, ta thành công khai thác.
>![](https://i.imgur.com/sofWxuV.png)

<hr>

### Bài 11: Authentication bypass via encryption oracle
>![](https://i.imgur.com/1VHewej.png)

Bài lab chứa lỗ hổng làm lộ mã hóa bí mật tới người dùng, khai thác lỗ hổng và đăng nhập dưới tài khoản admin và xóa user `carlos`

Vào trang login, ta thấy có option stay-logged-in, thử đăng nhập với tùy chọn đó, ta thấy trang web trả về cookie `stay-logged-in` với giá trị gì đó đã được mã hóa
>![](https://i.imgur.com/aU7nbsj.png)

Sử dụng các chức năng của trang web và theo dõi response trả về trên Burp Proxy History, ta thấy nếu comment chứa `email` không hợp lệ, trang web sẽ gán thêm cho ta một cookie `notification` cũng ở dạng mã hóa
>![](https://i.imgur.com/PCZVjf6.png)

Kiểm tra kĩ hơn thì thấy nội dung trang web được bổ sung thêm 1 dòng như sau ở đầutrang
>![](https://i.imgur.com/Sn26m02.png)

Vì cookie có tên là `notification` nên ta đoán nó có liên quan đến dòng chữ `Invalid email address ...` kia, thử thay nó bằng giá trị cookie `stay-logged-in` của ta, dòng chữ đã thay đổi thành
>![](https://i.imgur.com/QcISrCr.png)

Định dạng `username:time`

Vậy nếu nhập email dạng: `administrator:time` thì ta được một mã encrypted cùng dạng với cookie của ta.

Tuy nhiên không bypass được authentication, dòng chữ "Invalid email address: " được thêm vào trước khi mã hóa, nên giá trị mà ta dùng làm cookie `stay-logged-in` đó đã dư một số byte.
>![](https://i.imgur.com/IcQx3Qn.png)

Để loại bỏ nó, ta cần xác định số byte mà chuỗi "Invalid email address: " chiếm - 23 Bytes
Urldecode -> Base64Decode ta có được chuỗi hex mà chương trình encrypt ra, loại bỏ 23 Bytes đầu đi, mã hóa Base64Encode -> Urlencode và thay vào lại trong cookie `notification` để kiểm tra thì ta nhận thông báo lỗi
>![](https://i.imgur.com/V6uVH53.png)

Theo đó, kiểu mã hóa mà server sử dụng mã hóa theo từng block 16 Bytes, việc ta xóa 23 Bytes đã làm số Bytes còn lại lẻ. Vì vậy, cần xóa một số bội của 16 và lớn hơn 23, ta chọn 32.

Để có thể xóa 32 Bytes mà ko ảnh hưởng đến chuỗi `administrator:time` thì ta cần đệm thêm 9 Bytes vào trước nó thành `000000000administrator:time`
Thực hiện lại các bước như cũ, nhưng xóa 32 Bytes thay vì 23 Bytes, ta đã có được chuỗi như hình, tức giá trị của `notification` cookie lúc này có thể được dùng như `stay-logged-in` cookie của admin
>![](https://i.imgur.com/g9f0VWS.png)

Xóa `session` cookie và refresh lại trang, ta đã ở trong trang của admin, xóa user `carlos` và solve bài lab
>![](https://i.imgur.com/K5HvMx4.png)

<hr>

