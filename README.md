# Cursor 9Router Automation

Bộ script macOS tự động bật/tắt 9Router, tạo public tunnel và cập nhật URL mới vào Cursor.

## Why

Khi không có server để host 9Router cố định, bạn phải chạy 9Router trên máy local. Tuy nhiên, Cursor không cho đặt một URL HTTP local làm OpenAI Base URL nên cần sử dụng public tunnel.

Mỗi lần bật 9Router, tunnel có thể tạo URL mới. Việc sao chép URL rồi cập nhật lại cấu hình Cursor bằng tay hằng ngày gây mất thời gian và dễ sai sót.

## Solves This

Bộ script tự động hóa toàn bộ quy trình:

- Bật hoặc tắt 9Router bằng một lệnh.
- Tạo public tunnel và lấy URL mới.
- Thêm hậu tố `/v1` vào endpoint.
- Tự động cập nhật `openAIBaseUrl` trong Cursor.
- Khởi động lại Cursor để áp dụng cấu hình.

## How It Works

### `router-cursor-on`

1. Kiểm tra các dependency và database cấu hình của Cursor.
2. Khởi động 9Router tại `127.0.0.1:20128`.
3. Bật Quick Tunnel do 9Router quản lý và đọc public URL.
4. Đóng Cursor an toàn và sao lưu database cấu hình.
5. Cập nhật `openAIBaseUrl` với URL tunnel có hậu tố `/v1`.
6. Mở lại Cursor sau khi cập nhật thành công.

Nếu cập nhật database thất bại, script sẽ tự khôi phục bản sao lưu cùng các file SQLite WAL/SHM liên quan.

### `router-cursor-off`

1. Tìm 9Router, listener trên cổng `20128` và tunnel liên quan.
2. Gửi `SIGTERM` để các tiến trình dừng an toàn.
3. Chỉ dùng `SIGKILL` nếu tiến trình không phản hồi sau thời gian chờ.

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

## Quick Start

Cấp quyền thực thi ở lần đầu:

```zsh
chmod +x router-cursor-on router-cursor-off
```

Khởi động 9Router, bật tunnel và cấu hình Cursor:

```zsh
./router-cursor-on
```

Dừng 9Router và tunnel:

```zsh
./router-cursor-off
```

> `router-cursor-on` có thể tự đóng rồi mở lại Cursor. Hãy lưu công việc đang chỉnh sửa trước khi chạy.

## File và dữ liệu liên quan

- Log 9Router: `~/.9router.log`
- Trạng thái tunnel: `~/.9router/tunnel/state.json`
- Database Cursor:
  `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb`
- Bản sao lưu gần nhất: `~/.cursor-state.vscdb.backup`

## Giới hạn

- Script hiện dành riêng cho macOS.
- Cổng `20128` đang được cấu hình cố định trong cả hai script.
- Cách lưu cấu hình nội bộ của Cursor có thể thay đổi sau một bản cập nhật. Nên kiểm tra lại script nếu Cursor không nhận Base URL.
