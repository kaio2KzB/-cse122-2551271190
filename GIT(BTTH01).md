# PHÂN TÍCH BỐ CỤC TRANG CHỦ https://tlu.edu.vn/ (bản nén)

> Nguồn: nội dung văn bản trang chủ (đọc ngày 25/11/2026). **Chưa đọc được CSS thật / giao diện render.**
> - **[THẤY]** = suy ra trực tiếp từ nội dung, thứ tự trên trang
> - **[SUY LUẬN]** = kỹ thuật CSS đề xuất theo khuôn mẫu thường gặp, cần kiểm tra bằng DevTools (F12)
> - Trang dựng bằng Next.js nên DOM thật có thể nhiều lớp bọc hơn.

**Quy trình tư duy:** Nhìn giao diện → khoanh hộp → đặt tên → xác định cha/con → chọn Flex/Grid/Block/Position → nhận diện nội dung → chọn HTML.

---

## 1. Bản đồ bố cục toàn trang (chỉ dùng div)

```text
page
├── site-header
│   ├── header-top
│   │   ├── brand                       (logo)
│   │   └── header-actions
│   │       ├── search-box
│   │       └── language-switch         (VN | EN + cờ)
│   └── main-nav
│       └── nav-list
│           └── nav-item × 8            (+ nav-dropdown)
├── hero
│   └── hero-slider ─ hero-slide × 7    (5 ảnh + 2 video mp4)
├── main-content
│   ├── news-section
│   │   ├── news-main  ─ news-slider ─ news-card × 7
│   │   └── news-side  ─ notice-list (6 dòng) + event-slider ─ event-card × 8
│   ├── stats-section ─ stats-list ─ stat-item × 5
│   ├── highlights-section
│   │   ├── leader-photo
│   │   └── highlight-list ─ highlight-card × 4 (Tuyển sinh/Đào tạo/Sinh viên/Cựu sinh viên)
│   ├── units-section ─ units-list (11 đơn vị) + unit-detail
│   ├── feature-section × 3 (Hợp tác quốc tế / Nghiên cứu / Vì cộng đồng)
│   │   └── featured-article + news-list
│   ├── partners-section
│   └── gallery-section ─ gallery-grid ─ gallery-item × 13
└── site-footer
    ├── footer-columns ─ footer-contact + footer-col × 3 + footer-map
    └── footer-bottom
```

**Hai quan sát quan trọng [THẤY]:**
1. **Menu xuất hiện 2 lần**: menu ngang + một menu có "Tất cả GIỚI THIỆU", logo, bản quyền → rất giống **drawer mobile** (2 bố cục cho cùng dữ liệu).
2. **Các slider lặp nội dung** (7 tin xuất hiện 2 lần, sự kiện lặp nhiều lần) → nhiều khả năng slider vòng lặp nhân bản phần tử, hoặc bản desktop/mobile riêng.

**Quy tắc vàng:** nền chạy full-width, nội dung nằm trong `container { max-width; margin: 0 auto }`.

---

## 2. Phân tích từng component (Layout → CSS → Nội dung/HTML)

### 2.1 Header top
```text
site-header └── header-top ├── brand └── header-actions ├── search-box └── language-switch
```
```css
.header-top     { display: flex; justify-content: space-between; align-items: center; }
.header-actions { display: flex; align-items: center; gap: 16px; }   /* Flex lồng */
```
| Nội dung | HTML |
|---|---|
| Logo | `a > img` |
| "Tìm kiếm" | `button` (hoặc `form > input + button`) |
| VN/EN + cờ | `a` nếu đổi URL, `button` nếu đổi bằng JS (cần kiểm tra); cờ = `img` |

