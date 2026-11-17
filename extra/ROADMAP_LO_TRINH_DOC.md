# 🗺️ LỘ TRÌNH ĐỌC TÀI LIỆU (READING ROADMAP)
> **Dành cho người mới bắt đầu từ số 0 — Thứ tự đọc tối ưu nhất để nắm trọn vẹn kiến thức**

---

## 🧭 BẢNG LỘ TRÌNH TỔNG QUAN 5 CHẶNG

```
[ CHẶNG 1: NHẬP MÔN & BỨC TRANH TOÀN CẢNH ]
       │  • Bài 01: Bối cảnh trò chơi Wikispeedia & Đồ thị
       │  • Bài 02: Từ điển bóc tách 15 file trong dự án
       │  • Bài 03: Báo cáo phân tích đặc tả đề bài
       ▼
[ CHẶNG 2: PHẦN A — PAGERANK TOÀN CỤC ]
       │  • Bài 01: Lý thuyết Chú khỉ lướt web, bẫy nhện & Ví dụ tính tay từng bước
       │  • Bài 02: Bóc tách từng dòng code hàm load_graph
       │  • Bài 03: Bóc tách từng dòng code hàm pagerank (forward-push)
       ▼
[ CHẶNG 3: PHẦN B — PAGERANK THEO CHỦ ĐỀ & TUYẾN TÍNH ]
       │  • Bài 01: Lý thuyết Cá nhân hóa, phép màu tuyến tính & Ví dụ tính tay
       │  • Bài 02: Bóc tách từng dòng code hàm topic_distribution
       │  • Bài 03: Bóc tách từng dòng code hàm mix_pageranks
       ▼
[ CHẶNG 4: PHẦN C — SHINGLING, MINHASH & JACCARD ]
       │  • Bài 01: Lý thuyết Cửa sổ trượt, Định lý MinHash & Ví dụ tính tay 4 hàm băm
       │  • Bài 02: Bóc tách code normalize_text & shingles
       │  • Bài 03: Bóc tách code shingle_id & minhash (NumPy)
       │  • Bài 04: Bóc tách code jaccard & signature_similarity
       ▼
[ CHẶNG 5: VẬN HÀNH HỆ THỐNG, CHẠY TEST & NỘP BÀI ]
          • Bài 01: Bóc tách pipeline hàm main() và quy trình xuất 4 file CSV
          • Bài 02: Sổ tay gõ lệnh PowerShell chạy test, chấm điểm 100/100
          • Bài 03: Hướng dẫn đóng gói và nộp bài
```

---

## 📖 CHI TIẾT THỨ TỰ TỪNG BÀI ĐỌC

### 🏁 CHẶNG 1: KHỞI ĐỘNG — HIỂU BÀI TOÁN VÀ DỮ LIỆU (Thời gian: ~20 phút)
*Mục tiêu: Biết mình đang làm việc trên dữ liệu gì, các file trong thư mục dùng để làm gì.*

1. 📄 **Đọc đầu tiên:** [01_nhap_mon_va_tong_quan/01_tong_quan_du_lieu_wikispeedia.md](01_nhap_mon_va_tong_quan/01_tong_quan_du_lieu_wikispeedia.md)
   - Hiểu trò chơi Wikispeedia là gì.
   - Nắm các khái niệm cơ bản: Đỉnh (bài viết), Cạnh (đường link), Nút cụt (dead-end), Cạnh tự nối (self-loop).
   - Đọc quy ước đọc dữ liệu và ví dụ minh họa 4 bài viết ở Mục 4.
2. 📄 **Đọc thứ hai:** [01_nhap_mon_va_tong_quan/02_tu_dien_15_file_du_an.md](01_nhap_mon_va_tong_quan/02_tu_dien_15_file_du_an.md)
   - Xem qua danh mục 15 tệp trong dự án: tệp nào là dữ liệu thô, tệp nào là cấu hình, tệp nào là đầu ra cần nộp.
3. 📄 **Đọc tham khảo thêm:** [01_nhap_mon_va_tong_quan/03_bao_cao_phan_tich_de_bai.md](01_nhap_mon_va_tong_quan/03_bao_cao_phan_tich_de_bai.md)
   - Báo cáo phân tích chuyên sâu về barem 13 tiêu chí chấm điểm và yêu cầu kỹ thuật.

