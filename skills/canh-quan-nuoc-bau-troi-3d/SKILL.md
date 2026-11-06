---
name: "canh-quan-nuoc-bau-troi-3d"
description: "Dựng cảnh quan 3D bằng Three.js — sông suối chảy đúng hướng, mặt nước, cây cỏ, bầu trời theo mặt trời thật, mây, sương, mưa — kèm kiểm chứng render headless. Dùng cho cảnh quan, sông suối, mây trời 3D."
---

# Cảnh quan, sông suối, mây trời 3D

## 0. Nền tảng kỹ thuật
- Three.js r128: `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js` + `https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/` (OrbitControls, objects/Sky.js, objects/Water.js, utils/BufferGeometryUtils.js). Giữ cùng phiên bản.
- `renderer.outputEncoding = THREE.sRGBEncoding`; texture màu `.encoding = THREE.sRGBEncoding`; ACESFilmic, exposure 0.7–1.1.
- Không gán `scene.environment` cho mọi vật liệu (làm bạc màu); chỉ dùng env map cho vật liệu phản xạ (nước, kính, inox).
- `setPixelRatio(Math.min(devicePixelRatio, 2))`; shadow map 2048, khung bóng ôm sát khu chính.

## 1. Mặt trời & bầu trời theo vị trí thật
- Tính azimuth/elevation mặt trời từ ngày giờ + vĩ độ/kinh độ (công thức thiên văn rút gọn: kinh độ hoàng đạo, xích vĩ, giờ sao GMST).
- Vector hướng mặt trời trong scene (+X Đông, −Z Bắc): `(cos el·sin az, sin el, −cos el·cos az)`.
- `THREE.Sky`: turbidity 6–10 (nhiệt đới ẩm), rayleigh 1.5–3, mieCoefficient 0.005, mieDirectionalG 0.8.
- Thanh trượt giờ trong ngày → cập nhật Sky, DirectionalLight, màu sương.

## 2. Mây & bóng mây
- Chân mây tích nhiệt đới 600–1 500 m; không đặt mây thấp hơn đỉnh núi trừ khi cố ý.
- Mây: cụm Sprite texture canvas (fBm + mặt nạ tròn mềm), `depthWrite:false`, trôi theo gió.
- Bóng mây: chèn noise trôi vào shader địa hình qua `onBeforeCompile`, tối 15–30 % ánh sáng trực tiếp.
- `FogExp2` mật độ ≈ 1.978 / tầm_nhìn_m; màu sương = màu chân trời.

## 3. Sông suối có dòng chảy
- Hình học: 2 đường bờ (hoặc tim + bề rộng thật), lấy mẫu cùng số điểm, nối dải tam giác; UV u = khoảng cách dọc dòng (m), v = 0→1 ngang dòng.
- **Hướng chảy đúng** theo cao độ DEM (thượng → hạ lưu). Mặt nước thấp hơn bờ 0.5–2 m, giảm dần theo dòng.
- Shader nước: 2 normal map cuộn theo u với tốc độ tỷ lệ lưu tốc thật, Fresnel trộn màu trời và màu nước (mùa mưa nâu phù sa, mùa khô xanh trong), bọt trắng hai bờ.
- Màu nước phải đủ tối để không bị nhìn thành cát.
- Hạt dòng chảy: `THREE.Points` trôi dọc tim suối; nếu bị mặt cắt che thì dùng `depthTest:false` + additive.

## 4. Cây cỏ & bề mặt
- Mặt nạ thực vật từ ảnh vệ tinh georef (ExG = 2G − R − B); rải cây bằng InstancedMesh với seed cố định, tránh đường/nhà/nước.
- Kích thước thật: bụi 1–3 m, cây ăn trái 4–8 m, cây rừng 10–25 m, cỏ 0.5–1.2 m.
- Lấy màu từ ảnh thực tế (trung vị vùng cỏ, rừng, đá, đất).

## 5. Mưa & hoạt họa
- Mưa: LineSegments 5–15 k vệt, 6–9 m/s.
- Mọi hoạt họa khác tốc độ thật phải hiện hệ số tua nhanh.

## 6. Nhãn & thuyết minh
- Nhãn HTML có tránh chồng lấn theo độ ưu tiên và khoảng cách hiển thị tối đa.
- Thuyết minh bằng Web Speech API: chọn giọng vi-VN/en-US có sẵn (Edge: HoaiMy/NamMinh), đổi số + đơn vị thành chữ (dùng regex có cờ `u` và `(?![\p{L}\d])` thay cho `\b` để không cắt nhầm chữ có dấu), chia câu, tự chuyển cảnh khi đọc xong.

## 7. Kiểm chứng render headless
- Playwright route `**/three.min.js` và `**/examples/js/**` sang `node_modules/three` local khi CDN bị chặn.
- Chromium `--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader`, viewport 1280–1600, `screenshot({timeout:150000})`; chạy nền nếu quá 2 phút.
- Bắt `pageerror` và console error; không bàn giao khi còn lỗi.
- Xem ảnh ở ít nhất 4 góc: toàn cảnh, trực giao từ trên, mặt cắt địa tầng, cận cảnh công trình/suối.
