# 3 Oscillation #
In this chapter:
#### 3.1 [Negative feedback: simple oscillator circuits](#3_1) ####
##### 3.1.1 [Using an AND-gate](#3_1_1) #####
##### 3.1.2 [OR works just as well](#3_1_2) #####

#### 3.2 [Oscilloscope](#3_2) ####
##### 3.2.1 [The build](#3_2_1) #####
##### 3.2.2 [The test](#3_2_2) #####

#### 3.x [Adjustable frequency oscillator](#3_x) ####
##### 3.x.y [Frequency divider](#3_x_y) #####
###### 3.x.y.z [Selecting the divisor](#3_x_y_z) ######

---

## <a name="3_1"></a>3.1 Negative feedback: simple oscillator circuits ##

#### <a name="3_1_1"></a>3.1.1 Using an AND-gate ####
Consider an AND-gate the output of which is **negated** and then **fed back** into one of the inputs:

![img/?](img/osci_1_AND_2ticks_00.png)

If we switch on `EN`, the thing starts oscillating:

![img/?](img/osci_1_AND_2ticks_01.png)
![img/?](img/osci_1_AND_2ticks_02.png)
![img/?](img/osci_1_AND_2ticks_03.png)
![img/?](img/osci_1_AND_2ticks_04.png) ...and starts over.

So this has a **period** of 4 ticks, or a **half-period** of 2.

We can make the period longer by inserting a buffer:

![img/?](img/osci_1_AND_3ticks_01.png)
![img/?](img/osci_1_AND_3ticks_02.png)
![img/?](img/osci_1_AND_3ticks_03.png)
![img/?](img/osci_1_AND_3ticks_04.png)
![img/?](img/osci_1_AND_3ticks_05.png)
![img/?](img/osci_1_AND_3ticks_06.png) ...and repeats.

The period is 6, the half-period is 3.
Note that the number of gates in the loop equals the length of the half-period.


#### <a name="3_1_2"></a>3.1.2 OR works just as well ####
![img/?](img/osci_1_OR_4ticks-anim.gif)

Just the same, except input and outputs are inverted.
So when `/EN` is switched off, it starts oscillating.
The pictures are shown below each other so we can see
more easily how the signal propagates through the loop

![img/?](img/osci_1_OR_4ticks_01.png) 1

![img/?](img/osci_1_OR_4ticks_02.png) 2

![img/?](img/osci_1_OR_4ticks_03.png) 3

![img/?](img/osci_1_OR_4ticks_04.png) 4

![img/?](img/osci_1_OR_4ticks_05.png) 5

![img/?](img/osci_1_OR_4ticks_06.png) 6

![img/?](img/osci_1_OR_4ticks_07.png) 7

![img/?](img/osci_1_OR_4ticks_08.png) 8

![img/?](img/osci_1_OR_4ticks_01.png) 1 (again)

...and so on.


## <a name="3_2"></a>3.2 Oscilloscope ##

### <a name="3_2_1"></a>3.2.1 The build ###
In the period-8 example above we start to see how patterns are shifted
through a series of buffers, from right to left.
The most recent signal (out of the NOT) is on the right (`Y4b`),
earlier ones are further to the left (`Y4c`, `Y4d`).

We can use this to build something that allows to inspect the development
of a signal over time in a much more comfortable way than with picture series as above.
Let's build an **oscilloscope**!

Start with a light bulb connected to a buffer:

![img/?](img/osci_2_build_01.png)

Copy it and place **the new one to the left of the old**.
Connect the buffers and push them together st. the light bulbs overlap like this:

![img/?](img/osci_2_build_02.png)
to make
![img/?](img/osci_2_build_03.png)

[\[animation\]](img/osci_2_build_02-anim.gif)

![img/?](img/osci_2_build_02-anim.gif)

Repeat with the thing you just build.
Make sure to **place the copy to the left** of the original,
since otherwise the light bulbs won't overlap correctly.

![img/?](img/osci_2_build_04.png)
to make
![img/?](img/osci_2_build_05.png)

It is crucial that you get the overlap right.
Here's how it looks like if you did it wrong:

![img/?](img/osci_2_build_06.png)

We repeat this until we have a suitably long chain.
The length determines how far "back in time" we will be able to see.
So we have to decide upon the "memory depth" of our oscilloscope.
32 is a bit short, 64 is too much. Let's make 48 = 16 + 32. That is:
Go up to 16, and keep a copy of this before doubling again.
Once you have the 32 you place that **to the left of the old 16**.
Finally it should look like this:

![img/?](img/osci_2_build_07.png)

Now it's time to add a second channel to our oscilloscope,
allowing us to see two different signal in relation to each other.
For this we copy the chain of 48 and **place the copy above the original**.

Before we drag them onto each other we have to decide how we want the "channel inputs"
to be ordered.
Obviously it's better to have them in the same order as their respective "channel displays",
so the light bulbs on top should be connected to the upper chain of buffers.

To achieve this: in the orignal (the bottom one), pull down the line of buffers to make space for
the buffers of the copy (the top one) we just made.
Finally drag the copy down over the original like so:

![img/?](img/osci_2_build_08.png)

to make

![img/?](img/osci_2_build_09.png)


