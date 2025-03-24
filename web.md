Bạn có thể sử dụng sự kiện `scroll` trong jQuery để kiểm tra khi người dùng cuộn đến cuối trang và sau đó tải thêm dữ liệu. Dưới đây là cách làm:  

### Cách hoạt động:
- Kiểm tra nếu người dùng cuộn đến cuối trang (`$(window).scrollTop() + $(window).height() >= $(document).height()`).
- Gọi AJAX để tải thêm dữ liệu khi đạt đến cuối trang.

### Code jQuery:
```javascript
$(document).ready(function () {
    $(window).scroll(function () {
        if ($(window).scrollTop() + $(window).height() >= $(document).height() - 50) {
            loadMoreData();
        }
    });

    function loadMoreData() {
        $.ajax({
            url: "load_more.php", // URL xử lý backend
            type: "GET",
            beforeSend: function () {
                $("#loading").show(); // Hiển thị icon loading
            },
            success: function (data) {
                $("#data-container").append(data); // Thêm dữ liệu vào danh sách
                $("#loading").hide(); // Ẩn icon loading
            },
            error: function () {
                alert("Có lỗi xảy ra!");
                $("#loading").hide();
            }
        });
    }
});
```

### HTML:
```html
<div id="data-container">
    <!-- Dữ liệu sẽ được load vào đây -->
</div>
<div id="loading" style="display: none;">
    <p>Loading...</p>
</div>
```

🔹 **Giải thích:**
- Khi cuộn đến cuối trang (cách đáy `50px`), hàm `loadMoreData()` sẽ được gọi.
- AJAX sẽ gửi request đến `load_more.php` để lấy thêm dữ liệu.
- Dữ liệu mới được thêm vào `#data-container` mà không cần reload trang.
- Hiển thị/Ẩn biểu tượng loading để thông báo quá trình tải.

Bạn có cần tối ưu hoặc thêm tính năng nào khác không? 🚀
