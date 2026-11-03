# **Phần 1: Báo cáo phân tích kiến trúc đa Profile**

## 1. Phân tích nguyên nhân sâu xa dẫn đến hiện tượng 'văng phiên đăng nhập' (Session Conflict) dưới góc độ lưu trữ Cookies trong thư mục User Data khi dùng chung 1 Profile.
 - Chia sẻ chung bộ nhớ cookie and session storage :các hệ thống web hiện đại định danh người dùng thông qua chuỗi session cookie và token xác thực lưu trong database tại thư mục
 - Hiện tượng chồng đè và ghi đè phiên : khi bác sĩ đang mở hệ thống , thực tập sinh mở thêm tab đăng nhập tài khoản cá nhân trên cùng cửa sổ đó , một số cơ chế bảo mật danh thiết bị . Hậu quả là hệ thống carepulse lập tức đá phiên của bác sĩ ra ngoaif để bảo vệ dữ liệu
 - xung đột tài nguyên bộ nhớ đệm : các đoạn mã chạy ngầm từ các trang web bên thứ ba có thể vô tình chiếm quyền kiểm soát hoặc gây tràn bộ đệm , làm gián đoạn các luồng API gọi dữ liệu thời gian của phần mềm khám chữa bệnh
## 2. Thiết kế Mô hình 3 Profile chuyên dụng cho máy tính trạm khám CarePulse
 | Tên Profile | mục đích sử dụng | màu theme | các loại dữ liệu được lữu trữ độc lập |
 | :--- | :---: | :---: | ---: |
 | CarePulse | dành riêng cho bác sĩ và điều dưỡng để thao tác nghiệp vụ khám chữa bệnh | xanh nước biển |-Cookies and session : Riêng biệt cho carepulse,History: Lưu lịch sử khám,Bookmarks:Thư mục y khoa, Extensions : chặn mã độc |
 | Personal | Dành cho nhân sự tra cứu thông tin cá nhân , kiểm tra email công cụ | xanh lá cây |-cookies and session : riêng biệt , History : riêng tư , bookmarks : thư mục y khoa , extensions : tiện ích văn phòng |
 | Sandbox | dành cho bác sĩ thực tập hoặc kiểm tra các tính năng hệ thống mới | màu cam |-cookies and sesion : xóa tự động hoặc cô lập hoàn toàn , history : không lưu trữ dài hạn , bookmarks : không có |
 # **Phần 2: Thiết kế hệ thống Bookmark & Quy trình an toàn**
   ## Dựng cây thư mục Bookmark chuẩn (`Ctrl + Shift + O`) gồm 3 thư mục chính phục vụ bác sĩ trực: `📁 01_CarePulse_Clinic`, `📁 02_Pharma_Portal`, `📁 03_Medical_Tools`. Liệt kê ít nhất 3 dịch vụ web cần lưu trong mỗi thư mục
 - **01_CarePulse_Clinic**
  1. *CarePulse EMR Dashboard* (`https://emr.carepulse.local/dashboard`) – Giao diện quản lý ca khám chính.
  2. *Lịch trực y tế phường* (`https://schedule.carepulse.local`) – Tra cứu ca trực của bác sĩ và điều dưỡng.
  3. *Hệ thống báo cáo y tế phường* (`https://reports.carepulse.local`) – Giao diện tổng hợp số liệu bệnh nhân hằng ngày.
- **02_Pharma_Portal**
  1. *Cổng tra cứu BHYT* (`https://gdbh.baohiemxahoi.gov.vn`) – Kiểm tra hạn thẻ bảo hiểm y tế của bệnh nhân ngay tại bàn khám.
  2. *Dược thư Quốc gia Việt Nam* (`https://duochu.gov.vn`) – Tra cứu liều lượng, chống chỉ định và tương tác thuốc chính thống.
  3. *Hệ thống quản lý dược phẩm VNPT* (`https://vnpt-pharm.local`) – Quản lý kho thuốc và xuất kho tại trạm.
- **03_Medical_Tools**
  1. *Google Dịch Y tế* (`https://translate.google.com`) – Hỗ trợ chuyển đổi nhanh thuật ngữ y khoa chuyên ngành.
  2. *Tra cứu mã bệnh ICD-10* (`https://icd.who.int/browse10`) – Tra cứu phân loại bệnh quốc tế chuẩn xác.
  3. *Hướng dẫn chẩn đoán và điều trị Bộ Y tế* (`https://kcb.vn/category/van-ban-chi-dao`) – Kho văn bản phác đồ điều trị chuẩn.
   
## **4.** Hướng dẫn áp dụng mẹo Favicon-Only cho 4 trang web dùng hàng ngày (CarePulse, Cổng BHYT, Dược thư, Google Dịch) để tiết kiệm diện tích thanh công cụ gấp 3 lần.
 - cách thực hiện :
   * B1 : chọn một bookmark bất kỳ trên thanh bookmarks bar
   * B2 : chọn edit
   * B3 :tại ô tên , xóa toàn bộ chữ viết
   * B4 :nhấn lưu
## **5.** Xử lý Bẫy an ninh: Khi có đoàn thanh tra hoặc khách mượn máy tính kiểm tra trong 30 phút, nhân viên phòng khám phải kích hoạt chế độ duyệt web nào (Guest Profile hay Incognito)? Giải thích lý do và cảnh báo nguy cơ nếu bật tính năng Cloud Sync tài khoản cá nhân trên máy trạm y tế này.
   
  - **Lựa chọn chế độ:** Bắt buộc sử dụng **Guest Profile (Hồ sơ Khách)**, tuyệt đối không dùng Incognito Mode (Ẩn danh).
- **Lý do chuyên môn:**
  - **Guest Profile** tạo ra một không gian hoàn toàn biệt lập, không chia sẻ bất kỳ cache hay dữ liệu nào với các Profile chính. Khi đóng cửa sổ Guest, toàn bộ dữ liệu duyệt web, lịch sử, tệp tải xuống và cookie sẽ bị xóa sạch vĩnh viễn mà không để lại dấu vết.
  - **Incognito Mode** vẫn có thể bị rò rỉ nếu người dùng vô tình mở rộng quyền Extension, hoặc để lộ các file tải xuống (`Downloads`) trong hệ thống Windows, hoặc giữ các kết nối mạng ngầm chưa đóng.
- **Cảnh báo nguy cơ tối thượng về Cloud Sync tài khoản cá nhân:**
  - Nhân sự tuyệt đối **không được phép** bật tính năng đồng bộ hóa tài khoản Google cá nhân (`Turn on Sync`) trên máy trạm y tế dùng chung.
  - *Hậu quả nghiêm trọng nếu vi phạm:* Trình duyệt sẽ tự động tải toàn bộ Mật khẩu đã lưu (Saved Passwords), Dữ liệu tự điền (Auto-fill Credit Cards/Forms), Lịch sử duyệt web cá nhân và Bookmark riêng tư từ tài khoản cá nhân đổ về máy tính trạm y tế. Khi sinh viên thực tập hoặc người khác sử dụng máy, họ có thể dễ dàng truy cập vào các tài khoản ngân hàng, mạng xã hội cá nhân hoặc làm lộ lọt dữ liệu định danh nhạy cảm của người bệnh.
 
 
 
