# I. Tổng quan đề tài

## 1. Mô tả dữ liệu

- Đây là bộ dữ liệu về danh sách các trò chơi điện tử trên nền tảng Steam Game được đóng góp lần cuối vào tháng 5 năm 2019
- Bộ dữ liệu gồm 27076 dòng và 18 cột thuộc tính
- Link dataset: www.kaggle.com/nikdavis/steam-store-games

## 2. Kiểu dữ liệu

| STT | Tên thuộc tính   | Second Header                                            | Kiểu dữ liệu  |
| --- | ---------------- | -------------------------------------------------------- | ------------- |
| 1   | Appid            | Mã trò chơi                                              | Int           |
| 2   | Name             | Tên trò chơi                                             | Nvarchar(255) |
| 3   | Release_date     | Ngày phát hành                                           | Datetime      |
| 4   | English          | Có hỗ trợ ngôn ngữ tiếng anh (1 là có, 0 là không)       | Int           |
| 5   | Developer        | Nhà phát triển                                           | Nvarchar(255) |
| 6   | Publisher        | Nhà phát hành                                            | Nvarchar(255) |
| 7   | Required_age     | Độ tuổi yêu cầu tối thiểu theo chuẩn PEGI Vương quốc Anh | Int           |
| 8   | Categories       | Danh mục trò chơi                                        | Nvarchar(255) |
| 9   | Genres           | Thể loại                                                 | Nvarchar(255) |
| 10  | Steampy_tags     | Danh sách thẻ trò chơi do người dung bình chọn           | Nvarchar(255) |
| 11  | Achievements     | Thành tích trong trò chơi                                | Float         |
| 12  | Positive_Ratings | Lượt đánh giá tích cực                                   | Float         |
| 13  | Negative_Ratings | Lượt đánh giá tiêu cực                                   | Float         |
| 14  | Average_Playtime | Thời gian chơi trung bình                                | Float         |
| 15  | Median_Playtime  | Thời gian chơi trung vị                                  | Float         |
| 16  | Owners           | Lượt tải về                                              | Nvarchar(255) |
| 17  | Price            | Giá trò chơi                                             | Float         |

## 2. Thiết kế kho dữ liệu
- **Lược đồ kho dữ liệu ( Lược đồ hình sao )**

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092031543880466463/image.png?width=543&height=434">
</p>

# II. SSIS
- Thực hiện SSIS\
https://youtu.be/OgGwR1aCDag

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092033085702418442/image.png?width=583&height=434">
</p>

# III. SSAS
## 1. Quá trình SSAS
https://youtu.be/p7-0ASqhq3Y
## 2. Ngôn ngữ MDX
### 2.1. **Những game được bán có giá lớn hơn 90$ và sắp xếp giảm dần theo giá**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Những game được bán có giá lớn hơn 90$ và sắp xếp giảm dần theo giá
select {[Measures].[Price]} on columns
,order(
	filter(
		[DIM Game].[Name].[Name],
		[Measures].[Price] >90
	),
		[Measures].[Price] 
, desc)
on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092034296845439027/image.png">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092035096300761088/image.png?width=876&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092035362647449660/image.png?width=801&height=434">
</p>

### 2.2. **Những nhà phát triển có chữ cái đầu là H và được sắp xếp tăng dần theo tổng giá các game đã phát triển**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Những nhà phát triển có chữ cái đầu là H và được sắp xếp tăng dần theo tổng giá các game đã phát triển
select [Measures].[Price] on columns
, filter(
	order(
	[DIM Developer].[Developer].[Developer].members,
	[Measures].[Price]
	,asc
	)
	,left([DIM Developer].[Developer].CurrentMember.Name, 1) = "H"
	) on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092035813111513098/image.png?width=805&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092035884280459334/image.png?width=746&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036003868446780/image.png?width=796&height=434">
</p>

### 2.3. **Trong mỗi quý năm 2010 truy vấn ra 3 nhà phát triển có nhiều game nhất**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Trong mỗi quý năm 2010 truy vấn ra 3 nhà phát triển có nhiều game nhất
select {[Measures].[Number of Game]} on columns
, non empty Generate(
	[DIM Time].[Quarter].[Quarter].members,
	topcount(
		[DIM Developer].[Developer].[Developer].members
		* [DIM Time].[Quarter].currentmember
		,3
		,[Measures].[Number of Game]
		)
	)on rows
