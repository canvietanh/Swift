#Chapter 1: Sự phát triển của lưu trữ dữ liệu

- Nhu cầu về dữ liệu tăng cao như dữ liệu đa phương tiện, game, video, hình ảnh, ca nhạc...
- Phát triển SDS (Sofware-defined storage)  khiến hệ thống lưu trữ dễ dàng truy cập và mở rộng
- Yêu cầu về lưu trữ dữ liệu cần đảm bảo các yếu tố: 
  + Độ bền: đảm bảo dữ liệu không bị hỏng hoặc mất kể cả với các lý do khách quan
  + Tính sẵn sàng: dữ liệu cần luôn sẵn sàng để được sử dụng với mọi loại thiết bị với người dùng khác nhau.
  + Khả năng quản lý: dữ liệu nhiều tuy nhiên phải được quản lý dễ dàng
  + Chi phí thấp: khả năng mở rộng sẽ rất cao nên cần chi phí phần cứng hay nhân công khi mở rộng cần thấp nhất có thể.
  
 - Định lý CAP về hệ thống máy tính phân tán: chỉ đảm bảo được 2 trong 3 yếu tố: tính nhất quán, tính sẵn có, hoặc khả năng phân vùng.
 - Swift đảm bảo được tính sẵn có và khả năng phân vùng
 - So sánh về các loại storage:
https://github.com/canvietanh/ghi-chep-storage/blob/master/Block-object-filesystem.md
- Kiến trúc lưu trữ mới với SDS (đề cao khả năng chịu lỗi) gồm 4 thành phần chính:
 + Định tuyến lưu trữ: định tuyến yêu cầu tránh các node lỗi
 + Khả năng phục hồi lưu trữ: Có quy trình kiểm tra dữ liệu lỗi để khôi phục
 + Phần cứng vật lý: dùng để lưu trữ
 + Bộ điều khiển: cần quản lý tập trung với quy mô lớn
 
#Chapter 2: Swift
- Là project multi-tennant dùng lưu trữ dữ liệu phi cấu trúc như tài liệu, nội dung web, backup...
- Đặc điểm của Swift
 + Khả năng mở rộng: chỉ cần thêm các node lưu trữ
 + Đảm bảo độ bền: có các bản sao dữ liệu đảm bảo độ bền
 + Đa khu vực: Thực hiện với các khu vực địa lý khác nhau.
 + Tính đồng thời: Sử dụng nhiều server 1 lúc
 + Lưu trữ linh hoạt: như vs các dữ liệu cần nhanh thì có thể dùng ổ ssd (dựa vào chính sách lưu trữ)
 + Mã nguồn mở
 + Hệ sinh thái lớn: nhiều tool phát triển xung quan để hỗ trợ.
 + Chạy trên các phần cứng thương mạ: ko cần chuyên dụng đảm bảo chi phí thấp.
 + Thân thiện với nhà phát triển: nhiều thư viện cho các ngôn ngữ khác nhau, nhiều tính năng hỗ trợ như đặt thời gian tồn tại url, dữ liệu....
 
#Chapter 3: Kiến trúc và mô hình dữ liệu Swift
Các thông số liên kết với nhau tạo ra điểm lưu trữ:

- /account: Tài khoản vị trí lưu trữ là 1 tên khu vực lưu trữ duy nhất, nơi sẽ chứa các metadata(thông tin mô tả) về tài khoản này, cũng như nơi chứa tài khoản. Lưu ý trong Swift 1 tài khoản ko phải là 1 danh tính người dùng mà là 1 khu vực lưu trữ.

- /account/container: là vùng lưu trữ được định nghĩa phía trong account nơi mà các metadata về container và danh sách các object mà container đó lưu trữ.

- /account/container/object: vị trí lưu trữ đối tượng là nơi dữ liệu đối tượng và metadata của nó đc lưu trữ.

<img src="http://i.imgur.com/y3Tyy9j.png">

##Kiến trúc lưu trữ:
###Cluster --> Region --> Zone --> Node

- Region: chia thành các vùng vật lý khác nhau. Có thể là single-region hoặc multi-region. Với multi-region dữ liệu được sao ở nhiều điểm khác nhau. Khi lấy dữ liệu sẽ lấy ở điểm có độ trễ thấp nhất (gọi là lựa chọn quen thuộc)
- Zone: chia thành các zone như các trung tâm lưu trữ hay các rack khác nhau để cô lập lỗi
- Node: Server vật lý chạy 1 hoặc nhiều máy chủ xử lý swift.
- Chính sách lưu trữ: được cấu hình để đảm bảo lưu trữ linh hoạt và mạnh mẽ.

<img src="http://i.imgur.com/8A9XBf6.png">

###Về các tiến trình lưu trữ:

- Proxy process: Đảm bảo giao tiếp với client
- Account process: cung cấp metadata của account và danh sách container trong account đó
- Container process: cung cấp metadata của container và danh sách các object trong nó.
- Object process: Lưu trữ, truy xuất, xóa dữ liệu trên các ổ cứng

