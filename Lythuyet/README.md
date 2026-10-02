Câu 1: Value Types và Reference Types (Stack vs. Heap)
Sự khác biệt cốt lõi nằm ở cách bộ nhớ được cấp phát và lưu trữ dữ liệu:

Value Types (Kiểu giá trị):

Cơ chế: Lưu trữ trực tiếp giá trị của dữ liệu.

Vùng nhớ: Thường được cấp phát trên bộ nhớ Stack (nếu là biến cục bộ). Stack hoạt động theo cơ chế LIFO (Vào sau ra trước), có tốc độ truy xuất cực nhanh.

Giải phóng: Bộ nhớ tự động được giải phóng ngay khi biến ra khỏi phạm vi (scope) của hàm.

Đại diện: int, float, double, bool, char, struct, enum.

Reference Types (Kiểu tham chiếu):

Cơ chế: Lưu trữ địa chỉ (tham chiếu) trỏ tới vị trí bộ nhớ chứa dữ liệu thực sự, không lưu trực tiếp dữ liệu.

Vùng nhớ: Dữ liệu thực sự được cấp phát trên bộ nhớ Heap. Biến tham chiếu (chứa địa chỉ bộ nhớ) thì vẫn nằm trên Stack.

Giải phóng: Phụ thuộc vào Garbage Collector (GC). Khi không còn biến nào trỏ tới vùng nhớ trên Heap, GC sẽ dọn dẹp nó trong tương lai, gây tốn tài nguyên xử lý hơn so với Stack.

Đại diện: class, string, array, delegate, object, interface.

Câu 2: Init-only Properties (init) so với setTính năng init (ra mắt từ C# 9) giải quyết bài toán tạo ra các đối tượng bất biến (immutable) nhưng vẫn có cú pháp khởi tạo tiện lợi.Đặc điểmTừ khóa setTừ khóa initKhả năng gán giá trịCó thể gán lại giá trị ở bất kỳ đâu, bất kỳ lúc nào trong vòng đời đối tượng.Chỉ được phép gán giá trị duy nhất một lần trong quá trình khởi tạo (Constructor hoặc Object Initializer).Tính bất biếnKhông đảm bảo (Mutable).Đảm bảo tính bất biến (Immutable). Sau khi khởi tạo xong, thuộc tính trở thành Read-only.

Câu 3: virtual và override trong Đa hình
Hai từ khóa này là cặp bài trùng để triển khai tính Đa hình động (Dynamic Polymorphism) tại thời điểm chạy (Runtime):

virtual (Nằm ở Lớp Cha):

Được sử dụng khi khai báo một phương thức ở lớp cơ sở (Base class). Nó báo hiệu cho trình biên dịch rằng: "Phương thức này có một hành vi mặc định, nhưng tôi cho phép các lớp con thay đổi hành vi này nếu chúng muốn".

override (Nằm ở Lớp Con):

Được sử dụng ở lớp kế thừa (Derived class) để ghi đè lại phương thức virtual (hoặc abstract) của lớp cha. Nó cung cấp một logic hoàn toàn mới hoặc bổ sung thêm logic cho phương thức cũ.

Sự khác biệt khi thực thi: Khi bạn gọi một phương thức thông qua con trỏ kiểu Lớp Cha đang trỏ tới đối tượng Lớp Con, C# sẽ tự động dò tìm xem phương thức đó có bị override không. Nếu có, nó sẽ chạy code của Lớp Con (Late Binding).

Câu 4: Truy xuất thành phần static qua thể hiện (Object)
Một thành phần khai báo là static (biến, phương thức) thuộc về chính bản thân Lớp (Class) đó, chứ không thuộc về bất kỳ một phiên bản (Object Instance) cụ thể nào.

C# không cho phép dùng toán tử new tạo object rồi gọi thành viên static (như myObject.MyStaticMethod()) vì:

Bản chất lưu trữ: Thành phần static chỉ được cấp phát bộ nhớ duy nhất một lần trong vùng nhớ đặc biệt (High-Frequency Heap) khi ứng dụng nạp Class đó. Tất cả các object đều dùng chung vùng nhớ này.

Thiết kế ngôn ngữ chặt chẽ: C# ép buộc bạn phải gọi qua tên Lớp (ví dụ: ClassName.MyStaticMethod()) để mã nguồn phản ánh đúng ý nghĩa logic. Nếu cho phép gọi qua myObject, lập trình viên có thể hiểu nhầm rằng họ đang tương tác với một trạng thái riêng của object đó, gây ra lỗi logic nghiêm trọng khi trạng thái này bị thay đổi và ảnh hưởng chéo đến toàn bộ ứng dụng. (Lưu ý: Ngôn ngữ Java có cho phép gọi static qua object kèm cảnh báo, nhưng C# cấm hoàn toàn ở mức compiler để bảo vệ tính tường minh).

