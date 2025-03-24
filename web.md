# 1. Scroll to load more 

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


# 2. Rê chuột trong 10 giây nhận voucher giảm giá 15%

Bạn có thể sử dụng jQuery để theo dõi khi người dùng di chuyển chuột trong 10 giây liên tục, sau đó hiển thị thông báo tặng voucher.  

### ✨ **Cách hoạt động**:
1. Khi người dùng di chuyển chuột (`mousemove`), một bộ đếm thời gian (`setTimeout`) được kích hoạt.
2. Nếu họ tiếp tục di chuyển chuột trong 10 giây, hiển thị thông báo tặng voucher.
3. Nếu người dùng dừng di chuyển chuột, bộ đếm sẽ bị hủy (`clearTimeout`).

---

### 📝 **Code jQuery**:
```javascript
$(document).ready(function () {
    let timer;
    let isVoucherGiven = false;

    $(document).mousemove(function () {
        if (!isVoucherGiven) {
            clearTimeout(timer);
            timer = setTimeout(function () {
                isVoucherGiven = true;
                alert("🎉 Chúc mừng! Bạn nhận được voucher giảm giá 15%!");
            }, 10000); // 10 giây
        }
    });
});
```

---

### 🎨 **Giải thích**:
- **`mousemove`**: Kích hoạt mỗi khi chuột di chuyển.
- **`setTimeout(10000)`**: Nếu chuột di chuyển liên tục trong 10 giây, hiển thị voucher.
- **`clearTimeout(timer)`**: Nếu chuột ngừng di chuyển, hủy bộ đếm và đặt lại.
- **`isVoucherGiven`**: Đảm bảo voucher chỉ hiển thị một lần.

Bạn muốn hiển thị voucher dưới dạng pop-up hay có hiệu ứng gì đặc biệt không? 🚀
