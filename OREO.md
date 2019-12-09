## Oreo
### 250 points - 100 points hint

Vì phấn khích khi là người đầu tiên giải được `No SQL Injection` và mở được bài này, nên mình quyết tâm đâm luôn

### Đề bài:
<pre>
Thôi còn chi nữa mà mong

<a href="http://oreo.ctf2019.yeuchimse.com">http://oreo.ctf2019.yeuchimse.com</a>

Challenge FA sẽ được mở khi bạn giải được bài này

Cập nhật 09:07 15/11/2019: Điểm số tăng từ 200 lên 250, điểm mua hint tăng từ 50 lên 100.
</pre>

### Giao diện của bài:

![][view]

![][view-register]

### Lời giải
Nhìn sơ qua có vẻ tính năng cũng hơi na ná với bài `No SQL Injection`. Chỉ khác là phần đăng ký yêu cầu phải có `Mã giới thiệu`.

`view-source` cái là biết lấy nó ở đâu nè

![][view-get-invite-code]

Paste path vô để truy cập và...ơ?...nó redirect mình qua trang này

![][view-register-disabled]

Nhưng mà thực ra là lừa thôi, bạn capture cái request đó bằng một con HTTP Debugger bất kì sẽ thấy

![][view-invite-code]

Invite code: `thekingdomisstillyoungbutsheisnot`

Đăng ký bừa một tài khoản để vào xem thế nào:

![][view-logged-in]

`view-source` trang này, thấy có vẻ sạch sẽ, không có gì mờ ám, mình có đi check thêm loanh quanh vài folder ảnh và JS, CSS, tuy nhiên cũng không có vẻ gì khả nghi nên mình bỏ qua, không nói thêm nhiều  

![][view-logged-in-source]

Coi headers, thấy có 2 cái cookie là lạ là `hash` và `session`

![][view-cookie]

Đoạn session kia có kí tự `%3D` là urlencoded của dấu `=`, nghi nghi đây là đoạn base64 tuy nhiên khi mình decode ra thì chỉ được mớ kí tự loằng ngoằng thôi

![][view-session-cookie-url-decoded]

Nghi nghi đây là dạng XOR encryption, tuy nhiên vì chưa làm dạng này bao giờ, nên sau tầm 3-4 tiếng vắt óc suy nghĩ thì mình quyết định mua hint. 

Tại thời điểm mình mua thì bài này 200 points và giá của hint chỉ là 50 points thôi, nên mặc dù mọi người xung quanh hết sức can ngăn nhưng vì nản quá nên mình mua đại :v

![][hint]

May quá được bộ source ngon full không che

<details>
  <summary>Source code</summary> 
  
  ```php
  <?php
  
  function get_db_connection()
  {
      $conn = new mysqli(DB_SERVER, DB_USER, DB_PASSWORD, DB_DATABASE);
  
      if ($conn->connect_error) {
          die('Lỗi database: ' . $conn->connect_error);
      }
  
      return $conn;
  }
  
  
  function register($username, $password)
  {
      $conn = get_db_connection();
  
      $stmt = $conn->prepare("INSERT INTO users (username, password) VALUES (?, ?)");
      $stmt->bind_param("ss", $username, md5(md5(md5($password))));
      $stmt->execute();
  
      $success = $conn->affected_rows > 0;
  
      $stmt->close();
      $conn->close();
  
      return $success;
  }
  
  
  function login($username, $password)
  {
      $conn = get_db_connection();
  
      $stmt = $conn->prepare("SELECT * FROM users WHERE username = ? and password = ?");
      $stmt->bind_param("ss", $username, md5(md5(md5($password))));
      $stmt->execute();
      $stmt->store_result();
  
      $success = $stmt->num_rows > 0;
  
      $stmt->close();
      $conn->close();
  
      return $success;
  }
  
  
  function encrypt($m, $k)
  {
      $result = '';
      for ($i = 0; $i < strlen($m); $i++) {
          $result .= chr(ord($m[$i]) ^ ord($k[$i % strlen($k)]));
      }
  
      return $result;
  }
  
  
  function create_cookie($username)
  {
      $session = json_encode(array(ANOTHER_SECRET_KEY => ANOTHER_SECRET, 'username' => $username, 'secret' => SECRET));
  
      return array('session' => base64_encode(encrypt($session, ENCRYPTION_KEY)), 'hash' => md5($session));
  }
  
  
  function check_auth()
  {
      if (isset($_COOKIE['session']) && isset($_COOKIE['hash'])) {
          $session = $_COOKIE['session'];
          $hash = $_COOKIE['hash'];
  
          $session = encrypt(base64_decode($session), ENCRYPTION_KEY);
          if (md5($session) == $hash) {
              try {
                  $session = json_decode($session, true);
                  if (isset($session['username']) && isset($session['secret']) && isset($session[ANOTHER_SECRET_KEY])) {
                      if ($session['secret'] === SECRET && $session[ANOTHER_SECRET_KEY] === ANOTHER_SECRET) {
                          return $session['username'];
                      }
                  }
              } catch (Exception $e) {
              }
          }
      }
  
      return null;
  }
  ```
