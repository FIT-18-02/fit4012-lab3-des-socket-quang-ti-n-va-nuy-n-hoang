# Threat Model - Lab 3

## Thông tin nhóm
- Thành viên 1: Đặng Quang Tiến - MSSV: 1871020570
- Thành viên 2: Nguyễn  - MSSV: 1871020250

## Assets
- Nội dung bản tin gốc (Plaintext message).
- Khóa mã hóa (DES Key).
- Vector khởi tạo (IV).

## Attacker model
Kẻ tấn công Man-in-the-Middle (MitM) có khả năng can thiệp vào đường truyền TCP giữa Sender và Receiver. Kẻ này có thể nghe lén (sniffing) toàn bộ lưu lượng mạng (ví dụ dùng Wireshark), chặn bắt, chỉnh sửa (tamper) hoặc phát lại các gói tin.

## Threats
1. **Lộ lọt thông tin (Information Disclosure):** Do hệ thống gửi trực tiếp Key và IV dưới dạng plaintext ở đầu gói tin (20 byte header), kẻ tấn công dễ dàng bắt được và dùng chính Key/IV này để giải mã Ciphertext, lộ hoàn toàn bản tin gốc.
2. **Thay đổi nội dung (Tampering / Malleability):** Kẻ tấn công có thể thay đổi các bit trong Ciphertext trên đường truyền. Dù dùng chế độ CBC có tính lan truyền lỗi, nhưng do không có cơ chế xác thực toàn vẹn (MAC), gói tin bị sửa đổi vẫn có thể được Receiver chấp nhận hoặc gây lỗi padding dẫn đến sập hệ thống (Crash/DoS).
3. **Tấn công phát lại (Replay Attack):** Kẻ tấn công bắt lại một gói tin hợp lệ cũ và gửi lại cho Receiver nhiều lần. Receiver hiện tại không có cơ chế phân biệt tin nhắn cũ/mới nên sẽ xử lý lại từ đầu.

## Mitigations
1. **Bảo vệ kênh truyền và khóa:** Không gửi Key chung với bản tin. Cần bọc toàn bộ kết nối socket bằng TLS/SSL, hoặc dùng giao thức trao đổi khóa an toàn như Diffie-Hellman (DHE) để hai bên thống nhất session key.
2. **Xác thực tính toàn vẹn (Integrity):** Chuyển sang sử dụng thuật toán mã hóa có xác thực (AEAD) như AES-GCM, hoặc sử dụng cơ chế MAC (Message Authentication Code) như HMAC gắn vào cuối gói tin để Receiver kiểm tra dữ liệu có bị chỉnh sửa hay không.
3. **Chống phát lại:** Thêm Sequence Number (số thứ tự) hoặc Timestamp (dấu thời gian) vào bản tin trước khi mã hóa để loại bỏ các gói tin bị gửi lại. Bỏ DES và thay bằng AES.

## Residual risks
Ngay cả khi áp dụng TLS/SSL và AES-GCM, nếu thiết bị của Sender hoặc Receiver bị nhiễm mã độc (malware, keylogger, memory scraper), kẻ tấn công vẫn có thể lấy được bản rõ (plaintext message) trước khi nó được mã hóa hoặc ngay sau khi nó được giải mã tại điểm cuối (Endpoint Compromise).
