# KIEMTRA — Kiểm tra trực tuyến tại lớp

**Địa chỉ chạy thật:** `https://diemdanhsv.com/kiemtra/`
**Chủ nhiệm:** ThS. Đinh Quốc Tuấn — Khoa Địa chất, Trường ĐH Khoa học Tự nhiên, ĐHQG-HCM

Hệ thống thi trắc nghiệm **trực tiếp trên lớp** bằng điện thoại/laptop của sinh viên:
quét QR → 5 phút chuẩn bị → làm bài có giám sát thiết bị → chấm tự động →
phân tích khảo thí và **mức đạt chuẩn đầu ra (CLO)** → chuyển điểm vào bảng điểm diemdanhsv.com.

## Các trang

| Trang | Ai dùng | Việc |
|---|---|---|
| `index.html` | SV | Nhập mã phòng (khi không quét QR) |
| `sv.html?c=MÃPHÒNG` | SV | Vào phòng bằng MSSV, chuẩn bị, làm bài, nộp, xem lại đáp án + giải thích |
| `gv.html` | GV | CLO, ngân hàng câu hỏi (ảnh, giải thích, nhập từ Excel), soạn đề theo ma trận, chiếu QR, giám sát, kết quả, khảo thí, báo cáo CLO |
| `api/index.php` | — | API JSON (PHP 8.1+, PDO MySQL) |

## Dùng chung với diemdanhsv.com

KIEMTRA **không có CSDL hay tài khoản riêng**:

- đọc `domains/diemdanhsv.com/private/config.php` (file bí mật sẵn có của site chính);
- GV đăng nhập bằng tài khoản `users` của hệ thống điểm danh; chỉ thấy lớp mình phụ trách (`classes.LecturerID`);
- SV phải có trong `enrollments` của lớp mới vào được phòng thi;
- điểm thang 10 được ghi vào `grade_columns`/`grades` (mặc định cột “Giữa kỳ”) — SV xem ở trang Xem điểm;
- chỉ thêm 7 bảng tiền tố `kt_` (`db/schema.mysql.sql`), không sửa bảng nào của site chính.

## Triển khai lên Hostinger (thư mục con `/kiemtra`)

1. **Tạo bảng**: hPanel → Databases → phpMyAdmin → chọn CSDL của diemdanhsv → *Import* → `db/schema.mysql.sql`.
2. **Git deploy**: hPanel → Websites → `diemdanhsv.com` → Advanced → **Git** → *Add repository*
   - Repository: `https://github.com/Tuan-23880095/KIEMTRA.git` · Branch: `main`
   - **Install path: `kiemtra`** (thành `public_html/kiemtra`; thư mục phải trống)
   - Repo riêng tư: chép *SSH key* hPanel hiển thị vào GitHub → repo → Settings → **Deploy keys**.
   - Bật **Auto deployment** (dán Webhook URL vào GitHub → Settings → Webhooks).
3. Trong repo **diemdanhsv**, thêm dòng `/kiemtra/` vào `.gitignore` để hai repo không giẫm lên nhau.
4. Kiểm tra: `https://diemdanhsv.com/kiemtra/api/index.php?action=ping` → `{"status":"success",…}`;
   mở `https://diemdanhsv.com/kiemtra/gv.html`, đăng nhập bằng tài khoản điểm danh.
5. (Tuỳ chọn) đặt liên kết “Kiểm tra trực tuyến” trên trang chủ diemdanhsv.com trỏ tới `/kiemtra/`.

> Nếu hPanel không cho thêm repo thứ hai cho cùng website: tạo subdomain `kiemtra.diemdanhsv.com`,
> deploy repo vào đó, và thêm vào `.htaccess` của site chính:
> `RewriteRule ^kiemtra(/.*)?$ https://kiemtra.diemdanhsv.com$1 [R=302,L]`.
> Khi đó sửa đường dẫn tìm `private/config.php` hoặc đặt `upload_dir` trong mục `kiemtra` của config.

Ảnh câu hỏi lưu ở `private/kiemtra-uploads/` (ngoài `public_html`, không bị auto-deploy xoá).

## Chống gian lận — làm được gì, không làm được gì

Trình duyệt **phát hiện và ghi nhận**, không điều khiển được hệ điều hành:

