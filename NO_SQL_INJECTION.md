## No SQL Injection
### 100 points

Đây là bài đầu tiên mình làm trong kì CTF này. Sáng hôm thi mới biết là phải vào mạng công ty mới làm bài được, vì quá phấn khích nên mình đã bùng học chạy hộc bơ lên công ty để thi. Mở scoreboard lên nghĩ quái lạ, bài này 100 points, nằm đầu tiên trong list bài mà lại chưa ai động, nên mình quyết định làm luôn. Cảm giác là người giải được đầu tiên bao giờ nó cũng vui hơn :)).

### Đề bài:
```
Trên đỉnh gò xưa
http://nosqlinjection.ctf2019.yeuchimse.com
Challenge Oreo sẽ được mở khi bạn giải được bài này
```

### Giao diện của bài:

### Lời giải
Tạo 1 tài khoản, đăng nhập thử:

```
Bạn có biết khác biệt giữa bạn và admin là gì không?
Là admin thì có flag, còn bạn thì không. Ha ha ha -_-
```

Ok, vậy chắc mục tiêu chính của bài này là làm sao vào được admin rồi. 

Thấy trong bài có source nên mình nghía thử luôn:

Source trông có vẻ tức mắt vì nhiều for với if quá, nhưng nếu bạn chịu khó đọc một chút thì sẽ thấy chỉ có đúng 2 phần tương ứng với 2 tính năng thôi. Sau đây là flow của các tính năng:

+ Đăng ký
1. Lấy username và password từ post data ($POST), loại bỏ các string `../`, `..\\` khỏi username
2. Check xem tên tài khoản đã có trên hệ thống chưa (ở đây hệ thống dùng các thư mục có tên tương ứng với username trong path /users)
3. Nếu tên tài khoản đã có thì trả về thông báo "Tên tài khoản đã được sử dụng.", không thì qua bước 4
4. Nối username với đường dẫn tới thư mục /users ($base_dir) và parse lại bằng hàm `get_absolute_path` (hàm này có tác dụng format lại đường dẫn và xử lý relative path trong đường dẫn của bạn. Ví dụ `/path/abc/users/../` sẽ bị replace thành `/path/abc`, sau đó check lại xem có $base_dir trong đường dẫn vừa format không. Nếu không có $base_dir thì trả về "Tên tài khoản không hợp lệ.", có thì qua bước 5. Đoạn này chắc để chống injection để thoát ra khỏi thư mục /users.
5. Tạo password file và ghi lên đó `md5($password)`. Gán session.

+ Đăng nhập
1. Lấy $username và $password từ post data ($POST)
2. Check xem tài khoản có hợp lệ không (giống bước 4 ở trên), nếu không thì trả về "Tên tài khoản không hợp lệ.", có thì qua bước 3
3. Lấy password hash từ password file tương ứng với username và check hash xem có giống md5($password) không. Nếu giống thì gán session, ngoài ra nếu là admin thì đổi password thành `md5($password . SECRET)`

+ Ngoài ra còn có phần logout tuy nhiên phần này chẳng có gì ngoài unset session nên mình không nói thêm

Trong khi đọc hiểu và phân tích tính năng thì mình thấy trong source có mấy comment `\~(*o*)~/`, `^_^` ở các phần check đường dẫn rất là ẩn ý. Mà cũng dễ thấy phần đầu tiên lọc bỏ `../` và `..\\`, nhưng phần dưới lại chống không cho dùng relative path để ra khỏi /users, chứng tỏ bài này có cách bypass qua đoạn check kia.

Từ đó mình nghĩ ra cách làm bài này là làm sao dùng tính năng đăng ký và path traversal để ghi đè password của mình lên password của admin. Sau đó đăng nhập vào lấy flag.
Nghĩ tí là ra ngay payload `....//users/admin` thôi. Các bạn xem đoạn test này của mình cho dễ hiểu

Flag:
> flag{you_cant_have_sql_injection_if_you_dont_use_sql}
