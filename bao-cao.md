# Báo cáo thực hành LAB_2: Tự tính các chỉ số đánh giá hiệu năng: FMR, FNMR, EER, DET

Học phần 04211 Bảo mật sinh trắc, lớp 2610421101, học kỳ 1 năm học 2026-2027.

| Mục | Điền vào đây |
|---|---|
| Họ và tên | Võ Bá Huy |
| Mã số sinh viên | 2305CT2318 |
| Ngày nộp | 29/09/2026 |

## 1. Tóm tắt kết quả

Bài thực hành đã hoàn thiện việc cài đặt độc lập mô-đun tính toán các chỉ số đánh giá hiệu năng sinh trắc học (`bio_metrics.py`) và vượt qua toàn bộ 21/21 phép kiểm thử tự động. Trên tập dữ liệu thực nghiệm (1.000 cặp cùng người và 10.000 cặp khác người), giá trị EER tự tính cho ba hệ thống A, B, C lần lượt là 2,81%, 2,72% và 2,61%, hoàn toàn khớp với thư viện tham chiếu quốc tế `pyeer` với độ lệch tuyệt đối rất nhỏ từ 0,005 đến 0,015 điểm phần trăm (đạt chuẩn giới hạn cho phép <= 0,5 pp). Hệ thống C đạt hiệu năng tổng thể tối ưu nhất nhờ chỉ số phân tách d' cao nhất (4,461) và duy trì FNMR thấp nhất (7,0%) tại vùng an ninh cao (FMR = 0,1%). Tất cả các yêu cầu về bảng số liệu CSV và đồ thị trực quan hóa (phân bố điểm, DET, ROC) đều đã được hoàn thành trọn vẹn.

## 2. Mức độ hoàn thành

| Bước hoặc yêu cầu trong đề | Trạng thái | Minh chứng tại mục |
|---|---|---|
| Cài đặt các hàm đánh giá hiệu năng trong `bio_metrics.py` (error_rates, eer, fnmr_at_fmr, decidability, roc_auc, fpir_from_fmr) | Hoàn thành | 4.1 |
| Chạy kiểm thử đơn vị với `test_bio_metrics.py` (vượt qua 21/21 ca kiểm thử) | Hoàn thành | 4.2 |
| Chạy kịch bản đánh giá `th01_main.py` với mã sinh viên 2305CT2318 | Hoàn thành | 4.3 |
| Xuất bảng số liệu tổng hợp `TH01_2305CT2318_bang-ket-qua.csv` và kiểm chứng chéo với `pyeer` | Hoàn thành | 5.1 |
| Trực quan hóa dữ liệu và xuất các đồ thị phân bố điểm, đường cong DET, ROC | Hoàn thành | 5.2 |
| Phân tích định lượng hiệu năng và đánh giá rủi ro an ninh quy mô lớn (FPIR) | Hoàn thành | 6, 7 |

## 3. Môi trường thực hiện và khả năng tái lập

| Thông tin | Giá trị |
|---|---|
| Hệ điều hành, CPU, RAM | Windows 11 Home 64-bit, Intel Core i5 / AMD Ryzen, RAM 16 GB |
| Phiên bản Python | Python 3.12.x (môi trường ảo venv) |
| Thư viện chính và phiên bản | numpy, scipy, matplotlib, pyeer, setuptools<70 |
| Dữ liệu | Tập điểm số thực nghiệm do đề bài cung cấp (1.000 cặp Genuine, 10.000 cặp Impostor cho mỗi hệ thống A, B, C) |
| Hạt giống ngẫu nhiên | Không dùng (dữ liệu được cố định theo mã sinh viên 2305CT2318) |
| Lệnh chạy chính | `pip install "setuptools<70" numpy scipy matplotlib pyeer`<br>`python test_bio_metrics.py`<br>`python th01_main.py --ma-sv 2305CT2318` |
| Thời gian chạy | Thuật toán tự tính: ~0,001 s mỗi hệ thống; Tổng thời gian toàn kịch bản: ~2,5 s |

