Include/opcode.h
- headerfile that defines all the bytecode
- the below numbers are from python2:
- 100: LOAD_CONST
- 90: HAVE_ARGUEMENT
	- everything above 90 takes an argument
	- everything below takes NO argument

Python/ceval.c


Interpreter
- python
 	- like if you just write "python" in bash
- can run a prompt or write a textfile and run it
	- python test.py

Compiling
- in python:
	>>> c = compile(open('test.py').read(), 'test.py', 'exec')                                                  
	<code object <module> at 0x106115d80, file "test.py", line 1>                                                       
- compile(source, filename, mode[, flags[, dont_inherit]]) -> code object
	- exec to compile a module (one .py)
- python interpreter does this for you
- code object
	- the compiled code
- c.co_code
	- inside of c
	- string
	- b'\x80\x00\x00\x00Z\x01p\x00Z\x02p\x01Y\x00Y\x01*\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00p\x02Y\x03\x1f\x00Y\x- 50                                                                                                                  


- these are compiled source code into bytes (string)

python -m dis test.py                                                            
  0           RESUME                   0                                                                                               
                                                                                                                                       
  1           LOAD_SMALL_INT           1                                  load small int 1                                                 
              STORE_NAME               0 (x)                              store into x                                             
                                                                                                                                       
  2           LOAD_SMALL_INT           2                                  load small int 2                                                 
              STORE_NAME               1 (y)                              store into y                                             
                                                                                                                                       
  3           LOAD_NAME                0 (x)                                                                                           
              LOAD_NAME                1 (y)                                                                                           
              BINARY_OP                0 (+)                                                                                           
              STORE_NAME               2 (z)                                                                                           
                                                                                                                                       
  4           LOAD_NAME                3 (print)                                                                                       
              PUSH_NULL                                                                                                                
              LOAD_NAME                2 (z)                                                                                           
              CALL                     1                                                                                               
              POP_TOP                                                                                                                  
              LOAD_CONST               1 (None)                                                                                        
              RETURN_VALUE                     

	- disassembler
	- bytecodes for each line
	- every line can have more than one bytecode
	- instruction name
	- optional arguments
	- the bytecodes in the string array match the bytecodes here based on opcode.h

dis
- inside /Lib
- dis is part of the standard library
- Lib/dis.py
	- a python file 
- -m loads the module (dis) and runs the main function


interpreter
- "stack machine"
- goes through output of dis test.py and runs that output's line by line
- loads it into a value stack
	- i.e. pushes LOAD_CONST's object 1 onto the stack
	- STORE_NAME takes whats at the top of the stack, pops it, and stores it into a piece of memory (x->1). nothing is on the stack now. x points to 1.
	- LOAD_CONSST pushes 2 onto the value stack
	- STORE_NAME pops the 2 off, y->2 (y points to 2 in memory)

	- LOAD_NAME the 1 gets pushed onto the stack, NOT the x. It loads whatever x CONTAINS, grabs its value and pushes it onto the stack.
	- the 1 is NOT copied. there are just two things that reference it, the value stack and x
		- "reference counting"
		- the 1 object has a reference count of 2
		- when it is popped off, the reference count decrements to 1
		- if there are no references, the 1 object basically is floating with no way to access it, so the garbage collector deallocates the memory to use it for something else

	- LOAD_NAME pushes 2 onto the stack (the value of y)
	- BINARY_ADD goes to the value stack, finds the top 2 things, and adds them. After it executes, the stack pops the 2 things and pushes the result (3) back on. 
	- STORE_NAME pops the 3 off the stack and points z to it (z->3)

	- LOAD_NAME loads 3 onto the stack (value of z)
	- PRINT_ITEM pops the top thing from the stack and prints it to the console
	- PRINT_NEWLINE prints a newline bc python print doesnt typically print newline
	- LOAD_CONST (None) loads an object called None onto the value stack
	- RETURN_VALUE pops None off the stack and returns it to its caller


PyObject *PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
- the main interpreter
- where everything happens
- arguments
	- pointer to PyFrameObject
	- throwflag is basically an exception
	- returns a PyObject * 
- PyObject is just an object in python
- takes a frame, does something, and returns an object
- a frame is conceptually a frame in the stack
- lots of macros (#define ___)
- if (f = NULL) return NULL;
	- if frame is null return null
- co = f->f_code;
	- co is code
	- grab it from frame
- for (;;) {...}
	- infinite loop
	- goes throguh bytecode one byte at a time and gets opcode
	- opcode = NEXTOP();
	- grab the argument
		- if (HAS_ARG(opcode)) oparg = NEXTARG();
	          switch(opcode)
		- same for other opcodes (but diff executions)
		- case RETURN_VALUE
			- pops value off stack, goto fast_block_end
		- fast_block_end:
			if (why != WHY_NOT) break;
- return retval
