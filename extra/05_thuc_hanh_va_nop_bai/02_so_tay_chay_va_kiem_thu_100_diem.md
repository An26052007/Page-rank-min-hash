# BÀI 05: HƯỚNG DẪN CHI TIẾT CÁC BƯỚC CHẠY VÀ KIỂM THỬ KẾT QUẢ
> **Sổ tay thực hành từ A-Z dành cho sinh viên**  
> **Môi trường thực hiện:** Windows PowerShell / Command Prompt / Terminal

---

## 🧭 SƠ ĐỒ QUY TRÌNH THỰC HIỆN

```
[ Bước 1: Mở Terminal & Kiểm tra Python ]
                   │
                   ▼
[ Bước 2: Kiểm tra giải nén Dữ liệu ]
                   │
                   ▼
[ Bước 3: Chạy thử nhanh với mẫu nhỏ (4 cặp) ]
                   │
                   ▼
[ Bước 4: Chạy chính thức toàn bộ kho dữ liệu (Sinh 4 CSV) ]
                   │
                   ▼
[ Bước 5: Chạy tự chấm autograder.py (Mục tiêu 100/100) ]
                   │
                   ▼
[ Bước 6: Soát lỗi thủ công & Kiểm định tính hợp lý của kết quả ]
```

---

## BƯỚC 1: MỞ TERMINAL VÀ KIỂM TRA MÔI TRƯỜNG

1. **Mở thư mục bài tập:**
   - Mở cửa sổ **File Explorer** tới:  
     `C:\Users\HP\Downloads\HKI_2026-2027\GIẢI THUẬT KHDL\wikispeedia-student-package`
   - Nhấn giữ phím `Shift` + nhấp chuột phải vào vùng trống $\to$ chọn **Open PowerShell window here** (hoặc gõ `powershell` vào thanh địa chỉ thư mục rồi nhấn `Enter`).

2. **Kiểm tra phiên bản Python:**
   Gõ lệnh sau vào PowerShell:
   ```powershell
   python --version
   ```
   *Yêu cầu:* Phải hiển thị `Python 3.10.x` trở lên (ví dụ: Python 3.11, Python 3.12).  
   *(Nếu máy báo lỗi không tìm thấy lệnh, hãy thử `python3 --version` hoặc `py --version`).*

3. **Kiểm tra thư viện NumPy:**
   ```powershell
   python -c "import numpy; print('NumPy version:', numpy.__version__)"
   ```
   - Nếu hiển thị phiên bản (ví dụ `NumPy version: 2.3.5` hoặc `1.26.x`) $\implies$ Rất tốt, MinHash sẽ chạy với tốc độ tối đa!
   - Nếu báo lỗi `No module named 'numpy'`, bạn có thể cài đặt bằng:
     ```powershell
     pip install numpy
     ```
     *(Lưu ý: Code vẫn chạy được bình thường bằng thư viện chuẩn Python nếu không có NumPy, chỉ mất thêm khoảng 1-2 phút khi tính MinHash).*

---

## BƯỚC 2: KIỂM TRA VÀ GIẢI NÉN DỮ LIỆU

Trong thư mục `data/` có hai tệp nén lớn:
- `wikispeedia_paths-and-graph.tar.gz` (chứa đồ thị và liên kết).
- `wikispeedia_articles_plaintext.tar.gz` (chứa toàn bộ bài viết).

### 2.1. Lệnh giải nén (Chỉ cần làm một lần duy nhất)
Chạy 2 lệnh sau:
```powershell
tar -xzf data/wikispeedia_paths-and-graph.tar.gz -C data
tar -xzf data/wikispeedia_articles_plaintext.tar.gz -C data
```

### 2.2. Kiểm tra xem giải nén thành công chưa
Chạy lệnh kiểm tra thư mục:
```powershell
Get-ChildItem -Path data
```
Bạn phải nhìn thấy **đủ 2 thư mục con mới xuất hiện**:
- `data/wikispeedia_paths-and-graph/` (bên trong có `articles.tsv`, `links.tsv`, `categories.tsv`).
- `data/plaintext_articles/` (bên trong có **4,604 tệp `.txt`**).

> [!TIP]
> **Tại sao bắt buộc phải giải nén?**  
> Dù bộ chấm `autograder.py` có thể đọc trực tiếp từ tệp nén `.tar.gz`, nhưng mỗi lần đọc một bài viết nó lại phải quét qua tệp 35MB, khiến bộ chấm bị chậm gấp 20 lần và dẫn đến lỗi `TimeoutError: Vượt 180 giây`. Khi đã giải nén ra đĩa, mọi thao tác đọc tệp chỉ mất tích tắc!

---

## BƯỚC 3: CHẠY THỬ NHANH VỚI MẪU NHỎ (SMOKE TEST)

Trước khi chạy toàn bộ 4,604 bài viết, ta nên chạy thử nghiệm nhanh với bộ dữ liệu mẫu nhỏ gồm 4 cặp (`assignment/pairs_sample.csv`) để đảm bảo code không dính lỗi cú pháp cơ bản:

```powershell
python submission.py --pairs assignment/pairs_sample.csv --output-dir output_test
```

- Lệnh này sẽ chạy xong trong vòng **3–5 giây**.
- Kiểm tra thư mục `output_test/`:
  ```powershell
  Get-ChildItem output_test
  ```
  Nếu thấy sinh đủ 4 tệp CSV và tệp `similarities.csv` có đúng 4 dòng dữ liệu (+1 header) $\implies$ **Mã nguồn đã sẵn sàng 100%!**  
- Sau khi thử xong, bạn có thể xóa thư mục test này:
  ```powershell
  Remove-Item -Recurse -Force output_test
  ```

---

## BƯỚC 4: CHẠY CHÍNH THỨC TOÀN BỘ KHO DỮ LIỆU

Bây giờ ta chạy chính thức để tạo ra 4 tệp nộp bài trong thư mục `output/`:

```powershell
python submission.py
```

### Quá trình diễn ra như sau:
1. **Nạp đồ thị:** Đọc `articles.tsv` (4,604 đỉnh) và `links.tsv` (119,882 cạnh) $\to$ Mất ~0.3 giây.
2. **PageRank toàn cục:** Lặp khoảng 50–60 vòng $\to$ Mất ~0.5 giây.
3. **PageRank theo chủ đề:** Tính 3 chủ đề (Science, History, Geography) và ghép vector $r_{\text{mix}} \to$ Mất ~1.5 giây.
4. **MinHash:** Duyệt qua toàn bộ 4,604 tệp bài viết, tạo 9-shingles và băm 128 hàm $\to$ Mất khoảng 1–2 phút.
5. **Đo tương đồng Jaccard:** So sánh 100 cặp trong `data/pairs.csv` $\to$ Mất ~0.2 giây.

Khi hoàn thành, màn hình sẽ hiển thị:
```text
Đã ghi bốn tệp kết quả vào C:\Users\...\wikispeedia-student-package\output
```

---

## BƯỚC 5: CHẠY BỘ TỰ CHẤM ĐIỂM AUTOGRADER (MỤC TIÊU 100/100)

Bộ chấm `autograder.py` sẽ kiểm tra mã nguồn của bạn một cách độc lập với 13 bài kiểm tra Oracle nghiêm ngặt.

### Lệnh chạy tự chấm chuẩn:
```powershell
python autograder.py submission.py --timeout 300 --report grading_reports/self_check.json
```

> [!IMPORTANT]
> Hãy luôn thêm cờ `--timeout 300` khi chạy kiểm thử trên máy cá nhân để bộ chấm có đủ thời gian tính toán kiểm tra 100 cặp bài viết ở nhóm C3 mà không bị ngắt giữa chừng.

### Kết quả mong đợi (Đạt 100/100 điểm):
```text
A1   5/5   Đọc đồ thị
A2   8/8   Một vòng PageRank
A3  12/12  Nghiệm PageRank: đồ thị nhỏ và Wikispeedia
A4   5/5   Dừng và bất biến xác suất
B1   6/6   Tập chủ đề và phân phối dịch chuyển
B2  10/10  Cập nhật theo chủ đề và bù nút cụt
B3   9/9   Nghiệm ba chủ đề
B4   5/5   Ghép các vector chủ đề
C1   8/8   Chuẩn hóa và shingle
C2  17/17  Mã băm và chữ ký MinHash
C3   6/6   Jaccard và tương đồng chữ ký
C4   4/4   Trường hợp biên và tính chất MinHash
IO   5/5   Chạy main, bốn CSV và tính tái lập
TỔNG: 100/100
```

---

## BƯỚC 6: SOÁT LỖI THỦ CÔNG & KIỂM ĐỊNH KẾT QUẢ (SPOT-CHECK)

Để hoàn toàn tự tin khi nộp bài và làm báo cáo, bạn có thể tự mình kiểm tra tính hợp lý của dữ liệu bằng các script ngắn sau:

### 6.1. Kiểm tra số dòng của 4 tệp kết quả CSV
Chạy lệnh PowerShell:
```powershell
@(Get-Content output/pagerank.csv).Length, @(Get-Content output/topic_pagerank.csv).Length, @(Get-Content output/minhash.csv).Length, @(Get-Content output/similarities.csv).Length
```
*Kết quả phải đúng chính xác:*
- `4605` (4,604 bài viết + 1 dòng header).
- `18417` (18,416 dòng gồm 4 nhóm × 4,604 bài + 1 header).
- `4605` (4,604 bài viết + 1 header).
- `101` (100 cặp so sánh + 1 header).

---

