---
layout: post
comments: true
title:  "DDOS - Bảo mật và phòng chống Ddos cho website WordPress (phần 3)"
title2:  "Bảo mật và phòng chống Ddos cho website WordPress (phần 3)"
date:   2021-05-03 15:10:00
permalink: 2021/05/03/Phong-chong-Ddos-website-WP-p3
mathjax: false
tags: Security DDOS WordPress
category: Security
sc_project: 12494739
sc_security: d785a534
img: /assets/Phong-chong-Ddos-website-WP-p3/Hinh1.jpg
summary: Bảo mật và phòng chống Ddos cho website WordPress
---

# Tối ưu Website để giảm thiệt hại của các đợt tấn công DDOS

Bài thứ ba của Tôi chia sẻ các kinh nghiệm nhỏ và các kỹ thuật phân tích để giảm thiệt hại từ các đợt tấn công DDOS hạng nặng. Tôi thường tự tìm hiểu và tối ưu hoá Website cá nhân để đạt tốc độ tốt nhất và tối ưu nhất. Nếu bạn truy cập Website của tôi, bạn sẽ gần như không cảm nhận thấy rằng Website đang tải mà nó tải xong luôn chỉ dưới 1 giây. Và điều mà tôi làm chỉ đơn giản là hướng tới sự đơn giản, tập trung vào phần nội dung chính là hồn cốt của Website chứ không phải vẻ bề ngoài. Tôi sẽ chia sẻ để bạn thấy quá trình tối ưu hoá website có tác dụng thế nào trong việc giảm thiểu các tác động của các đợt tấn công vào Website.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p3/Hinh1.jpg" width = "800">
</div>
<div class="thecap">Hình 1</div>
</div>
<hr>

## 1. Bật CDN miễn phí

Tôi dùng WordPress và chính vì vậy, có nhiều giải pháp để tích hợp CDN. Ban đầu tôi dùng CDN của Bizfly nhưng sau vài tháng sử dụng với chi phí chỉ khoảng 25-30k/tháng. Có vẻ Bizfly giở trò và chơi trò tối thiểu. Xài tối thiểu 1 tháng là 100GB với giá hơn 100.000VND. Nghĩa là trước đây tôi chỉ mất 1/3 chi phí. Cái khó chịu của Bizfly là khi Website của tôi bị tấn công DDOS và lưu lượng tăng tới 1TB (Chi phí khoảng 1tr) thì Bizfly không hề có cảnh báo gì tới tôi mà cứ vậy trừ phí. Chưa kể, ở thời điểm bị tấn công mạnh, CDN của Bizfly cũng tèo luôn làm cho Web của tôi không thể tải được các tài nguyên tĩnh và từ đó sập luôn. Tôi quyết định từ bỏ CDN của Bizfly để dùng dịch vụ CDN MIỄN PHÍ của jetpack là Photon bằng cách cài Plugin Jetpack
- Bật tính năng CDN ở mục Performance trong cài đặt
- Enable site accelerator và tích chọn cả Speed up image load times và Speed up static file load times
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p3/Hinh3.jpg" width = "800">
</div>
<div class="thecap">Hình 3</div>
</div>
<hr>

Với việc dùng CDN miễn phí này, toàn bộ các Request vào hình ảnh và các file tĩnh như css và js sẽ được load qua dịch vụ của Jetpack (Công ty con của Automatic - Chủ quản WordPress.com). Điều này giảm thiểu các thiệt hại cho tôi và Webserver không cần phải đáp ứng các request đó dẫn tới quá tải.

## 2. Không Combine css và js mà xoá những phần không cần thiết ở những nơi không cần dùng.

Đây là một yếu tố quan trọng nữa để giảm thiểu thiệt hại từ DDOS và giúp Website sống khoẻ mặc dù các đợt tấn công dồn dập không ngừng nghỉ. Tôi có mua Plugin hỗ trợ Cache là WP-Rocket và có đọc rất kỹ các hướng dẫn. Website của tôi theo khuyến cáo của WP-Rocket thì không cần phải combine file css và js. Lý do là Website của Tôi hỗ trợ HTTP/2.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p3/Hinh4.jpg" width = "800">
</div>
<div class="thecap">Hình 4</div>
</div>
<hr>

Lý giải đơn giản: Khác với giao thức HTTP/1.1, giao thức HTTP/2 có cách truyền tải dữ liệu hoàn toàn mới. Nó tương tự như cách chúng ta combine JS, CSS. Các dữ liệu sẽ được gộp chung vào một kết nối TCP (multiplexed) để giảm số lượng HTTP request.

Như vậy, việc combine css và js là không cần thiết. Tuy nhiên nếu như giữ các file css và js không cần thiết ở trang chủ chẳng hạn, thì nó sẽ làm tăng Page size và cũng phần nào đó ảnh hưởng tới tốc độ tải trang. Hiểu đơn giản thì việc tải nhiều css và js cũng giống việc bạn phải chở quá nhiều thứ trên một chiếc xe thồ vậy. Càng chất thêm lên thì càng nặng.

