### Giao thức VXLAN
Ngày nay VXLAN đã trở thành một giao thức chồng lấn phổ biến nhất. VXLAN hỗ trợ rất nhiều nhà cung cấp, bao gồm Cisco, VMWare.. cùng với một thực tế là nó chạy qua mạng IP core, điều này làm cho VXLAN trở thành một sự lựa chọn phổ biến cho việc triển khai trong các trung tâm dữ liệu quy mô lớn.

VXLAN sử dụng các thiết bị kết cuối đường hầm VXLAN – VTEP để ánh xạ các đầu cuối vào các phân đoạn mạng. Chúng ta có thể hình dung VTEP có hai giao diện – Một kết nối với mạng IP ở giữa và giao diện còn lại kết nối tới phân đoạn mạng nội bộ – Được hỗ trợ đóng gói VXLAN.

Mô hình triển khai VXLAN
<img src="/img/2.jpg">


Hiện nay, rất nhiều thiết bị chuyển mạch ảo đã hỗ trợ đóng gói VXLAN. Các khách hàng trung tâm dữ liệu đã có thể nhanh chóng đưa ra các ứng dụng ảo với các dịch vụ tích hợp bằng cách triển khai mô hình VXLAN tới tận các host là các máy chủ ảo.

Trong vài năm qua đã có sự gia tăng đáng kinh ngạc trong việc phổ biến VXLAN. Định dạng chồng lấn MAC dựa trên IP/UDP cùng với khả năng hỗ trợ lên tới 16 triệu phân đoạn mạng ảo đã làm cho VXLAN trở thành sự lựa chọn hàng đầu cho việc triển khai trung tâm dữ liệu đa người dùng.

Trong một phát biểu trên trang cá nhân, Ivan Pepelnjak – Chuyên gia trong lĩnh vực SDN, NFV và là tác giả của nhiều đầu sách về hệ thống mạng trung tâm dữ liệu đã có một số  nhận định sau về tương lai của các công nghệ chồng lấn:

Tất cả các data centers đều sử dụng VLAN để cô lập dữ liệu ở layer 2. Khi data centers cần mở rộng mạng Layer2 qua data center hoặc có thể bên ngoài data center thì ta thấy những thiết sót của LAN là hiển nhiên. Những thiếu sót này là:

* Trong data center, yêu cầu hàng nghìn VLAN để cô lập traffic môi trường multi-tenant chia sẻ như cơ sở hạ tầng L2/L3 cho Cloud Service Provider. Hiện tại, giới hạn 4096 VLANs là ko đủ.

* Do ảo hóa server, mỗi Virtual Machine (VM) yêu cầu 1 địa chỉ MAC duy nhất và 1 địa chỉ IP duy nhất. Vì vậy, có hàng ngàn bảng MAC trên các thiết bị switches. Điều này đặt ra nhu cầu lớn hơn về công suất của switches.

* VLANs quá hạn chế về khoảng cách và triển khai. VTP có thể được sử dụng để triển khai các VLAN qua các L2 switchese nhưng hầu hết mọi người muốn tắt VTP vì tính tác hại của nó.

* Sử dụng STP để cung cấp L2 loop topology vô hiệu hóa hầu các liên kết sự phòng. Do đó, chi phí Equal-Cost Multi-Path (ECMP) tất khó đạt được. Tuy nhiên, ECMP rất dễ để đạt được trong mạng IP.

Rõ ràng VXLAN đã thể hiện ưu thế vượt trội so với các kỹ thuật còn lại, một thực tế là tất cả các nhà cung cấp thiết bị, giải pháp cung cấp kết nối cho hạ tầng máy chủ ảo đều lựa chọn VXLAN như một giao thức đóng gói ưu tiên hàng đầu. Điều này hoàn toàn phù hợp với xu hướng phát triển trong một thế giới phẳng, công nghệ thay đổi hằng ngày.

### VXLAN - Virtual eXtensible LAN

- VXLAN là giao thức tunneling, thuộc giữa lớp 2 và lớp 3.

- Địa chỉ VXLAN giải quyết các thách thức ở trên. Công nghệ VXLAN cung cấp các dịch vụ kết nối các Ethernet end systems và cung cấp phương tiện mở rộng mạng LAN qua mạng L3. VXLAN ID (VXLAN Network Identifier hoặc VNI) là 1 chuỗi 24-bits so với 12 bits của của VLAN ID. Do đó cung cấp hơn 16 triệu ID duy nhất.

- VXLAN Tunnel End Point (VTEP) dùng để kết nối switch (hiện tại là virtual switch) đến mạng IP. VTEP nằm trong hypervisor chứa VMs. Chức năng của VTEP là đóng gói VM traffic trong IP header để gửi qua mạng IP.

<img src="/img/3.png">

* VM-to-VM communication : Unicast traffic VM trong server bên trái trong hình trên gửi traffic đến VM trong server bên phải. Dựa trên cấu hình trong Bridge Domain, VM traffic được chỉ định VNI. VTEP xác định xem đích của VM có trên cùng 1 segment hay không? VTEP đóng gói Ethernet frame với outer MAC header, outer IP header và VXLAN header. Gói tin đầy đủ được gửi đến mạng IP với địa chỉ IP đích của remote VTEP được kết nối với VM đích. Remote VTEP decapsulates packet và forward frame đến VM được kết nối. Remote VTEP cũng học địa chỉ inner Source MAC và địa chỉ outer Source IP .

* VM-to-VM communication : Broadcast và Unknown Unicast traffic VM trong server bên trái muốn truyền thông với VM trong server bên phải trong cùng 1 subnet. VM sử gửi gói tin ARP Broadcast, UDP header và VXLAN header. Tuy nhiên, gói tin này được gửi đến IP Multicast group mà liên kết với VXLAN ID. VTEP sẽ gửi đi gói tin IGMP Membership Report tới upstream router để kết nối/ rời IP multicast groups liên quan đến VXLAN. Remote VTEP là bộ thộ thu cho IP multicast groups đó, nhận được traffic từ VTEP nguồn. 1 lần nữa, nó tạp ra 1 mapping của địa chỉ inner Source MAC và địa chỉ outer Source IP, và chuyển tiếp lưu lượng truy cập đến VM đích. VM đích gửi ARP response tiêu chuẩn sử dụng IP unicast. VTEP đóng gói gói tin trở lại cho VTEP kết nối VM bằng cách sử dụng IP unicast VXLAN encapsulation.

### VXLAN packet format

### Cấu trúc gói tin VXLAN Encapsulation

- Ngoài IP header và VXLAN header, VTEP cũng chèn thêm UDP header. Trong ECMP, switch/router bao gồm UDP header để thực hiện chức năng băm. VTEP tính source port bằng cách thực hiện băm inner Ethernet frame của header. Destination UDP port là VXLAN port.

- Outer IP header chứa địa chỉ Source IP của VTEP thực hiện việc encapsulation. Địa chỉ IP đích là địa chỉ IP remote VTEP hoặc địa chỉ IP Multicast group. VXLAN đôi khi còn được gọi là công nghệ MAC-in-IP-encapsulation.

- VXLAN thêm 50 bytes. Để tránh phân mảnh và tái lắp ráp, tất cả các thiết bị mạng vật lý vẫn chuyển lưu lượng VXLAN phải chứa được gói tin này. Vì vậy, MTU cũng nên được điều chỉnh tương ứng.

<img src="/img/4.png">

<img src="/img/5.png">

### VXLAN Header

- VXLAN header có 8 byte. Sau đây là cấu trúc của VXLAN header:

<img src="/img/6.png">

### Mô hình LAB

<img src="/img/2.jpg">










