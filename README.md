# Duyên Deal — Netlify starter

Website tĩnh + Netlify Function tiếp nhận đơn + Supabase lưu đơn. Chưa được deploy và chưa kết nối Zalo mặc định.

## Tính năng
- Form vé xem phim, voucher đồ uống, Shopee.
- Không yêu cầu tên, số điện thoại hoặc email.
- Netlify Function kiểm tra dữ liệu và ghi đơn vào Supabase.
- Webhook riêng phía máy chủ để nối với một workflow thông báo Zalo đã được cấu hình.
- Không có trang quản trị đơn hàng trong phiên bản starter này; quản trị viên có thể xem dữ liệu trong Supabase Dashboard.

## 1. Tạo Supabase database
1. Tạo project tại https://supabase.com/.
2. Mở SQL Editor, chạy toàn bộ `supabase-schema.sql`.
3. Vào Project Settings → API, lấy Project URL và `service_role` key. Giữ kín service role key.

## 2. Deploy lên Netlify
1. Giải nén thư mục, đẩy các file lên một repository GitHub mới.
2. Đăng nhập https://app.netlify.com/ → Add new site → Import an existing project → chọn repository.
3. Build command có thể để trống; publish directory đặt `.`. `netlify.toml` cũng đã khai báo cấu hình.
4. Vào Site configuration / Project configuration → Environment variables, thêm:
   - `SUPABASE_URL` = URL project Supabase
   - `SUPABASE_SERVICE_ROLE_KEY` = service_role key (chỉ phía Functions)
   - `NOTIFY_WEBHOOK_URL` = URL webhook của workflow tích hợp Zalo (tùy chọn)
5. Deploy site. Sau khi deploy thành công, Netlify sẽ cấp URL dạng `https://ten-site.netlify.app`. URL chỉ tồn tại sau khi bạn thực hiện bước này.
6. Sau khi đổi environment variables, deploy lại để thay đổi có hiệu lực.

## 3. Tích hợp Zalo — cần lưu ý
Website không thể tự gửi tin nhắn vào tài khoản Zalo cá nhân bất kỳ chỉ nhờ biết tên người nhận. Cần một phương án được Zalo hỗ trợ, thường là Zalo Official Account (OA) và API/đối tác phù hợp, hoặc một workflow riêng có quyền gửi thông báo theo điều kiện của Zalo. `NOTIFY_WEBHOOK_URL` là điểm nối tùy chọn cho endpoint đó; nó chưa tự động kết nối Zalo.

Khi cấu hình webhook, dùng dịch vụ/endpoint đáng tin cậy, xác thực request, giới hạn quyền và không để token Zalo trong frontend. Đảm bảo OA/API có quyền gửi đến đúng đối tượng đích; nếu không, đơn vẫn được lưu database nhưng thông báo có thể không gửi được.

## 4. Xem đơn hàng
Mở Supabase → Table Editor → `orders`. Các trạng thái hợp lệ: `new`, `checking`, `confirmed`, `completed`, `cancelled`. Không công khai service role key. Starter này chưa cung cấp trang quản trị có đăng nhập để đổi trạng thái.

## 5. Chạy thử trên máy tính
Cài Node.js LTS, sau đó trong thư mục dự án chạy:
```bash
npm install
npx netlify-cli dev
```
Cần cấu hình các biến môi trường để gửi đơn thành công. Không đưa `.env` chứa secret lên GitHub.

## Bảo mật / giới hạn
- Không thu thập tên, email, số điện thoại trong form.
- Không nhập mật khẩu, OTP hoặc thông tin thanh toán Shopee.
- Cần bổ sung rate limiting/chống spam và trang quản trị xác thực trước khi dùng rộng rãi.
- Cần bổ sung quy trình lưu trữ/xóa đơn theo thời hạn phù hợp.
- Không tự động đặt vé hay đặt đơn Shopee; website chỉ nhận yêu cầu.


## Cấu hình Supabase key mới (sb_secret_*)
Bản này hỗ trợ biến `SUPABASE_SECRET_KEY` cho khóa bí mật mới của Supabase (`sb_secret_...`).
Trong Netlify, đặt `SUPABASE_URL` bằng Project URL gốc (không thêm `/rest/v1/`) và `SUPABASE_SECRET_KEY` bằng khóa bí mật. Có thể giữ `SUPABASE_SERVICE_ROLE_KEY` nếu đang dùng khóa `service_role` kiểu cũ. Sau khi thay mã nguồn/biến môi trường, hãy deploy lại. Không đưa khóa bí mật vào frontend hoặc GitHub.
