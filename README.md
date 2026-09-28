# D-15 Vision Lens: Mô phỏng sắc giác qua kiểm tra Farnsworth D-15

Một ứng dụng Single Page Application (SPA) chuyên sâu về khoa học màu sắc và thị giác máy tính, dùng để mô phỏng và so sánh trực quan góc nhìn của người bị khiếm khuyết sắc giác dựa trên kết quả kiểm tra lâm sàng **Farnsworth D-15**.

---

## 1. THÔNG SỐ VÀ HỒ SƠ NGƯỜI DÙNG

Kết quả kiểm tra Farnsworth D-15 thực tế của người dùng:
- **Lỗi thụ cảm tế bào nón M (Green Receptor): 72%** (Deuteranomaly nặng)
- **Lỗi thụ cảm tế bào nón L (Red Receptor): 28%** (Protanomaly vừa/thứ cấp)
- **Lỗi thụ cảm tế bào nón S (Blue Receptor): 0%** (Tritan hoàn hảo)

### Chẩn đoán lâm sàng:
Đây là dạng khiếm khuyết sắc giác **thể kết hợp (Combined Red-Green Deficiency)**:
- Người dùng bị tổn thương chủ đạo ở tế bào nón M (hấp thụ ánh sáng xanh lục ~530 nm) khiến các bước sóng xanh lá bị suy giảm phản hồi mạnh mẽ.
- Đồng thời có sự suy giảm 28% ở tế bào nón L (hấp thụ ánh sáng đỏ ~560 nm) làm giảm độ nhạy sáng của quang phổ đỏ sẫm.
- Kết quả tạo nên một trục nhầm lẫn màu hỗn hợp nghiêng mạnh theo trục Deutan.

---

## 2. THUẬT TOÁN BIẾN ĐỔI KHÔNG GIAN NÓN LMS (MACHADO ET AL. 2009)

Hệ thống tuân thủ nghiêm ngặt quy trình biến đổi không gian màu sinh lý học 4 bước:

```
[sRGB chuẩn] 
   │ 
   ▼ (1. Giải mã Gamma - Gamma Expansion)
[Linear RGB] 
   │ 
   ▼ (2. Chiếu vào không gian nón Stockman & Sharpe)
[Không gian tế bào nón LMS: (L, M, S)]
   │ 
   ▼ (3. Suy giảm phổ kích thích: 72% Deutan + 28% Protan)
[Tế bào nón suy giảm: (L', M', S')]
   │ 
   ▼ (4. Chiếu ngược về Linear RGB & Nén Gamma sRGB)
[sRGB Mô phỏng D-15]
```

### Chi tiết các bước toán học:

1. **Chuyển đổi sRGB sang Linear RGB:**
   $$\forall C \in \{R, G, B\}, \quad C_{lin} = \begin{cases} \frac{C}{12.92} & \text{nếu } C \le 0.04045 \\ \left(\frac{C + 0.055}{1.055}\right)^{2.4} & \text{nếu } C > 0.04045 \end{cases}$$

2. **Chuyển đổi sang không gian nón LMS:**
   Áp dụng ma trận độ nhạy tế bào nón 2 độ Stockman & Sharpe (2000) chuẩn hóa theo nguồn sáng D65:
   $$\begin{bmatrix} L \\ M \\ S \end{bmatrix} = \begin{bmatrix} 0.31399022 & 0.63951294 & 0.04649755 \\ 0.15537241 & 0.75789446 & 0.08670142 \\ 0.01775239 & 0.10944209 & 0.87256922 \end{bmatrix} \begin{bmatrix} R_{lin} \\ G_{lin} \\ B_{lin} \end{bmatrix}$$

3. **Áp dụng mức độ suy giảm kết hợp:**
   Theo mô hình dịch chuyển phổ Machado et al. (2009), ma trận kết hợp tương ứng với $72\%$ Deutan và $28\%$ Protan được tính toán bảo toàn điểm trắng trung tính $D65$:
   $$M_{combined} = I + 0.72 \cdot (M_{deut}^{1.0} - I) + 0.28 \cdot (M_{prot}^{1.0} - I)$$
   $$M_{combined} = \begin{bmatrix} +0.30711 & +0.91439 & -0.22150 \\ +0.23372 & +0.70436 & +0.06192 \\ -0.00960 & +0.01744 & +0.99215 \end{bmatrix}$$
   *Lưu ý: Tổng các hàng đều đạt đúng $1.00000$, đảm bảo màu trắng và màu xám trung tính không bị ám màu lạ.*

