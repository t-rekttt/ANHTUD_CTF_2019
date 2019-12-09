## Facebook
### 100 points

Có lẽ điều duy nhất để nói về bài này là nên thử thay vì cố nghĩ nhiều :))

### Đề bài
<pre>
RIP facebook của tui đi
<a href="http://45.76.161.61:8002/">http://45.76.161.61:8002/</a>
</pre>

### Giao diện của bài

![view-login]

### Lời giải

Bấm `Download sourcecode`, được một đoạn source như thế này

![view-source]

Rõ ràng phải có gì đó sử dụng hàm `make_token` này, nên vì chưa biết nên tạm để đó đã

`view-source` trang login, chẳng có tí thông tin nào

![view-source-login]

Nên mình bấm thử vào `Forgot Password?` xem sao

![view-forgot-password]

Có vẻ như ta sẽ phải nhập `username` vào để gửi token, sau đó forgot password để đăng nhập vào tài khoản.
`view-source` tiếp trang này, thấy ngay `username` là `yeuthammevo_9x`

![view-source-forgot-password]

Nhập thử `username` vào form và bấm `Send token to email`, trang chỉ hiện thêm dòng `Token has been sent to your email!`
Rồi, có lẽ bây giờ việc còn lại của mình là tìm cách lấy nốt cái token. Mình đem chạy thử đoạn source kia thì phát hiện ra cái token được tạo ra từ hàm này không thay đổi, luôn là `29601931810389988476395720444006`

![view-make-token]

Đem vứt đoạn token đấy vào trang `Verify token` là xong

### Flag

![flag]

```
Flag{du0ngqu4_tim_c0c0}
```

[view-login]: assets/FACEBOOK/view-login.png
[view-source]: assets/FACEBOOK/view-source.png
[view-source-login]: assets/FACEBOOK/view-source-login.png
[view-forgot-password]: assets/FACEBOOK/view-forgot-password.png
[view-source-forgot-password]: assets/FACEBOOK/view-source-forgot-password.png 
[view-make-token]: assets/FACEBOOK/view-make-token.png
[flag]: assets/FACEBOOK/flag.png