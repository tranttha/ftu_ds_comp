
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


## Question 
### 1. (5đ) Thống kê ***5 mặt hàng*** có **tổng thời gian nhìn và cầm xem lâu nhất**?

        ans = df_cust.groupby('si_id')[['looking_at_item_(s)', 'holding_the_item_(s)']].sum().reset_index()
        ans['total_time']=ans['looking_at_item_(s)']+ans['holding_the_item_(s)']
        ans=pd.merge(right=ans,left=df_item, how='right', on='si_id')
        ans[['shelf_id'	,'item_id',	'name','total_time']].sort_values(by='total_time', ascending=False).head(5)

> ANSWER: Sữa chua uống Yakult, Sữa chua uống Probi, Sữa ông thọ, Bim bim Oishi, Snack khoai tây Lays

|shelf_id|item_id|name|total_time|
|-|-|-|-|
|130|7|6|Sữa chua uống Yakult|22896|
|131|7|7|Sữa chua uống Probi|22896|
|39|2|8|Sữa ông thọ|13939|
|9|0|6|Bim bim Oishi|13866|
|10|0|7|Snack khoai tây Lays|13362|

---
### 2. (5đ) Thống kê ***5 mặt hàng*** thường **được cầm lên rồi trả lại nhiều nhất**?

        # picking_up_item & returning_item have no nulls 
        ans2=df_cust[['si_id','picking_up_item','returning_item']]
        ans2=ans2[(ans2['picking_up_item']==True) & (ans2['returning_item']==True)].groupby('si_id').count().sort_values(by='picking_up_item',ascending=False).reset_index()
        ans2=pd.merge(right=ans2, left=df_item[['name', 'si_id']], how='right', on='si_id' )
        ans2=ans2.drop(columns=['picking_up_item'])
        ans2.head(5)

> ANSWER: 

||name|si_id|returning_item|
|-|-|-|-|
|0|4 hộp sữa lúa mạch Milo 180ml|22|134|
|1|Snack khoai tây Lays|07|127|
|2|Mý ý SG Food|712|117|
|3|Nước lẩu Barona|714|116|
|4|Sữa chua Vinamik|72|114|

### 3. (5đ) Các nhóm khách hàng theo độ tuổi (Thiếu niên: 18 - 30; Trung niên: 31 - 60; Cao tuổi: > 60) **mua** mặt hàng nào nhiều nhất?

        shopper_buy=pd.merge(left=demo[['age_group', 'si_id', 'person_id','timestamp']], right=i_i[['si_id', 'person_id', 'timestamp','is_buy']], how='inner', on=['si_id', 'person_id', 'timestamp'])
        shopper_buy=pd.merge(left=shopper_buy.groupby(['age_group','si_id'])['is_buy'].sum().reset_index(), right=df_item, how='right', on='si_id')
        shopper_buy=shopper_buy[['si_id','age_group', 'is_buy','name']].sort_values(by='is_buy',ascending=False).reset_index(drop=True)
        shopper_buy.drop_duplicates(subset=['age_group'],keep='first').sort_values(by='age_group',ascending=True)

> ANSWER: thieu nien = Bánh trứng Custard, trung nien = Kem tràng tiền, cao tuoi = Bánh trứng Custard

||si_id|age_group|is_buy|name|
|-|-|-|-|-|
|0|70|2|86|Kem tràng tiền|
|4|04|3|50|Bánh trứng Custard|
|10|04|1|41|Bánh trứng Custard|

### 4. (5đ) Ngày nào trong tuần có doanh thu cao nhất?

        ans=pd.merge(left=i_i[['dow', 'si_id', 'is_buy']], right=df_item[['si_id', 'name', 'price_vnd']], how='right', on='si_id')
        ans[ans['is_buy']==True].groupby('dow')['price_vnd'].sum().sort_values(ascending=False)

> ANSWER Saturday

|dow||
|-|-|
|Saturday|     174194600|

