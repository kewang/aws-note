# VPC
Virtual Private Cloud

## 概念
你可以完全自訂這個網路裡面的subnet、ip range、routing table、gateway、security...等，當然也可以加EC2的instance進去。

### Subnet
是一段IP位址的範圍，aws的resource都可以擺在subnet裡面。一般會分為兩種：
* Public Subnet：可以連上Internet的叫做Public Subnet
* Private Subnet：沒辦法連上Internet的叫做Private Subnet

### default VPC
如果你的region只有EC2-VPC而沒有EC2-Classic的話，在新增instance的時候就會有一個預設的VPC可以使用，並且有以下特性：
* **每一個instance都有一個public IP**
* subnet預設為172.31.0.0/16，在新建instance的時候會自己切兩個sub-subnet，分別為172.31.0.0/20及172.31.16.0/20。

### nondefault VPC
如果想要自建私有雲，或是想客製化網路環境的話，可以自建VPC實作，基本上有以下四種情境。

* Public Subnet – 用來提供公眾型的應用服務幾乎都應該適用，可大大增加安全性，也能自定內部IP位置
* Public + Private Subnet – 有些用戶會想把DB Server等重要服務移至不能直接存取的區域，便可選此模式
* VPN + Public + Private Subnet – 就是上一個模式中加上一個Hardware VPN設備來與企業的VPN設備串接
* VPN + Private Subnet – 把AWS完全用來當做私有雲，一樣用Hardware VPN設備來與企業的VPN設備串接