# Đăng trang tải KidSafe và chạy thử trên hai máy

GitHub Pages phát hành trang tải và bộ cài Windows. Máy chính chạy máy chủ KidSafe cùng bảng điều khiển phụ huynh. Máy của con chạy bộ cài và kết nối với máy chính qua cùng mạng Wi-Fi/LAN.

Bộ cài đã kèm Python và thư viện. Máy con cần quyền quản trị khi cài, một tài khoản Windows thường cho con, tệp kết nối và mã ghép đôi lấy từ bảng điều khiển. Tệp kết nối là cấu hình của gia đình. Sau khi ghép, con dùng ứng dụng Windows bằng tài khoản Windows thường.

## A. Đưa trang tải lên GitHub Pages

1. Giải nén `KidSafe-github-pages.zip` vào một thư mục riêng. Mở thư mục đó và kiểm tra thấy `index.html`, `KidSafe-Setup-x64.exe`, `KidSafe-Setup-x64.exe.sha256`, `.nojekyll`, `README.md` và thư mục `static`.
2. Đăng nhập GitHub, bấm **New repository**, đặt tên `kidsafe-download`, chọn **Public** nếu dùng GitHub Free, rồi bấm **Create repository**. Đây là repo riêng cho bộ tải công khai.
3. Chọn **uploading an existing file** ở repo trống, hoặc **Add file → Upload files**. Trong Chrome hoặc Edge, kéo toàn bộ **nội dung đã giải nén** vào vùng tải lên, gồm cả thư mục `static`. Giữ `index.html` ở gốc repo. Chọn **Commit changes**. Nếu thiếu `.nojekyll`, dùng **Add file → Create new file**, đặt đúng tên `.nojekyll` và lưu.
4. Vào **Settings → Pages**. Ở **Source**, chọn **Deploy from a branch**. Chọn nhánh **main** và **/(root)** rồi bấm **Save**.
5. Khi mục **Actions** báo triển khai thành công, mở lại **Settings → Pages → Visit site**. Link có dạng `https://TEN-GITHUB.github.io/kidsafe-download/`. Dùng nút **Tải cho Windows** để kiểm tra nhận được `KidSafe-Setup-x64.exe`. Gửi link trang này cho người dùng thử.

Chỉ tải lên nội dung của gói phát hành. Giữ riêng thư mục dự án, `.runtime`, database, khóa chứng chỉ, tệp kết nối và mã ghép đôi. Bộ cài hiện chưa có chữ ký Authenticode nên Windows có thể hiện cảnh báo nhà phát hành. Có thể đối chiếu SHA-256 bằng `Get-FileHash -Algorithm SHA256 -LiteralPath '.\KidSafe-Setup-x64.exe'` với tệp `.sha256` đi kèm.

[Hướng dẫn GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) · [Tải tệp qua trình duyệt](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository)

## B. Bật máy chủ trên máy chính

1. Kết nối hai máy vào cùng Wi-Fi tin cậy. Trên máy chính mở PowerShell và chạy `ipconfig`. Tìm **IPv4 Address** ở mục **Wireless LAN adapter Wi-Fi** hoặc **Ethernet adapter** đang dùng. Ví dụ bên dưới là `192.168.1.10`. Hãy thay bằng địa chỉ thật của máy chính.
2. Trong Windows Settings, đặt mạng đang dùng là **Private**. Mở PowerShell bình thường, dán khối lệnh sau và giữ cửa sổ này mở.

```powershell
Set-Location 'D:\DoAN MMT'
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
$serverIp = '192.168.1.10'
.\Start-KidSafe.ps1 -RuntimeDirectory '.runtime\lan' -CertificateHosts $serverIp -Bind $serverIp -Port 8444
```

3. Trên máy chính, mở thêm **PowerShell → Run as administrator** để cho máy con vào cổng 8444 từ mạng nội bộ.

```powershell
if (-not (Get-NetFirewallRule -Name 'KidSafe-LAN-8444' -ErrorAction SilentlyContinue)) {
    New-NetFirewallRule -Name 'KidSafe-LAN-8444' -DisplayName 'KidSafe LAN 8444' -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8444 -Profile Private -RemoteAddress LocalSubnet
}
```

