**Học:** Maven lifecycle, dependency, Spring bean, constructor injection, MVC, cấu hình theo môi trường và migration.

**Làm:** tạo Spring Boot project, Maven Wrapper, Git, Compose cho MySQL, Flyway; tạo bảng category/product/SKU cần thiết và dữ liệu demo. Xây một trang danh sách sản phẩm đọc từ DB qua đầy đủ controller → service → repository.

**Đầu ra:** ứng dụng chạy local, README hướng dẫn khởi động và một trang có dữ liệu thật. Ghi log khởi động rõ ràng; có test dựng context và migration.

**Xong khi:** một database trống được tạo schema bằng migration, app khởi động được và hiển thị sản phẩm; chạy lại không tạo dữ liệu trùng.

### Giai đoạn 2 — Tài khoản, quyền và shop (16–22 giờ)

**Học:** authentication = bạn là ai; authorization = bạn được làm gì; hash mật khẩu, session, CSRF, Bean Validation.

**Làm:** đăng ký/đăng nhập/đăng xuất, địa chỉ, tạo shop, role; chống tự cấp quyền; kiểm tra chủ sở hữu. Tài khoản mới là BUYER, tạo shop hợp lệ mới bổ sung SELLER.

**Đầu ra:** hai người dùng độc lập, một người bán và admin demo; test email trùng, sai mật khẩu, thiếu CSRF và truy cập trái quyền.

**Xong khi:** sửa ID trong request không giúp người A đọc/sửa dữ liệu riêng của B; email trùng bị chặn cả ở DB.

### Giai đoạn 3 — Catalog và khu người bán (20–28 giờ)

**Học:** JPA mapping, lazy loading, DTO, pagination, index và N+1 query. N+1 là việc tải danh sách rồi phát sinh thêm một truy vấn cho từng bản ghi.

**Làm:** CRUD danh mục cho admin; tạo/sửa/ẩn sản phẩm và SKU cho seller; tìm kiếm, lọc, phân trang, chi tiết sản phẩm và trang shop. MVP chọn ảnh từ tập ảnh demo có sẵn, chưa upload tùy ý.

**Đầu ra:** danh sách sản phẩm, trang chi tiết và màn hình quản lý có thể sử dụng.

**Xong khi:** shop A không sửa được hàng shop B; SKU mặc định và SKU có lựa chọn đều hoạt động; danh sách lớn có giới hạn trang. Kiểm tra SQL và xử lý N+1 ở các màn hình chính. Chủ động tải dữ liệu cần thiết trong service; dùng `open-in-view=false` sau khi bố trí DTO/truy vấn phù hợp.

### Giai đoạn 4 — Giỏ hàng và checkout an toàn (28–40 giờ)

**Học:** ACID, transaction isolation ở mức cần dùng, race condition, khóa dòng, idempotency và snapshot. Race condition là kết quả sai do các request chạy xen kẽ.

**Làm:** giỏ nhóm theo shop, preview checkout, xác nhận giá, tạo đơn COD, trừ kho và xử lý gửi lặp theo thiết kế mục 6.

**Đầu ra:** người mua đi từ chọn SKU đến trang đơn thành công; integration test với MySQL thật trong container.

**Xong khi:** tồn kho không âm; không có đơn nửa chừng; gửi lại request không tạo thêm đơn; lỗi một SKU rollback toàn bộ đơn của shop.

### Giai đoạn 5 — Xử lý đơn và hoàn thiện trải nghiệm (18–26 giờ)

**Học:** state machine, lịch sử thay đổi, Post/Redirect/Get và kiểm thử luồng người dùng.

**Làm:** lịch sử đơn buyer, danh sách đơn seller, xác nhận/giao/hoàn tất/hủy; hoàn kho một lần; lịch sử trạng thái. Hoàn thiện responsive, lỗi form, trạng thái rỗng và hết hàng.

**Đầu ra:** kịch bản demo đầy đủ từ đăng ký đến hoàn tất đơn và một kịch bản hủy.

**Xong khi:** không thể nhảy từ PENDING_CONFIRMATION thẳng đến COMPLETED; hai lần hủy chỉ hoàn kho một lần; snapshot đơn không đổi khi sửa sản phẩm/địa chỉ.

### Giai đoạn 6 — Củng cố chất lượng và triển khai demo (24–34 giờ)

**Học:** CI = tự động build/test sau thay đổi; container, health check, log, HTTPS, backup/restore và quản lý cấu hình.

**Làm:** đóng gói app, pin phiên bản image, chạy app + DB bằng Compose. Thiết lập CI chạy Maven verify và test MySQL; deploy một app instance sau reverse proxy HTTPS khi có môi trường. DB dùng volume bền vững và không mở công khai; thực hành backup rồi restore vào DB riêng.

