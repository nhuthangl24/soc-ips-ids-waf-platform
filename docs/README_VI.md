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

![Sơ đồ mạng](../images/network_map.png)

Trên sơ đồ có thể thấy các thiết bị/thành phần chính gồm: pfSense + IPS (chính/backup), WAF, Web Server, Database, IDS Server, ELK (SIEM), cùng các vùng WAN/LAN/DMZ.

Sau khi nắm được sơ đồ, dưới đây là cấu hình VMware Virtual Network (VMnet) và các subnet ảo dùng trong bài lab.

![IP](../images/ip.png)

Đây là file cấu hình mạng cho ai quan tâm có thể dùng file cấu hình này để import và dựng đúng như sơ đồ trên: [SOC NETWORK LAB](../files)

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

Tại đây sẽ cài đặt pfSense phiên bản 2.7.2 CE, sau đó cập nhật lên bản mới nhất để giảm độ phức tạp.

- Link tải [pfSense 2.7.2 CE](https://repo.ialab.dsu.edu/pfsense/pfSense-CE-2.7.2-RELEASE-amd64.iso.gz)

Sau khi cài xong ta vào VMWARE để tiến hành cài đạt pfSense

![pfSense_01](../images/INSTALL/install_pfsense.png)

Đến bước này bấm vào **Customize Hardware** để thay đổi thông số trong pfSense

![pfSense_02](../images/INSTALL/pfsense_02.png)

## Chính các thông số như sau :

| Component             | Configuration                         |
| --------------------- | ------------------------------------- |
| **RAM**               | `2GB` or `4GB` (Tuỳ vào cấu hình máy) |
| **Processor**         | `Number of processors: 2`             |
| **Network Adapter 1** | `NAT`                                 |
| **Network Adapter 2** | `Custom (VMnet1)`                     |
| **Network Adapter 3** | `Custom (VMnet2)`                     |
| **Network Adapter 4** | `Custom (VMnet3)`                     |
| **Network Adapter 5** | `Custom (VMnet4)`                     |
| **Network Adapter 6** | `Custom (VMnet5)`                     |

Sau đó **OK** và **Finish** để tiến hành cài đạt pfSense, trong pfSense sẽ hiển thị giao diện cài đạt lần lượt chọn **Accept** -> **OK** -> **OK** -> **Select** -> **OK** màn hình sau sẽ xuất hiện:

![pfsense_03](../images/INSTALL/pfsense_03.png)

Đến đây bấm dấu cách để chọn sau đó **OK** , **Yes** và **Reboot** để tiếp tục

Cấu hình địa chỉ IP LAN để truy cập vào pfSense

![pfsense_04](../images/INSTALL/pfsense.png)

- Chọn 2 - **Assign Interfaces**

![pfsense_05](../images/INSTALL/pfsense_05.png)

- Chọn 2 - **LAN (em1 - static)**

![pfsense_06](../images/INSTALL/pfsense_06.png)

- Trong trường hợp này, `em1` tương ứng với `VMnet1`.
- Đầu tiên điền địa chỉ IP muốn đặt cho LAN , ví dụ ở đây sẽ đặt là `192.168.10.35`
- Enter a new LAN IPv4 Subnet sẽ để là `24`

![pfsense_07](../images/INSTALL/pfsense_07.png)

- Tại bước này, thực hiện lần lượt các thao tác: nhấn `Enter`, nhập `n`, tiếp tục nhấn `Enter`, sau đó nhập `n` để hoàn tất quá trình cấu hình.

# Lưu ý

- Để truy cập giao diện GUI của pfSense, máy client phải được cấu hình IP nằm **cùng subnet / cùng dải mạng** với địa chỉ LAN của pfSense.

Sau khi hoàn tất quá trình cài đặt và cấu hình địa chỉ IP LAN cho pfSense, truy cập vào giao diện quản trị (GUI) thông qua địa chỉ IP LAN đã cấu hình ở mục trước.

> Địa chỉ GUI của pfSense trong bài lab này là: `https://192.168.10.35`

![pfsense_08](../images/INSTALL/pfsense_08.png)

Tài khoản và mật khẩu mặc định của pfSense là : `admin / pfsense`

Sau khi truy cập thành công vào pfSense, bước đầu tiên là thực hiện cấu hình ban đầu thông qua **Setup Wizard**.

![GUI_PFSENSE](../images/GUI/GUI_pfsense.png)

- Ở bước này ta cấu hình hostname tuỳ ý ở bài lab này sẽ để mặc định là `pfSense`
- Primary DNS Server : `8.8.8.8` (Hoặc bỏ trống)
- Secondary DNS Server : `8.8.4.4` (Hoặc bỏ trống)

![GUI_PFSENSE](../images/GUI/GUI_pfsense1.png)

- Tiếp tục **Next** tới bước này , ở đây TimeZone sẽ chọn `Asia/Ho_Chi_Minh`
  và tiếp tục bấm **Next**

![GUI_PFSENSE](../images/GUI/GUI_pfsense3.png)

- Ở bước này ta tích bỏ chọn 2 ô `Block private networks from entering via WAN` và `Block non-Internet routed networks from entering via WAN`

![GUI_PFSENSE](../images/GUI/GUI_pfsense4.png)

- Bấm **Next** tới bước này , điền mật khẩu mới để truy cập vào pfSense , ở bài lab này vẫn để nguyên mật khẩu cũ là `pfsense`

![GUI_PFSENSE](../images/GUI/GUI_pfsense5.png)

- Đợi tầm 15s - 20s

![GUI_PFSENSE](../images/GUI/GUI_pfsense6.png)

- Bấm **Next** để vào giao diện Dashboard của pfSense

![GUI_PFSENSE](../images/GUI/GUI_pfsense7.png)

## Cập nhật pfSense lên bản mới nhất

Để thuận tiện việc có thể làm 1 bài lab suôn sẻ, cần cập nhật pfSense lên phiên bản mới nhất để tránh phát sinh lỗi xảy ra khi làm lab , dưới đây là các bước để cập nhật pfSense

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense1.png)