---

### 🚀 CHẶNG 2: PHẦN A — THUẬT TOÁN PAGERANK TOÀN CỤC (Thời gian: ~30 phút)
*Mục tiêu: Hiểu bản chất cách Google xếp hạng trang web và đọc hiểu code Phần A.*

4. 📄 **Đọc thứ tư (LÝ THUYẾT A):** [02_phan_a_pagerank_toan_cuc/01_ly_thuyet_va_vi_du_tinh_tay.md](02_phan_a_pagerank_toan_cuc/01_ly_thuyet_va_vi_du_tinh_tay.md)
   - Đọc câu chuyện hòn đảo và chú khỉ lướt web ngẫu nhiên.
   - Hiểu 2 hiểm họa: Bẫy nhện (*Spider Traps*) và Nút cụt (*Dead Ends*).
   - **Đặc biệt xem kỹ Mục 6:** Ví dụ tính toán bằng số từng bước trên đồ thị 3 đảo để thấy từng con số hội tụ ra sao.
5. 📄 **Đọc thứ năm (CODE A1):** [02_phan_a_pagerank_toan_cuc/02_giai_thich_code_load_graph.md](02_phan_a_pagerank_toan_cuc/02_giai_thich_code_load_graph.md)
   - Hiểu từng dòng code đọc TSV, sắp xếp byte UTF-8 thô, cách dùng Dictionary `index` để tra cứu $O(1)$ và dùng `set` để gộp cạnh trùng.
6. 📄 **Đọc thứ sáu (CODE A2):** [02_phan_a_pagerank_toan_cuc/03_giai_thich_code_pagerank.md](02_phan_a_pagerank_toan_cuc/03_giai_thich_code_pagerank.md)
   - Hiểu cách gom điểm nút cụt `dangling`, kỹ thuật Forward-push bắn điểm về phía trước trên đồ thị thưa, và tại sao tổng điểm luôn bằng 1.0.

---

### 🎯 CHẶNG 3: PHẦN B — PAGERANK THEO CHỦ ĐỀ & TÍNH CHẤT TUYẾN TÍNH (Thời gian: ~25 phút)
*Mục tiêu: Hiểu cơ chế cá nhân hóa kết quả tìm kiếm và tính chất ghép vector.*

7. 📄 **Đọc thứ bảy (LÝ THUYẾT B):** [03_phan_b_pagerank_theo_chu_de/01_ly_thuyet_va_vi_du_tinh_tay.md](03_phan_b_pagerank_theo_chu_de/01_ly_thuyet_va_vi_du_tinh_tay.md)
   - Tại sao người tìm kiếm khác nhau lại cần kết quả khác nhau (Ví dụ từ khóa *"Apple"*).
   - Mô hình chú khỉ nhảy dù có sở thích chủ đề.
   - **Xem kỹ Mục 4:** So sánh sự đảo chiều thứ hạng giữa PageRank toàn cục và PageRank chủ đề trên cùng đồ thị 3 đảo.
   - Hiểu phép màu của tính chất tuyến tính giúp ghép vector $r_{\text{mix}}$ trong $0.001$ giây.
8. 📄 **Đọc thứ tám (CODE B1):** [03_phan_b_pagerank_theo_chu_de/02_giai_thich_code_topic_distribution.md](03_phan_b_pagerank_theo_chu_de/02_giai_thich_code_topic_distribution.md)
   - Hiểu cách lọc tiền tố cây danh mục `label.startswith(prefix + ".")` và gán xác suất $1/|S|$.
9. 📄 **Đọc thứ chín (CODE B2):** [03_phan_b_pagerank_theo_chu_de/03_giai_thich_code_mix_pageranks.md](03_phan_b_pagerank_theo_chu_de/03_giai_thich_code_mix_pageranks.md)
   - Hiểu code nhân tích vô hướng ghép 3 vector theo trọng số $0.5, 0.3, 0.2$.

---

### 🔍 CHẶNG 4: PHẦN C — TƯƠNG ĐỒNG VĂN BẢN VỚI SHINGLING & MINHASH (Thời gian: ~35 phút)
*Mục tiêu: Hiểu cách phát hiện đạo văn/trang web trùng lặp trong dữ liệu lớn và thuật toán MinHash.*

