# AD脚本-易河图洛书
通过powershell 批量导出域里的所有windows10或其他类型的操作系统的hostname ，带个上次登录用户和时间的参数。

```powershell
# 获取域中所有计算机的名称
$computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name

# 创建空数组用于存储结果
$results = @()

# 遍历每台计算机并获取所需信息
foreach ($computer in $computers) {
    # 尝试连接到计算机
    try {
        $session = New-PSSession -ComputerName $computer -ErrorAction Stop
        
        # 获取操作系统信息
        $os = Invoke-Command -Session $session -ScriptBlock {
            Get-WmiObject -Class Win32_OperatingSystem | Select-Object -ExpandProperty Caption
        } -ErrorAction Stop
        
        # 获取主机名
        $hostname = Invoke-Command -Session $session -ScriptBlock {
            (Get-WmiObject -Class Win32_ComputerSystem).Name
        } -ErrorAction Stop
        
        # 获取上次登录用户和时间
        $loggedInUser = Invoke-Command -Session $session -ScriptBlock {
            Get-WmiObject -Class Win32_ComputerSystem | Select-Object -ExpandProperty UserName
        } -ErrorAction Stop
        
        $lastLoginTime = Invoke-Command -Session $session -ScriptBlock {
            Get-WmiObject -Class Win32_ComputerSystem | Select-Object -ExpandProperty LocalDateTime
        } -ErrorAction Stop
        
        # 将结果添加到数组中
        $result = [PSCustomObject]@{
            ComputerName = $hostname
            OSVersion = $os
            LoggedInUser = $loggedInUser
            LastLoginTime = $lastLoginTime
        }
        $results += $result
        
        # 关闭会话
        Remove-PSSession -Session $session
    } catch {
        Write-Host "无法连接到计算机: $computer"
    }
}

# 导出结果为 CSV 文件
$results | Export-Csv -Path "C:\path\to\output.csv" -NoTypeInformation

```
