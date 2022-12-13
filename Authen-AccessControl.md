## ***Nguyễn Văn Thọ - Burp Suite Lab***
# Authentication
### Bài 1: Username enumeration via different responses
Yêu cầu của đề bài là brute-force usename và password, biết rằng đã cho trước 2 list để brute-force
>![](https://i.imgur.com/KYGzj0M.png)

Đầu tiên, em dùng Burp Suite Intruder để thực hiện quét qua list Username.Với mọi request đăng nhập thất bại, trang web sẽ trả về ```Invalid Username```; Nếu đúng Username thì trang web trả về ```Incorrect password```. Dựa vào đây em lọc kết quả trả về và biết được Username là **al**

Sau đó, tiếp tục thực hiện scan với list password mà đề cung cấp. Bây giờ ta đã biết Username rồi, chỉ cần brute-force password. Lọc các kết quả trả về để lấy kết quả có http status code là 3xx thì ta có được password là **thomas**
>![](https://i.imgur.com/CJgZldO.png)
<hr>

### Bài 2: 2FA simple bypass
Bài này, đề đã cung cấp cho ta tài khoản và mật khẩu đăng nhập, nhiệm vụ của ta là bypass 2-Factor-Authentication 
>![](https://i.imgur.com/rUnEk41.png)

Đầu tiên, thử vào tài khoản của chính mình là ```wiener:peter```, em thấy rằng sau khi đăng nhập sẽ hiện ra ô điền 2FA code như sau:
>![](https://i.imgur.com/UO712Vm.png)

Mở Email Client lên thì có nhận được code để đăng nhập, là một số 4 chữ số.
>![](https://i.imgur.com/Vqb9JJt.png)

Vậy là số khá nhỏ, có thể brute-force. Em thực hiện brute-force giá trị 2FA từ 0000 đến 9999.
>![](https://i.imgur.com/3ljJ5tQ.png)

Trong lúc đợi em thử xóa route trên đường dẫn thì thấy có thể vào trang new feed mà không cần xác thực gì nữa, vậy là ta không cần phải brute-force mà vẫn có thể dễ dàng bypass 2FA.
**Kết quả:**
>![](https://i.imgur.com/aDa8pO8.png)
<hr>

### Bài 3: Password reset broken logic
Bài này đề mô tả rằng chức năng ```reset password``` của trang web đang bị lỗi
>![](https://i.imgur.com/3bm4SbW.png)
Thử đăng nhập bằng tài khoản của chính mình và sử dụng chức năng này, ta thấy nó sẽ gửi về email của ta một link reset password.
>![](https://i.imgur.com/SVZ9K9S.png)
Dùng Burp Suite để theo dõi hoạt động của chức năng này ta lưu ý những thứ sau:
- Có token đính kèm trong URL của truy vấn
- Khi submit form, thì 4 tham số sau được gửi đi: ```temp-forgot-password-token```, ```username```, ```new-password-1```, ```new-password-2```

Sau khi form submit thì được redirect tới trang chủ mà không cần đăng nhập lại
Thử lại request trên, đầu tiên t thay username từ ```wiener``` thành ```carlos``` thì kết quả là được luôn.

Cầm password mới thay đổi để đăng nhập, ta truy cập được vào tài khoản của ```carlos``` và hoàn thành bài lab
>![](https://i.imgur.com/Jx1G8eJ.png)

<hr>

### Bài 4: Username enumeration via subtly different responses
Nhiệm vụ của bài này giống bài số 1, là brute-force username, password từ list cho trước.
>![](https://i.imgur.com/qbMvB3y.png)

Tuy nhiên, kết quả trả về của bài này khi đăng nhập sai là sẽ ```Invalid username or password``` chứ không phải chỉ rõ là sai cái nào như bài 1, vì thế ta chỉ có cách brute-force username và password cùng lúc (tầm 10100 requests).

Tuy nhiên đây là challenge ```subtly different``` tức khác biệt nhỏ, nên em đoán là kết quả trả về sẽ có chút khác nhau rất nhỏ giữa đúng và sai.
Thử brute-force username và quan sát kết quả trả về, ta nhận thấy có 1 request với username = **at** có kết quả trả về khác với các response khác ở dấu **.** đằng sau ```Invalid username or password```

Thử brute-force password với username này, ta có kết quả là tìm được mật khẩu **dallas**

Hình ảnh minh chứng
>![](https://i.imgur.com/YicphiD.png)

<hr>

### Bài 5: Username enumeration via response timing
Như bài 4 và bài 1, ta cũng sẽ tiếp tục thực hiện brute-force username và password dựa trên list cho sẵn. Cách thức là phân tích thời gian response của các request để biết đã brute-force đúng hay chưa.
>![](https://i.imgur.com/yRxJCIN.png)

Bên cạnh đó, challenge cũng bổ sung thêm chặn brute-force dựa trên IP.
>![](https://i.imgur.com/gwudqEf.png)

Đầu tiên, để bypass chặn brute-force thì ta có thể dùng X-Forwarded-For, phần địa chỉ IP ta sẽ random liên tục (nhờ chức năng brute-force trong intruder)
Quan sát kết quả trả về, ta thấy có một request (username = **al**) mà thời gian phản hồi của nó gấp 4 lần các request khác, em đoán đây là username.
>![](https://i.imgur.com/todYHl2.png)

Việc còn lại là brute-force password với username này, em có kết quả là **12345**

Hình ảnh minh chứng
>![](https://i.imgur.com/YZzyO3j.png)

<hr>

### Bài 6: Broken brute-force protection, IP block
Cũng là một bài brute-force password, ta đã biết Username là ```carlos```
Thử thách của bài này là server sẽ chặn IP nếu người dùng đăng nhập sai quá 3 lần. Để unlock thì cần đăng nhập đúng tài khoản là được.
Ta đã có sẵn tài khoản valid là `wiener:peter`, ta sẽ chèn nó vào giữa các request brute-force của mình, như sau:
>![](https://i.imgur.com/fJmRRoY.png)
Sau cùng ta thu được password cho account **carlos** là **1quaz2wsx**

>![](https://i.imgur.com/8KSnJW2.png)

<hr>

### Bài 7: Username enumeration via account lock

Ở bài này, server thực hiện các biện pháp mạnh mẽ hơn, là chặn không cho đăng nhập thêm, nếu người đó có các hành vi đáng ngờ (thường là đăng nhập fail nhiều lần)

Đầu tiên, ta tìm valid username bằng cách brute-force qua mỗi username với null password, sẽ có duy nhất 1 tài khoản nhận được error message `You have made too many incorrect login attempts. Please try again in 1 minute`. Tới đây ta tìm được tài khoản là `akamai`, error message cho biết tài khoản đăng nhập sai nhiều lần và đã bị tạm thời khóa.

Mang tài khoản này đi brute-force với password list, tất cả các response đề báo error message như trên, duy chỉ có 1 response cho kết quả đăng nhập thành công sẽ không chứa câu đó. Lọc ra ta thấy nó có giá trị là `chelsea`
Đăng nhập vào web và thành công.
>[color=#5753c1] Ban đầu em nghĩ phải gửi request brute-force cách nhau 1 phút để đợi tài khoản được mở khóa rồi đăng nhập tiếp. Tổng cộng ta sẽ cần đợi tối đa 100p. Tuy nhiên xem solution thì thấy không cần phải vậy..

>![](https://i.imgur.com/FrIt2Eq.png)

<hr>

### Bài 8: 2FA broken logic
Bài này cũng tương tự như bài 2FA phía trên, sau khi đăng nhập ta sẽ được yêu cầu nhập tiếp Authentication code được gửi qua mail.
>![](https://i.imgur.com/4lYXaDi.png)
Vậy để đăng nhập vào tài khoản của `carlos` ta cần phải biết 2 thứ đó là password và authentication code trong email của `carlos`
Đầu tiên, quan sát hoạt động của server thông qua việc đăng nhập tài khoản `wiener`, ta thấy khi gửi authentication code đi, nó có đính kèm username trong cookie và kết quả trả về là redirect tới trang chủ
>![](https://i.imgur.com/EZpHcOm.png)

Vậy là server sẽ xác thực qua mã code cùng với cookie này.
Về phần mã code thì ta có thể brute-force, nhưng làm sao để nó sinh ra code khi ta không biết mật khẩu của `carlos`. Thử kiểm tra lại kĩ hơn, ta thấy sau khi đăng nhập, sẽ có 1 GET request đến `/login2` để yêu cầu server generate ra 2FA code cho username trong cookie
>![](https://i.imgur.com/Pf8OTT9.png)
Chỉ cần đổi mục giá trị `verify` trong cookie là ta có thể yêu cầu server sinh ra 2FA code cho `carlos` mà không cần đăng nhập trước.

Em thực hiện brute-force từ  0 - >10.000 thì ra kết quả tìm được một request có http response status code là 302, tức brute-force thành công.
>![](https://i.imgur.com/MqCyxmd.png)

Gửi 1 post request với mã tìm được này đến server, ta sẽ được redirect tới account của `carlos`, vậy là thành công bypass
>![](https://i.imgur.com/jium6Ng.png)

<hr>

### Bài 9: Brute-forcing a stay-logged-in cookie
Như tiêu đề của bài lab, đây sẽ là một bài mà ta thực hiện brute-force session cookie của user.
>![](https://i.imgur.com/bKrDQHN.png)
Để xem cookie đó trông như thế nào, ta sẽ login vào tài khoản được cấp
>![](https://i.imgur.com/AG2RWpQ.png)
Decode base64 với giá trị trên thì ta có
>![](https://i.imgur.com/XFmJVAQ.png)
Vậy nó sẽ là tên user cùng với dấu `:` và một chuỗi đằng sau
Đây là mã hex của 1 chuỗi nào đó (do chỉ toàn số và ký tự trước chữ `f`), mò mẫm 1 hồi thì em dò được đây là hex(hashMD5(password))
>![](https://i.imgur.com/314EP3o.png)

Kiểm tra lại ta thấy chỉ cần có cookie `stay-logged-in` hợp lý là có thể truy cập vào trang cá nhân mà không cần đăng nhập
Vậy nhiệm vụ bây giờ của ta là brute-force password với password list đã được cho trước, với mỗi password, ta hash nó và `carlos:` với mã hash, sau đó base64 encode là được.
Dùng intruder để brute-force với rule trên, ta thu được cookie
>![](https://i.imgur.com/FZlWkdN.png)
Đăng nhập với cookie brute-force được, ta có kết quả mong muốn
>![](https://i.imgur.com/fn70veh.png)

<hr>

### Bài 10: Offline password cracking
Đây là một dạng bài XSS để trộm Cookie, mô tả của đề như sau:
>![](https://i.imgur.com/IC9nU7K.png)
Ta sẽ đăng nhập với tài khoản được cấp, sau đó vào bài post bất kỳ và để lại comment là payload như sau
>![](https://i.imgur.com/2D85xOR.png)

Đợi carlos truy cập vào và bị XSS, ta có được cookie `stay-logged-in`, dùng nó để truy cập vào tài khoản của `carlos` là solve được challenge, bước cuối decrypt MD5 hash của cookie để lấy password của `carlos`
>![](https://i.imgur.com/QVs74PX.png)

<hr>

### Bài 11: Password reset poisoning via middleware
Đề bài như sau:
>![](https://i.imgur.com/lD5zYr4.png)

Thử sử dụng các tính năng của trang web như đăng nhập/forgot-password và theo dõi request/response của nó trên Burp Suite, ta thấy rằng: Sau khi Post 1 request forgot-password đi thì server sẽ gửi tới email client ta một đường dẫn để ta cập nhật password mới.
![](https://i.imgur.com/PmMMmZu.png)
Như hình trên, ta thấy đường dẫn /forgot-password có tham số `temp-forgot-password-token`. Mỗi user khi request `forgot-password` sẽ có 1 token như vậy, chỉ cần ta có được token của `carlos` là có thể đổi passwd của anh ấy.

Xác định target là lấy token này, nó được đính kèm trên đường link gửi tới `carlos`. 
Đường link này sẽ được tạo và gửi về email khi ta yêu cầu trong POST request sau:
>![](https://i.imgur.com/gTtRtXg.png)

Cấu trúc của đường link server trả về có dạng:
`https://<DOMAIN>/forgot-password?temp-forgot-password-token=<xxxx>`
Đoán rằng `<Domain>` trên đường link có thể lấy từ request, ta thử thay đổi trường `Host` trong request thì thấy không được phép. 
Thử lại với `X-Forwarded-Host` thì thấy server nhận giá trị của nó để cấu trúc nên đường Link kia.
>![](https://i.imgur.com/ldWsdSV.png)

Vậy ta có thể chỉnh sửa để đường Link trỏ tới server của ta, khi `carlos` click vào link sẽ vô tình gửi request kèm token tới ta.

Đầu tiên, cần thay thế trường `username` thành `carlos`, server sẽ generate token cho carlos.

Sau đó thêm trường `X-Forwarded-Host` với giá trị là exploit server của ta. 
>![](https://i.imgur.com/gyhjTuZ.png)

Khi carlos click vào phishing link trong email sẽ vô tình gửi request kèm token tới server của ta, như sau:
>![](https://i.imgur.com/byMPlbI.png)
Dùng token này thay vào trong reset password link là đã có thể thay được password của `carlos`

**Kết quả**:
>![](https://i.imgur.com/S8VouCi.png)

<hr>

### Bài 12: Password brute-force via password change
>![](https://i.imgur.com/PMQZomu.png)
Mô tả: Chức năng thay đổi password của trang web dễ bị brute-force.
Đầu tiên, thử sử dụng chức năng này để xem hành vi của trang web, ta nhận thấy rằng sau khi đăng nhập, ta có thể sử dụng chức năng thay đổi password. 
Chức năng này nhận 4 tham số gồm: `username`,`current-password`, `new-password-1`, `new-password-2`
>![](https://i.imgur.com/xPmDQEF.png)
Thử thay đổi username thành `carlos` rồi brute-force ta thấy không có kết quả. Có lẽ server chặn nếu ta thực hiện sai nhiều lần.

Thử lại dựa theo error message, khi ta nhập new-password-1 và new-password-2 khác nhau sẽ ra error message như sau. 
>![](https://i.imgur.com/SW71DzR.png)

Có lẽ nó được xử lý bởi hàm khác, mà tại đó ta không bị chặn nếu nhập sai nhiều lần.
Brute-force lại lần nữa, với 2 tham số đó khác nhau, ta được kết quả tìm được password của `carlos` là `thomas`
Đăng nhập với mật khẩu này là xong.
![](https://i.imgur.com/IEHrjb9.png)

<hr>

### Bài 13: Broken brute-force protection, multiple credentials per request
>![](https://i.imgur.com/JIv0mNo.png)

Đây cũng là một bài brute-force nhắm vào chức năng bảo vệ chống brute-force của trang web.
Thử kiểm tra tổng quan trang web ta thấy rằng nếu cố tình đăng nhập sai nhiều lần thì sẽ bị block trong 1 phút, không có cách để reset. Vậy nếu muốn brute-force, ta phải gửi request đăng nhập hợp lệ sau mỗi lần brute-force.
>![](https://i.imgur.com/pUSewKc.png)
Em đã thử gửi xen kẽ payload hợp lệ và payload brute-force như không thành công, vẫn bị chặn.
Xem kĩ lại, username và passwor được gửi dưới dạng json, vậy cũng có thể server sẽ xác thực kiểu
```python3!
if('xxx' in request.password) {
#do_sth
}
```
Em tạo một mảng gồm tất cả các password mà đề cho, sau đó gửi đi như sau
>![](https://i.imgur.com/rSZCnzc.png)

Thì có kết quả là ta truy cập vào tài khoản `carlos` thành công
>![](https://i.imgur.com/nsp6h2I.png)
<hr>

### Bài 14: 2FA bypass using a brute-force attack
>![](https://i.imgur.com/J7PeBDb.png)

Ở bài này đề đã cung cấp tài khoản và mật khẩu hợp lệ, nhiệm vụ của ta là bypass 2FA bằng brute-force.

Kiểm tra ta thấy nếu 2 lần nhập 2FA code sai thì sẽ bị redirect tới trang đăng nhập lại. Vậy sẽ khó hơn cho ta trong việc brute-force
>![](https://i.imgur.com/oXIDNrP.png)

Sau mỗi lần brute-force, ta cần phải lại gửi request đăng nhập lại. Sẽ là rất khó khăn nếu mỗi lần ta đăng nhập thì 2FA token được regenerate, nhưng bài này lại không, ta chỉ cần gửi 1 bộ 3 request `GET /login1`,`POST /login1`, `GET /login2` và brute-force 2FA là xong

Kết quả:
>![](https://i.imgur.com/S8M0jyF.png)

<hr>
<hr>

# Access control vulnerabilities
### Bài 1: Unprotected admin functionality
>![](https://i.imgur.com/uR0bxcH.png)

Bài này sẽ chứa lỗ hổng `access control` trong kênh dành cho admin, mục tiêu của ta là truy cập vào vị trí đó và xóa user `carlos`

Truy cập trang web, ta thử brute-force các đường dẫn tới trang của admin như `/admin`, `/administrator`, `/admin-panel`,`admin_panel`,... Thì tìm thấy đường dẫn `/administrator-panel` dẫn tới trang dành cho admin, tuy nhiên nó lại không có bất kỳ cơ chế access control nào
>![](https://i.imgur.com/4VHEWLT.png)

Xóa user `carlos` là ta solve challenge
>![](https://i.imgur.com/4b6uByR.png)

<hr>

### Bài 2: Unprotected admin functionality with unpredictable URL
Theo mô tả sau của đề bài, bài này cũng khá giống bài trên, nhiệm vụ của ta cũng là đi tìm đường dẫn tới trang admin
>![](https://i.imgur.com/V9Vxqyk.png)

View source code của trang web, ta thấy nó có đoạn code javascript như sau
```javascript!
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-0efeh5');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```
Từ trên, ta có thể suy đoán được đường dẫn là `/admin-0efeh5`
Truy cập vào đó và thấy rằng nó không có cơ chế kiểm tra truy cập, ta xóa user `carlos` là solve được challenge

>![](https://i.imgur.com/7WYb7XT.png)

<hr>

### Bài 3: User role controlled by request parameter

>![](https://i.imgur.com/5TbpwYN.png)

Theo mô tả, đường dẫn `/admin` của trang web sẽ xác thực admin thông qua một cookie.
Thử đăng nhập vào user bình thường và xem cookie server gán cho ta, ta thấy có trường `isAdmin` có giá trị `false`.
>![](https://i.imgur.com/m6YyueK.png)

Ta thử đổi nó thành `true` và truy cập `/admin` thì thấy thành công. Xóa user `carlos` là solved

>![](https://i.imgur.com/6dLrSRg.png)

<hr>

### Bài 4: User role can be modified in user profile

>![](https://i.imgur.com/S5RtchP.png)

Ở bài này, server kiểm soát truy cập thông qua `roleid` trong profile của user. Đăng nhập trang web với user thường, và thử chức năng `Update email` ta thấy server trả về `roleid` trong kết quả
>![](https://i.imgur.com/Ti5f5oo.png)

Thử thêm vào payload của request giá trị `roleid:2` thì thấy server trả về roleid mới của ta, tức nó vô tình cập nhật lại roleid trong câu truy vấn update
>![](https://i.imgur.com/5nY9xUB.png)

Bây giờ ta đã có thể truy cập `/admin`
Xóa `carlos` là thành công

>![](https://i.imgur.com/3e2A4tF.png)

<hr>

### Bài 5: User ID controlled by request parameter
Theo mô tả của tiêu đề, tham số của request dễ bị tấn công leo quyền
>![](https://i.imgur.com/Fo7u37p.png)

Đăng nhập dưới người dùng thường, ta thấy `id parameter` lúc này có giá trị là tên đăng nhập của ta, thử đổi thành `carlos` ta thấy có thể truy cập vào tài khoản của người khác

>![](https://i.imgur.com/C8vs4Vp.png)

Lấy api-key của `carlos` submit là đã có thể solve

>![](https://i.imgur.com/WsZxRYJ.png)

<hr>

### Bài 6: User ID controlled by request parameter, with unpredictable user IDs 

Bài này cũng giống  bài trên, tuy nhiên xác thực bằng unpredicted UserID người dùng
>![](https://i.imgur.com/MFhlsTd.png)

Đăng nhập bằng tài khoản user thường, ta thấy ID của ta lúc này là `c80f2760-226e-4494-923d-489de5fa1974`, là chuỗi sinh ngẫu nhiên.

Kiểm tra trang home, ta thấy có 1 bài blog của `carlos`, click vào bài viết ta thấy id của carlos nằm trên đường dẫn như sau:

>![](https://i.imgur.com/h1tmMFn.png)

Vào lại trang profile, đổi id của ta với id tìm được ở trên, ta có thể vào trang cá nhân của carlos và lấy api-key

>![](https://i.imgur.com/FKTXvB0.png)

<hr>

### Bài 7: User ID controlled by request parameter with data leakage in redirect
Theo đề, thông tin nhạy cảm sẽ bị leak trong phần body của redirect response
>![](https://i.imgur.com/9MnGGZF.png)

Truy cập trang web và follow từng redirect response, ta thấy khi ta thay đổi giá trị id trên param, nó sẽ tạo ra 1 redirect response. Tuy nhiên, redirect này lại hiển thị ra nhiều nội dung nhạy cảm

>![](https://i.imgur.com/Plwurzd.png)

Có api-key, ta submit là solve

>![](https://i.imgur.com/TM0sEIU.png)

<hr>

### Bài 8: User ID controlled by request parameter with password disclosure
Theo đề, trang cá nhân của user sẽ có chứa password ẩn
>![](https://i.imgur.com/jq9hCBY.png)

Truy cập trang web và vào trang cá nhân của mình, ta thấy có trường password được filled sẵn như sau
>![](https://i.imgur.com/4WvwuXm.png)

Thay đổi giá trị của `id` parameter trên đường dẫn thành administrator, ta có thể vào trang cá nhân của `administrator`

Inspect trang cá nhân, ta thấy được password
>![](https://i.imgur.com/mdlkzum.png)

Dùng tài khoản `administrator` với password trên, ta solve thành công bài lab

>![](https://i.imgur.com/ZEVkQuV.png)

<hr>

### Bài 9: Insecure direct object references

>![](https://i.imgur.com/jLKFsP9.png)

Mô tả: trang web này lưu chat logs trên file system và truy cập tới bằng URL tĩnh

Khi sử dụng tính năng `live chat`, trang web sẽ cho phép ta tải transcript về
>![](https://i.imgur.com/MgMaz6h.png)

Kiểm tra các request bằng Burp Suite ta thấy khi click `View transcript` sẽ gửi request như sau
>![](https://i.imgur.com/sl96QbY.png)

Sửa tên file thành `1.txt` ta thấy nội dung file như sau:
>![](https://i.imgur.com/ORCRpIL.png)

Lấy password trong file log đó đăng nhập vào tài khoản `carlos`, ta thấy thành công solve bài lab

>![](https://i.imgur.com/QXCSvmI.png)

<hr>

### Bài 10: URL-based access control can be circumvented

>![](https://i.imgur.com/ZR3DqSi.png)

Theo như mô tả của đề, trang web có chứa đường dẫn `/admin` nhưng chặn không cho truy cập từ client, và đề cũng cho hint là backend supports `X-Original-URL` header.

Header này giúp ghi đè lên đường dẫn chính của request. Server sẽ thực hiện truy vấn tới giá trị url của header này và trả kết quả về front-end.

Ta thử sửa request đến /admin thành một đường dẫn sai nào đó, và thêm trường `X-Original-URL` kia vào header với giá trị là `/admin`. Nhận thấy kết quả là có thể truy cập tới admin panel.
>![](https://i.imgur.com/waJ58cU.png)

Sửa lại giá trị của nó thành `/admin/delete?username=carlos `là ta có thể xóa user `carlos` và solve bài lab
>![](https://i.imgur.com/fNDAbQy.png)

<hr>

### Bài 11: Method-based access control can be circumvented
>![](https://i.imgur.com/7e23fj5.png)
Theo đề, bài này access control có 1 phần thông qua request method
Admin panel chỉ có thể truy cập từ tài khoản admin, nên ta sẽ dùng tài khoản đề cho để mở nó lên trong Burp Repeater.
Sau đó thay session cookie của admin thành của Wiener, nó sẽ tương tự như khi ta mở admin panel với tài khoản của `wiener` ,ta thấy kết quả như sau:
>![](https://i.imgur.com/jAtxohU.png)
Tuy nhiên khi convert POST method thành GET method thì ta thấy server không còn xác thực access control nữa, bypass được là ta có thể solve bài lab
>![](https://i.imgur.com/cOMYssg.png)

<hr>

### Bài 12: Multi-step process with no access control on one step

>![](https://i.imgur.com/sbEM9U4.png)
Bài này cũng không khác gì yêu cầu bài trước, chỉ là ta sẽ trải qua nhiều bước hơn (2 bước) để promote bản thân lên quyền quản trị

Việc chuyển method khác không còn giá trị, hơn nữa, sau khi gửi request sẽ có thông báo xác nhận hành vi.
Theo gợi ý từ trong đề bài, có 1 step mà việc kiểm soát truy cập bị fail
Ta thử khai thác ở bước ***confirm*** của server, tham số sẽ gồm `confirmed`, `username`, `action`. Cộng với việc thay cookie như bài trên là ta giải được bài toán.
>![](https://i.imgur.com/Obt9AQZ.png)

<hr>

### Bài 13: Referer-based access control

>![](https://i.imgur.com/R9UBLOq.png)

Ở bài lab cuối, server xác thực quyền quản trị thông qua header `Referer`

Ta cũng sẽ thử dùng account admin để xem format request và sau đó thay session bằng session của wiener. Kết quả là ta có thể tự promote lên account admin. (Không tìm thấy điểm nào liên quan tới trường `Referer`...)

>![](https://i.imgur.com/od7iKBS.png)