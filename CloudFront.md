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
預設儲存24hr，如果逾時後，edge location會再去orogin server下載一次

### 使用方式