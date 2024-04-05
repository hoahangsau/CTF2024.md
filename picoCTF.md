#**Interencdec(Cryptography)**
Mở file enc_flag ra thì mình thấy một đoạn mã Base64 nên mình đã decode bằng CyberChef.
![image](https://github.com/hoahangsau/CTF2024.md/assets/153940762/f82c7190-be93-4a60-b26a-d956eba78f19)

![image](https://github.com/hoahangsau/CTF2024.md/assets/153940762/ef258e10-7e6e-4503-b953-d3ffa5a858f9)

Sau khi decode lần 2 FromBase64 thì mình thấy một đoạn mã giống format flag, nên mình đoán ngay đây là Ceasar cipher
![image](https://github.com/hoahangsau/CTF2024.md/assets/153940762/5e6bc33a-8b01-43c8-9dbc-df0ea4d6821d)

#**Custom encryption**
Dưới đây là đoạn code được cho để encrypt ciphertext 
<pre>
from random import randint
import sys


def generator(g, x, p):
    return pow(g, x) % p


def encrypt(plaintext, key):
    cipher = []
    for char in plaintext:
        cipher.append(((ord(char) * key*311)))
    return cipher


def is_prime(p):
    v = 0
    for i in range(2, p + 1):
        if p % i == 0:
            v = v + 1
    if v > 1:
        return False
    else:
        return True


def dynamic_xor_encrypt(plaintext, text_key):
    cipher_text = ""
    key_length = len(text_key)
    for i, char in enumerate(plaintext[::-1]):
        key_char = text_key[i % key_length]
        encrypted_char = chr(ord(char) ^ ord(key_char))
        cipher_text += encrypted_char
    return cipher_text


def test(plain_text, text_key):
    p = 97
    g = 31
    if not is_prime(p) and not is_prime(g):
        print("Enter prime numbers")
        return
    a = randint(p-10, p)
    b = randint(g-10, g)
    print(f"a = {a}")
    print(f"b = {b}")
    u = generator(g, a, p)
    v = generator(g, b, p)
    key = generator(v, a, p)
    b_key = generator(u, b, p)
    shared_key = None
    if key == b_key:
        shared_key = key
    else:
        print("Invalid key")
        return
    semi_cipher = dynamic_xor_encrypt(plain_text, text_key)
    cipher = encrypt(semi_cipher, shared_key)
    print(f'cipher is: {cipher}')


if __name__ == "__main__":
    message = sys.argv[1]
    test(message, "trudeau")
</pre>
Các function chúng ta cần để ý đó là **generator()**, **encrypt()**. **dynamic_xor_encrypt()**.

Function **generator()** trả về giá trị _pow(g,x)%p_, hàm này sẽ được sử dụng để tạo key để encrypt trong hàm **test**.

Function **encrypt()** encrypt plaintext bằng cách nhân mã Unicode của từng ký tự với key với 311.

Function **dynamic_xor_encrypt()** thực hiện phép XOR giữa từng ký tự trong _plaintext[::-1]_ với từng ký tự trong _text_key_ rồi trả về cipher_text.

Dựa vào các hàm đã có ở trên, mình viết được đoạn script để lấy flag:
<pre>
def generator(g, x, p):
    return pow(g, x) % p

def decrypt(ciphertext, key):
    decrypt_message= ""
    for cipher_value in ciphertext:
        decrypted_char = chr(cipher_value // (key * 311))
        decrypt_message += decrypted_char  
    return decrypt_message[::-1]

def dynamic_xor_decrypt(cipher_text, text_key):
    decrypted_text = ""
    key_length = len(text_key)
    for i, char in enumerate(cipher_text[::-1]):
        key_char = text_key[i % key_length]
        decrypted_char = chr(ord(char) ^ ord(key_char))
        decrypted_text += decrypted_char
    return decrypted_text[::-1] 


if __name__ == "__main__":
    ciphertext = [151146, 1158786, 1276344, 1360314, 1427490, 1377108, 1074816, 1074816, 386262, 705348, 0, 1393902, 352674, 83970, 1141992, 0, 369468, 1444284, 16794, 1041228, 403056, 453438, 100764, 100764, 285498, 100764, 436644, 856494, 537408, 822906, 436644, 117558, 201528, 285498]
    a = 97
    b = 22
    u = generator(31, a, 97)
    v = generator(31, b, 97)
    key = generator(v, a, 97)
    b_key = generator(u, b, 97)
    if key == b_key:
        shared_key = key
    else:
        print("Invalid key")    
    
    message = decrypt(ciphertext, shared_key)
    decrypted = dynamic_xor_decrypt(message, "trudeau")
    print(decrypted)
</pre>

Function **decrypt()** hoạt động ngược lại với function **encrypt()** bằng cách chia giá trị unicode của từng ký tự trong ciphertext với _(key * 311)_ rồi trả về decrypt_message.

Function **dynamic_xor_decrypt()** trả về decrypted_text bằng cách XOR ngược lại các ký tự trong ciphertext với text_key.

FLAG: picoCTF{custom_d2cr0pt6d_e4530597}

#**C3**
Đoạn code encrypt được cho như sau
<pre>
import sys
chars = ""
from fileinput import input
for line in input():
  chars += line

lookup1 = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
lookup2 = "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"

out = ""
prev = 0
for char in chars:
  cur = lookup1.index(char)
  out += lookup2[(cur - prev) % 40]
  prev = cur

sys.stdout.write(out)
</pre> 

Để decrypt thì ta cần phải lấy ký tự từ lookup2.index() sau đó tìm ngược index của lookup1 bằng cách _(cur + prev) % 40_
<pre>

lookup1 = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
lookup2 = "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"
cipher = "DLSeGAGDgBNJDQJDCFSFnRBIDjgHoDFCFtHDgJpiHtGDmMAQFnRBJKkBAsTMrsPSDDnEFCFtIbEDtDCIbFCFtHTJDKerFldbFObFCFtLBFkBAAAPFnRBJGEkerFlcPgKkImHnIlATJDKbTbFOkdNnsgbnJRMFnRBNAFkBAAAbrcbTKAkOgFpOgFpOpkBAAAAAAAiClFGIPFnRBaKliCgClFGtIBAAAAAAAOgGEkImHnIl"
back = ""
prev = 0

for char in cipher:
    cur = lookup2.index(char)
    back += lookup1[(cur + prev) % 40]
    prev = (cur + prev) % 40

print(back)

</pre>

Output của script trên sẽ là một đoạn code khác:
<pre>
#asciiorder
#fortychars
#selfinput
#pythontwo

chars = ""
from fileinput import input
for line in input():
    chars += line
b = 1 / 1

for i in range(len(chars)):
    if i == b * b * b:
        print chars[i] #prints
        b += 1 / 1
</pre>

Đọc các comment thì mình thấy có hint *selfinput , nên mình đã lưu đoạn code trên với tên input.txt để input vào chính đoạn code đó.
<pre>
import fileinput

chars = ""
with fileinput.input(files=('input.txt')) as file:
    for line in file:
        chars += line
    b = 1 / 1

    for i in range(len(chars)):
        if i == b * b * b:
            print(chars[i])  
            b += 1 / 1
</pre>

Chạy đoạn script trên ta sẽ có được FLAG: picoCTF{adlibs}




