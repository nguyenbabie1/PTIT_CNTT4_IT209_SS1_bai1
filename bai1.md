# Bài 1: Khởi tạo Droplet trên DigitalOcean

- Họ tên: Đoàn Trung Nguyên
- Mã sinh viên: B24DTCN435
- Lớp: HN-K24-CNTT4
- Đường dẫn nộp: `homework/session_02/ex1/README.md`

> Đây là hướng dẫn và mẫu báo cáo. Chưa có IP, ảnh hoặc log thật chứng minh hoàn thành. Sau khi thực hành, điền phần kết quả và đính kèm minh chứng. Bài yêu cầu DigitalOcean; nhà cung cấp khác hoặc máy ảo tại máy cá nhân chỉ thay thế khi giảng viên chấp nhận.

## 1. Chuẩn bị

Cần máy Windows có Internet, PowerShell, tài khoản DigitalOcean và repository GitHub. Không cần tạo project Java, IntelliJ hoặc Docker. DigitalOcean là dịch vụ có phí; xem tổng giá trước khi tạo máy chủ.

Nếu đã tạo Droplet, dùng lại máy chủ đó, không tạo thêm một máy chỉ để làm lại hướng dẫn.

## 2. Mở PowerShell và kiểm tra SSH

Mở Start, tìm PowerShell rồi mở. Nếu đang ở dòng nhập passphrase, nhấn Ctrl + C để trở về dấu nhắc PowerShell.

```powershell
ssh -V
```

Nếu hiện `OpenSSH_for_Windows...`, máy đã có SSH. Không thêm dấu `\` ở cuối lệnh. Nếu lệnh không được nhận diện, vào Windows Settings → Optional features, tìm và cài OpenSSH Client rồi mở lại PowerShell.

## 3. Kiểm tra khóa đã tạo trước đó

```powershell
Test-Path "$env:USERPROFILE\.ssh\digitalocean_ex1"
Test-Path "$env:USERPROFILE\.ssh\digitalocean_ex1.pub"
```

Nếu cả hai trả về `True`, dùng lại cặp khóa và sang bước 5. Không ghi đè khóa đã gán cho Droplet. Nếu chưa có cả hai file, làm bước 4. Nếu chỉ có một file, kiểm tra lại trước khi tạo hoặc ghi đè.

## 4. Tạo cặp khóa SSH mới

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.ssh"
ssh-keygen -t ed25519 -C "digitalocean-ex1" -f "$env:USERPROFILE\.ssh\digitalocean_ex1"
```

Nếu hỏi `Overwrite (y/n)?`, nhập `n` để giữ khóa hiện có.

Khi hỏi `Enter passphrase (empty for no passphrase):`, có hai lựa chọn:

- Đặt passphrase để bảo vệ Private Key: nhập cùng một mật khẩu ở hai lần. Khi gõ, màn hình không hiện ký tự. SSH có thể hỏi passphrase khi kết nối; đây không phải mật khẩu root của máy chủ.
- Để trống cho bài thực hành muốn kết nối không phải nhập passphrase: nhấn Enter ở cả hai lần, không gõ dấu cách. Private Key không được bảo vệ bằng passphrase nên cần giữ file cẩn thận.

Nếu báo `Passphrases do not match`, hai lần nhập khác nhau; thực hiện lại hai lần nhập.

Thành công sẽ có thông báo đã lưu identification và public key.

| File | Vai trò |
|---|---|
| `C:\Users\HCLaptop.com\.ssh\digitalocean_ex1` | Private Key: giữ riêng trên máy, không đưa lên GitHub |
| `C:\Users\HCLaptop.com\.ssh\digitalocean_ex1.pub` | Public Key: thêm vào DigitalOcean |

## 5. Copy Public Key

Xem khóa:

```powershell
Get-Content "$env:USERPROFILE\.ssh\digitalocean_ex1.pub"
```

Nội dung thật phải bắt đầu bằng `ssh-ed25519`, tiếp theo là chuỗi mã hóa và nhãn của khóa trên một dòng.

Copy toàn bộ Public Key:

```powershell
Get-Content "$env:USERPROFILE\.ssh\digitalocean_ex1.pub" | Set-Clipboard
```

Lệnh copy thường không in kết quả. Không copy câu lệnh, đường dẫn, dấu nhắc `PS ...>` hoặc nội dung Private Key vào ô SSH Key content.

## 6. Đăng ký DigitalOcean

1. Mở https://cloud.digitalocean.com/.
2. Đăng ký hoặc đăng nhập.
3. Xác minh email và hoàn thành yêu cầu thanh toán của tài khoản.
4. Vào Create → Droplets.

