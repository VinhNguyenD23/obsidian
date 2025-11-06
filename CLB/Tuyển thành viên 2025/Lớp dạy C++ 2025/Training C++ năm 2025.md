### Ngày 1: Nhập Môn và Cấu Trúc Cơ Bản 

| **Thời gian**   | **Nội dung bài giảng (60 phút)**                                                                       | **Code Ví dụ & Bài Tập (30 phút)**                                  |
| --------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| **0 - 60 phút** | **1. Cài đặt & Môi trường:** Giới thiệu C++, cài đặt IDE (VS Code/CLion/Dev-C++), Biên dịch (GCC/G++). | **Cài đặt IDE.** Viết chương trình đầu tiên: **Hello World.**       |
|                 | **2. Cấu trúc chương trình C++:** `#include`, `main()`, `std::cout`, `std::cin`.                       | **Ví dụ:** In ra tên, nhập và in ra một số nguyên.                  |
|                 | **3. Biến & Kiểu dữ liệu:** `int`, `long long`, `double`, `char`, `bool`. Khai báo, gán giá trị.       | **Ví dụ:** Tính tổng, hiệu, tích, thương hai số nguyên và làm tròn. |
|                 | **4. Phép toán:** Toán tử số học, quan hệ, logic.                                                      | **Bài tập:** Kiểm tra một số là chẵn hay lẻ.                        |
|                 | 5. **Nói lại git/github**                                                                              | ...                                                                 |
|                 |                                                                                                        |                                                                     |

---

### Ngày 2: Điều Khiển Luồng và Mảng Cơ Bản 

| **Thời gian**   | **Nội dung bài giảng (60 phút)**                                                                 | **Code Ví dụ & Bài Tập (30 phút)**                          |
| --------------- | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| **0 - 60 phút** | **1. Câu lệnh điều kiện:** `if`, `else`, `else if`, toán tử 3 ngôi.                              | **Ví dụ:** Giải phương trình bậc nhất, xác định học lực.    |
|                 | **2. Vòng lặp:** `for`, `while`, `do-while`. `break`, `continue`.                                | **Ví dụ:** In bảng cửu chương, tính tổng các số từ 1 đến N. |
|                 | **3. Mảng (Array):** Khai báo, truy cập, duyệt mảng 1 chiều.                                     | **Ví dụ:** Nhập và in ra các phần tử của một mảng.          |
|                 | **4. Chuỗi ký tự cơ bản (C-style string):** Khái niệm, đọc/ghi. (Sẽ học `std::string` ở Ngày 3). | **Bài tập:** Tìm giá trị lớn nhất/nhỏ nhất trong mảng.      |

---

- [x] ### Ngày 3: Thư Viện Chuẩn STL (Phần 1) - Cơ sở cho Contest 

| **Thời gian**   | **Nội dung bài giảng (60 phút)**                                                         | **Code Ví dụ & Bài Tập (30 phút)**                                                                 |
| --------------- | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **0 - 60 phút** | **1. Thư viện STL:** Giới thiệu vai trò quan trọng trong lập trình cạnh tranh.           | **Ví dụ:** Dùng `std::vector` thay cho mảng tĩnh.                                                  |
|                 | **2. Vector (`std::vector`):** Khai báo, kích thước, `push_back`, truy cập, duyệt.       | **Ví dụ:** Đọc một lượng dữ liệu không xác định và lưu vào vector.                                 |
|                 | **3. Chuỗi (`std::string`):** Khai báo, các thao tác cơ bản (nối, tìm kiếm, kích thước). | **Ví dụ:** Đếm số lần xuất hiện của một ký tự trong chuỗi.                                         |
|                 | **4. Hàm (`Function`):** Khái niệm, khai báo, định nghĩa, truyền tham số (giá trị).      | **Bài tập:** Viết hàm kiểm tra số nguyên tố, sau đó dùng vector để lưu các số nguyên tố nhỏ hơn N. |

---

### Ngày 4: Thư Viện Chuẩn STL (Phần 2) - Cấu Trúc Dữ Liệu 

| **Thời gian**   | **Nội dung bài giảng (60 phút)**                                                                            | **Code Ví dụ & Bài Tập (30 phút)**                           |
| --------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| **0 - 60 phút** | **1. Đọc/Ghi nhanh:** Tối ưu I/O (`ios::sync_with_stdio(false); cin.tie(NULL);`).                           | **Ví dụ:** So sánh tốc độ đọc dữ liệu lớn.                   |
|                 | **2. Set (`std::set`):** Khái niệm, ưu điểm (duy nhất, tự sắp xếp), các thao tác cơ bản (`insert`, `find`). | **Ví dụ:** Đếm số lượng phần tử khác nhau trong một dãy số.  |
|                 | **3. Map (`std::map`):** Khái niệm, cặp Key-Value, các thao tác cơ bản.                                     | **Ví dụ:** Đếm tần suất xuất hiện của các từ trong một câu.  |
|                 | **4. Sort (`std::sort`):** Giới thiệu và cách sử dụng hàm sắp xếp (trên mảng, vector) và custom so sánh.    | **Bài tập:** Sắp xếp một vector các số theo thứ tự giảm dần. |