from [Steam DW]
where [DIM Time].[Year].&[2010];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036333486231695/image.png?width=411&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036333712707686/image.png?width=609&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036333947592724/image.png?width=818&height=434">
</p>

### 2.4. **Những game có lượt đánh giá tiêu cực và tích cực > 3000**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Những game có lượt đánh giá tiêu cực và tích cực > 3000
select {[Measures].[Positive Ratings], [Measures].[Negative Ratings]}on columns
, filter(filter(
	order(
		[DIM Game].[Name].[Name].members,
		[Measures].[Positive Ratings]
		,desc
		)
	,[Measures].[Positive Ratings] > 3000 
	) ,[Measures].[Negative Ratings] > 3000) on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036739331268650/image.png">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036922249056307/image.png?width=638&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092036740342091796/image.png?width=798&height=434">
</p>

### 2.5. **Trong mỗi quý đầu mỗi năm truy vấn ra 3 nhà phát hành có số game giảm dần**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Trong mỗi quý đầu mỗi năm truy vấn ra 3 nhà phát hành có số game giảm dần
select [Measures].[Number of Game] on columns
, generate(
	generate(
		[DIM Time].[Y Q M].[Year].members,
		{[DIM Time].[Y Q M].currentmember.firstchild}
	),
	topcount(
	{[DIM Time].[Y Q M].currentmember.children }   * [DIM Time].[Y Q M D].[Year]
	* [DIM Developer].[Developer].[Developer].members
	,3
	,[Measures].[Number of Game]
	)
) on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092037137672704080/image.png?width=551&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092037137907589121/image.png?width=614&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092037138146656356/image.png?width=806&height=434">
</p>

### 2.6. **Tổng lượt đánh giá tích cực của quý hiện tại trừ đi tổng đánh giá tích cực của quý trước. Truy vấn tổng đánh giá quý hiện tại và tổng đánh giá quý trước**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Tổng lượt đánh giá tích cực của quý hiện tại trừ đi tổng đánh giá tích cực của quý trước. Truy vấn tổng đánh giá quý hiện tại và tổng đánh giá quý trước
with 
member [Measures].[Positive Ratings Prev] as
		([DIM Time].[Y Q].currentmember.prevmember,
		[Measures].[Positive Ratings])
member [Measures].[Positive Ratings hieu] as 
		([Measures].[Positive Ratings] - [Measures].[Positive Ratings Prev])
select { [Measures].[Positive Ratings], 
		[Measures].[Positive Ratings Prev], 
		[Measures].[Positive Ratings hieu]} on columns
,crossjoin([DIM Time].[Y Q].[Quarter],
			[DIM Time].[Year].[Year]) on rows
from [Steam DW]; 
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092037872456040468/image.png?width=860&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092037872686739597/image.png?width=623&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092037873018097704/image.png?width=801&height=434">
</p>

### 2.7. **Truy vấn những nhà phát triển có game mà số lượt tải về là 2000000-5000000 và tổng giá của các game đó**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Truy vấn những nhà phát triển có game mà số lượt tải về là 2000000-5000000 và tổng giá của các game đó
select[Measures].[Price] on columns

,filter(

	crossjoin(
		[DIM Developer].[Developer].[Developer]
		,[DIM Owners].[Owners].&[2000000-5000000])
		,[DIM Owners].[Owners].&[2000000-5000000]) on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038153088552960/image.png?width=966&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038153537323048/image.png?width=639&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038153830940752/image.png?width=800&height=434">
</p>

### 2.8. **Truy vấn 10 nhà phát hành có tổng thời gian chơi trung bình lớn nhất trong quý 1 năm 2012**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Truy vấn 10 nhà phát hành có tổng thời gian chơi trung bình lớn nhất trong quý 1 năm 2012
select [Measures].[Average Playtime] on columns
,non empty topcount( order(
		[DIM Developer].[Developer].[Developer].members
		,[Measures].[Average Playtime]
		,desc)
		,10
		,[Measures].[Average Playtime])on rows
from [Steam DW]
where [DIM Time].[Y Q].[Quarter].&[1]&[2012];	
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038456147984405/image.png">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038456361881710/image.png?width=650&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038456655499275/image.png?width=789&height=434">
</p>

