# DoS Mitigation Lab

Lab mô phỏng việc bảo vệ một web server (nginx) trước tấn công từ chối dịch vụ dạng **Slowloris**, bằng cách giám sát real-time với Netdata và áp dụng cấu hình giảm thiểu (mitigation) ở tầng nginx.

## Mục tiêu

- Dựng một môi trường server mục tiêu (nginx) trong máy ảo cô lập.
- Giám sát số lượng kết nối TCP theo thời gian thực bằng Netdata.
- Áp dụng cấu hình chống chịu tấn công Slowloris (giới hạn kết nối theo IP, rút ngắn timeout).
- Kiểm thử để xác nhận server vẫn phục vụ người dùng hợp lệ bình thường sau khi áp dụng mitigation.

## Kiến trúc lab

| Thành phần | Mô tả |
|---|---|
| Hypervisor | VMware Workstation Pro |
| Máy ảo mục tiêu | Ubuntu Server 24.04 LTS |
| Web server | nginx 1.28.3 |
| Công cụ giám sát | Netdata |
| Mạng | Host-only (cô lập, không ra Internet) |

## Cài đặt môi trường

### 1. Cài nginx

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx
```

### 2. Cài Netdata

```bash
sudo bash -c "$(curl -SsL https://my-netdata.io/kickstart.sh)"
```

Truy cập dashboard giám sát tại: `http://<IP-server>:19999`

## Cấu hình chống DoS (Slowloris Mitigation)

File cấu hình: [`nginx-dos-mitigation.conf`](./nginx-dos-mitigation.conf), được đặt tại `/etc/nginx/conf.d/dos-mitigation.conf` trên server.

```nginx
limit_conn_zone $binary_remote_addr zone=addr:10m;
client_header_timeout 5s;
client_body_timeout 5s;
send_timeout 5s;
keepalive_timeout 5;
limit_conn addr 10;
```

**Giải thích:**

| Directive | Vai trò |
|---|---|
| `limit_conn_zone` | Khai báo vùng nhớ lưu số kết nối hiện tại theo từng IP |
| `client_header_timeout` | Đóng kết nối nếu client không gửi xong HTTP header trong 5 giây — chặn kiểu Slowloris gửi header rất chậm |
| `client_body_timeout` | Đóng kết nối nếu client không gửi xong phần body trong 5 giây |
| `send_timeout` | Giới hạn thời gian server chờ gửi response |
| `keepalive_timeout` | Rút ngắn thời gian giữ kết nối keep-alive, giảm số kết nối "treo" |
| `limit_conn addr 10` | Mỗi địa chỉ IP chỉ được tối đa 10 kết nối đồng thời — ngăn 1 nguồn chiếm hết connection pool |

### Áp dụng cấu hình

```bash
sudo nginx -t                    # kiểm tra cú pháp
sudo systemctl reload nginx      # áp dụng không gián đoạn dịch vụ
```

## Kiểm thử sau khi áp dụng mitigation

Gửi 15 request tuần tự để xác nhận server vẫn phục vụ người dùng hợp lệ bình thường:

```bash
for i in $(seq 1 15); do curl -s -o /dev/null -w "Request $i : %{http_code}\n" http://localhost; done
```

**Kết quả:** 15/15 request trả về mã `200 OK` — xác nhận cấu hình mitigation không ảnh hưởng tới truy cập hợp lệ thông thường, trong khi về mặt kỹ thuật vẫn giới hạn được số kết nối treo (slow/hanging connections) kiểu Slowloris.

![Kết quả test](./screenshots/test-results.png)
![Netdata monitoring](./screenshots/netdata-dashboard.png)

## Kết luận

Việc kết hợp giới hạn kết nối theo IP (`limit_conn`) và rút ngắn các timeout liên quan đến việc nhận dữ liệu từ client là phương án hiệu quả, nhẹ, không cần công cụ bên thứ ba để giảm thiểu tấn công Slowloris trên nginx. Giám sát bằng Netdata giúp quan sát trực quan số lượng kết nối theo thời gian thực, hỗ trợ phát hiện sớm các dấu hiệu bất thường.

## Thành viên thực hiện

| Thành viên | Nhiệm vụ |
|---|---|
| (Điền tên) | Tạo môi trường demo, cài máy chủ, đưa mã nguồn lên GitHub, viết README |
| (Điền tên) | Thiết lập MFA, tạo tài khoản/nhóm/phân quyền, chạy test case, viết phần kết quả demo |
