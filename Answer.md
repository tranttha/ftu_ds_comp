
## Assumption 
#### Defining is_buy column

        i_i['is_buy'] = (
        (i_i['returning_item'] == False) &
        (i_i['taking_item_out_of_bag'] == False) & 
        (
                (i_i['putting_item_into_bag_in_the_2nd_time'] == True) | 
                (
                (i_i['putting_item_into_bag_in_the_2nd_time'] != True) & 
                (i_i['putting_item_into_bag'] ==True) & (i_i['taking_item_out_of_bag'] == False)
                )
        )
        ).astype(bool)

---
___
## Questions & Answers

### 1. (5đ) Thống kê ***5 mặt hàng*** có **tổng thời gian nhìn và cầm xem lâu nhất**?

        ans = df_cust.groupby('si_id')[['looking_at_item_(s)', 'holding_the_item_(s)']].sum().reset_index()
        ans['total_time']=ans['looking_at_item_(s)']+ans['holding_the_item_(s)']
        ans=pd.merge(right=ans,left=df_item, how='right', on='si_id')
        ans[['shelf_id'	,'item_id',	'name','total_time']].sort_values(by='total_time', ascending=False).head(5)

> ANSWER: Sữa chua uống Yakult, Sữa chua uống Probi, Sữa ông thọ, Bim bim Oishi, Snack khoai tây Lays

|shelf_id|item_id|name|total_time|
|-|-|-|-|
|7|6|Sữa chua uống Yakult|22896|
|7|7|Sữa chua uống Probi|22896|
|2|8|Sữa ông thọ|13939|
|0|6|Bim bim Oishi|13866|
|0|7|Snack khoai tây Lays|13362|

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
___
### 3. (5đ) Các nhóm khách hàng theo độ tuổi (Thiếu niên: 18 - 30; Trung niên: 31 - 60; Cao tuổi: > 60) **mua** mặt hàng nào nhiều nhất?
        shopper_buy=pd.merge(left=demo[['age_group', 'si_id', 'person_id','timestamp']], right=i_i[['si_id', 'person_id', 'timestamp','is_buy']], how='inner', on=['si_id', 'person_id', 'timestamp'])
        shopper_buy=pd.merge(left=shopper_buy.groupby(['age_group','si_id'])['is_buy'].sum().reset_index(), right=df_item, how='right', on='si_id')
        shopper_buy=shopper_buy[['si_id','age_group', 'is_buy','name']].sort_values(by='is_buy',ascending=False).reset_index(drop=True)
        ans3=shopper_buy.drop_duplicates(subset=['age_group'],keep='first').sort_values(by='age_group',ascending=True)
        ans3

> ANSWER: thieu nien = Bánh trứng Custard, trung nien = Kem tràng tiền, cao tuoi = Bánh trứng Custard

||si_id|age_group|is_buy|name|
|-|-|-|-|-|
|10|04|1|41|Bánh trứng Custard|
|0|70|2|86|Kem tràng tiền|
|4|04|3|50|Bánh trứng Custard|
___
### 4. (5đ) Ngày nào trong tuần có doanh thu cao nhất?

        ans4=pd.merge(left=i_i[['dow', 'si_id', 'is_buy']], right=df_item[['si_id', 'name', 'price_vnd']], how='right', on='si_id')
        ans4[ans4['is_buy']==True].groupby('dow')['price_vnd'].sum().sort_values(ascending=False).

> ANSWER Saturday

|dow||
|-|-|
|Saturday|     174194600|
___
### 5. (7đ) Trong 3 nhóm tuổi sau: Thiếu niên (18 - 30), Trung niên (31 - 60), Cao tuổi: (> 60), nhóm tuổi nào có **số người đi siêu thị nhiều nhất?**

        shopper_freq=df_cust[['person_id','age', 'date']]
        bins = [18, 31, 61, shopper_freq['age'].max() + 1]
        labels = [1, 2, 3]

        shopper_freq['age_group'] = pd.cut(shopper_freq.age, bins=bins, labels=labels, right=False)
        shopper_freq=shopper_freq.drop(columns=['age'])
        ans5=shopper_freq.groupby('age_group').count().reset_index(drop=True)
        ans5

> ANSWER: trung nien

 |person_id|date|
 |-|-|
| 0|3574|3574|
| 1|7617|7617|
| 2|4203|4203|
___
### 6. (5đ) Top 5 các mặt hàng giảm giá được người dùng **mua** nhiều nhất?
        ans6=pd.merge(left=i_i.groupby('si_id')['is_buy'].sum().sort_values(ascending=False), right=df_item, how='left',on='si_id').sort_values(by='is_buy', ascending=False)
        ans6[ans6['discount'] > 0].sort_values(by='is_buy',ascending=False)[['name','si_id', 'is_buy','discount']].set_index('si_id').head(5)

> ANSWER:

|si_id|name|is_buy|discount|
|-|-|-|-|
|04|Bánh trứng Custard|159|10|
|70|Kem tràng tiền|154|5|
|27|Sữa bột Milo|154|15|
|11|Dầu gội Romano|95|25|
|111|Khăn mặt Shine|90|20