### 2.9. **Top 5 nhà phát triển có số lượng game nhiều nhất trong năm 2012 đến 2014**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Top 5 nhà phát triển có số lượng game nhiều nhất trong năm 2012 đến 2014
select [Measures].[Number of Game] on columns
,non empty topcount(
order(
 generate(
	[DIM Time].[Year].[Year].members,
	[DIM Developer].[Developer].[Developer]
	)
	,[Measures].[Number of Game],
	desc),5
,[Measures].[Number of Game])on rows
from [Steam DW]
where {[DIM Time].[Year].&[2005], [DIM Time].[Year].&[2013],[DIM Time].[Year].&[2010]};
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038843634556978/image.png">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038843877830676/image.png?width=627&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092038844314042449/image.png?width=802&height=434">
</p>

### 2.10. **Top 3 nhà phát triển có tổng game nhiều nhất theo quý trong năm 2013 và 2014**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Top 3 nhà phát triển có tổng game nhiều nhất theo quý trong năm 2013 và 2014
select [Measures].[Number of Game] on columns
,non empty generate(
		[DIM Time].[Y Q].[Quarter].members
		,topcount(
			[DIM Developer].[Developer].[Developer]*
			 [DIM Time].[Y Q].currentmember 
			,3
			,[Measures].[Number of Game]
		)

	)on rows
from [Steam DW]
where {[DIM Time].[Year].&[2013],[DIM Time].[Year].&[2014]};
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039226612265091/image.png?width=461&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039226880696360/image.png?width=634&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039227157512222/image.png?width=806&height=434">
</p>

### 2.11. **Thống kê theo từng năm tổng số game, tổng thành tựu theo thể loại (Trừ action và gore), gom nhóm theo owners của từng nhà phát triển**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Thống kê theo từng năm tổng số game, tổng thành tựu theo thể loại (Trừ action và gore), gom nhóm theo owners của từng nhà phát triển
select {[Measures].[Number of Game],[Measures].[Achievements]} on columns
,non empty crossjoin(crossjoin(
	[DIM Time].[Year].[Year], 
	except([DIM Genres].[Genres].children,{[DIM Genres].[Genres].&[Action],[DIM Genres].[Genres].&[Gore]}))
	,[DIM Owners].[Owners].[Owners]
	) on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039474545954866/image.png?width=850&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039474793422868/image.png?width=635&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039475103805561/image.png?width=801&height=434">
</p>

### 2.12. **Thống kê số average_playtime, median_playtime theo từng năm**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Thống kê số average_playtime, median_playtime theo từng năm
select {[Measures].[Median Playtime], [Measures].[Average Playtime]} on columns
,non empty generate(
		[DIM Time].[Year].[Year].members
		,[DIM Time].[Year].[Year] 
	) on rows
from [Steam DW];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039744113868890/image.png?width=326&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039744331989032/image.png?width=637&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092039744600416307/image.png?width=807&height=434">
</p>

### 2.13. **Theo từng năm in ra game ở nhà phát triển Valve có giá cao nhất**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Theo từng năm in ra game ở nhà phát triển Valve có giá cao nhất
select [Measures].[Price] on columns
,non empty generate(
	[DIM Time].[Year].[Year].members

	,crossjoin([DIM Time].[Year].[Year],topcount([DIM Game].[Name].[Name]
		
	,1
	,[Measures].[Price]
	) ))on rows
from [Steam DW]
where [DIM Developer].[Developer].&[Valve]
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092040016496181308/image.png?width=479&height=434">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092040016752025640/image.png?width=633&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092040017003692102/image.png?width=804&height=434">
</p>

### 2.14. **In ra 10 nhà phát triển đứng chót có tổng lượt đánh giá tiêu cực > 1000 trong năm 2017**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--In ra 10 nhà phát triển đứng chót có tổng lượt đánh giá tiêu cực > 1000 trong năm 2017
select [Measures].[Negative Ratings] on columns
,non empty head( filter(
		order(
			[DIM Publisher].[Publisher].[Publisher]
			,[Measures].[Negative Ratings]
			,asc
)
,[Measures].[Negative Ratings] > 1000)
,10)on rows
from [Steam DW]
where [DIM Time].[Year].&[2017];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092040246012678174/image.png">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092040246243377243/image.png?width=628&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092040246570524792/image.png?width=793&height=434">
</p>

