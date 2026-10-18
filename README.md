# TKA FieldCare — Quản lý nhà vườn

MVP web tiếng Việt, responsive, dùng React + Vite + Supabase Postgres/Auth/RLS. Dành cho kỹ thuật viên thăm vườn sầu riêng và quản lý giám sát.

## Tính năng đã có
- Đăng nhập email / mật khẩu qua Supabase Auth; vai trò `manager` và `staff`.
- Hồ sơ vườn: chủ vườn, điện thoại, địa chỉ, GPS, diện tích, giống, giai đoạn.
- Check-in / check-out với GPS hiện tại (vị trí, thời gian, 1 phiên đang mở / nhân viên).
- Nhật ký kỹ thuật (hiện trạng, hướng xử lý, pH, EC).
- Danh mục sản phẩm, đơn thuốc nhiều dòng, tổng tiền, trạng thái bán hàng.
- Lịch tái thăm, hoàn tất lịch, dashboard tổng hợp và báo cáo theo nhân viên.
- Lịch sử đơn thuốc, nhật ký và lượt ghé theo vườn.

## Cài và chạy
1. Cài Node.js 20.19+ hoặc 22.12+ và npm.
2. Tạo một dự án Supabase (https://supabase.com). Mở SQL Editor, chạy toàn bộ `supabase/schema.sql` **trên cơ sở dữ liệu mới**.
3. Trong Supabase Auth > Users, tạo user bằng email/mật khẩu; xác nhận email khi được yêu cầu.
4. Trong SQL Editor, tạo tổ chức và hồ sơ user (thay ID bằng ID thực của Auth > Users):
   ```sql
   insert into public.organizations (id,name)
   values ('11111111-1111-4111-8111-111111111111','Đội kỹ thuật TKA');

   insert into public.profiles(id,organization_id,full_name,role)
   values ('AUTH_USER_UUID','11111111-1111-4111-8111-111111111111','Quản lý khu vực','manager');
   ```
   Tạo các user tiếp theo trong Supabase Auth và thêm vào `profiles` với vai trò `staff` và cùng `organization_id`. Lưu ý: `AUTH_USER_UUID` phải thay bằng UUID hợp lệ.
5. Sao chép `.env.example` thành `.env.local`, cấu hình `VITE_SUPABASE_URL` và `VITE_SUPABASE_ANON_KEY` (publishable/anon key, **không** dùng service_role).
6. Chạy:
   ```bash
   npm install
   npm run dev
   ```
   Truy cập địa chỉ được in ra (mặc định http://localhost:5173).
7. Kiểm tra TypeScript và build: `npm run build`.

## Quyền hạn
- `manager`: đọc dữ liệu toàn tổ chức (bao gồm hoạt động theo nhân viên), quản lý danh mục sản phẩm, xem báo cáo toàn đội.
- `staff`: chỉ thấy vườn mình phụ trách và các lượt thăm, nhật ký, đơn hàng, tái thăm do mình tạo; xem danh mục sản phẩm toàn tổ chức.
- Cơ chế RLS xử lý phân quyền từ database, không dựa vào ẩn/hiện nút trên giao diện. Người dùng không tự chọn vai trò/quản lý.
- `organization_id` của bản ghi được chèn tự động qua trigger từ hồ sơ xác thực. Đây là cấu trúc MVP, cần kiểm thử bảo mật thêm trước triển khai thực tế.

## Lưu ý quan trọng trước triển khai thật
- **Dữ liệu:** Chưa có môi trường Supabase được kết nối trong bộ source này. Không có tài khoản hay dữ liệu khách hàng thực đi kèm. Người quản trị phải thiết lập như trên.
- **GPS:** Geolocation chỉ hoạt động tại HTTPS hoặc localhost và cần người dùng cho phép. GPS có thể sai lệch, vị trí thiết bị có thể bị giả mạo; check-in không phải bằng chứng xác minh tuyệt đối.
- **Đơn hàng:** MVP tạo `orders` và `order_items` bằng 2 yêu cầu liên tiếp. Cần triển khai transaction RPC PostgreSQL để tránh đơn trống nếu ghi chi tiết thất bại; giá/tổng tiền hiện từ client và cần xác thực lại ở server trước khi dùng doanh số cho kế toán.
- **Dữ liệu phiếu:** Sản phẩm và liều dùng do người dùng nhập, không có khuyến cáo BVTV được kiểm định. Kiểm tra nhãn, đăng ký, cách dùng và thời gian cách ly thực tế trước khi tư vấn cho khách.
- **Chưa bao gồm:** giao diện quản trị tạo user, upload ảnh cây, lưu offline, thông báo đẩy, xuất PDF/Excel, chỉnh sửa hồ sơ, tracking tự động, chống giả GPS, audit log, nhiều tổ chức tự đăng ký, xác nhận vị trí trong bán kính vườn, phân quyền giám sát theo địa bàn.
- **Vận hành:** Nên thêm audit log, backup, chính sách dữ liệu khách hàng, đổi mật khẩu, xác minh email, kiểm thử RLS, xác thực input phía backend và phân quyền API trước khi đưa lên môi trường thực.

## Cấu trúc
- `src/App.tsx`: UI, tính năng ứng dụng
- `src/style.css`: giao diện mobile-first responsive
- `src/data.ts`: kiểu dữ liệu và tiện ích
- `src/supabase.ts`: kết nối Supabase
- `supabase/schema.sql`: mô hình dữ liệu, trigger và RLS

## Roadmap
**Giai đoạn 2:** lưu ảnh, phân loại sâu bệnh, quản lý theo mùa vụ, lọc ngày/nhân viên, edit/delete có audit, RPC tạo đơn an toàn, báo cáo Excel/PDF, PWA/offline. **Giai đoạn 3:** quản trị nhân sự, phân tuyến đi vườn, KPI, nhắc tái thăm, dashboard bản đồ, tích hợp hệ thống bán hàng.
