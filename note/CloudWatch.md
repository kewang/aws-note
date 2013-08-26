# CloudWatch
針對全部AWS系統的數據做監視及統計，有以下特性：
* 只留存2週。
* 若要針對應用面的數據做統計，則應該要再加上MRTG的方式做處理。

總共分為Basic及Detailed兩種型式，差異如下：
* Basic：取樣最快5 mins
* Detailed：取樣最快可以1 min

## 新增Alarm
![CloudWatch Alarm](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/images/AlarmsGraph.png)

可以用不同的評測資料(metric)來新增alarm，評測基準如下：
* EC2的instance CPU Utilization、EBS的write bytes...等，有非常多樣化。
* 計算方式：平均值、最大值、總和
* 取樣時間：1min、5mins...最長1day

### 門檻(threshold)
依照取樣時間設計，若現在想做連續兩次取樣10mins，則threshold要設計為20mins，而取樣時間要設計為10mins。

### 狀態
* OK：尚未達到threshold時的狀態
* ALARM：超過threshold時的狀態
* INSUFFICIENT：資料不完整的狀態

### Alarm轉換狀態時的動作(Action)
依照不同評測資料而有不同的action，例如EC2的Instance metric可以寄送mail、關機，而EBS的相關metric只能寄送mail。另外，也會依照不同的action type而有不同的執行次數。
* 執行auto scaling時，action會在alarm時持續執行，直到轉換狀態。
* 執行SNS時，action只會在第一次達到alarm時執行一次。

### 客製Metric
[Amazon CloudWatch Concepts](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/cloudwatch_concepts.html)