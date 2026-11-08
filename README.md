# Nguyên tắc và Quy trình phát triển Theme Shopify

## 0. Hướng dẫn sử dụng tài liệu

Tài liệu gồm hai lớp **bổ trợ cho nhau**, không phải hai quy trình độc lập hay thay thế nhau:

- Phần I - Nguyên tắc: Phương pháp luận (Design Thinking, 5W1H), định hướng kỹ thuật HTML/CSS/JS, cách chọn đơn vị kiến trúc và thẻ tài sản, bộ công cụ kiểm tra.
- Phần II - Quy trình thực hiện: 7 bước từ tiếp nhận yêu cầu đến release/rollback
- Phần III - Quy ước: Đặt tên, nhánh Git, commit và PR
  |

Phần I là "luật chơi" được áp dụng **bên trong** từng bước của Phần II. Phần III là tài liệu tra cứu dùng xuyên suốt. Bảng dưới cho thấy mỗi giai đoạn Design Thinking được hiện thực hóa ở bước nào của quy trình:

| Giai đoạn Design Thinking | Bước quy trình (Phần II)              | Nguyên tắc áp dụng (Phần I) | Đầu ra                                |
| ------------------------- | ------------------------------------- | --------------------------- | ------------------------------------- |
| Empathize                 | Bước 1 — Tiếp nhận và làm rõ yêu cầu  | I.1 (5W1H)                  | Phiếu yêu cầu (GitHub Issue)          |
| Define                    | Bước 2 — Tiêu chí chấp nhận           | I.1                         | Danh sách AC                          |
| Ideate                    | Bước 3 — Chọn giải pháp và chia task  | I.2, I.3, I.4               | Quyết định kiến trúc + danh sách task |
| Prototype                 | Bước 4 — Code, commit, mở PR          | I.2, Phần III               | Pull Request                          |
| Test                      | Bước 5, 6 — Review/merge, QA, hồi quy | I.5                         | Bản đã qua QA trên Preview            |
| (Bàn giao)                | Bước 7 — Release và rollback          | —                           | Bản chạy trên Production              |

Vì Design Thinking là phi tuyến tính: khi bước Test phát hiện yêu cầu hiểu sai thì quay lại Bước 1–2; khi phát hiện giải pháp không đạt thì quay lại Bước 3.

---

# PHẦN I — NGUYÊN TẮC

## I.1 Phương pháp luận

### Design Thinking

Phương pháp luận dùng để xây dựng codebase là **Design Thinking** — gồm 5 giai đoạn, không bắt buộc đi theo đường thẳng, cho phép lặp lại để cải tiến liên tục:

- **Empathize** — Nghiên cứu, khai thác thông tin về yêu cầu, người dùng và bối cảnh từ Designer để hiểu sâu sắc bối cảnh và người dùng.
- **Define** — Tái định nghĩa vấn đề thực tế mà người dùng gặp phải, trong bối cảnh sử dụng và trong giới hạn của nền tảng Shopify.
- **Ideate** — Đưa ra các phương án giải quyết khả thi trên nền tảng Shopify và so sánh ưu/nhược.
- **Prototype** — Dựng giải pháp chạy được trên theme, bắt đầu từ bản tối giản (khung markup + style) rồi hoàn thiện dần để đáp ứng được các tiêu chí chấp nhận của các yêu cầu.
- **Test** — Kiểm tra lại theo yêu cầu của Designer và các yêu cầu về Performance, Accessibility (A11y) và SEO của nền tảng. Kết quả Test là đầu vào cho vòng lặp tiếp theo.

### 5W1H — công cụ của giai đoạn Empathize/Define

Hai nhóm người dùng chính cần khai thác là **Merchant** (người cấu hình theme trong theme editor) và **người mua hàng**. Bên cạnh đó là các ràng buộc kỹ thuật từ nền tảng Shopify về performance, A11y và SEO. Thứ tự hỏi được sắp theo logic "hiểu vấn đề trước, tìm cách làm sau":

| Câu hỏi               | Khai thác                                            | Ví dụ câu hỏi cụ thể cho theme                                                                               |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Why** (Tại sao)     | Mục đích, vấn đề cần giải quyết, tiêu chí thành công | Merchant/khách đang gặp vấn đề gì? Làm xong thì đo bằng gì?                                                  |
| **Who** (Ai)          | Đối tượng thụ hưởng, đặc điểm, thói quen             | Merchant cần tự đổi được gì trong editor? Khách dùng thiết bị gì, có dùng bàn phím/trình đọc màn hình không? |
| **What** (Cái gì)     | Nội dung yêu cầu: thành phần, dữ liệu, setting       | Hiển thị những phần tử nào? Dữ liệu lấy từ đâu (product, metafield, metaobject)?                             |
| **Where** (Ở đâu)     | Nơi yêu cầu xuất hiện và bối cảnh                    | Template nào? Section group (header/footer)? Mobile hay desktop? Trong theme editor có hành vi khác không?   |
| **When** (Khi nào)    | Thời điểm/trạng thái kích hoạt, mốc thời gian        | Xuất hiện khi nào (chọn variant, hết hàng, sau khi thêm vào giỏ)? Hạn bàn giao?                              |
| **How** (Như thế nào) | Hướng thực hiện sơ bộ và ràng buộc nền tảng          | Giải pháp dự kiến? Có đụng giới hạn nền tảng không (ví dụ Liquid chỉ trả tối đa 250 variant)?                |

