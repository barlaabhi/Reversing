            
# Pretend Simple

[challenge_file](https://github.com/teambi0s/InCTF/raw/master/2018/Reversing/simple/Handout/simple)


This challenge basically takes a string of length of 15 and left rotates each char by 1 and compares each character with a value

So we break at the point where comparision happens and find with what value our left-rotated input string is getting compared to and then right-rotate the each character to get the actual input 


Initially we load the binary and set a breakpoint at the `cmp` instruction. Whenever the values in the `rcx` and `r10` registers are different the value in `rcx` is right-rotated and appended into `file.txt`, after we do this for 15 times, we remove all breakpoints and run it with the new input.
`file.txt` is initially empty

```py
import gdb

gdb.execute('file 1.simple')
gdb.execute('b *0x00000000004001e3')


for i in range(15):
	

	gdb.execute("run < file.txt")

	r10 = gdb.execute('p/x $r10',to_string=True)
	r10 = int(r10[-3:-1],16)
	
	rcx = gdb.execute('p/x $ecx',to_string=True)
	rcx = int(rcx[-3:-1],16)

	while(r10==rcx):
		gdb.execute('continue')

		r10=gdb.execute('p/x $r10',to_string=True)
		r10 = int(r10[-3:-1],16)

		rcx = gdb.execute('p/x $ecx',to_string=True)
		rcx = int(rcx[-3:-1],16)


	val=gdb.execute('p/x $ecx',to_string=True)

	val = int(val[-3:-1],16)
	val = bin(val)[2:]
	val = val[-1]+val[:-1] #right rotate by 1
	val = int(val,2)
	

	with open('file.txt','a+') as f:
		f.write(chr(val))
        
	gdb.execute('continue')


gdb.execute('delete')
gdb.execute('run < file.txt')
```

##### Solution-2

```py 
a=[97, 87, 53, 106, 100, 71, 90, 114, 78, 71, 108, 54, 77, 50, 52]

for i in range(15):
    a[i] = ((a[i]^12)+6)>>1 & 0xff

flag = "".join(chr(i) for i in a)

```

#### Flag: `inctf{aW5jdGZrNGl6M24}`

###### tags: `inctf` `gdb_scripting`