# BÀI 04: HƯỚNG DẪN CHẠY THỰC NGHIỆM, TỰ CHẤM ĐIỂM VÀ NỘP BÀI

---

## 1. Chuẩn bị môi trường và dữ liệu

### 1.1. Yêu cầu môi trường
- **Python:** Phiên bản 3.10 trở lên.
  ```bash
  python --version
  ```
- **NumPy (khuyến nghị):** Để tăng tốc độ tính toán ma trận chữ ký MinHash (đã được cấu hình tự động nhận diện trong `submission.py`).
  ```bash
  pip install numpy
  ```

### 1.2. Giải nén dữ liệu (Bắt buộc)
Mở terminal tại thư mục gốc của bài tập `wikispeedia-student-package` và chạy lệnh giải nén:
```bash
tar -xzf data/wikispeedia_paths-and-graph.tar.gz -C data
tar -xzf data/wikispeedia_articles_plaintext.tar.gz -C data
```
> [!IMPORTANT]
> Việc giải nén sẵn hai thư mục `wikispeedia_paths-and-graph/` và `plaintext_articles/` vào trong `data/` giúp chương trình đọc tệp trực tiếp từ ổ cứng với tốc độ nhanh gấp hàng chục lần so với việc giải nén tạm trong bộ nhớ.

---

## 2. Chạy bài làm để tạo 4 tệp CSV kết quả

Chạy lệnh:
```bash
python submission.py
```

Chương trình sẽ tự động thực hiện lần lượt:
1. Nạp đồ thị `articles.tsv` và `links.tsv`.
2. Tính PageRank toàn cục $\implies$ Ghi vào `output/pagerank.csv`.
3. Tính PageRank cho 3 chủ đề (Science, History, Geography) và vector ghép $r_{\text{mix}} \implies$ Ghi vào `output/topic_pagerank.csv`.
4. Tạo shingle và tính chữ ký MinHash 128 tọa độ cho toàn bộ 4,604 bài viết $\implies$ Ghi vào `output/minhash.csv`.
5. Đọc 100 cặp bài viết trong `data/pairs.csv`, tính Jaccard chính xác và Jaccard ước lượng $\implies$ Ghi vào `output/similarities.csv`.

---

## 3. Kiểm tra định dạng 4 tệp đầu ra trong `output/`

Sau khi chạy xong, hãy kiểm tra số dòng của từng tệp trong thư mục `output/`:

| Tệp kết quả | Header (Dòng tiêu đề) | Số dòng dữ liệu | Tổng số dòng (kể cả Header) |
|---|---|:---:|:---:|
| `pagerank.csv` | `article_id,score` | **4,604** | 4,605 |
| `topic_pagerank.csv` | `topic,article_id,score` | **18,416** | 18,417 |
| `minhash.csv` | `article_id,shingle_count,h000,...,h127` | **4,604** | 4,605 |
| `similarities.csv` | `article_id_a,article_id_b,jaccard_exact,jaccard_estimate` | **100** | 101 |

- **Giải thích tệp `topic_pagerank.csv`:**
  Tệp gồm 4 nhóm dữ liệu xếp chồng lên nhau theo thứ tự: `Science`, `History`, `Geography`, `r_mix`. Mỗi nhóm chứa đủ 4,604 bài viết $\implies 4 \times 4604 = 18,416$ dòng dữ liệu.
- **Quy chuẩn ghi số thực:** Các điểm số được ghi với độ chính xác 17 chữ số có nghĩa (`format(val, ".17g")`), dùng dấu chấm thập phân, không làm tròn số trong quá trình tính toán.

---

## 4. Cách sử dụng bộ tự chấm `autograder.py` (100 Điểm)

Đề bài cung cấp công cụ tự chấm điểm độc lập `autograder.py`. Công cụ này kiểm tra bài làm của bạn qua **13 nhóm tiêu chí riêng biệt**.

### Lệnh chạy tự chấm:
```bash
python autograder.py submission.py --timeout 300 --report grading_reports/self_check.json
```