## 7. Chọn cấu hình

Các mức giá sau là cấu hình xuất hiện trong Console đã cung cấp trong bài thực hành; đối chiếu lại màn hình trước khi tạo.

| Mục | Chọn |
|---|---|
| Region | Singapore, SGP1 |
| Image | Ubuntu 22.04 LTS x64 hoặc Ubuntu LTS mới hơn nếu không có |
| Plan | Basic, Shared CPU |
| CPU Options | Regular, SSD |
| Size | Gói rẻ nhất khả dụng; màn hình đã cung cấp có $4/tháng, 1 vCPU, 512 MB RAM, 10 GB SSD |
| Volumes | Không thêm |
| Backups | Không bật nếu không cần |
| Authentication | SSH Keys |
| Public IPv4 | Bật để kết nối từ Windows qua Internet |
| Public IPv6 | Không bắt buộc |
| Monitoring | Tùy chọn miễn phí |
| Startup scripts | Để trống |
| Managed Database | Không thêm |
| Quantity | 1 |
| Name | nguyen-ubuntu-ex1 |
| Project | first-project hoặc project đang dùng |
| Tags | Để trống |

512 MB RAM phù hợp bài tạo máy và kiểm tra SSH; không phải cấu hình để chạy cả hệ thống Spring Boot microservices.

Trong Authentication:

1. Chọn SSH Keys → Add an SSH Key.
2. Dán Public Key bằng Ctrl + V vào SSH Key content.
3. Đặt tên khóa `Nguyen-Windows`.
4. Lưu và tích chọn khóa vừa thêm.
5. Kiểm tra cấu hình, tổng giá rồi nhấn Create Droplet.

## 8. Lấy Public IPv4

Chờ máy khởi tạo xong. Vào Droplets, tìm `nguyen-ubuntu-ex1`. IP có thể hiện ngay trong danh sách; nếu chưa thấy, nhấn tên máy và tìm Public IPv4.

Copy bằng nút cạnh IP hoặc bôi đen rồi Ctrl + C. Không dùng Private IPv4 cho kết nối này. Nếu danh sách trống, kiểm tra đúng project và việc tạo Droplet đã hoàn tất.

## 9. Kết nối từ Windows

Để tránh quên thay chữ IP_DROPLET, chạy hai lệnh sau. Lệnh đầu sẽ hỏi IP; dán địa chỉ Public IPv4 thật rồi Enter.

```powershell
$dropletIp = Read-Host "Dan Public IPv4 cua Droplet"
ssh -i "$env:USERPROFILE\.ssh\digitalocean_ex1" "root@$dropletIp"
```

Bạn cũng có thể dùng dạng lệnh đề bài, nhưng phải thay `IP_DROPLET` bằng IP thật:

```powershell
ssh -i "$env:USERPROFILE\.ssh\digitalocean_ex1" root@IP_DROPLET
```

Không nhập nguyên chữ IP_DROPLET, không thêm dấu < > và không có dấu cách sau root@.

### Xác nhận máy chủ lần đầu

SSH có thể hiện fingerprint và hỏi có tiếp tục không. Trong trang Droplet, mở Access / Console, truy cập máy chủ và chạy:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Nếu SSH trên Windows hiển thị loại ED25519, đối chiếu fingerprint SHA256 với kết quả trên Console. Chỉ khi khớp mới nhập `yes`. Nếu loại khóa khác, dùng file host key tương ứng; không bỏ qua khi fingerprint không khớp.

Console dùng để kiểm tra; minh chứng cuối cùng vẫn phải là kết nối SSH từ máy cá nhân theo yêu cầu bài.

## 10. Kiểm tra kết nối thành công

Sau khi thấy lời chào Ubuntu và dấu nhắc `root@...:~#`, chạy trên máy chủ:

```bash
whoami
cat /etc/os-release
hostname
```

Kiểm tra `whoami` trả về `root`, hệ điều hành đúng Ubuntu và hostname đúng máy đã tạo. Chụp ảnh Terminal gồm lệnh SSH từ Windows và kết quả kiểm tra. Không cần cài thêm ứng dụng để hoàn thành bài 1.

Để thoát về PowerShell:

```bash
exit
```

## 11. Xử lý lỗi thường gặp