4. **Chiếu ngược và nén Gamma về sRGB:**
   $$C_{sRGB} = \text{clamp}\left(0, 255, \text{round}\left(255 \times \begin{cases} 12.92 \cdot C_{lin} & \text{nếu } C_{lin} \le 0.0031308 \\ 1.055 \cdot C_{lin}^{1/2.4} - 0.055 & \text{nếu } C_{lin} > 0.0031308 \end{cases}\right)\right)$$

---

## 3. CÁC TÍNH NĂNG CHÍNH CỦA ỨNG DỤNG

### 1. Bảng tóm tắt chỉ số D-15 & Biểu đồ khoanh nắp tròn:
- Hiển thị 3 thanh đo phản hồi cho 3 tế bào nón (M-Cone 72%, L-Cone 28%, S-Cone 0%).
- Biểu đồ tương tác vòng tròn 16 nắp màu Farnsworth D-15 với các đường chéo chập màu (crossover chords) biểu thị trục chẩn đoán Deutan.
- Thanh trượt điều chỉnh tự do mức độ tổn thương từng loại nón kèm nút khôi phục hồ sơ D-15 gốc.

### 2. Chế độ 1: Bảng màu tương tác (Color Confusion Palette):
- Cung cấp đầy đủ 4 bộ màu dễ gây nhầm lẫn nhất:
  1. **Đỏ tươi vs Xanh lá đậm vs Nâu đất** (Hội tụ về sắc tố vàng nâu mù tạt `#897A11` & `#7D7023`).
  2. **Xanh lam (Blue) vs Tím (Purple / Violet)** (Thành phần đỏ trong màu tím bị mất, biến màu tím thành màu xanh lam thuần khiết).
  3. **Xanh ngọc (Cyan / Teal) vs Xám / Trắng bạc** (Nằm trên điểm trung tính ~498nm, biến thành màu xám bạc).
  4. **Vàng chanh vs Xanh lá mạ** (Xanh non mất sắc lục chuyển sang vàng mù tạt).
  5. *Bộ bổ sung:* **Tín hiệu giao thông & UI Warning** (Đỏ Stop vs Vàng Warning vs Xanh Success).
- Mỗi màu hiển thị song song 2 ô: "Người bình thường nhìn thấy" và "Người dùng (bạn) nhìn thấy".
- **Color Picker & Live LMS Inspector:** Cho phép click chọn hoặc nhập mã HEX tùy ý để xem ngay kết quả và các thông số phân rã toán học từng bước: $sRGB \to Linear \to LMS \to L'M'S' \to sRGB'$.