10. 📄 **Đọc thứ mười (LÝ THUYẾT C):** [04_phan_c_shingling_va_minhash/01_ly_thuyet_va_vi_du_tinh_tay.md](04_phan_c_shingling_va_minhash/01_ly_thuyet_va_vi_du_tinh_tay.md)
    - Tại sao so sánh văn bản trực tiếp lại thất bại trong Big Data.
    - Khái niệm cửa sổ trượt $k$-shingle và hệ số tương đồng Jaccard $\frac{|A \cap B|}{|A \cup B|}$.
    - **Trọng tâm đặc biệt (Mục 5 & 6):** Định lý MinHash kinh điển và bài toán tính tay qua 4 hàm băm để thấy tại sao số tọa độ trùng nhau lại phản ánh đúng Jaccard.
11. 📄 **Đọc thứ mười một (CODE C1):** [04_phan_c_shingling_va_minhash/02_giai_thich_code_normalize_text_va_shingles.md](04_phan_c_shingling_va_minhash/02_giai_thich_code_normalize_text_va_shingles.md)
    - Cạm bẫy vì sao không được dùng `.lower()` mà phải dùng `str.maketrans` chỉ hạ 26 chữ ASCII, giữ nguyên Unicode; và code cắt cửa sổ trượt 9 ký tự.
12. 📄 **Đọc thứ mười hai (CODE C2):** [04_phan_c_shingling_va_minhash/03_giai_thich_code_shingle_id_va_minhash.md](04_phan_c_shingling_va_minhash/03_giai_thich_code_shingle_id_va_minhash.md)
    - Code băm SHA-256 4 byte đầu modulo số nguyên tố Mersenne $p = 2^{31}-1$; kỹ thuật vector hóa với `np.fromiter` trên mảng `int64` (chứng minh không bị tràn số) giúp tăng tốc độ gấp 20 lần.
13. 📄 **Đọc thứ mười ba (CODE C3):** [04_phan_c_shingling_va_minhash/04_giai_thich_code_jaccard_va_similarity.md](04_phan_c_shingling_va_minhash/04_giai_thich_code_jaccard_va_similarity.md)
    - Code tính Jaccard chính xác và đếm tọa độ chữ ký trùng nhau, xử lý tập rỗng.

---

### ⚙️ CHẶNG 5: VẬN HÀNH TOÀN BỘ, CHẠY THỬ NGHIỆM VÀ NỘP BÀI (Thời gian: ~20 phút)
*Mục tiêu: Chạy ra kết quả, tự chấm điểm 100/100 và sẵn sàng nộp bài.*

14. 📄 **Đọc thứ mười bốn (PIPELINE):** [05_thuc_hanh_va_nop_bai/01_giai_thich_code_pipeline_ham_main.md](05_thuc_hanh_va_nop_bai/01_giai_thich_code_pipeline_ham_main.md)
    - Hiểu cách hàm `main()` liên kết toàn bộ các hàm từ A $\to$ B $\to$ C thành một dây chuyền tự động, kỹ thuật tiết kiệm RAM và định dạng số `.17g`.
15. 📄 **Đọc thứ mười lăm (THỰC HÀNH CHẠY & TEST):** [05_thuc_hanh_va_nop_bai/02_so_tay_chay_va_kiem_thu_100_diem.md](05_thuc_hanh_va_nop_bai/02_so_tay_chay_va_kiem_thu_100_diem.md)
    - Mở PowerShell gõ từng lệnh: giải nén $\to$ smoke test $\to$ chạy thật $\to$ tự chấm với cờ `--timeout 300` $\to$ chạy các lệnh spot-check kiểm tra số liệu.
16. 📄 **Đọc cuối cùng (ĐÓNG GÓI NỘP BÀI):** [05_thuc_hanh_va_nop_bai/03_huong_dan_dong_goi_nop_bai.md](05_thuc_hanh_va_nop_bai/03_huong_dan_dong_goi_nop_bai.md)
    - Kiểm tra lại danh mục file nộp (chỉ nộp `submission.py` và 4 file CSV trong `output/`).

---

🎉 **Chúc bạn có một trải nghiệm học tập thật hào hứng và đạt điểm số tối đa trong bài tập này!**
