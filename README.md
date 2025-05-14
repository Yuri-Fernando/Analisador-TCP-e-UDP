# Analisador-TCP-e-UDP
Analisa a rede e descrimina todos os serviços e ips associados

Coleta de Conexões TCP e UDP via PowerShell:
O projeto em Python foi inicialmente estruturado com base na coleta de dados obtidos por esses dois comandos em PowerShell — um para conexões TCP e outro para conexões UDP. Esses comandos têm como objetivo mapear, de forma detalhada, as conexões de rede ativas, identificando portas locais/remotas, processos responsáveis e, quando aplicável, serviços associados ao processo svchost.


TCP - Completo:

Get-NetTCPConnection | ForEach-Object {
    $proc = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
    $services = if ($proc -and $proc.ProcessName -eq 'svchost') {
        # Coleta os serviços associados ao processo svchost
        Get-WmiObject -Class Win32_Service | Where-Object { $_.ProcessId -eq $proc.Id } | Select-Object -ExpandProperty Name
    }
    [PSCustomObject]@{
        LocalAddress  = $_.LocalAddress
        LocalPort     = $_.LocalPort
        RemoteAddress = $_.RemoteAddress
        RemotePort    = $_.RemotePort
        ProcessId     = $_.OwningProcess
        ProcessName   = if ($proc) { $proc.ProcessName } else { "N/A" }
        Services      = if ($services) { $services -join ", " } else { "Nenhum serviço associado" }
        State         = $_.State
    }
} | Sort-Object State, ProcessName | Format-Table -Property LocalAddress, LocalPort, RemoteAddress, RemotePort, ProcessId, ProcessName, Services, State -AutoSize


UDP - Completo:

Get-NetUDPEndpoint | ForEach-Object {
    $p = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
    $services = if ($p -and $p.ProcessName -eq 'svchost') {
        # Coleta os serviços associados ao processo svchost
        Get-WmiObject -Class Win32_Service | Where-Object { $_.ProcessId -eq $p.Id } | Select-Object -ExpandProperty Name
    }
    [PSCustomObject]@{
        LocalAddress   = $_.LocalAddress
        LocalPort      = $_.LocalPort
        RemoteAddress  = $_.RemoteAddress
        RemotePort     = $_.RemotePort
        ProcessId      = $_.OwningProcess
        ProcessName    = if ($p) { $p.ProcessName } else { "Desconhecido/Finalizado" }
        Services       = if ($services) { $services -join ", " } else { "Nenhum serviço associado" }
    }
} | Sort-Object LocalPort | Format-Table -AutoSize
