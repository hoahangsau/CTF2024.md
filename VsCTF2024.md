<h1>
    Not quite ceasar
</h1>
<h2>
    Problem
</h2>
<Pre>
import random
random.seed(1337)
ops = [
    lambda x: x+3,
    lambda x: x-3,
    lambda x: x*3,
    lambda x: x^3,
]


flag = list(open("flag.txt", "rb").read())
out = []
for v in flag:
    out.append(random.choice(ops)(v))
print(out)</Pre>

Output:
<pre>[354, 112, 297, 119, 306, 369, 111, 108, 333, 110, 112, 92, 111, 315, 104, 102, 285, 102, 303, 100, 112, 94, 111, 285, 97, 351, 113, 98, 108, 118, 109, 119, 98, 94, 51, 56, 159, 50, 53, 153, 100, 144, 98, 51, 53, 303, 99, 52, 49, 128]
</pre>

<h2>
    Solution
</h2>

Để encrypt flag, bài đã sử dụng các phép toán +,-,* và ^ áp dụng ngẫu nhiên các phép toán này lên từng ký tự ascii của flag. 
Nhưng ở đây chúng ta thấy đề sử dụng seed _random.seed(1337)_, tức là các giá trị ngẫu nhiên hoàn toàn lặp lại. Vì vậy để decrypt, ta chỉ cần đảo ngược lại các phép toán.

<pre>
import random

random.seed(1337)

encrypted_output = [354, 112, 297, 119, 306, 369, 111, 108, 333, 110, 112, 92, 111, 315, 104, 102, 285, 102, 303, 100, 112, 94, 111, 285, 97, 351, 113, 98, 108, 118, 109, 119, 98, 94, 51, 56, 159, 50, 53, 153, 100, 144, 98, 51, 53, 303, 99, 52, 49, 128]


reverse_ops = [
    lambda x: x - 3,
    lambda x: x + 3,
    lambda x: x // 3,
    lambda x: x ^ 3,
]

decrypted_output = []
for value in encrypted_output:
    chosen_op = random.choice(reverse_ops)
    original_value = chosen_op(value) % 256
    decrypted_output.append(original_value)

decrypted_flag = ''.join(map(chr, decrypted_output))
print(decrypted_flag)</pre>

=> FLAG: vsctf{looks_like_ceasar_but_isnt_a655563a0a62ef74}
