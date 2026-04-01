# Hệ thống SOC - IPS / IDS / WAF

# Thành phần hệ thống

- **pfSense + IPS**: firewall, NAT, giám sát lưu lượng mạng, IPS rules
- **WAF (DMZ)**: bảo vệ ứng dụng web, lọc các request HTTP/HTTPS độc hại
- **Web Server**: host các website và ứng dụng web nội bộ
- **Database Server**: lưu trữ dữ liệu ứng dụng
- **IDS Server**: giám sát lưu lượng nội bộ, phát hiện xâm nhập
- **ELK Stack**: thu thập, tập trung và phân tích log từ tất cả các thiết bị và dịch vụ

# Luồng dữ liệu

- Lưu lượng từ bên ngoài đi qua **pfSense + IPS** để lọc và giám sát
- Request HTTP/HTTPS được NAT tới **WAF** trong DMZ
- WAF lọc request độc hại và chuyển tiếp lưu lượng sạch tới **Web Server**
- **IDS Server** giám sát lưu lượng nội bộ, phát hiện hành vi bất thường
- Toàn bộ log từ firewall, WAF, Web Server, Database và IDS được gửi về **ELK Stack** để phân tích

# Sơ đồ mạng

Dưới đây là sơ đồ mạng tổng quan về dự án

![Sơ đồ mạng](./img/network_map.png)

Trên sơ đồ có thể thấy các thiết bị/thành phần chính gồm: pfSense + IPS (chính/backup), WAF, Web Server, Database, IDS Server, ELK (SIEM), cùng các vùng WAN/LAN/DMZ.

Sau khi nắm được sơ đồ, dưới đây là cấu hình VMware Virtual Network (VMnet) và các subnet ảo dùng trong bài lab.

![IP](./img/ip.png)

Đây là file cấu hình mạng cho ai quan tâm có thể dùng file cấu hình này để import và dựng đúng như sơ đồ trên: [SOC NETWORK LAB](./files/SOC_NETWORK)

## Network Subnet Design

- **WAN Subnet:** `192.168.75.0/24`
  - Gateway: `192.168.75.2`
  - Purpose: kết nối Internet / upstream router

- **LAN Subnet:** `192.168.10.0/24`
  - Gateway: `192.168.10.1`
  - Purpose: mạng nội bộ cho user và máy quản trị

- **DMZ Subnet:** `192.168.20.0/24`
  - Gateway: `192.168.20.1`
  - Purpose: chứa WAF

- **DMZ_WEB Subnet:** `192.168.30.0/24`
  - Gateway: `192.168.30.1`
  - Purpose: chứa Web Server, IDS Server, Database

  - **SEC-ZONE Subnet:** `192.168.40.0/24`
  - Gateway: `192.168.40.1`
  - Purpose: chứa ELK

## Static IP Assignment

- pfSense Firewall:
  - WAN : `192.168.75.131`

  - LAN : `192.168.10.10`

- pfSense Firewall (Backup):
  - WAN : `192.168.75.135`

  - LAN : `192.168.75.20`

- ELK Stack: `192.168.40.60`
- **IDS Server:**
  - **NIC 1:** Không cấu hình địa chỉ IP. NIC này được sử dụng ở chế độ **sniffing/monitoring** để giám sát lưu lượng trong vùng mạng **DMZ_WEB**.
  - **NIC 2:** `192.168.40.50` - Được sử dụng để truyền log và cảnh báo về hệ thống **ELK Stack** nhằm phục vụ việc tổng hợp, phân tích và giám sát sự kiện.
- Kali Linux: `192.168.75.10`
- DVWA Web Server: `192.168.20.30`
- Reverse Proxy / WAF: `192.168.30.40`

# Cài đặt PFSENSE

- Tạo máy ảo PFSENSE trên VMware
- Cấu hình các interface: LAN, WAN, DMZ
- Thiết lập **Firewall Rules** cơ bản theo từng interface
- NAT / Port Forward cho các dịch vụ Web Server và DVWA
- Cấu hình IPS để phát hiện và chặn các xâm nhập

# Triển khai dịch vụ web

## Windows Server + IIS

- Thiết lập IP tĩnh trong DMZ
- Cài IIS 10 và deploy site mẫu
- Kiểm tra truy cập từ LAN, DMZ và WAN

## Ubuntu + DVWA / Docker

- Thiết lập IP tĩnh trong DMZ
- Cài Docker và chạy container DVWA
- Kiểm tra truy cập web từ LAN, DMZ, WAN

# Triển khai SOC

- Cài đặt công cụ SOC (ELK, Wazuh, Graylog...)
- Thu thập logs từ pfSense, WAF, Web Server, Database, IDS
- Cấu hình dashboards, cảnh báo theo các rule bảo mật
- Giám sát tình trạng mạng và sự cố theo thời gian thực

# Cài đặt IPS / IDS

- Cài Snort hoặc Suricata
- Thiết lập rule cơ bản để phát hiện các xâm nhập mạng, malware, exploit
- Kết hợp với SOC để gửi alert và ghi log tập trung

# Triển khai WAF

- Cài ModSecurity hoặc WAF tương thích với IIS / Web Server
- Thiết lập policy để bảo vệ các endpoint web
- Kiểm tra hiệu quả lọc các request độc hại

# High Availability (HA)

- Thiết lập **pfSense HA** với CARP VIP để đảm bảo failover cho firewall
- Backup và failover cho các máy chủ quan trọng
- Kiểm tra tính khả dụng của dịch vụ trong trường hợp lỗi

# Kiểm tra & đánh giá

- Ping test từ LAN/DMZ → WAN
- Truy cập các dịch vụ web từ WAN
- Kiểm tra alert SOC / IPS / IDS / WAF
- Đánh giá failover HA và tính sẵn sàng dịch vụ

# Đang chuẩn bị

> Thông tin và nội dung đang được cập nhật, sẽ bổ sung trong thời gian sớm nhất
