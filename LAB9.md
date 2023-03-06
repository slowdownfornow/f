
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
* Bước thứ hai là chạy quét cổng để xác định các cổng và dịch vụ đang mở trên máy mục tiêu. Sử dụng công cụ `Nmap` để quét cổng hơn, vì nó hoạt động hiệu quả và có sẵn trên Kali Linux theo mặc định. Trong khu vực được đánh dấu của ảnh chụp màn hình sau, chúng ta có thể thấy lệnh Nmap mà chúng ta đã sử dụng để quét các cổng trên máy mục tiêu của mình. Các cổng mở được xác định cũng có thể được nhìn thấy trong ảnh chụp màn hình bên dưới.
* Sau đó check trên giao diện web

```
http://192.168.8.152/
```