```html
<header class="site-header">
  <div class="header-top">
    <a class="brand" href="/"><img src="logo.webp" alt="Trường Đại học Thủy lợi"></a>
    <div class="header-actions">
      <button class="search-toggle" type="button">Tìm kiếm</button>
      <div class="language-switch">
        <a href="/en" lang="en"><img src="uk.webp" alt="English"></a>
        <a href="/" lang="vi"><img src="vn.webp" alt="Tiếng Việt"></a>
      </div>
    </div>
  </div>
</header>
```
**Kết luận:** brand + actions → Flex row space-between (lồng 1 Flex) → `header > a>img + button + a×2`.

### 2.2 Main nav (8 mục, có menu con)
```text
main-nav └── nav-list └── nav-item × 8 └── nav-dropdown
```
[THẤY] "eTLU" chỉ là nhãn không link → nhiều khả năng là nút mở menu.
```css
.nav-list     { display: flex; justify-content: space-between; list-style: none; }
.nav-item     { position: relative; }                 /* mốc */
.nav-dropdown { position: absolute; top: 100%; left: 0; display: none; }
.nav-item:hover .nav-dropdown { display: block; }
```
- Flex Container: `nav-list`; Flex Items: 8 `nav-item`.
- `nav-dropdown` là **Position**, không phải Flex; các link con xếp dọc theo **Normal Flow**.

| Nội dung | HTML |
|---|---|
| Khối menu | `nav` |
| 8 mục | `ul > li > a` |
| Menu con | `ul > li > a` lồng trong `li` cha |
| eTLU | `button` |

```html
<nav class="main-nav" aria-label="Menu chính">
  <ul class="nav-list">
    <li class="nav-item">
      <a href="/gioi-thieu">GIỚI THIỆU</a>
      <ul class="nav-dropdown">
        <li><a href="/co-cau-to-chuc">Cơ cấu tổ chức</a></li>
      </ul>
    </li>
    <li class="nav-item"><button type="button">eTLU</button><ul class="nav-dropdown">…</ul></li>
  </ul>
</nav>
```
**Kết luận:** Flex lo hàng ngang; Position lo menu thả.

### 2.3 Hero
```text
hero └── hero-slider └── hero-track └── hero-slide × 7
```
[SUY LUẬN] Có khả năng là slider/carousel, cần kiểm tra HTML/interaction.
```css
/* A: track ngang */  .hero-slider{overflow:hidden} .hero-track{display:flex} .hero-slide{flex:0 0 100%}
/* B: chồng lớp */    .hero-slider{position:relative} .hero-slide{position:absolute;inset:0;opacity:0}
```
Nếu chỉ 1 ảnh tĩnh: không cần Flex, `img { width:100% }`.
| Nội dung | HTML |
|---|---|
| Banner | `img` (alt hiện là tên file như "pl1" → nên viết alt mô tả) |
| Video | `video > source` |
| Slider | `section > ul > li` |

```html
<section class="hero" aria-label="Banner nổi bật">
  <ul class="hero-slider">
    <li class="hero-slide"><img src="20thang11.webp" alt="Kỷ niệm Ngày Nhà giáo Việt Nam 20/11"></li>
    <li class="hero-slide"><video src="1.mp4" muted autoplay loop></video></li>
  </ul>
</section>
```

### 2.4 News section (Tin tức + Thông báo + Sự kiện)
**Khung 2 cột** [SUY LUẬN]:
```css
.news-section { display: flex; gap: 24px; }
.news-main { flex: 2; }   /* tỷ lệ 2 : 1 */
.news-side { flex: 1; }
```
**News card** (lặp 7 lần):
```text
news-card ├── news-image └── news-body ├── news-category ├── news-title ├── news-excerpt └── news-date
```
```css
.news-card  { display: flex; flex-direction: column; gap: 8px; }  /* hoặc row: ảnh trái, chữ phải */
.news-image { flex: 0 0 120px; }  /* nếu row */
.news-body  { flex: 1; }          /* bên trong: Normal Flow */
```
| Nội dung | HTML |
|---|---|
| Ảnh | `a > img` |
| "Tin tức chung" | `span` (hoặc `a`) |
| "Hot" | `span` |
| Tiêu đề | `h3 > a` |
| Mô tả | `p` |
| Ngày | `time datetime="2026-11-25"` |

