# gdb-debugger-note
by <gn02596703>
###### tags: `Jserv`, `gdb`
## What is GDB
GDB is a popular debugger that runs in many unix-like system. It allows you to investigate the inside of the program and supports you find out where the problem happens.

You can see more detailed introduction of GDB in the main page as shown below.
Also, more information is available in [GDB official website](https://www.gnu.org/software/gdb/). 


![](https://i.imgur.com/JlRO0Dk.png)

## Using GDB
The code used for this note is a slightly modified version of the sample code from the tutorial from [this tutorial](http://www.cs.cmu.edu/~gilpin/tutorial/). Similar experiments are conducted following the tutorial.

In the source file, an linkedlist class is implemented. To check the impelmentation is correct or not. We need to write test cases to test our program. So, we can see the test cases in the $main$ function of the source file as follows

```C++
int main (int argc, char **argv) {
  LinkedList<int> *list = new LinkedList<int> ();

  list->insert (1);
  list->insert (2);
  list->insert (3);
  list->insert (4);

  cout << "The fully created list is:" << endl;
  list->print ();

  cout << endl << "Now removing elements:" << endl;
  list->remove (4);
  list->print ();
  cout << endl;

  list->remove (1);
  list->print ();
  cout << endl;

  list->remove (2);
  list->print ();
  cout << endl;

  list->remove (3);
  list->print ();

  delete list;

  return 0;
}
```

We expected to see
```
4
3
2
1
```
from the first print function and then the elemets will be removed one by one in a different order.

Now, we start to compile the program and run the program. Note that we need to add $-g$ flag in the complie command to enable debug information for debugger. We will see the compiled program runs as following.


```console
$ make
g++ -g -Wall -o main main.cpp
$ ls
Makefile  main  main.cpp
$ ./main
Creating Node, 1 are in existence right now
Creating Node, 2 are in existence right now
Creating Node, 3 are in existence right now
Creating Node, 4 are in existence right now
The fully created list is:
4
3
2
1

Now removing elements:
Creating Node, 5 are in existence right now
Destroying Node, 4 are in existence right now
4
3
2
1

Segmentation fault
```

In here, we already find two errors in our program.
* Segmentation fault
* The node deleting function is not implemented correctly.

From the test case output, we can know that the implementation of **remove** function might be wrong. It creates a new node and then deletes a node. In the end, the linkedlist stay unchanged. 

In the following, I will focus on using gdb to find out the the cause of Segmentation Fault. 

---
### gdb executable_file_name

Now, let us start gdb. 
To start using gdb, type $gdb$ $executive$_$name$ in the terminal as follows.

``` console
$ gdb main
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from main...done.
(gdb)
```
We can see now gdb is waiting commands from the terminal.

---
### run
First, we can type $run$ to run the program.
``` console 
(gdb) run
Starting program: /home/ag/Jserv/week_1/gdb/example/main
Creating Node, 1 are in existence right now
Creating Node, 2 are in existence right now
Creating Node, 3 are in existence right now
Creating Node, 4 are in existence right now
The fully created list is:
4
3
2
1

Now removing elements:
Creating Node, 5 are in existence right now
Destroying Node, 4 are in existence right now
4
3
2
1


Program received signal SIGSEGV, Segmentation fault.
0x0000000000401014 in Node<int>::next (this=0x0) at main.cpp:33
33        Node<T>* next () const { return next_; }
(gdb)
```
We can see same results happened. Instead of only telling you segmentation fault, this time, we can also see where the segmentation fault happened. 

It happened at line 33 of main.cpp file. However, we don't know where exactly the start of this error. To address this, we can use **backtrace** command. 

---
### backtrace
With **backtrace** command, the program can tell us where the error happened. Now, type $backtrace$ or $bt$ in the terminal and we can see following. 

``` console
(gdb) backtrace
#0  0x0000000000401014 in Node<int>::next (this=0x0) at main.cpp:33
#1  0x0000000000400f2e in LinkedList<int>::remove (this=0x614c20,
    item_to_remove=@0x7fffffffe19c: 1) at main.cpp:82
#2  0x0000000000400bd6 in main (argc=1, argv=0x7fffffffe2a8) at main.cpp:125
(gdb)
```
Now, we know the error happened when the program exectued   **remove** function in class **LinkedList** at line 125 of **main.cpp**. 

Also, we can see from the terminal, the input variable **item_to_remove** in **remove** function is "1". 
We now know that error is caused by the function **remove**. Now, we can focus on debugging the function. To do this, we will use **breakpoint** to support deeper debugging. 

---
### break

**break** command allows you to set breakpoint in the program. When executing the program under gdb, it will stop when the program runs to the place where a breakpoint is placed. Then, we can investigate what is happening inside the program.
To set a breakpoint, following commands can be used. 
* $break$ $line$ 
* $break$ $filename$:$line$
* $break$ $function$

First, we can set our fist breakpoint at line 33 of main.cpp where segmentation fault happened. 
``` console
(gdb) break 33
Breakpoint 1 at 0x401010: file main.cpp, line 33.
```

Usually, multiple breakpoints will be set. To check the list of all breakpoint. Type $info$ $break$ in the terminal.
```console
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000401010 in Node<int>::next() const
                                                   at main.cpp:33

```

Here, we can see our first breakpoint is listed and it shows that the place of the breakpoint is at line 33 of main.cpp.

Now, we have set our first breakpoint. We re-run the program the program by typing $run$ or $r$ again.

```console
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/ag/Jserv/week_1/gdb/example/main
Creating Node, 1 are in existence right now
Creating Node, 2 are in existence right now
Creating Node, 3 are in existence right now
Creating Node, 4 are in existence right now
The fully created list is:
4

Breakpoint 1, Node<int>::next (this=0x6150b0) at main.cpp:33
33        Node<T>* next () const { return next_; }
(gdb)

```
The program stopped at the place where we put the breakpoint. We can start investigating what is happening inside the program in detail. To do this, usually, we will use command $list$, $print$ and $x$. 

---
### list

Usually, it would be helpful for us to understand what should be happned in the program if we can check lines around the place we put breakpoint. With command $list$, we can print out lines of the source file.

In the console below, we can see it prints 10 lines around the place where we place the breakpoint (line 33). 

```console
(gdb) list
28        }
29
30        // Explanations found on the internet about about the declaration
31        // for const after function name
32        // https://stackoverflow.com/questions/15999123/const-before-parameter-vs-const-after-function-name-c
33        Node<T>* next () const { return next_; }
34        void next (Node<T> *new_next) { next_ = new_next; };
35        const T& value () const { return value_; }
36        void value (const T &value) { value_ = value; }
37
(gdb)

```

If you type $list$ again, following 10 lines will be printed. 

``` console
(gdb) list
38      private:
39        Node ();
40        T value_;
41        Node<T> *next_;
42      };
43
44      template <class T>
45      class LinkedList {
46      public:
47        LinkedList () : head_(0) {};
(gdb)
```

If you want to see specific lines in a source file, you can type $list$ $filename$:$line$. In the following, you can see the console shows 10 lines around the line 27 at main.cpp as we command. 

```console
(gdb) list main.cpp:27
22        }
23        ~Node () {
24          cout << "Destroying Node, "
25               << --number_instantiated
26               << " are in existence right now" << endl;
27          next_ = 0;
28        }
29
30        // Explanations found on the internet about about the declaration
31        // for const after function name
(gdb)
```

### print & x 
During debugging, it would also be helpful for us to know what is the value of the variable at the time. We can do this by the command $print$ or $p$ and command $x$.

Remember that currently the program is on hold before executing the **remove** function in the class **Node**. In the class **Node**, following variables are defined. 
```c++
private:
  T value_;
  Node<T> *next_;
```

With $print$ or $p$ command, we can check the value of variables. From the below, you can see we use $print$ to check the value of **value_** in current **Node** class. Note that, to check the value that a pointer pointed to, we need to add $*$ symbol before the pointer variable as we do in C. Or, the $print$ command will display the address of the pointer 
```console
(gdb) print value_
$8 = 4
(gdb) print next_
$9 = (Node<int> *) 0x615090
(gdb) print *next_ 
$10 = {value_ = 3, next_ = 0x615070}
(gdb)
```

We know that from the test case, currently the linked list is stored with 4-->3-->2-->1. From the output above, we can see that current value is 4 and next value is Node with value 3.

With command $x$, we can check the memory content of specific address. 

```console
(gdb) x next_
0x615090:       0x00000003
(gdb) print next_
$15 = (Node<int> *) 0x615090
```
From the console, we can see the $x$ function show us the value stored in the class **Node** that the pointer *next_* (**0x615090**) points to. 

---
### continue 

From the console, it seems that the place of the breakpoint is not the exact position that casued the error. We need to run the program till the timing where the error happened. 

To do this, we use the $continue$ or $c$ command. Bu typing $continue$, the program will execute following parts till the end of the program or stop when it meets a breakpoint again. 

```console
(gdb) continue
Continuing.
3

Breakpoint 1, Node<int>::next (this=0x615090) at main.cpp:33
33        Node<T>* next () const { return next_; }
(gdb) c
Continuing.
2

```
...
``` console
(gdb) c
Continuing.

Now removing elements:

Breakpoint 1, Node<int>::next (this=0x6150b0) at main.cpp:33
33        Node<T>* next () const { return next_; }

.
.
.

(gdb) c
Continuing.
Creating Node, 5 are in existence right now
Destroying Node, 4 are in existence right now
4

Breakpoint 1, Node<int>::next (this=0x6150d0) at main.cpp:33
33        Node<T>* next () const { return next_; }

.
.
.

(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401014 in Node<int>::next (this=0x0) at main.cpp:33
33        Node<T>* next () const { return next_; }

```
We can see the program keeps running and stops after it meets the breakpoint again. In the end, it reaches the result of Segmentation fault again. 

Now we notice that current breakpoint position is not good for us to debug. So we will remove this breakpoint and set a new one at another position. Of course you can keep current breakpoint and set a new one. The program will stop when it meets each breakpoint.

To remove a break point, type $delete$ $breakpoint$_ $number$ in the console.

```console
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000401010 in Node<int>::next() const
                                                   at main.cpp:33
        breakpoint already hit 11 times
(gdb) delete 1
(gdb) info break
No breakpoints or watchpoints.

```
You can see the breakpoint is removed. Now we set a new breakpoint at the **remove** function.

```console
(gdb) break main.cpp:82
Breakpoint 4 at 0x400f22: file main.cpp, line 82.

(gdb) info break
Num     Type           Disp Enb Address            What
4       breakpoint     keep y   0x0000000000400f22 in LinkedList<int>::remove(int const&) at main.cpp:82
```

Then, we re-run the program.

```console
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/ag/Jserv/week_1/gdb/example/main
Creating Node, 1 are in existence right now
Creating Node, 2 are in existence right now
Creating Node, 3 are in existence right now
Creating Node, 4 are in existence right now
The fully created list is:
4
3
2
1

Now removing elements:
Creating Node, 5 are in existence right now
Destroying Node, 4 are in existence right now
4
3
2
1


Breakpoint 4, LinkedList<int>::remove (this=0x614c20,
    item_to_remove=@0x7fffffffe19c: 1) at main.cpp:82
82            marker = marker->next();
(gdb)
```

The new breakpoint seems to let the program hold at a good position for us to investigate the program. 

First, we can use commands introduced before to check what is happening in the program.

``` console
(gdb) list
77                return 0;
78              }
79            }
80            marker = 0;  // reset the marker
81            temp = marker;
82            marker = marker->next();
83          }
84
85          return -1;  // failure
86        }
(gdb) print marker
$18 = (Node<int> *) 0x0
(gdb) print *marker
Cannot access memory at address 0x0
(gdb) x marker
0x0:    Cannot access memory at address 0x0
```
We notice that **marker** is a pointer points to a Node. Its current value is **0** in the program and we can not access memory at address **0x0**. However, in the program, we are about to access it as it shown in line 82.
```c=
82            marker = marker->next();
```
It seems that we have found out the reason of Segmentation Fault.

---
### step

Now, we think we have found out what causes the Segmentatino Fault happend. We can further confirm by investigating the program step by step with $step$ command.

```console
(gdb) step
Node<int>::next (this=0x0) at main.cpp:33
33        Node<T>* next () const { return next_; }
```
We can see the next step of the program is to run the **next()** function of **marker** and the marker is at address **0x0** (**this=0x0**). However, the program can not access the address **0x0**. If we type $step$ again. We can see Segmentation fault happens.

``` console
(gdb) step

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401014 in Node<int>::next (this=0x0) at main.cpp:33
33        Node<T>* next () const { return next_; }
```

Now we find that **marker** is set to **0x0** in the program and this caused Segmentaiton Fault. From the source code printed with $list$ function above, we can find that the marker is set to 0 at line 80. If we delete the line, the error might disappear. 

---

## Reference
1. [GDB: The GNU Project Debugger](https://www.gnu.org/software/gdb/) - official website
2. [gdb Tutorial](http://www.cs.cmu.edu/~gilpin/tutorial/) - a good introduction and tutorial about using gdb. This note is also based on this tutorial.
3. [Guide to Faster, Less Frustrating Debugging](http://heather.cs.ucdavis.edu/~matloff/UnixAndC/CLanguage/Debug.html) - a comprehensive tutorial about debugging and gdb