## BCD-to-Binary Conversion ##

### Algorithm ###
The algorithm works from LSB to MSB of the BCD representation, and produces the binary digits from LSB to MSB, too.

Start with a binary running result of no bits at all, and for `n` bits of BCD repeat `n` times:

1. add LSB of BCD repr. as MSB to your binary running result
2. right-shift the whole BCD repr. by one bit (throwing away the LSB)
3. subtract 3 from every (4-bit) BCD digit if it's (now) >7


**Why subtract 3?**
Well, the basic idea is of course to successively divide by 2 while keeping track of remainders mod 2.
However, since we have our number represented as 4-bit portions of BCD we cannot simply right-shift the whole thing in order to correctly divide it by 2.
We have to compensate for the fact that the bit that's shifted in to each BCD digit is worth 10/2=5, not 8.
Now, shifting in a 0 from the left is fine, since 0\*5 = 0\*8 = 0.
But shifting in a 1 actually means halving the current BCD digit (which is fine) and adding 8 - not quite, should be 5!
That's why you have to subtract 3 if a 1 happens to be shifted in.


### Circuit ###

Now let's implement this algorithm as a circuit in logicly.
Of course we do not (yet) want to commit to a particular number `n` of BCD bits, ie. roughly `n/4` decimal digits.
So we will aim for a suitably sized building block, from which we can then build a complete converter as large as we want.

An input size of 4 bits seems obvious, so let's just do this.
The shifting part is easy, we just have to wire up inputs to outputs appropriately.
This will cover steps 1 and 2 from above, so the inputs to our building block will be *the already right-shifted*
bits of BCD.
As for the conditional subtraction of 3, we consider each input pattern and its intended output.
There are at most 16=2^4 possible inputs, so let's just list them all:

#### Truth table 1 ###
```
D (dec) | D[3:0] | Q (dec) | Q[3:0]
--------|--------|---------|-------
     0  |  0000  |      0  |  0000
     1  |  0001  |      1  |  0001
     2  |  0010  |      2  |  0010
     3  |  0011  |      3  |  0011
     4  |  0100  |      4  |  0100
     5  |  0101  |      5  |  0101
     6  |  0110  |      6  |  0110
     7  |  0111  |      7  |  0111
     8  |  1000  |      5  |  0101
     9  |  1001  |      6  |  0110
    10  |  1010  |      7  |  0111
    11  |  1011  |      8  |  1000
    12  |  1100  |      9  |  1001
    13  |  1101  |     10  |  1010
    14  |  1110  |     11  |  1011
    15  |  1111  |     12  |  1100
```
Input values D=0..7 (decimal) are passed through as is, while D=8..15 get mapped to D-3, ie 5..12.

But hey, there can't be 16 valid input patterns if we assume a valid BCD input, which has only 10 different patterns per BCD digit.
**Which can actually occur and which cannot?**
Remember, the input bits are coming from *the already right-shifted* BCD representation.
The BCD values range from 0..9 in decimal, or 0000..1001 in binary.
Shifting in a 0 from the left gives 0..4, or 0000..0100.
Shifting in a 1 from the left gives 8..12, or 1000..1100.
So the outputs for input values 5, 6, 7, and 13, 14, 15 actually don't matter.

The following table shows the so-called **Don't-Care**s as dashes "-".

#### Truth table 2 ###
```
D (dec) | D[3:0] | Q (dec) | Q[3:0]
--------|--------|---------|-------
     0  |  0000  |      0  |  0000
     1  |  0001  |      1  |  0001
     2  |  0010  |      2  |  0010
     3  |  0011  |      3  |  0011
     4  |  0100  |      4  |  0100
     5  |  0101  |      -  |  ----
     6  |  0110  |      -  |  ----
     7  |  0111  |      -  |  ----
     8  |  1000  |      5  |  0101
     9  |  1001  |      6  |  0110
    10  |  1010  |      7  |  0111
    11  |  1011  |      8  |  1000
    12  |  1100  |      9  |  1001
    13  |  1101  |      -  |  ----
    14  |  1110  |      -  |  ----
    15  |  1111  |      -  |  ----
```

#### A better kind of truth table ####