```html
<article class="news-card">
  <a class="news-image" href="/bai-viet"><img src="…" alt="Tiêu đề bài"></a>
  <div class="news-body">
    <span class="news-category">Tin tức chung</span>
    <h3 class="news-title"><a href="/bai-viet">Tiêu đề</a></h3>
    <p class="news-excerpt">Căn cứ nhu cầu giảng dạy…</p>
    <time datetime="2026-11-25">25/11/2026</time>
  </div>
</article>
```
**Slider (nút ‹ ›):**
```text
news-slider ├── slider-prev └── slider-viewport └── slider-track └── news-card × N └── slider-next
```
```css
.slider-viewport { overflow: hidden; }
.slider-track    { display: flex; gap: 16px; transition: transform .4s; }
.news-card       { flex: 0 0 calc(100% / 3); }
```
Nút ‹ › = `button` (thao tác, không điều hướng).

**Notice list "Thông Báo"** [THẤY] 6 dòng = link + ngày → chỉ cần **Normal Flow** (`ul > li > a + time`). Muốn ngày căn phải mới thêm `flex; justify-content: space-between`.

**Event card** [THẤY] ảnh + ngày "19 / Tháng 11" + tiêu đề:
```css
.event-image { position: relative; }
.event-date  { position: absolute; top: 8px; left: 8px; display: flex; flex-direction: column; align-items: center; }
```
→ Position để đè lên ảnh + Flex để xếp ngày trên tháng.
```html
<article class="event-card">
  <a href="…"><img src="…" alt="…"><time class="event-date" datetime="2026-11-19"><strong>19</strong><span>Tháng 11</span></time></a>
  <h3><a href="…">Trực tiếp Lễ kỷ niệm…</a></h3>
</article>
```
**Kết luận:** Flex 2:1 → Flex track (slider) → Flex card; nội dung: `section > article > img, h3>a, p, time`; `ul>li>a` cho thông báo.

### 2.5 Stats ("Những Con Số Ấn Tượng")
[THẤY] 5 nhãn: Người học · Đối tác trong & ngoài nước · SV tốt nghiệp có việc làm · Ngành học · Giảng viên Tiến sĩ. **Các con số không có trong dữ liệu** (nhiều khả năng đếm số bằng JS).
```text
stats-section ├── section-heading └── stats-list └── stat-item × 5 ├── stat-number └── stat-label
```
```css
.stats-list { display: flex; justify-content: space-between; gap: 24px; flex-wrap: wrap; }
.stat-item  { flex: 1 1 180px; text-align: center; }
/* hoặc Grid: repeat(auto-fit, minmax(180px, 1fr)) – đều hơn khi responsive */
```
```html
<section class="stats-section">
  <h2>Những Con Số Ấn Tượng</h2>
  <ul class="stats-list"><li class="stat-item"><strong class="stat-number">…</strong><span class="stat-label">Ngành học</span></li></ul>
</section>
```
(Có thể dùng `dl/dt/dd` cho cặp nhãn–số.) **Kết luận:** Flex wrap (hoặc Grid auto-fit) → `section > h2 + ul>li>strong+span`.

