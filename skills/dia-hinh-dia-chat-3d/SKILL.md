---
name: "dia-hinh-dia-chat-3d"
description: "Dựng mô hình 3D địa hình, núi non, địa tầng, giếng khoan, nước ngầm ĐÚNG TỶ LỆ THẬT bằng Three.js từ DEM, bản đồ VN-2000, lỗ khoan. Dùng cho mô hình mỏ khoáng, địa chất thủy văn, mặt cắt, block diagram."
---

# Dựng địa hình & địa chất 3D đúng tỷ lệ

## 0. Luật vàng (không được vi phạm)
1. **1 đơn vị scene = 1 mét, cả ngang lẫn đứng.** Phóng đại đứng (vertical exaggeration) chỉ là nút bật/tắt; khi ≠ 1 thì luôn hiện nhãn `Phóng đại đứng ×N` trên màn hình và trong ảnh chụp.
2. **Mọi đối tượng phải có nguồn.** Thứ tự tin cậy: `[Khảo sát/DWG]` > `[DEM]` > `[Ảnh vệ tinh]` > `[Ảnh thực địa]` > `[Giả định]`. Ghi vào `obj.userData.source`, hiện trong tooltip/chú giải. Phần giả định tô viền nét đứt hoặc ghi chữ "giả định".
3. **Không bịa số liệu địa chất**: tên hệ tầng, độ sâu lớp, nhiệt độ, lưu lượng, đứt gãy chỉ lấy từ báo cáo thăm dò/lỗ khoan/bản đồ địa chất. Thiếu thì hỏi người dùng hoặc ghi "chưa có tài liệu".
4. **Không giả núi bằng sin/cos.** Địa hình lấy từ DEM hoặc đường đồng mức; procedural chỉ dùng cho chi tiết nhỏ (<2 m) như gợn đá.

## 1. Hệ tọa độ
- VN-2000 Khánh Hòa: TM múi 3°, kinh tuyến trục 108°15′, k0 = 0.9999, x0 = 500 000 m.
  ```
  +proj=tmerc +lat_0=0 +lon_0=108.25 +k=0.9999 +x_0=500000 +y_0=0 +ellps=WGS84
  +towgs84=-191.90441429,-39.30318279,-111.45032835,-0.00928836,0.01975479,-0.00427372,0.252906278 +units=m +no_defs
  ```
  (Tỉnh khác: tra kinh tuyến trục theo Thông tư 973/2001/TT-TCĐC.) Dùng `pyproj` (`pip install pyproj --break-system-packages`).
- **Bẫy quy ước VN**: trong bảng tọa độ, **X = Bắc (Northing ~1 35x xxx)**, **Y = Đông (Easting ~5xx xxx)**. Trong bản vẽ CAD thì trục x của CAD = Easting.
- Scene: gốc tại mốc chính (vd giếng khai thác) → `x = E − E0` (+X = Đông), `z = −(N − N0)` (−Z = Bắc), `y = H` (cao độ tuyệt đối, m).
- Ghi la bàn (mũi tên Bắc) và thước tỷ lệ thật trong scene.

## 2. Lấy DEM (cao độ thật)
Nguồn: Copernicus GLO-30, SRTM 1″, ALOS AW3D30 (~30 m, sai số đứng ~±5–10 m, là DSM nên có tán cây/nhà).
- Nếu sandbox chặn mạng: chạy trong trình duyệt (built-in browser / Chrome) đoạn JS lấy tile Terrarium AWS (`https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png`, có CORS), giải mã `h = R*256 + G + B/256 − 32768`, lấy mẫu song tuyến theo lưới đích (hàng 0 = mép Bắc), trả về base64 Uint16 (decimet) kèm checksum từng hàng.
- Lưới 4×4 km bước 30–40 m ≈ 10–18 k điểm. Lớn hơn thì tăng bước hoặc chia ô.
- Phương án thay thế: GeoTIFF Copernicus từ OpenTopography → đọc bằng `tifffile`/`rasterio`.
- **Hiệu chỉnh bắt buộc**: (a) san phẳng nền công trình theo cốt khảo sát; (b) khoét lòng suối theo polyline bờ suối từ DWG; (c) có điểm cao độ khảo sát thì dịch DEM cho khớp và báo sai số còn lại.

## 3. Lưới địa hình & khối địa chất (block diagram)
- `PlaneGeometry(W, D, nx−1, ny−1).rotateX(−Math.PI/2)` → vertex k ứng hàng j = floor(k/nx) (j=0 mép Bắc), cột i. Gán `y = H`; `computeVertexNormals()`.
- Mép khối: dựng 4 vách từ đường viền địa hình xuống đáy; tô màu theo lớp địa tầng tại từng độ cao.
- Lõi khu mỏ 5–10 m/ô, vùng xa 30–40 m.

## 4. Địa tầng & lỗ khoan
- Dữ liệu chuẩn hóa: `{gieng, E, N, H_mieng, lop:[{tu, den, ky_hieu, ten, mo_ta, nguon}]}`.
- 1 lỗ khoan: lớp bồi tích song song địa hình; đá gốc theo thế nằm trong báo cáo. ≥2 lỗ khoan: nội suy IDW (p = 2), kẹp không cho mặt lớp cắt nhau.
- Màu theo chuẩn bản đồ địa chất: Q vàng nhạt, N vàng cam, K xanh lá, J xanh lam, granit γ đỏ hồng. Ghi ký hiệu trong chú giải.
- **Mặt cắt**: clipping plane + thanh trượt; vẽ "nắp" mặt cắt tô theo lớp tại vị trí cắt.
- **Đứt gãy** phương vị α, góc dốc δ, hướng cắm β = α + 90°: `normal = (sinδ·sinβ, cosδ, −sinδ·cosβ)`.
- **Giếng khoan**: theo sơ đồ cấu trúc giếng; ống nhỏ vẽ thêm ống biểu tượng phóng đại bán kính và ghi chú hệ số.
- **Nước ngầm**: cột áp tự phun đúng mét, mũi tên dòng thấm từ vùng cấp → vùng thoát, ghi rõ hoạt họa có tua nhanh.

## 5. Ảnh vệ tinh / georeference
- ≥4 điểm khống chế thấy cả trên ảnh lẫn DWG; giải affine bình phương tối thiểu, báo RMS. RMS > 3 m → thêm điểm hoặc dùng homography.
- Dùng ảnh làm texture thì ghi bản quyền ảnh ở góc màn hình.

## 6. Kiểm chứng trước khi bàn giao
- Trang có `window.__audit()` trả về khoảng cách các điểm chính, cao độ mốc, độ sâu giếng, diện tích ranh mỏ, hệ số phóng đại hiện hành.
- Render headless (Playwright + SwiftShader) góc trực giao từ trên và 1 mặt cắt qua giếng.
- Báo cáo bảng "đại lượng – giá trị mô hình – giá trị nguồn – sai số – nguồn".