- Chọn `System` -> `Update`

- Sau khi vào phần `Update` bấm vào `Branch` và chọn `Current Stable Version (2.8.1)` (Hiện tại phiên bản 2.8.1 là bản mới nhất khi làm bài lab này)

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense3.png)

- Nếu bị lỗi `Unable to check for updates` thì làm theo các bước sau

- Chọn `Diagnostics` -> `Command Prompt` và dán các lệnh sau vào ô `Execute Shell Command`

| Bước  | Lệnh                                                      |
| ----- | --------------------------------------------------------- |
| **1** | `certctl rehash`                                          |
| **2** | `pkg-static update`                                       |
| **3** | `pkg-static install -fy pkg pfSense-repo pfSense-upgrade` |

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense4.png)

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense5.png)

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense6.png)

- Sau khi thực hiện các lệnh trong bảng trên, tiến hành quay trở cập nhật cho pfSense.
- Bấm vào `Confirm` để cập nhật pfSense

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense7.png)

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense8.png)

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense9.png)

- Sau khi quá trình cập nhật hoàn tất, pfSense sẽ tự động khởi động lại. Sau khi reboot xong, tiến hành kiểm tra lại phiên bản của pfSense để xác nhận cập nhật đã thành công.

![UPDATE_pfSense](../images/UPDATE/UPDATE_pfsense10.png)

## Thiết lập **Firewall Rules** cơ bản theo từng interface

- Nếu muốn truy cập pfSense từ WAN thì thiết lập rule như dưới đây

![pfSense_Rules](../images/Rules/Rule_7.png)

- Chọn **Add** và điền các thông số giống như sau

| Trường             | Giá trị cấu hình | Mô tả                                                                 |
| ------------------ | ---------------- | --------------------------------------------------------------------- |
| **Action**         | `Pass`           | Cho phép các gói tin thỏa mãn điều kiện của rule được đi qua firewall |
| **Interface**      | `WAN`            | Áp dụng rule cho lưu lượng đi vào từ cổng WAN                         |
| **Address Family** | `IPv4`           | Rule áp dụng cho địa chỉ IPv4                                         |
| **Protocol**       | `Any`            | Áp dụng cho mọi giao thức (TCP, UDP, ICMP,...)                        |
| **Source**         | `Any`            | Chấp nhận gói tin từ mọi địa chỉ nguồn                                |
| **Destination**    | `Any`            | Cho phép đi đến mọi địa chỉ đích                                      |