### 2.6 Highlights (Tuyển sinh · Đào tạo · Sinh viên · Cựu sinh viên)
```text
highlights-section ├── leader-photo (ảnh GS. Nguyễn Trung Việt, alt="Trang Chủ")
                   └── highlight-list └── highlight-card × 4
                         (Tuyển sinh: heading, text, ul 3 link, nút "Đăng ký ngay"
                          3 khối còn lại: heading + bài nổi bật ảnh/link/mô tả)
```
[SUY LUẬN] Vị trí/vai trò của ảnh lãnh đạo chưa rõ, cần xem giao diện thật.
```css
.highlight-list   { display: grid; grid-template-columns: repeat(2, 1fr); gap: 24px; }  /* 4 thẻ đều → Grid */
.highlight-card   { display: flex; flex-direction: column; }
.highlight-button { margin-top: auto; }   /* đẩy nút xuống đáy */
```
| Nội dung | HTML |
|---|---|
| Tên khối | `h2 > a` |
| Mô tả | `p` |
| ›Đại học / ›Thạc sĩ / ›Tiến sĩ | `ul > li > a` (dấu › tạo bằng `::before`) |
| "Đăng ký ngay" | `a` (dẫn sang ts.tlu.edu.vn), tạo kiểu như nút |
| Bài nổi bật | `article` |

> **Link vs Button:** đi tới trang khác → `a`; làm gì đó ngay trên trang → `button`.

### 2.7 Units ("Các đơn vị đào tạo, KHCN")
[THẤY] 11 đơn vị (mỗi cái có website riêng) + chỉ hiện mô tả Khoa Kinh tế và Quản lý → khớp kiểu tab/hover [SUY LUẬN].
```text
units-section ├── section-heading ├── units-list └── unit-detail ├── unit-description └── unit-link
```
```css
.units-section { display: flex; gap: 32px; }
.units-list    { flex: 1; }   /* tỷ lệ ước lượng 1 : 2 ; bên trong Normal Flow */
.unit-detail   { flex: 2; }
```
```html
<section class="units-section">
  <h2>Các đơn vị đào tạo, KHCN</h2>
  <ul class="units-list"><li><a href="https://fem.tlu.edu.vn/">Khoa Kinh tế và Quản lý</a></li>…</ul>
  <div class="unit-detail"><p>Với hơn 45 năm xây dựng và phát triển…</p><a href="https://fem.tlu.edu.vn/">Khám phá Khoa Kinh tế và Quản lý</a></div>
</section>
```
Nếu là tab đổi nội dung bằng JS: `button role="tab"`; vì mỗi mục có URL ngoài nên `a` hợp lý hơn.

### 2.8 Feature sections ×3 (Hợp tác quốc tế · Nghiên cứu · Vì cộng đồng)
**Cùng một khuôn mẫu → nên đặt tên dùng chung.**
```text
feature-section ├── section-heading (link) ├── featured-article (ảnh lớn + danh mục + tiêu đề + ngày)
                └── news-list └── news-item × 5–6 (danh mục + tiêu đề + ngày)
```
```css
.feature-section  { display: flex; gap: 24px; }
.featured-article { flex: 3; }    /* 3 : 2 (ước lượng) */
.news-list        { flex: 2; }    /* bên trong: Normal Flow, dùng border-bottom */
```
HTML: `section > h2>a + article + ul>li`. Modifier: `feature-section--research`, `--community`.

### 2.9 Partners ("Mạng Lưới Đối Tác")
[THẤY] chỉ có tiêu đề + đoạn mô tả; **logo không có trong dữ liệu** (có khả năng băng logo cuộn hoặc lưới, cần kiểm tra).
```css
.partners-logos { display: flex; flex-wrap: wrap; justify-content: center; align-items: center; gap: 24px; }
```
HTML: `section > h2 + p + ul>li>img`.

### 2.10 Gallery ("Khoảnh Khắc TLU")
[THẤY] 13 ảnh, không có chú thích riêng (alt = tiêu đề bài).
```css
.gallery-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }   /* cần thẳng hàng/cột → Grid */
```
HTML: `section > h2 + ul>li>a>img`. Chỉ dùng `figure/figcaption` khi chú thích thật sự hiển thị. Nếu thực tế là slider/masonry thì đổi kỹ thuật.

