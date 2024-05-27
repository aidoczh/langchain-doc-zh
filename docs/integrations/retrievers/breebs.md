# BREEBS（开放知识）

[BREEBS](https://www.breebs.com/) 是一个开放的协作知识平台。

任何人都可以创建 Breeb，即基于存储在 Google Drive 文件夹中的 PDF 的知识胶囊。

Breeb 可以被任何 LLM/聊天机器人使用，以提高其专业知识，减少幻觉并获得信息来源。

在幕后，Breebs 实现了几个检索增强生成（RAG）模型，以在每次迭代中无缝地提供有用的上下文。

## 可用 Breeb 列表

要获取 Breeb 的完整列表，包括它们的关键（breeb_key）和描述：

https://breebs.promptbreeders.com/web/listbreebs。

社区已经创建了数十个 Breeb，并可供免费使用。它们涵盖了广泛的专业知识，从有机化学到神话，以及有关诱惑和去中心化金融的提示。

## 创建新的 Breeb

要生成新的 Breeb，只需在公开共享的 Google Drive 文件夹中编制 PDF 文件，并通过单击“创建 Breeb”按钮在[BREEBS 网站](https://www.breebs.com/)上启动创建过程。您目前可以包括多达 120 个文件，总字符限制为 1500 万。

## 检索器示例

```python
from langchain_community.retrievers import BreebsRetriever
```

```python
breeb_key = "Parivoyage"
retriever = BreebsRetriever(breeb_key)
documents = retriever.invoke(
    "What are some unique, lesser-known spots to explore in Paris?"
)
print(documents)
```

```output
[Document(page_content="de poupées• Ladurée - Madeleine• Ladurée - rue Bonaparte• Flamant• Bonnichon Saint Germain• Dinh Van• Léonor Greyl• Berthillon• Christian Louboutin• Patrick Cox• Baby Dior• FNAC Musique - Bastille• FNAC - Saint Lazare• La guinguette pirate• Park Hyatt• Restaurant de Sers• Hilton Arc de Triomphe• Café Barge• Le Celadon• Le Drouant• La Perouse• Cigale Recamier• Ledoyen• Tanjia• Les Muses• Bistrot du Dôme• Avenue Foch• Fontaine Saint-Michel• Funiculaire de Montmartre• Promotrain - Place Blanche• Grand Palais• Hotel de Rohan• Hotel de Sully• Hotel des Ventes Drouot• Institut de France• Place des Invalides• Jardin d'acclimatation• Jardin des plantes Zoo• Jouffroy (passage)• Quartier de La Défense• La Villette (quartier)• Lac Inferieur du Bois de Boulogne• Les Catacombes de Paris• Place du Louvre• Rue Mazarine• Rue Monsieur le Prince11/12/2023 07:51Guide en pdf Paris à imprimer gratuitement.", metadata={'source': 'https://breebs.promptbreeders.com/breeb?breeb_key=Parivoyage&doc=44d78553-a&page=11', 'score': 1}), Document(page_content="cafés et des restaurants situésdans les rues adjacentes. Il y a également une cafétéria dans le musée, qui propose des collations, desboissons et des repas légers.À voir et visiter autour :Le Muséum d'histoire naturelle de Paris est situé àproximité de plusieurs autres attractions populaires, notamment le Jardin des Plantes, la Grande Mosquéede Paris, la Sorbonne et la Bibliothèque nationale de France.Comment y aller en bus, métro, train :LeMuséum d'histoire naturelle de Paris est facilement accessible en transports en commun. Les stations demétro les plus proches sont la station Censier-Daubenton sur la ligne 7 et la station Jussieu sur les lignes 7et 10. Le musée est également accessible en bus, avec plusieurs lignes desservant la zone, telles que leslignes 24, 57, 61, 63, 67, 89 et 91. En train, la gare la plus proche est la Gare d'Austerlitz, qui est desserviepar plusieurs lignes, notamment les lignes RER C et les trains intercités. Il est également possible de serendre au musée en utilisant les services de taxis ou de VTC.Plus d'informations :+33140795601,6 euros,Ouverture : 10h - 17h, Week end: 10h - 18h ; Fermeture: Mardi(haut de", metadata={'source': 'https://breebs.promptbreeders.com/breeb?breeb_key=Parivoyage&doc=44d78553-a&page=403', 'score': 1}), Document(page_content="Le célèbre Drugstore des Champs Elysées abrite de nombreuses boutiques dans un décor design. V ouspourrez y découvrir un espace beauté, des expositions éphémères, une pharmacie et des espaces réservésaux plaisirs des sens. A noter la façade d'architecture extérieure en verrePlus d'informations :+33144437900, https://www.publicisdrugstore.com/, Visite libre,(haut de page)• Place du Marché Sainte-CatherinePlace du Marché Sainte-Catherine, Paris, 75008, FR11/12/2023 07:51Guide en pdf Paris à imprimer gratuitement.\nPage 200 sur 545https://www.cityzeum.com/imprimer-pdf/parisUne place hors de l'agitation de la capitale, où vous découvrirez des petits restaurants au charme certaindans un cadre fort agréable. Terrasses au rendez-vous l'été! Un bar à magie pour couronner le toutPlus d'informations :15-30 euros,(haut de page)• Rue de Lappe, ParisRue de Lappe, Paris, FR", metadata={'source': 'https://breebs.promptbreeders.com/breeb?breeb_key=Parivoyage&doc=44d78553-a&page=198', 'score': 1}), Document(page_content="des visiteurs pour la nature etles attractions du parc. Les visiteurs peuvent prévoir de passer entre 1 à 2 heures pour visiter le parcL'accès au parc Montsouris est gratuit pour tous les visiteurs. Aucune réservation n'est nécessaire pourvisiter le parc. Cependant, pour les visites guidées, il est conseillé de réserver à l'avance pour garantir uneplace. Les tarifs pour les visites guidées peuvent varier en fonction de l'organisme proposant la visite.Ensomme, le parc Montsouris est un endroit magniﬁque pour se détendre et proﬁter de la nature en pleincœur de Paris. Avec ses attractions pittoresques, son paysage verdoyant et ses visites guidées, c'est unendroit idéal pour une sortie en famille ou entre amis.Plus d'informations :https://www.parisinfo.com/musee-monument-paris/71218/Parc-Montsouris,Gratuit,Ouverture : 8h/9h - 17h30/21h30(haut de page)• Parc des Buttes Chaumont", metadata={'source': 'https://breebs.promptbreeders.com/breeb?breeb_key=Parivoyage&doc=44d78553-a&page=291', 'score': 1})]
```