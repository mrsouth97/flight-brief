# CLAUDE.md — Flight Brief

Bối cảnh project cho Claude Code (chuyển từ ChatGPT/Codex ngày 23/09/2026). Đọc hết trước khi làm việc.

## Quy trình làm việc
- Trả lời bằng tiếng Việt, ngắn gọn. Người dùng nói yêu cầu tự nhiên, không cần prompt kỹ thuật.
- Thay đổi tối thiểu; không refactor/sửa phần không liên quan.
- Mỗi lần phát hành HTML: tăng version ở `<meta name="flight-brief-version">` và `<title>`, commit, push `main`. Báo người dùng bấm **Update Flight Brief** trên iPad.
- Thay đổi native (Swift) → build & cài lại lên iPad (iPad cắm vào Mac).
- Logic WIND / NOTAM / WEATHER / OFP / MEL: hỏi và chốt với người dùng trước khi code.

## App là gì
- App trên iPad giúp phi công (A350) tổng hợp tài liệu và chuẩn bị chuyến bay: OFP, WIND/Temp, Weather, NOTAM, (Loadsheet, MEL/CDL…).
- Khởi đầu từ MVP "Preflight Document Check" (checklist + thư viện PDF offline bằng IndexedDB), sau phát triển thành Flight Brief.
- Mục đích cá nhân, không thương mại.

## Kiến trúc: 2 phần
1. **Web — `index.html`** (repo `github.com/mrsouth97/flight-brief`, máy Mac: `~/Documents/GitHub/flight-brief`). Toàn bộ app một file: parser + màn hình OFP, WIND, WX, NOTAM…, và từ V5.91 cả **trang chủ** (danh sách Briefings).
   - Cập nhật: sửa → commit → push `main` (remote SSH) → trên iPad bấm **Update Flight Brief** là nhận bản mới, không cần Mac.
   - Version ghi ở `<meta name="flight-brief-version">` và `<title>Flight Brief V5.xx</title>`; tăng mỗi lần phát hành.
2. **Native iOS — Swift wrapper** (máy Mac: `~/Downloads/FlightBriefNativeRemote`, file chính `ContentView.swift`, WebView).
   - Lo quyền truy cập thư mục chuyến bay trong Files của iPad, đọc PDF, chuyển nội dung cho HTML. HTML gửi lựa chọn chuyến bay → Swift đọc PDF → trả về HTML. File không lên GitHub.
   - Sửa phần native thì phải build & cài lại lên iPad (Codex làm bằng dòng lệnh, iPad cắm vào Mac, không cần mở Xcode).
   - App tự chọn bản HTML mới hơn giữa bản cài sẵn và bản đã tải.
   - Icon app: pixel art máy bay + tài liệu, nền xanh, 1024×1024 (`Assets.xcassets/AppIcon.appiconset/AppIcon.png`).

## Quy tắc nghiệp vụ đã chốt (không tự đổi khi chưa được yêu cầu)
- **WIND:** in đậm gió lệch ≥ 30° hoặc 30 kt so với CFP; gió không đổi để chữ thường. Nhiệt độ lệch quá 5°C cũng in đậm (vd NINOP −51 / DOMET −55).
- **Step climb:** nếu F-PLN có step, phải kiểm tra gió đặt đúng ở waypoint đầu tiên sau step, ở cả FL ban đầu và FL step.
- Giao diện WIND: không khung viền; tên điểm, dòng dưới là FL in nghiêng (vd `T-O-C` / *FL380*).
- Bỏ tiêu đề lặp như "CFP Wind / Temp", "Weather Review", "NOTAM Review" ở từng trang; bỏ đánh số 1. 2. 3. trong OFP detail/WX nhưng giữ tiêu đề.
- **NOTAM:** UIR và FIR coi như nhau; phải có NOTAM cho các sân ở Đức và Baku (từng có lỗi bị thiếu).
- Thay đổi logic WIND / NOTAM / WEATHER / OFP / MEL phải chốt logic với người dùng trước khi code. Thay đổi UI thì làm thẳng.

## Lịch sử gần đây
- `ee3170f` Ignore macOS .DS_Store.
- Trang chủ: 10 chuyến mới nhất, cuộn tải thêm từng 10; giữ pull-to-refresh; refresh/quay lại app thì về 10 chuyến đầu.
- `4a328f7` (V5.90) Chuyển trang chủ từ Swift sang HTML.
- `7f4660e` (V5.91) Danh sách nằm trong **khung cố định cao đúng 10 dòng**, cuộn bên trong khung (infinite scroll), trang ngoài không dài ra. Thêm icon app.