Now there are way too many connections crossing each other, and below components.
That is, too many for us to see anything useful in them during simulation.
That's not a problem - we've got the light bulbs compact and nicely ordered for this -
but then we can as well drag the buffer chains on top of each other in order to save
some space.

Drag down the upper chain, rather than the lower one up, as even more channels are to be
added.
![img/?](img/osci_2_build_10.png)

6 channels are sufficient. As a last step, the buffers of the right-most column have been dragged out and rotated (individually), in order to make connecting to them easier.

![img/?](img/osci_2_build_11.png)
Two rulers have also been added, one measuring from left to right (that would be the time in ticks), and the other from measuring from right to left (for convenience).

It's a bit fiddly to get the ruler marks properly aligned with the light bulbs.
Here's the label text for the upper one (right to left):

	|      |      |     4|5    |      |       |      |    4|0    |      |      |      |     3|5    |       |      |      |    3|0    |      |      |       |    2|5    |      |       |      |    2|0    |      |      |      |     1|5    |      |       |      |    1|0    |       |      |       |      |5    |       |      |      |      |0 

And for the lower one (left to right):

	|0    |      |       |      |      |5     |      |      |      |    1|0    |      |       |      |     1|5    |      |      |      |    2|0    |      |       |      |    2|5     |      |      |      |    3|0    |      |       |      |    3|5     |      |      |      |     4|0    |      |       |      |     4|5    |      |      |  


### <a name="3_2_2"></a>3.2.2 The test ###
Finally, it's time to test the oscilloscope.
Connect the simple oscillator circuits from the beginning.

![img/?](img/osci_2_build_12.png)

Here's a complete trace with `EN` switched on from t=8 to t=43 (channel 0).
The oscillation starts at t=10 and ends at t=45.
![img/?](img/osci_2_trace_01.png)
[\[animation\]](img/osci_2_trace_01-anim.gif)

Channels 4 and 5 show oscillators built with OR, which give inverted outputs compared to those built with AND (channels 1, 2, 3).

They all start out in sync, but due to the different periods they don't stay in sync.
Channels 1 and 2 (or, of course, 4 and 5) come back into sync every 12 ticks - the least common multiple of 4 and 6.
All three, 1, 2 and 3 come back into sync every 24 ticks - the least common multiple of 4, 6 and 8.

## <a name="3_x"></a>3.x Adjustable frequency oscillator ##

### <a name="3_x_y"></a>3.x.y Frequency divider ###

#### <a name="3_x_y_z"></a>3.x.y.z Selecting the divisor ####
	half-period [ticks] (n)
	       \ base half-period
	        \   2       3       5       7       9      11      13      15      17
	 divisor +-------+-------+-------+-------+-------+-------+-------+-------+-------+
	    DIV1 |  2    |  3    |  5    |  7    |  9    | 11    | 13    | 15    | 17(1) |
	         +-------+-------+-------+-------+-------+-------+-------+-------+-------+
	    DIV2 |  4    |  6    | 10    | 14    |       |       |       |       |       |
	         +-------+-------+-------+-------+-------+-------+-------+-------+-------+
	    DIV4 |  8    | 12    |       |       |       |       |       |       |       |
	         +-------+-------+-------+-------+-------+-------+-------+-------+-------+
	    DIV8 | 16(0) |       |       |       |       |       |       |       |       |
	         +-------+-------+-------+-------+-------+-------+-------+-------+-------+

Karnaugh maps for DIV1, DIV2, DIV4 and DIV8:

	   DIV1                                 DIV2
	      \ S[1:0]                             \ S[1:0]
	S[3:2] \  00   01   11   10          S[3:2] \  00   01   11   10 
	        +----+----+----+----+                +----+----+----+----+
	     00 |  0 |  1 |  1 |  1 |             00 |    |    |    |    |
	        +----+----+----+----+                +----+----+----+----+
	     01 |  0 |  1 |  1 |  0 |             01 |  4 |    |    |  6 |
	        +----+----+----+----+                +----+----+----+----+
	     11 |  0 |  1 |  1 |  0 |             11 |    |    |    | 14 |
	        +----+----+----+----+                +----+----+----+----+
	     10 |  0 |  1 |  1 |  0 |             10 |    |    |    | 10 |
	        +----+----+----+----+                +----+----+----+----+
	        NOR(DIV2, DIV4, DIV8)               /S3 S2 /S0 + S3 S1 /S0

	   DIV4                                 DIV8
	      \ S[1:0]                             \ S[1:0]
	S[3:2] \  00   01   11   10          S[3:2] \  00   01   11   10 
	        +----+----+----+----+                +----+----+----+----+
	     00 |    |    |    |    |             00 |  1 |  0 |  0 |  0 |
	        +----+----+----+----+                +----+----+----+----+
	     01 |    |    |    |    |             01 |  0 |  0 |  0 |  0 |
	        +----+----+----+----+                +----+----+----+----+
	     11 | 12 |    |    |    |             11 |  0 |  0 |  0 |  0 |
	        +----+----+----+----+                +----+----+----+----+
	     10 |  8 |    |    |    |             10 |  0 |  0 |  0 |  0 |
	        +----+----+----+----+                +----+----+----+----+
                 S3 /S1 /S0                       NOR(S3, S2, S1, S0)

blah