- Sau khi thiết lập các thông số như trên thì bấm **Save** để lưu lại và thử truy cập lại vào **pfSense** từ cổng **WAN**

![pfSense_Rules](../images/Rules/Rule_9.png)

- **Ở bài lab này không set Rule truy cập từ cổng WAN vào pfSense nên sẽ không có rule đó**

![pfSense_Rules](../images/Rules/Rule_1.png)

- Tại LAN tạo các Rules như sau

| STT | Trạng thái | Protocol   | Source        | Source Port | Destination     | Dest Port   | Ý nghĩa                                                                     |
| --- | ---------- | ---------- | ------------- | ----------- | --------------- | ----------- | --------------------------------------------------------------------------- |
| 1   | Allow      | `*`        | `*`           | `*`         | `LAN Address`   | `2402`      | Cho phép truy cập vào pfSense qua cổng quản trị để tránh tự khóa chính mình |
| 2   | Block      | `IPv4 TCP` | `LAN subnets` | `*`         | `192.168.20.30` | `80 (HTTP)` | Chặn máy trong LAN truy cập web server tại IP `192.168.20.30` qua port `80` |
| 3   | Allow      | `IPv4 *`   | `LAN subnets` | `*`         | `*`             | `*`         | Cho phép toàn bộ máy trong LAN truy cập mọi nơi                             |
| 4   | Allow      | `IPv6 *`   | `LAN subnets` | `*`         | `*`             | `*`         | Cho phép truy cập toàn bộ bằng IPv6                                         |

![pfSense_Rules](../images/Rules/Rule_2.png)

- Tại DMZ tạo các Rules như sau

| STT | Trạng thái | Protocol | Source          | Source Port | Destination     | Dest Port   | Action | Ý nghĩa                                                |
| --- | ---------- | -------- | --------------- | ----------- | --------------- | ----------- | ------ | ------------------------------------------------------ |
| 1   | Allow      | IPv4 TCP | `192.168.30.40` | `*`         | `192.168.20.30` | `80 (HTTP)` | Pass   | Cho phép WAF/host cụ thể truy cập Web Server trong DMZ |
| 2   | Block      | IPv4 \*  | `*`             | `*`         | `192.168.20.30` | `*`         | Block  | Chặn toàn bộ truy cập khác tới Web Server              |
| 3   | Allow      | IPv4 \*  | `DMZ subnets`   | `*`         | `*`             | `*`         | Pass   | Cho phép các máy trong DMZ đi ra ngoài                 |

![pfSense_Rules](../images/Rules/Rule_3.png)

- Tại DMZWAF tạo các Rules như sau

  | STT | Trạng thái | Protocol | Source           | Destination | Action | Ý nghĩa                                            |
  | --- | ---------- | -------- | ---------------- | ----------- | ------ | -------------------------------------------------- |
  | 1   | Allow      | IPv4 \_  | `DMZWAF subnets` | `*`         | Pass   | Cho phép toàn bộ subnet WAF truy cập các zone khác |

![pfSense_Rules](../images/Rules/Rule_4.png)

- Phần này sẽ làm lại các rules khác , chưa làm phần này

![pfSense_Rules](../images/Rules/Rule_5.png)

- Tại PFSENSESYNC tạo các Rules như sau

| STT | Trạng thái | Protocol | Source                | Destination | Action | Ý nghĩa                             |
| --- | ---------- | -------- | --------------------- | ----------- | ------ | ----------------------------------- |
| 1   | Allow      | IPv4 \*  | `PFSENSESYNC subnets` | `*`         | Pass   | Cho phép đồng bộ HA giữa 2 firewall |

## NAT / Port Forward cho các dịch vụ Web Server

- Vào **Firewall** -> **NAT** -> **Port Forward** và chọn **Add** và chỉnh cấu hình lại như dưới đây

![pfSense_Rules](../images/NAT/NAT_1.png)

