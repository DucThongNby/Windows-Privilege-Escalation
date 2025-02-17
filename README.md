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
