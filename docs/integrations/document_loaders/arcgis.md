# ArcGIS
本文介绍了如何使用 `langchain_community.document_loaders.ArcGISLoader` 类。
您需要安装 ArcGIS API for Python `arcgis`，以及可选的 `bs4.BeautifulSoup`。
您可以使用 `arcgis.gis.GIS` 对象进行身份验证数据加载，或者留空以访问公共数据。
```python
from langchain_community.document_loaders import ArcGISLoader
URL = "https://maps1.vcgov.org/arcgis/rest/services/Beaches/MapServer/7"
loader = ArcGISLoader(URL)
docs = loader.load()
```
让我们来测量加载器的延迟。
```python
%%time
docs = loader.load()
```
```output
CPU times: user 2.37 ms, sys: 5.83 ms, total: 8.19 ms
Wall time: 1.05 s
```
```python
docs[0].metadata
```
```output
{'accessed': '2023-09-13T19:58:32.546576+00:00Z',
 'name': 'Beach Ramps',
 'url': 'https://maps1.vcgov.org/arcgis/rest/services/Beaches/MapServer/7',
 'layer_description': '(Not Provided)',
 'item_description': '(Not Provided)',
 'layer_properties': {
   "currentVersion": 10.81,
   "id": 7,
   "name": "Beach Ramps",
   "type": "Feature Layer",
   "description": "",
   "geometryType": "esriGeometryPoint",
   "sourceSpatialReference": {
     "wkid": 2881,
     "latestWkid": 2881
   },
   "copyrightText": "",
   "parentLayer": null,
   "subLayers": [],
   "minScale": 750000,
   "maxScale": 0,
   "drawingInfo": {
     "renderer": {
       "type": "simple",
       "symbol": {
         "type": "esriPMS",
         "url": "9bb2e5ca499bb68aa3ee0d4e1ecc3849",
         "imageData": "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAAXNSR0IB2cksfwAAAAlwSFlzAAAOxAAADsQBlSsOGwAAAJJJREFUOI3NkDEKg0AQRZ9kkSnSGBshR7DJqdJYeg7BMpcS0uQWQsqoCLExkcUJzGqT38zw2fcY1rEzbp7vjXz0EXC7gBxs1ABcG/8CYkCcDqwyLqsV+RlV0I/w7PzuJBArr1VB20H58Ls6h+xoFITkTwWpQJX7XSIBAnFwVj7MLAjJV/AC6G3QoAmK+74Lom04THTBEp/HCSc6AAAAAElFTkSuQmCC",
         "contentType": "image/png",
         "width": 12,
         "height": 12,
         "angle": 0,
         "xoffset": 0,
         "yoffset": 0
       },
       "label": "",
       "description": ""
     },
     "transparency": 0,
     "labelingInfo": null
   },
   "defaultVisibility": true,
   "extent": {
     "xmin": -81.09480168806815,
     "ymin": 28.858349245353473,
     "xmax": -80.77512908572814,
     "ymax": 29.41078388840041,
     "spatialReference": {
       "wkid": 4326,
       "latestWkid": 4326
     }
   },
   "hasAttachments": false,
   "htmlPopupType": "esriServerHTMLPopupTypeNone",
   "displayField": "AccessName",
   "typeIdField": null,
   "subtypeFieldName": null,
   "subtypeField": null,
   "defaultSubtypeCode": null,
   "fields": [
     {
       "name": "OBJECTID",
       "type": "esriFieldTypeOID",
       "alias": "OBJECTID",
       "domain": null
     },
     {
       "name": "Shape",
       "type": "esriFieldTypeGeometry",
       "alias": "Shape",
       "domain": null
     },
     {
       "name": "AccessName",
       "type": "esriFieldTypeString",
       "alias": "AccessName",
       "length": 40,
       "domain": null
     },
     {
       "name": "AccessID",
       "type": "esriFieldTypeString",
       "alias": "AccessID",
       "length": 50,
       "domain": null
     },
     {
       "name": "AccessType",
       "type": "esriFieldTypeString",
       "alias": "AccessType",
       "length": 25,
       "domain": null
     },
     {
       "name": "GeneralLoc",
       "type": "esriFieldTypeString",
       "alias": "GeneralLoc",
       "length": 100,
       "domain": null
     },
     {
       "name": "MilePost",
       "type": "esriFieldTypeDouble",
       "alias": "MilePost",
       "domain": null
     },
     {
       "name": "City",
       "type": "esriFieldTypeString",
       "alias": "City",
       "length": 50,
       "domain": null
     },
     {
       "name": "AccessStatus",
       "type": "esriFieldTypeString",
       "alias": "AccessStatus",
       "length": 50,
       "domain": null
     },
     {
       "name": "Entry_Date_Time",
       "type": "esriFieldTypeDate",
       "alias": "Entry_Date_Time",
       "length": 8,
       "domain": null
     },
     {
       "name": "DrivingZone",
       "type": "esriFieldTypeString",
       "alias": "DrivingZone",
       "length": 50,
       "domain": null
     }
   ],
   "geometryField": {
     "name": "Shape",
     "type": "esriFieldTypeGeometry",
     "alias": "Shape"
   },
   "indexes": null,
   "subtypes": [],
   "relationships": [],
   "canModifyLayer": true,
   "canScaleSymbols": false,
   "hasLabels": false,
   "capabilities": "Map,Query,Data",
   "maxRecordCount": 1000,
   "supportsStatistics": true,
   "supportsAdvancedQueries": true,
   "supportedQueryFormats": "JSON, geoJSON",
   "isDataVersioned": false,
   "ownershipBasedAccessControlForFeatures": {
     "allowOthersToQuery": true
   },
   "useStandardizedQueries": true,
   "advancedQueryCapabilities": {
     "useStandardizedQueries": true,
     "supportsStatistics": true,
     "supportsHavingClause": true,
     "supportsCountDistinct": true,
     "supportsOrderBy": true,
     "supportsDistinct": true,
     "supportsPagination": true,
     "supportsTrueCurve": true,
     "supportsReturningQueryExtent": true,
     "supportsQueryWithDistance": true,
     "supportsSqlExpression": true
   },
   "supportsDatumTransformation": true,
   "dateFieldsTimeReference": null,
   "supportsCoordinatesQuantization": true
 }}
```
### 获取几何信息
如果您想要获取要素的几何信息，可以使用 `return_geometry` 关键字。
每个文档的几何信息将存储在其元数据字典中。
```python
loader_geom = ArcGISLoader(URL, return_geometry=True)
```
```python
%%time
docs = loader_geom.load()
```
```output
CPU times: user 9.6 ms, sys: 5.84 ms, total: 15.4 ms
Wall time: 1.06 s
```
```python
docs[0].metadata["geometry"]
```
```output
{'x': -81.01508803280349,
 'y': 29.24246579525828,
 'spatialReference': {'wkid': 4326, 'latestWkid': 4326}}
```
```python
for doc in docs:
    print(doc.page_content)
```
```output
{"OBJECTID": 4, "AccessName": "UNIVERSITY BLVD", "AccessID": "DB-048", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "900 BLK N ATLANTIC AV", "MilePost": 13.74, "City": "DAYTONA BEACH", "AccessStatus": "OPEN", "Entry_Date_Time": 1694597536000, "DrivingZone": "BOTH"}
{"OBJECTID": 18, "AccessName": "BEACHWAY AV", "AccessID": "NS-106", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "1400 N ATLANTIC AV", "MilePost": 1.57, "City": "NEW SMYRNA BEACH", "AccessStatus": "OPEN", "Entry_Date_Time": 1694600478000, "DrivingZone": "YES"}
{"OBJECTID": 24, "AccessName": "27TH AV", "AccessID": "NS-141", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "3600 BLK S ATLANTIC AV", "MilePost": 4.83, "City": "NEW SMYRNA BEACH", "AccessStatus": "CLOSED FOR HIGH TIDE", "Entry_Date_Time": 1694619363000, "DrivingZone": "BOTH"}
{"OBJECTID": 26, "AccessName": "SEABREEZE BLVD", "AccessID": "DB-051", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "500 BLK N ATLANTIC AV", "MilePost": 14.24, "City": "DAYTONA BEACH", "AccessStatus": "OPEN", "Entry_Date_Time": 1694597536000, "DrivingZone": "BOTH"}
{"OBJECTID": 30, "AccessName": "INTERNATIONAL SPEEDWAY BLVD", "AccessID": "DB-059", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "300 BLK S ATLANTIC AV", "MilePost": 15.27, "City": "DAYTONA BEACH", "AccessStatus": "OPEN", "Entry_Date_Time": 1694598638000, "DrivingZone": "BOTH"}
{"OBJECTID": 33, "AccessName": "GRANADA BLVD", "AccessID": "OB-030", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "20 BLK OCEAN SHORE BLVD", "MilePost": 10.02, "City": "ORMOND BEACH", "AccessStatus": "4X4 ONLY", "Entry_Date_Time": 1694595424000, "DrivingZone": "BOTH"}
{"OBJECTID": 39, "AccessName": "BEACH ST", "AccessID": "PI-097", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "4890 BLK S ATLANTIC AV", "MilePost": 25.85, "City": "PONCE INLET", "AccessStatus": "4X4 ONLY", "Entry_Date_Time": 1694596294000, "DrivingZone": "BOTH"}
{"OBJECTID": 44, "AccessName": "SILVER BEACH AV", "AccessID": "DB-064", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "1000 BLK S ATLANTIC AV", "MilePost": 15.98, "City": "DAYTONA BEACH", "AccessStatus": "OPEN", "Entry_Date_Time": 1694598638000, "DrivingZone": "YES"}
{"OBJECTID": 45, "AccessName": "BOTEFUHR AV", "AccessID": "DBS-067", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "1900 BLK S ATLANTIC AV", "MilePost": 16.68, "City": "DAYTONA BEACH SHORES", "AccessStatus": "OPEN", "Entry_Date_Time": 1694598638000, "DrivingZone": "YES"}
{"OBJECTID": 46, "AccessName": "MINERVA RD", "AccessID": "DBS-069", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "2300 BLK S ATLANTIC AV", "MilePost": 17.52, "City": "DAYTONA BEACH SHORES", "AccessStatus": "OPEN", "Entry_Date_Time": 1694598638000, "DrivingZone": "YES"}
{"OBJECTID": 56, "AccessName": "3RD AV", "AccessID": "NS-118", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "1200 BLK HILL ST", "MilePost": 3.25, "City": "NEW SMYRNA BEACH", "AccessStatus": "OPEN", "Entry_Date_Time": 1694600478000, "DrivingZone": "YES"}
{"OBJECTID": 65, "AccessName": "MILSAP RD", "AccessID": "OB-037", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "700 BLK S ATLANTIC AV", "MilePost": 11.52, "City": "ORMOND BEACH", "AccessStatus": "4X4 ONLY", "Entry_Date_Time": 1694595749000, "DrivingZone": "YES"}
{"OBJECTID": 72, "AccessName": "ROCKEFELLER DR", "AccessID": "OB-034", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "400 BLK S ATLANTIC AV", "MilePost": 10.9, "City": "ORMOND BEACH", "AccessStatus": "CLOSED - SEASONAL", "Entry_Date_Time": 1694591351000, "DrivingZone": "YES"}
{"OBJECTID": 74, "AccessName": "DUNLAWTON BLVD", "AccessID": "DBS-078", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "3400 BLK S ATLANTIC AV", "MilePost": 20.61, "City": "DAYTONA BEACH SHORES", "AccessStatus": "OPEN", "Entry_Date_Time": 1694601124000, "DrivingZone": "YES"}
{"OBJECTID": 77, "AccessName": "EMILIA AV", "AccessID": "DBS-082", "AccessType": "OPEN VEHICLE RAMP", "GeneralLoc": "3790 BLK S ATLANTIC AV", "MilePost": 21.38, "City": "DAYTONA BEACH SHORES", "AccessStatus": "OPEN", "Entry_Date_Time": 1694601124000, "DrivingZone": "BOTH"}
```
| OBJECTID | AccessName   | AccessID | AccessType           | GeneralLoc             | MilePost | City              | AccessStatus     | Entry_Date_Time | DrivingZone |
|----------|--------------|----------|----------------------|------------------------|----------|-------------------|------------------|-----------------|-------------|
| 84       | VAN AV       | DBS-075  | OPEN VEHICLE RAMP    | 3100 BLK S ATLANTIC AV | 19.6     | DAYTONA BEACH SHORES | OPEN             | 1694601124000   | YES         |
| 104      | HARVARD DR   | OB-038   | OPEN VEHICLE RAMP    | 900 BLK S ATLANTIC AV  | 11.72    | ORMOND BEACH       | OPEN             | 1694597536000   | YES         |
| 106      | WILLIAMS AV  | DB-042   | OPEN VEHICLE RAMP    | 2200 BLK N ATLANTIC AV | 12.5     | DAYTONA BEACH      | OPEN             | 1694597536000   | YES         |
| 109      | HARTFORD AV  | DB-043   | OPEN VEHICLE RAMP    | 1890 BLK N ATLANTIC AV | 12.76    | DAYTONA BEACH      | CLOSED - SEASONAL | 1694591351000   | YES         |
| 138      | CRAWFORD RD  | NS-108   | OPEN VEHICLE RAMP - PASS | 800 BLK N ATLANTIC AV | 2.19     | NEW SMYRNA BEACH  | OPEN             | 1694600478000   | YES         |
| 140      | FLAGLER AV   | NS-110   | OPEN VEHICLE RAMP    | 500 BLK FLAGLER AV     | 2.57     | NEW SMYRNA BEACH  | OPEN             | 1694600478000   | YES         |
| 144      | CARDINAL DR  | OB-036   | OPEN VEHICLE RAMP    | 600 BLK S ATLANTIC AV  | 11.27    | ORMOND BEACH       | 4X4 ONLY         | 1694595749000   | YES         |
| 174      | EL PORTAL ST | DBS-076  | OPEN VEHICLE RAMP    | 3200 BLK S ATLANTIC AV | 20.04    | DAYTONA BEACH SHORES | OPEN             | 1694601124000   | YES         |
以上是一些海滩车辆通行道路的信息。每个通行道路都有一个唯一的OBJECTID，包括通行道路的名称（AccessName）、通行道路的ID（AccessID）、通行道路的类型（AccessType）、通行道路的位置（GeneralLoc）、里程标（MilePost）、所在城市（City）、通行状态（AccessStatus）、录入日期时间（Entry_Date_Time）和驾驶区域（DrivingZone）。
这些通行道路主要用于海滩车辆通行，包括一些开放的车辆坡道和一些季节性关闭的车辆坡道。例如，HARTFORD AV通行道路是季节性关闭的。此外，有一些通行道路只允许四轮驱动车辆通行。