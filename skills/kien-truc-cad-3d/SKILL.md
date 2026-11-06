---
name: "kien-truc-cad-3d"
description: "Đọc bản vẽ CAD (DWG/DXF) và dựng công trình kiến trúc 3D đúng kích thước bằng Three.js — nhà xưởng, mái, đường, cầu, rào, tuyến ống — đối chiếu ảnh thực tế. Dùng khi có file .dwg/.dxf hoặc cần dựng nhà máy 3D."
---

# Kiến trúc 3D từ bản vẽ CAD

## 1. Đọc DWG trong sandbox (không cần AutoCAD)
```bash
mkdir -p ~/dwgtool && cd ~/dwgtool && npm init -y && npm i @mlightcad/libredwg-web three@0.128.0
```
```js
// read.mjs — node read.mjs in.dwg out.json
import { Dwg_File_Type, LibreDwg } from '@mlightcad/libredwg-web'; import fs from 'fs';
const lib = await LibreDwg.create('./node_modules/@mlightcad/libredwg-web/wasm/');
const dwg = lib.dwg_read_data(new Uint8Array(fs.readFileSync(process.argv[2])), Dwg_File_Type.DWG);
const db = lib.convert(dwg); lib.dwg_free(dwg);
fs.writeFileSync(process.argv[3], JSON.stringify(db, (k, v) => typeof v === 'bigint' ? Number(v) : v));
```
- Mã lỗi 64 khi mở thường là cảnh báo không nghiêm trọng — vẫn kiểm tra số entity.
- File lớn: ghi NDJSON từng entity thay vì một JSON khổng lồ.
- DXF: `pip install ezdxf --break-system-packages`.

## 2. Quy trình bắt buộc trước khi dựng
1. Kiểm kê layer: đếm `type|layer`, in vài TEXT mỗi layer để hiểu ý nghĩa.
2. Vẽ lại mặt bằng ra PNG (matplotlib) và XEM ảnh để xác nhận đúng bản vẽ, đúng đơn vị (`INSUNITS` 4 = mm, 6 = m).
3. Đọc bảng tọa độ, khung tên, tỷ lệ, ngày đo; ghi nhật ký nguồn.
4. Bản vẽ quy hoạch thường không có tọa độ thật: nắn về VN-2000 bằng phép similarity theo các mốc ranh chung, báo hệ số tỷ lệ, góc xoay, RMS.
5. Trích đối tượng ra GeoJSON cục bộ (mét, gốc chung với mô hình địa hình).

## 3. Hình học CAD
- LWPOLYLINE có `bulge`: θ = 4·atan(bulge), bulge > 0 là cung ngược chiều kim đồng hồ; bán kính r = c / (2·sin(θ/2)).
- INSERT (block): điểm = insertion + R(rotation)·(scale·điểm_block); block lồng nhau thì nhân ma trận dần.
- Nhà vẽ bằng LINE rời: ghép đa giác bằng `shapely.ops.polygonize` (snap 0.05 m trước).
- Diện tích footprint so với diện tích ghi trên bản vẽ, lệch > 3 % thì xem lại.

## 4. Dựng công trình trong Three.js (r128)
- Footprint → `THREE.Shape` (Đông, Bắc) → `ExtrudeGeometry({depth: chiều_cao, bevelEnabled:false})` → `rotateX(−Math.PI/2)`; `position.y` = cốt nền.
- Chiều cao lấy từ mặt cắt/mặt đứng; không có thì ước tính (nhà xưởng khung thép: diềm 7–10 m, mái dốc 10–15 %) và ghi `source: 'ước tính'`.
- Mái 2 mái / 4 mái / 1 mái có độ vươn 0.5–1.0 m; tôn sóng bằng texture canvas.
- Chi tiết nhỏ (cửa, ô kính) bằng texture; chỉ dựng khối cho chi tiết > 1 m.
- Tuyến: đường (ribbon bề rộng thật), cầu (bản mặt + trụ), rào (InstancedMesh), ống (TubeGeometry trên mố đỡ).
- Hiệu năng: gộp geometry theo vật liệu, InstancedMesh; mục tiêu < 300 draw call.

## 5. Đối chiếu ảnh thực tế
- EXIF tiêu cự quy đổi 35 mm f → `vFOV = 2·atan(24 / (2f))`. Đặt camera ở vị trí chụp ước tính, chồng ảnh thật 50 % để chỉnh vị trí/chiều cao.

## 6. Trung thực
- Hiện trạng theo bản vẽ có ngày tháng; phương án quy hoạch để lớp riêng, nhãn "QUY HOẠCH".
- Không thêm công trình không có trong bản vẽ/ảnh.
