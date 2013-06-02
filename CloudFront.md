# CloudFront
利用CDN讓client更快的下載資料

## 概念
* 在世界各地架設edge server，在client要下載資料時，就去最靠近client的edge server下載。
* 利用DNS resolve時，回應最快速度(TTFB, Time To First Byte)的edge server就去那邊下載。
* 一般用CDN放的資料有html, assets(cs, img, js)，甚至還有streaming

## 工作原理
* origin server是存放CloudFront要服務的原始檔案位置
* edge location是存放origin server檔案的副本
