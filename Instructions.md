
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
“looking at item” → “picking up item” → “holding the item” → “put item into bag”.


## Question 
01. (5đ) Thống kê ***5 mặt hàng*** có **tổng thời gian nhìn và cầm xem lâu nhất**?

    result1 = df_cust.groupby('si_id')[['looking_at_item_(s)', 'holding_the_item_(s)']].sum().reset_index()
    result1['total_time']=result1['looking_at_item_(s)']+result1['holding_the_item_(s)']
    result1=pd.merge(right=result1,left=df_item, how='right', on='si_id')
    result1[['shelf_id'	,'item_id',	'name','total_time']].sort_values(by='total_time', ascending=False).head(5)

> ANSWER:
shelf_id	item_id	name	total_time
130	7	6	Sữa chua uống Yakult	22896
131	7	7	Sữa chua uống Probi	22896
39	2	8	Sữa ông thọ	13939
9	0	6	Bim bim Oishi	13866
10	0	7	Snack khoai tây Lays	13362



08. (6đ) Top 3 quầy hàng có thời lượng trung bình quan tâm đến sản phẩm, trên số lượt tương tác, là lâu nhất (quan tâm tương ứng với việc nhìn và cầm xem)?

        # interaction per person per date-time per product
    interact= df_cust[['si_id','shelf_id','item_id','person_id','date', 'time', 'holding_the_bag','picking_up_item','returning_item'	,'putting_item_into_bag'	,'taking_item_out_of_bag'	,'putting_item_into_bag_in_the_2nd_time']]
        # interest count per person epr product (if time looking >0 and if time holding >0 )
    interest=pd.concat([df_cust['holding_the_item_(s)'] >0, df_cust['looking_at_item_(s)'] >0],axis=1)
        # df of bool value of iterest and iteraction per person per date-time
    i_i=pd.concat([interact,interest],axis=1)
    person_ii = i_i.groupby(['person_id','si_id' ,'shelf_id'])[['holding_the_item_(s)','looking_at_item_(s)']].sum().reset_index()
        # total duration of interest (look vs sum)
    person_ii['total_ii']=person_ii['holding_the_item_(s)']+person_ii['looking_at_item_(s)']
    product_ii=person_ii.groupby('si_id')[['holding_the_item_(s)','looking_at_item_(s)']].sum().reset_index()
    product_ii
    result8 = df_cust.groupby(['si_id', 'shelf_id', 'item_id'])[['looking_at_item_(s)', 'holding_the_item_(s)']].sum().reset_index()
    result8=pd.merge(right=result8, left=product_ii, how='left', on='si_id')
        # sum of the average duration per look count for every product in that shelf + sum of the average duration per hold count for every product in that shelf
    result8['avg_look_t']=result8['looking_at_item_(s)_y']/result8['looking_at_item_(s)_x']
    result8['avg_hold_t']=result8['holding_the_item_(s)_y']/result8['holding_the_item_(s)_x']
    result8['avg_i_t']=result8['avg_look_t']+result8['avg_hold_t']
    result8=result8.groupby('shelf_id')['avg_i_t'].sum()
    result8.sort_values(ascending=False)

> ANSWER:
3    1512.723763
4    1100.533550
5     998.517544

**note**: df_cus[timestamp, (shelf id)] sum( timestammp) groupby item id / count(item id)


10. (8đ) Người dùng có thói quen di chuyển giữa 2 quầy hàng nào nhiều nhất?
Ví dụ: Giả sử:
Người A có đường đi giữa các quầy hàng: (1) → (3) → (2) → (6)
Người B có đường đi: (3) → (2) → (6)
Người C có đường đi: (4) → (5) → (2) → (6)
Vậy: (1) → (3): 1 người, (3) → (2): 2 người, (2) → (6): 3 người, (4) → (5): 1 người, (5)
→ (2) 1 người.
Vậy người dùng có thói quen di chuyển nhiều nhất từ quầy (2) sang quầy (6).
    path=i_i[['person_id', 'shelf_id','date']].drop_duplicates().reset_index(drop=True)

    result = path.groupby(['person_id', 'date'])['shelf_id'].apply(list).reset_index()
    path_count={}
    def create_pairwise_path(shelf_ids):
        pairs = []
        if len(shelf_ids) ==1:
            pairs.append((shelf_ids[0],None))
        else:
            for i in range(len(shelf_ids) - 1):
                pairs.append((shelf_ids[i],shelf_ids[i + 1]))
        return pairs
    result['path'] = result['shelf_id'].apply(create_pairwise_path)
    result.drop(columns='shelf_id', inplace=True)
    all_paths = [path for sublist in result['path'] for path in sublist]
    from collections import Counter
    path_counts = Counter(all_paths)
    most_common_path, most_common_count = path_counts.most_common(1)[0]
> ANSWER:
(7,0)
97 times in a week

**note**: df_cus[timestamp, shelf id]

___
05. (7đ) Trong 3 nhóm tuổi sau: Thiếu niên (18 - 30), Trung niên (31 - 60), Cao tuổi: (> 60), nhóm tuổi nào có **số người đi siêu thị nhiều nhất?**
    shopper=df_cust[['person_id','age', 'date']]
    bins = [18, 31, 61, shopper['age'].max() + 1]
    labels = [1, 2, 3]

    shopper['age_group'] = pd.cut(shopper.age, bins=bins, labels=labels, right=False)
    shopper=shopper.drop(columns=['age'])
    shopper.groupby('age_group').count().reset_index(drop=True)

> ANSWER: trung nien
person_id	date
0	3574	3574
1	7617	7617
2	4203	4203

**note**: df_cus

---
02. (5đ) Thống kê ***5 mặt hàng*** thường **được cầm lên rồi trả lại nhiều nhất**?

03. (5đ) Các nhóm khách hàng theo độ tuổi (Thiếu niên: 18 - 30; Trung niên: 31 - 60; Cao tuổi: > 60) **mua mặt hàng nào nhiều nhất?**
06. (5đ) Top 5 các mặt hàng giảm giá được người dùng **mua** nhiều nhất?
07. (5đ) Top 5 các mặt hàng được chạy quảng cáo được người dùng **mua** nhiều nhất?
**note**: df_cus (primary_key = 'Item ID','Shelf ID')
___ 
04. (5đ) Ngày nào trong tuần có doanh thu cao nhất?
**note**: df_cus (primary_key = 'Item ID','Shelf ID'), get bought pk by day and join df_item['Price_vnd']
___
09. (7đ) Top 3 quầy hàng có số sản phẩm được **mua** nhiều nhất?
    **note**: df_shelf 
___

11. (10đ) Câu hỏi mở: Có những mặt hàng nào cần được sắp xếp lại trong cửa hàng không?
(Phần trình bày của thí sinh không quá 500 từ)

12. (10đ) Câu hỏi mở: Phân tích hiệu quả của các chiến dịch quảng cáo, sale đối với các mặt
hàng. (Phần trình bày của thí sinh không quá 500 từ)