### 2.11 Footer
```text
site-footer ├── footer-columns ├── footer-contact (logo, địa chỉ, email, điện thoại, social-links)
                              ├── footer-col "Giới thiệu" (4 link)
                              ├── footer-col "Truy cập nhanh" (5 link)
                              ├── footer-col "Theo dõi TLU" (4 link)
                              └── footer-map ("Tải bản đồ tương tác")
            └── footer-bottom (bản quyền + đơn vị phát triển)
```
```css
.footer-columns { display: flex; justify-content: space-between; gap: 32px; flex-wrap: wrap; }
.footer-contact { flex: 2; }
.footer-col     { flex: 1; }
.social-links   { display: flex; gap: 12px; }   /* Flex lồng */
.footer-bottom  { text-align: center; }          /* Normal Flow */
```
```html
<footer class="site-footer">
  <div class="footer-columns">
    <address class="footer-contact">
      <img src="logo.webp" alt="Trường Đại học Thủy lợi">
      <p>175 Tây Sơn - P. Kim Liên - Hà Nội</p>
      <a href="mailto:…">Email</a> <a href="tel:02438522201">(024) 38522201</a>
    </address>
    <nav class="footer-col" aria-label="Giới thiệu"><h3>Giới thiệu</h3><ul><li><a href="/gioi-thieu">Tổng quan</a></li></ul></nav>
  </div>
  <p class="footer-bottom"><small>Bản quyền © 2026 Trường Đại học Thủy lợi</small></p>
</footer>
```
Icon mạng xã hội: `ul > li > a > img/svg` + `aria-label`. "Tải bản đồ tương tác": `button` nếu tải trên trang, `a` nếu mở trang khác.

---

## 3. Bản đồ Flexbox toàn trang

```text
PAGE
├── HEADER  └── header-top [FLEX space-between] └── header-actions [FLEX]  ← lồng
├── NAV     └── nav-list [FLEX] └── nav-item (relative) └── nav-dropdown (absolute, KHÔNG Flex)
├── HERO    └── hero-track [FLEX nowrap] (nếu slider ngang)
├── MAIN
│   ├── news-section [FLEX 2:1]
│   │   ├── slider-track [FLEX nowrap] └── news-card [FLEX column/row]   ← lồng 3 cấp
│   │   ├── notice-list (Normal Flow)
│   │   └── event-card > event-date [FLEX column + absolute]
│   ├── stats-list [FLEX wrap | Grid]
│   ├── highlight-list [GRID 2 cột] └── highlight-card [FLEX column]
│   ├── units-section [FLEX 1:2] └── units-list (Normal Flow)
│   ├── feature-section ×3 [FLEX 3:2] └── news-list (Normal Flow)
│   ├── partners-logos [FLEX wrap center]
│   └── gallery-grid [GRID 4 cột]
└── FOOTER  └── footer-columns [FLEX wrap] ├── flex:2 + flex:1 ×3 └── social-links [FLEX]
```
**KHÔNG cần Flex:** `nav-dropdown`, `notice-list`, `news-body`, `news-list`, `units-list`, `footer-col` bên trong, `footer-bottom` (đã xếp dọc theo Normal Flow).

---

## 4. Nhận diện Grid

| Vùng | Flex được không? | Grid phù hợp hơn? | Chọn |
|---|---|---|---|
| gallery 13 ảnh | Được nhưng hàng cuối lệch | **Có** (cần hàng × cột) | **Grid** `repeat(4,1fr)` |
| highlight 4 thẻ đều | Được | **Có** (2×2) | **Grid** `repeat(2,1fr)` |
| stats 5 ô, 1 hàng | **Được** | Tốt hơn khi responsive | Flex hoặc Grid `auto-fit` |
| news-section 2:1 | **Được** (một chiều) | Không cần | **Flex** |
| nav-list, footer-columns | **Được** | Không cần | **Flex** |

> Một hàng/một cột → **Flex**. Cả hàng lẫn cột thẳng nhau → **Grid**.

---

