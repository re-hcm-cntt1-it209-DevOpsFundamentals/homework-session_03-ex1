# Bài 1: Thay đổi cổng kết nối SSH

## 1. Mục tiêu

Thay đổi cổng SSH mặc định từ `22` sang `2222` trên Ubuntu Server.

Mục đích của bài thực hành là làm quen với việc thay đổi cấu hình SSH và cấu hình tường lửa phù hợp trước khi restart SSH service để tránh mất kết nối với máy chủ.

Trong bài này, tôi thực hiện bằng tài khoản `devops` có quyền `sudo`.

---

## 2. Kiểm tra kết nối SSH hiện tại

Trước khi thay đổi cấu hình, tôi kiểm tra kết nối SSH hiện tại bằng port `22`.

Từ máy tính cá nhân:

```bash
ssh -i ~/.ssh/id_ed25519 devops@IP_ADDRESS_DROPLET
```

Sau khi đăng nhập thành công, kiểm tra user hiện tại:

```bash
whoami
```

Kết quả:

```text
devops
```

Tôi giữ nguyên terminal SSH hiện tại trong suốt quá trình thực hiện bài để phòng trường hợp cấu hình SSH mới xảy ra lỗi.

---

## 3. Kiểm tra trạng thái UFW

Trên Droplet, tôi kiểm tra trạng thái UFW:

```bash
sudo ufw status
```

Nếu port 22 đang được cho phép từ bài trước, kết quả có thể tương tự:

```text
22/tcp    ALLOW
80/tcp    ALLOW
```

---

## 4. Mở port 2222 trên UFW

Trước khi thay đổi cổng SSH, tôi mở port `2222/tcp` trên UFW:

```bash
sudo ufw allow 2222/tcp
```

Kiểm tra lại:

```bash
sudo ufw status
```

Kết quả phải có:

```text
2222/tcp    ALLOW
```

Việc mở port mới trước khi restart SSH giúp tránh trường hợp SSH daemon chuyển sang port 2222 nhưng firewall lại chặn kết nối tới port này.

---

## 5. Thay đổi port SSH

Tôi sử dụng tài khoản `devops` và quyền `sudo` để chỉnh sửa file cấu hình SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Tìm dòng:

```text
#Port 22
```

hoặc:

```text
Port 22
```

Sau đó thay đổi thành:

```text
Port 2222
```

Lưu file cấu hình sau khi chỉnh sửa.

---

## 6. Kiểm tra cấu hình SSH

Trước khi restart SSH, tôi kiểm tra cú pháp file cấu hình:

```bash
sudo sshd -t
```

Nếu không có output thì cấu hình không có lỗi cú pháp.

Sau đó có thể kiểm tra port SSH đang được cấu hình:

```bash
sudo sshd -T | grep port
```

Kết quả mong đợi:

```text
port 2222
```

---

## 7. Restart SSH service

Sau khi kiểm tra cấu hình thành công, tôi restart SSH:

```bash
sudo systemctl restart ssh
```

Kiểm tra trạng thái:

```bash
sudo systemctl status ssh
```

SSH service cần ở trạng thái đang hoạt động.

Lưu ý: Tôi không đóng terminal SSH hiện tại ngay sau khi restart service.

---

## 8. Kiểm tra kết nối bằng port 2222

Tôi mở một cửa sổ Terminal mới trên máy tính cá nhân.

Sử dụng port mới để đăng nhập:

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 devops@IP_ADDRESS_DROPLET
```

Nếu kết nối thành công, kiểm tra:

```bash
whoami
```

Kết quả:

```text
devops
```

Điều này chứng minh SSH đã chuyển sang sử dụng port `2222`.

Ảnh/log minh họa:

> Chèn ảnh Terminal đăng nhập thành công bằng port 2222 tại đây.

---

## 9. Kiểm tra port SSH cũ

Sau khi xác nhận port `2222` hoạt động, tôi thử kết nối lại bằng port `22`:

```bash
ssh -i ~/.ssh/id_ed25519 -p 22 devops@IP_ADDRESS_DROPLET
```

Kết nối tới port 22 không còn thành công vì SSH daemon không còn lắng nghe trên port này và port 22 không còn được sử dụng cho SSH.

Kết quả có thể là:

```text
Connection refused
```

hoặc:

```text
Connection timed out
```

tùy theo cấu hình firewall.

Ảnh/log minh họa:

> Chèn ảnh Terminal thể hiện kết nối port 22 không thành công tại đây.

---

## 10. Kiểm tra các port SSH đang lắng nghe

Có thể sử dụng lệnh sau trên Droplet:

```bash
sudo ss -tlnp | grep ssh
```

Kết quả cần thể hiện SSH đang lắng nghe trên port `2222`.

Ví dụ:

```text
LISTEN 0 128 0.0.0.0:2222 0.0.0.0:* users:(("sshd",...))
```

Điều này giúp kiểm tra trực tiếp SSH daemon đang sử dụng port mới.

---

## 11. Kết quả

Sau khi thực hiện bài tập, tôi đã:

* Sử dụng tài khoản `devops` có quyền `sudo`.
* Mở port `2222/tcp` trên UFW trước khi restart SSH.
* Thay đổi SSH port từ `22` sang `2222`.
* Kiểm tra cấu hình SSH trước khi áp dụng.
* Restart SSH service.
* Đăng nhập thành công bằng SSH Key thông qua port `2222`.
* Kiểm tra port `22` và xác nhận không còn sử dụng để kết nối SSH.

Cấu hình cuối cùng:

```text
SSH port: 2222
UFW: 2222/tcp ALLOW
User: devops
Authentication: SSH Key
```

---

## 12. Các lệnh chính đã sử dụng

```bash
sudo ufw status

sudo ufw allow 2222/tcp

sudo nano /etc/ssh/sshd_config

sudo sshd -t

sudo sshd -T | grep port

sudo systemctl restart ssh

sudo systemctl status ssh

sudo ss -tlnp | grep ssh
```

Từ máy tính cá nhân:

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 devops@IP_ADDRESS_DROPLET
```

Kiểm tra port cũ:

```bash
ssh -i ~/.ssh/id_ed25519 -p 22 devops@IP_ADDRESS_DROPLET
```

---

## 13. Cấu trúc thư mục bài tập

```text
homework/
└── session_03/
    └── ex1/
        └── README.md
```
