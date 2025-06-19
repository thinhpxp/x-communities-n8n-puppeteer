# n8n Workflow: Tự động đăng bài lên X (Twitter) Community

![Made with n8n](https://img.shields.io/badge/Made%20with-n8n-blue?style=for-the-badge&logo=n8n)

##  Tổng quan

Đây là một workflow n8n cơ bản được thiết kế để tự động hóa việc đăng bài lên một **Community** cụ thể trên nền tảng X (trước đây là Twitter). Workflow sẽ đọc nội dung từ một Google Sheet, sử dụng cookie để xác thực phiên đăng nhập, và điều khiển trình duyệt ảo (thông qua Puppeteer) để thực hiện các thao tác như một người dùng thực thụ.
Nó phù hợp nếu:
- Bạn không có ý định mua gói premium của x để tận dụng hết các API cao cấp.
- Bạn không có ý định đăng tải quá nhiều nội dung trong một ngày lên các hội nhóm của x.com.
- Bạn đang selfhost n8n [Vì n8n cloud không hỗ trợ puppeteer].
![image](https://github.com/user-attachments/assets/895b34ff-f377-4d9f-9b01-e87db01080cc)


## Tính năng chính

-   **Nguồn nội dung linh hoạt**: Lấy nội dung bài đăng và tên Community mục tiêu trực tiếp từ Google Sheet.
-   **Đăng nhập không cần mật khẩu**: Sử dụng cookie đã được lưu để xác thực, tăng cường bảo mật và tránh các cơ chế phát hiện bot phức tạp.
-   **Mô phỏng hành vi người dùng**: Sử dụng Puppeteer để điều khiển trình duyệt, tự động click vào các nút, nhập văn bản và chọn đúng Community.
-   **Cơ chế chờ thông minh**: Thêm các khoảng trễ ngẫu nhiên giữa các hành động để giảm khả năng bị X phát hiện và chặn.
-   **Báo cáo kết quả**: Trả về trạng thái thành công hoặc thất bại sau khi thực thi, giúp dễ dàng theo dõi.

## Luồng hoạt động của Workflow

Workflow được xây dựng từ các node sau, thực hiện một chuỗi các bước logic:

1.  **Trigger (When clicking ‘Execute workflow’)**: Kích hoạt workflow thủ công (có thể dễ dàng thay thế bằng trigger theo lịch trình - Cron).
2.  **Content (Google Sheets)**: Đọc dữ liệu từ một Google Sheet được chỉ định. Sheet này chứa nội dung bài đăng, tên Community, các bạn có thể tùy biến theo nhu cầu thực tế.
3.  **load_cookies_file (Read Binary File)**: Đọc tệp `x-cookies.json` chứa thông tin cookie xác thực từ hệ thống tệp của n8n. Các bạn có thể lưu trữ cookies theo các hình thức khác tùy nhu cầu.
4.  **Extract to json (Extract from File)**: Chuyển đổi nội dung tệp cookie từ dạng nhị phân sang định dạng JSON mà các node sau có thể sử dụng.
5.  **merge_input_data (Code)**: Tổng hợp và cấu trúc lại dữ liệu từ các bước trước (nội dung, tên community, và cookie) thành một đối tượng JSON duy nhất để truyền vào node Puppeteer.
6.  **x (Puppeteer)**: Đây là node chính thực hiện các công việc:
    -   Điều hướng đến trang chủ X.
    -   Mở hộp thoại đăng bài (Post).
    -   Nhập nội dung bài viết.
    -   Mở danh sách đối tượng ("Choose audience").
    -   Tìm và chọn đúng Community từ danh sách.
    -   Nhấn nút "Post" để hoàn tất.

## Yêu cầu và Cài đặt

Trước khi có thể sử dụng workflow này, bạn cần chuẩn bị các yếu tố sau:

### 1. Selfhost n8n
- Cá nhân mình tận dụng laptop cũ và tự selfhost n8n chạy trên docker, trong môi trường windows 10.
- Sử dụng [image thinhpxp/n8nplus](https://hub.docker.com/r/thinhpxp/n8nplus). Nó được tùy biến dựa trên image n8nio gốc, được tích hợp thêm:
  **Puppeteer** và các thư viện cần thiết; **Curl** để có thể sử dụng hết khả năng của Execute Command node trong n8n.
- Khi cài đặt nhớ sử dụng chức năng Bind Mounts của docker để liên kết n8n container với máy tính vật lý. Việc này sẽ giúp thuận tiện khi các bạn dùng n8n xử lý các tệp lớn. Ví dụ với cấu hình bên dưới, n8n lúc này đã có thể tương tác với thư mục D:\n8n_share trên máy tính vật lý. Trong các node n8n cần xử lý tệp chỉ cần để đường dẫn là **/files**
  ![image](https://github.com/user-attachments/assets/112bc8c2-9920-484d-bf2d-2e3dd3a9069a)


### 2. Cài đặt Community Node Puppeteer
Ở mục số 1, chúng ta đã sử dụng image **thinhpxp/n8nplus** được mình tích hợp sẵn các thư viện cần thiết của `Puppeteer`. Ở bước này, các bạn cần kích hoạt puppeteer node để sử dụng trong n8n bằng cách:
-   Vào **Settings > Community Nodes**.
-   Nhấn **Install**.
-   Nhập `n8n-nodes-puppeteer` và tiến hành cài đặt.

### 3. Chuẩn bị Google Sheet
Tạo một Google Sheet có cấu trúc cột như sau. Tên cột (dòng đầu tiên) phải chính xác để workflow hoạt động. Tùy nhu cầu thực tế mà các bạn có thể tùy biến sheet này. Sử dụng node Google Sheet trong n8n để kết nối.

| CONTENT                      | COMMUNITIES              | STATUS  |
| ---------------------------- | ------------------ | ------- |
| Đây là nội dung bài viết 1. | Tên Community của bạn | Waiting |
| Đây là nội dung bài viết 2.     | Tên Community của bạn | Waiting |

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
    -   Tạo một tệp mới tên là `x-cookies.json` trong thư mục `D:\n8n_share` trên máy tính windows của bạn (đây là thư mục mặc định mà n8n có thể truy cập).
    -   Dán nội dung JSON đã sao chép vào tệp này và lưu lại.
    -   Khi cấu hình node Read/Write From Disk trong n8n, chỉ cần nhập đường dẫn **/files/x-cookies.json** là sẽ truy cập được tệp.

## Hướng dẫn sử dụng

1.  **Tải Workflow**: Lưu tệp `X_community.json` vào máy tính của bạn.
2.  **Import vào n8n**:
    -   Trong giao diện n8n, chọn **Import > From File**.
    -   Chọn tệp `X_community.json` để tải lên.
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
