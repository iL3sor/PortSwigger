# Nguyễn Văn Thọ - Cross-site Request Forgery (CSRF)

<hr>

### Bài 1: CSRF vulnerability with no defenses

> [color=#210b70]![](https://i.imgur.com/QGwExqN.png)


Bài lab chứa lỗ hổng CSRF ở chức năng `email change`. Để solve bài lab, tạo 1 HTML để tự động thay đổi email của người dùng khác.

Kiểm tra source-code HTML, ta thấy chức năng change email không có sử dụng CSRF token.

> [color=#210b70]![](https://i.imgur.com/Re7Cxmf.png)

Sử dụng chức năng `Generate CSRF PoC` của BurpSuite, ta tạo được một HTML giúp thực hiện tấn công CSRF trên trang web để thay đổi email người khác.

> [color=#210b70]![](https://i.imgur.com/SfkJXTr.png)

Kết quả:

> [color=#210b70]![](https://i.imgur.com/nVvB2Es.png)


<hr>

### Bài 2: CSRF where token validation depends on request method

> [color=#210b70]![](https://i.imgur.com/ayDVxxU.png)

Chức năng `change email` bây giờ đã sử dụng `CSRF token` để chặn tấn công, nhưng chỉ kiểm tra token trên một số request method.

Method mặc định của request là POST, tuy nhiên khi thử đã không thành công.

Ta thử convert method của request sang GET, cũng tạo HTML bằng `Generate CSRF PoC` của BurpSuite để tạo payload

> [color=#210b70]![](https://i.imgur.com/67KfYmk.png)


Kết quả:

> [color=#210b70]![](https://i.imgur.com/pH97gzS.png)


<hr>

### Bài 3: CSRF where token validation depends on token being present

> [color=#210b70]![](https://i.imgur.com/yoxLkkN.png)

Chức năng thay đổi email chứa lỗ hổng CSRF, theo tiêu đề thì nếu không có CSRF token, trang web sẽ không thực hiện validate

Thử trên chính tài khoản của mình (xóa token khỏi form) thì ta thấy có thể khai thác thật.

Việc còn lại là tạo fake form HTML và xóa trường CSRF đi.

Kết quả:

> [color=#210b70]![](https://i.imgur.com/zQrnktD.png)


<hr>

### Bài 4: CSRF where token is not tied to user session

> [color=#210b70]![](https://i.imgur.com/uSZqcb3.png)

Bài lab chứa lỗ hổng CSRF khi token không gắn với session của người dùng.

Tìm hiểu ngữ cảnh cụ thể ta thấy rằng:

> [color=#210b70]![](https://i.imgur.com/V04UwBa.png)

Tức là chương trình kiểm tra CSRF token có phải do nó sinh ra hay không mà không quan tâm token đó được nó sinh ra cho user nào

Vậy ta chỉ cần dùng token hợp lệ của ta để thay đổi email của user khác.

> [color=#210b70]![](https://i.imgur.com/Lvb2pqI.png)


<hr>

### Bài 5: CSRF where token is tied to non-session cookie

> [color=#210b70]![](https://i.imgur.com/U8Vrdo5.png)

Lỗ hổng CSRF tồn tại khi CSRF token không gắn với `session cookie`


Kiểm tra trang web ta thấy ngoài cookie `session` còn có cookie `csrfkey` (sẽ cập nhật lại mỗi lần refresh). Server sẽ kiểm tra xem csrf token có link với giá trị cookie này không. 

Để khai thác, ta cần 2 giá trị này link với nhau => ta cần sửa lại cookie `csrfKey` của nạn nhân thành của ta, và token của nạn nhân thành token của ta là khai thác được.

Đầu tiên, tìm cách sửa cookie của nạn nhân, ta tìm kiếm thấy chức năng search được chương trình ghi lại nội dung tìm kiếm vào cookie `lastSearchTerm`

> [color=#210b70]![](https://i.imgur.com/xBs2Z4A.png)

Thử search lại với nội dung:

```
1111;%0d%0aSet-Cookie=csrfKey=HL11Fs1AxXevF0bnbCD4wajkN8rtEKvv
```

Ta đã inject được vào giá trị cookie với giá trị hợp lệ và link với csrftoken của chính mình, việc còn lại là tìm cách tạo 1 fake form HTML để nhét được 1 request inject cookie vào payload CSRF.

Ta thực hiện như các bài trên, nhưng thay vì submit form ngay lập tức thì sửa lại như sau để có thể inject cookie lên nạn nhân.

```htmlembedded!
<img src="https://0a70001604547e37c0138b4300070052.web-security-academy.net/?search=1111%3b%0d%0aSet-Cookie:csrfKey=HL11Fs1AxXevF0bnbCD4wajkN8rtEKvv%3b%20SameSite=None" onerror="document.forms[0].submit()">
```


> [color=#210b70]*(Chú ý phải có thuộc tính SameSite=None thì mới thực hiện được cross-site)*


> [color=#210b70]![](https://i.imgur.com/HESDro1.png)


<hr>

### Bài 6: CSRF where token is duplicated in cookie

> [color=#210b70]![](https://i.imgur.com/Fbswy31.png)

Theo ngữ cảnh của đề, server không lưu trữ cookie mà nó đã 'phát hành' trên server, thay vào đó nó lưu trên cookie của người dùng. Khi 1 form được submit, nó sẽ check xem CSRF token có trùng với giá trị cookie đó không.

Chính vì lí do trên, nếu trang web có một chức năng gọi đến `Set-Cookie`, nó sẽ dễ dàng bị tiêm fake cookie vào và thực hiện CSRF (vì lúc này đã biết được giá trị token)

Kiểm tra lại chức năng `Search` như bài trên, ta thấy rằng nó vẫn thực hiện việc set cookie `lastSearchTerm`

> [color=#210b70]![](https://i.imgur.com/1JZpOiI.png)

Vậy chỉ cần thực hiện các bước tương tự như bài trên là khai thác được (giá trị của cookie và từ token có thể lấy của chính mình).

> [color=#210b70]![](https://i.imgur.com/5BDkuRU.png)


<hr>

### Bài 7:SameSite Lax bypass via method override

> [color=#210b70] ![](https://i.imgur.com/S93Ndml.png)

Tìm cách override request method để tấn công CSRF (vì lúc này cookie đã có thuộc tính `SameSite=Lax`)

Kiểm tra ta thấy cookie `session` không có giá trị => mặc định của nó là `Lax`. Vậy các third-party chỉ có thể gửi request thông qua GET method.

Thử dùng `change Request Method` của Burpsuite ta thấy chức năng này không chấp nhận GET method.

> [color=#210b70] ![](https://i.imgur.com/8gvTqj4.png)

Thử override method GET bằng `_method=POST` (framework Symfony chấp nhận kiểu override này). Vậy là bên thứ 3 có thể gửi một POST request thông qua GET request, nhờ đó có thể thực hiện tấn công CSRF.

> [color=#210b70] ![](https://i.imgur.com/ynoWgsl.png)


<hr>

### Bài 8:SameSite Strict bypass via client-side redirect

> [color=#210b70] ![](https://i.imgur.com/o2dnS4G.png)

Bypass thuộc tính `SameSite` có giá trị `Strict` để thực hiện CSRF.

Nếu cookie có thuộc tính `SameSite` với giá trị `Strict`, thì các request cross-site sẽ không chứa cookie này. Buộc ta phải tạo một request dựa trên các redirection có sẵn trên trang web. Và để khai thác thì cần kiểm soát được tham số mà redirection đó nhận.

Kiểm tra ta thấy sau khi comment thì trang web dừng khoảng vài giây rồi tự redirect về lại post đó 

```
/post/comment/confirmation?postId=5
```
Tham số postId sẽ được server dùng để nhúng vào đường dẫn `post/X` để redirect.

Vì kiểm soát được tham số này, ta sẽ sửa lại cho nó redirect tới chức năng thay đổi email.

```
X/../../my-account/change-email
```

Chức năng này không sử dụng CSRF token nên dễ bị khai thác.

Kiểm tra thêm ta thấy nó còn chấp nhận GET method, vậy ta cần sửa payload trên lại thành như sau:

```
X/../../my-account/change-email?email=aaaaaaaa%40gmail.com%26submit=1
```

Cuối cùng, lưu trên exploit server đoạn mã như sau, ta có thể khai thác CSRF.


```javascript!
<script>
document.location= 'https://0aed00dd04c37eaac07bb86e009000b0.web-security-academy.net/post/comment/confirmation?postId=foo/../../my-account/change-email?email=aaaaaaaa%40gmail.com%26submit=1'
</script>

```

> [color=#210b70] ![](https://i.imgur.com/5SmsGpT.png)


<hr>

### Bài 9:SameSite Strict bypass via sibling domain

> [color=#210b70] ![](https://i.imgur.com/pSBv8kX.png)

Chức năng live chat của trang web đang bị lỗi Cross-site WebSocket hijacking. Để solve bài lab, tìm cách đăng nhập vào tài khoản nạn nhân (bị lộ trong chat history).


<hr>

### Bài 10:SameSite Lax bypass via cookie refresh

> [color=#210b70] ![](https://i.imgur.com/3fIvqsd.png)

Chức năng thay đổi email của bài lab bị lỗi CSRF. Tìm cách thay đổi email của nạn nhân.

Kiểm tra ta thấy trang web khi set cookie cho phiên đăng nhập, đã để `SameSite` có giá trị mặc định, tức `Lax`.

> [color=#210b70] ![](https://i.imgur.com/M7mT4dc.png)

Vì nó là `Lax` nên trang web chỉ include cookie trong các request cross-site với method là GET và nó phải là top-level navigation (navigation khiến thay đổi luôn trên url)

Nghiên cứu kĩ thuật, ta thấy việc dùng giá trị `Lax` cho thuộc tính SameSite của cookie sẽ dẫn đến break SSO (single sign-on) [Tìm hiểu bài viết đính kèm link bên dưới].

> [color=#210b70] Tham khảo: https://stackoverflow.com/questions/63402508/feasibility-of-sso-with-samesite-lax-cookies-only

Theo tài liệu của PortSwigger, bài lab này phòng tránh việc break SSO bằng cách không áp dụng `SameSite=Lax` lên cookie trong 2 phút đầu khi người ta truy cập, điều đó dẫn tới trang web dễ bị tấn công CSRF trong khoảng thời gian này. Nếu thời gian quá 2p, `Lax` sẽ được áp dụng và trang web không đính kèm cookie trong các cross-site request

Thêm vào đó nữa, chức năng thay đổi email của trang web lại không chứa csrf token nên tăng khả năng tấn công (nếu bypass được hạn chế của `SameSite`)

Lỗ hổng ở đây bắt nguồn từ việc 2 phút không có `Lax` và chức năng thay đổi email không có CSRF token dẫn tới nó sẽ bị khai thác.

Chú ý rằng mỗi lần truy cập vào đường dẫn `/social-login` ta sẽ được OAuth cấp session mới. Không quan tâm là ta có đang đăng nhập ở trang web (mà ta muốn vào) hay không.

Mỗi lần refresh session token như vậy sẽ lộ ra khoảng 2p cho các cuộc tấn công CSRF như miêu tả bên trên.

Viết lại script để điều hướng nạn nhân tới trang `/social-login` để refresh token, tạo khoản thời gian 2p, sau đó thực hiện CSRF.

Sau 5s kể từ khi nạn nhân click vào điểm bất kì trên trang, payload CSRF sẽ được gọi để thực thi. Lúc này cookie sessiond đã được cấp mới, lưu trên browser và không có bất kì hạn chế nào.

```javascript!
<form method="POST" action="https://0ac1000a04bd84ccc0bd54ba005d0086.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="pwned@portswigger.net">
</form>
<p>Click me</p>
<script>
    window.onclick = () => {
        window.open('https://0ac1000a04bd84ccc0bd54ba005d0086.web-security-academy.net/social-login');
        setTimeout(changeEmail, 5000);
    }

    function changeEmail() {
        document.forms[0].submit();
    }
</script>
```



> [color=#210b70] ![](https://i.imgur.com/p5BGmJX.png)


<hr>

### Bài 11:CSRF where Referer validation depends on header being present

> [color=#210b70] ![](https://i.imgur.com/sEQRCpj.png)

Chức năng thay đổi email của bài lab bị lỗi CSRF, nó cố chặn các cross-site request nhưng mà vẫn chưa hoàn toàn an toàn.

Kiểm tra gói tin HTTP gửi đi khi thực hiện change email, ta thấy rằng nếu sửa đổi domain của trường `referer` trong header thì sẽ nhận được thông báo lỗi `Invalid request header`. Gửi lại request và đưa domain dưới dạng tham số  thì vẫn nhận báo lỗi như trên.

Tuy nhiên khi loại bỏ header này ra khỏi request thì sẽ bypass được.

Vậy ta cần điều chỉnh cho exploit server của ta KHÔNG chứa referer header trong request gửi đi

```htmlembedded!
<meta name="referrer" content="never">
```
> [color=#210b70] ![](https://i.imgur.com/AiEifz5.png)


<hr>

### Bài 12:CSRF with broken Referer validation

> [color=#210b70] ![](https://i.imgur.com/DjJaxWP.png)

Chức năng thay đổi email của bài lab bị lỗi CSRF, mặc dù nó đã chặn cross-site request nhưng kĩ thuật detection có thể bị bypass.

Fuzzing các trường trong gói tin `change email` gửi đi, ta thấy rằng nếu `Referer` thay đổi phần domain, sẽ có thông báo `Invalid referer header` trả về.

Tuy nhiên trường hợp ta đưa domain trang web ra làm tham số thì vẫn được chấp nhận. Nghĩa là server sẽ check xem trong `Referer Header` có chứa domain đã xác định hay không, điều này có thể bypass.

Kiểm tra chức năng `change email`, ta thấy không có sử dụng CSRF token => dễ bị tấn công CSRF.

Vậy tóm lại ta biết được server phòng chống CSRF bằng cách ngăn chặn các cross-site request thông qua trường `Referer` trong header

Để set `Referer` header trong CSRF payload của ta thành đường dẫn chứa tham số như mong muốn, thêm vào payload đoạn javascript sau:

```javascript!
history.pushState('', '', '/?0a4e00a804359c07c025364900370062.web-security-academy.net')
```

Generate payload và thử lại thì ta thấy attack không thành công, lý do là browser đã loại bỏ các tham số ra khỏi http referer.

Chỉnh lại exploit server và thêm vào header `Referrer-Policy: unsafe-url`

Kết quả

> [color=#210b70] ![](https://i.imgur.com/YxeaDvu.png)

<hr>