### 2.15. **Top 3 game cho tuổi 12 thể loại "Action" và độ tuổi quy định là 12 có điểm thành tích cao nhất**
- ## <u >**Câu truy vấn MDX**</u> 
```sql
--Top 3 game cho tuổi 12 thể loại "Action" và độ tuổi quy định là 12 có điểm thành tích cao nhất
select [Measures].[Achievements] on columns
,topcount(
	filter(
		order(
			[DIM Game].[Name].[Name],
			[Measures].[Achievements],
			desc
		)
	,[DIM Genres].[Genres].&[Action]
	)
	,3
	,[Measures].[Achievements]
) on rows
from [Steam DW]
where [DIM Game].[Required Age].&[1.2E1];
```
- Kết quả
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092041096072278071/image.png">
</p>

- ## <u >**Chạy trên VS Studio**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092041096319750144/image.png?width=639&height=434">
</p>

- ## <u >**Chạy trên Pivot Excel**</u> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092041096571392010/image.png?width=801&height=434">
</p>

# IV. SSRS
## 1. Report with Visual Studio
https://youtu.be/ZBbdzbw-cPc

### <u>**Thống kê theo từng năm, tháng số lượng game của từng nhà phát triển và tổng giá**</u>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092041570217361408/image.png?width=359&height=434">
</p>

### <u>**Thống kê theo từng năm, quý, tổng giờ chơi trung bình, trung vị, số lượng đánh giá tích cực, tiêu cực của từng nhà phát triển**</u>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092041570494205952/image.png?width=373&height=434">
</p>

### <u>**Thống kê theo từng năm, gom nhóm theo lượt tải về, số lượng game, tổng điểm thành tích và tổng số lượng đánh giá tích cực của từng nhà phát triển**</u>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092041570775220254/image.png?width=292&height=434">
</p>

## 2. Report with Power BI
### <u>**Thống kê theo từng năm, tháng số lượng game của từng nhà phát triển và tổng giá**</u>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092042310151323729/image.png?width=733&height=434">
</p>

### <u>**Thống kê theo từng năm, quý, tổng giờ chơi trung bình, trung vị, số lượng đánh giá tích cực, tiêu cực của từng nhà phát triển**</u>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092042310394589334/image.png?width=734&height=434">
</p>

### <u>**Thống kê theo từng năm, gom nhóm theo lượt tải về, số lượng game, tổng điểm thành tích và tổng số lượng đánh giá tích cực của từng nhà phát triển**</u>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092042310683992154/image.png?width=733&height=434">
</p>

