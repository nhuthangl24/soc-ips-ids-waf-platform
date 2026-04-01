# Hệ thống SOC - IPS / IDS / WAF

# Mục tiêu

* Ngăn chặn xâm nhập từ bên ngoài vào mạng nội bộ
* Bảo vệ ứng dụng web khỏi các tấn công phổ biến (SQLi, XSS, RCE)
* Giám sát và phân tích lưu lượng mạng theo thời gian thực
* Tập trung và phân tích log để hỗ trợ phản ứng sự cố
* Đảm bảo tính sẵn sàng cao của các dịch vụ quan trọng

# Thành phần hệ thống

* **pfSense + IPS**: firewall, NAT, giám sát lưu lượng mạng, IPS rules
* **WAF (DMZ)**: bảo vệ ứng dụng web, lọc các request HTTP/HTTPS độc hại
* **Web Server**: host các website và ứng dụng web nội bộ
* **Database Server**: lưu trữ dữ liệu ứng dụng
* **IDS Server**: giám sát lưu lượng nội bộ, phát hiện xâm nhập
* **ELK Stack**: thu thập, tập trung và phân tích log từ tất cả các thiết bị và dịch vụ

# Luồng dữ liệu

* Lưu lượng từ bên ngoài đi qua **pfSense + IPS** để lọc và giám sát
* Request HTTP/HTTPS được NAT tới **WAF** trong DMZ
* WAF lọc request độc hại và chuyển tiếp lưu lượng sạch tới **Web Server**
* **IDS Server** giám sát lưu lượng nội bộ, phát hiện hành vi bất thường
* Toàn bộ log từ firewall, WAF, Web Server, Database và IDS được gửi về **ELK Stack** để phân tích

# Sơ đồ mạng

* Ảnh sơ đồ mạng tổng quan
* File cấu hình mạng (VMware / PFSENSE)
* Mô tả chi tiết các subnet: WAN, LAN, DMZ và các IP tĩnh cho từng máy ảo

# Cài đặt PFSENSE

* Tạo máy ảo PFSENSE trên VMware
* Cấu hình các interface: LAN, WAN, DMZ
* Thiết lập **Firewall Rules** cơ bản theo từng interface
* NAT / Port Forward cho các dịch vụ Web Server và DVWA
* Cấu hình IPS để phát hiện và chặn các xâm nhập

# Triển khai dịch vụ web

## Windows Server + IIS

* Thiết lập IP tĩnh trong DMZ
* Cài IIS 10 và deploy site mẫu
* Kiểm tra truy cập từ LAN, DMZ và WAN

## Ubuntu + DVWA / Docker

* Thiết lập IP tĩnh trong DMZ
* Cài Docker và chạy container DVWA
* Kiểm tra truy cập web từ LAN, DMZ, WAN

# Triển khai SOC

* Cài đặt công cụ SOC (ELK, Wazuh, Graylog...)
* Thu thập logs từ pfSense, WAF, Web Server, Database, IDS
* Cấu hình dashboards, cảnh báo theo các rule bảo mật
* Giám sát tình trạng mạng và sự cố theo thời gian thực

# Cài đặt IPS / IDS

* Cài Snort hoặc Suricata
* Thiết lập rule cơ bản để phát hiện các xâm nhập mạng, malware, exploit
* Kết hợp với SOC để gửi alert và ghi log tập trung

# Triển khai WAF

* Cài ModSecurity hoặc WAF tương thích với IIS / Web Server
* Thiết lập policy để bảo vệ các endpoint web
* Kiểm tra hiệu quả lọc các request độc hại

# High Availability (HA)

* Thiết lập **pfSense HA** với CARP VIP để đảm bảo failover cho firewall
* Backup và failover cho các máy chủ quan trọng
* Kiểm tra tính khả dụng của dịch vụ trong trường hợp lỗi

# Kiểm tra & đánh giá

* Ping test từ LAN/DMZ → WAN
* Truy cập các dịch vụ web từ WAN
* Kiểm tra alert SOC / IPS / IDS / WAF
* Đánh giá failover HA và tính sẵn sàng dịch vụ

# Đang chuẩn bị

> Thông tin và nội dung đang được cập nhật, sẽ bổ sung trong thời gian sớm nhất