## I.2 Định hướng kỹ thuật

Định hướng: xây dựng theme hiện đại theo hướng **Theme block**, dùng **HTML và CSS là chính**, **JavaScript chỉ cho những tác vụ thực sự cần**.

**HTML** — Dùng đúng ngữ nghĩa của element để tận dụng tối đa hành vi có sẵn của trình duyệt, qua đó đáp ứng A11y, SEO và Performance mà không cần can thiệp nhiều bằng JavaScript. Ví dụ: `<button>` cho hành động và `<a>` cho điều hướng (không dùng `<div>` bắt click); `<details>/<summary>` cho accordion; `<dialog>` với `showModal()` cho modal (trình duyệt tự xử lý phím Esc và vô hiệu hóa phần nền); `<label>` gắn với từng trường form.

**CSS** — Ưu tiên để hiệu ứng chuyển động chạy trên **Compositor thread** để vẫn mượt khi main thread đang bận. Cụ thể:

- Animation/transition ưu tiên dùng `transform` và `opacity` — đây là nhóm thuộc tính có thể được xử lý hoàn toàn ở compositor khi được khai báo bằng CSS (hoặc Web Animations API).
- Tránh animate các thuộc tính gây tính lại layout (`width`, `height`, `top`, `left`, `margin`, `padding`...).
- Các thuộc tính chỉ gây vẽ lại (`color`, `background-color`, `box-shadow`...) vẫn chạy trên main thread; chấp nhận được cho hiệu ứng ngắn, vùng nhỏ như hover, nhưng không phải compositor-only.
- Mọi chuyển động phải tôn trọng `prefers-reduced-motion`.

**JavaScript** — Hạn chế, chỉ dùng khi HTML/CSS không làm được. Khi dùng thì đóng gói hành vi trong **custom element**: `connectedCallback` tự chạy lại mỗi khi element được chèn vào DOM, nên hành vi vẫn đúng khi theme editor nạp lại section hoặc khi Section Rendering API trả markup mới. Dọn dẹp listener trong `disconnectedCallback`.

## I.3 Chọn đơn vị kiến trúc

Với mỗi section cần đánh giá dùng theme block hay section kiểu cũ có lợi hơn. Có **4 đơn vị kiến trúc**:

| Đơn vị                          | Mô tả                                                                                                                      | Merchant thấy/sắp xếp được? |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| **Section nguyên khối**         | Section không khai block nào; mọi thứ cứng trong file                                                                      | Chỉ đổi được setting        |
| **Section block** (local block) | Khai trong mảng `blocks` của `{% schema %}` của chính section đó; chỉ tồn tại trong section đó, không đem đi nơi khác được | Có                          |
| **Theme block**                 | File riêng trong `/blocks/`, có `{% schema %}` riêng, dùng lại ở nhiều section, lồng nhau được                             | Có                          |
| **Snippet**                     | Đơn vị tái sử dụng cho dev                                                                                                 | Không                       |

Một số quy tắc quyết định:

- Khảo sát phạm vi sử dụng (Scope): Block này chỉ sống trong Section này hay có thể nằm ở bất kỳ đâu? Nếu chỉ nằm trong Section này -> Chọn Section Block.
- Khảo sát khả năng tái sử dụng: Nếu bạn đang xây dựng các thành phần Atomic (các khối nhỏ như Text, Media, Button) muốn cho phép Merchant kéo thả linh hoạt vào nhiều Section khác nhau -> Chọn Theme Block
- Giữ cấu trúc đơn giản: Không lạm dụng Theme Block cho những tính năng đơn giản, việc cô lập logic bên trong Section block giúp chạy nhẹ hơn và hạn chế xung đột style giữa các phần tử.
- **Snippet ↔ theme block** (ranh giới hay nhầm nhất): merchant cần thêm/bớt/kéo thả phần tử → **theme block**; chỉ dev cần dùng lại đoạn markup ở nhiều nơi → **snippet**.
- Snippet thường được sử dụng với các thành phần cố định tuyệt đối về mặt UI và bắt buộc phải luôn hiển thị và được sử dụng ở nhiều nơi và không muốn Merchant có thể tùy biến.
- Static block được gọi trực tiếp bằng tag Liquid, Merchant không thể xóa, không thể kéo/thả thay đổi vị trí nhưng có thể bấm ẩn/hiện và chỉnh sửa setting trong Theme Editor. khi các thành phần bắt buộc nằm ở một vị trí cố định và cho phép Merchant quyền tùy biến.
- Với vùng cần "cứng" (ví dụ tiêu đề, giá, nút mua ở trang product): dùng **static block** `{% content_for 'blocks' %}` cho vùng merchant được tự do sắp xếp. Lưu ý: static block vẫn có thể bị merchant ẩn, chỉ không xóa/kéo được.

