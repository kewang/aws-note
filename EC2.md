# EC2
強大的虛擬機器

## 概述
依照CPU及MEM的數量分為很多種type，免費版本(free-tier)是T1 Micro(t1.micro)

### AMI
有amazon官方提供的，也有其他linux distro.提供的，但amazon提供的Amazon Linux(based on CentOS)有最佳化過。

### 開機時的檢查(Status Check)
總共會做System Status Check以及Instance Status Check，若任何一種失敗可以觸發Alarm。

### 系統監控(Monitoring)
5分鐘取樣一次，若要改為1分鐘取樣一次就要另外收費，也可以觸發Alarm。
* 平均值、最大值、最小值、總和...等
* CPU使用率、IO讀寫次數、網路吞吐量...等