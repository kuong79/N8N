# 36NEWS – Kuong36 Entertainment

Bộ công cụ làm video tin tức TikTok cho kênh **@kuong36**: logo, khung video Canva và workflow n8n tự lấy tin từ báo chính thống rồi viết tiêu đề bằng AI.

## Có gì trong repo

| File | Dùng để |
|---|---|
| `assets/36news-logo.png` | Logo chữ ký "36NEWS" nền trong suốt (trắng, có bóng mờ) – đặt góc trên phải video |
| `n8n/36news-workflow.json` | Workflow n8n: lấy tin → lọc → AI viết tiêu đề/caption/hashtag → lưu Google Sheets + gửi email duyệt |

Thiết kế trên Canva (nằm trong tài khoản Canva của bạn):
- Khung video tin tức 9:16: https://canva.link/qt6j65upm7bd2r5
- Logo ảnh đại diện K36 ENTERTAINMENT: https://canva.link/3mnu7wcjgti7g54

## Workflow n8n hoạt động thế nào

```
Lịch chạy (7h, 12h, 19h giờ VN)
  → Danh sách báo chính thống (VnExpress, Tuổi Trẻ, Thanh Niên, Dân Trí, VTV)
  → Đọc RSS (nguồn nào lỗi thì bỏ qua, không dừng cả luồng)
  → Lọc tin mới (trong 6 giờ gần nhất, tối đa 8 tin/lần)
  → Bỏ tin đã xử lý ở các lần chạy trước
  → Claude viết tiêu đề IN HOA + caption + hashtag + phân loại + cờ "nhạy cảm"
  → Lưu vào Google Sheets  +  Gửi bản nháp qua Gmail để bạn duyệt
```

AI chỉ được viết lại từ tiêu đề và tóm tắt của bài báo, không được thêm chi tiết. Tin về tai nạn, tội phạm, trẻ em, chính trị hoặc chưa xác nhận sẽ được đánh dấu **CẦN DUYỆT KỸ**. Caption luôn kèm dòng "Nguồn: <tên báo>".

## Cài đặt (khoảng 15 phút)

1. **Nhập workflow:** n8n → *Workflows* → *Import from File* → chọn `n8n/36news-workflow.json`.
2. **Khóa API Claude:**
   - Tạo khóa tại https://console.anthropic.com → *API Keys*.
   - Trong n8n mở node **Claude viết tiêu đề** → *Credential* → *Create new* → loại **Header Auth**.
   - *Name*: `x-api-key`, *Value*: khóa vừa tạo.
3. **Google Sheets:**
   - Tạo một Google Sheet, đặt tên tab là `Tin`, dòng 1 ghi các cột:
     `date | headline | caption | hashtags | category | sensitive | status | source | title | link | publishedAt`
   - Mở node **Lưu vào Google Sheets**, kết nối tài khoản Google, dán link Sheet thay cho `DAN_LINK_GOOGLE_SHEET_VAO_DAY`.
4. **Gmail:** mở node **Gửi bản nháp qua Gmail**, kết nối tài khoản Gmail, thay `EMAIL_CUA_BAN@gmail.com` bằng email nhận bản nháp.
5. **Chạy thử:** bấm *Test workflow*. Kiểm tra Sheet và hộp thư. Ổn thì bật **Active** để chạy tự động.

### Tùy chỉnh nhanh
- **Đổi giờ chạy:** node *Lịch chạy*, biểu thức cron `0 7,12,19 * * *`.
- **Thêm/bớt báo:** node *Danh sách báo chính thống*, sửa mảng `FEEDS` (mỗi dòng gồm `source` và `url` RSS).
- **Số tin mỗi lần / khung giờ lọc:** node *Lọc tin mới*, sửa `MAX_ITEMS` và `HOURS`.

### Chi phí AI
Mỗi tin là một lần gọi Claude Opus 5.5 ở mức `effort: low`, ước tính khoảng 1–2 xu đô mỗi tin. Nếu chạy đủ 3 lần/ngày × 8 tin thì khoảng 10–15 USD/tháng; giảm `MAX_ITEMS` để tiết kiệm. Xem giá mới nhất tại https://www.anthropic.com/pricing.

## Quy trình ra một video

1. Mở email "36NEWS – N tin mới chờ duyệt", chọn tin muốn làm. **Bấm vào bài gốc để kiểm tra.**
2. Trên Canva mở khung video → *Tạo bản sao*.
3. Thay ảnh/video ở nửa trên, dán **tiêu đề** và **ngày đăng** từ email.
4. Xuất MP4 → đăng TikTok, dán **caption + hashtag**, ghi nguồn.

## Lưu ý pháp lý
- Chỉ dùng video/ảnh bạn tự quay, được phép dùng, hoặc có giấy phép. Không tải lại video của báo khác.
- Luôn ghi nguồn bài báo trong caption.
- Không đăng tin chưa kiểm chứng. Tin có cờ **CẦN DUYỆT KỸ** cần đọc kỹ bài gốc trước khi đăng.
