---
name: "do-hoa-3d-dien-anh"
description: "Nâng chất lượng hình ảnh mô hình 3D Three.js lên mức điện ảnh: hậu kỳ màu/ánh sáng, vật liệu PBR, cây chi tiết, nước phản chiếu, mức chất lượng theo thiết bị. Dùng khi cần làm đẹp hoặc chi tiết hơn một cảnh 3D đã dựng."
---

# Đồ họa 3D điện ảnh cho Three.js

Dùng cùng các skill `dia-hinh-dia-chat-3d`, `canh-quan-nuoc-bau-troi-3d`, `kien-truc-cad-3d`. Skill này KHÔNG đổi dữ liệu (tọa độ, tỷ lệ thật, số liệu); chỉ đổi cách hiển thị.

## 0. Nguyên tắc
- Đúng trước, đẹp sau: không hy sinh tỷ lệ thật, vị trí hay số liệu để có hình đẹp.
- Mỗi nâng cấp phải có ảnh trước/sau (render headless cùng góc máy) và đo FPS/số tam giác.
- Luôn có 3 mức chất lượng: Thấp (điện thoại), Vừa (laptop), Cao (máy chiếu/PC rời). Tự chọn theo thiết bị, cho người dùng đổi.
- Giữ chạy được trong 1 file HTML; thư viện chỉ từ cdnjs / cdn.jsdelivr.net/npm, khóa phiên bản (vd three r128 + examples/js).

## 1. Ánh sáng và màu (hiệu quả lớn nhất, làm trước)
- renderer.outputEncoding = sRGBEncoding; toneMapping = ACESFilmicToneMapping; exposure 0.8–1.1 theo giờ.
- Môi trường: tạo PMREM từ Sky (đã có) làm scene.environment để mọi vật liệu PBR nhận phản xạ trời; cập nhật khi đổi giờ (không mỗi khung hình).
- Mặt trời: DirectionalLight + HemisphereLight (trời xanh / đất nâu) cường độ thấp; tránh AmbientLight phẳng.
- Bóng: PCFSoftShadowMap, shadow frustum bám theo camera (đã có) ; mức Cao dùng mapSize 4096, Thấp 1024 hoặc tắt bóng cây.
- Sương khí quyển: FogExp2 màu lấy từ màu chân trời của Sky để núi xa nhạt dần.

## 2. Hậu kỳ (post-processing) với EffectComposer (examples/js r128)
Thứ tự pass: RenderPass → SAOPass hoặc SSAOPass (che khuất góc, chỉ mức Cao) → UnrealBloomPass (strength 0.15–0.3, threshold 0.85; chỉ cho nắng, nước, đèn) → ShaderPass color grading (lift/gamma/gain + saturation, ấm nhẹ) → vignette → FXAA/SMAA.
- Flycam: thêm BokehPass (depth of field) ở cảnh cận; tắt khi người dùng tự xoay.
- Hậu kỳ phải tôn trọng logarithmicDepthBuffer: SSAO cần depth tuyến tính — nếu lỗi, dùng SAOPass hoặc tắt AO.
- Tắt toàn bộ hậu kỳ ở mức Thấp.

## 3. Vật liệu PBR
- Tạo texture thủ tục bằng canvas: albedo + normal (đạo hàm Sobel từ height noise) + roughness. Mẫu: tôn sóng, bê tông, gạch lát, ngói, đá granit, inox xước.
- Mái tôn: normalMap sóng + metalness 0.4–0.6 + roughness 0.35; ngói: normal theo hàng ngói.
- Inox/ống: metalness 1, roughness 0.25, envMap = PMREM.
- Địa hình: splat theo độ dốc/độ cao/độ cong (đất, cỏ, đá, cát) + detail normal map lặp 2–4 m khi nhìn gần; texture tổng ≥ 0,5 m/px quanh khu nhà máy (dùng texture riêng cho vùng nhỏ).

## 4. Cây cỏ
- Thay tán khối tròn bằng cụm lá dạng thẻ (alpha cards) 12–30 tấm/cây, texture lá có alphaTest, mặt sau sáng hơn (translucency giả).
- Biến thể màu/tỷ lệ/xoay qua instanceColor + instanceMatrix; LOD: gần = cards, xa = impostor billboard 2 tấm chéo.
- Cỏ: InstancedMesh lá cỏ chỉ trong bán kính ~60 m quanh camera, sway bằng shader gió (WIND.uTime).
- Cau, dừa: tàu lá chia nhiều lá chét (texture alpha), thân có vòng đốt.

## 5. Nước
- Hồ, suối: dùng Water (examples/js/objects/Water.js) hoặc shader tự viết: normal map gợn 2 lớp trượt ngược chiều, phản chiếu PMREM theo Fresnel, màu sâu theo độ sâu (màu nông/sâu), bọt trắng ở mép.
- Suối chảy: trượt UV theo hướng dòng (đúng hướng thực tế), tốc độ theo độ dốc.
- Mạch nước nóng: hơi nước bằng soft particles, ăn sương khi lạnh buổi sáng.

## 6. Chi tiết kiến trúc từ ảnh
- Đếm cửa, ô kính, cột, máng xối từ ảnh thực tế; dựng bằng InstancedMesh.
- Biển hiệu, logo: canvas texture độ phân giải cao, anisotropy tối đa.
- Thêm chi tiết nhỏ có giá trị nhận diện: đèn đường, cột điện, bồn chứa, ống đứng, hàng rào.

## 7. Hiệu năng và mức chất lượng
- Đo: renderer.info (draw calls, tris), FPS trung bình 5 s.
- Mục tiêu: Thấp ≥ 30 fps trên điện thoại tầm trung, < 800k tam giác, < 150 draw calls; Cao không giới hạn tam giác nhưng ≥ 45 fps trên PC rời.
- Kỹ thuật: gộp hình học tĩnh (mergeBufferGeometries), InstancedMesh, LOD theo khoảng cách, frustum culling, pixelRatio min(devicePixelRatio, 1.5) ở Thấp/Vừa, tắt bóng vật nhỏ.
- Tự hạ chất lượng nếu FPS < 24 trong 3 s.

## 8. Kiểm chứng
- Render headless (Playwright + SwiftShader) cùng bộ góc máy chuẩn: tổng quan, nhà máy, hồ, giếng, mặt cắt, 3 khung flycam.
- So ảnh trước/sau, ghi FPS và tam giác vào bảng; không bàn giao nếu mức Thấp mất chức năng (nhãn, bảng, flycam).
- Kiểm tra cả chế độ xuyên thấu đất, mặt cắt và phóng đại đứng vẫn đúng sau khi thêm hậu kỳ.