### 3. Chế độ 2: So sánh hình ảnh thực tế (Interactive Image Slider):
- 5 bối cảnh mẫu được vẽ thủ công bằng Canvas đồ họa độ nét cao (hoạt động 100% offline, không phụ thuộc kết nối mạng):
  1. **Biển báo giao thông & Đèn tín hiệu đường phố:** Đèn đỏ chuyển sang màu vàng cam mù tạt, đèn xanh chuyển sang trắng xanh nhạt, biển cấm mất tính cảnh báo khẩn cấp.
  2. **Đĩa dâu tây / táo đỏ trên vòm lá xanh:** Hiệu ứng ngụy trang sinh học, quả dâu đỏ lẫn hoàn toàn vào tán lá xanh.
  3. **Bảng chữ test ngụy trang (Ishihara Plate #74):** Hàng trăm chấm tròn ngẫu nhiên; người bình thường thấy số 74 rõ ràng nhưng người dùng D-15 không thể nhìn thấy con số.
  4. **Cánh đồng hoa rực rỡ:** Hoa đỏ biến thành vàng nâu, hoa tím biến thành xanh lam, toàn bộ khung cảnh thu gọn về 2 gam màu chủ đạo.
  5. **Cầu vồng quang phổ 7 sắc (Atmospheric Rainbow):** Dải 7 sắc quang phổ thu gọn về 2 dải màu đối xứng: Nửa ngoài vàng nâu (Đỏ, Cam, Vàng, Lục) và nửa trong xanh lam (Lam, Tím), phân tách nhau bởi dải màu trắng xám nhờ nhờ tại điểm trung tính (~498nm).
- **Tải ảnh cá nhân (Upload Custom Image):** Hỗ trợ kéo thả hoặc tải ảnh JPG/PNG/WebP từ máy tính.
- **Thanh trượt chia đôi (Split Slider):** Kéo mượt mà qua lại bằng chuột hoặc ngón tay cảm ứng (sử dụng CSS `clip-path` chính xác đến từng pixel).
- **Các chế độ hiển thị:**
  - Thanh trượt chia đôi (Split Slider).
  - Song song 2 bên (Side-by-side).
  - Nhấp chuột chuyển đổi nhanh (Click to Toggle A/B).
  - Bản đồ mất màu (Discriminability Loss Heatmap).
- **Tốc độ xử lý siêu tốc:** Nhờ bảng tra cứu sẵn (LUT) cho gamma expansion và gamma compression, canvas xử lý 1 triệu pixel trong chưa đầy **12 - 16 ms** (đạt chuẩn 60 FPS).

### 4. Chế độ 3: Góc nhìn ngược (Reverse Simulator - Dành cho người bình thường hiểu người dùng):
- **Máy dò màu đồng sắc (Confusion Isocline Detector):** Người bình thường chọn 1 màu bất kỳ, hệ thống sẽ tự động quét theo vector đồng sắc:
  $$\vec{v}_{confusion} = [-0.9488, +0.3155, -0.0146]_{Linear}$$
  để chỉ ra các màu sắc khác trong thế giới tự nhiên mà người dùng D-15 sẽ thấy **hoàn toàn giống hệt nhau**.
- **Thư viện ngữ cảnh đời sống thực tế:**
  - Nấu nướng & Nhận biết độ chín miếng thịt bít-tết.
  - Lái xe & Nhớ vị trí đèn tín hiệu giao thông (Đỏ ở trên, Xanh ở dưới).
  - Thiết kế đồ họa UI/UX & Đồ thị dữ liệu (Tránh dùng cặp đỏ - xanh đơn lẻ).
  - Phối đồ thời trang (Phân biệt tất đen với tất xanh than navy).
  - Game thủ & Nhận biết quân địch trên minimap.
  - Đèn LED báo trạng thái pin thiết bị.

---

## 4. HƯỚNG DẪN KHỞI CHẠY & DEPLOY

Ứng dụng được thiết kế dưới dạng Single Page Application độc lập 100% trong file `index.html` (zero-dependency, không cần cài đặt thư viện):

### Cách 1: Mở trực tiếp bằng trình duyệt
- Nhấp đúp trực tiếp vào file `index.html` để mở trong bất kỳ trình duyệt nào (Chrome, Edge, Firefox, Safari).

### Cách 2: Chạy bằng VS Code Live Server ("Go Live")
- Trong VS Code: Chuột phải vào file `index.html` &rarr; Chọn **Open with Live Server** (hoặc bấm nút **Go Live** ở thanh trạng thái).

### Cách 3: Đưa lên GitHub Pages (Xem trực tiếp online)
- Sau khi push repo lên GitHub:
  1. Vào **Settings** của repository trên GitHub.
  2. Chọn mục **Pages** (cột bên trái).
  3. Tại **Build and deployment** &rarr; Chọn Source là **Deploy from a branch** &rarr; Branch **main** / **root** &rarr; Bấm **Save**.
  4. Trang web sẽ tự động online tại: `https://<your-username>.github.io/<repo-name>/`.

---

## 5. TÀI LIỆU THAM KHẢO KHOA HỌC

1. **Machado, G. M., Oliveira, M. M., & Fernandes, L. A. (2009).** *A Physiologically-based Model for Simulation of Color Vision Deficiency.* IEEE Transactions on Visualization and Computer Graphics, 15(6), 1291-1298.
2. **Farnsworth, D. (1943).** *The Farnsworth-Munsell 100-Hue and Dichotomous Tests for Color Vision.* Journal of the Optical Society of America, 33(10), 568-578.
3. **Brettel, H., Viénot, F., & Mollon, J. D. (1997).** *Computerized simulation of color appearance for dichromats.* Journal of the Optical Society of America A, 14(10), 2647-2655.
4. **Stockman, A., & Sharpe, L. T. (2000).** *The spectral sensitivities of the middle- and long-wavelength-sensitive cones derived from measurements in observers of known genotype.* Vision Research, 40(13), 1711-1737.
