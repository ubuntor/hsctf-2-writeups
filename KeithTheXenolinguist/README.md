# Keith the Xenolinguist

## TL;DR
 * It's a stack-based language
 * stnrpbjghakeo are digits from -6 to 6
 * fwooosh is add
 * orkshnoddle is print/check
 * flordoralscorp is integer division by 2
 * halgemijig is hailstone
 * clogenheim is lcm

## Writeup

### Number system and fwooosh
On first look, the lines look like some sort of stack-based language.
From some analysis, it looks like `orkshnoddle` looks like an answer keyword,
since it's the next-to-last word of every line, and there are a lot of lines of
the form `[word] orkshnoddle [word]`.

We can determine that `fwooosh` and `clogenheim` are *binary* operations, and
`flordoralscorp` and `halgemijig` are *unary* operations.

We guess that `fwooosh` is addition, since its output has similar lengths.
We suspect that the other words are really numbers in a different base, since
there are a limited number of characters used. From looking at frequencies of
letters, the letters `rpbjgha` appear in words, with `stnkeo` appearing only as
the first letters of each word. This looks like a base-7 number system, but
lines like `bjh kb fwooosh orkshnoddle rg` don't make sense: 2 numbers adding to
make a smaller number? Either `fwooosh` is subtraction, or there are negative
numbers. Subtraction doesn't make sense because of lines like
`gjp or fwooosh orkshnoddle hph` and `op kh fwooosh orkshnoddle gaj`, since
order matters in subtraction. It doesn't look like negative signs are used,
since there are 6 letters that only appear at the start of the word, so maybe
digits can be negative? This sounds like a balanced number system.

To verify, let's shove some lines (that don't use `stnkeo`) into Mathematica and
see what we get:

```
In[1]:= FindInstance[{7^5*b + 7^4*j + 7^3*a + 7^2*r + 7*h + g +
                      7^4*r + 7^3*a + 7^2*a + 7*b + h ==
                      7^5*b + 7^4*p + 7^3*b + 7^2*j + 7*g + a,
                      7^5*p + 7^4*b + 7^3*r + 7^2*a + 7*p + r +
                      7^4*a + 7^3*r + 7^2*h + 7*h + g ==
                      7^5*p + 7^4*g + 7^3*h + 7^2*p + 7*j + p,
                      r != p, -7 < r < 7, -7 < p < 7, -7 < b < 7, -7 < j < 7,
                      -7 < g < 7, -7 < h < 7, -7 < a < 7},
                      {r, p, b, j, g, h, a}, Integers]

Out[1]= {{r -> 6, p -> 4, b -> 2, j -> 0, g -> -2, h -> -4, a -> -6}}
```

Note that the actual digits can be any multiples of this, including negative
multiples, which would flip the alphabet. For reasons that will become clear
later, we will use the digits `rpbjgha` as values from -3 to 3, instead of
its reverse.

Now that we have the values of `rpbjgha`, we look at the digits `stnkeo`,
and find that they are `{-6,-5,-4,4,5,6}` (respectively), so we have the whole
alphabet: `stnrpbjghakeo` are digits from -6 to 6.

Now that we can decode the numbers, lets look at the other operations:

### flordoralscorp
The line
```
ejpghbgjghrh flordoralscorp orkshnoddle haargjrjgrhg
```
gets decoded into
```
9813237968 flordoralscorp orkshnoddle 4906618984
```
This looks like a division by 2. Looking at other examples verifies that
this is an integer division by 2. (Note that if the alphabet were reversed,
then this would instead be a ceiling of a division by two, which is uncommon.)


### halgemijig
Playing around with the first five instances of `halgemijig`, we see that
sometimes it divides by 2, other times it multiplies by 3 (although not
exactly). Hmm, this sounds like Hailstone numbers! If the number is even, it
divides by 2, while if it's odd, it multiplies by 3 and adds 1. (Note that if
the alphabet were reversed, this would multiply by 3 and subtract 1.)

### clogenheim
Playing around with lines that use `clogenheim`, it seems like multiplication.
However, looking at some lines and trying to reverse them, some numbers have
factors that mysteriously appear after using `clogenheim`. Other times, the
signs of numbers suddenly switch to positive. Hmm, an operation that works
like multiplication, but can also add other factors and is always positive...
sounds like taking the LCM! This works with other lines, so we've decoded
everything! We only need to evaluate the line given in the problem
description to get the final answer.

### Final code
```python
#           FEDCBA0123456
alphabet = "stnrpbjghakeo"

def n(s):
    r = 0
    for i in s:
        r *= 7
        r += alphabet.index(i)-6
    return r

def l(s):
    r = ''
    while -7 >= s or s >= 7:
        r = alphabet[(s+3)%7-3+6]+ r
        s -= (s+3)%7-3
        s //= 7
    if s != 0:
        r = alphabet[s+6] + r
    return r

def lcm(p,q):
    p, q = abs(p), abs(q)
    m = p * q
    if not m: return 0
    while True:
        p %= q
        if not p: return m // q
        q %= p
        if not q: return m // p

for s in open("xenolinguist.txt").read().split('\n'):
    if s == '':
        continue
    print(s)
    stack = []
    ins = s.split()

    for i in ins:
        if i == "fwooosh":
            stack.append(stack.pop()+stack.pop())
        elif i == "flordoralscorp":
            stack.append(stack.pop()//2)
        elif i == "orkshnoddle":
            st = stack.pop()
            print(st,l(st))
            print(n(ins[-1]),ins[-1])
            print(l(st)==ins[-1])
        elif i == "clogenheim":
            stack.append(lcm(stack.pop(),stack.pop()))
        elif i == "halgemijig":
            st = stack.pop()
            if st%2 == 0:
                stack.append(st//2)
            else:
                stack.append(3*st+1)
        else:
            stack.append(n(i))
    print()
```
