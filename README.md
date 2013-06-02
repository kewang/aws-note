# EC2

# S3
* Simple Storage Service
* 儲存使用者上傳的照片、每天爆增的log或備份檔...等
* 內部儲存架構為扁平式，例如photo/image1.jpg和photo/image2.jpg可以儲存不同的cluster，達成**高延展性、高可用性、高耐久性**

## 簡介
* 使用方式：REST、SOAP
* 每個儲存的物件會在不同機房複製多份
* 可用性99.99%

## 收費方式
* 前1TB：$0.150
* 第1~50TB：$0.135
* etc...

## bucket架構
S3最上層分類，一個帳號可以建立100個bucket，通常是一個service開一個bucket
* URL naming 1：http://s3.amazonaws.com/<bucket-name>/<object-key>
* URL naming 2 **建議使用(虛擬主機)** ：http://<bucket-name>.s3.amazonaws.com/<object-key>

### 使用虛擬主機
* 可以用自己的domain
* 可以使用root directory
* no-redirect problem, reduce latency
* 其他AWS會使用S3，如CloudFront、EMR...etc.

## object實務
存放在bucket的資料，可以是任何檔案，單個object大小為5TB

### 可以將S3視為CDN的方式使用
* http://mysite.s3.amazonaws.com/static/img/header/top.gif
* http://mysite.s3.amazonaws.com/static/js/ajax.js
* http://mysite.s3.amazonaws.com/static/css/home/profile.css

### 如果是放上傳的照片，可以如下方式使用
unique id要夠長才不容易重複
* http://mysite.s3.amazonaws.com/userphoto/A3fdA53fdytHUf.jpg

### 好處
簡單快速產生object的URL，不用另外做URLEncode

## region架構
* 為global specific，意指bucket為全S3，所以name要unique
* 放在跟EC2同一個region，因為EC2跟S3在同地區網路傳輸不收費

## 儲存等級
* Standard：一般都使用這種，預設也是Standard，耐久性(durability)為99.999999999%，$0.15(每GB)
* RRS(Reduced Redundancy Storage)：低備份儲存，適合儲存非原始來源的檔案，例如圖檔縮圖。如果物件消失，S3會回應HTTP 405，耐久性為99.99%，$0.10(每GB)

## 操作方式
* HTTP PUT：上傳object，會回傳versionId
* HTTP GET 1：不給versionId時，讀取最新一筆object
* HTTP GET 2：給versionId時，讀取特定versionId的object
* HTTP DELETE 1：不給versionId時，會將object加上delete marker。造成GET 1會得到404 file not found
* HTTP DELETE 2：給versionId時，會永久刪除object

## HTTP Header實務
先行讀取header，判斷是否要讀取內容、下載、快取...etc.
* **Content-Length**：object length
* **Content-Type**：object type, using mimetype format
* **Cache-Control**：HTTP Cache，要明確寫明cache時間；若不能cache也要註明no-cache。在網站速度最佳化及ColudFront配合時非常重要。
* Content-Disposition：按連結時自動另存新檔
* Content-Encoding：壓縮檔要加
* Last-Modified & Content-MD5(只用其一)：了解S3的object與local是否有差異，快速比對版本
* x-amz-meta-*：自定的metadata，例如table及table-id

## ACL
一般分為兩種：private及public-read，其餘較少使用

## 分割上傳
傳大檔較常用到。

## 範圍讀取
* 可以只讀取object的一部分，例如在HTTP Header放入Range: bytes=0-1023，只讀取前面1KB
* 可以用來讀取EXIF...etc.

# SimpleDB