| Lỗi | Cách xử lý |
|---|---|
| Passphrases do not match | Nhập hai lần giống nhau hoặc để trống cả hai lần |
| SSH key content must be a valid SSH key | Copy lại toàn bộ nội dung file .pub; kiểm tra bắt đầu bằng ssh-ed25519 |
| Could not resolve hostname ip_droplet | Đang dùng chữ placeholder; thay bằng Public IPv4 thật hoặc dùng Read-Host ở bước 9 |
| Identity file ... not accessible | Kiểm tra đường dẫn và file Private Key bằng Test-Path |
| Permission denied (publickey) | Dùng root, đúng Private Key và Public Key đã được gán cho Droplet lúc tạo; chỉ thêm khóa vào tài khoản sau khi tạo chưa chắc đã thêm vào máy chủ |
| Connection timed out | Kiểm tra IP, trạng thái Droplet và firewall cho phép TCP 22 từ IP máy cá nhân |
| Connection refused | Chờ máy khởi động xong; kiểm tra dịch vụ SSH trong Console |

Kiểm tra cổng SSH từ Windows khi cần:

```powershell
Test-NetConnection -ComputerName $dropletIp -Port 22
```

Nếu lỗi chưa hết, lưu nguyên đoạn lỗi để kiểm tra; không xóa hoặc tạo lại Droplet ngay.

## 12. Chuẩn bị minh chứng và nộp GitHub

Tải file README này về. Trong repository, đặt tại `homework/session_02/ex1/README.md`. Tạo thêm thư mục `images` trong `ex1` nếu nộp ảnh.

- Ảnh Console: lưu thành `images/droplet-console.png`.
- Ảnh SSH thành công: lưu thành `images/ssh-success.png`.
- Có thể dùng log Terminal thật thay ảnh theo đề bài. Không dùng log minh họa làm minh chứng thực tế.

Trên GitHub:

1. Mở repository của bạn → Add file → Create new file.
2. Nhập tên `homework/session_02/ex1/README.md`.
3. Copy nội dung file này, điền phần 13 bằng thông tin thực tế và Commit changes.
4. Vào thư mục ex1 → Add file → Upload files, tải thư mục images chứa ảnh rồi Commit changes.
5. Mở lại README, kiểm tra ảnh và nội dung hiển thị đúng.

Không đưa Private Key lên GitHub. File này vẫn là mẫu nếu chưa có minh chứng thực tế.

## 13. Báo cáo kết quả thực tế — sinh viên điền sau khi thực hành

| Nội dung | Kết quả thực tế |
|---|---|
| Ngày thực hành | [Điền] |
| Tên Droplet | [Điền] |
| Region | [Điền] |
| Ubuntu | [Điền] |
| CPU / RAM / SSD | [Điền] |
| Giá hiển thị | [Điền] |
| Authentication | [Điền] |
| Kết nối SSH từ Windows | [Thành công / Chưa thành công] |
| Kết quả whoami | [Điền] |

### Ảnh minh chứng

Chỉ giữ các liên kết dưới đây khi đã có ảnh đúng tên trong thư mục images; nếu dùng log, thay phần này bằng log thật.

![Giao diện quản lý Droplet](images/droplet-console.png)

![Kết nối SSH thành công từ Windows](images/ssh-success.png)

### Log kết nối thật (nếu sử dụng)

```text
[Dán log thật từ Terminal tại đây; xóa dòng này sau khi điền]
```

### Kiểm tra trước khi nộp

- [ ] Đã tạo Droplet Ubuntu trên DigitalOcean tại Singapore.
- [ ] Đã chọn Basic, Shared CPU, Regular SSD và cấu hình tối giản.
- [ ] Đã gán Public Key cho Droplet.
- [ ] Đã SSH từ Windows bằng tài khoản root và đúng Private Key.
- [ ] Đã có ảnh hoặc log thật chứng minh kết nối thành công.
- [ ] Đã điền kết quả thực tế và xóa các placeholder không còn dùng.
- [ ] README nằm đúng homework/session_02/ex1/README.md.
- [ ] Không có Private Key trong repository.

## 14. Sau khi hoàn thành

Giữ máy chủ nếu còn bài tiếp theo cần dùng. Nếu không cần nữa, lưu minh chứng và dữ liệu cần giữ trước khi chủ động Destroy Droplet. Tắt máy (Power Off) vẫn có thể bị tính phí vì tài nguyên vẫn được giữ; Destroy xóa máy chủ và dữ liệu trên máy. Kiểm tra riêng các tài nguyên trả phí khác nếu đã tạo.

## 15. Tài liệu tham khảo

- Tạo Droplet: https://docs.digitalocean.com/products/droplets/how-to/create/
- SSH Keys: https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/
- Kết nối OpenSSH: https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/openssh/
- Chi phí: https://docs.digitalocean.com/products/droplets/details/pricing/

Hướng dẫn soạn ngày 01/10/2026. Giao diện và giá thực tế có thể thay đổi; dùng cấu hình và giá trên Console của bạn.