| Trường                      | Giá trị            | Mô tả                                      |
| --------------------------- | ------------------ | ------------------------------------------ |
| **Interface**               | `WAN`              | Áp dụng cho lưu lượng đi vào từ mạng ngoài |
| **Address Family**          | `IPv4`             | Chỉ áp dụng cho IPv4                       |
| **Protocol**                | `TCP/UDP`          | Cho phép cả TCP và UDP                     |
| **Source**                  | `Any`              | Chấp nhận từ mọi IP nguồn                  |
| **Destination**             | `192.168.75.242`   | Địa chỉ WAN/VIP public nhận request        |
| **Destination Port**        | `80 (HTTP)`        | Cổng dịch vụ web                           |
| **Redirect Target IP**      | `192.168.30.40`    | IP của máy WAF nội bộ                      |
| **Redirect Target Port**    | `80 (HTTP)`        | Chuyển tiếp sang port HTTP của WAF         |
| **Description**             | `NAT to WAF (WAN)` | Mô tả rule                                 |
| **Filter Rule Association** | `Rule NAT to WAF`  | Tự tạo firewall rule đi kèm                |

- Lưu ý: Trong cấu hình NAT Port Forward, địa chỉ `192.168.75.242` được sử dụng thay vì địa chỉ WAN vật lý như `192.168.75.131` vì đây là mô hình HA Firewall gồm 1 Master và 1 Backup. Địa chỉ `192.168.75.242 `là **CARP VIP** (Virtual IP), đóng vai trò địa chỉ đại diện cho cả cụm firewall. Ở trạng thái bình thường, VIP sẽ được gán cho Master Firewall để xử lý lưu lượng. Khi Master gặp sự cố hoặc ngừng hoạt động, Backup Firewall sẽ tự động tiếp quản VIP và tiếp tục xử lý kết nối mà không làm gián đoạn dịch vụ. Vì vậy, các rule NAT và truy cập từ bên ngoài luôn phải trỏ tới **CARP VIP** thay vì IP vật lý của từng firewall nhằm đảm bảo khả năng failover và tính sẵn sàng cao của hệ thống. (Xem mục **High Availability (HA)** sẽ hiểu CARP VIP là gì)

# High Availability (HA)

- Thiết lập **pfSense HA** với CARP VIP để đảm bảo failover cho firewall
- Backup và failover cho các máy chủ quan trọng
- Kiểm tra tính khả dụng của dịch vụ trong trường hợp lỗi

# Triển khai hệ thống theo từng thành phần

## Triển khai Web Server (DMZ-WEB)

- Thiết lập IP tĩnh cho **Ubuntu Web Server (192.168.20.30)**
- Cài đặt **Docker / Docker Compose**
- Khởi chạy container **DVWA** để mô phỏng ứng dụng web dễ bị tấn công
- Kiểm tra dịch vụ HTTP/HTTPS hoạt động nội bộ

## Triển khai WAF (DMZ)

- Cấu hình **WAF (192.168.30.40)** tại vùng DMZ
- Thiết lập rule lọc **SQL Injection, XSS, LFI/RFI**
- Chuyển tiếp lưu lượng từ WAN vào Web Server thông qua WAF

## Triển khai IDS Monitoring

- Cấu hình **IDS Server (192.168.40.50)**
- Thu thập log và phát hiện tấn công mạng
- Giám sát traffic từ DMZ và Security Zone

## Tích hợp ELK Logging

- Gửi log từ **pfSense, WAF, Web Server, IDS** về **ELK Server (192.168.40.60)**
- Kiểm tra dashboard, alert và timeline sự kiện

## Kiểm tra truy cập và giám sát

- Kiểm tra truy cập từ **LAN / DMZ / WAN**
- Mô phỏng tấn công DVWA và xác minh log xuất hiện trên ELK
- Đối chiếu cảnh báo giữa IPS, IDS và WAF

# Triển khai SOC

- Cài đặt công cụ SOC (ELK, Wazuh, Graylog...)
- Thu thập logs từ pfSense, WAF, Web Server, Database, IDS
- Cấu hình dashboards, cảnh báo theo các rule bảo mật
- Giám sát tình trạng mạng và sự cố theo thời gian thực

