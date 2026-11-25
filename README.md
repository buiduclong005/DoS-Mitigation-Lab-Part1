ANM - Kiểm thử HTTP chậm trên Nginx trước khi áp dụng biện pháp phòng chống DoS
# DoS Mitigation Lab - Part 1

## 1. Giới thiệu đề tài
Đồ án môn An Ninh Mạng: Nghiên cứu cơ chế DoS Slowloris và biện pháp phòng chống trên máy chủ Nginx.

## 2. Mục tiêu phần 1
- Xây dựng môi trường thực nghiệm trên VMware.
- Cài đặt và cấu hình Ubuntu Server.
- Triển khai Nginx Web Server.
- Cài đặt Netdata để giám sát hệ thống.
- Quan sát kết nối HTTP chưa hoàn chỉnh.
- Thu thập minh chứng trước khi áp dụng biện pháp phòng chống.

## 3. Môi trường thực nghiệm

| Thành phần | Thông tin |
|---|---|
| Phần mềm ảo hóa | VMware Workstation |
| Hệ điều hành | Ubuntu Server 24.04.5 LTS |
| Tên máy chủ | dos-lab-server |
| RAM | 6 GB |
| CPU | 2 vCPU |
| Ổ cứng | 20 GB |
| Web Server | Nginx 1.24.0 |
| Công cụ giám sát | Netdata 2.13.0 |
| Ngôn ngữ kiểm thử | Python 3.12.3 |

## 4. Cấu hình mạng

| Card mạng | Chế độ | Địa chỉ IP |
|---|---|---|
| ens33 | NAT | 192.168.15.128/24 |
| ens37 | Host-only | 192.168.124.128/24 |
| VMnet1 | Windows Host | 192.168.124.1/24 |

Card NAT được ngắt trong quá trình kiểm thử nhằm cô lập môi trường thực nghiệm.

## 5. Nội dung thực hiện
- Cài đặt hệ điều hành và các công cụ cơ bản.
- Cài đặt OpenSSH Server.
- Cài đặt và kiểm tra Nginx.
- Cấu hình hai card mạng.
- Cài đặt Netdata.
- Thu thập thông số hệ thống ở trạng thái bình thường.
- Thực hiện kiểm thử HTTP giới hạn trong môi trường nội bộ.
- Ghi nhận kết quả và kiểm tra khả năng phản hồi của máy chủ.

## 6. Kết quả thực nghiệm

| Nội dung | Kết quả |
|---|---|
| Nginx | Hoạt động bình thường |
| SSH | Active (running) |
| Netdata | Active (running) |
| CPU trước kiểm thử | Khoảng 2.27% |
| RAM trước kiểm thử | Khoảng 8.29% |
| TCP trước kiểm thử | 6 kết nối tại thời điểm chụp Dashboard |
| Kết nối HTTP kiểm thử | 8 kết nối |
| Thời gian kiểm thử | 20 giây |
| Phản hồi HTTP sau kiểm thử | 200 OK |

Kết quả cho thấy số kết nối TCP tăng trong quá trình kiểm thử và giảm sau khi chương trình kết thúc. Máy chủ vẫn phản hồi HTTP bình thường sau kiểm thử.

File tcp-before.csv hiện chứa dữ liệu đo khi chương trình kiểm thử không hoạt động. Kết quả trong quá trình kiểm thử được minh chứng bằng ảnh terminal và biểu đồ Netdata.

## 7. Cấu trúc repository

- config/: Các file cấu hình.
- evidence/: Dữ liệu và kết quả kiểm tra.
- screenshots/: Hình ảnh minh chứng thực nghiệm.
- README.md: Tài liệu mô tả đồ án.

## 8. Phạm vi và an toàn
Bài thực nghiệm chỉ được thực hiện trong môi trường lab cô lập, với số lượng kết nối giới hạn, nhằm minh họa cơ chế HTTP chậm, không nhằm làm gián đoạn dịch vụ thực tế.

## 9. Tài liệu tham khảo
- https://github.com/MervJB7/DoS-Mitigation-Lab
- https://nginx.org/en/docs/
- https://learn.netdata.cloud/
- https://documentation.ubuntu.com/server/

## 10. Hướng phát triển phần 2
- Áp dụng cấu hình phòng chống DoS trên Nginx.
- Kiểm tra khả năng phản hồi của máy chủ.
- Đồng bộ điều kiện kiểm thử.
- So sánh kết quả trước và sau khi áp dụng biện pháp phòng chống.