## 4. Các bước thực hiện và minh chứng

### 4.1. Cài đặt mô-đun tính toán `bio_metrics.py`

Đã triển khai các hàm toán học cốt lõi:
- `error_rates`: Quét ngưỡng và sử dụng `np.searchsorted` để tối ưu thời gian tính FMR và FNMR, xử lý đúng cả trường hợp điểm tương đồng (hệ thống A, B) và khoảng cách khoảng cách (hệ thống C).
- `eer`: Tìm điểm giao thoa giữa FMR và FNMR bằng phương pháp nội suy tuyến tính (linear interpolation), đóng gói kết quả trong lớp `EERResult` hỗ trợ cả unpack tuple lẫn truy cập từ điển.
- `fnmr_at_fmr`: Tìm ngưỡng tối ưu mang lại FNMR nhỏ nhất thỏa mãn điều kiện `FMR <= target_fmr`.
- `decidability`: Tính chỉ số phân tách d' dựa trên kỳ vọng và phương sai tổng thể (`ddof=0`).
- `roc_auc`: Tính diện tích dưới đường cong ROC bằng tích phân hình thang `trapezoid` sau khi chuẩn hóa mảng bằng `lexsort`.
- Các hàm trực quan hóa: `plot_distributions`, `plot_det`, `plot_roc` tiếp nhận cấu trúc dữ liệu linh hoạt và tham số từ khóa `title`.

### 4.2. Kiểm thử đơn vị tự động (`test_bio_metrics.py`)

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

### 4.3 Chạy kịch bản chính (th01_main.py)

Thực thi kịch bản đánh giá với mã sinh viên:

(venv) PS C:\Users\HUY\sinhtrac-2305CT2318\lab02> python th01_main.py --ma-sv 2305CT2318
[A] EER tự tính = 2.810%, pyeer = 2.805%, lệch 0.005 điểm phần trăm: ĐẠT
[B] EER tự tính = 2.720%, pyeer = 2.715%, lệch 0.005 điểm phần trăm: ĐẠT
[C] EER tự tính = 2.610%, pyeer = 2.595%, lệch 0.015 điểm phần trăm: ĐẠT

Hệ thống A tại ngưỡng 0.6674: FMR đo được = 0.100%
  N =         1,000: số so khớp sai kỳ vọng N*FMR = 1.00; FPIR = 1-(1-FMR)^N = 0.6323
  N =     1,000,000: số so khớp sai kỳ vọng N*FMR = 1,000.00; FPIR = 1-(1-FMR)^N = 1.0000
  N =   100,000,000: số so khớp sai kỳ vọng N*FMR = 100,000.00; FPIR = 1-(1-FMR)^N = 1.0000

Đã ghi bảng và đồ thị vào C:\Users\HUY\sinhtrac-2305CT2318\lab02\ket-qua (tiền tố TH01_2305CT2318)

## 5. Kết quả định lượng

Dữ liệu kết xuất từ tệp ket-qua/TH01_2305CT2318_bang-ket-qua.csv:
| Hệ thống | Cặp cùng người | Cặp khác người | EER tự tính (%) | EER nội suy (%) | EER pyeer (%) | Chênh lệch (pp) | Ngưỡng EER | FNMR @ FMR=1% (%) | FNMR @ FMR=0.1% (%) | Phân tách (d′) | AUC | Thời gian tự tính (s) | Thời gian pyeer (s) |
| A | 1.000 | 10.000 | 2,81 | 2,813 | 2,805 | 0,005 | 0,5144 |	8,9 | 21,2 | 4,238 | 0,9962 | 0,001 | 0,073 |
| B | 1.000 | 10.000 | 2,72 | 2,738 | 2,715 | 0,005 | 0,481 | 33,6 | 79,3 |	4,273 |	0,9916 | 0,001 | 0,007 |
| C | 1.000 | 10.000 | 2,61 | 2,6 |	2,595 |	0,015 |	0,7112 | 3,4 | 7 | 4,461 | 0,9958 |	0 |	0,006 |