# V. Data Mining
## 1. Database
-	Bộ dữ liệu gồm 16719 dòng và 16 cột thuộc tính
-	Link dataset: https://www.kaggle.com/rush4ratio/video-game-sales-with-ratings
- Thuộc tính
  - Name: Tên của trò chơi
  - Platform: Nền tảng để tiến hành chơi trò chơi ( 32 nhà phát hành )
  - Year_of_Release: Năm phát hành trò chơi ( Từ 1980 - 2016 )
  - Genre: Thể loại của trò chơi ( 13 thể loại )
  - Publisher: Nhà phát hành của trò chơi ( 583 nhà phát hành )
  - NA_Sales: Doanh số bán hàng ở Bắc Mỹ
  - EU_Sales: Doanh số bán hàng ở Châu Âu (Tính bằng triệu bản)
  - JP_Sales: Doanh số bán hàng tại Nhật Bản (Tính bằng triệu bản)
  - Other_Sales: Doanh số bán hàng ở các nước khác (Tính bằng triệu bản)
  - Global_Sales: Doanh số bán hàng trên toàn cầu (Tính bằng triệu bản)
  - Critic_score: Điểm số được đánh giá bởi Metacritic 
    - Theo thang điểm từ 0 - 100:
      - Từ 0 - 19 điểm: Game được cho là quá tệ
      - Từ 20 - 49 điểm: Game không được hay lắm
      - Từ 50 - 74 điểm: Game ở mức độ trung bình, có cả khen và chê
      - Từ 75 - 89 điểm: Game khá hay và nên chơi
      - Từ 90 - 100 điểm: Game hay và được đánh giá cao trên toàn cầu
  - Criticcount: Số lượng nhà phê bình tham gia để đưa ra điểm phê bình
  - User_score: Điểm số trung bình của người dùng Metacritic tham gia đánh giá ( Từ 0 đến 10 và tbd (To be discussed - Sẽ được thảo luận )) 
  - User_Count: Số lượng người dùng đã cho điểm 
  - Developer: Nhà phát triển trò chơi ( 1697 nhà phát triển )
  - Rating: Đánh giá trò chơi theo ESRB ( 8 đánh giá )
    - EC: Early Childhood (Trẻ nhỏ) - ( Game dành cho trẻ nhỏ, phù hợp cho trẻ từ 3 tuổi hoặc 3 tuổi trở lên ) 
    - E: Everyone (Mọi người) - ( Phù hợp với mọi lứa tuổi )
    - K-A: Kid to Adults ( Từ trẻ em tới người lớn ) - Là tên của xếp hạng E trước năm 1998
    - E10+: Everyone 10+( Mọi người từ 10 tuổi trở lên ) - Game mang tính khiêu gợi, bạo lực nhiều hơn, dành cho lứa tuổi từ 10 trở lên
    - T: Teen ( Thiếu niên) - Mang nội dung mạnh hơn E10+ có thể mang tính chất máu me, dành cho lứa tuổi từ 13 trở lên
    - M: Mature 17+ ( Thanh niên 17+ ) - Game phù hợp cho lứa tuổi từ 17 trở lên 
    - AO: Adults Only 18+ ( Chỉ dành cho người lớn ) - Game dành cho người trên 18 tuổi trở lên
    - RP: Rating Pending (RP) ( Đang xếp hạng ) - Sản phẩm đang được trình lên cho ESRB và chờ đánh giá cuối cùng. Rate này chỉ xuất hiện trong quảng cáo trước khi game phát hành 
## 2. Thực hiện Data Mining trên Visual Studio
https://youtu.be/fme2mSc4lSo

## 3. Nhận xét thuật toán
### 3.1. Decision Tree
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092044554179448862/image.png?width=919&height=434">
</p>
- Những game có điểm critic dưới 82 đa phần thuộc đánh giá E dành cho tất cả mọi người bao gồm các thể loại như Puzzle 68,38%, Simulation 42,08%, Adventure 29,36%, Platform 68,36%, Misc 52,18%, Racing 64,7%, Sports 73,86% còn lại là các game có đánh giá T khá cao bao gồm thể loại Role-Playing 50,24%, Strategy 49,87%, Action 28,94%, Fighting 78,50%. Cuối cùng là thể loại Shooter với đánh giá M lớn nhất 46,66%
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092044921151692831/image.png?width=666&height=434">
</p>
- Với những game không có điểm đánh giá critic và không có điểm người dùng đánh giá bao gồm Simulation, Puzzle, Misc, Fighting, Role-Playing và đa phần là không có đánh giá.
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092044930119127170/image.png">
</p>
- Với những game có điểm đánh giá critic >=82 ở thể loại Sport thì đánh giá T là lớn nhất 31,63%, theo sau là M với 30,48% và E với 20,97%. 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092044968052400208/image.png?width=770&height=434">
</p>
- Ở thể loại Racing và Puzzle thì đánh giá E là lớn nhất theo trình tự là 72,59% và 68,06%, theo sau là E10+ với 10,91% và 31,46%. 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092044985613959228/image.png?width=741&height=434">
</p>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092044997391568947/image.png?width=760&height=434">
</p>
- Thể loại Shooter và Action thì đánh giá M lớn nhất với 70,64% và 58,13%, tiếp đó là T với 24,37% và 20,9%
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092045035761061888/image.png?width=925&height=434">
</p>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092045970080010250/image.png?width=750&height=434">
</p>
- Dependency Network của cây quyết định 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092045067667116062/image.png?width=749&height=434">
</p>

=> **Thuộc tính giảm dần: Critic Score -> Genre -> User Score**

