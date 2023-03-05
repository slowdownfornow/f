# LAB 8
## Chuẩn bị
* Máy kali có ip 172.16.203.129
![image](https://user-images.githubusercontent.com/91528234/222954758-a550f88e-2eb5-41e7-b640-ff78bd3a8937.png)

* Máy nạn nhân chưa rõ ip
![image](https://user-images.githubusercontent.com/91528234/222948225-4077e6ea-938e-4bf5-bc56-4a579d92470a.png)
## Quét mạng
* Quét mạng bằng công cụ `netdiscover`

![image](https://user-images.githubusercontent.com/91528234/222954800-fadad886-be42-4c21-8988-50b0a8a12831.png)
* Chúng ta quét được ip máy nạn nhân có ip `172.16.203.128`
* Chúng tôi hiện đang bắt đầu Nmap để thúc đẩy quá trình này. Chúng tôi đã thực hiện (-A) để liệt kê cổng mở và phát hiện ra thông tin cổng sau
```A
nmap -A 172.16.203.128
```
* Theo đầu ra Nmap, chúng tôi có

  * một máy chủ SSH chạy trên cổng 22

  * một dịch vụ HTTP đang chạy (Máy chủ Apache) trên cổng 80, cũng như một trang http-git
![image](https://user-images.githubusercontent.com/91528234/222954858-e94cab2d-72f5-463e-97ea-f54b95f6f02a.png)

## Liệt kê
* Trước tiên, chúng tôi sẽ cố gắng sử dụng HTTP. Hãy kiểm tra cổng 80 để xem có điều gì thú vị xuất hiện không. Vì Máy chủ Apache đang lắng nghe trên cổng 80 nên chúng tôi có thể xác minh ngay lập tức trên trình duyệt.
![image](https://user-images.githubusercontent.com/91528234/222954871-4087aaa2-d0a9-41d4-b4b7-8ba9c95ec503.png)

* Ngoại trừ trang đăng nhập, trang này không chứa thông tin hữu ích nào. Vì vậy, chúng tôi quyết định xem trước trang đăng nhập.
![image](https://user-images.githubusercontent.com/91528234/222948784-7e779f73-1dd1-4ad4-b21e-7b3c756abf4f.png)
* Sau đó, chúng tôi quyết định xem trang http-git mà chúng tôi đã phát hiện ra trước đây trong quá trình quét tích cực Nmap.
![image](https://user-images.githubusercontent.com/91528234/222948814-c2e29ead-2f7c-4b42-814d-2a54050c6c11.png)
* Chúng tôi đã giới thiệu một công cụ gọi là gitdumper để cải thiện tính thẩm mỹ của trang http-git này. Nó là một công cụ để có được một kho lưu trữ git từ một trang web để hiểu rõ hơn về tập dữ liệu.

* Chúng tôi chỉ cần sử dụng chức năng git clone để cài đặt cái này.
```
git clone https://github.com/arthaud/git-dumper.git
cd git-dumper
```
* Sau khi tải xuống công cụ, chúng tôi cố gắng chạy nó bằng python.

Một điều khác mà chúng tôi phải làm là cung cấp cho họ một tên thư mục để lưu các nhật ký git này (trong trường hợp của chúng tôi, chúng tôi đặt tên này là bản sao lưu cho trang http-git này).