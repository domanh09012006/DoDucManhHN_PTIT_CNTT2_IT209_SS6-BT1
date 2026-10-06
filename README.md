# Bài 1: Khảo sát FHS và Phân quyền File/Folder nâng cao

## 1. Thông tin bài thực hành
* **Môn học:** IT209 - Quản trị Hệ thống / Linux
* **Session:** Session 06
* **Bài tập:** Bài 1 - Khảo sát Filesystem Hierarchy Standard (FHS) và Phân quyền File/Folder nâng cao
* **Tài khoản thực hiện (Non-root User):** `student`
* **Nhóm dịch vụ (Group):** `www-data`

---

## 2. Mục tiêu
* Làm quen và áp dụng chuẩn phân cấp thư mục Linux (Filesystem Hierarchy Standard - FHS) vào mô hình ứng dụng web thực tế tại `/var/www`.
* Thực hành phân quyền thư mục bằng ký hiệu bát phân (Octal notation): `chmod 750` và `chmod 770`.
* Thực hành thay đổi quyền sở hữu người dùng và nhóm người dùng (`chown`).
* Nắm vững nguyên tắc phân quyền tối thiểu (Principle of Least Privilege), bảo vệ an toàn cho ứng dụng web (tách biệt quyền giữa `public` và `logs`, cô lập người dùng ngoài hệ thống `others`).

---

## 3. Cấu trúc thư mục yêu cầu

Cấu trúc thư mục được triển khai theo chuẩn FHS tại đường dẫn `/var/www/my-app/`:

```text
/var/www/my-app/
├── public/
└── logs/
```

### Ý nghĩa theo chuẩn FHS:
* `/var`: Chứa các dữ liệu biến đổi (variable data) trong quá trình hệ thống và ứng dụng vận hành.
* `/var/www`: Vị trí tiêu chuẩn để lưu trữ mã nguồn, tài nguyên của các ứng dụng web và web server (như Nginx, Apache).
* `public/`: Thư mục chứa các tài nguyên tĩnh hoặc file công khai (HTML, CSS, JS, hình ảnh) được web server phục vụ ra ngoài.
* `logs/`: Thư mục lưu trữ nhật ký hoạt động, lỗi hệ thống (application logs) do ứng dụng và web server sinh ra trong lúc chạy.

---

## 4. Các lệnh đã thực hiện

Toàn bộ thao tác được thực hiện bằng tài khoản non-root `student` kết hợp `sudo` để có quyền quản trị:

### Bước 1: Tạo cấu trúc thư mục
Sử dụng cờ `-p` (`--parents`) để tạo tự động các thư mục cha nếu chưa tồn tại:
```bash
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs
```

### Bước 2: Thiết lập quyền truy cập bằng Octal (chmod)
* Phân quyền cho thư mục `public` đạt đúng `750` (`drwxr-x---`):
```bash
sudo chmod 750 /var/www/my-app/public
```

* Phân quyền cho thư mục `logs` đạt đúng `770` (`drwxrwx---`):
```bash
sudo chmod 770 /var/www/my-app/logs
```

### Bước 3: Phân quyền sở hữu Owner và Group (chown)
Thay đổi chủ sở hữu sang user non-root `student` và group sang `www-data` cho toàn bộ thư mục `/var/www/my-app`:
```bash
sudo chown -R student:www-data /var/www/my-app
```

### Bước 4: Kiểm tra kết quả
```bash
ls -la /var/www/my-app
```

---

## 5. Giải thích chi tiết quyền Octal `750` và `770`

Hệ thống phân quyền Linux sử dụng 3 bộ quyền cơ bản:
* **r (Read):** Giá trị `4`
* **w (Write):** Giá trị `2`
* **x (Execute):** Giá trị `1` (đối với thư mục, `x` cho phép truy cập, duyệt thư mục)
* Không có quyền: Giá trị `0`

Một chuỗi phân quyền 3 chữ số tương ứng với: **Owner** (Người sở hữu) - **Group** (Nhóm) - **Others** (Người dùng khác).

### 5.1. Phân quyền `750` (`drwxr-x---`) cho `public/`
| Thành phần | Ký hiệu | Giá trị tính toán | Quyền thực tế |
| :--- | :---: | :---: | :--- |
| **Owner** | `rwx` | $4 + 2 + 1 = 7$ | Đọc, ghi và truy cập thư mục |
| **Group** | `r-x` | $4 + 0 + 1 = 5$ | Đọc và truy cập thư mục (không có quyền ghi/xóa) |
| **Others** | `---` | $0 + 0 + 0 = 0$ | Hoàn toàn không có quyền truy cập, đọc hoặc ghi |

