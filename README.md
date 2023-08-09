# UCB CS 161

Everything I found interested from [Computer Security](https://www2.eecs.berkeley.edu/Courses/CS161/).

## 1. Security Principle
- Know your threat model
  - **Threat model**: A model of who your attacker is and what resources they have
  - It all comes down to people: The attackers
 
- Consider human factors
- Security is economics
- Detect if you can’t prevent
- Defense in depth
- Least privilege
- Separation of responsibility
- Ensure complete mediation
  - Ensure that every access point is monitored and protected
  - The time of check to time of use (**TOCTTOU**) vulnerability usually arises when enforcing access control policies such as when using a reference monitor.   
- Don’t rely on security through obscurity
- Use fail-safe defaults
  - Construct systems that fail in a safe state, balancing security and usability 
- Design in security from the start


## 2. Memory Safety

### 2.1 x86 Assembly and Call Stack

#### 2.1.1 Four main steps to running a C program

- The **compiler** translates your C code into assembly instructions ((RISC-V, x86))
- The **assembler** translates the assembly instructions from the compiler into machine code (raw bits)
- The **linker** resolves dependencies on external libraries
- The **loader** sets up an address space in memory and runs the machine code instructions in the executable

#### 2.1.2 C memory layout

At runtime, the loader tells your OS to give your program a big blob of memory. On a 32-bit system, the memory has 32-bit addresses. Each address refers to one byte, which means you have  $2^{32}$ bytes of memory
- **_x86 Memory Layout_**
  - **Code**
    - The program code itself (also called “text”)
  - **Data**
    -  Static variables, allocated when the program is started
  - **Heap**
    - Dynamically allocated memory using malloc and free. As more and more memory is allocated, it grows upwards
  - **Stack**
    - Local variables and stack frames as you make deeper and deeper function calls, ti grows downwards
- Little-endian words
  - x86 is a little-endian system. For example, here we are storing the word 0x44332211 in memory: Note that the least significant byte 0x11 is stored at the lowest address, and the most significant byte 0x44 is stored at the highest address.   

#### 2.1.3 Register 

In addition to the  $2^{32}$ bytes of memory in the address space, there are also registers, which store memory directly on the CPU. Each register can store one word (4 bytes). Unlike memory, registers do not have addresses. Instead, we refer to registers using names. There are three special x86 registers that are relevant for these notes:

- _eip_ is the instruction pointer, and it stores the address of the machine instruction currently being executed. In RISC-V, this register is called the PC (program counter).
- _ebp_ is the base pointer, and it stores the address of the top of the current stack frame. In RISC systems, this register is called the FP (frame pointer).
- _esp_ is the stack pointer, and it stores the address of the bottom of the current stack frame. In RISC-V, this register is called the SP (stack pointer).

Note that the top of the current stack frame is the highest address associated with the current stack frame, and the bottom of the stack frame is the lowest address associated with the current stack frame. 

Since the values in these three registers are usually addresses, sometimes we will say that a register points somewhere in memory. This means that the address stored in the register is the address of that location in memory. 

#### 2.1.4 x86 syntax

 - Register references are preceded with a percent sign %
    - Example: %eax, %esp, %edi
  
 -  Immediates are preceded with a dollar sign $
    - Example: $1, $161, $0x4
  
 - Memory references use parentheses and can have immediate offsets
    - Example: 8(%esp) dereferences memory 8 bytes above the address contained in ESP

#### 2.1.5 Stack Layout

- When your code calls a function, space is made on the stack for local variables. This space is known as the stack frame for the function. The stack frame goes away once the function returns. 
- The stack starts at higher addresses. Every time your code calls a function, the stack makes extra space by growing down
- To keep track of the current stack frame, we store two pointers in registers
  - The EBP (base pointer) register points to the top of the current stack frame
  - The ESP (stack pointer) register points to the bottom of the current stack frame

Sometimes we want to remember a value by saving it on the stack. There are two steps to adding a value on the stack. First, we have to allocate additional space on the stack by decrementing the `esp`. Then, we store the value in the newly allocated space. The x86 `push` instruction does both of these steps to add a value to the stack.

We may also want to remove values from the stack. The x86 `pop` instruction increments `esp` to remove the next value on the stack. It also takes the value that was just popped and copies the value into a register.

#### 2.1.6 Calling convention







## 3. Cryptography

## 4. Web Security

## 5. Network Security

## 6. Miscellaneous Topics