**Đầu ra:** hướng dẫn chạy, tài liệu deploy/khôi phục, kết quả kiểm thử, ảnh demo và giới hạn sản phẩm. Cấu hình rõ session trong bộ nhớ: restart làm người dùng đăng nhập lại; chưa hỗ trợ nhiều app instance.

**Xong khi:** từ checkout mã nguồn mới có thể chạy bằng README; CI xanh; smoke test sau deploy thành công; dữ liệu vẫn còn sau restart container; bản backup phục hồi được. Migration khi nâng cấp phải được thử trước, có phương án phục hồi; không giả định rollback code sẽ tự rollback schema.

## 9. Những bài test có giá trị nhất

Viết test cùng tính năng, không chờ tới giai đoạn cuối. Unit test kiểm tra quy tắc nhỏ; integration test kiểm tra Spring/SQL/transaction thật; kiểm thử trình duyệt xác nhận người dùng thực hiện được luồng.

| Tình huống | Kết quả cần chứng minh |
|---|---|
| Hai buyer cùng mua SKU chỉ còn 1 | Một đơn thành công, một đơn bị từ chối, tồn kho cuối = 0 |
| Đơn có hai SKU, SKU thứ hai hết hàng | Không tạo đơn, SKU thứ nhất không bị trừ kho |
| Hai request cùng checkout key | Một đơn, trừ kho một lần, kết quả trỏ về cùng đơn |
| Cùng key nhưng khác nội dung | Bị từ chối do xung đột |
| Giá ở browser bị sửa | Server không dùng giá giả; xử lý theo giá DB và bước xác nhận |
| Request trộn SKU của hai shop | Bị từ chối, không có thay đổi dữ liệu |
| Người A đoán ID đơn/địa chỉ của B | Không đọc hoặc sử dụng được dữ liệu của B |
| Seller A sửa SKU hoặc xử lý đơn shop B | Bị từ chối phía server |
| Hủy hai lần, hoặc hủy đồng thời với xác nhận | Chỉ một chuyển trạng thái hợp lệ; không hoàn kho lặp |
| Đổi tên/giá hàng và địa chỉ sau khi mua | Đơn cũ giữ nguyên snapshot và tổng tiền |
| POST thiếu CSRF token | Bị Security từ chối |
| Migration trên DB trống và DB phiên bản trước | Cả hai nâng lên schema mong muốn, không mất dữ liệu mẫu cần giữ |

Dùng Testcontainers chạy MySQL cùng dòng/bản đã chốt cho test quan hệ, khóa và transaction; không lấy H2 làm bằng chứng cho hành vi khóa của MySQL. Test đồng thời cần connection và transaction độc lập, có đồng bộ thời điểm chạy để thực sự tạo tranh chấp. [Testcontainers MySQL module](https://java.testcontainers.org/modules/databases/mysql/).

## 10. Quy trình làm việc cho mỗi tính năng

1. Viết user story và 3–5 acceptance criteria, gồm ít nhất một trường hợp thất bại và quyền truy cập.
2. Phác thảo màn hình/request, quy tắc nghiệp vụ và thay đổi schema.
3. Viết migration nếu cần, rồi implement một luồng xuyên suốt có thể chạy được.
4. Test nghiệp vụ có rủi ro; kiểm tra dữ liệu và SQL thực tế; thử thao tác bằng trình duyệt.
5. Tự review: quyền sở hữu, transaction, validation, lỗi và dữ liệu nhạy cảm.
6. Commit một thay đổi dễ hiểu, cập nhật tài liệu khi thay đổi quyết định.

Definition of Done: đáp ứng acceptance criteria, kiểm tra quyền tại server, có migration khi thay schema, test liên quan qua, chạy được theo README và không chứa bí mật. Độ bao phủ test là thông tin tham khảo; ưu tiên chứng minh những quy tắc có thể làm sai đơn hàng hoặc lộ dữ liệu.

## 11. Các bước đầu tiên để bắt đầu học và làm

- [ ] Viết lại bằng lời của bạn 8 quy tắc MVP và luồng checkout một shop.
- [ ] Vẽ ERD cho User → Shop → Product → SKU và Buyer → Order → OrderItem.
- [ ] Ôn HTTP, form và cookie; viết một bài JDBC đọc sản phẩm bằng truy vấn có tham số.
- [ ] Chuẩn bị JDK 21, IDE Java, Git, Docker có môi trường chạy Linux containers và công cụ xem MySQL.
- [ ] Tạo project Spring Boot với Maven Wrapper, Flyway và MySQL.
- [ ] Hoàn thành mục tiêu nhỏ đầu tiên: **mở `/products` và nhìn thấy sản phẩm đọc từ MySQL**.

Sau mỗi bước, hãy tự trả lời: “Đầu vào đến từ đâu? Quy tắc nào được bảo vệ? Dữ liệu nào được ghi? Nếu lỗi giữa chừng thì điều gì xảy ra?”. Trả lời được những câu này là thước đo tiến bộ quan trọng của dự án.
