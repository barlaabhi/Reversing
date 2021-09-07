# Jigger



[challenge_file](https://github.com/teambi0s/InCTF/raw/master/2018/Reversing/jigger/Handout/inctfst)

Given binary takes two command line arguments and makes sure that both the number's product %10 is 7 

The following operations are done on Our input 
1. each character is Xored with a value from memory 
2. Then the input is sent to a function which shuffles 5 values from our input at a time in an order and the resultant string is again passed to the same function but to shuffle with a different order 
3. Then the resultant shuffled string is compared to a string in memory 

Using Z3

Here each element in the inp array is a z3 object we xor each element of it with a value and then shuffle the elements in the inp array and finally compare it with the array `str_cmp`

```py
from z3 import *

Xored_bytes=[0x39,0x33,0x32,0x36,0x61,0x39,0x32,0x36,0x62,0x35,0x30,0x30,0x33,0x39,0x35,0x32,0x34,0x34,0x34,0x32,0x35,0x36,0x30,0x36,0x37,0x33,0x33,0x37,0x39,0x34,0x38,0x33,0x37,0x37,0x36]
str_cmp=[0x69,0x6f,0x35,0x5d,0x5e,0x2c,0x6c,0x6e,0x6f,0x72,0x5a,0x6c,0x67,0x6a,0x73,0x5c,0x5c,0x71,0x61,0x68,0x6a,0x5c,0x77,0x70,0x72,0x66,0x6c,0x75,0x6e,0x5e,0x63,0x66,0x75,0x70,0x5e]
shfl1=[2,4,0,1,3]
shfl2=[4,2,1,3,0]
dummy = [0,0,0,0,0]



inp = [BitVec('inp%s'%i,16) for i in range(35)]

s=Solver()

for i in range(35):
	inp[i]=inp[i]^(7*i%10)^Xored_bytes[i]

k=0
while (k+5)<=35:

	for j in range(5):
		dummy[j]=inp[k]
		k+=1
	k-=5
	for l in range(5):
		inp[k]=dummy[shfl1[l]]
		k+=1


k=0
while (k+5)<=35:

	for j in range(5):
		dummy[j]=inp[k]
		k+=1
	k-=5
	for l in range(5):
		s.add(str_cmp[k]==dummy[shfl2[l]])
		k+=1

s.check()

b=s.model()

inp = [BitVec('inp%s'%i,16) for i in range(35)] 

inp_str=""

for i in inp:
	inp_str+=chr(int(str(b[i])))

print(inp_str)

```

 inp is initialized again the same way because the z3 object changes when it is xored, which makes it difficult to print/gather the values in the order required, since the values obtained in s.model() are stored, it doesn't make any changes

##### flag: `inctf{_7ranspos1tion_is_easy_to_work_with}`

###### tags: `inctf` `Z3`