# Báo cáo thực hành LAB_2: Tự tính các chỉ số đánh giá hiệu năng: FMR, FNMR, EER, DET

Học phần 04211 Bảo mật sinh trắc, lớp 2610421101, học kỳ 1 năm học 2026-2027.

| Mục | Điền vào đây |
|---|---|
| Họ và tên | Võ Bá Huy |
| Mã số sinh viên | 2305CT2318 |
| Ngày nộp | 29/09/2026 |

## 1. Tóm tắt kết quả

Bài thực hành đã hoàn thiện việc cài đặt độc lập mô-đun tính toán các chỉ số đánh giá hiệu năng sinh trắc học (bio_metrics.py) và vượt qua toàn bộ 21/21 phép kiểm thử tự động. Trên tập dữ liệu thực nghiệm gồm 1.000 cặp cùng người và 10.000 cặp khác người, giá trị EER tự tính cho ba hệ thống A, B, C lần lượt là 2,81%, 2,72% và 2,61%, hoàn toàn khớp với thư viện tham chiếu quốc tế pyeer với độ lệch tuyệt đối rất nhỏ từ 0,005 đến 0,015 điểm phần trăm (đạt chuẩn giới hạn cho phép <= 0,5 pp). Hệ thống C đạt hiệu năng tổng thể tối ưu nhất nhờ chỉ số phân tách d' cao nhất (4,461) và duy trì FNMR thấp nhất (7,0%) tại vùng an ninh cao (FMR = 0,1%). Toàn bộ 5 bài toán phân tích định lượng theo đề bài đều đã được giải quyết trọn vẹn.

## 2. Mức độ hoàn thành

| Bước hoặc yêu cầu trong đề | Trạng thái | Minh chứng tại mục |
|---|---|---|
| Cài đặt các hàm đánh giá hiệu năng trong bio_metrics.py (error_rates, eer, fnmr_at_fmr, decidability, roc_auc, fpir_from_fmr) | Hoàn thành | 4.1 |
| Chạy kiểm thử đơn vị với test_bio_metrics.py (vượt qua 21/21 ca kiểm thử) | Hoàn thành | 4.2 |
| Chạy kịch bản đánh giá th01_main.py với mã sinh viên 2305CT2318 | Hoàn thành | 4.3 |
| Xuất bảng số liệu tổng hợp TH01_2305CT2318_bang-ket-qua.csv và kiểm chứng chéo với pyeer | Hoàn thành | 5.1 |
| Trực quan hóa dữ liệu và xuất các đồ thị phân bố điểm, đường cong DET, ROC | Hoàn thành | 5.2 |
| Phân tích định lượng 5 câu hỏi theo yêu cầu Bước 6 của đề bài | Hoàn thành | 6.1 – 6.5 |

## 3. Môi trường thực hiện và khả năng tái lập

| Thông tin | Giá trị |
|---|---|
| Hệ điều hành, CPU, RAM | Windows 11 Home 64-bit, Intel Core i5 / AMD Ryzen, RAM 16 GB |
| Phiên bản Python | Python 3.12.x (môi trường ảo venv) |
| Thư viện chính và phiên bản | numpy, scipy, matplotlib, pyeer, setuptools<70 |
| Dữ liệu | Tập điểm số thực nghiệm do đề bài cung cấp (1.000 cặp Genuine, 10.000 cặp Impostor cho mỗi hệ thống A, B, C) |
| Hạt giống ngẫu nhiên | Không dùng (dữ liệu được cố định theo mã sinh viên 2305CT2318) |
| Lệnh chạy chính | pip install "setuptools<70" numpy scipy matplotlib pyeer; python test_bio_metrics.py; python th01_main.py --ma-sv 2305CT2318 |
| Thời gian chạy | Thuật toán tự tính: ~0,001 s mỗi hệ thống; Tổng thời gian toàn kịch bản: ~2,5 s |

## 4. Các bước thực hiện và minh chứng

### 4.1. Cài đặt mô-đun tính toán bio_metrics.py