Now, if we were to stare at the above table just long enough we might eventually "see" a good way of combining the input bits with AND, OR and the like s.t. they produce the output bits we want.

However, there is a better way. Let's rewrite the table in this form:
```
Q (decimal)                          Q[3:0] (binary)
      \ D[1:0]                             \ D[1:0]
D[3:2] \  00   01   11   10          D[3:2] \  00   01   11   10 
        +----+----+----+----+                +----+----+----+----+
     00 |  0 |  1 |  3 |  2 |             00 |0000|0001|0011|0010|
        +----+----+----+----+                +----+----+----+----+
     01 |  4 | -- | -- | -- |             01 |0100|----|----|----|
        +----+----+----+----+                +----+----+----+----+
     11 |  9 | -- | -- | -- |             11 |1001|----|----|----|
        +----+----+----+----+                +----+----+----+----+
     10 |  5 |  6 |  8 |  7 |             10 |0101|0110|1000|0111|
        +----+----+----+----+                +----+----+----+----+
```
Rows are labelled with the upper two input bits D3 and D2, columns with the lower two D1 and D0.
This again gives 16 entries, 6 of which we don't care about.

But **why are they so strangely ordered?** The point here is to have exactly one label bit changed between any two neighbouring rows or columns, respectively. Note that this is also true for the top and bottom row, and for the left and right column.
So we can consider these as "neighbours" in that sense as well.

Why this is helpful, we'll see in a minute. First let's split it up for each Q bit individually,
s.t. patterns and commonalities stick out even better:
```
    Q0                                 Q1
      \ D[1:0]                           \ D[1:0]
D[3:2] \  00   01   11   10        D[3:2] \  00   01   11   10 
        +----+----+----+----+              +----+----+----+----+
     00 |  0 |  1 |  1 |  0 |           00 |  0 |  0 |  1 |  1 |
        +----+----+----+----+              +----+----+----+----+
     01 |  0 |  - |  - |  - |           01 |  0 |  - |  - |  - |
        +----+----+----+----+              +----+----+----+----+
     11 |  1 |  - |  - |  - |           11 |  0 |  - |  - |  - |
        +----+----+----+----+              +----+----+----+----+
     10 |  1 |  0 |  0 |  1 |           10 |  0 |  1 |  0 |  1 |
        +----+----+----+----+              +----+----+----+----+
     
    Q2                                 Q3
      \ D[1:0]                           \ D[1:0]
D[3:2] \  00   01   11   10        D[3:2] \  00   01   11   10 
        +----+----+----+----+              +----+----+----+----+
     00 |  0 |  0 |  0 |  0 |           00 |  0 |  0 |  0 |  0 |
        +----+----+----+----+              +----+----+----+----+
     01 |  1 |  - |  - |  - |           01 |  0 |  - |  - |  - |
        +----+----+----+----+              +----+----+----+----+
     11 |  0 |  - |  - |  - |           11 |  1 |  - |  - |  - |
        +----+----+----+----+              +----+----+----+----+
     10 |  1 |  1 |  0 |  1 |           10 |  0 |  0 |  1 |  1 |
        +----+----+----+----+              +----+----+----+----+
```
These are the [Karnough maps](https://en.wikipedia.org/wiki/Karnaugh_map)
for our little BCD-to-binary building block.


#### (Blindly) translating a Karnough map to logic ####

Before we go on using the Karnough maps for optimization - which is actually what they're really for -
let's get out logicly and practice a little.

Take Q3: it's got three 1s. We'll address the entries by first stating D3 and D2, then D1 and D0; like so:
`11 00`, `10 11`, `10 10`.
This just means that Q3 should be 1 if
 - D3=1 AND D2=1 AND D1=0 AND D0=0
 - OR (D3=1 AND D2=0 AND D1=1 AND D0=1)
 - OR (D3=1 AND D2=0 AND D1=1 AND D0=0)
 or, like arithmetic where AND is implicit (like multiplication), OR is "+" (like addition)
 and NOT is "/" (like unary minus, or negation):
 ```
  Q3 = D3 D2 /D1 /D0 + D3 /D2 D1 D0 + D3 /D2 D1 /D0
 ```
 With gates:
 
 (BCD-to-bin_01.png)
 
