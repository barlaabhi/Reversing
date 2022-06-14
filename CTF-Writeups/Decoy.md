# Decoy


[challenge_file](https://github.com/teambi0s/InCTFi/raw/master/2018/Reversing/Decoy/Handout/Decoy.exe)

The disassembly of the binary seems very big but there are a very few functions which will give us the solution

There are three `goto's` (jumps) which when jumped terminates the execution and doesn't give us the flag and there are also functions which return different values when we use a debugger

Analysing carefully we can see that there is atleast one `goto` for most of the `if/else` blocks at the end, which means that irresepective of the statements inside the block the jump will definately execute and terminate the program

Eliminating all such blocks we will be left with 6 conditions out of which one verifies that length of input is 44 and another one verifies that we aren't using a debugger and the rest 4 functions divide the input into 4 parts and does some operartions on it to make sure we've entered the correct string

Using Z3 to conquer the constraints we get the flag

```py

from z3 import *

flag=''
inp = [ BitVec('inp%s'%i,32) for i in range(11) ]


arr = [ BitVec('arr%s'%i,32) for i in range(11) ]
j=0
s=Solver()
for i in range(5): 
	s.add(arr[j+6]==inp[i]+4)
	s.add(arr[j]==inp[i+5]+2)
	j+=1

s.add(arr[5]==inp[10]+2)

str1 = "}[2waHmrgxj"
str1 = [ord(i) for i in str1]

for i in range(11):
	s.add(str1[i]==arr[i])
s.check()
b = s.model()

for i in inp:
	flag+=chr(int(str(b[i])))


#--------------------------------------------------------------------
	


inp = [ BitVec('inp%s'%i,32) for i in range(11) ]


arr = [ BitVec('arr%s'%i,32) for i in range(24) ]

s = Solver()


for i in range(11):
	s.add(arr[i+12]==(inp[i]^0xb)^0x13)


str1 = "#f}wLG{ L} "
str1 = [ord(i) for i in str1]

for i in range(11):
	s.add(arr[i]==(str1[i]^0xb))


for i in range(11):
	s.add(arr[i+12]==arr[i])

s.check()
b = s.model()
for i in inp:
	flag+=chr(int(str(b[i])))


#--------------------------------------------------------------------

s = Solver()

inp = [ BitVec('inp%s'%i,32) for i in range(11) ]

v3 = [0]*11


v3[0] = 110
v3[1] = 93
v3[2] = 57
v3[3] = 85
v3[4] = 102
v3[5] = 57
v3[6] = 112
v3[7] = 30
v3[8] = 65
v3[9] = 21
v3[10] = 125

	

v6=1
for i in range(10,-1,-1):
	num = inp[i]
	num=num^v6
	v6=num
	s.add(num == v3[i] )

s.check()
b = s.model()

for i in inp:
	flag+=chr(int(str(b[i])))



#--------------------------------------------------------------------

s=Solver()

str1 = '{`I5z%u5dL~'

str1 = [ord(i) for i in str1]

inp = [ BitVec('inp%s'%i,32) for i in range(11) ]


arr = [ BitVec('arr%s'%i,32) for i in range(11) ]

for i in range(11):
	s.add(arr[i]==(inp[i]+1))


for i in range(11):
	s.add(str1[i]==(arr[i]))
s.check()
b = s.model()

for i in inp:
	flag+=chr(int(str(b[i])))

print(flag)
```
Flag: ```inctf{Y0u_F0und_Th3_n33dl3_In_Th|z_H4y$t4cK}```


###### tags: `inctfi` `Z3`