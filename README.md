# SOC - IPS / IDS / WAF Security Monitoring System

## Giới thiệu

Đây là trang tổng quan của dự án, giúp người đọc nắm nhanh kiến trúc, mục tiêu và điều hướng tới các tài liệu chi tiết.

Dự án mô phỏng một hệ thống phòng thủ an ninh mạng nhiều lớp, bao gồm firewall, WAF, IDS/IPS và trung tâm phân tích log ELK.

## Tài liệu chi tiết

Vui lòng xem các tài liệu bên dưới để đọc chi tiết theo ngôn ngữ mong muốn:

* [**README_VI.md**](https://github.com/nhuthangl24/soc-ips-ids-waf-platform/blob/main/README_VI.md) : Tài liệu chi tiết tiếng Việt
* [**README_EN.md**](https://github.com/nhuthangl24/soc-ips-ids-waf-platform/blob/main/README_EN.md) : Detailed documentation in English

## Nội dung chính của dự án

* pfSense + IPS
* WAF tại vùng DMZ
* Web Server và Database
* IDS Server giám sát mạng nội bộ
* ELK Stack tập trung log và dashboard SOC

## Mục tiêu

README này đóng vai trò trang điều hướng chính của repository.
Người dùng có thể bắt đầu từ đây và truy cập sang các tài liệu chi tiết để xem kiến trúc, luồng dữ liệu và mô tả thành phần hệ thống.

## Đóng góp

Mọi đóng góp cho dự án đều được hoan nghênh.

Nếu bạn muốn cải thiện hệ thống, vui lòng:

1. Fork repository
2. Tạo branch mới cho tính năng hoặc bản sửa lỗi
3. Commit các thay đổi với mô tả rõ ràng
4. Tạo Pull Request để được xem xét

Các nội dung có thể đóng góp:

* Cải thiện kiến trúc SOC / SIEM
* Tối ưu rule cho IDS / IPS / WAF
* Bổ sung dashboard ELK
* Mô phỏng kịch bản tấn công và phòng thủ
* Hoàn thiện tài liệu kỹ thuật

Mọi ý kiến đóng góp, báo lỗi hoặc đề xuất tính năng mới đều rất được khuyến khích.