---

### Ngày 5: Con Trỏ, Cấu Trúc (Struct) và Kiểu Dữ Liệu Tự Định Nghĩa 

|**Thời gian**|**Nội dung bài giảng (60 phút)**|**Code Ví dụ & Bài Tập (30 phút)**|
|---|---|---|
|**0 - 60 phút**|**1. Con Trỏ (`Pointer`):** Khái niệm, toán tử `*` (dereference) và `&` (address of).|**Ví dụ:** Hoán đổi giá trị hai biến dùng con trỏ.|
||**2. Truyền Tham Biến:** Truyền tham số bằng tham chiếu (`&`) và con trỏ. (Quan trọng cho hiệu năng).|**Ví dụ:** Viết lại hàm hoán đổi bằng tham chiếu.|
||**3. Cấu trúc (`Struct`):** Khai báo, truy cập thành viên (`.`).|**Ví dụ:** Định nghĩa cấu trúc `Student` (ID, Name, GPA).|
||**4. Vector của Struct:** Lưu trữ đối tượng.|**Bài tập:** Nhập danh sách sinh viên, sắp xếp theo GPA.|

---

### Ngày 6: Kỹ Thuật Lập Trình Cơ Bản (Giải Thuật) 

| **Thời gian**   | **Nội dung bài giảng (60 phút)**                                                                                       | **Code Ví dụ & Bài Tập (30 phút)**                                                        |
| --------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **0 - 60 phút** | **1. Độ phức tạp thời gian (Time Complexity):** Khái niệm $O(N)$, $O(N^2)$, $O(\log N)$. Giới hạn thời gian (TL).      | **Ví dụ:** Phân tích độ phức tạp của thuật toán tìm kiếm tuyến tính vs tìm kiếm nhị phân. |
|                 | **2. Đệ Quy (Recursion):** Khái niệm, trường hợp cơ sở.                                                                | **Ví dụ:** Hàm tính giai thừa, số Fibonacci bằng đệ quy.                                  |
|                 | **3. Thuật toán tìm kiếm:** Tìm kiếm tuyến tính, **Tìm kiếm nhị phân (Binary Search)** (rất quan trọng trong Contest). | **Ví dụ:** Tìm kiếm nhị phân trên một mảng đã sắp xếp.                                    |
|                 | **4. Thuật toán Sàng Eratosthenes (Sàng Nguyên Tố).**                                                                  | **Bài tập:** Tìm kiếm nhị phân để giải một bài toán tìm kiếm giá trị tối ưu.              |

---

### Ngày 7: Giải Thuật Nâng Cao và Kỹ Thuật Contest 

|**Thời gian**|**Nội dung bài giảng (60 phút)**|**Code Ví dụ & Bài Tập (30 phút)**|
|---|---|---|
|**0 - 60 phút**|**1. Kỹ thuật Tham Lam (Greedy):** Nguyên tắc, ví dụ.|**Ví dụ:** Bài toán đổi tiền, sắp xếp công việc.|
||**2. Quy Hoạch Động (DP) Cơ Bản:** Giới thiệu tư duy DP (lưu kết quả đã tính).|**Ví dụ:** Bài toán tính số Fibonacci bằng DP (khử đệ quy).|
||**3. Đồ Thị Cơ Bản (Khái niệm):** Đỉnh, cạnh, ma trận kề/danh sách kề (chỉ giới thiệu).|**Ví dụ:** Dùng `std::vector<std::vector<int>>` để biểu diễn đồ thị.|
||**4. Debug và Kiểm tra lỗi:** Sử dụng trình gỡ lỗi (debugger) của IDE.|**Bài tập:** Giải một bài toán Tham Lam cơ bản.|

---

### Ngày 8: Luyện Tập Contest 

| **Thời gian**   | **Nội dung bài giảng (60 phút)**                                                                                       | **Code Ví dụ & Bài Tập (30 phút)**                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **0 - 60 phút** | **1. Review tổng quan:** Các điểm cốt lõi C++, STL, độ phức tạp.                                                       | **Làm 2-3 bài Contest mẫu** (dễ, tập trung vào I/O, `vector`, `sort`, `map`, `if/for`). |
|                 | **2. Format Contest ICPC:** Quy tắc, cách đọc đề (Input/Output/Constraints), Nộp bài.                                  | **Phân tích:** Đọc đề và phân tích **Constraint** để chọn thuật toán.                   |
|                 | **3. Các lỗi thường gặp trong Contest:** Lỗi tràn số (`int` vs `long long`), lỗi Index, lỗi Time Limit Exceeded (TLE). | **Debug** các lỗi mẫu.                                                                  |
|                 | **4. Chiến lược làm bài Contest:** Đọc tất cả đề, chọn bài dễ nhất, quản lý thời gian.                                 | **Luyện Tập Contest** (Xem phần tiếp theo).                                             |
