## BCD-to-Binary Conversion ##

### Algorithm ###
The algorithm works from LSB to MSB of the BCD representation, and produces the binary digits from LSB to MSB, too.

Start with a binary running result of no bits at all, and for `n` bits of BCD repeat `n` times:

1. add LSB of BCD repr. as MSB to your binary running result
2. right-shift the whole BCD repr. by one bit (throwing away the LSB)
3. subtract 3 from every (4-bit) BCD digit if it's (now) >7


**Why subtract 3?** Well, the basic idea is of course to successively divide by 2 while keeping track of remainders mod 2. However, since we have our number represented as 4-bit portions of BCD we cannot simply right-shift the whole thing in order to correctly divide it by 2. We have to compensate for the fact that the bit that's shifted in to each BCD digit is worth 10/2=5, not 8. Now, shifting in a 0 from the left is fine, since 0`*`5 = 0`*`8 = 0. But shifting in a 1 actually means halving the current BCD digit (which is fine) and adding 8 - not quite, should be 5! That's why you have to subtract 3 if a 1 happens to be shifted in.


### Circuit ###

#### Truth table ###
```
X (dec) | X[3:0] | Q (dec) | Q[3:0]
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