</details>

Sau khi coi source thì mình đã confirm được chính xác là dùng XOR encryption rồi. Mấy hàm SQL dùng hàm `bind_param` để query cũng như `md5` password tận 3 lần nên cũng không có vẻ gì là khả thi lắm. Vì vậy mình quyết định giải quyết theo hướng tìm key để XOR lại và tạo payload.

Sau đây là flow của các hàm liên quan:

**encrypt**: Input của hàm này gồm có $m và $k tương đương với message và key. Hàm này lấy từng kí tự của message và XOR với kí tự tương ứng của key bằng công thức `chr(ord($m[$i]) ^ ord($k[$i % strlen($k)]))`. Vì kí tự tương ứng của key được chọn bằng `$i % strlen($k)` nên chắc chắn nếu message dài hơn key thì phần key sẽ bị lặp lại.

**create_cookie**: Input gồm có $username tương ứng với username đang đăng nhập. Hàm này có tác dụng tạo ra cookie `session` và `hash` cho authentication, các bước như sau:
1. Gán `$session = json_encode(array(ANOTHER_SECRET_KEY => ANOTHER_SECRET, 'username' => $username, 'secret' => SECRET));`. Theo mình mục đích là giấu cái username vào giữa đoạn JSON này để khi bạn tìm key sẽ mất công hơn (nếu không có source), vì phải tự đoán cấu trúc của phần secret này, và không biết nó XOR với cái gì.
2. Trả về `array('session' => base64_encode(encrypt($session, ENCRYPTION_KEY)), 'hash' => md5($session));`, phần `base64_encode(encrypt($session, ENCRYPTION_KEY))` chính là phần base64 mà ta bắt được trong khi coi session

**check_auth**: Hàm này thì không có input vì nó lấy data trực tiếp từ cookie (`$_COOKIE`) luôn, có nhiệm vụ là giải mã phần session kia từ `session` cookie, các bước như sau:
1. Dùng `$session = encrypt(base64_decode($session), ENCRYPTION_KEY)`, hay dùng chính hàm `encrypt` để giải mã, hợp lý với tính chất nghịch đảo của XOR.
2. Check xem `md5($session)` có giống `$hash` không, không giống thì không cho đăng nhập, giống thì qua bước 3.
3. Check xem có `$session['username']` không, và các secret đã đúng chưa, nếu không đúng cũng không cho đăng nhập nốt, đúng thì qua bước 4.
4. Cho phép đăng nhập dưới tên `$session['username']`.

Từ đó mình xác định được mục tiêu của bài là làm sao set được `$session['username']` thành `admin` để lấy flag.