* **Ý nghĩa thực tế:** Tài khoản ứng dụng (`student`) có toàn quyền chỉnh sửa tài nguyên tĩnh; Web server thuộc group `www-data` có quyền đọc và duyệt file để phục vụ client, nhưng không thể ghi đè/sửa đổi file (tăng cường tính bảo mật); Các tài khoản khác trên hệ điều hành (`others`) bị chặn hoàn toàn.

### 5.2. Phân quyền `770` (`drwxrwx---`) cho `logs/`
| Thành phần | Ký hiệu | Giá trị tính toán | Quyền thực tế |
| :--- | :---: | :---: | :--- |
| **Owner** | `rwx` | $4 + 2 + 1 = 7$ | Toàn quyền (đọc, ghi file log, truy cập thư mục) |
| **Group** | `rwx` | $4 + 2 + 1 = 7$ | Toàn quyền (đọc, ghi file log, truy cập thư mục) |
| **Others** | `---` | $0 + 0 + 0 = 0$ | Hoàn toàn không có quyền |

* **Ý nghĩa thực tế:** Cả application service (chạy dưới user `student`) và web server daemon (chạy dưới group `www-data`) đều cần quyền ghi log liên tục vào thư mục này. Người dùng khác (`others`) không có quyền để tránh lộ các thông tin nhạy cảm (sensitive info, stack traces, credentials) được ghi lại trong log.

---

## 6. Lệnh `chown` và ý nghĩa

Lệnh `chown` (Change Owner) dùng để thay đổi quyền sở hữu file/thư mục:
```bash
sudo chown -R student:www-data /var/www/my-app
```

* **`sudo`:** Chạy với đặc quyền root vì thao tác đổi chủ sở hữu trong `/var/www` yêu cầu quyền quản trị viên.
* **`-R` (`--recursive`):** Áp dụng đệ quy cho cả thư mục `my-app` và tất cả các thư mục/file con bên trong (`public/`, `logs/`).
* **`student:www-data`:**
  * **User Owner:** `student` (tài khoản non-root của nhà phát triển/ứng dụng).
  * **Group Owner:** `www-data` (nhóm tài khoản mặc định của các web server như Apache/Nginx trên Linux).
* **Tại sao không dùng `root` làm Owner:**
  * Nếu cấp quyền sở hữu trực tiếp cho `root`, khi tiến trình gặp lỗi bảo mật (như lỗ hổng tải file hoặc Remote Code Execution), kẻ tấn công có thể leo thang đặc quyền kiểm soát toàn bộ hệ điều hành.
  * Việc sử dụng tài khoản non-root tuân thủ nguyên tắc an ninh tối thiểu (Principle of Least Privilege).

---

## 7. Output thực tế

Kết quả thực tế khi thực thi lệnh kiểm tra:

```bash
ls -la /var/www/my-app
```

```text
total 4
drwxr-xr-x    4 student  www-data      1024 Oct  6 16:47 .
drwxr-xr-x    3 root     root          1024 Oct  6 16:47 ..
drwxrwx---    2 student  www-data      1024 Oct  6 16:47 logs
drwxr-x---    2 student  www-data      1024 Oct  6 16:47 public
```

Kiểm tra chi tiết từng thư mục bằng `ls -ld`:
```bash
ls -ld /var/www/my-app /var/www/my-app/public /var/www/my-app/logs
```

```text
drwxr-xr-x    4 student  www-data      1024 Oct  6 16:47 /var/www/my-app
drwxrwx---    2 student  www-data      1024 Oct  6 16:47 /var/www/my-app/logs
drwxr-x---    2 student  www-data      1024 Oct  6 16:47 /var/www/my-app/public
```

---

## 8. Kết luận kiểm tra

Bảng đối chiếu giữa yêu cầu đề bài và kết quả cấu hình thực tế:

| Mục kiểm tra | Yêu cầu đề bài | Kết quả thực tế | Trạng thái |
| :--- | :--- | :--- | :---: |
| **Cấu trúc thư mục** | `/var/www/my-app/{public, logs}` | Đã tạo đầy đủ `public/` và `logs/` | **ĐẠT** |
| **Quyền thư mục public** | `750` (`drwxr-x---`) | `drwxr-x---` | **ĐẠT** |
| **Quyền thư mục logs** | `770` (`drwxrwx---`) | `drwxrwx---` | **ĐẠT** |
| **Owner** | Non-root user (`student`) | `student` | **ĐẠT** |
| **Group** | `www-data` | `www-data` | **ĐẠT** |
| **Ngăn chặn Others** | Others không có quyền (`---`) | `---` ở cả 2 thư mục | **ĐẠT** |

**Kết luận chung:** Cấu trúc thư mục, chủ sở hữu, nhóm người dùng và phân quyền thư mục hoàn toàn chính xác theo yêu cầu đề bài.
