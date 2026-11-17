# BÀI 03: PHẦN C — SHINGLING VÀ CHỮ KÝ MINHASH (TÌM TƯƠNG ĐỒNG VĂN BẢN DỮ LIỆU LỚN)

---

## 1. Vấn đề thực tế: Làm sao phát hiện đạo văn hoặc trang web trùng lặp?

Hãy tưởng tượng bạn là một kỹ sư tại Google hoặc một giảng viên cần chấm hàng nghìn bài luận:
- Người ta copy một bài viết, sau đó họ sửa một vài từ, đổi thứ tự vài câu, chèn thêm vài khoảng trắng hoặc đổi chữ hoa chữ thường.
- Nếu bạn dùng phép so sánh chuỗi chính xác (`văn_bản_A == văn_bản_B`), hai bài sẽ bị coi là hoàn toàn khác nhau!
- Nếu bạn so sánh từng từ, với hàng triệu văn bản, số lượng phép so sánh sẽ bùng nổ lên tới hàng nghìn tỷ phép tính, làm sập hệ thống máy chủ.

Để giải quyết bài toán này, các nhà khoa học dữ liệu sử dụng bộ ba giải thuật kinh điển:
1. **Chuẩn hóa & K-Shingling:** Chuyển văn bản thành một tập hợp các chuỗi con.
2. **Độ đo tương đồng Jaccard:** Đo mức độ trùng lặp giữa hai tập hợp.
3. **Chữ ký MinHash:** Nén tập hợp khổng lồ thành một vector nhỏ gồm **128 con số** để so sánh cực nhanh!

---

## 2. Bước 1: Chuẩn hóa văn bản (`normalize_text`)

Trước khi so sánh, văn bản cần được đưa về một quy chuẩn chung:
- Ký tự viết hoa `A`–`Z` được chuyển thành viết thường `a`–`z`.
- **Lưu ý đặc biệt:** Không đổi chữ có dấu Unicode (ví dụ `É` vẫn giữ nguyên là `É`, không chuyển thành `é`).
- Tất cả các ký tự khoảng trắng thừa (phím cách, tab `\t`, xuống dòng `\n`, `\r`,...) được gộp lại thành đúng **1 dấu cách** duy nhất.
- Cắt bỏ khoảng trắng ở đầu và cuối bài.

```
Văn bản gốc:  "  Hello    World!  \n  This is    Wikispeedia. "
Sau chuẩn hóa: "hello world! this is wikispeedia."
```

---

## 3. Bước 2: Tạo tập Shingle 9 ký tự (`shingles`)

Một **$k$-shingle** (hoặc $k$-gram) là một chuỗi gồm **$k$ ký tự liên tiếp** trích xuất từ văn bản bằng kỹ thuật **cửa sổ trượt** (*sliding window*).

Trong bài tập này, đề bài quy định **$k = 9$ ký tự Unicode** liên tiếp.

### Ví dụ trực quan với chuỗi ngắn `"chó con ngủ"` và $k = 4$:
Ta đặt một khung kính 4 ký tự và trượt từ đầu đến cuối:
1. `[chó ]`
2. `[hó c]`
3. `[ó co]`
4. `[ con]`
5. `[con ]`
6. `[on n]`
7. `[n ng]`
8. `[ ngủ]`

Mỗi bài viết Wikipedia sẽ được đại diện bởi một **tập hợp (`set`) các shingle 9 ký tự phân biệt**. Nếu một shingle xuất hiện nhiều lần trong bài, ta cũng chỉ giữ lại 1 bản sao.

---

## 4. Độ tương đồng Jaccard (Jaccard Similarity)

Sau khi chuyển hai bài viết $A$ và $B$ thành hai tập hợp shingle $S_A$ và $S_B$, ta tính độ tương đồng giữa chúng bằng **Hệ số Jaccard**:

$$J(A, B) = \frac{|S_A \cap S_B|}{|S_A \cup S_B|} = \frac{\text{Số lượng shingle trùng nhau}}{\text{Tổng số lượng shingle phân biệt của cả hai bài}}$$

```
[       Tập A       ]  [       Tập B       ]
         \             /
          \   GIAO    /
           [ Trùng ]
------------------------------------------------
   Jaccard = |GIAO| / |HỢP TOÀN BỘ HAI TẬP|
```

- $J(A, B) = 1.0$: Hai bài viết có tập shingle giống hệt nhau $100\%$.
- $J(A, B) = 0.0$: Hai bài viết hoàn toàn không có shingle nào chung.
- **Quy ước biên:**
  - Nếu cả 2 bài đều rỗng: $J = 1.0$.
  - Nếu đúng 1 bài rỗng: $J = 0.0$.

---

## 5. Bước 3: Mã hóa Shingle sang số nguyên (`shingle_id`)

Mỗi shingle là một chuỗi 9 ký tự (tốn khoảng 9–18 byte bộ nhớ). Một bài viết có thể có tới $30,000$ shingle, khiến việc lưu trữ và so sánh tập hợp chuỗi rất tốn RAM.

Ta chuyển mỗi shingle thành một số nguyên 32-bit:
1. Tính mã băm **SHA-256** của chuỗi byte UTF-8.
2. Cắt lấy **4 byte đầu tiên**.
3. Đọc 4 byte đó thành số nguyên không dấu big-endian.
4. Lấy phần dư khi chia cho số nguyên tố Mersenne $p = 2^{31} - 1 = 2147483647$.