| Phát hiện (tính vi phạm) | Ghi chú |
|---|---|
| Rời trang: chuyển tab, mở app khác, về màn hình chính, tắt màn hình | `visibilitychange` — mọi thiết bị |
| Mất tiêu điểm lâu hơn N giây: thanh thông báo, cửa sổ nổi, cửa sổ khác | `blur` — có ngưỡng bỏ qua để tránh tính oan |
| Chia đôi màn hình / thu nhỏ cửa sổ | so diện tích cửa sổ với lúc bắt đầu (bỏ qua khi bàn phím ảo bật) |
| Thoát toàn màn hình | laptop, Android; **iPhone không có API toàn màn hình** |
| Sao chép / cắt / dán, F12, Ctrl+Shift+I, PrintScreen | bị chặn + ghi |
| Tải lại / mở lại trang khi đang thi | |
| Cùng MSSV vào từ thiết bị thứ hai | bị **chặn**; GV bấm “Cho đổi máy” khi máy hỏng/hết pin |

Vượt ngưỡng (mặc định 3) → **khoá bài** (GV nhập PIN 6 số tại máy SV hoặc mở trên bảng giám sát; mỗi lần mở
cho thêm 1 lượt) hoặc **tự nộp** (tuỳ thiết lập). Các tín hiệu cách nhau < 2 giây gộp làm một vi phạm.
Lúc chờ trong phòng chuẩn bị, rời trang **không** bị tính.

Bổ trợ: xáo câu và phương án theo từng SV (giữ thứ tự với phương án “Tất cả các ý trên”, “Cả A và B”…),
hình mờ MSSV + họ tên, đáp án **không bao giờ** gửi xuống máy SV trước khi công bố, đồng hồ theo máy chủ,
câu hỏi đóng băng khi mở phòng, nhật ký sự kiện từng SV.

**Không phát hiện được** (giám thị giám sát trực tiếp): điện thoại/thiết bị thứ hai, chụp bằng phím cứng
trên điện thoại, nhìn bài, trao đổi, AI trên máy khác.

## Khảo thí & CLO

- Phổ điểm, TB, SD, trung vị, tỉ lệ ≥ 5; **Cronbach α** (= KR-20 khi chấm 0/1), **SEM**.
- Từng câu: độ khó **p**, độ phân biệt **D** (nhóm 27% cao/thấp), **r_pb** câu–tổng (đã loại câu đó),
  phân tích phương án nhiễu (số SV chọn, nhóm cao/thấp), cảnh báo câu quá dễ/khó, phân biệt âm, phương án không ai chọn.
- CLO: SV **đạt** khi % điểm phần CLO ≥ ngưỡng (mặc định 50%); lớp **đạt** khi % SV đạt ≥ mục tiêu (mặc định 70%).
  Báo cáo CLO theo lớp gộp nhiều bài kiểm tra.
- Xuất CSV (UTF-8, mở thẳng bằng Excel): bảng điểm + % CLO từng SV; phân tích câu + CLO + thống kê.

## Nhập câu hỏi từ Excel

Cột (dòng đầu là tiêu đề): `loai, cau_hoi, A, B, C, D, E, F, dap_an, giai_thich, clo, bloom, do_kho, diem, chu_de`

- `loai`: `single` | `multi` | `short` — `dap_an`: `B` hoặc `A,C`; câu `short`: `thạch anh|quartz`
- `bloom`: 1–6 hoặc Nhớ/Hiểu/Vận dụng/Phân tích/Đánh giá/Sáng tạo — `do_kho`: 1–3 hoặc Dễ/TB/Khó
- Mẫu 18 câu Đá & khoáng vật: `docs/mau-cau-hoi-da-khoang-vat.csv` (**nội dung mẫu — thầy rà soát trước khi dùng**).

## Chạy thử trên máy

```bash
php tools/dev-setup.php .dev                       # SQLite + dữ liệu mẫu: gv / 123456, lớp 25DC1, 40 SV
KIEMTRA_CONFIG=$PWD/.dev/config.php php -S 127.0.0.1:8090 -t .
php tools/e2e-test.php http://127.0.0.1:8090       # kiểm thử trọn luồng (≈ 90 bước)
```

`tools/`, `db/`, `docs/`, `api/lib/`, `.git` bị `.htaccess` chặn trên host thật.