# Cài đặt IPS / IDS

## Cấu hình IPS để phát hiện và chặn các xâm nhập

- Đầu tiên vào **System** -> **Package Manager** để tải và cài đạt **Suricata**

![IPS](../images/IPS/IPS.png)

![IPS](../images/IPS/IPS_1.png)

- Sau khi tải và cài đạt xong **Suricata** thì vào **Services** -> **Suricata** và chọn **ADD** và cấu hình giống các thông số dưới đây

![IPS](../images/IPS/IPS_2.png)

| Nhóm cấu hình        | Trường                     | Giá trị                 | Ý nghĩa                                   |
| -------------------- | -------------------------- | ----------------------- | ----------------------------------------- |
| **General Settings** | Enable                     | `Checked`               | Kích hoạt Suricata trên interface         |
|                      | Interface                  | `WAN (em0)`             | Giám sát lưu lượng đi vào từ Internet     |
|                      | Description                | `WAN`                   | Mô tả cho instance                        |
| **Logging Settings** | Send Alerts to System Log  | `Checked`               | Gửi cảnh báo vào system log của pfSense   |
|                      | Enable Stats Collection    | `Unchecked`             | Không thu thập thống kê hiệu năng định kỳ |
|                      | Enable HTTP Log            | `Checked`               | Ghi log HTTP traffic đã giải mã           |
|                      | HTTP Log File Type         | `Regular`               | Ghi log vào file thông thường             |
|                      | Append HTTP Log            | `Checked`               | Ghi nối tiếp log khi restart              |
|                      | Log Extended HTTP Info     | `Checked`               | Ghi chi tiết HTTP header / method / URI   |
|                      | Enable TLS Log             | `Unchecked`             | Không ghi log TLS handshake               |
|                      | Enable File-Store          | `Unchecked`             | Không trích xuất file từ traffic          |
|                      | Enable Packet Log          | `Unchecked`             | Không lưu pcap packet log                 |
|                      | Enable Verbose Logging     | `Unchecked`             | Không ghi log chi tiết khi khởi động      |
| **EVE Output**       | EVE JSON Log               | `Unchecked`             | Không xuất log JSON                       |
| **Alert & Block**    | Block Offenders            | `Checked/As configured` | Tự động block IP sinh cảnh báo            |
| **Performance**      | Run Mode                   | `AutoFP`                | Chạy đa luồng, tối ưu cho IDS             |
|                      | AutoFP Scheduler           | `Hash`                  | Phân luồng theo hash flow                 |
|                      | Max Pending Packets        | `1024`                  | Số packet chờ xử lý tối đa                |
|                      | Detect Engine Profile      | `Medium`                | Cân bằng giữa hiệu năng và RAM            |
|                      | MPM Algorithm              | `Auto`                  | Tự chọn thuật toán match tốt nhất         |
|                      | SPM Algorithm              | `Auto`                  | Tự chọn single-pattern matcher            |
|                      | Inspection Recursion Limit | `3000`                  | Giới hạn đệ quy khi inspect               |
|                      | Promiscuous Mode           | `Checked`               | Bắt toàn bộ traffic đi qua interface      |
|                      | PCAP Snaplen               | `1518`                  | Kích thước packet capture tối đa          |
| **Networks**         | Home Net                   | `default`               | Các mạng nội bộ / VIP cần bảo vệ          |
|                      | External Net               | `default`               | Mạng ngoài Internet                       |
| **Filtering**        | Suppression                | `default`               | Không suppression custom                  |

- Cài Snort hoặc Suricata
- Thiết lập rule cơ bản để phát hiện các xâm nhập mạng, malware, exploit
- Kết hợp với SOC để gửi alert và ghi log tập trung


# Kiểm tra & đánh giá

- Ping test từ LAN/DMZ → WAN
- Truy cập các dịch vụ web từ WAN
- Kiểm tra alert SOC / IPS / IDS / WAF
- Đánh giá failover HA và tính sẵn sàng dịch vụ

# Đang chuẩn bị

> Thông tin và nội dung đang được cập nhật, sẽ bổ sung trong thời gian sớm nhất
