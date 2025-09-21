# Potion Master

## Recon

Haskell file.

## Analysis

```hs
chunks :: Int -> [a] -> [[a]]
chunks n l
  | n == 0 = []
  | n == 1 = [[x] | x <- l]
  | length l <= n = [l]
  | otherwise = [take n l] ++ (chunks n (drop n l))

a = [-43, 61, 58, 5, -4, -11, 64, -40, -43, 61, 62, -51, 46, 15, -49, -44, 47, 4, 6, -7, 47, 7, -59, 52, 17, 11, -56, 61, -74, 52, 63, -21, 53, -17, 66, -10, -58, 0]
b = [6, 106, 10, 0, 119, 52, 51, 101, 0, 0, 15, 48, 116, 22, 10, 58, 93, 59, 106, 43, 30, 47, 93, 62, 97, 63]
c = [304, 357, 303, 320, 304, 307, 349, 305, 257, 337, 340, 309, 396, 333, 320, 380, 362, 368, 286]
d = [52, 52, 95, 95, 110, 49, 51, 51, 95, 110, 110, 53, 116, 51, 98, 63]

checkFlag :: String -> Bool
checkFlag flag =
  length content == 76 &&
  all (==True) (map (\ (l,r) -> l == r) (zip one a)) &&
  all (==True) (map (\ (l,r) -> l == r) (zip two b)) &&
  all (==True) (map (\ (l,r) -> l == r) (zip three c)) &&
  all (==True) (map (\ (l,r) -> l == r) (zip four d))
  where content = map ord (extractFlag flag)
        one     = map (\ [l, r] -> (l - r)) (chunks 2 content)
        two     = map (foldr xor 0) (chunks 3 content)
        three     = map (foldr (+) 0) (chunks 4 content)
        four     = map head (chunks 5 content)
```

Input characters are subject to arithmetic operations on each other and compared to values.

```py
from z3 import *

FLAG_LENGTH = 76
A = [
    -43,
    61,
    58,
    5,
    -4,
    -11,
    64,
    -40,
    -43,
    61,
    62,
    -51,
    46,
    15,
    -49,
    -44,
    47,
    4,
    6,
    -7,
    47,
    7,
    -59,
    52,
    17,
    11,
    -56,
    61,
    -74,
    52,
    63,
    -21,
    53,
    -17,
    66,
    -10,
    -58,
    0,
]
B = [
    6,
    106,
    10,
    0,
    119,
    52,
    51,
    101,
    0,
    0,
    15,
    48,
    116,
    22,
    10,
    58,
    93,
    59,
    106,
    43,
    30,
    47,
    93,
    62,
    97,
    63,
]
C = [
    304,
    357,
    303,
    320,
    304,
    307,
    349,
    305,
    257,
    337,
    340,
    309,
    396,
    333,
    320,
    380,
    362,
    368,
    286,
]
D = [52, 52, 95, 95, 110, 49, 51, 51, 95, 110, 110, 53, 116, 51, 98, 63]

solver = Solver()
flag = [BitVec(f"flag_{i}", 8) for i in range(FLAG_LENGTH)]

for char in flag:
    solver.add(char >= 32, char <= 126)

for i in range(0, FLAG_LENGTH - 1, 2):
    solver.add(flag[i] - flag[i + 1] == A[i // 2])

for i in range(0, FLAG_LENGTH - 2, 3):
    solver.add(flag[i] ^ flag[i + 1] ^ flag[i + 2] == B[i // 3])

for i in range(0, FLAG_LENGTH - 3, 4):
    solver.add(flag[i] + flag[i + 1] + flag[i + 2] + flag[i + 3] == C[i // 4])

for i in range(0, FLAG_LENGTH - 4, 5):
    solver.add(flag[i] == D[i // 5])

if solver.check() == z3.sat:
    model = solver.model()
    flag = [model[char].as_long() for char in flag]
    flag = "HTB{" + "".join(chr(value) for value in flag) + "}"
    print(flag)
```

```plaintext
HTB{4_m0n4d_15_ju57_4_m0n01d_1n_7h3_c47360ry_0f_3nd0fUnc7or5-wh4t5_th3_pr0bl3m??}
```