___
### 7. (5đ) Top 5 các mặt hàng được chạy quảng cáo được người dùng **mua** nhiều nhất?
        ans7=pd.merge(left=i_i.groupby('si_id')['is_buy'].sum().sort_values(ascending=False), right=df_item, how='left',on='si_id').sort_values(by='is_buy', ascending=False)
        ans7[ans7['marketing_strategy'] == True].sort_values(by='is_buy',ascending=False)[['name','si_id', 'is_buy','marketing_strategy']].set_index('si_id').head(5)


> ANSWER:

|si_id|name|is_buy|marketing_strategy|
|-|-|-|-|
|04|Bánh trứng Custard|159|True|
|70|Kem tràng tiền|154|True|
|27|Sữa bột Milo|154|True|
|111|Khăn mặt Shine|90|True|
|110|Khăn tắm Shine|87|True|

___

### 8. (6đ) Top 3 quầy hàng có thời lượng trung bình quan tâm đến sản phẩm, trên số lượt tương tác, là lâu nhất (quan tâm tương ứng với việc nhìn và cầm xem)?
        si_total_ii_time = df_cust.groupby(['shelf_id'])[['holding_the_item_(s)','looking_at_item_(s)']].sum().reset_index()
        si_total_ii_count= i_i.groupby(['shelf_id'])[['holding_the_item_(s)','looking_at_item_(s)']].sum().reset_index()
        ans8=pd.merge(left=si_total_ii_time, right=si_total_ii_count, how='left', on='shelf_id')
        ans8['avg_ii'] = ans8['holding_the_item_(s)_x']/ans8['holding_the_item_(s)_y'] + ans8['looking_at_item_(s)_x']/ans8['looking_at_item_(s)_y']
        ans8=ans8[['avg_ii', 'shelf_id']].sort_values(by='avg_ii', ascending=False)
        ans8=pd.merge(left=ans8, right=df_shelf, on='shelf_id', how='left' )
        ans8[['shelf_id','description', 'avg_ii']].sort_values(by='avg_ii', ascending=False).head(3)

> ANSWER: 5 7 3  


|shelf_id|description|avg_ii|
|-|-|-|
|5|Quầy gia dụng|62.260184|
|7|Quầy đông lạnh|61.009985|
|3|Quầy thực phẩm|59.845622|

___
### 9. (7đ) Top 3 quầy hàng có số sản phẩm được **mua** nhiều nhất?
        ans9=pd.merge(left=i_i[['shelf_id','is_buy']], right=df_shelf[['shelf_id', 'description', 'number_of_items']], how='left', on='shelf_id')
        ans9[ans9['is_buy']==True].groupby(['shelf_id', 'description', 'number_of_items']).count().reset_index().sort_values(by='is_buy',ascending=False).head(3)

> ANSWER : Quầy hoá mỹ phẩm, Quầy đông lạnh, Quầy bánh kẹo


|shelf_id|description|number_of_items|is_buy|
|-|-|-|-|
|1|Quầy hoá mỹ phẩm|18|840|
|7|Quầy đông lạnh|16|675|
|0|Quầy bánh kẹo|13|586|

___
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

97 times in a week


----

### 11. (10đ) Câu hỏi mở: Có những mặt hàng nào cần được sắp xếp lại trong cửa hàng không?
(Phần trình bày của thí sinh không quá 500 từ)
`<skip>`
___
### 12. (10đ) Câu hỏi mở: Phân tích hiệu quả của các chiến dịch quảng cáo, sale đối với các mặt hàng. (Phần trình bày của thí sinh không quá 500 từ)

***Methodology***: 

I used two machine learning models - logistic regression and random forest classifier - to figure out how discounts and marketing strategies affect customer buying behavior. I prepared the dataset, performed basic data transformation (dropped null values as it is an invalid observation, one hot encoded features, drop unique identifier and action-definition features), split it into training and test sets with, and applied both models to understand what features matter most when predicting if someone will make a purchase.
- For the logistic regression, I standardized the features and looked at the coefficients to see how each variable influenced the probability of a purchase. 
- For the random forest classifier, I started by using its built-in feature importance method. However, it is noted that this method might be biased towards features with lots of unique values, so I also used permutation importance on both test and training sets to further validate the findings.
***Findings***:

- Logistic regression model:
- - Discount and marketing strategy has the highest positive coefficients (0.561731 and 0.481352), indicating strong positive link to purchasing behavior.

- Random forest classifier’s feature importance method:
- - Discount and marketing strategy ranked 7th and 8th in importance.
- - This might be skewed because of bias towards features like moving speed and how long customers look at items.

- The permutation importance method on the test set:
- - Marketing strategy (0.063466) and discount (0.057743) is in the top two spots.
- - This matched up with what I saw in the logistic regression’s coefficient result.

**Models’ performance**:
- Both were decent with no tuning, accuracy 75%.
- The random forest model showed signs it might be overfitting, as the results were quite different between the training and test sets with the test set having poor recall for Buy class.

***Conclusion***: My analysis consistently shows that discounts and marketing strategies are super effective in influencing whether customers buy or not. I'm confident about this because:
- They had high positive coefficients in the logistic regression model.
- They ranked top in permutation importance for the random forest model.
- These results were consistent across different modeling approaches.

*Proposed enhancement* to address the overfitting in the random forest model include: grouping some of those features with lots of unique values, or tweak the model settings to find a better balance.