## I.4 Chọn thẻ tài sản (style, stylesheet, javascript, script)

### **{% style %}**

    - Render Liquid? Có.

    - Có Subsetting không? Không. Thẻ này được inline trực tiếp vào HTML tại vị trí Section/Block xuất hiện dưới dạng thẻ <style data-shopify>.

    - Cách thức hoạt động của thẻ trong Theme Shopify:
        + Shopify đọc, biên dịch toàn bộ mã Liquid bên trong thẻ thành CSS thuần và chèn trực tiếp (inline) vào DOM HTML nơi Section/Block đó được gọi.
        + Không có cơ chế khử trùng lặp (Deduplication): Nếu một Section xuất hiện n lần trên một trang thì Shopify sẽ in ra n thẻ <style data-shopify> lặp đi lặp lại trực tiếp trong markup HTML, làm tăng dung lượng DOM.

    - Khi nào nên sử dụng:
        + CSS phụ thuộc trực tiếp vào cài đặt tùy biến của Merchant trong Theme Editor.
        + Khi khai báo CSS Custom Properties (biến CSS) và bắt buộc neo vào ID của section.

### **{% stylesheet %}**

    - Render Liquid? Không. Shopify bỏ qua hoàn toàn và không xử lý bất kỳ câu lệnh Liquid nào viết bên trong thẻ này.

    - Có Subsetting không? Có.

    - Cách thức hoạt động của thẻ trong Theme Shopify:
        + Shopify gộp nội dung {% stylesheet %} của tất cả các Section, Block, Snippet xuất hiện trên trang thành một file CSS duy nhất và chèn đường link nhúng file này thông qua {{ content_for_header}}.
        + File CSS tổng hợp này được lọc chính xác theo những Section/Block/Snippet thực sự có mặt trên trang.
        + Có khử trùng lặp: Dù Section/Block/Snippet xuất hiện nhiều lần thì khối CSS chỉ xuất hiện đúng 1 lần trong file CSS tổng hợp.

    - Khi nào nên sử dụng:
        + Style tĩnh, biệt lập của riêng từng Section/Block/Snippet.
        + Không dùng cho: Base CSS, Typography, Layout khung dùng chung.

### **{% javascript %}**

    - Render Liquid? Không. Shopify không biên dịch bất kỳ cú pháp Liquid nào bên trong thẻ này.

    - Có Subsetting không? Không. Mọi dòng JS viết ở đây đều đóng gói và tải trên tất cả các trang dù Section/Block có xuất hiện hay không.

    - Cách thức hoạt động của thẻ trong Theme Shopify:
        + Shopify gom nội dung theo loại file thành 3 file bundle riêng biệt: section -> script.js, block -> block-scripts.js, snippet -> snippet-scripts.js. Sau đó chèn vào trang qua {{content_for_header}} bằng thẻ <script defer>.
        + Nội dung của mỗi thẻ được tự động bọc trong một hàm tự thực thi (IIFE) để cô lập scope, đảm bảo biến không rò rỉ ra ngoài.
        + Vòng đời (Lifecycle): Chỉ thực hiện đúng một lần lúc tải trang. Nó không chạy lại khi Theme Editor nạp lại section và không chạy lại khi Section Rendering API trả về markup mới.

    - Khi nào nên sử dụng:
        + Khuyến nghị mặc định: KHÔNG NÊN DÙNG. Chi phí tải JS trên toàn bộ các trang gây lãng phí tài nguyên performance.
        + Ngoại lệ hợp lý: Là thành phần Global xuất hiện ở tất cả các trang ví dụ như menu, giỏ hàng trượt...hoặc code vô cùng ngắn.

### **<script>**

    - Render Liquid?
        + Không. Nếu nhúng file .js tĩnh từ thư mục assets/
        + Có. Nếu mở thẻ <script> trực tiếp ngay trong file .liquid.

    - Có Subsetting không? Không. Tuy nhiên, trình duyệt cũng có hỗ trợ thực hiện cơ chế tiến quét, tải, khử trùng lặp và cache để sử dụng với file js khi bạn nhúng thẻ <script src="" defer>.

    - Cách thức hoạt động của thẻ trong Theme Shopify:
        + File JS được tách thành các file tĩnh độc lập nằm trong thư mục assets/.
        + Nhúng vào Section/Block bằng thẻ HTML <script src="{{'my-component.js' | asset_url}}" defer> </script>.
        + Browser Caching: File JS tĩnh trên Shopify CDN được trình duyệt lưu bản cache tối ưu trên trình duyệt.
        + Tương thích hoàn hỏa với Web Component: Kết hợp rất tốt với kiến trúc Custom Elements.

    - Khi nào nên sử dụng?
        + Đây là lựa chọn chuẩn mực nên sử dụng cho hầu hết các xử lý logic Javascript trên theme Shopify hiện nay.

## I.5 Bộ công cụ kiểm tra

