# CloudFront
利用CDN讓client更快的下載資料

## 概念
* 在世界各地架設edge server，在client要下載資料時，就去最靠近client的edge server下載。
* 利用DNS resolve時，回應最快速度(TTFB, Time To First Byte)的edge server就去那邊下載。
* 一般用CDN放的資料有html, assets(cs, img, js)，甚至還有streaming

## 工作原理
* origin server是存放CloudFront要服務的原始檔案位置
* edge location是存放origin server檔案的副本
* client下載資料時會先去edge location看看有沒有這個檔案，若沒有的話edge location會去origin server下載回來，所以第一次速度會比較慢。

## 收費

### 輸出資料量
* 前10TB：$0.201(每GB)
* 10TB~50TB：$0.148(每GB)

### 請求次數 (request)
* 每10000次http request：$0.0095
* 每10000次http**s** request：$0.0130
* 前1000次主動清除：免費
* 超過1000次主動清除：0.005

## 檔案逾時(expiration)
預設儲存24hr，如果逾時後，edge location會再去origin server下載一次

### 設定方式
* 使用HTTP Header的「Cache-Control」、「Expire」、「Pragma」來判斷，**所以S3的bucket-object規劃很重要**。
* 最短只能設定1hr，最大無上限
* **逾時設定1~24hr時，要重新思考是否要使用CDN，或是改變系統架構**

### 實務作法
* 一般逾時是設定1yr以上
* 若在逾時前origin server就已經變更檔案內容，可以利用versioning URL的方式騙過browser，使browser重新下載檔案

#### Versioning URL
把版本資訊加在URL裡面，例如開發時的檔名叫做js/jquery.js，但是在deploy到origin server時，把檔案存成js/v1/jquery.js或js/jquery_v1.js

1. 把版本號v1變成metadata存在系統設定裡
2. 輸出URL到網頁上時，把版號加在URL內，如js/v1/jquery.js
3. 因為URL改變，所以client一定會下載新的檔案

## edge location到origin server抓檔案的時機
* 檔案沒有request過
* 檔案逾時
* 被edge location移除(eviction)

### 檔案移除(eviction)
若某個檔案還沒逾時，但是很少request，edge location為了調節資源，所以將少request的檔案移除。