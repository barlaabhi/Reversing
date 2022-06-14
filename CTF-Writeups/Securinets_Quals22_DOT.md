# DOT

Securinets DOT challenge

This is a small VM challenge, where the opcodes are fetched from a database at runtime. The operation happens on our input array, so eventually these operations change the value inside the array and then the array is `UTF-8` encoded and then converted to a base-64 string

We first get the base64 decoded array and then we can use Z3 to get our flag

## Reverse the base-64 string


* size(char) - 2 bytes in C#
*  if the ASCII value of a character is more than 0x7f, `UTF-8` encoding scheme adds `0xc2`,`0xc3` accordingly as the MSB
* This gives out an array of length 58 when normally decode, To avoid this we encode our base64 decoded string in `UTF-8` and then convert it to char array
 
To reverse this, we first decode the base-64 string to bytes and then convert to `char` array with `utf-8` encoding

C# script to get the decoded bytearray

```cs
var decoded_str = Convert.FromBase64String("w4Vkw4bDqcOxwqbDj8OKw7XDmcOqZHLDinBdw4/Dul9mw4JfbsOIaG7Dil3DmWxfbMOTwr3DkWJoYg==");
char[] to_cmp = Encoding.UTF8.GetString(decoded_str).ToCharArray();

foreach (Int16 item in to_cmp)
{
    Console.Write(item);
    Console.Write(",");
}
```

## Use Z3 to get our flag

Now we have the array we can Use z3 to get our input

```py
import mysql.connector
from mysql.connector import Error
from z3 import *

text = 'c2hpZmwK00c2hpZmwK01c2hpZmwK02c2hpZmwK03c2hpZmwK04c2hpZmwK05c2hpZmwK06c2hpZmwK07c2hpZmwK08c2hpZmwK09c2hpZmwK0ac2hpZmwK0bc2hpZmwK0cc2hpZmwK0dc2hpZmwK0ec2hpZmwK0fc2hpZmwK10c2hpZmwK11c2hpZmwK12c2hpZmwK13c2hpZmwK14c2hpZmwK15c2hpZmwK16c2hpZmwK17c2hpZmwK18c2hpZmwK19c2hpZmwK1ac2hpZmwK1bc2hpZmwK1cc2hpZmwK1dc2hpZmwK1ec2hpZmwK1fc2hpZmwK20c2hpZmwK21c2hpZmwK22c2hpZmwK23c2hpZmwK24c2hpZmwK25b3BYT1IK000005b3BYT1IK050005b3BYT1IK000005b3BYT1IK03030Ab3BYT1IK0A030Ab3BYT1IK03030Ab3BfcGwK040db3BfcGwK080db3BfcGwK0c0db3BfcGwK100db3BfcGwK140db3BfcGwK180db3BfcGwK1c0db3BfcGwK200db3BfcGwK240db3BYT1IK141422b3BYT1IK221422b3BYT1IK141422b3BYT1IK252511b3BYT1IK112511b3BYT1IK252511b3BYT1IK010121b3BYT1IK210121b3BYT1IK010121b3BYT1IK0b0b16b3BYT1IK160b16b3BYT1IK0b0b16b3BfTUkK000db3BfTUkK030db3BfTUkK060db3BfTUkK090db3BfTUkK0c0db3BfTUkK0f0db3BfTUkK120db3BfTUkK150db3BfTUkK180db3BfTUkK1b0db3BfTUkK1e0db3BfTUkK210db3BfTUkK240d'
id_array = ['q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'q', 'd', 'd', 'd', 'd', 'd', 'd', 'G', 'G', 'G', 'G', 'G', 'G', 'G', 'G', 'G', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'm', 'm', 'm', 'm', 'm', 'm', 'm', 'm', 'm', 'm', 'm', 'm', 'm']

array = [BitVec('a%s'%i,32) for i in range(38)]


sol = Solver()
for i in array:
	sol.add(i>31)
	sol.add(i<128)


try:
	connection = mysql.connector.connect(host="20.233.43.53",user="QUALS", password="QUALS",port = 3306,database='strings')
	if connection.is_connected():
		mySql_Create_Table_Query = """select * from ENI1;"""
		cursor = connection.cursor()
		cursor.execute(mySql_Create_Table_Query)
		
		for x in cursor:
			text = x[0]
		
		num = 0
		
		while(num<len(text)):
			
			text2 = text[num:num+8]

			cursor.execute("select id from ENI where str=\"" + text2 + "\";")
			for x in cursor:
				one_char = x[0]

			if one_char=="d":
				num2 = int(text[num+8:num+8+2],16) 
				num3 = int(text[num+10:num+10+2],16)
				num4 = int(text[num+12:num+12+2],16)
				array[num2] = (array[num3] ^array[num4])
				num+=14

			elif one_char=="G":
				num2 = int(text[num+8:num+8+2],16)
				num3 = int(text[num+10:num+10+2],16)
				array[num2] = (array[num2] + num3)
				num+=12

			elif one_char=="m":
				num2 = int(text[num+8:num+8+2],16)
				num3 = int(text[num+10:num+10+2],16)
				array[num2] = (array[num2] - num3)
				num += 12
			elif one_char=="q":
				num2 = int(text[num+8:num+8+2],16)
				array[num2] = (array[num2]<<1) 
				num += 10
				
		# from C# code
		to_cmp = [197,100,198,233,241,166,207,202,245,217,234,100,114,202,112,93,207,250,95,102,194,95,110,200,104,110,202,93,217,108,95,108,211,189,209,98,104,98]

		for i in range(len(array)):
			sol.add(to_cmp[i]==array[i])
		
		if sol.check()==sat:
			b=sol.model()

			array = [BitVec('a%s'%i,32) for i in range(38)]

			for i in array:
				print(chr(int(str(b[i]))),end='')
		else:
			print(sol.check())



except Error as e:

	print("\n[*] Connection Failed", e)

finally:

	if connection.is_connected():
		cursor.close()
		connection.close()
		print("\n[*] Connection closed")
```

###### flag: `Securinets{79e85a163b62d47e5f666c2a14}`
 
###### tags: `Z3` `vm` `CTF`