## 6. Phân tích và thảo luận

<!-- Trả lời từng câu hỏi phân tích của đề, đánh số theo đề. Mỗi nhận định: hiện tượng, cơ chế
(dẫn tới hàm hoặc tham số), bằng chứng (dẫn tới bảng hoặc hình ở mục 5), giới hạn của kết luận. -->

{phân tích}
### 6.1. Về độ chính xác và tính hội tụ của thuật toán:
Thuật toán tự cài đặt cho kết quả trùng khớp gần như tuyệt đối với thư viện chuẩn pyeer. Độ lệch điểm phần trăm EER của cả 3 hệ thống chỉ đạt 0,005 pp (hệ thống A, B) và 0,015 pp (hệ thống C), nhỏ hơn rất nhiều so với ngưỡng khắt khe 0,5 pp.
Nhờ sử dụng thuật toán tìm kiếm nhị phân (np.searchsorted) trên mảng điểm đã sắp xếp, thời gian tính toán tự cài đặt chỉ mất 0,000 – 0,001 giây, nhanh hơn đáng kể so với việc tính toán qua các vòng lặp thông thường của pyeer (0,006 – 0,073 giây).

### 6.2. So sánh bản chất phân bố điểm số giữa các hệ thống (Hình 1):
Hệ thống A và B hoạt động dựa trên điểm tương đồng (Similarity Score): phân bố Genuine nằm bên phải (điểm cao hơn) và Impostor nằm bên trái (điểm thấp hơn).Hệ thống C hoạt động dựa trên thước đo khoảng cách (Distance Metric): phân bố Genuine nằm bên trái (khoảng cách nhỏ hơn, tập trung quanh giá trị 0,3) và Impostor nằm bên phải (khoảng cách lớn hơn, tập trung quanh giá trị 1,05). Vùng giao thoa (overlap) giữa hai phân bố của Hệ thống C hẹp nhất, dẫn đến chỉ số phân tách d' cao nhất trong cả ba hệ thống (d' = 4,461$).


## 7. Ý nghĩa đối với bảo mật
Trong thực tế triển khai bảo mật sinh trắc học, việc lựa chọn ngưỡng hoạt động phụ thuộc chặt chẽ vào mục tiêu an ninh:

Cấu hình đề xuất: Trong ba hệ thống, Hệ thống C là lựa chọn an toàn và tối ưu nhất để đưa vào môi trường sản xuất. Hệ thống này cho phép siết chặt tỷ lệ nhận nhầm kẻ tấn công xuống mức rất thấp (FMR = 0,1%) trong khi tỷ lệ người dùng hợp lệ bị từ chối oan chỉ ở mức 7,0%, đảm bảo hài hòa giữa độ an toàn và trải nghiệm người dùng.

Rủi ro của Hệ thống B: Tuyệt đối không sử dụng Hệ thống B cho các hệ thống an ninh nghiêm ngặt (như ngân hàng, cửa khẩu, phòng máy chủ). Nếu cấu hình ngưỡng tại FMR = 0,1% để ngăn chặn kẻ tấn công, sẽ có tới gần 80% người dùng hợp lệ bị từ chối đăng nhập (FNMR = 79,3%), làm tê liệt hoạt động của người dùng.

Tác động của FPIR trong bài toán nhận dạng 1:N: Khi mở rộng từ bài toán xác thực 1:1 sang nhận dạng 1:N ở quy mô lớn (ví dụ cơ sở dữ liệu quốc gia N = 1.000.000 danh tính), dù FMR ở mức rất thấp (0,1%), xác suất xảy ra ít nhất một lần nhận nhầm (FPIR) vẫn chạm mức xấp xỉ 100% với kỳ vọng 1.000 kết quả báo động giả cho mỗi lượt quét. Do đó, trong hệ thống nhận dạng quy mô lớn, bắt buộc phải kết hợp nhận dạng đa sinh trắc học hoặc sử dụng các bộ lọc phân tầng (pre-filtering) trước khi đối sánh chi tiết.
## 8. Sự cố gặp phải và cách xử lý

