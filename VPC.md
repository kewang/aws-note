# VPC
Virtual Private Cloud

## 概念

### VPC
VPC就是一個虛擬網路的意思，你可以完全自訂這個網路裡面的subnet、ip range、routing table、gateway、security...等，當然也可以加EC2的instance進去。

### Subnet
是一段IP位址的範圍，aws的resource都可以擺在subnet裡面。一般會分為兩種：
* Public Subnet：可以連上Internet的叫做Public Subnet
* Private Subnet：沒辦法連上Internet的叫做Private Subnet