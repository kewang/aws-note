# Other
關於其他server或相關服務操作筆記

## NFS
安裝RPC及NFS相關工具
<pre>sudo yum install rpcbind nfs-utils</pre>

### NFS Server
編輯要分享的目錄，此處為/opt/nfs，目的為172.31.0.0/16，權限為可讀寫(rw)
<pre>echo "/opt/nfs	172.31.0.0/16(rw)" | sudo tee -a /etc/exports</pre>

啟動RPC及NFS service，並在開機時自動執行
<pre>sudo /etc/init.d/rpcbind start
sudo /etc/init.d/nfs start
sudo /etc/init.d/nfslock start
sudo chkconfig rpcbind on
sudo chkconfig nfs on
sudo chkconfig nfslock on</pre>

檢查是否有啟動成功
<pre>sudo tail /var/log/messages 
rpcinfo -p localhost
showmount -e localhost</pre>

### NFS Client
啟動RPC
<pre>sudo /etc/init.d/rpcbind start
sudo /etc/init.d/nfslock start</pre>

檢查是否可與NFS Server連線
<pre>showmount -e 172.31.14.244</pre>