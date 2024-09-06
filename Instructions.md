# FTU Data science competition round 1 notebook

## Scenario:
Bạn là một nhà phân tích dữ liệu tài năng, được giao trọng trách tối ưu hóa hoạt động kinh doanh của chi nhánh HomeMart. Hằng ngày có tới hàng nghìn khách hàng ghé thăm và mua sắm tại cửa hàng. Chi nhánh của bạn là một trong các chi nhánh trọng điểm nên đã được bố trí hệ thống AI camera . Với nguồn dữ liệu khổng lồ thu thập được từ hệ thống, bạn sẽ phải trổ tài để khám phá những bí mật ẩn sâu trong hành vi mua sắm của khách hàng.

## Mục tiêu:

- Phân tích sâu sắc: Xây dựng một hệ thống báo cáo toàn diện, cung cấp cái nhìn chi tiết về doanh thu, xu hướng tiêu dùng của từng sản phẩm, nhóm khách hàng.
- Dự báo thông minh: Dựa trên dữ liệu thu thập, dự đoán chính xác nhu cầu của khách hàng, từ đó đưa ra các đề xuất về kế hoạch nhập hàng, khuyến mãi, marketing hiệu quả.
- Vươn lên dẫn đầu: Đứng trong top 5 chi nhánh có doanh thu cao nhất của năm để mang về giải thưởng hấp dẫn 100.000.000 VNĐ.

## Thách thức:

**Dữ liệu có thể chứa nhiều thông tin nhiễu, cần được làm sạch và xử lý trước khi phân tích**
## File mô tả:
- Shelf information: Thông tin các gian hàng có trong cửa hàng của Home-Mart
- Customer behavior: Thông tin các hành vi khi mua sắm của khách hàng
- Item information: Thông tin của sản phẩm trong cửa hàng

## **Note:**
Một số giả thiết cần lưu ý:
- Mặt hàng được xem là “được mua” khi mặt hàng đấy được bỏ vào giỏ hàng và không được lấy ra. Đây là một ví dụ các hành vi có thể có của khách hàng khi mua một sản phẩm:
        - (putting_item_into_bag Nan & putting_item_into_bag_in_the_2nd_time Nan) & \
        [! returning_item & [ ! taking_item_out_of_bag | taking_item_out_of_bag Nan]] 
        - putting_item_into_bag & [ ! returning_item & [ ! taking_item_out_of_bag | taking_item_out_of_bag Nan]] 
        - putting_item_into_bag_in_the_2nd_time & [! returning_item & [ ! taking_item_out_of_bag | taking_item_out_of_bag Nan]]
    

“looking at item” → “picking up item” → “holding the item” → “put item into bag”.

___ 
Contents:
1. Main note book for data wrangling, EDA, and answers 
2. Answer.md file for the concise answers in text formate 