### 5. (7đ) Trong 3 nhóm tuổi sau: Thiếu niên (18 - 30), Trung niên (31 - 60), Cao tuổi: (> 60), nhóm tuổi nào có **số người đi siêu thị nhiều nhất?**

        shopper_freq=df_cust[['person_id','age', 'date']]
        bins = [18, 31, 61, shopper_freq['age'].max() + 1]
        labels = [1, 2, 3]

        shopper_freq['age_group'] = pd.cut(shopper_freq.age, bins=bins, labels=labels, right=False)
        shopper_freq=shopper_freq.drop(columns=['age'])
        shopper_freq.groupby('age_group').count().reset_index(drop=True)

> ANSWER: trung nien

 |person_id|date|
 |-|-|
| 0|3574|3574|
| 1|7617|7617|
| 2|4203|4203|

### 8. (6đ) Top 3 quầy hàng có thời lượng trung bình quan tâm đến sản phẩm, trên số lượt tương tác, là lâu nhất (quan tâm tương ứng với việc nhìn và cầm xem)?

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

> ANSWER: 5 7 3  


|shelf_id|description|avg_ii|
|-|-|-|
|5|Quầy gia dụng|62.260184|
|7|Quầy đông lạnh|61.009985|
|3|Quầy thực phẩm|59.845622|

**note**: df_cus[timestamp, (shelf id)] sum( timestammp) groupby item id / count(item id)

### 10. (8đ) Người dùng có thói quen di chuyển giữa 2 quầy hàng nào nhiều nhất?
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
> ANSWER: (7,0)
|97 times in a week|

**note**: df_cus[timestamp, shelf id]

___


**note**: df_cus

----
#### define is_buy column

        i_i['is_buy'] = (
            (i_i['returning_item'] == False) &
            (i_i['taking_item_out_of_bag'] == False) & 
            (
                (i_i['putting_item_into_bag_in_the_2nd_time'] == True) | 
                (
                    (i_i['putting_item_into_bag_in_the_2nd_time'] != True) & 
                    (i_i['putting_item_into_bag'] == True)
                )
            )
        ).astype(bool)
----

06. (5đ) Top 5 các mặt hàng giảm giá được người dùng **mua** nhiều nhất?
        result6=pd.merge(left=i_i.groupby('si_id')['is_buy'].sum().sort_values(ascending=False), right=df_item, how='left',on='si_id').sort_values(by='is_buy', ascending=False)
        result6[result6['discount'] > 0].sort_values(by='is_buy',ascending=False)[['name','si_id', 'is_buy','discount']].set_index('si_id').head(5)

> ANSWER:

si_id	name	is_buy	discount

04	Bánh trứng Custard	159	10
70	Kem tràng tiền	154	5
27	Sữa bột Milo	154	15
11	Dầu gội Romano	95	25
111	Khăn mặt Shine	90	20


07. (5đ) Top 5 các mặt hàng được chạy quảng cáo được người dùng **mua** nhiều nhất?

        result7=pd.merge(left=i_i.groupby('si_id')['is_buy'].sum().sort_values(ascending=False), right=df_item, how='left',on='si_id').sort_values(by='is_buy', ascending=False)
        result7[result7['marketing_strategy'] == True].sort_values(by='is_buy',ascending=False)[['name','si_id', 'is_buy','marketing_strategy']].set_index('si_id').head(5)

> ANSWER:

si_id	name	is_buy	marketing_strategy
			
04	Bánh trứng Custard	159	True
70	Kem tràng tiền	154	True
27	Sữa bột Milo	154	True
111	Khăn mặt Shine	90	True
110	Khăn tắm Shine	87	True



09. (7đ) Top 3 quầy hàng có số sản phẩm được **mua** nhiều nhất?

> ANSWER : Quầy hoá mỹ phẩm, Quầy đông lạnh, Quầy bánh kẹo


shelf_id	description	number_of_items	is_buy
1	1	Quầy hoá mỹ phẩm	18	840
7	7	Quầy đông lạnh	16	675
0	0	Quầy bánh kẹo	13	586
    **note**: df_shelf 

___

11. (10đ) Câu hỏi mở: Có những mặt hàng nào cần được sắp xếp lại trong cửa hàng không?
(Phần trình bày của thí sinh không quá 500 từ)

12. (10đ) Câu hỏi mở: Phân tích hiệu quả của các chiến dịch quảng cáo, sale đối với các mặt hàng. (Phần trình bày của thí sinh không quá 500 từ)