### 3.2. Clustering
- Đổi màu cụm đậm nhất là High (khả năng không có đánh giá cao), cụm nhạt nhất là Low (khả năng không có đánh giá thấp)
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046306559676527/image.png?width=792&height=434">
</p>
- Ở phần high đa số những game không có đánh giá lớn nhất với 83,8%, theo sau là E với 11,4% và T 2,8%, E10+ 2% bao gồm thể loại Misc, Sport và Strategy
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046336943214723/image.png?width=745&height=434">
</p>
- Ở low đa số những game có đánh giá E là lớn nhất với 57,7%, theo sau là T với 18,1% và E10+ với 11,6%, cuối cùng là M với 5,6% và chưa được đánh giá là 7%
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046351388393492/image.png?width=775&height=434">
</p>

### 3.3. Naive Bayes
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046723079229541/image.png?width=870&height=434">
</p>
- Đánh giá E lớn nhất rơi vào thể loại Sport, AO ở thể loại Action, E10+ ở Strategy, EC ở Adventure, K-A không có đánh giá nào, M ở thể loại Shooter, Chưa được đánh giá nhiều nhất ở Adventure và cuối cùng là T ở Fighting
- 
</br>

~~~
Value: AO
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046736542937108/image.png?width=911&height=434">
</p>

~~~
Value: E
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046794478845972/image.png?width=960&height=434">
</p>

~~~
Value: E10+
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046812900241408/image.png?width=919&height=434">
</p>

~~~
Value: Ec
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046820038938635/image.png?width=943&height=434">
</p>

~~~
Value: M
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046833691406396/image.png?width=923&height=434">
</p>

~~~
Value: RP
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046848845430794/image.png?width=873&height=434">
</p>

~~~
Value: T
~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046864817324112/image.png?width=866&height=434">
</p>

~~~
Value: missing

~~~
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092046869175205908/image.png?width=908&height=434">
</p>

### 3.4. Đánh giá các thuật toán bằng Mining Accuracy Chart
- Line chart cho ta thấy tỷ lệ chính xác của 3 thuật toán khi mining những trường hợp đánh giá của game
- Với value đánh giá là E
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092048349546422312/image.png">
</p>  

  - Microsoft Decision Tree: 0.87 Score.
  - Microsoft Clustering: 0.75 Score.
  - Microsoft Naive Bayes: 0.77 Score.

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092048433247961170/image.png?width=908&height=434">
</p> 

- Với value đánh giá là T
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092048684180578466/image.png">
</p> 

    - Microsoft Decision Tree: 0.87 Score.
    - Microsoft Clustering: 0.84 Score.
    - Microsoft Naive Bayes: 0.70 Score.

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092048684503531570/image.png?width=941&height=434">
</p> 

- Với value đánh giá là M
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092048925592141844/image.png">
</p> 

    - Microsoft Decision Tree: 0.92 Score.
    - Microsoft Clustering: 0.80 Score.
    - Microsoft Naive Bayes: 0.84 Score.

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092048925839597608/image.png?width=894&height=434">
</p> 

- Với value đánh giá là E10+
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049163946033272/image.png">
</p> 

    - Microsoft Decision Tree: 0.76 Score.
    - Microsoft Clustering: 0.73 Score
    - Microsoft Naive Bayes: 0.57 Score.

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049164348698694/image.png?width=953&height=434">
</p> 

- Với value đánh giá là EC
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049400563503114/image.png">
</p> 

    - Microsoft Decision Tree: 0.23 Score.
    - Microsoft Clustering: 0.22 Score.
    - Microsoft Naive Bayes: 0.41 Score.

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049400966172722/image.png?width=946&height=434">
</p> 

- Với value đánh giá là K-A
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049603404255352/image.png">
</p> 

    - Microsoft Decision Tree: 0.66 Score.
    - Microsoft Clustering: 0.66 Score.
    - Microsoft Naive Bayes: 0.66 Score.

<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049603693654026/image.png?width=918&height=434">
</p> 

- Với value đánh giá là RP và AO 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049918987862047/image.png">
</p> 
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049918702661662/image.png">
</p> 

    - Microsoft Decision Tree: 0 Score.
    - Microsoft Clustering: 0 Score.
    - Microsoft Naive Bayes: 0 Score.

* Với RP
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049919222751292/image.png?width=977&height=434">
</p> 

* Với AO
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1092049919461830696/image.png?width=923&height=434">
</p> 