###Tiến trình thống nhất

Các tiến trình trên để đảm bảo lưu trữ. Còn có các tiến trình để đảm bảo tính thống nhất, khả năng chịu lỗi cho dữ liệu.

- Auditor(account, container, object):  Những auditor account, container, object sẽ liên tục quét trên các ỏ cứng trên các nốt lưu trữ dữ liệu để đảm bảo rằng dữ liệu không bị hệ thống làm hư hỏng. Nếu phát hiện ra dữ liệu bị hư hỏng, các auditor sẽ chuyển dữ liệu đó đến khu vực kiểm tra riêng.
- Repicator (account, container, object): Tiến trình này sẽ đảm bảo đủ các bản copy của dữ liệu mới nhất được lưu trữ trên các cluster. Các replicator account, container, object chạy ngầm với các tiến trình tương ứng. Luôn so sánh với dữ liệu ở 1 node trung tâm để có thể kiểm tra backup.
- Reaper (account): Khi reaper account thấy 1 account bị đánh dấu là xóa, nó sẽ bắt đầu gỡ bỏ hết các liên kết giữa các container và object liên quan đến account đó và sau cùng là lại bỏ bản ghi account đó. Để tránh lỗi, các reaper account sẽ được cấu hình với 1 độ trễ nhất định trước khi nó bắt đầu xóa dữ liệu. 
- Updater (container, object): chịu trách nhiệm giữ các danh sách container tới 1 kỳ hạn. Nó sẽ cập nhật số lượng các object, số lượng container và số byte được sử dụng trong metadata account.
- Expier (object): chỉ định đối tượng tự động xóa sau 1 khoảng thời gian nhằm thanh lọc dữ liệu được chỉ định.

###Định vị dữ liệu:

- Sử dụng hàm băm Rings: dùng md5 băm vị trí đối tượng, rồi chia cho số ổ lấy dư ra ổ lưu trữ
- Hàm băm cải tiến: Băm thông tin ổ (tên ổ, địa chỉ ổ...) Xếp vào 1 vòng trong (vòng rings) hàm bă dữ liệu gần với thông số, trong khoảng nào thì sẽ được lưu trong ổ đó.

<img src="http://i.imgur.com/dUZln7t.png">

- Sửa đổi hàm băm cho phù hợp: Mỗi ổ sẽ có  nhiều khoảng trong vòng rings chứ không chi 1 khoảng. Và các khoảng sẽ được cố định. Được gọi là các partion.
Thông số partion power(?).
 số (partion) trong cluster = 2^(partion power)

<img src="http://i.imgur.com/lgkmL2H.png"> 

Replica count:(Số bản sao) thường là 3
Replica locks: Khóa dữ liệu khi có sự dịch chuyển ổ.

Ngoài sử dụng vòng rings thì còn sử dụng 2 thông số:

- Weight: đánh trọng số cho thiết bị, trọng số càng cao thiết bị càng được nhiều partion.
- Unique as possible: đảm bảo các bản sao được phân bố xa nhau nhất có thể.

###Tạo và cập nhật rings.
- Dựa vào file ring-builder: Các file builder riêng biệt được tạo cho các account, container và tứng chính sách lưu trữ object, nó chứa thông tin như partion power, repica count, replica lock time và vị trí của ổ trong cluster. Swift sử dụng ring-builder để cập nhật các file builder với các thông tin cập nhật  trong ổ đĩa.Lưu ý quan trọng là thông tin trong mỗi file builder này là tách biệt với các file khác. Ví dụ như có thể thêm 1 số ổ đĩa cho account và conatainer nhưng không được cho object
- Cân bằng rings: Khi các builder file được tạo hoặc cập nhật với tất cả thông tin cần thiết, rings sẽ được tạo. Tiện ích rings-builder chạy sử dụng các lệnh rebalance với builder file là tham số đầu vào. Điều này được thực hiện cho mỗi rings: accouunt, container, object. Mỗi khi rings được tạo , nó phải sao chép tới tất cả các node, thay thế cho những  rings cũ.

###Bên trong các rings:
Mỗi lần Swift xây dựng rings, 2 cấu trúc dữ liệu nội bộ quan trọng được tạo ra và truyền vào trong file builder:

- Devices list: Ring-builder chứa danh sách tất cả thiết bị mà chúng ta muốn thêm và file ring-builder. Mỗi ổ bao gồm các thông số ID, zone, trọng số, địa chỉ IP, port và tên thiết bị.

<img src="http://i.imgur.com/9nNh5Pi.png">

- Bảng tra cứu thiết bị: Bảng này chứa 1 hàng cho mỗi bản sao và cột là cho mỗi partion trong cluster, thường là có 3 hàng và hàng ngàn cột. Ring-bui;để tính toán các ổ đĩa tối ưu để đặt mỗi bản sao phân vùng trên ổ đĩa bằng cách sử dụng các trọng số và thuật toán sắp xếp unique-as-posible. Sau đó ghi lại các ổ vào trong bảng

<img src="http://i.imgur.com/TWTHthh.png">