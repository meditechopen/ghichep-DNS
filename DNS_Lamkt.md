# Tổng Quan giao thức DNS
## Mục lục

[I.Khái niệm] (#I)

[II.Cấu trúc của hệ thống tên miền] (#II)

[1.Cấu trúc cơ sở dữ liệu] (#1)

[2.Cấu trúc của tên miền] (#2)

[3.Máy chủ quản lý tên miền (Domain name server-dns)] (#3)

[4.Các bản ghi thường có trong cơ sở dữ liệu của DNS serrver] (#4)

[III.Phân loại DNS server](#III)

[IV.DNS Zone] (#IV)

[V.Đồng bộ dữ liệu giữa các DNS server] (#V)

[VI.Hoạt động của DNS server] (#VI)

[VII.Tham Khảo] (#VII)

<a name="I"></a>
### I.Khái niệm
- DNS (Domain Name System) được biết đến như là nameserver, là 1 hệ thống mạng có thể nối các hostname với các địa chỉ IP tương ứng của chúng.
- Với người dùng thì họ chỉ cần nhớ tên miền mà không cần nhớ đến địa chỉ IP mà vẫn có thể truy cập vào được.
- Hệ thống DNS là hệ thống sử dụng cơ sở dữ liệu phân tán và phân cấp hình cây do đó việc quản lý sẽ dễ dàng hơn 
	và cũng rất thuận tiên cho việc chuyển đổi từ tên miền sang địa chỉ IP và ngược lại.

<a name="II"></a>
### II.Cấu trúc của hệ thống tên miền

<a name="1"></a>
#### 1.Cấu trúc cơ sở dữ liệu
<img src="http://i.imgur.com/X6vryVd.png" />

- Cơ sở dữ liệu của hệ thống DNS là hệ thống cơ sở dữ liệu phân tán và phân cấp hình cây.
- Với .Root server là đỉnh của cây và sau đó các miền (domain) được phân nhánh dần xuống dưới và phân quyền quản lý.
- Khi một máy khách (client) truy vấn một tên miền nó sẽ đi lần lượt từ root phân cấp xuống dưới để đến DNS quản lý domain cần truy vấn.
- DNS cho phép phân chia tên miền để quản lý và nó chia hệ thống tên miền thành các zone và trong zone quản lý tên miền được phân chia đó.
- Các Zone chứa thông tin vê miền cấp thấp hơn, có khả năng chia thành các zone cấp thấp hơn và phân quyền cho các DNS server khác quản lý.

<a name="2"></a>
#### 2.Cấu trúc của tên miền
##### a.Cách đặt tên miền.
- Tên miền sẽ có dạng : Label.label.label….label.
- Độ dài tối đa của một tên miền là 255 ký tự.
- Mỗi một label tối đa là 63 ký tự bao gồm cả dấu “.”.
- Label phải được được bắt đầu bằng chữ số và chỉ được chứa chữ, số, dấu trừ (-).

##### b.Phân loại tên miền
| Tên | Ý nghĩa |
| --- | ------- |
| Com | Tên miền này được dùng cho các tổ chức thương mại |
| Edu | Tên miền này được dùng cho các cơ quan giáo dục, trường học |
| Net | Tên miền này được dùng cho các tổ chức mạng lớn |
| Gov | Tên miền này được dùng cho các tổ chức chính phủ |
| Org | Tên miền này được dùng cho các tổ chức khác |
| Int | Tên miền này dùng cho các tổ chức quốc tế |
| Info | Tên miền này dùng cho việc phục vụ thông tin |
| Arpa | Tên miền ngược |
| Mil | Tên miền dành cho các tổ chức quân sự, quốc phòng |
- Mã các nước:các quốc gia này được qui định bằng hai chữ cái theo tiêu chuẩn ISO-3166 (Vd : Việt Nam là .vn, Singapo la sg….)
Tổ chức ICANN đã thông qua hai tên miền mới là :
•	Travel : Tên miền dành cho tổ chức du lịch
•	Post : Tên miền dành cho các tổ chức bưu chính
Các tên miền dưới mức root này đươc gọi là Top –Level – Domain.

##### c.Cấu trúc tên miền
Tên miền được phân thành nhiêu cấp như:
- Gốc (Domain root):Nó là đỉnh của nhánh cây của tên miền. Nó xác định kết thúc của 
domain.Nó thể diễn đơn giản chỉ là dấu chấm “.”
- Tên miền cấp một (Top-level-domain) :Là gồm vài kí tự xác định một nước ,khu vưc hoặc tổ chức.Nó đươc thể hiện là “.com”
- Tên miền cấp hai(Second-level-domain):Nó rất đa dạng rất đa dạng có thể là tên một công ty, một tổ chức hay một cá nhân.
- Tên miền cấp nhỏ hơn (Subdomain): Chia thêm ra của tên miền cấp hai trở xuống thường được sử dụng như
 chi nhánh, phòng ban của một cơ quan hay chủ đề nào đó.Như phone.fpt.vn là một phòng của công ty Fpt.
Một số chú ý khi đặt tên miền:
- Tên miền nên đặt giới hạn từ cấp 3 đến cấp 4 vì nhiều hơn nữa việc nhớ tên và quản trị khó khăn.
- Sử dụng tên miền la phải duy nhất trong mạng Internet.
- Nên đặt tên đơn giản gợi nhớ và tránh.

<a name="3"></a>
#### 3.Máy chủ quản lý tên miền (Domain name server-dns)
- Máy chủ quản lý tên miền (dns) theo từng khu vực, theo từng cấp như : một tổ chức, một công ty hay một vùng lãnh thổ.
- Máy chủ đó chứa thông tin dữ liệu về địa chỉ và tên miền trong khu vực, trong cấp mà nó quản lý
- Nhiệm vụ:
<ul>
	<li>Chuyển giữa tên miền và địa chỉ IP</li>
	<li>Hỏi các máy chủ quản lý tên miền khác hoặc cấp cao hơn nó, để có thể đáp các truy vấn về những tên miền
	không thuộc quyền quản lý của nó và ngược lại.</li>
</ul>
- Máy chủ cấp cao nhất là Root Server do tổ chức ICANN quản lý:
<ul>
	<li>Là server quản lý toàn bộ cấu trúc của hệ thống tên miền.</li>
	<li>Không chứa dữ liệu thông tin về cấu trúc hệ thống DNS.</li>
	<li>Chuyển quyền (delegate) quản lý xuống cho các server cấp thấp hơn.</li>
	<li>Có khả năng định đường đến của một domain tại bất kì đâu trên mạng.</li>
	<li>Hiện nay trên thế giới có khoảng 13 root server quản lý toàn bộ hệ thống Internet</li>
</ul>
- Một DNS server có thể nằm bất cứ vị trí nào trên mạng Internet.
- Nên đặt DNS tại vị trí nào gần với các client để dễ dàng truy vấn đến đồng thời cũng gần với vị trí của DNS server cấp cao hơn trực tiếp quản lý nó.
- Vị trí các root server trên thế giới
<img src="http://i.imgur.com/5UL65Wv.png" />

<a name="4"></a>
#### d.Các bản ghi thường có trong cơ sở dữ liệu của DNS serrver
| Tên trường | Tên đầy đủ | Mục đích |
| ---------- | ---------- | -------- |
| SOA | Start of Authority | Xác định máy chủ DNS có thẩm quyền cung cấp thông tin về tên miền xác định trên DNS |
| NS | Name Server | Chuyền quyền quản lý tên miền xuống 1 DNS cấp thấp hơn |
| A | Host | Ánh xạ giữa tên của một máy tính trên mạng và địa chỉ IP của một máy tính trên mạng |
| MX | Mail Exchanger | Xác định host có quyền quản lý thư điện tử cho 1 tên miền xác định |
| PTR | Pointer | Chuyển đổi địa chỉ IP sang tên miền |
| CNAME | Canonical Name | Thường sử dụng xác địch dịch vụ Web Hosting |

<a name="III"></a>
### III.Phân loại DNS server
Có 3 loại DNS server sau.
- Primary server
  <ul>
	<li>Nguồn xác thực thông tin chính thức cho các domain mà nó được phép quản lý.</li>
	<li>Thông tin về tên miền do nó được phân cấp quản lý thì được lưu trữ tại đây và sau đó có thể được chuyển sang cho các secondary server.</li>
	<li>Các tên miền do primary server quản lý thì được tạo và sửa đổi tại primary server và được cập nhật đến các secondary server.</li>
	<li>Primary server nên đặt gần với các client để có thể phục vụ truy vấn tên miền một cách dễ dàng và nhanh hơn.</li>
  </ul>
  
- Secondary server
  <ul>
	<li>Dùng để lưu trữ dự phòng cho primary server nên là không bắt buộc phải có.</li>
	<li>Nó được phép quản lý những dữ liệu về tên miền, nhưng không tạo ra các bản ghi về tên miền mà lấy về từ primary server.</li>
	<li>Khi lượng truy vấn zone tăng cao tại primary server thì nó sẽ chuyển bớt tải sang cho secondary server.</li>
	<li>Khi primary server gặp sự cố không hoạt động được thì secondary server sẽ hoạt động thay thế cho đến khi primary server hoạt động trở lại.</li>
	<li>Secondary server nên được đặt ở gần với primary server và client để có thể phục vụ cho việc truy vấn tên miền dễ dàng hơn.</li>
	<li>Không nên cài đặt secondary server trên cùng một mạng con (subnet) hoặc cùng một kết nối với primary server, vì khi 
	primary server có kết nối bị hỏng thì không ảnh hưởng đến nó</li>
	<li>2 cơ chế cho phép lấy thông tin về các zone mới từ primary server: lấy toàn bộ hoặc chỉ lấy phần thay đổi.</li>
  </ul>
  
- Caching-only server: Không quản lý domain, chỉ phục vụ cho phép truy vấn tìm kiếm nhanh.
- Stub server: chứa danh sách các DNS server đã được authoritative từ primary DNS, giúp tăng tốc độ phân giải tên và dễ quản lý.

<a name="IIII"></a>
### IV.DNS Zone
- DNS Zone là tập hợp các ánh xạ từ host đến địa chỉ IP và từ IP đến host của một phần liên tục trong một nhánh của domain.
- Thông tin của DNS Zones là những record gồm tên Host và địa chỉ IP được lưu trong DNS Server, 
DNS Server quản lý và trả lời những yêu cầu từ Client liên quan đến DNS Zones này.
- Zone file : Lưu thông tin của Zone, có thể ở dạng text hoặc trong Active Dicrectory.
- Có 2 loại DNS Zone : Standard Primary Zone và Active Directory Integrated Zones.

#### 1.Standard Primary Zone
- Được sử dụng trong các single domain, không có Active Dicrectory .
- Tất cả những thay đổi trong Zone sẽ không ảnh hưởng đến các Zone khác.
- Nếu ta tạo thêm một Zone (Secondary Zone), thì Zone này sẽ bị ảnh hưởng từ Primary Zone. Secondary Zone sẽ lấy thông tin từ Primary Zone.

#### 2.Active Directory Integrated Zones
- Mặc định sẽ được tạo khi máy tính chạy DNS Server được nâng cấp thành Domain Contronller. 
Active Directory Integrated Zones thực chất là Zone được nâng cấp lên từ Standard Primary Zone khi lên Domain Controller.
- DNS Zones được lưu như một đối tượng trong cơ sở dữ liệu của Active Directory. 
- Thông tin về DNS Zones đều chứa trên tất cả Domain Contronller. Cho phép việc cập nhật tự động cơ sở dữ liệu DNS Zones bảo mật (secure updates ).

### V.Đồng bộ dữ liệu giữa các DNS server
#### 1.Các phương pháp đồng bộ dữ liệu giữa các DNS server
- Để đảm bảo hoạt động ổn định, thì ta nên dùng trên 1 DNS server.
- Do vậy ta phải có cơ chế chuyển dữ liệu các zone và đồng bộ giữa các DNS server # nhau:
  <ul>
	<li>Truyền toàn bộ zone(all zone transfer ): nhân toàn bộ dữ liệu từ primary server sang secondary server.</li>
	<li>Truyền phần thay đổi(Incremental zone): chỉ truyền những những dữ liệu thay đổi của zone . </li>
  </ul>
- Truyền zone xảy ra trong các trường hợp sau:
  <ul>
	<li>Khi quá trình làm mới của zone kết thúc</li>
	<li>Khi secondary server được thông báo zone đã thay đổi tại server nguồn quản lý zone</li>
	<li>Khi dịch vụ DNS bắt đầu chạy lại secondary server</li>
	<li>Tại secondary server yêu cầu chuyển zone </li>
  </ul>

#### 2.Cơ chế hoạt động đồng bộ dữ liệu giữa các DNS server
- Viết tắt:IXFR Zone được thực hiện chỉ khi số serial của nguồn dữ liệu và bản sao của nó khác nhau.
- Các bước yêu cầu truyền zone:
<ul>
	<li>Khi cấu hình DNS Server mới, nó sẽ gửi truy vấn yêu cầu gửi toàn bộ Zone ( all Zone transfer request (AXFR) ) đến DNS Server chính của Zone.</li>
	<li>DNS Server chính trả lời và chuyển toàn bộ dữ liệu về Zone cho Secondary Server (destination) mới cấu hình.</li>
	<li>Để xác định có chuyển dữ liệu hay không thì nó dựa vào số serial được khai báo bằng bản ghi SOA.</li>
	<li>Khi thời gian làm mới của Zone đã hết, thì DNS Server nhận dữ liệu sẽ truy vấn yêu cầu làm mới Zone tới DNS Server chính.</li>
	<li>DNS Server chính sẽ trả lời truy vấn và gửi lại dữ liệu. Trả lời truy vấn dữ liệu gồm số serial của Zone tại DNS Server chính.</li>
	<li>DNS Server nhận dữ liệu về Zone và sẽ kiểm tra số serial trong trả lời và quyết định xem có cần truyền dữ liêu không :
		<ul>
		<li>Nếu số serial bằng nhau thì nó kết thúc luôn, thiết lập lại các thông số cũ lưu trong máy.</li>
		<li>Nếu số serial tại Primary Server lớn hơn giá trị serial hiện tại DNS nhận dữ liệu.
		Thì nó kết luận Zone cần được cập nhật và cần đồng bộ dữ liệu giữa hai DNS Server</li>
		</ul>
	</li>
	<li>Nếu có kết luận cần truyền zone thì nó sẽ gửi yêu cầu IXFR tới DNS Server chính để yêu cầu truyền dữ liệu của Zone.</li>
	<li>DNS Server chính sẽ trả lời với việc gửi những thay đổi của Zone hoặc toàn bộ Zone tùy xem nó có hỗ trợ hay không.</li>
</ul>

<a name="VI"></a>
### VI.Hoạt động của DNS server
- Hệ thống DNS hoạt động tại mức4 của mô hình OSI, nó sử dụng truy vấn bằng giao thức UDP, sử dụng cổng 53 để trao đổi thông tin.
- Các DNS server đều được kết nối logic với nhau, tất cả các DNS server phải biết ít nhất một cách đến root server, một máy tính 
kết nối mạng phải biết làm thế nào để liên lạc với ít nhất là một DNS server.
- Khi DNS client cần xác định cho một tên miền nó sẽ truy vấn DNS.
- Mỗi message truy vấn gồm 3 phần:
	<ul>
		<li>Tên của miền cần truy vấn</li>
		<li>Xác định loại bản ghi(mail, web…)</li>
		<li>Lớp tên miền</li>
	</ul>
- Khi DNS nhận được một truy vấn.Trước tiên nó sẽ kiểm tra thông tin có phải trong bản ghi nó đang quản lý không, nếu có sẽ trả lời và kết thúc.
- Nếu không có nó sẽ kiểm tra trong cache xem có truy vấn nào trước đây tương tự không nếu có sẽ trả lời và kết thúc.
- Nếu không có thông tin phù hợp nó sẽ nhờ tới DNS server khác để trả lời.
- VD
<img src="http://i.imgur.com/OqTEOMX.png" />

- Bước 1:PC A truy vấn DNS server vdc.com.vn tên miền www.abc.com.sg.
- Bước 2:DNS server vdc.com.vn không quản lý tên miền www.abc.com.sg vậy nó sẽ chuyển lên root server.
- Bước 3:Root server sẽ không xác định được DNS server quản lý trực tiếp tên miền www.abc.com.sg, 
		nó sẽ căn cứ vào cấu trúc của hệ thống tên miền để chuyển tới DNS server cấp cao hơn của abc.com.sg đó là com.sg,
		nó xác định được rằng DNS server DNS.com.sg quản lý tên miền com.sg
- Bước 4:DNS.com.sg sau đó sẽ xác định được rằng DNS server DNS.abc.com.sg có quyền quản lý tên miền www.abc.com.sg.
- Bước 5:DNS.abc.com.sg sẽ lấy bản ghi xác định cho tên miền DNS.abc.com.sg để trả lời cho DNS server DNS.com.sg.
- Bước 6:DNS.com.sg chuyển câu trả lời cho root server.
- Bước 7:Root server chuyển câu trả lời cho DNS server vdc.com.vn
- Bước 8:DNS server vdc.com.vn trả lời về cho PC A, PC A đã kết nối được tới host tên miền www.abc.com.sg.
- Khi truy vấn lặp đi lặp lại thì hệ thống DNS có khả năng thiết lập chuyển quyền trả lời tới DNS trung gian mà không cần qua root server
 và nó cho phép thời gian truy vấn giảm đi.
- Khi DNS xử lý các yêu cầu truy vấn của client và sử dụng các truy vấn lặp lại. Nó sẽ xác định và lưu lại các thông tin quan trọng của tên miền mà cliet truyvấn.
 Thông tin đó được ghi nhớ trong bộ nhớ cache của DNS server.
 
<a name="VII"></a>
### VII.Tham Khảo
- http://forum.itlab.com.vn/threads/tong-quan-ve-dns-va-cau-truc-dns.1661/
- http://monhoc.vn/tai-lieu/bai-giang-mang-can-ban-chuong-5-he-thong-ten-mien-dns-2655/
- http://webcache.googleusercontent.com/search?q=cache:http://maiquocthai.blogspot.com/2010/11/dnsdns-zoneong-bo-dl-giua-cac-dns.html
