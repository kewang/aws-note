# EC2
強大的虛擬機器，依照CPU及MEM的等級分為很多種type，免費版本(free-tier)是T1 Micro(t1.micro)

## Supported Platform
每個區域都有不同的支援平台，所有的區域都有EC2-VPC。2013/3之前的帳號，應該還會保留EC2-Classic，但2013/3之後的帳號，應該只剩EC2-VPC可以使用。[Amazon EC2 Update - Virtual Private Clouds for Everyone!](http://aws.typepad.com/aws/2013/03/amazon-ec2-update-virtual-private-clouds-for-everyone.html)

### EC2-Classic
* 在launch的時候，選擇的是AZ(availability zone)，跟subnet無關。
* 所以aws會自己配置IP給instance，一般是10.0.0.0/8。
* 另外沒辦法自己設計網路環境，所以新的帳號都沒有EC2-Classic選項了。

### EC2-VPC
* VPC為Virtual Private Cloud的縮寫，可以直接在VPC內建置自己的網路環境。
* AWS為每個僅有VPC的region都做了一個預設VPC(default VPC)，裡面的網路環境是172.31.0.0/16。
* 在launch的時候選擇的是subnet，就可以選擇你要使用哪一個subnet來launch instance。
* VPC詳細資料可以看另一篇整理的VPC文件。

## AMI(Amazon Machine Image)
新建instance時必須要掛載AMI，類似系統安裝檔。有amazon官方提供的，也有其他linux distro.提供的，但amazon提供的Amazon Linux(based on CentOS)有最佳化過。

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
類似SAN(Storage area network)的概念，可以隨時mount儲存區到EC2上面，並且可以隨時做snapshot，方便備份。Mount的指令如下，假設目前位置為/dev/xvdf：

1. sudo mkfs.ext4 /dev/xvdf
2. sudo mkdir -m 000 /vol
3. echo "/dev/xvdf /vol auto noatime 0 0" | sudo tee -a /etc/fstab
4. sudo mount /vol

## Security Groups
防火牆的規則，可以設定inbound(連入)及outbound(連出)。

## EIP (Elastic IP)
因為每次Instance重開機後，Public IP都會變更，所以如果要固定IP時，就要使用EIP與instance綁住。

## Load Balancer
一般而言，網路服務架設EC2都會安裝許多台web在上面，所以如果要分散Loading時，就要安裝Load Balancer。

### 運作原理(Health Check)
LB會ping已經設定好的instance，若ping有通過，則LB就會導到這一台，ping的方式如下
* Protocol：可以用HTTP, HTTPS, TCP, SSL，若服務的是web server，一般是使用HTTP
* Timeout：預設為5秒
* Interval：預設為30秒，每30秒會ping一次。
* Unhealthy Threshold：ping連續失敗的次數之後，就視為沒通過(不健康)，預設為2次。
* Healthy Threshold：ping連續成功的次數之後，就視為通過(健康)，預設為10次。

### 保持session狀態(Sticky Session)
client在第一次連到LB時，會取得一個cookie，這個cookie的作用是把下一次client又連到LB時，讓client連到同一台instance，如此instance就不用重新建立connection，節省資源。分為兩種sticky session：

* duration-based：設定expiration time，在時間之內client都是連到同一台instance
* application-controlled：讓ap server自訂cookie name

## SSH Login
ssh -i keypair.pem xxxx@a.b.c.d

### 不使用憑證連線的方式
1. 修改/etc/ssh/sshd_config：將PasswordAuthentication改為yes
2. 重啟sshd：sudo service sshd restart
3. 記得修改登入帳號的密碼：sudo passwd user-name
4. 回到本機端，使用ssh登入：ssh xxx@a.b.c.d

## EC2 API Tools
因為不是所有的EC2功能(如：Auto Scaling)都有網頁介面，所以必須要利用Command Line操作EC2。操作Tools之前要設定憑證及環境變數。

### 設定憑證
因為一般操作EC2或其他AWS服務的管理人員可能不一樣，所以要在IAM上面設定憑證來控制權限，控制哪些服務可以給哪些管理人員使用。

1. openssl genrsa -out pk-amazon.pem 2048
2. openssl req -new -x509 -key pk-amazon.pem -out **cert-amazon.pem** -days 3650
3. openssl pkcs8 -topk8 -in pk-amazon.pem -nocrypt > pk-temp.pem
4. mv pk-temp.pem **pk-amazon.pem**

以上指令完成之後所產生的cert-amazon及pk-amazon就是我們要的憑證檔，最後到IAM裡面的User設定值有一個Manage Signing Certificates，複製cert內容就完成憑證設定。
* cert-amazon.pem：為EC2的cert
* pk-amazon.pem：為EC2的private key

### 設定環境變數(外部)
在shell設定檔裡面(.zshrc, .bashrc...等)，設定以下的環境變數就可以了。

* export EC2_PRIVATE_KEY="/home/kewang/aws/key/pk-amazon.pem"
* export EC2_CERT="/home/kewang/aws/key/cert-amazon.pem"
* export EC2_URL="http://ec2.ap-northeast-1.amazonaws.com"
* export EC2_REGION="ap-northeast-1"

### 設定環境變數(內部)
在IAM新增User時，有下載一個credentials.csv，裡面包含了AWS_ACCESS_KEY及AWS_SECRET_KEY，這個部分必須設定在環境變數裡。

* export AWS_ACCESS_KEY="AKIXXXAQEO5AMXXXX3PQ"
* export AWS_SECRET_KEY="bZuXXXXVcTwu4QMVo8XXXX5ytVy4yN5OKXXXXeh1"

### 測試是否成功
執行ec2-describe-instances，若有出現多個instances就表示設定完成。

## 讀取metadata及userdata
可以在instance內利用這些資料做到自動初始化，取得方式如下：
* wget -qO- 169.254.169.254/latest/user-data
* wget -qO- 169.254.169.254/latest/meta-data

### metadata
跟instance相關的資料，例如instance-id、local-ipv4、public-ipv4...等。

### userdata
管理者提供給instance的資料，類似instance的tag。