Kết quả: Mỗi shingle biến thành một con số nguyên nằm trong khoảng $[0, 2147483646]$.

---

## 6. Bước 4: Chữ ký MinHash (MinHash Signature)

Đây là phát minh đoạt giải thưởng lớn của nhà khoa học Andrei Broder (phát minh khi làm việc tại Altavista/Compaq):

### 6.1. Ý tưởng cốt lõi
Giả sử ta có 128 hàm băm ngẫu nhiên dạng affine:
$$h_j(x) = (a_j \cdot x + b_j) \bmod p, \quad j = 0, 1, \dots, 127$$
(128 cặp số $a_j, b_j$ đã được cho sẵn trong `minhash_params.json`).

Với một bài viết $D$, ta áp dụng hàm băm $h_j$ lên tất cả các mã shingle của bài viết đó, rồi **chỉ chọn giữ lại con số NHỎ NHẤT (giá trị min)**:
$$\sigma_j(D) = \min_{x \in S(D)} h_j(x)$$

Làm như vậy với 128 hàm băm, ta thu được một **vector gồm đúng 128 con số nguyên**, gọi là **Chữ ký MinHash (MinHash Signature)** của bài viết!

### 6.2. Định lý MinHash kỳ diệu
> **Xác suất để giá trị MinHash của hai bài viết A và B trùng nhau đúng bằng Hệ số tương đồng Jaccard thực tế giữa chúng!**
> $$P(\sigma_j(A) == \sigma_j(B)) = J(A, B)$$

Do đó, để ước lượng xem hai bài viết có tương đồng hay không:
- Thay vì phải giữ lại hàng chục nghìn chuỗi shingle và tính phép giao/hợp tập hợp cực kỳ chậm...
- Ta chỉ cần đếm xem **trong 128 con số của hai chữ ký có bao nhiêu vị trí bằng nhau**, rồi chia cho 128!

$$\hat{J}(A, B) = \frac{\text{Số vị trí có } \sigma_j(A) == \sigma_j(B)}{128}$$

*Ví dụ:* Nếu hai bài viết có 64 trong số 128 số giống nhau $\implies$ Độ tương đồng ước lượng $\hat{J} = \frac{64}{128} = 0.5$ ($50\%$).

---

## 7. Giải thích code trong `submission.py`

### 7.1. Chuẩn hóa chuỗi và tách shingle
```python
def normalize_text(text: str) -> str:
    # 1. Chỉ đổi ASCII A-Z thành a-z (giữ nguyên Unicode)
    ascii_lower = text.translate(str.maketrans(
        "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "abcdefghijklmnopqrstuvwxyz"))
    # 2. Gộp mọi khoảng trắng liên tiếp thành 1 dấu cách và strip hai đầu
    return re.sub(r"[ \t\n\r\v\f]+", " ", ascii_lower).strip(" ")

def shingles(text: str, k: int) -> set[str]:
    if not isinstance(k, int) or isinstance(k, bool) or k <= 0:
        raise ValueError("k phải là số nguyên dương")
    # Cắt cửa sổ trượt k ký tự
    return {text[i:i + k] for i in range(len(text) - k + 1)}
```

### 7.2. Băm Shingle và Tính MinHash
```python
def shingle_id(shingle: str, p: int) -> int:
    if p <= 1:
        raise ValueError("p phải lớn hơn 1")
    digest = hashlib.sha256(shingle.encode("utf-8")).digest()
    # Lấy 4 byte đầu, chuyển thành số nguyên big-endian, rồi modulo p
    return int.from_bytes(digest[:4], "big") % p

def minhash(shingle_ids: set[int], hash_params: dict) -> list[int]:
    p = hash_params["p"]
    functions = hash_params["hash_functions"]

    # Trường hợp tập rỗng: Trả về vector gồm 128 số p
    if not shingle_ids:
        return [p] * len(functions)

    # Tối ưu hóa bằng NumPy (xử lý vector hóa cực nhanh cho hàng nghìn số)
    if np is not None:
        values = np.fromiter(shingle_ids, dtype=np.int64, count=len(shingle_ids))
        return [int(np.min((f["a"] * values + f["b"]) % p)) for f in functions]

    # Fallback cho Python thuần nếu không có NumPy
    return [min((f["a"] * x + f["b"]) % p for x in shingle_ids) for f in functions]
```

### 7.3. Tính Jaccard chính xác và Ước lượng qua chữ ký
```python
def jaccard(set_a: set[str], set_b: set[str]) -> float:
    # Kích thước tập hợp = |A| + |B| - |A ∩ B|
    union_size = len(set_a) + len(set_b) - len(set_a & set_b)
    # Nếu cả 2 tập đều rỗng (union_size == 0) trả về 1.0
    return len(set_a & set_b) / union_size if union_size else 1.0

def signature_similarity(
    sig_a: list[int], sig_b: list[int], empty_a: bool, empty_b: bool
) -> float:
    if not sig_a or len(sig_a) != len(sig_b):
        raise ValueError("Chữ ký phải có cùng độ dài dương")
    # Xử lý trường hợp biên theo quy ước đề bài
    if empty_a or empty_b:
        return 1.0 if empty_a and empty_b else 0.0
    # Đếm số vị trí có giá trị bằng nhau rồi chia cho độ dài chữ ký (128)
    return sum(a == b for a, b in zip(sig_a, sig_b)) / len(sig_a)
```