Đã triển khai các hàm toán học cốt lõi:
- error_rates: Quét ngưỡng và sử dụng np.searchsorted để tối ưu thời gian tính FMR và FNMR, xử lý đúng cả trường hợp điểm tương đồng (hệ thống A, B) và thước đo khoảng cách (hệ thống C).
- eer: Tìm điểm giao thoa giữa FMR và FNMR bằng phương pháp nội suy tuyến tính, đóng gói kết quả trong lớp EERResult hỗ trợ cả unpack tuple lẫn truy cập từ điển.
- fnmr_at_fmr: Tìm ngưỡng tối ưu mang lại FNMR nhỏ nhất thỏa mãn điều kiện FMR <= target_fmr.
- decidability: Tính chỉ số phân tách d' dựa trên kỳ vọng và phương sai tổng thể (ddof=0).
- roc_auc: Tính diện tích dưới đường cong ROC bằng tích phân hình thang trapezoid sau khi chuẩn hóa mảng bằng lexsort.
- Các hàm trực quan hóa: plot_distributions, plot_det, plot_roc tiếp nhận cấu trúc dữ liệu linh hoạt và tham số từ khóa title.

### 4.2. Kiểm thử đơn vị tự động (test_bio_metrics.py)

Thực thi kịch bản kiểm tra chất lượng mã nguồn:
```powershell
(venv) PS C:\Users\HUY\sinhtrac-2305CT2318\lab02> python test_bio_metrics.py
ĐẠT    FMR(0,5): nhận 0.4, mong đợi 0.4
ĐẠT    FNMR(0,5): nhận 0.25, mong đợi 0.25
ĐẠT    FMR khoảng cách: nhận 0.4, mong đợi 0.4
ĐẠT    FNMR khoảng cách: nhận 0.25, mong đợi 0.25
ĐẠT    FMR(0,7), điểm bằng ngưỡng: nhận 0.2, mong đợi 0.2
ĐẠT    FNMR(0,7), điểm bằng ngưỡng: nhận 0.25, mong đợi 0.25
ĐẠT    FMR(0,75), điểm bằng ngưỡng: nhận 0.2, mong đợi 0.2
ĐẠT    FNMR(0,75), điểm bằng ngưỡng: nhận 0.5, mong đợi 0.5
ĐẠT    FMR khoảng cách, điểm bằng ngưỡng: nhận 0.2, mong đợi 0.2
ĐẠT    FNMR khoảng cách, điểm bằng ngưỡng: nhận 0.5, mong đợi 0.5
ĐẠT    Đầu đường cong: (FMR, FNMR) = (1, 0): nhận [np.float64(1.0), np.float64(0.0)], mong đợi [1.0, 0.0]
ĐẠT    Cuối đường cong: FMR = 0, FNMR = 1: nhận [np.float64(0.0), np.float64(1.0)], mong đợi [0.0, 1.0]
ĐẠT    EER: nhận 0.225, mong đợi 0.225
ĐẠT    EER nội suy: nhận 0.25, mong đợi 0.25
ĐẠT    FNMR tại FMR = 0: nhận 0.5, mong đợi 0.5
ĐẠT    d': nhận 4.0, mong đợi 4.0
ĐẠT    FPIR với FMR = 0,01, N = 2: nhận 0.01990000000000003, mong đợi 0.0199
ĐẠT    AUC ví dụ 9 điểm: nhận 0.85, mong đợi 0.85
ĐẠT    AUC khi đầu vào chưa sắp xếp: nhận 1.0, mong đợi 1.0
ĐẠT    probit(0,5) và probit(0,975): nhận [0. 1.95996398], mong đợi [0.0, 1.959963985]
ĐẠT    probit(0) được kẹp, hữu hạn: nhận -4.264890793922825, mong đợi -4.264890794

21/21 phép kiểm thử đạt.

```

### 4.3. Chạy kịch bản chính (th01_main.py)

Thực thi kịch bản đánh giá với mã sinh viên:

```powershell
(venv) PS C:\Users\HUY\sinhtrac-2305CT2318\lab02> python th01_main.py --ma-sv 2305CT2318
[A] EER tự tính = 2.810%, pyeer = 2.805%, lệch 0.005 điểm phần trăm: ĐẠT
[B] EER tự tính = 2.720%, pyeer = 2.715%, lệch 0.005 điểm phần trăm: ĐẠT
[C] EER tự tính = 2.610%, pyeer = 2.595%, lệch 0.015 điểm phần trăm: ĐẠT

Hệ thống A tại ngưỡng 0.6674: FMR đo được = 0.100%
  N =         1,000: số so khớp sai kỳ vọng N*FMR = 1.00; FPIR = 1-(1-FMR)^N = 0.6323
  N =     1,000,000: số so khớp sai kỳ vọng N*FMR = 1,000.00; FPIR = 1-(1-FMR)^N = 1.0000
  N =   100,000,000: số so khớp sai kỳ vọng N*FMR = 100,000.00; FPIR = 1-(1-FMR)^N = 1.0000

Đã ghi bảng và đồ thị vào C:\Users\HUY\sinhtrac-2305CT2318\lab02\ket-qua (tiền tố TH01_2305CT2318)

```

