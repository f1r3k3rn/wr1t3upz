# SRDNLEN MPDH 

```
Author: @lrnzsir
description: Just another Diffie-Hellman, but what group are we in?
points: 474
difficulty: easy/medium
link: https://ctf.srdnlen.it/challenges#challenge-11
```
### ATTACHMENTS:

- [chall.py](./MPDH/chall.py)

- [output.txt](./MPDH/output.txt)

### MY APPROACH

first let's look at the source:

```python
    from random import SystemRandom
    from sympy.ntheory.generate import nextprime
    from Crypto.Cipher import AES
    from Crypto.Util.Padding import pad, unpad
    from Crypto.Hash import SHA256

    flag = b"srdnlen{????????????????????????????????????????????????????????????????????????????????????}"

    n = 32
    q = nextprime(int(0x1337**7.331))
    random = SystemRandom()


    class MPDH:
        n = n
        q = q

        def __init__(self, G=None) -> None:
            if G is None:
                J = list(range(n))
                random.shuffle(J)
                self.G = [(j, random.randrange(1, q)) for j in J]
            else:
                self.G = list(G)

        def one(self) -> "list[tuple[int, int]]":
            return [(i, 1) for i in range(self.n)]
        
        def mul(self, P1, P2) -> "list[tuple[int, int]]":
            return [(j2, p1 * p2 % self.q) for j1, p1 in P1 for i, (j2, p2) in enumerate(P2) if i == j1]
        
        def pow(self, e: int) -> "list[tuple[int, int]]":
            if e == 0:
                return self.one()
            if e == 1:
                return self.G
            P = self.pow(e // 2)

            P = self.mul(P, P)
            if e & 1:
                P = self.mul(P, self.G)
            return P


    mpdh = MPDH()

    a = random.randrange(1, q - 1)
    A = mpdh.pow(a)

    b = random.randrange(1, q - 1)
    B = mpdh.pow(b)

    Ka = MPDH(G=B).pow(a)
    Kb = MPDH(G=A).pow(b)
    assert Ka == Kb

    key = SHA256.new(str(Ka).encode()).digest()[:AES.key_size[-1]]
    flag_enc = AES.new(key, AES.MODE_ECB).encrypt(pad(flag, AES.block_size))
    assert flag == unpad(AES.new(key, AES.MODE_ECB).decrypt(flag_enc), AES.block_size)

    print(f"G = {mpdh.G}")
    print(f"{A = }")
    print(f"{B = }")
    print(f'flag_enc = "{flag_enc.hex()}"')

```


_I like thinking backwards, it makes it easier to both explain and understand:_

1) to have the flag we need the AES key

```python
    key = SHA256.new(str(Ka).encode()).digest()[:AES.key_size[-1]]
    flag_enc = AES.new(key, AES.MODE_ECB).encrypt(pad(flag, AES.block_size))
    assert flag == unpad(AES.new(key, AES.MODE_ECB).decrypt(flag_enc), AES.block_size)
```

2) to have the AES key we need Ka or Kb "they are the same"

```python
    Ka = MPDH(G=B).pow(a)
    Kb = MPDH(G=A).pow(b)
    assert Ka == Kb
```

3) to have Ka we must understand what pow does and what a or b is, "never jump to hasty conclusions"

```python
    def pow(self, e: int) -> "list[tuple[int, int]]":
            if e == 0:
                return self.one()
            if e == 1:
                return self.G
            P = self.pow(e // 2)

            P = self.mul(P, P)
            if e & 1:
                P = self.mul(P, self.G)
            return P
```

We can see that it reflects the implementation of fast pow but we don't know what is G and what mul does so let's understand the final part of our scripts (fast pow)->"https://www.geeksforgeeks.org/modular-exponentiation-power-in-modular-arithmetic/"

