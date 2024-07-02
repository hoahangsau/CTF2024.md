<h1>
    X Marked the Spot
</h1>
<h2>
    Problem
</h2>
<pre>
from itertools import cycle

flag = b"uiuctf{????????????????????????????????????????}"
\# len(flag) = 48
key  = b"????????"
\# len(key) = 8
ct = bytes(x ^ y for x, y in zip(flag, cycle(key)))

with open("ct", "wb") as ct_file:
    ct_file.write(ct)</pre>
<h2>
    Solution
</h2>

Ta đã biết được format flag là "uiuctf{...}" nên hoàn toàn có thể biết được 7 bytes đầu của xorkey là "uiuctf{". Để tìm ra byte còn lại thì ta có thể xor ký tự cuối của ciphertext với dấu "}" của flag. => xorkey: uiuctf{n

<pre>from itertools import cycle

def find_key(ct, known_start):
    key_length = 8
    key = bytearray(key_length)

    for i in range(len(known_start)):
        key[i % key_length] = ct[i] ^ known_start[i]

    return key

def decrypt(ct, key):
    decrypted = bytes(x ^ y for x, y in zip(ct, cycle(key)))
    return decrypted

def main():
    with open("ct", "rb") as ct_file:
        ct = ct_file.read()   
    known_start = b"uiuctf{n"   
    key = find_key(ct, known_start)
    decrypted_flag = decrypt(ct, key)   
    print(decrypted_flag.decode())

if __name__ == "__main__":
    main()
</pre>

=>FLAG: uiuctf{n0t_ju5t_th3_st4rt_but_4l50_th3_3nd!!!!!}
