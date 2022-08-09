# Qiling labs

```python
from binascii import unhexlify
from logging import getLogger
from qiling import *
from qiling.const import *
from qiling.os import *
from qiling.os.mapper import *
from rev import *
from pwn import *

#ql.add_fs_mapper("file",my_class())
#ql.hook_address(my_function,address)
#ql.os.set_api('function_name',my_function)
#ql.os.set_syscall("XXX_SYSCALL_XXX",my_function,QL_INTERCEPT.EXIT)


def set_eax_0(ql):
	ql.arch.regs.eax=0

def set_eax_1(ql):
	ql.arch.regs.eax = 1

def set_edx_1(ql):
	ql.mem.write(ql.arch.regs.rdx,b'\x01')

def change_parms(ql):
	ql.arch.regs.rsi = 0x696C6951
	ql.arch.regs.rcx = 0x614C676E
	ql.arch.regs.rax = 0x20202062


def my_lower(ql,*args):
	ql.arch.regs.rax = ql.arch.regs.rdi

def my_sleep(ql,*args):
	ql.arch.regs.rdi = 0

def my_getrandom(ql, *args):
	ql.mem.write(args[0], b'\x01'*32)

class my_urand(QlFsMappedObject):

	def read(self,size):
		if size==1:
			return b'\x00'
		return b'\x01'*size
	
	def close(self):
		return 0 

class my_cmdline(QlFsMappedObject):

	def read(self,size):
		return b'qilinglab'

	def close(self):
		return 0

def my_uname(ql,*args):
	buf = ql.arch.regs.rdi 
	ql.mem.write(buf,b'QilingOS')
	ql.mem.write(buf+(65*3),b'ChallengeStart')

def challenge1(ql):
	ql.mem.map(0x1337//4096*4096,4096)
	ql.mem.write(0x1337,p32(1337))

def challenge2(ql):
	ql.os.set_syscall('uname',my_uname,QL_INTERCEPT.EXIT)

def challenge3(ql):
	ql.os.set_syscall('getrandom',my_getrandom,QL_INTERCEPT.EXIT)
	ql.add_fs_mapper('/dev/urandom',my_urand())

def challenge4(ql):
	ql.hook_address(set_eax_1,0x000555555554E43)

def challenge5(ql):
	ql.os.set_api('rand',set_eax_0,QL_INTERCEPT.EXIT)

def challenge6(ql):
	ql.hook_address(set_eax_0,0x000555555554F16)

def challenge7(ql):
	ql.os.set_api('sleep',my_sleep,QL_INTERCEPT.ENTER)

def challenge8(ql):
	ql.hook_address(set_edx_1,0x000555555554FB1)

def challenge9(ql):
	ql.os.set_api('tolower',my_lower,QL_INTERCEPT.EXIT)

def challenge10(ql):
	ql.add_fs_mapper('/proc/self/cmdline',my_cmdline())

def challenge11(ql):
	ql.hook_address(change_parms,0x000555555555195)


def run_sandbox(path, rootfs, verbose):
	ql = Qiling(path, rootfs, verbose=QL_VERBOSE.DEBUG, console=False)

	challenge1(ql) # write to memory
	challenge2(ql) # hijack syscall on exit
	challenge3(ql) # hijack file object and syscall
	challenge4(ql) # hooking an address/instruction
	challenge5(ql) # hijacking an api on exit
	challenge6(ql) # hooking an address/instruction
	challenge7(ql) # hijacking an api on entry
	challenge8(ql) # hooking an address/instruction
	challenge9(ql) # hijacking an api on exit
	challenge10(ql) # hijack file object 
	challenge11(ql) # hooking an address/instruction
	ql.run()



def usage():
	print("Script.py <arch> <file>")

if __name__ == "__main__":
	if len(sys.argv) < 2 or len(sys.argv) > 4:
		usage()
	else:
		arch = sys.argv[1]
		file = sys.argv[2]
		if arch == "x86":
			run_sandbox([file], "../rootfs/x86_linux", QL_VERBOSE.DEBUG)
		elif arch == "x64":
			run_sandbox([file], "../rootfs/x8664_linux", QL_VERBOSE.DEBUG)



```