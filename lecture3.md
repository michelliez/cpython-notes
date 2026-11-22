PyEval_EvalFrameEx
- what python does to each frame
- macros for accessing the value stack
- co = f->f_code;
	- code object from function
- for(;;)
	- for each iteration, it grabs the next opcode and its arguments (if its opcode > 90)
	- switch statement (condition for every opcode)
- also has an environment

Tracing new test.py
x = 10                                                                                                              
                                                                                                                    
def foo(x):                                                                                                         
        y = x * 2                                                                                                   
        return bar(y)                                                                                               
                                                                                                                    
def bar(x):                                                                                                         
        return y = x / 2                                                                                                   
                                                                                                                    
z = foo(x)  


Global Frames
- x = 10
	- global scope has x pointing to 10
- def foo(x)
	- defines the name foo to point to a function object with the code to foo
	- binds name foo to actual function
- def bar(x)
	- defines name bar to bind to function object

No longer global, foo() frame
- z = foo(x)
	- jumps into the actual foo function
	- creates another frame
	- x is now local with value 10
- y = x * 2 (inside foo)
	- uses local x to get local y
	- y -> 20

bar() frame
- return bar(y)
	- jumps into actual bar function
- y = x / 2 (inside bar)
	- was called with y, which was 20, so bar local x-> 20
	- 20/2 is 10, assigned to y
	- y -> 10
	- returns the value of y, which is 10
	

Popping
- bar() frame is popped
	- sends return value (10) to foo()
- foo() frame is popped
	- sends return value (10) to global frame
- global frame receives return value into z
	- z->10

- functions must also have an environment


DISASSEMBLY

  0           RESUME                   0

  1           LOAD_SMALL_INT          10
              STORE_NAME               0 (x)

  3           LOAD_CONST               1 (<code object foo at 0x1098c3230, file "test.py", line 3>)
              MAKE_FUNCTION
              STORE_NAME               1 (foo)

  7           LOAD_CONST               2 (<code object bar at 0x10982b040, file "test.py", line 7>)
              MAKE_FUNCTION
              STORE_NAME               2 (bar)

 11           LOAD_NAME                1 (foo)
              PUSH_NULL
              LOAD_NAME                0 (x)
              CALL                     1
              STORE_NAME               3 (z)
              LOAD_COMMON_CONSTANT     7 (None)
              RETURN_VALUE

Disassembly of <code object foo at 0x1098c3230, file "test.py", line 3>:
  3           RESUME                   0

  4           LOAD_FAST_BORROW         0 (x)
              LOAD_SMALL_INT           2
              BINARY_OP                5 (*)
              STORE_FAST               1 (y)

  5           LOAD_GLOBAL              1 (bar + NULL)
              LOAD_FAST_BORROW         1 (y)
              CALL                     1
              RETURN_VALUE

Disassembly of <code object bar at 0x10982b040, file "test.py", line 7>:
  7           RESUME                   0

  8           LOAD_FAST_BORROW         0 (x)
              LOAD_SMALL_INT           2
              BINARY_OP               11 (/)
              STORE_FAST               1 (y)

  9           LOAD_FAST_BORROW         1 (y)
              RETURN_VALUE


Stack of Frame Objects
- implemented as a linked list in C
- typedef struct _frame {..}
- Frame Stack contains global, foo, bar, that link back to each other with the struct _frame *f_back frield
- so the frame object contains a pointer to the LAST frame object (bar)
- bar points to the object before it (foo)
- foo points to global
- the frame stack also has a globals locals and valuestack, so each frame has its own private (local) valuestack that it uses for its computations. it is cleared when the frame is exited (so no memory is leaked


call_function
- calls the function
- leaves the return value on the stack
- next opcode grabs it off the stack and does something
- argument is the number of things to pop off the stack
- checks to see if the function is in C of python