```python
    n = 32
    q = nextprime(int(0x1337**7.331))
    random = SystemRandom()


    class MPDH:
        n = n
        q = q

        def __init__(self, G=None) -> None:
            if G is None:
                J = list(range(n))
                random.shuffle(J)
                self.G = [(j, random.randrange(1, q)) for j in J]
            else:
                self.G = list(G)

        def one(self) -> "list[tuple[int, int]]":
            return [(i, 1) for i in range(self.n)]
        
        def mul(self, P1, P2) -> "list[tuple[int, int]]":
            return [(j2, p1 * p2 % self.q) for j1, p1 in P1 for i, (j2, p2) in enumerate(P2) if i == j1]
```


let's analyze init creates a list of n elements from 0 to 31 and then mixes them, finally adds for every elements a random intager modulus q let's remember what q is worth it's important

```python
    q = nextprime(int(0x1337**7.331))
```

now let's see mul for those who don't know this Python syntax very well you always have to think the other way around reading from right to left so mul takes as input two arrays of size 32 (in our case)
we call \\((x,a)\\) the pair of the first array e
\\((i,y,b)\\) the triple of the second therefore
\\(a\\) and \\(b\\) will be multiplied if and only if \\(i=x\\) finally
we will return the pair \\((y,a*b)\\)

## STRATEGY

the first thing that needs to be kept in mind is that when we multiply each element of the two arrays it will have a unique correspondence, the second thing that needs to be kept in mind is what we want to 

the first thing that needs to be kept in mind is that when we multiply each element of the two arrays it will have a unique correspondence, the second thing that needs to be kept in mind is what we analyzed before, this function performs multiplications, if it didn't mess up the multiplications, what would it be? \\( dlog(n=q,a=B[0],b=G[0]) \\)
our goal is to arrive at a discrete logarithm so the thing that immediately comes to mind is there will be a period in which the permutation resets and gives us the possibility of grouping the factors

letz see!

```python
    P=[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31]
    G=[27, 30, 6, 11, 20, 12, 24, 25, 17, 2, 16, 18, 19, 9, 0, 10, 26, 28, 31, 13, 8, 23, 15, 5, 21, 14, 22, 29, 4, 1, 7, 3]
    def rotate(F):
        R=[]
        for i in F:
            for j,k in enumerate(G):
                if i==j:
                    R.append(k)
        return R

    K=G
    for i in range(2,400):
        K=rotate(K)
        if K==P:
            print(i)
    # output:
    #  40
    #  80
    #  120
    #  160
    #  200
    #  240
    #  280
    #  320
    #  360
```


what did I do here?
I simply went to test my theory , the generator at a certain point will become a basic permutation 0,1,2,3 ... 31 and after 40 steps we see that the permutation repeats so the period is **40**

now we just need to put things together use \\[ pow(40) \\] and \\[ pow(80) \\]
what does that mean?

1) wants to multiply 40 times and arrive at the permutation 0,1,2,3,

2) wants to multiply  80 times and arriving at the permutation 0,1,2,3

but if the indices are equal if the quantities by which we are multiplying are equal as I said before we group the factors,therefore raising by a number \\[ x \\] means raising the ith position
\\[ \lfloor{x/40}\rfloor \\] times and then use mul the remaining \\[ x\bmod40 \\] times
at this point we have our dlog problem but what about the complexity?do you remember q
```python
q=1161847121043741987678715951
phi= q-1 = 2 * 3 * (5**2) * 8237 * 20477 * 45922162480592077
```
we have three options

1) bsgs "baby step giant step" \\[ \mathcal{O}(\sqrt{q}) \\]

    https://en.wikipedia.org/wiki/Baby-step_giant-step

2) pollard rho is heuristic but is about the same complexity as bsgs   \\[\mathcal{O}(\sqrt{q}*"it depends") \\]

    https://en.wikipedia.org/wiki/Pollard%27s_rho_algorithm

3) Pohlig–Hellman algorithm the complexity is let n be the number of distinct factors let \\[ phi[i] \\] how many times this factor is repeated in phi ,so we have \\[ \mathcal{O}(\sum_{i=0}^{n}\sqrt{phi[i]}*\log n)) \\]

    https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm

we choose the third because \\[ \sqrt{45922162480592077} = 214294569 \\] and the  others factors are negligible

# solution : [solve.py](./solve.py)