Các công cụ dưới đây được **gọi đến** trong Bước 3–6 của Phần II (xem thêm cột "Tự động/thủ công" ở Bước 5).

**Performance**

- `shopify theme check` — Lint theme theo khuyến nghị của nền tảng. Chạy liên tục khi code qua extension Shopify Liquid của VS Code, và chạy tự động trên PR bằng GitHub Action `shopify/theme-check-action`.
- `shopify theme profile --url /products/<handle>` — Đo thời gian máy chủ thực thi Liquid để dựng một trang cụ thể, kết quả mở bằng Speedscope. Dùng để tìm đoạn Liquid chậm.
- Chrome DevTools — **Performance tab**: ghi lại quá trình tải/tương tác để tìm điểm nghẽn (long task, layout shift). **Lighthouse tab**: chấm điểm 0–100 cho 4 hạng mục Performance, Accessibility, Best Practices, SEO kèm danh sách khuyến nghị. Điểm Lighthouse dao động giữa các lần chạy → lấy trung vị của nhiều lần chạy.

**Accessibility**

- Đối chiếu nghiệp vụ và tiêu chí chập nhận đã đáp ứng chưa.
- Dùng **bàn phím** đi hết luồng: Tab/Shift+Tab, Enter/Space, Esc; focus luôn nhìn thấy và không bị kẹt.
- Dùng **trình đọc màn hình thật**: NVDA (Windows, miễn phí), VoiceOver (macOS/iOS), TalkBack (Android). ChromeVox của Google chủ yếu dùng trên ChromeOS.
- Phóng to 200% và thu hẹp khung nhìn 320px để kiểm tra reflow; bật `prefers-reduced-motion`.
- DevTools → Elements → **Accessibility pane** để xem tên/vai trò/trạng thái mà trình đọc màn hình nhận được. Lưu ý: kiểm tra tự động (Lighthouse, axe) chỉ phát hiện được một phần lỗi, không thay thế kiểm tra bằng bàn phím và trình đọc màn hình.

**Tương thích** — Kiểm tra layout trên các kích thước màn hình và trình duyệt khác nhau (tối thiểu Chrome, Safari iOS, Firefox).

---

# PHẦN II — QUY TRÌNH THỰC THI

## Bước 1: Tiếp nhận và làm rõ yêu cầu

Tiếp nhận yêu cầu từ các stakeholder (Designer là nguồn chính). Dùng bộ câu hỏi **5W1H (mục I.1)** để làm rõ 4 yếu tố:

| Yếu tố cần làm rõ                          | Tương ứng 5W1H |
| ------------------------------------------ | -------------- |
| Nội dung yêu cầu                           | Why, What      |
| Bối cảnh thực hiện                         | Where, When    |
| Hành vi của người dùng                     | Who, When      |
| Ràng buộc (nghiệp vụ, thời gian, nền tảng) | How, When      |

**Đầu ra:** một GitHub Issue có mã yêu cầu (ví dụ `PRD-12`) ghi lại kết quả 5W1H. Mã này được dùng xuyên suốt trong tên nhánh, commit và PR.

## Bước 2: Tiêu chí chấp nhận (Acceptance Criteria — AC)

AC là căn cứ để quyết định yêu cầu **đã hoàn thành hay chưa** khi bàn giao. Viết theo cấu trúc **Cho trước – Khi – Thì** (Given – When – Then):

> **AC-1.** _Cho trước_ sản phẩm có variant đã hết hàng, _khi_ khách chọn variant đó, _thì_ nút "Thêm vào giỏ" bị vô hiệu hóa, nhãn đổi thành "Hết hàng", và trình đọc màn hình thông báo trạng thái mới.

Mỗi yêu cầu nên có ít nhất một AC phi chức năng về performance/A11y (ví dụ: "không gây layout shift khi đổi variant", "thao tác được hoàn toàn bằng bàn phím").

## Bước 3: Chọn giải pháp và chia task

**3.1 Chọn giải pháp.** Áp dụng mục I.3 (đơn vị kiến trúc) và I.4 (thẻ tài sản) để chọn phương án; ghi quyết định và lý do vào Issue.

**3.2 Chia task theo tầng.** Mỗi task làm trọn một tầng, thường theo thứ tự:

| Task    | Tầng                | Phạm vi                                                     |
| ------- | ------------------- | ----------------------------------------------------------- |
| Task.01 | Khung               | `schema` + settings + markup rỗng (HTML đúng ngữ nghĩa)     |
| Task.02 | Nội dung + style    | Nội dung + style token, **không JS**                        |
| Task.03 | Hành vi             | JS (custom element)                                         |
| Task.04 | Trạng thái          | Trạng thái của hành vi (loading, lỗi, hết hàng...) — nếu có |
| Task.05 | i18n                | Chuỗi hiển thị đưa vào `locales/`                           |
| Task.06 | Rà soát Performance | Đo tổng thể, tối ưu, xác nhận ngân sách                     |
| Task.07 | Rà soát A11y        | Kiểm tra tổng thể bằng bàn phím + trình đọc màn hình        |

