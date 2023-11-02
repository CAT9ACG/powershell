# AD脚本-易河图洛书
通过powershell 批量导出域里的所有windows10或其他类型的操作系统的hostname ，带个上次登录用户和时间的参数。

```powershell
# 获取域中所有计算机的主机名
$computers = Get-ADComputer -Filter * | Where-Object { $_.OperatingSystem -like "Windows 10*" -or $_.OperatingSystem -like "Windows 7*" -or $_.OperatingSystem -like "Windows 8*" }

# 准备一个空数组以存储计算机信息
$computerInfo = @()

# 遍历每台计算机并获取相关信息
foreach ($computer in $computers) {
    $hostname = $computer.Name  # 获取计算机主机名
    $lastLogon = (Get-ADComputer $computer | Get-ADObject -Properties "LastLogon").LastLogon  # 获取上次登录时间
    $lastLogonDate = [DateTime]::FromFileTime($lastLogon)  # 将时间戳转换为日期时间格式
    $lastLogonUser = (Get-WmiObject -Class Win32_ComputerSystem -ComputerName $computer.Name).UserName  # 获取上次登录用户

    # 创建一个包含计算机信息的自定义对象，并将其添加到数组中
    $computerInfo += New-Object PSObject -property @{
        "Hostname" = $hostname
        "LastLogonUser" = $lastLogonUser
        "LastLogonTime" = $lastLogonDate
    }
}

# 输出结果到CSV文件
$computerInfo | Export-Csv -Path "ComputerInfo.csv" -NoTypeInformation

```
