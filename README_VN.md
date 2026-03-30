# Hệ thống SOC - IPS / IDS / WAF

> Hệ thống giám sát và phòng thủ an ninh mạng nhiều lớp, tích hợp SOC, IDS, IPS và WAF

==================================================
TỔNG QUAN
==================================================

Đây là hệ thống bảo mật mạng được xây dựng theo mô hình nhiều lớp
nhằm giám sát, phát hiện và ngăn chặn các cuộc tấn công mạng theo
thời gian thực.

Thành phần chính:
- pfSense + IPS
- DMZ Zone
- Web Server
- IDS Server
- Database
- SOC / SIEM

==================================================
KIẾN TRÚC HỆ THỐNG
==================================================

                    [ Internet / Người dùng ]
                              |
                              v
                +-----------------------------+
                |       pfSense + IPS         |
                |     WAN: 75.75.75.x         |
                |     LAN: 10.0.0.10          |
                +-----------------------------+
                              |
            -----------------------------------------
            |                                       |
            v                                       v
    +--------------------+               +--------------------+
    |      Vùng DMZ      |               |   Vùng bảo mật     |
    |   192.168.50.0     |               |   SOC / SIEM       |
    +--------------------+               +--------------------+
            |
    ----------------------------
    |            |             |
    v            v             v
 Web Server   IDS Server    Database

==================================================
CHỨC NĂNG CHÍNH
==================================================

[1] pfSense + IPS
- Firewall
- NAT / Routing
- Chặn truy cập trái phép
- Lọc packet độc hại

[2] IDS Server
- Phát hiện xâm nhập
- Giám sát traffic
- Cảnh báo bất thường

[3] Web Server
- Cung cấp dịch vụ web
- Chạy ứng dụng

[4] Database
- Lưu trữ dữ liệu
- Lưu log truy cập

[5] SOC / SIEM
- Thu thập log
- Dashboard giám sát
- Cảnh báo thời gian thực

==================================================
MỤC TIÊU
==================================================

- Phát hiện tấn công
- Ngăn chặn xâm nhập
- Giám sát tập trung
- Phân tích log
- Hỗ trợ phản ứng sự cố
