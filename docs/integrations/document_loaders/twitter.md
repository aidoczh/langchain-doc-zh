# Twitter

>[Twitter](https://twitter.com/) 是一个在线社交媒体和社交网络服务。

这个加载器使用 `tweepy` Python 包从一组 `Twitter` 用户的推文中获取文本。

您需要使用您的 `Twitter API` 令牌初始化加载器，并且需要传入您想要提取的 Twitter 用户名。

```python
from langchain_community.document_loaders import TwitterTweetLoader
```
```python
%pip install --upgrade --quiet  tweepy
```
```python
loader = TwitterTweetLoader.from_bearer_token(
    oauth2_bearer_token="YOUR BEARER TOKEN",
    twitter_users=["elonmusk"],
    number_tweets=50,  # 默认值为 100
)
# 或者从访问令牌和消费者密钥加载
# loader = TwitterTweetLoader.from_secrets(
#     access_token='YOUR ACCESS TOKEN',
#     access_token_secret='YOUR ACCESS TOKEN SECRET',
#     consumer_key='YOUR CONSUMER KEY',
#     consumer_secret='YOUR CONSUMER SECRET',
#     twitter_users=['elonmusk'],
#     number_tweets=50,
# )
```
```python
documents = loader.load()
documents[:5]
```
```output
[Document(page_content='@MrAndyNgo @REI One store after another shutting down', metadata={'created_at': 'Tue Apr 18 03:45:50 +0000 2023', 'user_info': {'id': 44196397, 'id_str': '44196397', 'name': 'Elon Musk', 'screen_name': 'elonmusk', 'location': 'A Shortfall of Gravitas', 'profile_location': None, 'description': 'nothing', 'url': None, 'entities': {'description': {'urls': []}}, 'protected': False, 'followers_count': 135528327, 'friends_count': 220, 'listed_count': 120478, 'created_at': 'Tue Jun 02 20:12:29 +0000 2009', 'favourites_count': 21285, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 24795, 'lang': None, 'status': {'created_at': 'Tue Apr 18 03:45:50 +0000 2023', 'id': 1648170947541704705, 'id_str': '1648170947541704705', 'text': '@MrAndyNgo @REI One store after another shutting down', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'MrAndyNgo', 'name': 'Andy Ngô 🏳️\u200d🌈', 'id': 2835451658, 'id_str': '2835451658', 'indices': [0, 10]}, {'screen_name': 'REI', 'name': 'REI', 'id': 16583846, 'id_str': '16583846', 'indices': [11, 15]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': 1648134341678051328, 'in_reply_to_status_id_str': '1648134341678051328', 'in_reply_to_user_id': 2835451658, 'in_reply_to_user_id_str': '2835451658', 'in_reply_to_screen_name': 'MrAndyNgo', 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 118, 'favorite_count': 1286, 'favorited': False, 'retweeted': False, 'lang': 'en'}, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': 'C0DEED', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': False, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/44196397/1576183471', 'profile_link_color': '0084B4', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'has_extended_profile': True, 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None, 'translator_type': 'none', 'withheld_in_countries': []}}),
 Document(page_content='@KanekoaTheGreat @joshrogin @glennbeck Large ships are fundamentally vulnerable to ballistic (hypersonic) missiles', metadata={'created_at': 'Tue Apr 18 03:43:25 +0000 2023', 'user_info': {'id': 44196397, 'id_str': '44196397', 'name': 'Elon Musk', 'screen_name': 'elonmusk', 'location': 'A Shortfall of Gravitas', 'profile_location': None, 'description': 'nothing', 'url': None, 'entities': {'description': {'urls': []}}, 'protected': False, 'followers_count': 135528327, 'friends_count': 220, 'listed_count': 120478, 'created_at': 'Tue Jun 02 20:12:29 +0000 2009', 'favourites_count': 21285, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 24795, 'lang': None, 'status': {'created_at': 'Tue Apr 18 03:45:50 +0000 2023', 'id': 1648170947541704705, 'id_str': '1648170947541704705', 'text': '@MrAndyNgo @REI One store after another shutting down', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'MrAndyNgo', 'name': 'Andy Ngô 🏳️\u200d🌈', 'id': 2835451658, 'id_str': '2835451658', 'indices': [0, 10]}, {'screen_name': 'REI', 'name': 'REI', 'id': 16583846, 'id_str': '16583846', 'indices': [11, 15]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': 1648134341678051328, 'in_reply_to_status_id_str': '1648134341678051328', 'in_reply_to_user_id': 2835451658, 'in_reply_to_user_id_str': '2835451658', 'in_reply_to_screen_name': 'MrAndyNgo', 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 118, 'favorite_count': 1286, 'favorited': False, 'retweeted': False, 'lang': 'en'}, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': 'C0DEED', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': False, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/44196397/1576183471', 'profile_link_color': '0084B4', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'has_extended_profile': True, 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None, 'translator_type': 'none', 'withheld_in_countries': []}}),
 Document(page_content='@KanekoaTheGreat The Golden Rule', metadata={'created_at': 'Tue Apr 18 03:37:17 +0000 2023', 'user_info': {'id': 44196397, 'id_str': '44196397', 'name': 'Elon Musk', 'screen_name': 'elonmusk', 'location': 'A Shortfall of Gravitas', 'profile_location': None, 'description': 'nothing', 'url': None, 'entities': {'description': {'urls': []}}, 'protected': False, 'followers_count': 135528327, 'friends_count': 220, 'listed_count': 120478, 'created_at': 'Tue Jun 02 20:12:29 +0000 2009', 'favourites_count': 21285, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 24795, 'lang': None, 'status': {'created_at': 'Tue Apr 18 03:45:50 +0000 2023', 'id': 1648170947541704705, 'id_str': '1648170947541704705', 'text': '@MrAndyNgo @REI One store after another shutting down', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'MrAndyNgo', 'name': 'Andy Ngô 🏳️\u200d🌈', 'id': 2835451658, 'id_str': '2835451658', 'indices': [0, 10]}, {'screen_name': 'REI', 'name': 'REI', 'id': 16583846, 'id_str': '16583846', 'indices': [11, 15]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': 1648134341678051328, 'in_reply_to_status_id_str': '1648134341678051328', 'in_reply_to_user_id': 2835451658, 'in_reply_to_user_id_str': '2835451658', 'in_reply_to_screen_name': 'MrAndyNgo', 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 118, 'favorite_count': 1286, 'favorited': False, 'retweeted': False, 'lang': 'en'}, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': 'C0DEED', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': False, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/44196397/1576183471', 'profile_link_color': '0084B4', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'has_extended_profile': True, 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None, 'translator_type': 'none', 'withheld_in_countries': []}}),
 Document(page_content='@KanekoaTheGreat 🧐', metadata={'created_at': 'Tue Apr 18 03:35:48 +0000 2023', 'user_info': {'id': 44196397, 'id_str': '44196397', 'name': 'Elon Musk', 'screen_name': 'elonmusk', 'location': 'A Shortfall of Gravitas', 'profile_location': None, 'description': 'nothing', 'url': None, 'entities': {'description': {'urls': []}}, 'protected': False, 'followers_count': 135528327, 'friends_count': 220, 'listed_count': 120478, 'created_at': 'Tue Jun 02 20:12:29 +0000 2009', 'favourites_count': 21285, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 24795, 'lang': None, 'status': {'created_at': 'Tue Apr 18 03:45:50 +0000 2023', 'id': 1648170947541704705, 'id_str': '1648170947541704705', 'text': '@MrAndyNgo @REI One store after another shutting down', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'MrAndyNgo', 'name': 'Andy Ngô 🏳️\u200d🌈', 'id': 2835451658, 'id_str': '2835451658', 'indices': [0, 10]}, {'screen_name': 'REI', 'name': 'REI', 'id': 16583846, 'id_str': '16583846', 'indices': [11, 15]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': 1648134341678051328, 'in_reply_to_status_id_str': '1648134341678051328', 'in_reply_to_user_id': 2835451658, 'in_reply_to_user_id_str': '2835451658', 'in_reply_to_screen_name': 'MrAndyNgo', 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 118, 'favorite_count': 1286, 'favorited': False, 'retweeted': False, 'lang': 'en'}, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': 'C0DEED', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': False, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/44196397/1576183471', 'profile_link_color': '0084B4', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'has_extended_profile': True, 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None, 'translator_type': 'none', 'withheld_in_countries': []}}),
 Document(page_content='@TRHLofficial What’s he talking about and why is it sponsored by Erik’s son?', metadata={'created_at': 'Tue Apr 18 03:32:17 +0000 2023', 'user_info': {'id': 44196397, 'id_str': '44196397', 'name': 'Elon Musk', 'screen_name': 'elonmusk', 'location': 'A Shortfall of Gravitas', 'profile_location': None, 'description': 'nothing', 'url': None, 'entities': {'description': {'urls': []}}, 'protected': False, 'followers_count': 135528327, 'friends_count': 220, 'listed_count': 120478, 'created_at': 'Tue Jun 02 20:12:29 +0000 2009', 'favourites_count': 21285, 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'verified': False, 'statuses_count': 24795, 'lang': None, 'status': {'created_at': 'Tue Apr 18 03:45:50 +0000 2023', 'id': 1648170947541704705, 'id_str': '1648170947541704705', 'text': '@MrAndyNgo @REI One store after another shutting down', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'MrAndyNgo', 'name': 'Andy Ngô 🏳️\u200d🌈', 'id': 2835451658, 'id_str': '2835451658', 'indices': [0, 10]}, {'screen_name': 'REI', 'name': 'REI', 'id': 16583846, 'id_str': '16583846', 'indices': [11, 15]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': 1648134341678051328, 'in_reply_to_status_id_str': '1648134341678051328', 'in_reply_to_user_id': 2835451658, 'in_reply_to_user_id_str': '2835451658', 'in_reply_to_screen_name': 'MrAndyNgo', 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 118, 'favorite_count': 1286, 'favorited': False, 'retweeted': False, 'lang': 'en'}, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': 'C0DEED', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': False, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1590968738358079488/IY9Gx6Ok_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/44196397/1576183471', 'profile_link_color': '0084B4', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'has_extended_profile': True, 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None, 'translator_type': 'none', 'withheld_in_countries': []}})]
```
```

```

### 无监督学习：一种简介

无监督学习是机器学习的一个重要分支，其目标是从数据中发现隐藏的模式和结构，而无需任何标签或人为注释。与监督学习不同，无监督学习的训练数据没有对应的输出标签。这使得无监督学习在处理大规模数据时变得尤为重要，因为标记数据的收集通常是耗时且昂贵的。

![无监督学习](https://example.com/unsupervised_learning.jpg)

在无监督学习中，算法需要自行确定数据中的模式和结构，这通常包括聚类、降维、异常检测等任务。通过这些技术，无监督学习可以帮助我们更好地理解数据，发现数据中的规律，并为进一步的分析和决策提供支持。

无监督学习的应用非常广泛，涵盖了各个领域，如自然语言处理、计算机视觉、生物信息学等。随着数据规模的不断增大和数据复杂性的提高，无监督学习的重要性也日益凸显。因此，深入了解无监督学习的原理和方法对于从事相关领域的研究人员和工程师来说至关重要。