# VPC
Virtual Private Cloud，你可以完全自訂這個網路裡面的subnet、ip range、routing table、gateway、security...等，當然也可以加EC2的instance進去。

## Subnet
是一段IP位址的範圍，aws的resource都可以擺在subnet裡面。一般會分為兩種：
* Public Subnet：可以連上Internet的叫做Public Subnet
* Private Subnet：沒辦法連上Internet的叫做Private Subnet

## default VPC
如果你的region只有EC2-VPC而沒有EC2-Classic的話，在新增instance的時候就會有一個預設的VPC可以使用，並且有以下特性：
* **每一個instance都有一個public IP**
* 網段預設為172.31.0.0/16，在新建instance的時候會自己切兩個subnet，分別為172.31.0.0/20及172.31.16.0/20。

## nondefault VPC
如果想要自建私有雲，或是想客製化網路環境的話，可以自建VPC實作。 **因為自建VPC並不提供Public IP，所以必須搭配EIP及NAT...等技術才能與Internet連線** 。基本上有以下四種情境，其他的變型情境都可以用這四種來衍生。

1. Public Subnet – 用來提供公眾型的應用服務幾乎都應該適用，可大大增加安全性，也能自定內部IP位置
2. **Public + Private Subnet – 有些用戶會想把DB Server等重要服務移至不能直接存取的區域，便可選此模式**
3. VPN + Public + Private Subnet – 就是上一個模式中加上一個Hardware VPN設備來與企業的VPN設備串接
4. VPN + Private Subnet – 把AWS完全用來當做私有雲，一樣用Hardware VPN設備來與企業的VPN設備串接

## 常見使用情境
由於第三及第四種使用情境比較不符我們的使用方式，所以在此不深入研究，這邊只介紹第一及第二種情境。

### Public Subnet
![Public Subnet](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/images/Case1_Diagram.png)

#### 基礎元件
* VPC底層使用10.0.0.0/16的網段，可以提供65536個private IP
* 一個subnet，並且設定為10.0.0.0/24，這可以提供256個private IP。
* 一個Internet Gateway，這可以讓VPC連到Internet以及其他的AWS服務上面(例如S3)。
* 一個EC2 instance，這個instance必須要加到這個VPC裡面才能跟其他instance連線，並且啟用EIP，這樣子instance才有Public IP可以讓Internet連線。
* 一個route table，控制VPC裡面的所有resource要如何連線。

#### 路由規則
<table>
	<tr>
		<th>Destination</th>
		<th>Target</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>10.0.0.0/16</td>
		<td>local</td>
		<td>表示所有連線到10.0.0.0/16網段的request，都要走local內網</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>igw-xxxxxxxx</td>
		<td>表示所有連線到0.0.0.0/0網段的request，都要走到internet gateway，這樣才能連線到Internet</td>
	</tr>
</table>

#### 安全規則

##### 流入
<table>
	<tr>
		<th>Source</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許Internet可以讓所有HTTP連線連進來</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許Internet可以讓所有HTTPS連線連進來</td>
	</tr>
	<tr>
		<td>x.x.x.x</td>
		<td>TCP</td>
		<td>22</td>
		<td>允許特定IP可以使用SSH連進來</td>
	</tr>
</table>

##### 流出
<table>
	<tr>
		<th>Destination</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
</table>

### Public + Private Subnet
![Public + Private Subnet](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/images/Case2_Diagram.png)

#### 基礎元件
* VPC底層使用10.0.0.0/16的網段，可以提供65536個private IP
* 一個public subnet，並且設定為10.0.0.0/24，這可以提供256個private IP。
* 一個private subnet，並且設定為10.0.1.0/24，這可以提供256個private IP。
* 一個Internet Gateway，這可以讓VPC連到Internet以及其他的AWS服務上面(例如S3)。
* 將要對外的instance放在public subnet，例如web server；將不對外的instance放在private subnet，例如ap server和db server。
* 一個NAT的instance，讓private subnet的instance利用這個instance可以與internet連線，主要功能是拿來做系統更新用。
* 兩個route table，分別控制VPC裡面public及private subnet的所有resource要如何連線。

#### 路由規則

