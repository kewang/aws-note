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
* -–termination-policies：定義關閉instance時的方式，稍後詳述。

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
AS開啟instance之後，會針對這些instance做監控，並分為healthy及unhealthy兩種狀態。若instance狀態變為unhealthy時，則會根據launch configuration，再launch一台新的instance，並且將unhealthy的instance關閉。但也會因為health check type分為EC2及ELB而有不同的評斷方式，分別介紹如下。

### EC2
預設的health check type，在新增auto scaling group未指定health check type時，就會使用這種類型。
* Healthy：若開啟的auto scaling instance為running，則為healthy。
* Unhealthy：若開啟的auto scaling instance不為running，則為unhealthy。

### ELB
若AS與ELB連結在一起時，則health check type就會 **多一種選擇** ，可以指定由EC2的instance status或ELB的health check來評斷。
* Healthy：若ELB裡面的instance為InService，則為healthy。
* Unhealthy：若ELB裡面的instance為OutOfService，則為unhealthy。

## Termination Policy
當scaling policy要關閉instance(例：原本5台，要關閉成3台；原本5台，要減少20%的instance...等)，或是health check監控到有unhealthy的instance時，就會觸發termination policy。policy總共分為下面5種：

* OldestInstance：AS會把最先launch的instance關閉。
* NewestInstance：AS會把最後launch的instance關閉。
* OldestLaunchConfiguration：因為同一個auto scaling group可能會套用多個launch configuration，所以這個方式會把使用最早的launch configuration所launch的instance關閉。
* ClosestToNextInstanceHour：因為instance每一個小時都會收費，所以這個方式大致就是依照最近一次要收費的instance關閉。
* Default：先用 **OldestLaunchConfiguration** 的方式，若有多個instance符合的話再用 **ClosestToNextInstanceHour** ，若有多個instance符合的話再用 **亂數選擇** 其中一台instance關閉。

## Scenario

### 手動維護固定數量
新增名稱為my-test-lc的launch configuration，在開啟instance時使用的AMI為ami-0078da69，而且開啟的instance等級為m1.small。
<pre>as-create-launch-config my-test-lc --image-id ami-0078da69 --instance-type m1.small</pre>
新增名稱為my-test-asg的auto scaling group，使用名稱為my-test-lc的launch configuration，並套用在us-east-1a的AZ，執行時最少開1台，最多開10台，初始化時先開啟1台。
<pre>as-create-auto-scaling-group my-test-asg --launch-configuration my-test-lc --availability-zones us-east-1a --min-size 1 --max-size 10 --desired-capacity 1</pre>

### 動態調整
![AS Workflow](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/images/AS-WorkFlow.png)

新增名稱為my-test-lc的launch configuration，在開啟instance時使用的AMI為ami-514ac838，而且開啟的instance等級為m1.small。
<pre>as-create-launch-config my-test-lc --image-id ami-514ac838 --instance-type m1.small</pre>
新增名稱為my-test-asg的auto scaling group，使用名稱為my-test-lc的launch configuration，並套用在us-east-1e的AZ，執行時最少開1台，最多開5台，初始化時先開啟1台。
<pre>as-create-auto-scaling-group my-test-asg --launch-configuration my-test-lc --availability-zones us-east-1e --max-size 5 --min-size 1</pre>
新增名稱為my-scaleout-policy的scaling policy，套用在名稱為my-test-asg的auto scaling group上面，並且在達到條件(CloudWatch)的時候，開啟現在instance數量30%的instance。
<pre>as-put-scaling-policy my-scaleout-policy -–auto-scaling-group my-test-asg --adjustment 30 --type PercentChangeInCapacity</pre>
新增名稱為my-scalein-policy的scaling policy，套用在名稱為my-test-asg的auto scaling group上面，並且在達到條件(CloudWatch)的時候，關閉兩台instance。
<pre>as-put-scaling-policy my-scalein-policy –auto-scaling-group my-test-asg --adjustment -2 --type ChangeInCapacity</pre>
新增名稱為AddCapacity的CloudWatch，監控方式為每120秒監控一次名稱為my-test-asg的auto scaling group，若連續兩次的CPU使用率(CPUUtilization)大於等於80%，則啟動名稱為SCALE-OUT-POLICY的scaling policy。
<pre>mon-put-metric-alarm --alarm-name AddCapacity --metric-name CPUUtilization --namespace "AWS/EC2" --statistic Average --period 120 --threshold 80 --comparison-operator GreaterThanOrEqualToThreshold --dimensions "AutoScalingGroupName=my-test-asg" --evaluation-periods 2 --alarm-actions {SCALE-OUT-POLICY}</pre>
新增名稱為RemoveCapacity的CloudWatch，監控方式為每120秒監控一次名稱為my-test-asg的auto scaling group，若連續兩次的CPU使用率(CPUUtilization)小於等於40%，則啟動名稱為SCALE-IN-POLICY的scaling policy。
<pre>mon-put-metric-alarm --alarm-name RemoveCapacity --metric-name CPUUtilization --namespace "AWS/EC2" --statistic Average --period 120 --threshold 40 --comparison-operator LessThanOrEqualToThreshold --dimensions "AutoScalingGroupName=my-test-asg" --evaluation-periods 2 --alarm-actions {SCALE-IN-POLICY}</pre>

### 定期調整
