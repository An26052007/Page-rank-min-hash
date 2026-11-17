# GIẢI THÍCH CHI TIẾT HÀM `normalize_text` VÀ `shingles`
> **Phần C — Chuẩn hóa văn bản và Cắt shingle ký tự (k-shingles)**  
> **Tệp:** `submission.py` (Dòng 167–192)

---

## 1. YÊU CẦU ĐỀ BÀI VÀ CÁC CẠM BẪY TIỀM ẨN

Trong mục **4.1. Chuẩn hóa và tạo tập shingle** của [bai-tap-wikispeedia.md](file:///C:/Users/HP/Downloads/HKI_2026-2027/GI%E1%BA%A2I%20THU%E1%BA%ACT%20KHDL/wikispeedia-student-package/bai-tap-wikispeedia.md), đề bài đưa ra các tiêu chuẩn xử lý văn bản cực kỳ nghiêm ngặt:

1. **Chỉ chuyển chữ cái ASCII hoa `A`–`Z` thành thường `a`–`z`:**
   - **Cạm bẫy chết người:** Tuyệt đối **KHÔNG dùng `text.lower()`**!
   - Hàm `text.lower()` của Python được thiết kế theo chuẩn Unicode toàn năng, nó sẽ chuyển đổi cả các ký tự có dấu như `É` $\rightarrow$ `é`, `À` $\rightarrow$ `à`, `İ` $\rightarrow$ `i`, `ß` $\rightarrow$ `ss`. Đề bài yêu cầu giữ nguyên vẹn mọi ký tự Unicode khác, chỉ đổi 26 chữ cái tiếng Anh cơ bản.
2. **Gộp khoảng trắng (Whitespace collapse):**
   - Thay mỗi dãy liên tiếp gồm các ký tự thuộc tập: `U+0020` (dấu cách), `\t` (tab), `\n` (xuống dòng), `\r`, `\v`, `\f` thành đúng **một dấu cách ASCII đơn** `' '`.
   - Cắt bỏ dấu cách thừa ở đầu và cuối chuỗi (`strip`).
3. **Giữ nguyên các thành phần khác:**
   - Dấu chấm câu, chữ số, dấu gạch nối, ký tự đặc biệt, nội dung bảng biểu giữ nguyên hoàn toàn.
4. **Tạo tập $k$-shingle:**
   - Dùng cửa sổ trượt độ dài $k$ ($k=9$ ký tự Unicode) chạy dọc chuỗi văn bản.
   - Tập hợp trả về phải khử trùng lặp (trả về kiểu `set`). Nếu chuỗi ngắn hơn $k$, trả về tập rỗng `set()`.
   - Kiểm tra hợp lệ: $k$ phải là số nguyên dương ($k \ge 1$), nếu sai ném `ValueError`.

---

## 2. BẢNG THÔNG SỐ ĐẦU VÀO VÀ ĐẦU RA

### 2.1. Hàm `normalize_text`
```python
def normalize_text(text: str) -> str:
```

| Tham số / Đầu ra | Kiểu dữ liệu | Ý nghĩa thực tế | Giá trị ví dụ |
|---|---|---|---|
| **`text`** | `str` | Nội dung văn bản thô đọc từ tệp bài viết Wikispeedia. | `"  Édouard   Manet was   an ARTIST.\n "` |
| **ĐẦU RA (`->`)** | `str` | Chuỗi văn bản đã được chuẩn hóa ASCII và gộp khoảng trắng. | `"Édouard manet was an artist."` |

### 2.2. Hàm `shingles`
```python
def shingles(text: str, k: int) -> set[str]:
```

| Tham số / Đầu ra | Kiểu dữ liệu | Ý nghĩa thực tế | Giá trị ví dụ |
|---|---|---|---|
| **`text`** | `str` | Chuỗi văn bản đã chuẩn hóa. | `"Édouard manet was an artist."` |
| **`k`** | `int` | Độ dài của mỗi mảnh shingle (mặc định của đồ án là $k=9$). | `9` (hoặc các số nguyên dương khác khi kiểm thử) |
| **ĐẦU RA (`->`)** | `set[str]` | Tập hợp (Set) chứa các chuỗi con $k$ ký tự không trùng lặp. | `{"Édouard m", "douard ma", "ouard man", ...}` |

---

## 3. TOÀN BỘ MÃ NGUỒN

```python
def normalize_text(text: str) -> str:
    ascii_lower = text.translate(str.maketrans(
        "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "abcdefghijklmnopqrstuvwxyz"))
    return re.sub(r"[ \t\n\r\v\f]+", " ", ascii_lower).strip(" ")


def shingles(text: str, k: int) -> set[str]:
    if not isinstance(k, int) or isinstance(k, bool) or k <= 0:
        raise ValueError("k phải là số nguyên dương")
    return {text[i:i + k] for i in range(len(text) - k + 1)}
```

---

## 4. TỪ ĐIỂN BIẾN NỘI BỘ (LOCAL VARIABLES)

| Tên biến | Kiểu dữ liệu | Ý nghĩa & Vai trò trong thuật toán |
|---|---|---|
| **`ascii_lower`** | `str` | Chuỗi kết quả sau khi chuyển đổi 26 chữ cái ASCII hoa thành chữ thường, toàn bộ ký tự Unicode được bảo toàn 100%. |
| **`r"[ \t\n\r\v\f]+"`**| Regex Pattern | Mẫu biểu thức chính quy đại diện cho 1 hoặc nhiều ký tự khoảng trắng liên tiếp thuộc tập 6 ký tự đặc tả của đề bài. |
| **`k`** | `int` | Độ rộng cửa sổ trượt (số ký tự trong một shingle). |
| **`i`** | `int` | Chỉ số vị trí bắt đầu cắt của cửa sổ trượt, chạy từ $0$ đến $\text{len}(text) - k$. |
| **`text[i:i + k]`** | `str` | Lát cắt chuỗi gồm đúng $k$ ký tự liên tiếp kể từ vị trí $i$. |

---

## 5. GIẢI THÍCH CHI TIẾT TỪNG DÒNG CODE

### 5.1. Hàm `normalize_text`

```python
    ascii_lower = text.translate(str.maketrans(
        "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "abcdefghijklmnopqrstuvwxyz"))
```
- **`str.maketrans("ABC...", "abc...")`:** Tạo một bảng tra cứu ánh xạ 1-1 (Translation Table) gồm 26 cặp ký tự hoa $\rightarrow$ thường trong bảng chữ cái Latinh.
- **`text.translate(...)`:** Quét nhanh qua chuỗi và chỉ thay thế những ký tự có trong bảng dịch chuyển.
  - Chữ `'M'` $\rightarrow$ `'m'`.
  - Chữ `'É'` không có trong bảng $\rightarrow$ **Giữ nguyên là `'É'`**.
  - Đây là giải pháp tối ưu nhất cả về tốc độ lẫn độ chính xác so với việc duyệt từng ký tự bằng tay.

```python
    return re.sub(r"[ \t\n\r\v\f]+", " ", ascii_lower).strip(" ")
```
- **`re.sub(r"[ \t\n\r\v\f]+", " ", ascii_lower)`:**
  - Dùng module `re` (Regular Expressions) để tìm tất cả các chuỗi liên tiếp chứa dấu cách thường (` `), tab (`\t`), ngắt dòng (`\n`), về đầu dòng (`\r`), tab đứng (`\v`), sang trang (`\f`).
  - Thay thế toàn bộ cụm đó bằng đúng một dấu cách đơn `" "`.
- **`.strip(" ")`:** Cắt bỏ các dấu cách ASCII đơn lẻ ở đầu chuỗi và cuối chuỗi nếu có.

---

### 5.2. Hàm `shingles`

```python
    if not isinstance(k, int) or isinstance(k, bool) or k <= 0:
        raise ValueError("k phải là số nguyên dương")
```
- **Kiểm tra kiểu dữ liệu và giá trị của $k$:**
  - `not isinstance(k, int)`: Chặn các kiểu số thực như `k = 9.0` hoặc chuỗi ký tự `k = "9"`.
  - `isinstance(k, bool)`: Chặn giá trị boolean như `k = True` (vì trong Python `bool` kế thừa từ `int`, `True == 1`).
  - `k <= 0`: Chặn các giá trị không dương như $k = 0, -1, -5$.
  - Nếu phạm luật $\implies$ Ném ngoại lệ `ValueError`.

```python
    return {text[i:i + k] for i in range(len(text) - k + 1)}
```
- **Cơ chế cửa sổ trượt (Sliding Window):**
  - Giả sử chuỗi có độ dài $N = \text{len}(text)$.
  - Vị trí bắt đầu cắt shingle đầu tiên là $i = 0$ (lấy từ $0$ đến $k-1$).
  - Vị trí bắt đầu cắt shingle cuối cùng là $i = N - k$ (lấy từ $N - k$ đến $N-1$).
  - Do đó $i$ chạy trong khoảng: `range(0, (N - k) + 1)` tức là `range(len(text) - k + 1)`.
- **Cú pháp `{...}` (Set Comprehension):**
  - Tự động gom các chuỗi con $k$ ký tự vào một tập băm `set`.
  - Nếu trong văn bản có nhiều cụm từ trùng nhau sinh ra shingle giống nhau, cấu trúc `set` sẽ tự động loại bỏ trùng lặp, chỉ giữ lại đúng 1 bản sao.
- **Trường hợp văn bản ngắn hơn $k$ ($N < k$):**
  - Khi đó $N - k + 1 \le 0$, `range(...)` trở thành rỗng $\implies$ Hàm tự động trả về tập hợp rỗng `set()` mà không hề bị lỗi!

---

## 6. TẠI SAO LẠI LÀM NHƯ VẬY? (THIẾT KẾ KỸ THUẬT & LÝ DO CỐT LÕI)

### 1. Tại sao dùng `translate(maketrans)` thay vì `text.lower()`?
- Trong tiếng Pháp, tên họa sĩ là `Édouard Manet`.
- Nếu dùng `text.lower()`: Chữ `É` biến thành `é` $\implies$ Sai lệch với văn bản gốc của bài báo khoa học Wikispeedia.
- Trong tiếng Đức, chữ `ß` (Eszett) khi gọi `lower()` hoặc `casefold()` có thể biến dạng; trong tiếng Thổ Nhĩ Kỳ chữ `I` và `İ` có quy tắc biến đổi riêng biệt.
- Sử dụng `str.maketrans` chỉ giới hạn phạm vi tác động đúng trên 26 ký tự ASCII thuần túy từ `'A'` đến `'Z'`, bảo toàn tuyệt đối 100% tính nguyên bản của mọi ngôn ngữ quốc tế khác.

### 2. Tại sao dùng biểu thức `r"[ \t\n\r\v\f]+"` thay vì `r"\s+"`?
- Trong Regex của Python 3, ký hiệu `\s` đại diện cho **tất cả các khoảng trắng trong chuẩn Unicode**, bao gồm cả dấu cách không ngắt dòng (Non-breaking space `\u00A0`), khoảng trắng mỏng (Thin space `\u2009`), em-space (`\u2003`),...
- Đề bài trong tài liệu quy định rõ ràng: chỉ gộp các ký tự thuộc tập: `U+0020`, `\t`, `\n`, `\r`, `\v`, `\f`.
- Việc chỉ định tường minh `[ \t\n\r\v\f]+` giúp chương trình tuân thủ chính xác từng ký tự theo đúng yêu cầu đề bài, tránh tác động phụ ngoài ý muốn lên các ký tự Unicode đặc thù.

### 3. Tại sao phải có `isinstance(k, bool)` dù đã có `isinstance(k, int)`?
- Trong ngôn ngữ Python, kiểu dữ liệu `bool` là một lớp con (subclass) trực tiếp của `int`:
  ```python
  issubclass(bool, int) == True
  isinstance(True, int) == True
  ```
- Nếu chỉ kiểm tra `if not isinstance(k, int):`, khi ai đó truyền `k = True` thì Python vẫn cho qua (vì `True` là `1`).
- Thêm điều kiện `or isinstance(k, bool)` là kỹ thuật phòng vệ mã nguồn (defensive programming) chuẩn mực để loại trừ hoàn toàn kiểu boolean.

### 4. Tại sao lại dùng shingle 9 ký tự (`k=9`)?
- Các từ tiếng Anh trung bình dài 5–6 ký tự.
- Cửa sổ 9 ký tự sẽ bắt được trọn vẹn mối liên kết giữa 2 từ liên tiếp hoặc 1 từ kèm theo ngữ cảnh (ví dụ: `"the apple"`, `"artist was"`).
- Nếu $k$ quá nhỏ ($k=2, 3$): Hầu như mọi văn bản đều có shingle giống nhau, không phân biệt được nội dung.
- Nếu $k$ quá lớn ($k=50$): Chỉ cần sai lệch một dấu phẩy là hai văn bản đã không còn shingle nào trùng khớp. $k=9$ là giá trị tối ưu được chứng minh thực nghiệm trong khai phá dữ liệu web.

---

## 7. VÍ DỤ SỐ HỌC VÀ CHUỖI VĂN BẢN TRỰC QUAN

Giả sử văn bản gốc là:
```text
"  Édouard   Manet was   an ARTIST.\n "
```

### Bước 1: Qua hàm `normalize_text`
- Ký tự `É` giữ nguyên là `É`.
- Các chữ `M`, `A`, `R`, `T`, `I`, `S`, `T` đổi thành chữ thường.
- Dãy 3 dấu cách `   `, ký tự ngắt dòng `\n` và khoảng trắng 2 đầu bị thu gọn.
- **Chuỗi sau chuẩn hóa:**
  ```text
  "Édouard manet was an artist."
  ```
  *(Độ dài $N = 28$ ký tự)*

### Bước 2: Qua hàm `shingles(text, k=9)`
- Số lượng vị trí trượt: $N - k + 1 = 28 - 9 + 1 = 20$ vị trí ($i = 0, 1, \dots, 19$).
- Các mảnh shingle được sinh ra:
  - $i=0$: `"Édouard m"`
  - $i=1$: `"douard ma"`
  - $i=2$: `"ouard man"`
  - $i=3$: `"uard mane"`
  - ...
  - $i=19$: `"n artist."`
- Kết quả trả về là một `set` gồm 20 chuỗi con độc nhất.