| Sự cố (thông báo lỗi) | Nguyên nhân | Cách xử lý |
|---|---|---|
| ModuleNotFoundError: No module named 'pkg_resources' | Gói setuptools phiên bản mới (v84.x) đã gỡ bỏ hoàn toàn module pkg_resources mà thư viện cũ pyeer phụ thuộc vào. | Hạ cấp phiên bản setuptools về bản tương thích bằng lệnh: pip install "setuptools<70" |
| TypeError: fnmr_at_fmr() got an unexpected keyword argument 'higher_is_better' | Kịch bản th01_main.py gọi hàm fnmr_at_fmr với điểm số thô (gen, imp, target_fmr, higher_is_better=...), trong khi hàm ban đầu chỉ nhận mảng tỷ lệ lỗi (fmr, fnmr, ...). | Nâng cấp hàm fnmr_at_fmr(*args, **kwargs) linh hoạt: tự động nhận diện nếu tham số là mảng điểm thô thì tính toán FMR/FNMR thông qua error_rates trước khi tra cứu. |
| AttributeError: module 'bio_metrics' has no attribute 'plot_distributions' | File bio_metrics.py trong quá trình sửa lỗi thuật toán đã bị ghi đè thiếu các hàm trực quan hóa đồ thị. | Bổ sung đầy đủ ba hàm vẽ biểu đồ plot_distributions, plot_det, plot_roc sử dụng matplotlib. |
| AttributeError: 'str' object has no attribute 'get' trong plot_distributions | Dữ liệu dists truyền từ th01_main.py là một từ điển (dict). Vòng lặp for item in dists: lặp qua khóa chuỗi chứ không phải từ điển con. | Viết hàm chuẩn hóa dữ liệu _extract_distributions và _extract_curves hỗ trợ bóc tách linh hoạt cả dạng dict, list, và tuple. |
| TypeError: plot_det() got an unexpected keyword argument 'title' | Hàm vẽ đồ thị chỉ khai báo tham số vị trí cố định, không nhận tham số từ khóa title. | Bổ sung tham số mặc định title="..." và **kwargs vào định nghĩa các hàm vẽ. |
| Biểu đồ DET trắng xóa và Biểu đồ ROC thiếu đường cong của ba hệ thống | Hàm bóc tách dữ liệu gán nhầm tuple (fmr, fnmr) thành khóa {"gen", "imp"} dẫn đến fmr và fnmr rỗng. | Sửa hàm _extract_curves trích xuất đúng vị trí phần tử trong tuple (fmr, fnmr) hoặc (thresholds, fmr, fnmr). |

## 9. Dữ liệu sinh trắc, nguồn tham khảo và công cụ AI

- [X] Kho không chứa ảnh vân tay, khuôn mặt, mống mắt, giọng nói của người thật, tập dữ liệu, tệp `.db`, `.pkl`, `.npy`, trọng số mô hình.
- [X] Mã dùng lại của người khác đã ghi nguồn ngay trong chú thích mã.

Nguồn tham khảo:
Tiêu chuẩn ISO/IEC 19795-1: Biometric performance testing and reporting.
Thư viện tham chiếu mã nguồn mở pyeer (Python Electronic Error Rate): https://github.com/mrquincymorris/pyeer, truy cập ngày 28/09/2026.
Tài liệu hướng dẫn thực hành Lab 02 - Học phần Bảo mật sinh trắc, Trường Đại học Hùng Vương TP. Hồ Chí Minh.

Công cụ AI: Sử dụng Gemini làm trợ lý kỹ thuật hỗ trợ dò lỗi cú pháp, gỡ lỗi kiểu dữ liệu đồ thị matplotlib, và tối ưu thuật toán tìm kiếm nhị phân trong bio_metrics.py.

## 10. Cam kết

Tôi cam kết các kết quả trong báo cáo này do chính tôi chạy trên máy của mình, các phần sử dụng lại của người khác đã được ghi nguồn đầy đủ.

Võ Bá Huy, ngày 29 tháng 09 năm 2026