Website của tôi ban đầu theo thống kê của GTMetrix thì có khoảng 62 Request trong đó chủ yếu là JS và CSS. Tôi phân tích và thấy các phần bổ trợ sau thêm vào các css và js không cần thiết cho các loại trang:
- Trang chủ
- Trang chuyên mục bài viết
- Trang từ khoá
- Trang tác giả

Các Plugin:
- KK Star Rating
- Ultimate membership
- Jetpack
- WooCommerce
- Glossary

Tôi viết các hàm để xoá js và css không cần thiết mà các plugin này thêm vào trang chủ như sau:

```
/**
 * Disable WooCommerce block styles (back-end).
 */
function slug_disable_woocommerce_block_editor_styles() {
  wp_deregister_style( 'wc-block-editor' );
  wp_deregister_style( 'wc-block-style' );
  wp_deregister_style( 'wc-block-style' );
}
add_action( 'enqueue_block_assets', 'slug_disable_woocommerce_block_editor_styles', 1, 1 );
/** Disable Css Unnessesary*/
function totrieu_remove_css_from_home() {
 if (is_front_page() || is_category() || is_tag() || is_author()){
 wp_dequeue_style( 'kk-star-ratings' ); // style id
 wp_dequeue_style( 'kk-star-ratings-inline' );
 wp_dequeue_style( 'ihc_front_end_style' );
 wp_dequeue_style( 'ihc_templates_style' );
 wp_dequeue_style( 'orgseries-default-css' );
 wp_dequeue_style( 'wpsm-comptable-styles' );
 wp_dequeue_style( 'wpg-main-style' );
 wp_dequeue_style( 'wpg-tooltipster-style' );
 }
} 
add_action( 'wp_enqueue_scripts', 'totrieu_remove_css_from_home', 9999);
//Remove JS
function totrieu_remove_js_from_home() {
 if (is_front_page() || is_category() || is_tag() || is_author()){
 wp_dequeue_script( 'ihc-jquery-ui' ); // script id
 wp_dequeue_script( 'ihc-front_end_js-js-extra' );
 wp_dequeue_script( 'ihc-front_end_js' );
 wp_dequeue_script( 'wpg-mixitup-script' );
 wp_dequeue_script( 'wpg-tooltipster-script' );
 wp_dequeue_script( 'wpg-main-script-js-extra' );
 wp_dequeue_script( 'wpg-main-script' );
 wp_dequeue_script( 'kk-star-ratings-js-extra' );
 wp_dequeue_script( 'kk-star-ratings' );
 }
} 
add_action( 'wp_print_scripts', 'totrieu_remove_js_from_home', 9999);
add_filter( 'jetpack_implode_frontend_css', '__return_false', 99 );
```

Sau khi thực hiện việc này, Website của tôi giảm từ 62 Request xuống chỉ còn 25 Request như các bạn có thể xem ở ảnh 05
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p3/Hinh5.jpg" width = "800">
</div>
<div class="thecap">Hình 5</div>
</div>
<hr>

## 3. Loại bỏ plugin Cache

Không hiểu WP-Rocket bị xung đột với plugin nào gây ra lỗi không hiển thị được nội dung bài viết thường xuyên, nên tôi bỏ luôn WP Rocket và dùng luôn các tính năng mặc định của Cloudflare:
- Tôi bật Auto Minify của Cloudflare với JS, HTML, CSS
- Bật chế độ nén Brotli để tăng tốc độ tải trang
- Dùng thêm dịch vụ Automatic Platform Optimization for WordPress $5/tháng để tối ưu hoá điểm tốc độ tải trang
- Và bật luôn tính năng Rocket Loader của Cloudflare (Miễn phí).

<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p3/Hinh6.jpg" width = "800">
</div>
<div class="thecap">Hình 6</div>
</div>
<hr>

## 4. Lời kết

Ngoài việc phân tích cách thức tấn công DDOS, việc quan tâm tới chính website cũng giúp bạn phần nào giảm thiểu thiệt hại của các đợt tấn công. Theo ý hiểu của tôi thì Website càng nặng, Request càng nhiều thì càng thiệt hại nặng khi bị tấn công DDOS. Website hiện đại đang cố giống như một cô gái xấu làm đẹp bằng cách dùng quá nhiều mỹ phẩm hoặc vung tiền để phẫu thuật thẩm mỹ. Mà càng làm thế thì càng tốn tiền và thật lòng mà nói thì giảm trải nghiệm người dùng rất nhiều. Tôi chỉ hướng tới sự đơn giản nhất có thể mà thôi. Tất nhiên triết lý này khó để áp dụng với các bạn đánh đổi giữa cái đẹp và tốc độ, cũng như các bạn đang duy trì các Website thương mại điện tử được.
<hr>
<div class="imgcap">
<div >
    <img src="/assets/Phong-chong-Ddos-website-WP-p3/Hinh7.jpg" width = "800">
</div>
<div class="thecap">Hình 7</div>
</div>
<hr>

Nguồn: [Tô Triều](https://www.facebook.com/groups/hieupcwithfriends/permalink/2867173740227668/)

---
