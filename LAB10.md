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
```![image](https://user-images.githubusercontent.com/91528234/223043812-b8f2daf1-4620-4b5a-a8fa-8b411492176b.png)