> [!TIP]
> **Tại sao cần tham số `--timeout 300`?**
> Nhóm tiêu chí `C3` kiểm tra độ tương đồng trên toàn bộ 100 cặp bài viết. Bộ chấm `autograder.py` tự tính lại chữ ký tham chiếu bằng mã nguồn Python thuần (không dùng NumPy), dẫn đến việc phải tính hàng trăm triệu phép toán băm. Trên một số máy tính cá nhân, quá trình này có thể tốn hơn 2 phút. Tăng timeout lên 300 giây đảm bảo bài làm không bị báo lỗi quá giờ (`TimeoutError`).

### Ý nghĩa của 13 tiêu chí chấm điểm:

```
Nhóm   Điểm   Ý nghĩa kiểm thử
---------------------------------------------------------------------------------------
A1      5đ    Kiểm tra load_graph: tập đỉnh UTF-8, gộp cạnh trùng, giữ self-loop.
A2      8đ    Kiểm tra 1 bước lặp PageRank: kiểm soát chính xác lượng bù nút cụt.
A3     12đ    Kiểm tra nghiệm PageRank: khớp nghiệm tham chiếu khử Gauss & Wikispeedia.
A4      5đ    Kiểm tra quy tắc dừng: tổng xác suất bằng 1, bất biến khi đảo thứ tự đỉnh.
B1      6đ    Kiểm tra topic_distribution: lọc đúng tiền tố, phân phối v hợp lệ.
B2     10đ    Kiểm tra cập nhật theo chủ đề: bù nút cụt theo u, teleport theo v.
B3      9đ    Kiểm tra nghiệm 3 chủ đề: khớp nghiệm tham chiếu hệ phương trình.
B4      5đ    Kiểm tra mix_pageranks: thỏa mãn tính chất tuyến tính của PageRank.
C1      8đ    Kiểm tra normalize_text và shingles: đúng quy tắc ký tự ASCII và Unicode.
C2     17đ    Kiểm tra shingle_id và minhash: khớp chính xác 100% 128 số nguyên.
C3      6đ    Kiểm tra jaccard và signature_similarity: tính đúng trên 100 cặp chính thức.
C4      4đ    Kiểm tra trường hợp biên: tập rỗng trả p, tính chất min(A ∪ B) = min(minA, minB).
IO      5đ    Kiểm tra chạy main(): sinh đủ 4 CSV với cấu trúc và định dạng chuẩn xác.
---------------------------------------------------------------------------------------
TỔNG  100đ    ĐIỂM TỐI ĐA
```

Nếu muốn xem báo cáo chi tiết dạng JSON (chứa thông tin stack trace nếu có lỗi), bạn có thể mở tệp:  
`grading_reports/self_check.json`.

---

## 5. Quy định đóng gói và nộp bài

Khi nộp bài cho giảng viên (theo thông báo của môn học):

### Các tệp CẦN nộp:
1. **`submission.py`:** Tệp mã nguồn duy nhất chứa toàn bộ lời giải.
2. **Thư mục `output/`** chứa đủ 4 tệp kết quả:
   - `output/pagerank.csv`
   - `output/topic_pagerank.csv`
   - `output/minhash.csv`
   - `output/similarities.csv`

### Các tệp KHÔNG CẦN nộp:
- Không nộp thư mục dữ liệu `data/` (rất nặng).
- Không nộp thư mục `assignment/` gốc hoặc tệp `autograder.py`.
- Không nộp báo cáo chấm `grading_reports/`.

### Lưu ý cuối cùng trước khi nộp bài:
- Đảm bảo trong docstring ở đầu tệp `submission.py` có ghi chú phiên bản NumPy bạn đang sử dụng (ví dụ: `NumPy 2.3.5; vẫn chạy được bằng thư viện chuẩn`).
- Chạy lại lệnh `python submission.py` một lần cuối cùng để đảm bảo các tệp trong `output/` là kết quả mới nhất được sinh ra từ chính mã nguồn `submission.py` của bạn.
