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

