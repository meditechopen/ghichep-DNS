# Cấu Hình Master-Slave DNS Server
## Mục lục
[1.Mô hình mạng] (#1)

[2.Giới thiệu] (#2)

[3.Các bước triển khai] (#3)

[4.Cấu hình chi tiết] (#4)

[5.Tham Khảo] (#5)

<a name="1"></a>
### 1.Mô hình mạng
<img src="http://i.imgur.com/Jt7lQM0.png" />

<a name="2"></a>
### 2.Giới thiệu
- DNS được dùng để phân giải tên miền cho các host.
- Master DNS server	là server chính xử lý dữ liệu, Slave DNS server là server dự phòng và nó copy tất cả thông tin tương tự từ Master.
- Nó sử dụng phương thức UDP để phân giải tên miền vì UDP không có quá trình xác thực do đó sẽ nhanh hơn TCP.
- Trong bài lab này t sử dụng 4 máy ảo, 1 Master DNS server , 1 Slave DNS server và 2 client.
```sh
--------------------------------------------------------------
Master DNS server
--------------------------------------------------------------
IP address		:		192.168.10.1
Host-name		:		dns1.lamtlu.com
OS				:		Centos 6.7 Final
```

```sh
--------------------------------------------------------------
Slave DNS server
--------------------------------------------------------------
IP address		:		192.168.10.2
Host-name		:		dns2.lamtlu.com
OS				:		Centos 6.7 Final
```

```sh
--------------------------------------------------------------
client 1
--------------------------------------------------------------
IP address		:		192.168.10.100
Host-name		:		centos.lamtlu.com
OS				:		Centos 6.7 Final
```

```sh
--------------------------------------------------------------
client 2
--------------------------------------------------------------
IP address		:		192.168.10.101
Host-name		:		win7.lamtlu.com
OS				:		window 7
```

- Các gói cài đặt
`bind, bind-utils, bind-chroot`
- Port và protocol được sử dụng
`53, UDP`

<a name="3"></a>
### 3.Các bước triển khai
- Cấu hình Master DNS server.
<ul>
	<li>Cài đặt gói bind.</li>
	<li>Cấu hình bind.</li>
	<li>Tạo các file zone.</li>
	<li>Thêm rule trong iptables.</li>
	<li>Khởi động dịch vụ named và test</li>
</ul>
- Cấu hình Slave DNS server tương tự Master.
- Cấu hình client.
- Test trên client.

<a name="4"></a>
### 4.Cấu hình chi tiết
#### a.Cấu hình DNS server
- Trước tiên kiểm tra lại IP address, hostname,phiên bản của DNS server.
```sh
ip a | grep inet
hostname
cat /etc/redhat-release
```
<img src="http://i.imgur.com/KQ3DyLd.png" />

##### I.Cài đặt gói bind
- Sau khi kiểm tra các thông tin đã đúng, ta cài đặt các gói cần thiết.
`yum -y install bind* -y`

##### II.Cấu hình bind
- Tiếp theo ta định nghĩa cho file zone trong file named.conf
`vi /etc/named.conf`
- Dưới đây là file named.conf của tôi.
```sh
options {
	listen-on port 53 { <strong style="color: yellow;">127.0.0.1; 192.168.10.1</strong>; }; # add IP của DNS server
	listen-on-v6 port 53 { none; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { localhost; 192.168.10.0/24; };
    <strong style="color: yellow;">recursion no</strong>;
	
	dnssec-enable yes;
	dnssec-validation yes;
	dnssec-lookaside auto;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
zone "." IN {
		type hint;
		file "named.ca";
};

## Define our forward & reverse Zone file here for lamtlu.com
zone "<strong style="color: yellow;">lamtlu.com</strong>" IN{
		type master;
		file "<strong style="color: yellow;">lamtlu.forward</strong>";
		allow-update {none; };
	};
zone "<strong style="color: yellow;">10.168.192.in-addr.arpa</strong>" IN{
		type master;
		file "<strong style="color: yellow;">lamtlu.reverse</strong>";
		allow-update {none; };
	};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
<img src="http://i.imgur.com/aqHAIHT.png" />

- Ý nghĩa các tham số quan trọng:
<ul>
	<li>listen-on port 53 – giúp DNS có thể listen trên các interface đã enable.</li>
	<li>allow-query: Dải subnet sẽ dùng dịch vụ DNS.</li>
	<li>allow-transfer : IP của slave DNS server.</li>
	<li>recursion no : nếu bạn để "yes", truy vấn đệ quy có thể làm cho server bị tấn công DDos.</li>
	<li>Zone Name : định nghĩa tên zone ví dụ như lamtlu.com.</li>
	<li>lamtlu.forward: File này có những thông tin cho các host của zone.</li>
	<li>allow-update none – không sử dụng dynamic dns(DDNS).</li>
</ul>

##### III.Tạo file zone
- Trước tiên ta phải tạo các file zone với tên đã được đưa ra trong file named.conf
- Ta sử dụng các file cấu hình mẫu để tạo file zone bằng cách copy.
```sh
cp /var/named/named.localhost /var/named/lamtlu.forward
cp /var/named/named.loopback /var/named/lamtlu.reverse
```
- Tiếp theo vào chỉnh sửa các file zone này .
`vi /var/named/lamtlu.forward`
- Trước khi chỉnh sửa file zone forward, ta nhìn qua file mẫu:
<img src="http://i.imgur.com/SE96RP2.png" />

- Sau đây là file zone forward của tôi, bạn có thể thay đổi theo ý của bạn
```sh
$TTL 86400
@   IN SOA     dns1.lamtlu.com. root.lamtlu.com. (
2016022801 ;Serial
3600       ;Refresh
1800       ;Retry
604800     ;Expire
86400       ;Minimum TTL
)
; Name server's
@       IN NS        	 dns1.lamtlu.com.
@       IN NS        	 dns2.lamtlu.com.


; Name server hostname to IP resolve.
@    	IN A          	 192.168.10.1
@    	IN A          	 192.168.10.2

; Hosts in this Domain
@    	IN A          	 192.168.10.100
@		IN A			 192.168.10.101
dns1    IN A          	 192.168.10.1
dns2    IN A          	 192.168.10.2
Centos  IN A          	 192.168.10.100
win7	IN A			 192.168.10.101

```
<img src="http://i.imgur.com/7ZaJ6VE.png" />

- Trước khi cấu hình file zone reverse, ta xem qua file mẫu
<img src="http://i.imgur.com/sUyoHGz.png" />

- Sau đây là file zone reverse của tôi
```sh

$TTL 86400
@   IN SOA     dns1.lamtlu.com. root.lamtlu.com. (
2016022801 ;Serial
3600       ;Refresh
1800       ;Retry
604800     ;Expire
86400       ;Minimum TTL
)
; Name server's
@      	IN NS         	dns1.lamtlu.com.
@      	IN NS         	dns2.lamtlu.com.
@       IN PTR        	lamtlu.com.

; Name server hostname to IP resolve
dns1    IN A          	192.168.10.1
dns2    IN A          	192.168.10.2


; Host in Domain
1       	IN PTR      dns1.lamtlu.com.
2       	IN PTR      dns2.lamtlu.com.  
100			IN PTR		centos.lamtlu.com
101			IN PTR		win7

centos  	IN A        192.168.10.100
win7		IN A		192.168.10.101

```
<img src="http://i.imgur.com/tqHIga7.png" />

- Thay đổi quyền nhóm "named" cho các file forward và reverse:
```sh
chgrp named /var/named/lamtlu.forward
chgrp named /var/named/lamtlu.reverse
```
- Ta kiểm tra lại
`ll /var/named`
<img src="http://i.imgur.com/eI3hGiu.png" />

- Tiếp theo check lỗi các file zone trước khi chạy dịch vụ DNS.Check file named.conf trước, sau check file zones.
```sh
named-checkconf /etc/named.conf
named-checkzone dns1.lamtlu.com /var/named/lamtlu.forward
named-checkzone dns1.lamtlu.com /var/named/lamtlu.reverse
```

##### IV.Thêm rule trong iptables
- Mặc định khi iptables chạy thì các yêu cầu tới DNS server bị hạn chế.
- Do đó ta cần phải add thêm rule inbound cho cổng 53.
```sh
iptables -I INPUT -p udp --dport 53 -m state --state NEW -j ACCEPT
```
- Kiểm tra lại rule đã được thêm vào đúng hay chưa.
```sh
iptables -L INPUT
```
- Cuối cùng save rule và restart firewall.
```sh
service iptables save
service iptables restart
```
<img src="http://i.imgur.com/wDMGEaO.png" />

##### V.Khởi động dịch vụ named và test
- Khởi động dịch vụ và để nó chạy mặc định khi bật máy.
```sh
service named start
chkconfig named on
chkconfig --list named
```
<img src="http://i.imgur.com/1dKGhQL.png" />

- Cuối cùng ta test service sử dụng 2 tool "dig và nslookup"
```sh
dig dns1.lamtlu.com		[forward zone]
```
<img src="http://i.imgur.com/Md1arjJ.png" />

```sh
dig -x 192.168.10.1		[reverse zone]
```
<img src="http://i.imgur.com/6kHWQj0.png" />

```sh
nslookup lamtlu.com
nslookup dns1.lamtlu.com
nslookup dns2.lamtlu.com
```
<img src="http://i.imgur.com/P8QBRhL.png" />

- Vậy là chúng ta đã cấu hình xong Master DNS server, tiếp theo ta cấu hình Slave DNS server.

#### b.Cấu hình Slave DNS server
##### I.Cài đặt gói bind
- Với Slave, ta cài đặt các gói giống như Master
```sh
yum install bind* -y
```

##### II.Cấu hình bind
- Chỉnh sửa file "named.conf"
```sh
vi /etc/named.conf
```
- Thêm thay đổi theo ý của bạn, dưới đây là file conf của tôi.
```sh
options {
	listen-on port 53 { <strong style="color: yellow;">127.0.0.1; 192.168.10.2</strong>}; #Add ip của slave
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { <strong style="color: yellow;">192.168.100.0/24</strong>; };
	recursion no;

	dnssec-enable yes;
	dnssec-validation no;
	dnssec-lookaside auto;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

## Define our slave forward and reverse zone, Zone files are replicated from master.

zone"<strong style="color: yellow;">lamtlu.com</strong>" IN {
type slave;
file "<strong style="color: yellow;">slaves/lamtlu.forward.slave</strong>";
masters { <strong style="color: yellow;">192.168.10.1</strong>; };
};

zone"<strong style="color: yellow;">0.168.192.in-addr.arpa</strong>" IN {
type slave;
file "<strong style="color: yellow;">slaves/lamtlu.reverse.slave</strong>";
masters { <strong style="color: yellow;">192.168.10.1</strong>; };
};

#####
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```
<img src="http://i.imgur.com/LnB09sL.png">
##### III.Tạo các file zone
- Tiếp theo các bạn tạo các file zone giống như của Master trong đường dẫn /var/named/slaves.Tôi đã tạo và kiểm tra lại.
<img src="http://i.imgur.com/231VObn.png" />

##### IV.Thêm rule trong iptables
- Tiếp theo ta mở port 53 trong iptables để cho phép các request tới DNS server.
```sh
iptables -I INPUT -p udp --dport 53 -m state --state NEW -j ACCEPT
iptables -L INPUT
```
- Lưu iptables rules và khởi động lại iptables
```sh
service iptables save
service iptables restart
chkconfig iptables on
chkconfig --list iptables
```
<img src="http://i.imgur.com/fKEIOWD.png" />
##### V.Khởi động dịch vụ named
- Start dịch vụ và bật nó mặc định chạy khi khởi động máy.
```sh
service named start
chkconfig named on
```
#### c.Cấu hình client.
##### I.Client Centos
- Cấu hình IP và DNS 
<img src="http://i.imgur.com/fmLC36X.png" />

- Chỉnh sửa file "/etc/resolv.conf" và thêm những dòng sau
```sh
search lamtlu.com
nameserver 192.168.10.1
nameserver 192.168.10.2
```

##### II.Win7
<img src="http://i.imgur.com/XYWhkKT.png" />

#### d.Test trên client.
##### I.Centos
- Kiểm tra nslookup
<img src="http://i.imgur.com/4sHHpOt.png" />

- Kiểm tra các tìm kiếm DNS forward and reverse
```sh
dig dns1.lamtlu.com
```
<img src="http://i.imgur.com/FAZGatc.png" />

```sh
dig -x 192.168.10.1
```
<img src="http://i.imgur.com/PdbBTTe.png" />

- Giái thích các tham số:
<ul>
	<li>Header: Cho ta biết các truy vấn và kết quả của các host.</li>
	<li>Status NOERROR: tức là ta truy vấn thành công.</li>
	<li>Question: Tên truy vấn, trong vd này là dns1.lamtlu.com. </li>
	<li>Answer: trả lời truy vấn nếu đó là thông tin đúng.</li>
	<li>Authority: Tên của server trả lời các truy vấn.</li>
	<li>Additional : Các thông tin thêm về name-servers như host-name và địa chỉ IP.</li>
	<li>Query time: Thời gian để phân giải tên miền.</li>
</ul>
- Ta kiểm tra chính host của mình.
`dig centos.lamtlu.com`
<img src="http://i.imgur.com/1t0WPgh.png" />

- Ta ping các máy còn lại:
```sh
ping dns1.lamtlu.com -c 2
ping dns2.lamtlu.com -c 2
ping 192.168.0.100 -c 2
ping 192.168.0.101 -c 2
```
<img src="http://i.imgur.com/buTiAn2.png" />

##### II.Win7
- Mở cmd và test
<img src="http://i.imgur.com/Cox3W28.png" />

<a name="5"></a>
### 5.Tham Khảo
- http://www.tecmint.com/setup-master-slave-dns-server-in-centos/