4. Trên máy chính, trong PowerShell của tài khoản dùng trình duyệt, tin đúng CA vừa tạo và mở bảng điều khiển. Thay IP ở dòng cuối.

```powershell
Set-Location 'D:\DoAN MMT'
Import-Certificate -FilePath '.runtime\lan\tls\ca.crt' -CertStoreLocation 'Cert:\CurrentUser\Root'
Start-Process 'https://192.168.1.10:8444/login'
```

5. Chọn **Tạo tài khoản phụ huynh**, đăng nhập rồi **Thêm hồ sơ con → Thiết bị & ghép đôi → Tải tệp kết nối**. Giữ tệp `KidSafe-ket-noi.kidsafe` để chuyển riêng cho máy con. Mã ghép đôi tạo ở bước C để còn đủ 10 phút.

Máy chủ dùng hồ sơ LAN riêng, khác hồ sơ localhost cổng 8446. Mỗi lần demo lại, dùng lệnh khởi động ở B2 với cùng IP. Nếu router cấp IP khác, chứng chỉ LAN phải khớp địa chỉ mới trước khi ghép. Để giữ địa chỉ ổn định qua nhiều buổi, có thể dành riêng IP cho máy chính trong router.

## C. Tải, cài và ghép đôi trên máy của con

1. Trên máy con, kiểm tra kết nối bằng PowerShell. Thay IP ví dụ bằng IP máy chính. Kết quả cần có `TcpTestSucceeded : True`.

```powershell
Test-NetConnection 192.168.1.10 -Port 8444
```

2. Trên máy con, tạo một tài khoản Windows **Standard user** cho con nếu chưa có. Đăng nhập một lần để Windows tạo hồ sơ, rồi đăng xuất và quay lại tài khoản quản trị.
3. Mở link GitHub Pages, bấm **Tải cho Windows**. Chuyển tệp `KidSafe-ket-noi.kidsafe` từ máy chính sang máy con, chẳng hạn bằng USB. Nếu hai máy cùng LAN, cũng có thể lấy bộ cài từ trang `https://IP-MAY-CHINH:8444/download` bằng trình duyệt đã tin CA của máy chủ.
4. Chạy `KidSafe-Setup-x64.exe` với quyền quản trị trên máy con. Chọn tệp kết nối, tài khoản Windows thường của con và card mạng Wi-Fi/Ethernet đang sử dụng.
5. Trên web phụ huynh, tạo **mã ghép đôi 8 ký tự** cho đúng hồ sơ, xác nhận đồng ý rồi nhập mã vào bộ cài trước khi hết 10 phút.
6. Cài xong, đăng xuất tài khoản quản trị và đăng nhập Windows bằng tài khoản của con. Mở **KidSafe** từ Start hoặc biểu tượng khay hệ thống. Trên web phụ huynh, kiểm tra thiết bị đã kết nối và thử gửi một yêu cầu thêm giờ từ app.

Nếu kiểm tra cổng ở C1 thất bại, kiểm tra đúng IP, server còn chạy, hai máy cùng mạng Private và mạng Wi-Fi khách có đang cách ly hai máy. Dùng **Installed apps → KidSafe → Uninstall** trong tài khoản quản trị của máy con để gỡ. Sau buổi thử, nhấn **Ctrl+C** ở cửa sổ PowerShell chạy server để dừng máy chủ.

## Khi nào cần máy chủ trên Internet?

Mô hình LAN đủ cho demo hai máy cùng mạng. GitHub Pages giúp tải bộ cài từ bất kỳ đâu. Để phụ huynh và máy con ở hai mạng khác nhau vẫn đồng bộ, cần triển khai FastAPI trên máy chủ chạy liên tục với HTTPS, WebSocket, ổ đĩa bền vững cho SQLite và chứng chỉ phù hợp với cơ chế pin của app. Bản Streamlit trong dự án dùng dữ liệu mẫu để xem trước giao diện. GitHub Pages chỉ phục vụ tệp tĩnh. [Giới hạn GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages)
