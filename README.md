# Cursor 9Router Automation

Bộ script macOS giúp khởi động hoặc dừng 9Router, tạo public tunnel và tự động cập nhật OpenAI Base URL trong Cursor.

## Vấn đề được giải quyết

Khi dùng 9Router hằng ngày nhưng không có server để host cố định, bạn phải chạy 9Router ở máy local thông qua một public tunnel. Mỗi lần khởi động, tunnel tạo ra URL mới, trong khi Cursor không cho cấu hình trực tiếp URL HTTP local. Vì vậy, bạn phải sao chép URL mới và cập nhật lại OpenAI Base URL trong Cursor theo cách thủ công.

Bộ script này tự động hóa toàn bộ quy trình: khởi động 9Router, lấy URL tunnel mới, cập nhật cấu hình Cursor và mở lại Cursor, giúp tiết kiệm thời gian mỗi ngày.

## Chức năng

- `9router-on`
  - Khởi động 9Router tại `127.0.0.1:20128`.
  - Bật Quick Tunnel do 9Router quản lý.
  - Lấy public URL và thêm hậu tố `/v1`.
  - Đóng Cursor an toàn, sao lưu database cấu hình rồi cập nhật `openAIBaseUrl`.
  - Mở lại Cursor sau khi cập nhật thành công.
- `9router-off`
  - Dừng 9Router, listener trên cổng `20128` và tunnel liên quan.
  - Gửi `SIGTERM` trước, chỉ dùng `SIGKILL` nếu tiến trình không phản hồi.

## Yêu cầu

- macOS.
- Cursor đã được cài đặt và từng khởi chạy.
- 9Router CLI có trong `PATH`.
- Các tiện ích hệ thống: `zsh`, `python3`, `pgrep`, `lsof`, `ps`, `sort`, `tr`, `osascript` và `open`.
- 9Router đã có thông tin tunnel cục bộ trong thư mục `~/.9router`.

Kiểm tra 9Router:

```zsh
command -v 9router
```

## Cách sử dụng

Cấp quyền thực thi ở lần đầu:

```zsh
chmod +x 9router-on 9router-off
```

Khởi động 9Router, bật tunnel và cấu hình Cursor:

```zsh
./9router-on
```

Dừng 9Router và tunnel:

```zsh
./9router-off
```

> `9router-on` có thể tự đóng rồi mở lại Cursor. Hãy lưu công việc đang chỉnh sửa trước khi chạy.

## File và dữ liệu liên quan

- Log 9Router: `~/.9router.log`
- Trạng thái tunnel: `~/.9router/tunnel/state.json`
- Database Cursor:
  `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb`
- Bản sao lưu gần nhất: `~/.cursor-state.vscdb.backup`

Nếu cập nhật database thất bại, script sẽ tự khôi phục bản sao lưu cùng các file SQLite WAL/SHM liên quan.

## Giới hạn

- Script hiện dành riêng cho macOS.
- Cổng `20128` đang được cấu hình cố định trong cả hai script.
- Cách lưu cấu hình nội bộ của Cursor có thể thay đổi sau một bản cập nhật. Nên kiểm tra lại script nếu Cursor không nhận Base URL.