##### Public subnet
<table>
	<tr>
		<th>Destination</th>
		<th>Target</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>10.0.0.0/16</td>
		<td>local</td>
		<td>表示所有連線到10.0.0.0/16網段的request，都要走local內網</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>igw-xxxxxxxx</td>
		<td>表示所有連線到0.0.0.0/0網段的request，都要走到internet gateway，這樣才能連線到Internet</td>
	</tr>
</table>

##### Private subnet
<table>
	<tr>
		<th>Destination</th>
		<th>Target</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>10.0.0.0/16</td>
		<td>local</td>
		<td>表示所有連線到10.0.0.0/16網段的request，都要走local內網</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>eni-xxxxxxxx / i-xxxxxxxx</td>
		<td>表示所有連線到0.0.0.0/0網段的request，都要走到特定的網路介面(eni-xxxxxxxx)或NAT instance(i-xxxxxxxx)，這樣才能透過Public subnet的NAT instance連線到Internet</td>
	</tr>
</table>

#### 安全規則

##### WebServerSG
給Public subnet裡面的web server使用

<table>
	<tr>
		<th colspan="4">流入</th>
	</tr>
	<tr>
		<th>Source</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許Internet可以讓所有HTTP連線連進來</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許Internet可以讓所有HTTPS連線連進來</td>
	</tr>
	<tr>
		<td>x.x.x.x</td>
		<td>TCP</td>
		<td>22</td>
		<td>允許特定IP可以使用SSH連進來</td>
	</tr>
	<tr>
		<th colspan="4">流出</th>
	</tr>
	<tr>
		<th>Destination</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
	<tr>
		<td>DBServerSG-id</td>
		<td>TCP</td>
		<td>3306</td>
		<td>允許套用這個rule的instance，可以直接連線到DB server(此處以MySQL為例)</td>
	</tr>
</table>

##### NATSG
給Public subnet裡面的NAT instance使用。

<table>
	<tr>
		<th colspan="4">流入</th>
	</tr>
	<tr>
		<th>Source</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>10.0.1.0/24</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許讓private subnet的HTTP連線連進來</td>
	</tr>
	<tr>
		<td>10.0.1.0/24</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許讓private subnet的HTTPS連線連進來</td>
	</tr>
	<tr>
		<td>x.x.x.x</td>
		<td>TCP</td>
		<td>22</td>
		<td>允許特定IP可以使用SSH連進來</td>
	</tr>
	<tr>
		<th colspan="4">流出</th>
	</tr>
	<tr>
		<th>Destination</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
</table>

##### DBServerSG
給Private subnet裡面的DB instance使用。

<table>
	<tr>
		<th colspan="4">流入</th>
	</tr>
	<tr>
		<th>Source</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>WebServerSG-id</td>
		<td>TCP</td>
		<td>3306</td>
		<td>允許public subnet可以用DB connection(此處以MySQL為例)連進來</td>
	</tr>
	<tr>
		<th colspan="4">流出</th>
	</tr>
	<tr>
		<th>Destination</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Comments</th>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>80</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
	<tr>
		<td>0.0.0.0/0</td>
		<td>TCP</td>
		<td>443</td>
		<td>允許套用這個rule的instance，可以直接連線到Internet，一般是拿來做系統更新用</td>
	</tr>
</table>

## Security
![Route Traffic](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/images/Route_Traffic.png)

連線順序如下：

1. Internet連線會先連到Internet Gateway之後，連進VPC。
2. VPC會依照Routing Table，將不同的連線導到不同的subnet。
3. 在連進subnet之前，會先套用Network ACL，限制連線的ALLOW或DENY。
4. 在subnet分派連線給instance之前，會再透過SG限制是否可以連到instance。

### Security groups (SG)
安全群組，屬於instance level的安全管理機制，必須與EC2 instance綁在一起。在launch一個instance如果沒選擇sg時，AWS會自動將instance指定到一個預設的sg。
* 只支援ALLOW規則
* 無論規則如何，response connection為永遠ALLOW

### Network access control lists (ACLs)
網路存取控制清單，屬於subnet level的安全管理機制，必須與subnet綁在一起。
* 支援ALLOW及DENY規則
* response connection會依照規則做控制
* 規則會依照rule number的順序做套用