# EC2
強大的虛擬機器

## 概述
依照CPU及MEM的數量分為很多種type，免費版本(free-tier)是T1 Micro(t1.micro)

### AMI
有amazon官方提供的，也有其他linux distro.提供的，但amazon提供的Amazon Linux(based on CentOS)有最佳化過。

### 開機檢查
總共會做兩種檢查，分別是System Status Check以及Instance Status Check。若檢查有誤，可以利用SNS通知管理者。
* System Status Check：檢查AWS在這個instance上面相關的功能都正常運作
* Instance Status Check：檢查這個instance的軟體及網路設計是否正常運作
