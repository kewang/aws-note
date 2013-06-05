# EC2
強大的虛擬機器，依照CPU及MEM的等級分為很多種type，免費版本(free-tier)是T1 Micro(t1.micro)

## AMI(Amazon Machine Image)
新建instance時必須要掛載AMI，類似安裝檔的類似。有amazon官方提供的，也有其他linux distro.提供的，但amazon提供的Amazon Linux(based on CentOS)有最佳化過。

### Create Image
建置完系統環境時，應該要做一份Image，這樣子之後要新增機器時，可以直接用這份Image快速建置。

## 開機時的檢查(Status Check)
總共會做System Status Check以及Instance Status Check，若任何一種失敗可以觸發Alarm。

## 系統監控(Monitoring)
5分鐘取樣一次，若要改為1分鐘取樣一次就要另外收費(detailed monitoring)，也可以觸發Alarm。
* 平均值、最大值、最小值、總和...等
* CPU使用率、IO讀寫次數、網路吞吐量...等

## Tag
EC2的metadata，因為EC2沒有階層關係，可以利用tag模擬階層。例如以下方式：
* owner: kewang, joseph, jason, winnie...
* environment: production, test, development...
* server-type: web, ftp, mail...

## EBS(elastic block store)
類似SAN(Storage area network)的概念，可以隨時mount到EC2上面，並且可以隨時做snapshot，方便備份。Mount的指令如下，假設目前位置為/dev/xvdf：

1.     sudo mkfs.ext4 /dev/xvdf
2.     sudo mkdir -m 000 /vol
3.     echo "/dev/xvdf /vol auto noatime 0 0" | sudo tee -a /etc/fstab
4.     sudo mount /vol