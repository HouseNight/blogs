# Alert

bài này khá dễ vì chỉ cần xem file mp3 ở chế độ Stegogram sẽ ra flag

![Screenshot 2025-08-23 194351](https://hackmd.io/_uploads/H1Env4vKgx.png)

**flag: PTITCTF{3asy_f0r_beginner}**

# Capture

ở bài này ta sẽ có được file capture.pcap

phân tích ta sẽ thấy người dùng tải về khá nhiều file 

![image](https://hackmd.io/_uploads/BkP8ONvFgx.png)

export các file này ra và phân tích thì ta thấy có file rar đáng nghi, nhưng nó đã bị encypt với pass

mình dùng john the ripper để giải mã và được pass: **peanuts**

ở đây có khá nhiều file .txt và nó bị mã hóa base64 thì mình sẽ gộp hết nội dung trong file .txt lại rồi decode base64 

![image](https://hackmd.io/_uploads/ryiMKVwtel.png)

ta sẽ được 1 file .exe, sử dụng Detect it easy thì biết được file này viết bằng C#

trong file này sẽ có khá nhiều hàm nên mình sử dụng ida để phân tích các hàm chính trước và thấy 
![image](https://hackmd.io/_uploads/BJF6KEwFee.png)

ở đây có hàm Flag_TryReveal

dùng DnSpy để tìm thử hàm này nha

![image](https://hackmd.io/_uploads/ryji5NPKel.png)

ở đây ta đã thấy hàm encrypt flag, giờ chỉ cần tìm dữ liệu

nó sẽ nằm ở hàm .cctor

```
// Embeds
// Token: 0x0600000C RID: 12 RVA: 0x000021D0 File Offset: 0x000003D0
// Note: this type is marked as 'beforefieldinit'.
static Embeds()
{
	Embeds.AesNonce = new byte[]
	{
		236, 85, 150, 249, 133, 223, 22, 97, 218, 211,
		38, 76
	};
	Embeds.AesTag = new byte[]
	{
		56, 74, 98, 242, 57, 243, 115, 204, 222, 253,
		56, 232, 197, 107, 14, 225
	};
	Embeds.AesCiphertext = new byte[]
	{
		174, 48, 7, 100, 207, 26, 27, 150, 166, 144,
		90, 153, 225, 176, 222, 113, 164, 197, 167, 77,
		133, 132, 235, 43, 43, 115, 86, 82, 85, 184,
		28, 28, 219, 201, 31, 202, 70, 19, 137, 96,
		159, 89, 137, 51, 168, 115
	};
	Embeds.KeyShard1 = new byte[]
	{
		121, 165, 33, 10, 18, 197, 254, 212, 240, 253,
		79, 245, 53, 48, 123, 46, 142, 215, 38, 213,
		25, 168, 2, 224, 53, 25, 9, 191, 221, 152,
		199, 246
	};
	Embeds.KeyShard2 = new byte[]
	{
		63, 201, 64, 109, 80, 176, 151, 184, 148, 152,
		61, 183, 76, 120, 26, 71, 224, 179, 22, 230,
		56, 137, 35, 193, 20, 56, 40, 158, 252, 185,
		230, 215
	};
}

```
 
 từ đây viết code giải mã: 
```
from Crypto.Cipher import AES

# dữ liệu từ Embeds
KeyShard1 = bytes([
    121, 165, 33, 10, 18, 197, 254, 212, 240, 253,
    79, 245, 53, 48, 123, 46, 142, 215, 38, 213,
    25, 168, 2, 224, 53, 25, 9, 191, 221, 152,
    199, 246
])

KeyShard2 = bytes([
    63, 201, 64, 109, 80, 176, 151, 184, 148, 152,
    61, 183, 76, 120, 26, 71, 224, 179, 22, 230,
    56, 137, 35, 193, 20, 56, 40, 158, 252, 185,
    230, 215
])

AesNonce = bytes([
    236, 85, 150, 249, 133, 223, 22, 97, 218, 211,
    38, 76
])

AesTag = bytes([
    56, 74, 98, 242, 57, 243, 115, 204, 222, 253,
    56, 232, 197, 107, 14, 225
])

AesCiphertext = bytes([
    174, 48, 7, 100, 207, 26, 27, 150, 166, 144,
    90, 153, 225, 176, 222, 113, 164, 197, 167, 77,
    133, 132, 235, 43, 43, 115, 86, 82, 85, 184,
    28, 28, 219, 201, 31, 202, 70, 19, 137, 96,
    159, 89, 137, 51, 168, 115
])

# tái tạo secretKey = XOR hai shard
secretKey = bytes([a ^ b for a, b in zip(KeyShard1, KeyShard2)])

# giải AES-GCM
cipher = AES.new(secretKey, AES.MODE_GCM, nonce=AesNonce)
plaintext = cipher.decrypt_and_verify(AesCiphertext, AesTag)

print(plaintext.decode("utf-8"))
```

**flag: PTITCTF{dotn3t_c4nn0t_mak4it_difficult_f0ry0u}**


# Virtual Image 1

ở bài này ta sẽ nhận được 1 file .ad1

phân tích nó thì thấy 
trong đường dẫn:

C:\Users\Hacker\AppData\Roaming\Microsoft\Windows\svhost.exe

![image](https://hackmd.io/_uploads/r1bhiNDFxg.png)

extract nó ra rồi quăng lên **Virustotal** 

![image](https://hackmd.io/_uploads/HyJ9nEPYgl.png)

từ đây sẽ có được: 

**md5: 0f266b60a2bd1818d752a3b6f3098b1c**

**Creation Time:    2025-07-23 02:56:24 UTC**

còn **Threat label: sẽ là malware, trojan **

thử 2 cái sẽ có flag: **PTITCTF{C:\Users\Hacker\AppData\Roaming\Microsoft\Windows\svhost.exe_0f266b60a2bd1818d752a3b6f3098b1c_2025-07-23T02:56:24Z_trojan}**

# Virtual Image 2

phân tích file svhost.exe thì nó bị packed bằng upx 

unpack nó ra rồi view bằng ida, ở đây sẽ bị guessy vì nó có nhiều hàm nên mình chuyển qua string xem cho nhanh

![image](https://hackmd.io/_uploads/ryU4C4vFlg.png)

thấy có khá nhiều PTITCTF

cùng xem hàm này như nào, 

![image](https://hackmd.io/_uploads/H1Bw0EPKeg.png)

ta thấy 4 hàm này sẽ là 4 hàm chính của file 

ta sẽ phải nhập password vào rồi

xor với biến byte_1401436A0 nếu i chẵn sẽ xor với (v7 - 1), lẻ thì (v7 + 1) xong sẽ check xem đúng header của file .png rồi xuất ra.

ở đây mình đã biết được data của biến byte_1401436A0. 

vì đã biết header của ảnh nên chỉ cần dùng các dòng đầu của biến byte_1401436A0 nên xor theo công thức sẽ được pass là: **NoCurrentThreats**

từ đây viết code giải mã thoi: 

```
HEX = """<dán toàn bộ chuỗi hex biến byte_1401436A0 vào đây>"""

key = b"NoCurrentThreats"
arr = bytes.fromhex(HEX)
out = bytearray(len(arr))

for i, b in enumerate(arr):
    k = key[i % len(key)]
    if i % 2 == 0:           # i chẵn
        out[i] = b ^ ((k + 1) & 0xFF)
    else:                    # i lẻ
        out[i] = b ^ ((k - 1) & 0xFF)

# ghi file
with open("whatisthis.png", "wb") as f:
    f.write(out)

print("Done -> whatisthis.png")

```

ta sẽ được ảnh whatisthis.png![whatisthis](https://hackmd.io/_uploads/H1FLlHDFeg.png)

nhưng đây chưa phải flag, mình dùng zsteg để tìm ra flag cuối cùng 

**PTITCTF{L3ast_S1gn1f1cant_B1t_M4g1c_1n_3x3}**


# Memory

ở đây mình sẽ có file memory.raw sử dụng vol để đọc thì mình thấy có 
![image](https://hackmd.io/_uploads/r1nVbBvYxe.png)

ta thấy mã độc tấn công vào keepass nên mình dùng plugins memmap để dump pid 2820

 và view bằng notepad++

![image](https://hackmd.io/_uploads/SJ0dMBPtxl.png)

thấy có aes data nên mình decode thử


![Screenshot 2025-08-23 103245](https://hackmd.io/_uploads/BJj9zBDtgg.png)

có được pass và sử dụng filescan để xem thì mình thấy có file database.kdbx

![image](https://hackmd.io/_uploads/H1sz7rwtel.png)

dump file này ra thì ta lại thấy có pass khác

![image](https://hackmd.io/_uploads/ry_LmBPKlg.png)
 
mình có thử submit nhưng không đúng và sau khi tìm kiếm thì mình thấy có real.txt 

![image](https://hackmd.io/_uploads/SJ8JVHvKel.png)

dump file này ra và ta thấy file này là file docx bị cài pass nên sử dụng pass: **NowY0uC4nF1ndM3!**

để giải mã và ta được flag

**PTITCTF{M3m0ry_Dumppppppppp!}**

# Summoner 2

bài này mình cũng tình cờ tìm được flag vì sau khi mở file bằng vmware thì mình có đọc event log nhưng không thấy gì 

nên mình thử strings các file: 
Windows 7 x64.vmdk, Windows 7 x64-Snapshot6.vmem

và bingo
![Screenshot 2025-08-23 162111](https://hackmd.io/_uploads/rJe3VBPKgx.png)

**flag: PTITCTF{W1P3R!!!_S3nd_h3lp_P1c45ss!!!}**