Để chắc chắn là mình tạo ra được một đoạn message dài hơn key thì mình tạo một tài khoản với username rất dài, hình như mình test max là được 127 kí tự. Mình chọn 127 chữ `a`. Các bạn có thể ném `'a' * 127` vào Python để nó gen ra cho nhanh.

![][username-generator]

Đăng nhập vào, được một đoạn `session` cookie như này:

`
HEQIUDsHMiRREQhkQ0knNVRaA09UQglYTRRZTUgZFkpFExlGLAohLEARCGRWQCM3VFUGQlBQCUtJBkhJRwxVBwYHC0I/BSEgRFJTJ1ZAIzdUVQZCUFAJS0kGSElHDFUHBgcLQj8FISBEUlMnVkAjN1RVBkJQUAlLSQZISUcMVQcGBwtCPwUhIERSUydWQCM3VFUGQlBQCUtJBkhJRwxVBwYHC0I/BSEgRFJTZBsDMTNWRgJXEwtKSF0TXU1HH0cRDgoGTTEQJDNcEU8=
`

Mình dùng Python để decode ra thành byte array, sau đó convert ra int cho dễ nhìn, vì nếu không Python sẽ cố convert ra thành character, một số kí tự nằm ngoài bảng mã trông sẽ rất là dị. Các bạn có thể xem hình dưới đây

![][decode]

Rồi, giờ mình biết trong này có một đống chữ `a` trong username, đã được XOR với key, nếu key bị lặp lại thì khi XOR với đống chữ `a` này cũng sẽ ra đoạn lặp lại

Mình vứt vào Sublime Text rồi thử tìm thì thấy ngay có 32 chars này lặp đi lặp lại 3 lần:

![][repetitive-text]

XOR ngược đoạn này với chữ `a` là ra key

![][key]

Tuy nhiên vì đoạn này lặp lại liên tục, nên mình chưa biết thứ tự đúng của key như thế nào. Vậy nên mình chạy qua thử các cách sắp xếp của key (theo vòng tròn), để xem xem cách nào giải mã đúng.

![][find-key-offset]

Vậy ở đây key đúng là `[103, 102, 106, 35, 94, 100, 64, 65, 37, 51, 50, 70, 55, 33, 66, 86, 53, 52, 103, 35, 49, 49, 104, 42, 40, 103, 41, 40, 38, 109, 52, 102]`, mình để ở dạng char code cho dễ XOR và vì đôi khi key không nằm trong dải encoding nên `chr` ra trông nó không đẹp.

Sửa lại đoạn code lúc nãy tí để in ra `session` và `hash`, sau khi đã thay username thành `admin`

![][final-key-and-hash]

Đem vứt vô browser cookies, sau đó vào lại `index.php`

![][final-cookies]

### Flag:
![][flag]

```
flag{dance_like_nobody_is_watching__encrypt_like_everyone_is}
```

[view]: assets/OREO/view.png
[view-register]: assets/OREO/view-register.png
[view-get-invite-code]: assets/OREO/view-get-invite-code.png
[view-register-disabled]: assets/OREO/view-register-disabled.png
[view-invite-code]: assets/OREO/view-invite-code.png
[view-logged-in]: assets/OREO/view-logged-in.png
[view-logged-in-source]: assets/OREO/view-logged-in-source.png
[view-cookie]: assets/OREO/view-cookie.png
[view-session-cookie-url-decoded]: assets/OREO/view-session-cookie-url-decoded.png
[hint]: assets/OREO/hint.png
[username-generator]: assets/OREO/username-generator.png
[decode]: assets/OREO/decode.png
[repetitive-text]: assets/OREO/repetitive-text.png
[key]: assets/OREO/key.png
[find-key-offset]: assets/OREO/find-key-offset.png
[final-key-and-hash]: assets/OREO/final-key-and-hash.png
[final-cookies]: assets/OREO/final-cookies.png
[flag]: assets/OREO/flag.png