- Trần thời gian mỗi task: **0,5–1 ngày**. Quá 1 ngày → khai thác lại yêu cầu và chia lại.
- Khuyến nghị: **1 task = 1 PR** để review nhỏ, dễ đọc diff.

**3.3 Performance và A11y nằm trong từng task, không dồn về cuối.** Task.06 và Task.07 chỉ đóng vai trò **rà soát, tối ưu và xác nhận tổng thể**. Các yêu cầu cơ bản phải đạt ngay trong task liên quan — đây là **Definition of Done theo tầng**:

| Task                | Performance phải đạt ngay trong task                                                                      | A11y phải đạt ngay trong task                                                     |
| ------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| 01 Khung            | Không có JS; ảnh có sẵn chỗ (width/height hoặc `aspect-ratio`)                                            | Element đúng ngữ nghĩa, thứ tự heading, landmark                                  |
| 02 Nội dung + style | Ảnh dưới màn hình đầu `loading="lazy"`, ảnh LCP không lazy; `sizes` đúng; không animate thuộc tính layout | Tương phản màu đạt; `:focus-visible` rõ; tôn trọng `prefers-reduced-motion`       |
| 03 Hành vi          | JS tải `defer`/module; không đọc-ghi layout xen kẽ; gỡ listener khi element bị gỡ                         | Tương tác được bằng bàn phím; trạng thái ARIA (`aria-expanded`...) cập nhật đúng  |
| 04 Trạng thái       | Không gây layout shift khi đổi trạng thái                                                                 | Thông báo thay đổi qua vùng `aria-live` khi cần                                   |
| 05 i18n             | —                                                                                                         | Không hardcode chuỗi, **kể cả** `aria-label` và văn bản ẩn cho trình đọc màn hình |
| 06 Rà soát Perf     | Lighthouse, `theme profile`, ngân sách JS cho cả section                                                  | —                                                                                 |
| 07 Rà soát A11y     | —                                                                                                         | Đi hết luồng bằng bàn phím + NVDA/VoiceOver                                       |

## Bước 4: Code, commit và mở Pull Request

1. Tạo nhánh từ `dev` theo quy ước III.2 (ví dụ `feat/PRD-12-price-by-variant`).
2. Code và xem trực tiếp bằng `shopify theme dev`. Lệnh này tạo một **development theme** riêng cho từng dev: ẩn trong admin, không tính vào giới hạn số theme của store, bị xóa sau 7 ngày không hoạt động hoặc khi `shopify auth logout`.
3. Commit theo quy ước III.3.
4. Push và **mở PR vào `dev` ngay khi có bản chạy được** (có thể mở ở dạng Draft). Mô tả PR theo mẫu ở III.3, gồm link preview và mục _Rủi ro hồi quy_.

## Bước 5: Review và điều kiện merge

**Review diễn ra trong PR.** PR là nơi reviewer đọc diff, trao đổi và approve.

**Điều kiện merge** — tách rõ cái nào chạy tự động, cái nào dev tự chạy:

| Kiểm tra                                                                                             | Cách chạy                                                                       | Khi nào                            | Chặn merge?           |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ---------------------------------- | --------------------- |
| `shopify theme check`: 0 error                                                                       | **Tự động** — extension VS Code khi code + GitHub Action trên mọi PR            | Liên tục                           | Có                    |
| Kiểm tra cấm: không có `.scss`/`.scss.liquid`, không có file minify (trừ ES6 + thư viện third-party) | **Tự động** — script trong CI                                                   | Mọi PR                             | Có                    |
| Console: 0 error                                                                                     | Thủ công — dev tự kiểm và ghi vào PR                                            | Mọi PR                             | Có                    |
| Lighthouse (DevTools) cho trang bị ảnh hưởng                                                         | **Thủ công** — dev tự chạy như một công đoạn rà soát trong mỗi task             | Mọi PR có thay đổi giao diện       | Không (ghi số vào PR) |
| Lighthouse tổng thể: trung bình trang product/collection/home, desktop + mobile                      | Task.06 + trước release; có thể tự động hóa bằng `shopify/lighthouse-ci-action` | Trước khi merge `staging` → `main` | Có                    |
| 1 approval từ reviewer                                                                               | Con người                                                                       | Mọi PR                             | Có                    |
| Mọi comment dạng "blocking" đã resolve                                                               | Con người                                                                       | Mọi PR                             | Có                    |

**Về ngưỡng Lighthouse**: Yêu cầu tối thiểu của Shopify Theme Store là Performance **≥ 90** và Accessibility **≥ 90** (trung bình product/collection/home, cả desktop lẫn mobile).

Nếu dùng `shopify/lighthouse-ci-action`: action này mặc định chấm theme với **setting mặc định**, muốn chấm theo cấu hình thật phải khai tham số `pull_theme`; cần một custom app có quyền `read_products` và `write_themes`.

**Ai được merge**: bật branch protection cho `dev`, `staging`, `main` (bắt buộc PR, 1 approval, check theme-check pass). Merge vào `dev`: tác giả PR sau khi có approval. Merge vào `staging` và `main`: Tech lead hoặc người được chỉ định.