### 6.2. Kiểm tra tính chất bảo toàn xác suất của PageRank
Tổng điểm PageRank của toàn bộ 4,604 bài viết phải bằng đúng **1.0**:
Chạy lệnh Python:
```powershell
python -c "import csv; scores = [float(row[1]) for row in list(csv.reader(open('output/pagerank.csv', encoding='utf-8')))[1:]]; print('Tổng điểm PageRank:', sum(scores)); print('Min:', min(scores), 'Max:', max(scores))"
```
*Kết quả mong đợi:*
- `Tổng điểm PageRank: 1.0` (hoặc lệch không quá $10^{-12}$).
- `Min > 0` (không có bài viết nào bị điểm âm).

---

### 6.3. Khám phá Top 10 bài viết quan trọng nhất trên Wikispeedia
Chạy lệnh Python để xem 10 bài viết có điểm PageRank cao nhất:
```powershell
python -c "import csv; rows = list(csv.reader(open('output/pagerank.csv', encoding='utf-8')))[1:]; rows.sort(key=lambda x: float(x[1]), reverse=True); [print(f'{i+1}. {r[0]}: {float(r[1]):.6f}') for i, r in enumerate(rows[:10])]"
```
*Bạn sẽ thấy kết quả rất trực quan:* Các bài viết đứng đầu là các trung tâm tri thức lớn của nhân loại như:
1. `United_States` (Nước Mỹ - được trỏ link nhiều nhất)
2. `United_Kingdom` (Vương quốc Anh)
3. `France` (Nước Pháp)
4. `Europe` (Châu Âu)
5. `England` (Nước Anh)
6. `World_War_II` (Chiến tranh thế giới thứ 2)
7. `Germany` (Nước Đức)
$\implies$ Điều này chứng minh thuật toán PageRank phản ánh cực kỳ chính xác cấu trúc mạng lưới thực tế!

---

### 6.4. Kiểm tra độ tương đồng Jaccard trong `similarities.csv`
Chạy lệnh xem 10 dòng đầu tiên của `similarities.csv`:
```powershell
python -c "import csv; [print(r) for r in list(csv.reader(open('output/similarities.csv', encoding='utf-8')))[:11]]"
```
*Điểm cần quan sát:*
- 10 dòng đầu tiên là **10 cặp bài viết tự so sánh với chính nó** (như `Glass, Glass`, `Marc_Pugh, Marc_Pugh`,...):
  - Cột `jaccard_exact` phải bằng đúng `1`.
  - Cột `jaccard_estimate` phải bằng đúng `1`.
- Các cặp khác nhau (ở các dòng tiếp theo): Giá trị `jaccard_estimate` từ MinHash phải xấp xỉ rất sát với giá trị `jaccard_exact`!

---

## 🛠️ CẨM NANG XỬ LÝ SỰ CỐ THƯỜNG GẶP (TROUBLESHOOTING)

| Hiện tượng lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| **`TimeoutError: Vượt 180 giây` tại nhóm C3** | Do chưa giải nén tệp `.tar.gz` vào `data/`, hoặc máy tính chạy các tiến trình khác làm chậm việc băm 100 cặp. | 1. Chạy lệnh giải nén ở Bước 2.<br>2. Thêm cờ `--timeout 300` khi chạy `autograder.py`. |
| **`FileNotFoundError: data/plaintext_articles/...`** | Thư mục văn bản chưa được giải nén đúng đường dẫn. | Kiểm tra lại xem trong thư mục `data/` đã có thư mục `plaintext_articles` chứa các tệp `.txt` chưa. |
| **`ValueError: Cạnh chứa ID không tồn tại`** | Do tự ý dùng `urllib.parse.unquote` để giải mã URL trong tên bài viết. | Đề bài yêu cầu giữ nguyên mã hóa percent-encoding (ví dụ `%C3%89douard_Manet`). Không được gọi `unquote`. |
| **`AssertionError: Điểm âm` hoặc `Tổng khác 1`** | Lỗi công thức bù nút cụt trong `pagerank`. | Kiểm tra biến `dangling`: phải cộng dồn điểm của các đỉnh không có cạnh ra `if not neighbors`, rồi chia đều $\frac{\beta \delta_t}{n}$. |
| **`AssertionError: Chữ ký không khớp từng số nguyên`** | Băm SHA-256 sai quy cách hoặc sai thứ tự 128 hàm băm. | Phải băm chuỗi byte UTF-8 của shingle, lấy 4 byte đầu dạng unsigned big-endian rồi lấy modulo $p$. |

---

## 📦 TỔNG KẾT DANH MỤC TỆP NỘP BÀI

Trước khi nộp bài cho giảng viên, hãy đảm bảo thư mục của bạn có:
1. Tệp mã nguồn: **`submission.py`**
2. Thư mục **`output/`** gồm đúng 4 tệp CSV:
   - `output/pagerank.csv`
   - `output/topic_pagerank.csv`
   - `output/minhash.csv`
   - `output/similarities.csv`

*(Không cần nộp các tệp dữ liệu `.tar.gz`, thư mục `assignment/` hay `autograder.py`).*
