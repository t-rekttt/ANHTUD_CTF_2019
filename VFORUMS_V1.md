## VForums V1
### 200 points

Một trong những bài mà mình làm tốn nhiều thời gian nhất

### Đề bài
<pre>
<a href="http://35.240.153.120:8001">http://35.240.153.120:8001/</a>
</pre>

### Giao diện của bài

Vẫn chỉ là trang login
![view-login]

### Lời giải

Tạo tài khoản và đăng nhập vào site, đi loanh quanh ta thấy có một vài tính năng chính: comment, change password, update profile
![view-comment]

![view-change-password]

![view-update-profile]

Dùng thử các tính năng này, và debug HTTP Request, mình phát hiện ra có `auth` cookie thay đổi mỗi khi `Name` được update (nếu `Name` giữ nguyên thì `auth` không thay đổi). Ngoài ra vì `auth` trông rất giống base64 và trước đấy mình có làm bài `Oreo` rồi nên liên tưởng ngay đến XOR encryption.
Vì vậy nên mình lặp lại các bước giống như ở bài `Oreo`:

Tìm đoạn lặp
![view-repetitive]

XOR lại để ra key (nhưng sắp xếp chưa đúng)
![view-xorred-key]

Tìm cách sắp xếp đúng
![view-real-key]

Ok, vậy là leak được data ra rồi, còn phần encryption lại thì mình tính sau nhé. Giờ xem cái data này để làm gì đã.
Ban đầu mình nhìn cái định dạng data này thì không hiểu nó là gì, vì trông nó lạ lạ, mình chưa gặp bao giờ.

Sau một hồi Google mò mẫm thì mình phát hiện ra nó là `PHP serialized object`, kèm theo đó là biết được có exploit `PHP Object Injection`, tuy nhiên vì đề bài không cho source nên mình nghĩ chắc không phải là lỗi này.

Vì vậy nên mình thử theo hướng thay `username` thành `1` và `id` thành `admin`, xem liệu có phải flag được giấu trong account admin hay không.

Tuy nhiên sau 2 ngày thử sml đủ mọi cách, từ thử chèn XSS hay PHP payload vào, tới iterate users trong db, mình cũng không thấy có vẻ gì là inject được, nên mình bắt đầu nghĩ có lẽ nó là hướng `PHP Object Injection` kia thật.
May mắn là đến ngày thứ 3 thì vớ được source, nằm ngay ở `/source`. 

Nhưng file source này không có extension. Vì thế mình vứt lên `checkfiletype.com` thì thấy detect ra là file `.zip`
![view-file-type]

Sửa lại extension thành `.zip`, mở ra là có source
![view-src-structure]

Dạo quanh source một lúc, mình tìm thấy biến `$flag` này trong file `config.php`
![view-config]

Do đó nên mình cho rằng, mục tiêu của bài này là làm sao đọc được file `config.php` ở đây để lấy flag

Tiếp đó mình lại tìm thấy thêm file `Logger.php`, với một magic function `__toString` khá là dị, mình thắc mắc tại sao không dùng getter để đọc logs, mà lại dùng `__toString`? Có vẻ là dụng ý của người ra đề đây rồi
![view-logger]

Ngoài ra mình tìm được file `Controller.php`, thấy trong file này có lệnh `@unserialize(OpenCipher::decript($_COOKIE['auth']));`, hàm decrypt thì vì mình đã đoán được thuật toán và tìm ra key nên không quan tâm nhiều. Còn lại cái đáng quan tâm nhất ở đây là hàm `unserialize`.

Như vậy mục tiêu của mình lại càng rõ ràng, đó là làm sao sử dụng `PHP Object Injection`, inject vào đó class `Logger`, trigger magic function `__toString` để đọc flag trong file `config.php`

Việc tiếp theo của mình thì đơn giản, đó là lên mạng tìm example payload, sau đó áp dụng vào với trường hợp của mình thôi

Và đây, mình tìm được cái example này hợp lý vãi, cũng có magic function `__toString`, cũng dùng một class khác để inject vào, mình chỉ phải tư duy nốt làm sao để chèn class `Logger` vào thay vì `SQL_Row_Value` của ví dụ thôi
![view-example]

![view-example1]

Mình viết lại một đoạn PHP để generate và serialize payload
![view-generated-object]

Sau đó sửa lại đoạn code Python một chút để encrypt lại
![view-encrypted-object]

Sau đó `urlencode` lại và thay vào `auth` cookie trên browser. Ngoài ra mình có test thì thấy nếu để nguyên cookie `PHPSESSID` thì web sẽ sử dụng session đó, nên mình cũng xoá luôn cookie này đi

### Flag

![flag]

```
flag{Y0u_win!!!__unsafe_deserialize}
``` 

[view-login]: assets/VFORUMS_V1/view-login.png
[view-update-profile]: assets/VFORUMS_V1/view-update-profile.png
[view-change-password]: assets/VFORUMS_V1/view-change-password.png
[view-comment]: assets/VFORUMS_V1/view-comment.png
[view-repetitive]: assets/VFORUMS_V1/view-repetitive.png
[view-xorred-key]: assets/VFORUMS_V1/view-xorred-key.png
[view-real-key]: assets/VFORUMS_V1/view-real-key.png
[view-file-type]: assets/VFORUMS_V1/view-file-type.png
[view-src-structure]: assets/VFORUMS_V1/view-src-structure.png
[view-config]: assets/VFORUMS_V1/view-config.png
[view-logger]: assets/VFORUMS_V1/view-logger.png
[view-example]: assets/VFORUMS_V1/view-example.png
[view-example1]: assets/VFORUMS_V1/view-example1.png
[view-encrypted-object]: assets/VFORUMS_V1/view-encrypted-object.png
[view-generated-object]: assets/VFORUMS_V1/view-generated-object.png
[flag]: assets/VFORUMS_V1/flag.png