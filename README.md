# Claude 3D Skills – Bộ skill dựng mô hình 3D địa chất, cảnh quan, nhà máy

Bộ 4 skill cho [Claude](https://claude.ai) giúp dựng **mô hình 3D tương tác đúng tỷ lệ thật** bằng Three.js trong **một file HTML**: địa hình và địa chất từ DEM, sông suối và bầu trời, nhà xưởng từ bản vẽ CAD, và lớp đồ họa điện ảnh.

Được đúc kết từ dự án thật: mô hình 3D địa chất – thủy văn một mỏ nước khoáng nóng và nhà máy đóng chai tại Khánh Hòa, Việt Nam.

| Skill | Dùng khi |
| --- | --- |
| [`dia-hinh-dia-chat-3d`](skills/dia-hinh-dia-chat-3d/SKILL.md) | Dựng địa hình, núi, địa tầng, lỗ khoan, nước ngầm đúng tỷ lệ từ DEM, bản đồ VN-2000, lỗ khoan; mặt cắt, block diagram |
| [`canh-quan-nuoc-bau-troi-3d`](skills/canh-quan-nuoc-bau-troi-3d/SKILL.md) | Sông suối chảy đúng hướng, mặt nước, cây cỏ, mặt trời theo vị trí thật, mây, sương, mưa; kiểm chứng bằng render headless |
| [`kien-truc-cad-3d`](skills/kien-truc-cad-3d/SKILL.md) | Đọc DWG/DXF (không cần AutoCAD) và dựng nhà xưởng, mái, đường, cầu, rào, tuyến ống đúng kích thước, đối chiếu ảnh thực tế |
| [`do-hoa-3d-dien-anh`](skills/do-hoa-3d-dien-anh/SKILL.md) | Làm đẹp cảnh đã dựng: hậu kỳ màu/ánh sáng, vật liệu PBR, tán lá chi tiết, nước phản chiếu, 3 mức chất lượng cho điện thoại – laptop – máy chiếu |

Bốn skill bổ trợ nhau: dựng hình bằng 3 skill đầu, làm đẹp bằng skill thứ tư.

## Cài đặt

**Claude (web / desktop):** bấm **Code → Download ZIP** trên trang này, giải nén, nén riêng thư mục của skill cần dùng trong `skills/` thành file `.zip`, rồi tải lên mục **Skills** trong phần cài đặt của Claude.

**Claude Code:** chép thư mục skill vào `~/.claude/skills/`:

```bash
git clone https://github.com/pnd979-cloud/claude-3d-skills.git
cp -r claude-3d-skills/skills/* ~/.claude/skills/
```

## Nguyên tắc chung của bộ skill

- **Đúng trước, đẹp sau:** tỷ lệ 1 đơn vị = 1 m, tọa độ thật; không đổi số liệu để có hình đẹp.
- **Một file HTML** chạy trên trình duyệt, thư viện từ CDN có khóa phiên bản (Three.js r128).
- **Kiểm chứng bằng ảnh:** render headless (Playwright + SwiftShader) trước khi bàn giao.
- **Trung thực về nguồn:** phân biệt số liệu có nguồn và phần diễn giải.

---

## English

Four [Claude](https://claude.ai) skills for building **true-scale interactive 3D models** with Three.js in a **single HTML file**: terrain & geology from DEMs, rivers & skies, factories from CAD drawings, and a cinematic graphics layer. Skill instructions are written in Vietnamese. Distilled from a real project: a 3D hydrogeological model of a hot mineral-water spring and its bottling plant in Khanh Hoa, Vietnam.

Install: download the repo (Code → Download ZIP), zip the skill folder you need from `skills/` and upload it to the Skills section of Claude's settings, or copy the folders in `skills/` to `~/.claude/skills/` for Claude Code.

## Giấy phép / License

[MIT](LICENSE) © 2026 Phạm Ngọc Duy
