# n8n Workflow: Tự động đăng bài lên X (Twitter) Community

![Made with n8n](https://img.shields.io/badge/Made%20with-n8n-blue?style=for-the-badge&logo=n8n)

##  tổng quan

Đây là một workflow n8n được thiết kế để tự động hóa việc đăng bài lên một **Community** cụ thể trên nền tảng X (trước đây là Twitter). Workflow sẽ đọc nội dung từ một Google Sheet, sử dụng cookie để xác thực phiên đăng nhập, và điều khiển trình duyệt ảo (thông qua Puppeteer) để thực hiện các thao tác như một người dùng thực thụ.

## Tính năng chính

-   **Nguồn nội dung linh hoạt**: Lấy nội dung bài đăng và tên Community mục tiêu trực tiếp từ Google Sheet.
-   **Đăng nhập không cần mật khẩu**: Sử dụng cookie đã được lưu để xác thực, tăng cường bảo mật và tránh các cơ chế phát hiện bot phức tạp.
-   **Mô phỏng hành vi người dùng**: Sử dụng Puppeteer để điều khiển trình duyệt, tự động click vào các nút, nhập văn bản và chọn đúng Community.
-   **Cơ chế chờ thông minh**: Thêm các khoảng trễ ngẫu nhiên giữa các hành động để giảm khả năng bị X phát hiện và chặn.
-   **Báo cáo kết quả**: Trả về trạng thái thành công hoặc thất bại sau khi thực thi, giúp dễ dàng theo dõi.

## Luồng hoạt động của Workflow

Workflow được xây dựng từ các node sau, thực hiện một chuỗi các bước logic:

1.  **Trigger (When clicking ‘Execute workflow’)**: Kích hoạt workflow thủ công (có thể dễ dàng thay thế bằng trigger theo lịch trình - Cron).
2.  **My Content1 (Google Sheets)**: Đọc dữ liệu từ một Google Sheet được chỉ định. Sheet này chứa nội dung bài đăng, tên Community, và trạng thái.
3.  **load_cookies_file (Read Binary File)**: Đọc tệp `x-cookies.json` chứa thông tin cookie xác thực từ hệ thống tệp của n8n.
4.  **Extract to json (Extract from File)**: Chuyển đổi nội dung tệp cookie từ dạng nhị phân sang định dạng JSON mà các node sau có thể sử dụng.
5.  **đóng gói (Code)**: Tổng hợp và cấu trúc lại dữ liệu từ các bước trước (nội dung, tên community, và cookie) thành một đối tượng JSON duy nhất để truyền vào node Puppeteer.
6.  **x (Puppeteer)**: Đây là node chính thực hiện các công việc:
    -   Mở một trang trình duyệt ảo.
    -   Thiết lập cookie để đăng nhập vào X.
    -   Điều hướng đến trang chủ X.
    -   Mở hộp thoại đăng bài (Post).
    -   Nhập nội dung bài viết.
    -   Mở danh sách đối tượng ("Choose audience").
    -   Tìm và chọn đúng Community từ danh sách.
    -   Nhấn nút "Post" để hoàn tất.

## Yêu cầu và Cài đặt

Trước khi có thể sử dụng workflow này, bạn cần chuẩn bị các yếu tố sau:

### 1. Cài đặt n8n
Đảm bảo bạn đã có một phiên bản n8n đang hoạt động.

### 2. Cài đặt Node Puppeteer
Workflow này yêu cầu node `n8n-nodes-puppeteer`. Nếu chưa có, bạn cần cài đặt nó trong n8n:
-   Vào **Settings > Community Nodes**.
-   Nhấn **Install**.
-   Nhập `n8n-nodes-puppeteer` và tiến hành cài đặt.

### 3. Chuẩn bị Google Sheet
Tạo một Google Sheet có cấu trúc cột như sau. Tên cột (dòng đầu tiên) phải chính xác để workflow hoạt động.

| CONTENT                      | GROUP              | STATUS  |
| ---------------------------- | ------------------ | ------- |
| Đây là nội dung bài viết 1. | Tên Community của bạn | Waiting |
| Nội dung thú vị khác đây.     | Tên Community của bạn | Waiting |

-   **CONTENT**: Nội dung bạn muốn đăng.
-   **GROUP**: Tên chính xác của X Community mà bạn muốn đăng bài vào.
-   **STATUS**: (Tùy chọn) Dùng để theo dõi trạng thái. Workflow này chưa cập nhật lại trạng thái, nhưng bạn có thể dễ dàng mở rộng để làm điều đó.

### 4. Lấy và lưu trữ Cookie của X
Đây là bước quan trọng nhất để xác thực.

1.  **Cài đặt Extension**: Cài đặt một extension trình duyệt giúp xuất cookie dưới dạng JSON, ví dụ:
    -   [Cookie-Editor](https://cookie-editor.com/) (Chrome, Firefox, Edge, Opera)
    -   [Get cookies.txt LOCALLY](https://chrome.google.com/webstore/detail/get-cookiestxt-locally/ggnmcofbehhlojdmmhmddecmjhonkihh) (Chrome)

2.  **Đăng nhập vào X**: Mở một tab mới và đăng nhập vào tài khoản X của bạn như bình thường.

3.  **Xuất Cookie**:
    -   Click vào biểu tượng extension vừa cài.
    -   Chọn **Export > Export as JSON**.
    -   Hành động này sẽ sao chép một chuỗi JSON chứa tất cả cookie vào clipboard của bạn.

4.  **Lưu tệp Cookie**:
    -   Tạo một tệp mới tên là `x-cookies.json` trong thư mục `.n8n/files/` của bạn (đây là thư mục mặc định mà n8n có thể truy cập).
    -   Dán nội dung JSON đã sao chép vào tệp này và lưu lại.

    *Lưu ý: Thư mục `files` có thể khác nếu bạn đã cấu hình `DATA_FOLDER` cho n8n.*

### 5. Cấu hình Credentials cho Google Sheets
-   Trong n8n, vào **Credentials > Add credential**.
-   Tìm **Google Sheets API** và tạo một credential mới bằng cách làm theo hướng dẫn để kết nối với tài khoản Google của bạn.

## Hướng dẫn sử dụng

1.  **Tải Workflow**: Lưu tệp `X_community (1).json` vào máy tính của bạn.
2.  **Import vào n8n**:
    -   Trong giao diện n8n, chọn **Import > From File**.
    -   Chọn tệp `X_community (1).json` để tải lên.
3.  **Cấu hình các Node**:
    -   **Node `My Content1` (Google Sheets)**:
        -   Chọn credential Google Sheets bạn đã tạo.
        -   Trong mục **Document ID**, dán ID của Google Sheet của bạn.
        -   Trong mục **Sheet Name**, chọn đúng trang tính chứa dữ liệu.
    -   **Node `load_cookies_file` (Read Binary File)**:
        -   Kiểm tra lại đường dẫn trong mục **File Path**. Mặc định là `/files/cookies/x-cookies.json`. Hãy chắc chắn rằng bạn đã tạo tệp cookie đúng vị trí này trong thư mục dữ liệu của n8n.
    -   **Node `đóng gói` (Code)**:
        -   Kiểm tra lại cách workflow lấy dữ liệu: `$('My Content1').first().json.CONTENT` và `$('My Content1').first().json.GROUP`. Đảm bảo tên cột trong Google Sheet của bạn khớp với `CONTENT` và `GROUP`.
4.  **Lưu và Kích hoạt**:
    -   Lưu lại workflow.
    -   Nhấn **Execute Workflow** để chạy thử.

## Lưu ý quan trọng và Gỡ lỗi

-   **Thay đổi giao diện của X**: X.com thường xuyên cập nhật giao diện người dùng. Điều này có thể làm cho các "selectors" (ví dụ: `a[aria-label="Post"]`, `div[role="textbox"]`) trong code Puppeteer bị lỗi thời. Nếu workflow thất bại, đây là nguyên nhân có khả năng cao nhất. Bạn sẽ cần cập nhật lại các selectors này bằng cách kiểm tra mã nguồn trang X.
-   **Cookie hết hạn**: Cookie có thời gian sống nhất định. Nếu workflow báo lỗi đăng nhập, bạn cần thực hiện lại quy trình xuất và lưu cookie mới.
-   **Rate Limit**: Việc đăng bài quá thường xuyên có thể khiến tài khoản của bạn bị giới hạn tạm thời bởi X. Hãy cấu hình trigger (Cron) với tần suất hợp lý.
-   **Headless Browser**: Node Puppeteer mặc định chạy ở chế độ "headless" (không có giao diện đồ họa). Để gỡ lỗi, bạn có thể tạm thời tắt chế độ này trong cài đặt của node Puppeteer để xem trình duyệt ảo đang làm gì.

---
*Workflow này được tạo ra cho mục đích tự động hóa cá nhân. Vui lòng sử dụng có trách nhiệm và tuân thủ các điều khoản dịch vụ của X.*
