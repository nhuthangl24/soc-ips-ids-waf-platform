# Hệ thống SOC - IPS / IDS / WAF

> Kiến trúc hệ thống được cập nhật theo sơ đồ mạng mới nhất

---

## Tổng quan

Hệ thống được xây dựng theo mô hình phòng thủ nhiều lớp, tích hợp firewall, IPS, WAF, IDS và hệ thống tập trung log ELK nhằm giám sát, phát hiện và ngăn chặn các mối đe dọa mạng.

## Kiến trúc hệ thống

```text
[Attacker] --> [Internet] --> [pfSense + IPS]
                               WAN: .75.249
                               LAN: .10.10
                                      |
                         NAT 80/443 -> 192.168.30.40
                                      |
                                [DMZ - WAF]
                               192.168.30.40
                                      |
                     -------------------------------
                     |                             |
                     v                             v
            [DMZ-WEB]                     [Vùng bảo mật]
      Web: 192.168.20.30                ELK: 192.168.40.60
      DB : 192.168.20.50                Nguồn log:
                                         - WAF -> ELK
                                         - IDS -> ELK
                                         - pfSense + IPS -> ELK

               IDS Server: 192.168.40.50
```

## Thành phần hệ thống

* pfSense + IPS
* WAF (DMZ)
* Web Server
* Database Server
* IDS Server
* ELK Stack

## Luồng dữ liệu

* Lưu lượng từ bên ngoài đi qua pfSense + IPS
* Request HTTP/HTTPS được NAT tới WAF
* WAF lọc và chuyển tiếp lưu lượng sạch tới Web Server
* IDS giám sát lưu lượng nội bộ
* Toàn bộ log được tập trung về ELK

## Mục tiêu

* Ngăn chặn xâm nhập
* Bảo vệ ứng dụng web
* Giám sát lưu lượng mạng
* Phân tích log tập trung
* Hỗ trợ phản ứng sự cố