## Bước 6: QA theo môi trường và kiểm thử hồi quy

### 6.1 Ba môi trường

|              | **Dev**                                          | **Preview (Staging)**                                                                     | **Production**                                   |
| ------------ | ------------------------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Nhánh Git    | `dev` (+ development theme cá nhân của từng dev) | `staging`                                                                                 | `main`                                           |
| Theme        | Theme chưa publish trên **store dev**            | Theme chưa publish trên **store thật**                                                    | Theme đang publish                               |
| Dữ liệu      | Dữ liệu store dev                                | Dữ liệu thật của store                                                                    | Dữ liệu thật, khách hàng thật                    |
| Ai test      | Dev                                              | QA + người duyệt nội dung (+ Lighthouse CI nếu có)                                        | Dev **và** QA — chỉ smoke test                   |
| Test được gì | Logic, markup, style, JS                         | Theme editor, dữ liệu thật, app block, tốc độ thật, luồng giỏ hàng → chuyển sang checkout | Theme sau publish có hoạt động bình thường không |

- Chia sẻ Preview bằng link `?preview_theme_id=<id>`.
- **Checkout không do theme render**, nên ở Preview chỉ kiểm tra được bước chuyển từ giỏ hàng sang checkout, không kiểm tra giao diện checkout.
- Lưu ý khi mỗi nhánh nối vào một theme qua Shopify GitHub integration: tích hợp đồng bộ **hai chiều** — push lên nhánh thì theme được cập nhật; ngược lại, thay đổi lưu trong theme editor/code editor trên admin được Shopify **tự commit ngược về nhánh**. Hệ quả cho quy trình xem Bước 7 và III.2.

### 6.2 Kiểm thử hồi quy

Hồi quy là kiểm tra những thứ **không đụng tới** có bị ảnh hưởng không. Chạy ở hai thời điểm: phạm vi hẹp khi review PR (theo mục _Rủi ro hồi quy_ trong mô tả PR), và toàn bộ ma trận trên Preview trước release.

**a) Ma trận trang × trạng thái**

| Trang        | Trạng thái cần kiểm                                                                                                                       |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Home         | 0 section / đủ section / section thiếu ảnh                                                                                                |
| Product      | 1 variant / > 250 variant (Liquid chỉ trả tối đa 250) / tổ hợp option không có variant / hết hàng / pre-order / gift card / không ảnh     |
| Collection   | 0 sản phẩm / có filter / > 5.000 sản phẩm (Shopify không hiển thị filter)                                                                 |
| Cart         | Rỗng / 1 dòng / 10 dòng / có mã giảm giá                                                                                                  |
| Search       | 0 kết quả / nhiều kết quả / > 100.000 kết quả (Shopify không hiển thị filter)                                                             |
| Customer     | Đã đăng nhập / chưa đăng nhập — kiểm tra `<shopify-account>` ở header (Theme Store bắt buộc từ 30/07/2026, hiển thị cả desktop và mobile) |
| Theme editor | Thêm section từ preset, kéo thả block, ẩn block, đổi setting — hành vi JS vẫn đúng sau khi editor nạp lại section                         |

**b) Bản đồ ảnh hưởng của snippet** — snippet là nguồn hồi quy số 1 vì được render ở nhiều nơi. Ví dụ:

- `snippets/price.liquid` → product, card ở collection, search, cart...
- `snippets/responsive-image.liquid` → mọi template có ảnh.

**Trách nhiệm cập nhật bản đồ:** _người sửa hoặc thêm nơi dùng snippet_ cập nhật bản đồ **trong cùng PR đó**; reviewer kiểm tra mục này khi review. Để không phụ thuộc trí nhớ, bản đồ có thể sinh lại tự động:

```bash
# Liệt kê mọi nơi render snippet 'price'
grep -rnE "render ['\"]price['\"]" --include=*.liquid sections blocks snippets layout templates
```

**Về lâu dài:** các ca hồi quy ổn định và lặp lại nhiều lần được chuyển thành checklist chuẩn (đính kèm mẫu PR), sau đó tự động hóa dần bằng kiểm thử trình duyệt.

## Bước 7: Release và rollback

### 7.1 Release

Điều kiện trước release: QA trên Preview đạt, Lighthouse tổng thể đạt ngưỡng, và **đã back-merge `main` → `staging`** (để nhận các thay đổi merchant đã lưu trên theme live — xem III.2).

1. **Tạo bản backup:** trong admin, _Duplicate_ theme đang live, đặt tên `backup-vX.Y.Z-YYYYMMDD`. Store có giới hạn số theme nên chỉ giữ khoảng 3 bản backup gần nhất.
2. Mở PR `staging` → `main`; khi merge, GitHub integration cập nhật ngay theme live.
3. Gắn tag Git `vX.Y.Z` lên commit vừa merge.
4. Dev và QA smoke test Production ngay sau release (các trang và luồng chính ở ma trận 6.2a).

### 7.2 Rollback khi Production lỗi nghiêm trọng