## Minima (V5.93, thay cho Charts V5.92 đã gỡ)
- Chart là Lido trong app mLido (không có PDF) → minima **nhập tay** (nút "Approach minima"), lưu trên iPad trong file native `Application Support/FlightBrief/minima.json` (bridge `minimaList`/`minimaSave`, V5.97; web thường dùng IndexedDB `flight-brief-minima`). Mở từ menu ⋯ trang chủ hoặc nút Minima trong chuyến bay; chọn sân bằng chip phía trên (sân của chuyến + sân đã lưu + "+ Airport"). Form 3 ô: Approach (gõ tự do kèm runway, vd "ILS CAT2 11L", "VOR 11L", "CIRCLING") · DH/MDH · Minima (m, hoặc km nếu < 20, vd 2.4). App tự nhận runway và loại: CIRCL → Circling; CAT3 → CAT III; CAT2 → CAT II; ILS/GLS → CAT I; còn lại (RNP/RNAV nhập cột LNAV, LOC, VOR, NDB) → NPA. Hàng CAT C.
- Tab WEATHER hiện MINIMA OK / BELOW MINIMA / Not Found cho DEST, DEST ALT, ENR ALT, FUEL ERA, EDTO ALT (không cho DEP).
- TAF VIS so trực tiếp với R (không có R thì V); ceiling chỉ BKN/OVC, so với số dòng trên (DH/MDH/ceiling). Sự kiện TAF lọc bằng `fomEventApplicable` (FOM 8.1.2-6).
- DEST: approach thấp nhất trên runway OFP (Cat II/III nếu có). Precision chỉ so RVR; NPA/circling thêm ceiling.
- DEST ALTN / FUEL ERA / ENR ALTN: dùng **runway + approach trong ASC APRT của OFP** (dispatch đã hạ bậc), khớp theo loại (ILS/LOC/RNP/VOR/NDB/CAT2/CAT3), nhiều bản khớp thì lấy cao nhất; không khớp → approach cao nhất trên runway đó → cao nhất đã nhập. Sau đó áp bảng FOM 8.1.2-7: CAT II/III → RVR CAT I; CAT I → VIS NPA + CIG ≥ MDH (CAT I/NPA để hạ bậc phải cùng runway, không có → Not Found); NPA → +1000 m / +200 ft; Circling → circling. Sân Mỹ: 8.1.2-8 +400 ft / +1600 m.
- EDTO ALTN: bảng 5.1 EDTO Ops Manual (= FOM 8.5.1; Supplement cũ không dùng). OFP ghi CAT2 → 300 ft / 1200 m; CAT3 → 200 ft / RVR 550 m; approach khác → approach cao nhất trên runway OFP +400 ft / +1600 m. Không dùng mức 2 runway. TEMPO/PROB chỉ so với landing minima.
- FOM Rev 19 (18/06/2026) đã đối chiếu: 8.1.2/P10, P22 sửa không ảnh hưởng minima.
- Tài liệu hãng (FOM, SOP, EDTO manual, DGM, DGR, FCOM/MEL A350, LIDO GENPART, ICAO…) nằm trên **Google Drive** "Tài Liệu", đồng bộ về Mac: `~/Library/CloudStorage/GoogleDrive-mrsouth97@gmail.com/My Drive/Tài Liệu/` (đọc trực tiếp bằng pypdf, không giới hạn dung lượng). Lấy logic từ đó, ghi rõ mục tham chiếu; không chép tài liệu vào repo.

## DGR (V6.04)
- Tab **DGR** (V6.07): Step 1 header NOTOC so với OFP (tick). Mỗi dòng "Item N" có ô UN/ID + chọn PG (tuỳ chọn, lọc dòng theo PG; V6.08) → hiện tất cả tên của UN đó, nút ✕ ẩn tên không liên quan (↺ để hiện lại), nút **Select** chọn đúng tên theo NOTOC (V6.09; chỉ còn 1 tên thì tự chọn) → hiện thẻ xanh thông tin Blue Pages để phi công tự đối chiếu với NOTOC giấy (Class/sub risk C, PG E, EQ, Ltd Qty G/H, PAX I/J, CAO K/L, ERG N); nút SP bấm để xem nội dung DGR 4.4 (`sp` trong dgr.json, thiếu A1/A17/A117/A132/A176); kiểm tra DGM 2.5.8 → "NOT ACCEPTED ON VNA" đỏ (kể cả chỉ CAO). Trạng thái theo PG: chưa chọn PG mà chỉ PG I bị chặn → vàng "PG I NOT ACCEPTED — select PG". Tên có dữ liệu giống hệt được gộp 1 thẻ (V6.10). Khung **Same position check · Table 9.3.A** (V6.11, một khung, không có "add position"): tích các item xếp cùng vị trí → kết quả ngay trong khung (Can be loaded in the same position / SEGREGATE + bảng từng cặp). Summary còn: item bị chặn, ERG drill.
- Dữ liệu IATA DGR có bản quyền → **chỉ nằm trong app iPad**: `FlightBrief/Resources/dgr.json` (bridge `dgrData`, `window.FLIGHT_BRIEF_DGR_VERSION`). Tuyệt đối không đưa dữ liệu DGR vào `index.html`/GitHub (repo công khai).
- Nguồn: `~/Downloads/DANGEROUS GOODS REGULATIONS.pdf` (67th ed. 2026, OCR scan). Script tách: `~/Downloads/FlightBriefNativeRemote/tools/dgr/`. ~11% mục có cờ ⚠ (đọc OCR chưa chắc) → app nhắc kiểm tra trang DGR. Mỗi năm có DGR mới: chạy lại script, build & cài app.

## Cách làm việc người dùng muốn
- Nói tiếng Việt, yêu cầu tự nhiên, không cần prompt kỹ thuật.
- Thay đổi tối thiểu, không sửa phần không liên quan.
- Push lên `main` qua SSH (remote `git@github.com:mrsouth97/flight-brief.git`).
