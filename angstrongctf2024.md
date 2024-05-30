<h1>
    erm what the enigma
</h1>

<h2>
    Problem
</h2>

In the dimly lit, smoke-filled war room, the Enigma M3 machine sat at the center of the table like a cryptographic sentinel. Its reflector, UKW B, gleamed ominously in the low light. The three rotors, I, II, and III, stood proudly in their respective slots, each at position 1 with their rings set to 1. The plugboard, conspicuously vacant, indicated a reliance on the rotor settings alone to encode the day's crucial messages. As the operator's fingers hovered over the keys, the silence was heavy with anticipation. Each clack of the keys transformed plaintext into an unbreakable cipher, the resulting ciphertext, brht{d_imhw_cexhrmwyy_lbvkvqcf_ldcz}, a guardian of wartime secrets.
<h2>
    Solution
</h2>

Dựa vào đề bài ta có thể set up tool Enigma Decoder như hình
![Screenshot 2024-05-30 072430](https://github.com/hoahangsau/CTF2024.md/assets/153940762/abb37883-257b-422f-b1bd-6082b3182baa)

 => flag: actf{i_love_enigmatic_machines_mwah}
 

<h1>
  PHIlosophy  
</h1>
<h2>
    Problem
</h2>

from Crypto.Util.number import getPrime
from secret import flag

p = getPrime(512)
q = getPrime(512)

m = int.from_bytes(flag.encode(), "big")

n = p * q
e = 65537
c = pow(m, e, n)

phi = (p + 1) * (q + 1)

print(f"n: {n}")
print(f"e: {e}")
print(f"c: {c}")
print(f"\"phi\": {phi}")

"""
n: 86088719452932625928188797700212036385645851492281481088289877829109110203124545852827976798704364393182426900932380436551569867036871171400190786913084554536903236375579771401257801115918586590639686117179685431627540567894983403579070366895343181435791515535593260495162656111028487919107927692512155290673
e: 65537
c: 64457111821105649174362298452450091137161142479679349324820456191542295609033025036769398863050668733308827861582321665479620448998471034645792165920115009947792955402994892700435507896792829140545387740663865218579313148804819896796193817727423074201660305082597780007494535370991899386707740199516316196758
"phi": 86088719452932625928188797700212036385645851492281481088289877829109110203124545852827976798704364393182426900932380436551569867036871171400190786913084573410416063246853198167436938724585247461433706053188624379514833802770205501907568228388536548010385588837258085711058519777393945044905741975952241886308
"""
<h2>
    Solution
</h2>

Vì phi = (p+1)*(q+1), và n=p*q nên ta có hệ phương trình sau


<pre>
p + q = phi - n - 1 = 18873512826871273426766179137608666660870794019936008938947887293234875222098328497861493193366574594073301664825215895863666365457125797814283440086595634
p * q = 86088719452932625928188797700212036385645851492281481088289877829109110203124545852827976798704364393182426900932380436551569867036871171400190786913084554536903236375579771401257801115918586590639686117179685431627540567894983403579070366895343181435791515535593260495162656111028487919107927692512155290673</pre>

Từ hệ phương trình trên ta có thể dễ dàng giải ra được giá trị của p và q, từ đó decrypt RSA thông thường

<Pre>
from Crypto.Util.number import long_to_bytes
from sympy import mod_inverse

n = 86088719452932625928188797700212036385645851492281481088289877829109110203124545852827976798704364393182426900932380436551569867036871171400190786913084554536903236375579771401257801115918586590639686117179685431627540567894983403579070366895343181435791515535593260495162656111028487919107927692512155290673
e = 65537
c = 64457111821105649174362298452450091137161142479679349324820456191542295609033025036769398863050668733308827861582321665479620448998471034645792165920115009947792955402994892700435507896792829140545387740663865218579313148804819896796193817727423074201660305082597780007494535370991899386707740199516316196758
p = 11158282525228889688006126870044047515722834736905353600033800784672966071954095221182355999939654085477427093464088811910784573274296445489954662335305921
q = 7715230301642383738760052267564619145147959283030655338914086508561909150144233276679137193426920508595874571361127083952881792182829352324328777751289713
phi = (p - 1) * (q - 1)
d = mod_inverse(e, phi)
m = pow(c, d, n)
flag = long_to_bytes(m)
print(flag)</Pre>

=> flag: actf{its_okay_i_figured_out_phi_anyway}

<h1>
    Layers
</h1>

<h2>
    Problem
</h2>
<pre>
import hashlib
import itertools
import os

def xor(key, data):
    return bytes([k ^ d for k, d in zip(itertools.cycle(key), data)])

def encrypt(phrase, message, iters=1000):
    key = phrase.encode()
    for _ in range(iters):
        key = hashlib.md5(key).digest()
        message = xor(key, message)
    return message

print('Welcome to my encryption service!')
print('Surely encrypting multiple time1s will make it more secure.')
print('1. Encrypt message.')
print('2. Encrypt (hex) message.')
print('3. See encrypted flag!')

phrase = os.environ.get('FLAG', 'missing')

choice = input('Pick 1, 2, or 3 > ')
if choice == '1':
    message = input('Your message > ').encode()
    encrypted = encrypt(phrase, message)
    print(encrypted.hex())
if choice == '2':
    message = bytes.fromhex(input('Your message > '))
    encrypted = encrypt(phrase, message)
    print(encrypted.hex())
elif choice == '3':
    print(encrypt(phrase, phrase.encode()).hex())
else:
    print('Not sure what that means.')
</pre>

<h2>
    Solution
</h2>

Vì encrypted_flag đã được mã hóa bằng cách XOR 1000 lần message với key(phrase), ta chỉ cần XOR ngược lại 1000 lần encrypted_flag với message là sẽ ra được phrase mà chúng ta cần tìm.

<pre>
from pwn import *
import hashlib
import itertools

def xor(key, data):
    return bytes([k ^ d for k, d in zip(itertools.cycle(key), data)])

def encrypt(phrase, message, iters=1000):
    key = phrase.encode()
    for _ in range(iters):
        key = hashlib.md5(key).digest()
        message = xor(key, message)
    return message

def get_encrypted_flag():
    conn = remote('challs.actf.co', 31398)
    conn.recvuntil(b'> ')
    conn.sendline(b'3')
    encrypted_flag_hex = conn.recvline().strip().decode()
    conn.close()
    return encrypted_flag_hex

def decrypt(encrypted_flag_hex):
    conn = remote('challs.actf.co', 31398)
    conn.recvuntil(b'> ')
    conn.sendline(b'2')
    conn.recvuntil(b'> ')
    conn.sendline(encrypted_flag_hex.encode())
    decrypted_output_hex = conn.recvline().strip().decode()
    conn.close()
    return decrypted_output_hex

def main():
    encrypted_flag_hex = get_encrypted_flag()
    decrypted_output_hex = decrypt(encrypted_flag_hex)
    print(bytes.fromhex(decrypted_output_hex))



if __name__ == '__main__':
    main()

</pre>

=> flag: actf{593a7043ca58fcac7ec972e3dcf01263}