**Thế nào là lỗi nghiêm trọng** (kích hoạt rollback): không thêm được vào giỏ / không sang được checkout; trang chính trắng hoặc lỗi Liquid; hiển thị sai giá; lỗi A11y chặn hoàn toàn một luồng mua hàng. Lỗi nhỏ hơn → sửa bằng hotfix, không rollback.

**Ai quyết định:** Tech lead (hoặc người trực được chỉ định), báo ngay cho Merchant/stakeholder.

**Rollback về đâu và làm thế nào:**

1. **Khôi phục ngay:** publish bản `backup-...` gần nhất trong admin — theme live trở về trạng thái trước release trong vài giây. Theme nối với `main` lúc này trở thành theme chưa publish.
2. **Sửa lỗi bằng hotfix:** tạo `hotfix/<mã>-<mô tả>` từ `main` → PR vào `main` (quy trình rút gọn: theme-check pass + 1 approval) → kiểm tra bản sửa trên theme của `main` bằng link preview.
3. **Publish lại** theme nối với `main` khi đã ổn.
4. **Đồng bộ lại nhánh:** back-merge `main` → `staging` → `dev`; xóa nhánh hotfix; xác nhận theme đang live **chính là** theme nối với `main`.
5. **Ghi nhận:** mô tả sự cố, nguyên nhân, cách phòng ngừa trong Issue; bổ sung ca hồi quy tương ứng vào ma trận 6.2.

> **Lưu ý khi rollback bằng backup:** những thay đổi merchant lưu trong theme editor _sau_ thời điểm tạo backup không có trong bản backup. Các thay đổi đó vẫn còn trong lịch sử nhánh `main` (do Shopify tự commit), nên sẽ quay lại khi publish lại theme của `main`.

---

# PHẦN III — QUY ƯỚC

## III.1 Quy ước đặt tên

| Đối tượng               | Quy tắc                                                                                                          | Ví dụ                                         |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| File section, block     | kebab-case, dùng danh từ                                                                                         | `main-product.liquid`, `product-price.liquid` |
| Snippet                 | kebab-case, cấu trúc `<vai trò>-<đối tượng>`                                                                     | `responsive-image.liquid`                     |
| Asset CSS               | Trùng tên section/block                                                                                          | `main-product.css`                            |
| Asset JS                | Trùng tên custom element                                                                                         | `product-form.js`                             |
| Custom element          | kebab-case, cấu trúc `<domain>-<component>`; **bắt buộc có dấu gạch ngang** (quy định của chuẩn Custom Elements) | `product-form`, `cart-drawer`                 |
| CSS class               | BEM, prefix theo section/block                                                                                   | `product-form__submit--sold-out`              |
| CSS custom property     | `--<Tiền tố (nếu cần)>-<scope>-<thuộc tính>-<trạng thái (nếu có)>`                                               | `--button-bg-hover`                           |
| Setting id trong schema | snake_case                                                                                                       | `show_vendor`, `image_ratio`                  |
| Translation key         | Đường dẫn phân cấp bằng dấu chấm, mỗi cấp snake_case                                                             | `products.product.add_to_cart`                |
| Biến JS                 | camelCase; boolean bắt đầu bằng `is`/`has`                                                                       | `isLoading`, `hasVariants`                    |

## III.2 Nhánh Git và luồng di chuyển code

**Nhánh dài hạn** — mỗi nhánh nối với một theme qua Shopify GitHub integration (1 nhánh ↔ 1 theme):

| Nhánh     | Theme                              | Ghi chú                                                              |
| --------- | ---------------------------------- | -------------------------------------------------------------------- |
| `main`    | Production (đang publish)          | Đổi tên từ `product` để tránh nhầm với khái niệm Product của Shopify |
| `staging` | Preview (chưa publish, store thật) |                                                                      |
| `dev`     | Dev (chưa publish, store dev)      | Khác với _development theme_ do `shopify theme dev` tạo cho từng dev |

**Nhánh ngắn hạn** — cấu trúc `<loại>/<mã yêu cầu>-<mô tả ngắn kebab-case>`, ví dụ `feat/PRD-12-price-by-variant`. Loại gồm: `feat` (tính năng mới), `fix` (sửa lỗi), `perf` (hiệu năng), `a11y` (khả năng truy cập), `refactor` (tái cấu trúc), `docs` (tài liệu), và bổ sung `hotfix` (sửa lỗi khẩn trên Production).

**Luồng di chuyển code:**

```
feat/* ─┐
fix/*  ─┼─ PR ─> dev ─ PR ─> staging ─ PR ─> main  (= theme live)
perf/* ─┤         ▲            ▲              │
a11y/* ─┘         └─ back-merge  <────────────┤
                                              |
hotfix/* ── tạo từ main ─ PR ─> main  <────────

```

