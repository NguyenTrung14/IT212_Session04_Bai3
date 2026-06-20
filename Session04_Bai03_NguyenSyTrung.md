# BÀI 3: Đọc hiểu & Dò lỗi qua Prompt

## Phát hiện lỗi logic lặp

### 1. Vì sao prompt thô dễ khiến AI bỏ sót lỗi?

Prompt **"Mã này bị lỗi gì?"** quá chung chung vì không cung cấp mục tiêu của hàm, hành vi mong đợi, kết quả thực tế hoặc ca kiểm thử làm bằng chứng. AI không có tiêu chí rõ ràng để phân biệt giữa lỗi cú pháp, lỗi biên dịch, lỗi thời gian chạy, lỗi hiệu năng và lỗi logic.

Đoạn mã `DuplicateFinder` hoàn toàn hợp lệ về cú pháp và có thể biên dịch bình thường. Trình biên dịch chỉ kiểm tra các quy tắc của ngôn ngữ như kiểu dữ liệu, tên biến và cấu trúc câu lệnh; nó không biết rằng lập trình viên muốn so sánh hai **vị trí khác nhau** trong mảng. Vì thế, vòng lặp `for (int j = i; ...)` không tạo cảnh báo biên dịch mặc dù gây sai nghiệp vụ.

Ngay ở lần lặp đầu tiên, `i = 0` và `j = 0`, điều kiện trở thành `arr[0] == arr[0]`. Biểu thức này luôn đúng, nên hàm lập tức trả về `arr[0]`. Ví dụ, với mảng `{1, 2, 3, 4}` không có phần tử trùng lặp, hàm vẫn trả về `1` thay vì `null`.

Nếu không đưa ca kiểm thử cụ thể vào prompt, AI có thể chỉ nhận xét về style, khả năng `arr` bị `null`, độ phức tạp `O(N²)` hoặc thậm chí kết luận code đúng. Prompt thô cũng không yêu cầu truy vết từng vòng lặp và không quy định cách sửa, nên đầu ra dễ thiếu trọng tâm hoặc chỉ sửa `j = i + 1` mà bỏ qua yêu cầu tối ưu hiệu năng.

### 2. Prompt tối ưu

```text
[VAI TRÒ]
Bạn là một Code Auditor chuyên review mã Java, phát hiện lỗi logic, thiết kế
ca kiểm thử và phân tích độ phức tạp thuật toán.

[MỤC TIÊU]
Hãy kiểm toán hàm findDuplicate dưới đây. Hàm phải trả về phần tử đầu tiên
được gặp lại khi duyệt mảng từ trái sang phải; nếu không có phần tử trùng
lặp thì trả về null.

[NGỮ CẢNH]
Mã nguồn hiện tại:

public class DuplicateFinder {
    public static Integer findDuplicate(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            for (int j = i; j < arr.length; j++) {
                if (arr[i] == arr[j]) {
                    return arr[i];
                }
            }
        }

        return null;
    }
}

Ca kiểm thử chứng minh lỗi:
- Input:  {1, 2, 3, 4}
- Kết quả mong đợi: null
- Kết quả hiện tại: 1

[RÀNG BUỘC]
1. Mô phỏng các giá trị i, j và phép so sánh ở lần lặp đầu tiên để xác định
   nguyên nhân gốc rễ.
2. Giải thích vì sao j bắt đầu từ i làm arr[i] tự so sánh với chính nó.
3. Không chỉ sửa thành j = i + 1. Hãy thay thuật toán hai vòng lặp bằng
   HashSet để độ phức tạp thời gian trung bình giảm từ O(N^2) xuống O(N).
4. Duyệt mảng từ trái sang phải. Dùng kết quả của Set.add(): nếu add trả về
   false thì phần tử đã xuất hiện và phải được trả về ngay.
5. Nếu arr là null hoặc rỗng thì trả về null.
6. Chỉ sử dụng thư viện chuẩn Java và giữ nguyên chữ ký phương thức:
   public static Integer findDuplicate(int[] arr).
7. Phân tích cả độ phức tạp thời gian và bộ nhớ của phiên bản mới.
8. Kiểm tra kết quả với ít nhất các mảng {1, 2, 3, 4}, {1, 2, 3, 2, 1}
   và {5, 5}.

[ĐỊNH DẠNG ĐẦU RA]
Trả lời bằng tiếng Việt theo thứ tự:
1. Lỗi logic và kết quả mô phỏng.
2. Mã nguồn Java hoàn chỉnh trong một khối code markdown.
3. Kết quả các ca kiểm thử.
4. Phân tích độ phức tạp trước và sau khi sửa.
```

### 3. Mã nguồn Java đã sửa bằng HashSet

```java
import java.util.HashSet;
import java.util.Set;

public class DuplicateFinder {

    public static Integer findDuplicate(int[] arr) {
        if (arr == null || arr.length == 0) {
            return null;
        }

        Set<Integer> seen = new HashSet<>();

        for (int value : arr) {
            if (!seen.add(value)) {
                return value;
            }
        }

        return null;
    }
}
```

### 4. Kiểm chứng và độ phức tạp

| Mảng đầu vào | Kết quả | Giải thích |
|---|---:|---|
| `{1, 2, 3, 4}` | `null` | Không có giá trị nào được thêm vào `HashSet` lần thứ hai. |
| `{1, 2, 3, 2, 1}` | `2` | `2` là phần tử đầu tiên được gặp lại khi duyệt từ trái sang phải. |
| `{5, 5}` | `5` | Lần thêm `5` thứ hai làm `seen.add(5)` trả về `false`. |
| `null` hoặc `{}` | `null` | Được xử lý ngay bởi điều kiện bảo vệ. |

Phiên bản cũ sử dụng hai vòng lặp nên có độ phức tạp thời gian `O(N²)` trong trường hợp tổng quát và `O(1)` bộ nhớ phụ. Phiên bản mới chỉ duyệt mảng một lần; thao tác thêm và tra cứu của `HashSet` có thời gian trung bình `O(1)`, do đó tổng thời gian trung bình là `O(N)`. Đổi lại, tập `seen` có thể chứa tối đa `N` phần tử nên độ phức tạp bộ nhớ là `O(N)`.
