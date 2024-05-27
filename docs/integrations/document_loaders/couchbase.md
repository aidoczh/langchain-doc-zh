# Couchbase

[Couchbase](http://couchbase.com/) 是一款备受赞誉的分布式 NoSQL 云数据库，为您的云端、移动、人工智能和边缘计算应用程序提供无与伦比的多功能性、性能、可伸缩性和财务价值。

## 安装

```python
%pip install --upgrade --quiet  couchbase
```

## 从 Couchbase 查询文档

有关连接到 Couchbase 集群的详细信息，请查看[Python SDK 文档](https://docs.couchbase.com/python-sdk/current/howtos/managing-connections.html#connection-strings)。

有关使用 SQL++（JSON 的 SQL）查询文档的帮助，请查看[文档](https://docs.couchbase.com/server/current/n1ql/n1ql-language-reference/index.html)。

```python
from langchain_community.document_loaders.couchbase import CouchbaseLoader
connection_string = "couchbase://localhost"  # 有效的 Couchbase 连接字符串
db_username = "Administrator"  # 具有对正在查询的存储桶的读取权限的有效数据库用户
db_password = "Password"  # 数据库用户的密码
# 查询是有效的 SQL++ 查询
query = """
    SELECT h.* FROM `travel-sample`.inventory.hotel h 
        WHERE h.country = 'United States'
        LIMIT 1
        """
```

## 创建加载器

```python
loader = CouchbaseLoader(
    connection_string,
    db_username,
    db_password,
    query,
)
```

您可以通过调用加载器的 `load` 方法来获取文档。它将返回包含所有文档的列表。如果要避免此阻塞调用，可以调用 `lazy_load` 方法，该方法返回一个迭代器。

```python
docs = loader.load()
print(docs)
```

```output
[Document(page_content='address: 8301 Hollister Ave\nalias: None\ncheckin: 12PM\ncheckout: 4PM\ncity: Santa Barbara\ncountry: United States\ndescription: Located on 78 acres of oceanfront property, this resort is an upscale experience that caters to luxury travelers. There are 354 guest rooms in 19 separate villas, each in a Spanish style. Property amenities include saline infinity pools, a private beach, clay tennis courts, a 42,000 foot spa and fitness center, and nature trails through the adjoining wetland and forest. The onsite Miro restaurant provides great views of the coast with excellent food and service. With all that said, you pay for the experience, and this resort is not for the budget traveler.  In addition to quoted rates there is a $25 per day resort fee that includes a bottle of wine in your room, two bottles of water, access to fitness center and spa, and internet access.\ndirections: None\nemail: None\nfax: None\nfree_breakfast: True\nfree_internet: False\nfree_parking: False\ngeo: {\'accuracy\': \'ROOFTOP\', \'lat\': 34.43429, \'lon\': -119.92137}\nid: 10180\nname: Bacara Resort & Spa\npets_ok: False\nphone: None\nprice: $300-$1000+\npublic_likes: [\'Arnoldo Towne\', \'Olaf Turcotte\', \'Ruben Volkman\', \'Adella Aufderhar\', \'Elwyn Franecki\']\nreviews: [{\'author\': \'Delmer Cole\', \'content\': "Jane and Joyce make every effort to see to your personal needs and comfort. The rooms take one back in time to the original styles and designs of the 1800\'s. A real connection to local residents, the 905 is a regular tour stop and the oldest hotel in the French Quarter. My wife and I prefer to stay in the first floor rooms where there is a sitting room with TV, bedroom, bath and kitchen. The kitchen has a stove and refrigerator, sink, coffeemaker, etc. Plus there is a streetside private entrance (very good security system) and a covered balcony area with seating so you can watch passersby. Quaint, cozy, and most of all: ORIGINAL. No plastic remods. Feels like my great Grandmother\'s place. While there are more luxurious places to stay, if you want the real flavor and eclectic style of N.O. you have to stay here. It just FEELS like New Orleans. The location is one block towards the river from Bourbon Street and smack dab in the middle of everything. Royal street is one of the nicest residential streets in the Quarter and you can walk back to your room and get some peace and quiet whenever you like. The French Quarter is always busy so we bring a small fan to turn on to make some white noise so we can sleep more soundly. Works great. You might not need it at the 905 but it\'s a necessity it if you stay on or near Bourbon Street, which is very loud all the time. Parking tips: You can park right in front to unload and it\'s only a couple blocks to the secure riverfront parking area. Plus there are several public parking lots nearby. My strategy is to get there early, unload, and drive around for a while near the hotel. It\'s not too hard to find a parking place but be careful about where it is. Stay away from corner spots since streets are narrow and delivery trucks don\'t have the room to turn and they will hit your car. Take note of the signs. Tuesday and Thursday they clean the streets and you can\'t park in many areas when they do or they will tow your car. Once you find a spot don\'t move it since everything is walking distance. If you find a good spot and get a ticket it will cost $20, which is cheaper than the daily rate at most parking garages. Even if you don\'t get a ticket make sure to go online to N.O. traffic ticket site to check your license number for violations. Some local kids think it\'s funny to take your ticket and throw it away since the fine doubles every month it\'s not paid. You don\'t know you got a ticket but your fine is getting bigger. We\'ve been coming to the French Quarter for years and have stayed at many of the local hotels. The 905 Royal is our favorite.", \'date\': \'2013-12-05 09:27:07 +0300\', \'ratings\': {\'Cleanliness\': 5, \'Location\': 5, \'Overall\': 5, \'Rooms\': 5, \'Service\': 5, \'Sleep Quality\': 5, \'Value\': 5}}, {\'author\': \'Orval Lebsack\', \'content\': \'I stayed there with a friend for a girls trip around St. Patricks Day. This was my third time to NOLA, my first at Chateau Lemoyne. The location is excellent....very easy walking distance to everything, without the chaos of staying right on Bourbon Street. Even though its a Holiday Inn, it still has the historical feel and look of NOLA. The pool looked nice too, even though we never used it. The staff was friendly and helpful. Chateau Lemoyne would be hard to top, considering the price.\', \'date\': \'2013-10-26 15:01:39 +0300\', \'ratings\': {\'Cleanliness\': 5, \'Location\': 5, \'Overall\': 4, \'Rooms\': 4, \'Service\': 4, \'Sleep Quality\': 5, \'Value\': 4}}, {\'author\': \'Hildegard Larkin\', \'content\': \'This hotel is a safe bet for a value stay in French Quarter. Close enough to all sites and action but just out of the real loud & noisy streets. Check in is quick and friendly and room ( king side balcony) while dated was good size and clean. Small balcony with table & chairs is a nice option for evening drink & passing sites below. Down side is no mimi bar fridge ( they are available upon request on a first come basis apparently, so book one when you make initial reservation if necessary) Bathroom is adequate with ok shower pressure and housekeeping is quick and efficient. TIP; forget paying high price for conducted local tours, just take the red trams to end of line and back and then next day the green tram to cross town garden district and zoo and museums. cost for each ride $2.00 each way!! fantastic. Tip: If you stay during hot weather make sure you top up on ice early as later guests can "run the machine dry" for short time. Overall experience met expectations and would recommend for value stay.\', \'date\': \'2012-01-01 18:48:30 +0300\', \'ratings\': {\'Cleanliness\': 4, \'Location\': 4, \'Overall\': 4, \'Rooms\': 3, \'Service\': 4, \'Sleep Quality\': 3, \'Value\': 4}}, {\'author\': \'Uriah Rohan\', \'content\': \'The Chateau Le Moyne Holiday Inn is in a perfect location in the French Quarter, a block away from the craziness on Bourbon St. We got a fantastic deal on Priceline and were expecting a standard room for the price. The pleasant hotel clerk upgraded our room much to our delight, without us asking and the concierge also went above an beyond to assist us with information and suggestions for places to dine and possessed an "can do" attitude. Nice pool area to cool off in during the midday NOLA heat. It is definitely a three star establishment, not super luxurious but the beds were comfy and the location superb! If you can get a deal on Priceline, etc, it\\\'s a great value.\', \'date\': \'2014-08-04 15:17:49 +0300\', \'ratings\': {\'Cleanliness\': 4, \'Location\': 5, \'Overall\': 4, \'Rooms\': 3, \'Service\': 5, \'Sleep Quality\': 4, \'Value\': 4}}]\nstate: California\ntitle: Goleta\ntollfree: None\ntype: hotel\nurl: http://www.bacararesort.com/\nvacancy: True')]
```

```python
docs_iterator = loader.lazy_load()
for doc in docs_iterator:
    print(doc)
    break
```

```output```

```markdown
地址: 8301 Hollister Ave
别名: 无
入住时间: 下午12点
退房时间: 下午4点
城市: 圣巴巴拉
国家: 美国
描述: 该度假村占地78英亩，位于海滨地区，是一处迎合奢华旅行者的高档体验场所。共有19栋西班牙风格的别墅，共354间客房。度假村设施包括盐水无边泳池、私人海滩、粘土网球场、面积42000平方英尺的水疗中心和健身中心，以及通往毗邻湿地和森林的自然步道。度假村内的Miro餐厅可以欣赏到海岸线的美景，提供美味佳肴和优质服务。总的来说，你需要为这样的体验付出代价，这个度假村并不适合预算有限的旅行者。除了报价之外，还需支付每天25美元的度假费，其中包括客房内一瓶葡萄酒、两瓶水、健身中心和水疗中心的使用，以及上网服务。
电子邮件: 无
传真: 无
免费早餐: 是
免费上网: 否
免费停车: 否
地理位置: {'准确度': '屋顶', '纬度': 34.43429, '经度': -119.92137}
ID: 10180
名称: 巴卡拉度假村及水疗中心
可携带宠物: 否
电话: 无
价格: $300-$1000+
公共点赞: ['Arnoldo Towne', 'Olaf Turcotte', 'Ruben Volkman', 'Adella Aufderhar', 'Elwyn Franecki']
评论: [{'作者': 'Delmer Cole', '内容': "Jane and Joyce make every effort to see to your personal needs and comfort. The rooms take one back in time to the original styles and designs of the 1800's. A real connection to local residents, the 905 is a regular tour stop and the oldest hotel in the French Quarter. My wife and I prefer to stay in the first floor rooms where there is a sitting room with TV, bedroom, bath and kitchen. The kitchen has a stove and refrigerator, sink, coffeemaker, etc. Plus there is a streetside private entrance (very good security system) and a covered balcony area with seating so you can watch passersby. Quaint, cozy, and most of all: ORIGINAL. No plastic remods. Feels like my great Grandmother's place. While there are more luxurious places to stay, if you want the real flavor and eclectic style of N.O. you have to stay here. It just FEELS like New Orleans. The location is one block towards the river from Bourbon Street and smack dab in the middle of everything. Royal street is one of the nicest residential streets in the Quarter and you can walk back to your room and get some peace and quiet whenever you like. The French Quarter is always busy so we bring a small fan to turn on to make some white noise so we can sleep more soundly. Works great. You might not need it at the 905 but it's a necessity it if you stay on or near Bourbon Street, which is very loud all the time. Parking tips: You can park right in front to unload and it's only a couple blocks to the secure riverfront parking area. Plus there are several public parking lots nearby. My strategy is to get there early, unload, and drive around for a while near the hotel. It's not too hard to find a parking place but be careful about where it is. Stay away from corner spots since streets are narrow and delivery trucks don't have the room to turn and they will hit your car. Take note of the signs. Tuesday and Thursday they clean the streets and you can't park in many areas when they do or they will tow your car. Once you find a spot don't move it since everything is walking distance. If you find a good spot and get a ticket it will cost $20, which is cheaper than the daily rate at most parking garages. Even if you don't get a ticket make sure to go online to N.O. traffic ticket site to check your license number for violations. Some local kids think it's funny to take your ticket and throw it away since the fine doubles every month it's not paid. You don't know you got a ticket but your fine is getting bigger. We've been coming to the French Quarter for years and have stayed at many of the local hotels. The 905 Royal is our favorite.", '日期': '2013-12-05 09:27:07 +0300', '评分': {'清洁度': 5, '位置': 5, '整体': 5, '房间': 5, '服务': 5, '睡眠质量': 5, '价值': 5}}, {'作者': 'Orval Lebsack', '内容': 'I stayed there with a friend for a girls trip around St. Patricks Day. This was my third time to NOLA, my first at Chateau Lemoyne. The location is excellent....very easy walking distance to everything, without the chaos of staying right on Bourbon Street. Even though its a Holiday Inn, it still has the historical feel and look of NOLA. The pool looked nice too, even though we never used it. The staff was friendly and helpful. Chateau Lemoyne would be hard to top, considering the price.', '日期': '2013-10-26 15:01:39 +0300', '评分': {'清洁度': 5, '位置': 5, '整体': 4, '房间': 4, '服务': 4, '睡眠质量': 5, '价值': 4}}, {'作者': 'Hildegard Larkin', '内容': 'This hotel is a safe bet for a value stay in French Quarter. Close enough to all sites and action but just out of the real loud & noisy streets. Check in is quick and friendly and room ( king side balcony) while dated was good size and clean. Small balcony with table & chairs is a nice option for evening drink & passing sites below. Down side is no mimi bar fridge ( they are available upon request on a first come basis apparently, so book one when you make initial reservation if necessary) Bathroom is adequate with ok shower pressure and housekeeping is quick and efficient. TIP; forget paying high price for conducted local tours, just take the red trams to end of line and back and then next day the green tram to cross town garden district and zoo and museums. cost for each ride $2.00 each way!! fantastic. Tip: If you stay during hot weather make sure you top up on ice early as later guests can "run the machine dry" for short time. Overall experience met expectations and would recommend for value stay.', '日期': '2012-01-01 18:48:30 +0300', '评分': {'清洁度': 4, '位置': 4, '整体': 4, '房间': 3, '服务': 4, '睡眠质量': 3, '价值': 4}}, {'作者': 'Uriah Rohan', '内容': 'The Chateau Le Moyne Holiday Inn is in a perfect location in the French Quarter, a block away from the craziness on Bourbon St. We got a fantastic deal on Priceline and were expecting a standard room for the price. The pleasant hotel clerk upgraded our room much to our delight, without us asking and the concierge also went above an beyond to assist us with information and suggestions for places to dine and possessed an "can do" attitude. Nice pool area to cool off in during the midday NOLA heat. It is definitely a three star establishment, not super luxurious but the beds were comfy and the location superb! If you can get a deal on Priceline, etc, it's a great value.', '日期': '2014-08-04 15:17:49 +0300', '评分': {'清洁度': 4, '位置': 5, '整体': 4, '房间': 3, '服务': 5, '睡眠质量': 4, '价值': 4}}]
州: 加利福尼亚
标题: 戈莱塔
免费电话: 无
类型: 酒店
网址: [巴卡拉度假村及水疗中心](http://www.bacararesort.com/)
空房: 是
```

```python
loader_with_selected_fields = CouchbaseLoader(
    connection_string,
    db_username,
    db_password,
    query,
    page_content_fields=[
        "address",
        "name",
        "city",
        "phone",
        "country",
        "geo",
        "description",
        "reviews",
    ],
    metadata_fields=["id"],
)
docs_with_selected_fields = loader_with_selected_fields.load()
print(docs_with_selected_fields)
```

```output
[Document(page_content='address: 8301 Hollister Ave\ncity: Santa Barbara\ncountry: United States\ndescription: Located on 78 acres of oceanfront property, this resort is an upscale experience that caters to luxury travelers. There are 354 guest rooms in 19 separate villas, each in a Spanish style. Property amenities include saline infinity pools, a private beach, clay tennis courts, a 42,000 foot spa and fitness center, and nature trails through the adjoining wetland and forest. The onsite Miro restaurant provides great views of the coast with excellent food and service. With all that said, you pay for the experience, and this resort is not for the budget traveler.  In addition to quoted rates there is a $25 per day resort fee that includes a bottle of wine in your room, two bottles of water, access to fitness center and spa, and internet access.\ngeo: {\'accuracy\': \'ROOFTOP\', \'lat\': 34.43429, \'lon\': -119.92137}\nname: Bacara Resort & Spa\nphone: None\nreviews: [{\'author\': \'Delmer Cole\', \'content\': "Jane and Joyce make every effort to see to your personal needs and comfort. The rooms take one back in time to the original styles and designs of the 1800\'s. A real connection to local residents, the 905 is a regular tour stop and the oldest hotel in the French Quarter. My wife and I prefer to stay in the first floor rooms where there is a sitting room with TV, bedroom, bath and kitchen. The kitchen has a stove and refrigerator, sink, coffeemaker, etc. Plus there is a streetside private entrance (very good security system) and a covered balcony area with seating so you can watch passersby. Quaint, cozy, and most of all: ORIGINAL. No plastic remods. Feels like my great Grandmother\'s place. While there are more luxurious places to stay, if you want the real flavor and eclectic style of N.O. you have to stay here. It just FEELS like New Orleans. The location is one block towards the river from Bourbon Street and smack dab in the middle of everything. Royal street is one of the nicest residential streets in the Quarter and you can walk back to your room and get some peace and quiet whenever you like. The French Quarter is always busy so we bring a small fan to turn on to make some white noise so we can sleep more soundly. Works great. You might not need it at the 905 but it\'s a necessity it if you stay on or near Bourbon Street, which is very loud all the time. Parking tips: You can park right in front to unload and it\'s only a couple blocks to the secure riverfront parking area. Plus there are several public parking lots nearby. My strategy is to get there early, unload, and drive around for a while near the hotel. It\'s not too hard to find a parking place but be careful about where it is. Stay away from corner spots since streets are narrow and delivery trucks don\'t have the room to turn and they will hit your car. Take note of the signs. Tuesday and Thursday they clean the streets and you can\'t park in many areas when they do or they will tow your car. Once you find a spot don\'t move it since everything is walking distance. If you find a good spot and get a ticket it will cost $20, which is cheaper than the daily rate at most parking garages. Even if you don\'t get a ticket make sure to go online to N.O. traffic ticket site to check your license number for violations. Some local kids think it\'s funny to take your ticket and throw it away since the fine doubles every month it\'s not paid. You don\'t know you got a ticket but your fine is getting bigger. We\'ve been coming to the French Quarter for years and have stayed at many of the local hotels. The 905 Royal is our favorite.", \'date\': \'2013-12-05 09:27:07 +0300\', \'ratings\': {\'Cleanliness\': 5, \'Location\': 5, \'Overall\': 5, \'Rooms\': 5, \'Service\': 5, \'Sleep Quality\': 5, \'Value\': 5}}, {\'author\': \'Orval Lebsack\', \'content\': \'I stayed there with a friend for a girls trip around St. Patricks Day. This was my third time to NOLA, my first at Chateau Lemoyne. The location is excellent....very easy walking distance to everything, without the chaos of staying right on Bourbon Street. Even though its a Holiday Inn, it still has the historical feel and look of NOLA. The pool looked nice too, even though we never used it. The staff was friendly and helpful. Chateau Lemoyne would be hard to top, considering the price.\', \'date\': \'2013-10-26 15:01:39 +0300\', \'ratings\': {\'Cleanliness\': 5, \'Location\': 5, \'Overall\': 4, \'Rooms\': 4, \'Service\': 4, \'Sleep Quality\': 5, \'Value\': 4}}, {\'author\': \'Hildegard Larkin\', \'content\': \'This hotel is a safe bet for a value stay in French Quarter. Close enough to all sites and action but just out of the real loud & noisy streets. Check in is quick and friendly and room ( king side balcony) while dated was good size and clean. Small balcony with table & chairs is a nice option for evening drink & passing sites below. Down side is no mimi bar fridge ( they are available upon request on a first come basis apparently, so book one when you make initial reservation if necessary) Bathroom is adequate with ok shower pressure and housekeeping is quick and efficient. TIP; forget paying high price for conducted local tours, just take the red trams to end of line and back and then next day the green tram to cross town garden district and zoo and museums. cost for each ride $2.00 each way!! fantastic. Tip: If you stay during hot weather make sure you top up on ice early as later guests can "run the machine dry" for short time. Overall experience met expectations and would recommend for value stay.\', \'date\': \'2012-01-01 18:48:30 +0300\', \'ratings\': {\'Cleanliness\': 4, \'Location\': 4, \'Overall\': 4, \'Rooms\': 3, \'Service\': 4, \'Sleep Quality\': 3, \'Value\': 4}}, {\'author\': \'Uriah Rohan\', \'content\': \'The Chateau Le Moyne Holiday Inn is in a perfect location in the French Quarter, a block away from the craziness on Bourbon St. We got a fantastic deal on Priceline and were expecting a standard room for the price. The pleasant hotel clerk upgraded our room much to our delight, without us asking and the concierge also went above an beyond to assist us with information and suggestions for places to dine and possessed an "can do" attitude. Nice pool area to cool off in during the midday NOLA heat. It is definitely a three star establishment, not super luxurious but the beds were comfy and the location superb! If you can get a deal on Priceline, etc, it\\\'s a great value.\', \'date\': \'2014-08-04 15:17:49 +0300\', \'ratings\': {\'Cleanliness\': 4, \'Location\': 5, \'Overall\': 4, \'Rooms\': 3, \'Service\': 5, \'Sleep Quality\': 4, \'Value\': 4}}]', metadata={'id': 10180})]
```