## 5. Responsive (dự kiến, [SUY LUẬN]; chỉ có drawer menu là bằng chứng trực tiếp)

| Vùng | Desktop | Tablet | Mobile |
|---|---|---|---|
| Nav | 8 mục ngang | thu gọn | ☰ + menu trượt |
| news-section | `Main \| Side` | hẹp hơn | xếp dọc |
| news slider | 3 thẻ | 2 thẻ | 1 thẻ |
| stats | `A B C D E` | `A B C / D E` | 1 cột |
| highlight | 2×2 | 2×2 | 1 cột |
| units | `List \| Detail` | giữ | List rồi Detail |
| gallery | 4 cột | 3 cột | 2 cột |
| footer | 4–5 cột | 2–3 cột | 1 cột |

```css
.stats-list { display: flex; flex-wrap: wrap; }
@media (max-width: 768px) { .news-section, .units-section, .feature-section { flex-direction: column; } }
.gallery-grid { grid-template-columns: repeat(4, 1fr); }
@media (max-width: 992px) { .gallery-grid { grid-template-columns: repeat(3, 1fr); } }
@media (max-width: 576px) { .gallery-grid { grid-template-columns: repeat(2, 1fr); } }
```

---

## 6. DOM tree semantic toàn trang

```text
body
├── header.site-header ─ div.header-top (a.brand>img, div.header-actions) + nav.main-nav>ul>li×8>a+ul
├── main
│   ├── section.hero ─ ul>li>img|video
│   ├── section.news-section ─ div.news-main (h2>a, article.news-card×7) + aside.news-side (section.notice-list, section.event-slider)
│   ├── section.stats-section ─ h2 + ul>li>strong+span
│   ├── section.highlights-section ─ section.highlight-card × 4
│   ├── section.units-section ─ h2 + ul>li>a + div.unit-detail
│   ├── section.feature-section × 3 ─ h2>a + article + ul>li
│   ├── section.partners-section ─ h2 + p + ul>li>img
│   └── section.gallery-section ─ h2 + ul>li>a>img
└── footer.site-footer ─ address + nav×3 + p>small
```
Chỉ **một** `h1` toàn trang (cần kiểm tra ở header/hero); tiêu đề khối `h2`, tiêu đề thẻ `h3`.

---

## 7. Bảng tổng hợp

| Component | Layout Box | Technique | Flex Container? | Flex Items | Nội dung | HTML semantic |
|---|---|---|---|---|---|---|
| Header top | `header-top` | Flex (lồng) | Có | brand, actions | Logo, tìm kiếm, ngôn ngữ | `header`, `a>img`, `button` |
| Main nav | `nav-list` | Flex + Position | Có | `nav-item` | Menu 8 mục | `nav`, `ul`, `li`, `a` |
| Nav dropdown | `nav-dropdown` | Position absolute | Không | - | Menu con | `ul`, `li`, `a` |
| Hero | `hero-slider` | Flex track / Position | Có (nếu track) | `hero-slide` | Ảnh, video | `section`, `img`, `video` |
| News section | `news-section` | Flex 2:1 | Có | main, side | Tin/thông báo/sự kiện | `section`, `aside` |
| News card | `news-card` | Flex | Có | image, body | Bài viết | `article`, `img`, `h3`, `p`, `time` |
| Notice list | `notice-list` | Normal Flow | Không | - | Thông báo + ngày | `ul`, `li`, `a`, `time` |
| Event card | `event-card` | Position + Flex | Có (ngày) | day, month | Sự kiện | `article`, `time`, `h3` |
| Stats | `stats-list` | Flex wrap / Grid | Có | `stat-item` | Số liệu | `ul`, `li`, `strong`, `span` |
| Highlights | `highlight-list` | Grid + Flex column | Có (trong card) | nội dung card | Giới thiệu khối | `section`, `h2`, `p`, `ul`, `a` |
| Units | `units-section` | Flex 1:2 | Có | list, detail | 11 đơn vị | `section`, `ul`, `li`, `a`, `p` |
| Feature ×3 | `feature-section` | Flex 3:2 | Có | featured, list | Tin theo chủ đề | `section`, `article`, `ul` |
| Partners | `partners-logos` | Flex wrap | Có | logo | Logo đối tác | `ul`, `li`, `img` |
| Gallery | `gallery-grid` | **Grid** | Không | - | Ảnh | `ul`, `li`, `a`, `img` |
| Footer | `footer-columns` | Flex wrap | Có | `footer-col` | Liên hệ, link | `footer`, `address`, `nav`, `small` |

