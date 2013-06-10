# VPC
Virtual Private Cloud
* 每台EC2 instance在啟用時的內部IP都是動態配置的，無法讓用戶自行指定。用了VPC後，便能滿足這樣的需求，每一台啟用的EC2都可以自己在subnet下設定固定IP，當然也能用自訂的DHCP提供。
* VPC可以用VPN + Private Subnet模式來滿足想要私有雲的需求。