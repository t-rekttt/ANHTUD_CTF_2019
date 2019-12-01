## Build PC
### 100 points

Không có nhiều điều để nói về bài này lắm :))

### Đề bài
<pre>
<a href="http://35.240.153.120:8003">http://35.240.153.120:8003</a>

<a href="assets/BUILD_PC/source.zip">source.zip</a>
</pre>

### Giao diện của bài:

![][view-register]

![][view-login]

### Lời giải
Tạo 1 tài khoản, đăng nhập thử

![][view-shop]

Có vẻ như với bài này chúng ta sẽ phải đi shopping rồi. Để ý thấy balance của chúng ta có 60 củ.

Nghía qua source tí, thấy nó tương ứng với dòng này trong file `RegisterController.php`. Mỗi user khi đăng ký đều được cấp 60 củ giống nhau.

![][view-register-controller-src]

Tuy nhiên flag thì tận 100 củ cơ, vậy 40 củ nữa chúng ta phải kiếm ở đâu?

Thử tất tay con RTX 2080 này xem nào. Sau khi hết tiền và đòi mua cố, chúng ta nhận được một câu chửi rất gắt :<.

![][view-out-of-money]

Mua xong, thử qua trang `Cart`, thấy có chức năng bán, bán hết thì tiền của ta lại về như ban đầu.

![][view-cart]

Vậy cơ bản mục tiêu của bài này là mua bán làm sao để biến balance từ 60 củ thành 100 củ để mua flag.

Soi lại trong source, thấy câu chửi lúc nãy nằm trong `ShopController.php`.

![][view-shop-controller-src]

Để ý chút thì ta thấy hàm `buyItem` rất có vấn đề, khi mà check xem user có đủ tiền không bằng `$user->money >= $item->price`, sau đó đưa luôn item vào giỏ hàng của user bằng lệnh `$con->queryUpdate("insert into user_items(user_id, item_id) values(?, ?)", [$user->id, $item->id])`, rồi mới trừ tiền user bằng `$user->money -= $item->price`.

Điều này có nghĩa rằng việc trừ tiền user sẽ bị delay một chút bởi việc kết nối tới database và thực hiện query kia. Thế thì khả năng là nếu chúng ta gửi một loạt requests đủ nhanh, nhanh hơn thời gian delay của server khi query database, thì chúng ta sẽ mua được nhiều hơn số tiền mình có.

Soi thêm hàm `sellItem` trong `CartController.php`, cũng thấy có vấn đề tương tự, tuy nhiên ở đây tiền lại được cộng trước khi sản phẩm bị xoá khỏi giỏ hàng, chứng tỏ đây là dụng ý của người ra đề.

![][view-cart-controller-src]

Sau khi biết hướng đi rồi, cộng với việc biết Javascript chạy bất đồng bộ và có tốc độ request khá là ổn, nên mình viết ngay một payload thế này để chạy trên browser console

```javascript
cookie = '<cookie>';

buy = () => fetch('http://35.240.153.120:8003/index.php?page=shop', { body: `item_id=2`, headers: {'Cookie': cookie, 'Content-Type': 'application/x-www-form-urlencoded'}, method: 'POST' })
sell = () => fetch('http://35.240.153.120:8003/index.php?page=cart', { body: `item_id=2`, headers: {'Cookie': cookie, 'Content-Type': 'application/x-www-form-urlencoded'}, method: 'POST' })

(async() => {
	let promises = [];

	for (let i = 0; i < 5; i++) {
	    // Send 5 sell requests at once to trigger race condition
		for (let j = 0; j <= 5; j++) promises.push(buy());

		await Promise.all(promises);

        // Sell 1 back to have enough money for the next buying round
		await sell();
    }
})();
```

Vì là `Race condition` nên mình nghĩ đôi khi nó trigger một cách hên xui, thành ra mình cứ phải chạy lại vài lần mới exploit được.

Sau vài lần chạy payload thì cuối cùng mình đã tích đủ thận để bán đi mua flag rùi :3

![][view-exploited]

Thực ra khi làm bài này thì mình không phân tích nhiều như thế mà nhảy vào spam requests luôn, thành ra mình phát hiện ra cách làm trước cả khi đọc source. Tuy nhiên mình vẫn muốn phân tích đầy đủ theo cách tư duy của mình để mọi người có thể hiểu một cách dễ dàng mạch lạc nhất :D.
### Flag
![][flag]
```
flag{Rac3_c0nditi0n__help_y0u}
```

[view-register]: assets/BUILD_PC/view-register.png
[view-login]: assets/BUILD_PC/view-login.png
[view-shop]: assets/BUILD_PC/view-shop.png
[view-register-controller-src]: assets/BUILD_PC/view-register-controller.png
[view-out-of-money]: assets/BUILD_PC/view-out-of-money.png
[view-shop-controller-src]: assets/BUILD_PC/view-shop-controller.png
[view-cart]: assets/BUILD_PC/view-cart.png
[view-cart-controller-src]: assets/BUILD_PC/view-cart-controller.png
[view-exploited]: assets/BUILD_PC/view-exploited.png
[flag]: assets/BUILD_PC/flag.png