---

## 8. Checklist tư duy cho sinh viên

```text
Nhìn thấy một vùng giao diện
        ↓
Khoanh hộp → Đặt tên theo VAI TRÒ → Xác định cha – con
        ↓
Các con cần đứng NGANG hàng?
   ├── Có → Flexbox (tràn → flex-wrap; chia tỷ lệ → flex: 2 / flex: 1)
   └── Không
        ↓
   Có cấu trúc hàng × cột đều?
   ├── Có → Grid
   └── Không
        ↓
   Có phần tử đè / thả xuống?
   ├── Có → Position (relative làm mốc, absolute để đặt)
   └── Không → Normal Flow (để yên!)
        ↓
Xác định kiểu nội dung → Chọn HTML semantic
```
**Hỏi nhanh mỗi hộp:** chứa gì? xếp ngang hay dọc? bấm đi đâu (`a`) hay làm gì (`button`)? có độc lập không (`article`)? có phải ngày giờ (`time`)?

**Đặt tên class:** theo vai trò, nhất quán: `site-header`, `main-nav`, `nav-list`, `nav-item`, `news-card`, `feature-section`, `footer-col`. Tránh `box1`, `left-area`, `blue-box`.

---

## 9. Lỗi tư duy phổ biến

| Lỗi | Cách đúng |
|---|---|
| `display: flex` cho mọi thứ | Chỉ dùng khi cần xếp ngang/căn chỉnh |
| Coi menu thả là Flex | Menu thả = Position (`relative` cha + `absolute` con) |
| Flex-wrap cho lưới ảnh, hàng cuối lệch | Dùng Grid |
| Nhầm Container với Item | Chỉ **con trực tiếp** của `display:flex` mới là Flex Item |
| `flex: 2` trên phần tử không nằm trong Flex Container | Kiểm tra cha đã `display: flex` chưa |
| `button` cho link sang trang khác ("Đăng ký ngay") | Dùng `a`, tạo kiểu như nút |
| `a` cho nút ‹ › slider | Dùng `button` |
| Mọi thứ là `div` | `article`, `time`, `nav`, `h2/h3`, `address`… |
| Chọn tiêu đề theo cỡ chữ | Cấp `h1–h6` theo thứ bậc nội dung, chỉnh cỡ chữ bằng CSS |
| `alt="pl1"` (tên file) | `alt` mô tả nội dung ảnh |
| Coi bản phân tích này là CSS thật | Đối chiếu bằng DevTools (Inspect → xem `display`, `flex`, `grid-template-columns`) |

---

## 10. Cần kiểm tra bằng DevTools (các điểm [SUY LUẬN])

1. Hero có thật là slider không; track dùng `transform` hay `opacity`?
2. `news-section` có đúng 2 cột 2:1 không?
3. Khối highlights sắp xếp thế nào; ảnh GS. Nguyễn Trung Việt nằm ở đâu?
4. Khối "Các đơn vị" có phải tab/hover không?
5. Gallery là Grid, masonry hay slider?
6. Số liệu thống kê hiển thị thế nào (đếm số bằng JS?); logo đối tác hiển thị thế nào?
7. Slider có nhân bản phần tử, hay có bản desktop/mobile riêng?