| Chuyển                       | Điều kiện                                                                              | Người merge **[cần team chốt]** |
| ---------------------------- | -------------------------------------------------------------------------------------- | ------------------------------- |
| `feat/*`, `fix/*`... → `dev` | Điều kiện merge Bước 5                                                                 | Tác giả PR sau approval         |
| `dev` → `staging`            | Task của yêu cầu đã xong; dev đã kiểm trên theme Dev                                   | Tech lead                       |
| `staging` → `main`           | QA Preview đạt; hồi quy đạt; Lighthouse tổng thể đạt; đã back-merge `main` → `staging` | Tech lead                       |
| `hotfix/*` → `main`          | Theme-check pass + 1 approval                                                          | Tech lead                       |

**Quy tắc back-merge (quan trọng).** Vì GitHub integration đồng bộ hai chiều, mọi chỉnh sửa merchant lưu trên theme live (trong `config/settings_data.json`, `templates/*.json`, `sections/*.json`...) được Shopify tự commit vào `main`. Nếu `staging` không nhận các commit này trước khi merge ngược lên `main`, release sẽ **ghi đè cấu hình merchant**. Do đó:

- Back-merge `main` → `staging` → `dev` **trước mỗi release** và **sau mỗi hotfix**.
- Khi có xung đột ở các file JSON cấu hình kể trên, mặc định **giữ bản của `main`** (bản merchant đang dùng); thay đổi cố ý vào các file này phải ghi rõ trong PR.

## III.3 Commit và Pull Request

**Cấu trúc commit** (theo Conventional Commits):

```
<loại>(<scope>): <mô tả ngắn, dạng mệnh lệnh>

<Lý do thực hiện> <Những việc đã làm>

Refs: <mã yêu cầu>
```

Ví dụ:

```
feat(main-product): render lại price qua Section Rendering API

product.variants trong Liquid chỉ trả tối đa 250 variant nên không thể
dựng JSON toàn bộ variant. Khi đổi variant, gọi
/products/{handle}?variant={id}&section_id={sectionId}
và thay phần tử [data-price] bằng markup trả về.

Refs: PRD-12
```

Ghi chú: loại `a11y` không nằm trong danh sách loại mặc định của cấu hình commitlint phổ biến (`@commitlint/config-conventional`); nếu dùng commitlint cần khai thêm vào `type-enum`.

**Pull Request** — mở ngay ở Bước 4, review diễn ra trong PR (Bước 5). Mẫu mô tả PR:

```
## Yêu cầu
PRD-xx — <tóm tắt>

## Cách làm và lý do
<phương án đã chọn, phương án đã loại và vì sao>

## Tiêu chí chấp nhận
- [ ] AC-1 ...
- [ ] AC-2 ...

## Preview
<link ?preview_theme_id=...>

## Kiểm tra đã chạy
- [ ] theme check: 0 error (CI)
- [ ] Console: 0 error
- [ ] Lighthouse trang bị ảnh hưởng: perf __ / a11y __ (mobile), perf __ / a11y __ (desktop)
- [ ] Bàn phím + trình đọc màn hình (nếu thay đổi tương tác)

## Rủi ro hồi quy
<snippet/section bị chạm → những nơi dùng chúng; đã cập nhật bản đồ ảnh hưởng chưa>
```

---

# PHẦN IV — KẾ HOẠCH GIAI ĐOẠN HIỆN TẠI

- **Mốc:** hoàn thành phần codebase anh Phúc bàn giao trước **30/10/2026**; sau đó tiếp tục nhận bàn giao và thực hiện phần tiếp theo.
- Vì đây là theme đầu tiên, em xin phép làm theo hướng làm – kiểm thử – đẩy liên tục, chưa lập kế hoạch chi tiết cho từng phần việc nhỏ. Các phần việc nhỏ được tạo thành **GitHub Issue** (mỗi Issue theo Bước 1–3), gom vào một **Milestone** có hạn 30/10/2026 để theo dõi tiến độ.
- **Ghi chép:** với mỗi block/section, ghi lại quá trình đề xuất giải pháp, kiểm thử và cải tiến ngay trong Issue và mô tả PR (mục _Cách làm và lý do_), làm nguồn cho tài liệu về sau.

---

**Nguồn**

- JavaScript and stylesheet tags — https://shopify.dev/docs/storefronts/themes/best-practices/javascript-and-stylesheet-tags
- Load first-paint resources before content_for_header — https://shopify.dev/docs/storefronts/themes/best-practices/performance/load-critical-resources-before-content-for-header
- Shopify CLI `theme profile` — https://shopify.dev/docs/api/shopify-cli/theme/theme-profile
- Shopify CLI for themes (development themes) — https://shopify.dev/docs/storefronts/themes/tools/cli
- Shopify GitHub integration for themes — https://shopify.dev/docs/storefronts/themes/tools/github
- Storefront filtering / Adding filters — https://shopify.dev/docs/storefronts/themes/navigation-search/filtering/storefront-filtering/support-storefront-filtering , https://help.shopify.com/en/manual/online-store/search-and-discovery/filters
- Theme Store requirements — https://shopify.dev/docs/storefronts/themes/store/requirements
- GitHub Actions: https://github.com/Shopify/theme-check-action , https://github.com/Shopify/lighthouse-ci-action
