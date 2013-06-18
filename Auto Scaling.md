# Auto Scaling
Auto Scaling(以下簡稱AS)能讓你動態或定期的調整運算資源。因為目前AWS還沒有介面可供設定，所以必須使用指令來操作Auto Scaling

* 維護固定數量的運算資源。
* 動態調整：依照不同條件可以做不同的修正。
* 定期調整：比如說每天下午八點多開一台instance。

## 基本元件

### Launch Configuration
定義AS在新增一台instance時，該instance的基本參數，以下列出較常用的兩個參數：
* --image-id：AMI的id，實務上一般是用自己製作的AMI
* --instance-type：instance的等級，視情況而定。

#### 新增Launch Configuration
<pre>as-create-launch-config {NAME} 基本參數</pre>

### Auto Scaling Group
定義AS在執行時，本身可以調整的參數，以下列出較常用的參數：
* --availability-zones：要執行的AZ在何處
* --launch-configuration：開啟instance時的基本參數
* --max-size：最多開幾台instance
* --min-size：最少開幾台instance
* --desired-capacity：AS預設執行時的instance數量，若沒有設定就以--min-size為主

#### 新增Auto Scaling Group
<pre>as-create-auto-scaling-group {NAME} 基本參數</pre>

## Scenario

### 維護固定數量

### 動態調整

### 定期調整