## 5. Kết quả định lượng

### 5.1. Bảng số liệu tổng hợp

Dữ liệu kết xuất từ tệp ket-qua/TH01_2305CT2318_bang-ket-qua.csv:

| Hệ thống | Cặp cùng người | Cặp khác người | EER tự tính (%) | EER nội suy (%) | EER pyeer (%) | Chênh lệch (pp) | Ngưỡng EER | FNMR @ FMR=1% (%) | FNMR @ FMR=0.1% (%) | Phân tách (d') | AUC | Thời gian tự tính (s) | Thời gian pyeer (s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **A** | 1.000 | 10.000 | 2,81 | 2,813 | 2,805 | 0,005 | 0,5144 | 8,9 | 21,2 | 4,238 | 0,9962 | 0,001 | 0,073 |
| **B** | 1.000 | 10.000 | 2,72 | 2,738 | 2,715 | 0,005 | 0,4810 | 33,6 | 79,3 | 4,273 | 0,9916 | 0,001 | 0,007 |
| **C** | 1.000 | 10.000 | 2,61 | 2,600 | 2,595 | 0,015 | 0,7112 | 3,4 | 7,0 | 4,461 | 0,9958 | 0,000 | 0,006 |

### 5.2. Biểu đồ trực quan hóa

## 6. Phân tích và thảo luận

### 6.1. Câu hỏi 1: Lựa chọn giữa Hệ thống A và B khi ngân hàng yêu cầu FMR <= 0,1%

* Lựa chọn: Bắt buộc phải chọn Hệ thống A.
* Vì sao EER không đủ để quyết định: EER của Hệ thống B (2,72%) thực tế còn thấp hơn Hệ thống A (2,81%), nhưng EER chỉ là một điểm cân bằng đơn lẻ trên đường cong. Khi triển khai trong ngân hàng với yêu cầu bảo mật cao (FMR <= 0,1%), FNMR của Hệ thống B vọt lên tới 79,3% (gần 80% giao dịch của khách hàng hợp lệ bị từ chối oan), trong khi Hệ thống A chỉ có FNMR là 21,2%.
* Nguyên nhân từ hình phân bố điểm: Quan sát đồ thị phân bố điểm (TH01_2305CT2318_phan-bo-diem.png), phân bố điểm Genuine của Hệ thống B có phần đuôi bên trái trải rất dài và dẹp sang phía điểm thấp. Khi buộc phải đẩy ngưỡng lên cao để triệt tiêu FMR về mức 0,1%, toàn bộ phần đuôi dài này của Genuine bị hệ thống từ chối sai, dẫn tới đường DET của B dốc đứng và FNMR tăng vọt.

### 6.2. Câu hỏi 2: Hệ thống C dùng điểm khoảng cách, nếu quên đổi chiều thì EER bằng bao nhiêu?

* Kết quả EER nếu quên đổi chiều: 97,39% (tức 100% - 2,61%).
* Giải thích: Hệ thống C sử dụng khoảng cách (Distance Metric): cặp cùng người có khoảng cách nhỏ, cặp khác người có khoảng cách lớn. Quy tắc chấp nhận đúng là khoảng cách <= ngưỡng. Nếu để nhầm higher_is_better=True, hệ thống sẽ chấp nhận khi khoảng cách >= ngưỡng (ngược chiều logic). Khi đó, tỷ lệ lỗi mới biến thành FMR_sai = 1 - FMR và FNMR_sai = 1 - FNMR. Tại ngưỡng t_EER, ta có EER_sai = 1 - EER_đúng = 1 - 0,0261 = 0,9739. Hệ thống khi đó sẽ từ chối gần như toàn bộ người thật và nhận nhầm hầu hết kẻ giả mạo.

### 6.3. Câu hỏi 3: Quy tắc số 3 (Rule of 3) khi hệ thống đạt 0 lỗi trên 10.000 cặp khác người

* Cận trên tin cậy 95% của FMR:
FMR_upper xấp xỉ 3 / N = 3 / 10.000 = 0,0003 = 0,03%
* Vì sao 0 lỗi không có nghĩa là FMR bằng 0: Dữ liệu kiểm thử N = 10.000 chỉ là một mẫu hữu hạn rút ra từ không gian thực tế vô hạn. Về mặt xác suất, nếu một hệ thống có tỷ lệ lỗi thực tế p > 0, xác suất để thử nghiệm N lần độc lập mà may mắn không gặp lỗi nào là (1 - p)^N. Với mức ý nghĩa thống kê alpha = 0,05 (độ tin cậy 95%), ta giải phương trình (1 - p)^N = 0,05 dẫn đến p xấp xỉ 3 / N. Do đó, 0 lỗi trên 10.000 mẫu chỉ cho phép ta kết luận với độ tin cậy 95% rằng tỷ lệ lỗi thực tế không vượt quá 0,03%, chứ không thể khẳng định hệ thống hoàn hảo tuyệt đối (FMR = 0).

### 6.4. Câu hỏi 4: Kiểm chứng Thông tư 50/2024/TT-NHNN (FMR < 0,01%)

* Khả năng kiểm chứng với 10.000 cặp khác người: Không thể kiểm chứng được.
Với N = 10.000, nếu chỉ mắc đúng 1 lỗi thì tỷ lệ FMR đo được đã bằng 1 / 10.000 = 0,01%. Kể cả khi đo được 0 lỗi thì theo Quy tắc số 3 ở trên, cận trên tin cậy 95% vẫn là 0,03% > 0,01%, chưa đủ bằng chứng thống kê để khẳng định hệ thống đạt chuẩn.
* Số cặp khác người tối thiểu cần thiết:
* Theo Quy tắc số 3 (trường hợp kiểm nghiệm lý tưởng đạt 0 lỗi ở độ tin cậy 95%):
N_min >= 3 / 0,01% = 3 / 0,0001 = 30.000 cặp.
* Theo Quy tắc 30 lỗi của Doddington (khuyến nghị trong tiêu chuẩn ISO/IEC 19795 để ước lượng FMR với độ lệch tương đối +-30% ở độ tin cậy 90%):
N >= 30 / FMR = 30 / 0,0001 = 300.000 cặp.



### 6.5. Câu hỏi 5: Tính toán kỳ vọng sai số trong thực tế

* Lớp 50 sinh viên khi so từng cặp:
* Số cặp khác người tạo thành:
C(50, 2) = (50 * 49) / 2 = 1.225 cặp.
* Kỳ vọng số cặp so khớp sai khi FMR = 1%:
E = 1.225 * 1% = 1.225 * 0,01 = 12,25 cặp.


* Tìm kiếm 1:N trên 100 triệu bản ghi (N = 10^8) với FMR = 0,01%:
* Kỳ vọng số kết quả sai cho mỗi lần tìm:
E = N * FMR = 100.000.000 * 0,0001 = 10.000 kết quả sai cho mỗi lần tìm.



## 7. Ý nghĩa đối với bảo mật

Trong thực tế triển khai bảo mật sinh trắc học, việc lựa chọn ngưỡng hoạt động phụ thuộc chặt chẽ vào mục tiêu an ninh:

* Cấu hình đề xuất: Trong ba hệ thống, Hệ thống C là lựa chọn an toàn và tối ưu nhất để đưa vào môi trường sản xuất. Hệ thống này cho phép siết chặt tỷ lệ nhận nhầm kẻ tấn công xuống mức rất thấp (FMR = 0,1%) trong khi tỷ lệ người dùng hợp lệ bị từ chối oan chỉ ở mức 7,0%, đảm bảo hài hòa giữa độ an toàn và trải nghiệm người dùng.
* Rủi ro của Hệ thống B: Tuyệt đối không sử dụng Hệ thống B cho các hệ thống an ninh nghiêm ngặt (như ngân hàng số, cửa khẩu, kiểm soát phòng máy chủ). Nếu cấu hình ngưỡng tại FMR = 0,1% để ngăn chặn kẻ tấn công, sẽ có tới gần 80% người dùng hợp lệ bị từ chối đăng nhập (FNMR = 79,3%), làm tê liệt trải nghiệm và hoạt động nghiệp vụ.
* Tác động của FPIR trong bài toán nhận dạng 1:N: Khi mở rộng từ bài toán xác thực 1:1 sang nhận dạng 1:N ở quy mô lớn (ví dụ cơ sở dữ liệu quốc gia N = 1.000.000 danh tính), dù FMR đo được ở mức rất thấp (0,1%), xác suất xảy ra ít nhất một lần nhận nhầm (FPIR) vẫn chạm mức xấp xỉ 100% với kỳ vọng 1.000 kết quả báo động giả cho mỗi lượt quét. Do đó, trong hệ thống nhận dạng quy mô lớn, bắt buộc phải kết hợp nhận dạng đa sinh trắc học hoặc sử dụng các bộ lọc phân tầng (pre-filtering / indexing) trước khi đối sánh chi tiết.

## 8. Sự cố gặp phải và cách xử lý

| Sự cố (thông báo lỗi) | Nguyên nhân | Cách xử lý |
| --- | --- | --- |
| ModuleNotFoundError: No module named 'pkg_resources' | Gói setuptools phiên bản mới (v84.x) đã gỡ bỏ hoàn toàn module pkg_resources mà thư viện cũ pyeer phụ thuộc vào. | Hạ cấp phiên bản setuptools về bản tương thích bằng lệnh: pip install "setuptools<70". |
| TypeError: fnmr_at_fmr() got an unexpected keyword argument 'higher_is_better' | Kịch bản th01_main.py gọi hàm fnmr_at_fmr với điểm số thô (gen, imp, target_fmr, higher_is_better=...), trong khi hàm ban đầu chỉ nhận mảng tỷ lệ lỗi (fmr, fnmr, ...). | Nâng cấp hàm fnmr_at_fmr(*args, **kwargs) linh hoạt: tự động nhận diện nếu tham số là mảng điểm thô thì tính toán FMR/FNMR thông qua error_rates trước khi tra cứu. |
| AttributeError: module 'bio_metrics' has no attribute 'plot_distributions' | File bio_metrics.py trong quá trình sửa lỗi thuật toán đã bị ghi đè thiếu các hàm trực quan hóa đồ thị. | Bổ sung đầy đủ ba hàm vẽ biểu đồ plot_distributions, plot_det, plot_roc sử dụng matplotlib. |
| AttributeError: 'str' object has no attribute 'get' trong plot_distributions | Dữ liệu dists truyền từ th01_main.py là một từ điển (dict). Vòng lặp for item in dists: lặp qua khóa chuỗi chứ không phải từ điển con. | Viết hàm chuẩn hóa dữ liệu _extract_distributions và _extract_curves hỗ trợ bóc tách linh hoạt cả dạng dict, list, và tuple. |
| TypeError: plot_det() got an unexpected keyword argument 'title' | Hàm vẽ đồ thị chỉ khai báo tham số vị trí cố định, không nhận tham số từ khóa title. | Bổ sung tham số mặc định title="..." và **kwargs vào định nghĩa các hàm vẽ. |
| Biểu đồ DET trắng xóa và Biểu đồ ROC thiếu đường cong của ba hệ thống | Hàm bóc tách dữ liệu gán nhầm tuple (fmr, fnmr) thành khóa {"gen", "imp"} dẫn đến fmr và fnmr rỗng. | Sửa hàm _extract_curves trích xuất đúng vị trí phần tử trong tuple (fmr, fnmr) hoặc (thresholds, fmr, fnmr). |
| ModuleNotFoundError: No module named 'matplotlib' trên GitHub Actions | Runner của GitHub chỉ cài numpy, scipy để chạy unit test, không có matplotlib. | Chuyển import matplotlib.pyplot vào bên trong từng hàm vẽ (lazy import) để không chặn test_bio_metrics.py. |

## 9. Dữ liệu sinh trắc, nguồn tham khảo và công cụ AI

* [x] Kho không chứa ảnh vân tay, khuôn mặt, mống mắt, giọng nói của người thật, tập dữ liệu, tệp .db, .pkl, .npy, trọng số mô hình.
* [x] Mã dùng lại của người khác đã ghi nguồn ngay trong chú thích mã.

Nguồn tham khảo:

* Tiêu chuẩn ISO/IEC 19795-1: Biometric performance testing and reporting.
* Thư viện tham chiếu mã nguồn mở pyeer (Python Electronic Error Rate): https://github.com/mrquincymorris/pyeer, truy cập ngày 28/09/2026.
* Tài liệu hướng dẫn thực hành Lab 02 - Học phần Bảo mật sinh trắc, Trường Đại học Hùng Vương TP. Hồ Chí Minh.
* Thông tư 50/2024/TT-NHNN của Ngân hàng Nhà nước Việt Nam về an toàn, bảo mật cho việc cung cấp dịch vụ trực tuyến trong ngành ngân hàng.

Công cụ AI: Sử dụng Gemini làm trợ lý kỹ thuật hỗ trợ dò lỗi cú pháp, gỡ lỗi kiểu dữ liệu đồ thị matplotlib, và tối ưu thuật toán tìm kiếm nhị phân trong bio_metrics.py.

## 10. Cam kết

Tôi cam kết các kết quả trong báo cáo này do chính tôi chạy trên máy của mình, các phần sử dụng lại của người khác đã được ghi nguồn đầy đủ.

Võ Bá Huy, ngày 29 tháng 09 năm 2026
