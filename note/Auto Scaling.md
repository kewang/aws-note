# Auto Scaling
Auto Scaling(以下簡稱AS)能讓你動態或定期的調整運算資源。因為目前AWS還沒有介面可供設定，所以必須使用指令來操作Auto Scaling

* 維護固定數量的運算資源。
* 動態調整：依照不同條件可以做不同的修正。
* 定期調整：比如說每天下午八點多開一台instance。

## Launch Configuration
定義AS在新增一台instance時，該instance的基本參數，以下列出較常用的兩個參數：
* --image-id：AMI的id，實務上一般是用自己製作的AMI
* --instance-type：instance的等級，視情況而定。

### 新增Launch Configuration
<pre>as-create-launch-config {NAME} 基本參數</pre>

## Auto Scaling Group
定義AS在執行時，本身可以調整的參數，以下列出較常用的參數：
* --availability-zones：要執行的AZ在何處
* --launch-configuration：開啟instance時的基本參數，為新增launch configuration時的name
* --max-size：最多開幾台instance
* --min-size：最少開幾台instance
* --desired-capacity：AS預設執行時的instance數量，若沒有設定就以--min-size為主

### 新增Auto Scaling Group
<pre>as-create-auto-scaling-group {NAME} 基本參數</pre>

## Scaling Policy
定義AS在動態調整instance時的方式，通常用在ELB上面，以下列出較常用的參數：
* --auto-scaling-group：為新增auto scaling group時的name
* --adjustment：一次要開啟instance的值，要搭配--type及--auto-scaling-group才能算出來instance操作的台數。計算出來的台範圍必須介於auto scaling group的min size及max size之間。
* --type：分為絕對數值、相對數值、百分比三種

### --type
* ExactCapacity：絕對數值，若--adjustment為10，就是無論現在台數為多少，一律調整到10台instance。
* ChangeInCapacity：相對數值，若--adjustment為5，就是現在的台數加5台；若--adjustment為-3，就是現在的台數減3台。
* PercentChangeInCapacity：百分比，若--adjustment為20，並假設auto scaling group的desired capacity為5，就是現在的台數加5*20%=1台。如果計算出來的值在0~1之間為1；大於1為無條件捨去。

### 新增Auto Scaling Policy
<pre>as-put-scaling-policy {NAME} 基本參數</pre>

## Health Check
AS開啟instance之後，會針對這些instance做監控，並分為healthy及unhealthy兩種狀態。若instance狀態變為unhealthy時，則會，但也會因為health check type分為EC2及ELB而有不同的評斷方式，分別介紹如下。

### EC2
預設的health check type，在新增auto scaling group未指定health check type時，就會使用這種類型。
* Healthy：若開啟的auto scaling instance為running，則為healthy。
* Unhealthy：若開啟的auto scaling instance不為running，則為unhealthy。

### ELB
若AS與ELB連結在一起時，則health check type就會 **多一種選擇** ，可以指定由EC2的instance status或ELB的health check來評斷。
* Healthy：若ELB裡面的instance為InService，則為healthy。
* Unhealthy：若ELB裡面的instance為OutOfService，則為unhealthy。

## Scenario

### 維護固定數量
新增名稱為my-test-lc的launch configuration，在開啟instance時使用的AMI為ami-0078da69，而且開啟的instance等級為m1.small。
<pre>as-create-launch-config my-test-lc --image-id ami-0078da69 --instance-type
m1.small</pre>
新增名稱為my-test-asg的auto scaling group，使用名稱為my-test-lc的launch configuration，並套用在us-east-1a的AZ，執行時最少開1台，最多開10台，初始化時先開啟1台。
<pre>as-create-auto-scaling-group my-test-asg --launch-configuration my-test-lc --
availability-zones us-east-1a --min-size 1 --max-size 10 --desired-capacity 1</pre>

### 動態調整
![AS Workflow](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/images/AS-WorkFlow.png)

### 定期調整
