# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

**Cách dùng:** Thay mọi dấu `…` bằng bài làm thật của bạn trước khi nộp link fork trên [LMS](https://aithucchien.vinuni.edu.vn). Giữ nguyên bốn mục và bảng để coach đọc nhanh. Viết ngắn, cụ thể theo ảnh/vùng; không cần thuật ngữ chuyên sâu. Ví dụ trong [hướng dẫn mẫu](../../reports/REPORT_TEMPLATE.md) chỉ giúp hiểu cách điền, không phải câu trả lời để chép lại.

- Mã học viên theo lớp: 2A202603076
- Ngày / CVAT local: 2026-11-11 / CVAT 2.74.1 chạy local (Docker), `http://localhost:8080`
- Công cụ đã dùng:
  - CVAT local, với công cụ Polygon và Mask.
  - Trợ lý AI Claude Code: đặt tọa độ polygon/mask theo ảnh phóng to có lưới, rồi đẩy vào CVAT qua API (bản vẽ tay của cả 9 task).
  - Gợi ý tự động SAM 3: hàm nuclio `segmentation_anything` (loại detector, chạy CPU) qua **Actions → Automatic annotation** của CVAT. Chỉ dùng cho 3 tier, chạy trên các task riêng `easy_semantic_sam`, `medium_instance_sam`, `hard_panoptic_sam`. Kết quả SAM không được chỉnh tay.
  - `scripts/inspect_submissions.py` để kiểm cấu trúc.
  - `scoring/score.py` và `scoring/scorecard.py` với reference được cấp, để so bản vẽ tay với bản SAM.

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh [LMS](https://aithucchien.vinuni.edu.vn) đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Không có export lỗi. Cả 9 ZIP đều qua `inspect_submissions.py` với kết quả `[OK]`.
- **3 tier** (`easy_semantic`, `medium_instance`, `hard_panoptic`): bản nộp là kết quả SAM 3 export thẳng từ CVAT.
- **6 checkpoint**: vẫn là bản vẽ tay, không dùng SAM.

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- **Ảnh, vị trí và object Medium đầu tiên tự vẽ:** ảnh `000000181542.jpg`, người phụ nữ mặc áo dài trắng đi giữa vạch qua đường, bbox khoảng x 192–303, y 132–458.
- **Class và quy tắc tôi dùng để chọn biên:**
  - Class `person`, vẽ bằng polygon 33 đỉnh, trước khi chạy SAM.
  - Biên đi theo mép ngoài của tóc, vai, tay, tà áo, ống quần và dép.
  - Dây túi đeo vai và tà áo bay ra ngoài thân vẫn tính là người.
  - Ở chỗ chồng lên xe máy và người lái phía sau, mép áo trắng là ranh giới, không lấn sang xe.
- **Nếu dùng gợi ý sau đó:**
  - Sau khi vẽ tay xong cả task, tôi chạy SAM 3 auto annotation trên một task riêng.
  - Với object này, SAM cũng cho đúng 1 `person`, bbox x 190–304, y 131–459, gần như trùng polygon vẽ tay. SAM cũng gán xe bán tải (pickup) là `truck`, giống quyết định của tôi theo quy ước COCO.
  - Khác biệt chính: SAM bám biên sát hơn và tìm thêm nhiều người/xe nhỏ ở xa mà bản vẽ tay bỏ sót.
  - Tôi chọn nộp bản SAM sau khi so điểm hai bản bằng `scoring/score.py` (xem mục 3).
- **Nếu không dùng gợi ý:** không áp dụng, vì bản nộp tier là bản SAM.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- **Task/ảnh/vùng:** `medium_instance`, ảnh `000000373353.jpg`, nhóm người đi bộ qua đường phía phải, x khoảng 428–513, y 300–425.
- **Lỗi thuộc loại:** gộp-tách. Trong bản vẽ tay, một người bị tách thành 2 instance, và một người khác bị gộp vào mask sai.
- **Bằng chứng tôi nhìn thấy:**
  - Ban đầu tôi vẽ "người đội mũ trắng" và "người áo khoác đỏ" thành 2 person.
  - Sau khi export, mask "áo đỏ" chỉ còn 28 px, vì gần như bị mask "mũ trắng" che hết.
  - Phóng to ×5 thì thấy mũ trắng, áo đỏ, ba lô đen và boot nâu thuộc cùng một người.
  - Phía trước còn một người đội mũ trùm đi boot be mà tôi chưa tách ra.
- **Quy tắc và hành động sửa:**
  - Quy tắc: mỗi người đếm được là một mask, xác định theo đầu, thân và chân nối liền nhau, không theo màu áo.
  - Tôi vẽ lại thành 3 person riêng: người tóc nâu áo đen (x 428–468), người mũ trùm boot be (x 452–500, ở gần nhất), và người mũ trắng áo đỏ (x 481–513, ở sau).
  - Phần bị che của người phía sau được trừ khỏi mask của người đó.
  - Bản SAM nộp cuối cũng tách vùng này thành đúng 3 person (x 427–465, x 445–501, x 480–512), khớp với cách sửa.
- **Sau sửa đã Save và export lại chưa?** Rồi.

Nếu bạn **đã xem Summary tự đánh giá trên GitHub Actions hoặc tự chạy script**, ghi ngắn một kết quả liên quan lỗi vừa sửa (ví dụ task, metric trước/sau nếu có):

- Đã tự chạy `scoring/score.py` và `scoring/scorecard.py` với reference được cấp. "Trước" là bản vẽ tay đã sửa lỗi trên, "sau" là bản SAM đang nộp.
- `medium_instance`: metric (mean matched IoU × recall) từ 0.434 lên 0.736, điểm từ 2.4 lên 23.9 / 32.
  - Mean matched IoU từ 0.700 lên 0.804. Recall (ngưỡng IoU 0.5) từ 0.62 lên 0.92. Precision (ngưỡng IoU 0.5) từ 0.75 xuống 0.72.
  - TP/FP/FN từ 44/15/27 thành 65/25/6. Nộp 90 mask, tham chiếu có 71 (dư 19).
  - Liên quan lỗi vừa sửa: bản vẽ tay sót nhiều người/xe nhỏ (FN 27). SAM giảm còn 6, nhưng đổi lại có thêm mask thừa (FP 25).
- `easy_semantic`: mIoU từ 0.652 lên 0.729, điểm từ 11.2 lên 14.6 / 20. Building tăng mạnh nhất (0.606 lên 0.845). Sidewalk (0.480) và vegetation (0.393) vẫn thấp.
- `hard_panoptic`: PQ từ 0.331 lên 0.436, điểm từ 8.8 lên 15.7 / 30. Sidewalk và bicycle vẫn PQ 0.000, person 0.177.
- Scorecard ba tier: từ 22.4 lên **54.2 / 82**. Không có review signal nào ở cả hai bản.

Scorecard ba tier tối đa **82**, không phải điểm cuối trên 100. Không tự ghi PASS/top 3/bonus; người phụ trách xác nhận theo tiêu chí lớp. Không đưa file ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1 | `cp6_coverage`, xe buýt trắng giữa ảnh (x 255–636) và xe tải hộp bên trái (x 45–256) | (a) gán sang lớp gần nhất (`car`) để phủ kín ảnh; (b) để trống vì không có lớp phù hợp | `classes.json` của cp6 chỉ có road, sidewalk, building, vegetation, sky, car, person. Không có bus hoặc truck. Không bịa class. | Để trống (bản vẽ tay, không dùng SAM). Độ phủ khoảng 70%, phần còn lại là xe buýt, xe tải, xe bán đồ ăn, trụ cứu hỏa và capô xe đang lái. Hỏi coach: tham chiếu có tính các vùng này vào độ phủ không? |
| 2 | `hard_panoptic`, ảnh `000000350023.jpg`, dải bê tông phân cách giữa đường (x 290–322, y 336–436) | (a) `road`, vì cùng mặt đường; (b) `sidewalk`, vì là dải nhô cao | Xe không chạy được và người không đi bộ trên đó. Ranh road/sidewalk theo chức năng, không theo màu. | Bản vẽ tay để void. Bản SAM đang nộp gộp dải này vào `road`, tôi không chỉnh tay. Hỏi coach: dải phân cách nên là road, sidewalk hay void? |
| 3 | `hard_panoptic`, ảnh `000000460147.jpg`, xe chở ô tô ở làn giữa (x 315–440, y 165–372) | (a) cả khối là một `truck`; (b) khung xe là `truck`, các ô tô chở trên xe là `car` riêng | Các ô tô trên xe nhìn rõ hình dạng xe con. Panoptic yêu cầu mỗi thing nhìn thấy là một segment. | Theo cách (b). Bản vẽ tay có khung `truck` và 4 `car`. Bản SAM đang nộp có khung `truck` và 3 `car` rõ trên xe. Hỏi coach: tham chiếu có gộp ô tô trên xe chở vào truck không? |
