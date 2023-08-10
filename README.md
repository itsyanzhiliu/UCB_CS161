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

x86 calling conventions are a set of rules and conventions that dictate how function calls and returns should be managed in assembly language and low-level programming on x86 and x86-64 architectures. 

- How to pass arguments
  - Arguments are pushed onto the stack in reverse order, so func(val1, val2, val3) will place val3 at the highest memory address, then val2, then val1
- How to receive return values
  - Return values are passed in EAX
- Which registers are caller-saved or callee-saved
  - Callee-saved: The callee must not change the value of the register when it returns
  - Caller-saved: The callee may overwrite the register without saving or restoring it

Consider the following C code:
```c
int main(void) {
    foo(1, 2);
}

void foo(int a, int b) {
    int bar[4];
}
```

The compiler would turn the foo function call into the following assembly instructions:
```Assembly
main:
    # Step 1. Push arguments on the stack in reverse order
    push $2
    push $1

    # Steps 2-3. Save old eip (rip) on the stack and change eip
    call foo

    # Execution changes to foo now. After returning from foo:

    # Step 11: Remove arguments from stack
    add $8, %esp

foo:
    # Step 4. Push old ebp (sfp) on the stack
    push %ebp

    # Step 5. Move ebp down to esp
    mov %esp, %ebp

    # Step 6. Move esp down
    sub $16, %esp

    # Step 7. Execute the function (omitted here)

    # Step 8. Move esp
    mov %ebp, %esp

    # Step 9. Restore old ebp (sfp)
    pop %ebp

    # Step 10. Restore old eip (rip)
    pop %eip
```

### 2.2 Memory Safety Vulnerabilities

- Buffer Overflow:
  - This occurs when a program writes more data into a buffer (e.g., an array) than it can hold. This can overwrite adjacent memory, leading to crashes, data corruption, and potential code execution.

- Use-After-Free:
  - A use-after-free vulnerability occurs when a program continues to use a memory location after it has been freed (released). Attackers can exploit this by manipulating memory, potentially leading to crashes or arbitrary code execution.

- Dangling Pointer:
  - Similar to use-after-free, a dangling pointer vulnerability happens when a pointer points to a memory location that has been deallocated. This can lead to crashes or unexpected behavior.

- Memory Leak:
  - Memory leaks occur when a program fails to release memory it no longer needs. Repeated memory leaks can lead to reduced system performance and resource exhaustion.

- Null Pointer Dereference:
  - This vulnerability happens when a program tries to access memory through a null (invalid) pointer. It can lead to crashes and denial-of-service situations.

### 2.3 Mitigating Memory-Safety Vulnerabilities

- Memory-safe languages
  - Using a memory-safe language (e.g. Python, Java) stops all memory safety vulnerabilities.
  - Why use a non-memory-safe language?
    - Commonly-cited reason, but mostly a myth: Performance
    - Real reason: Legacy, existing code
- Writing memory-safe code
  - Carefully write and reason about your code to ensure memory safety in a non-memory-safe language
- Building secure software
  - Use tools for analyzing and patching insecure code
  - Test your code for memory safety vulnerabilities
  - Keep any external libraries updated for security patches
- Mitigation: Non-executable pages
  - Make portions of memory either executable or writable, but not both
  - Defeats attacker writing shellcode to memory and executing it
  - Subversions
    - Return-to-libc: Execute an existing function in the C library
    - Return-oriented programming (ROP): Create your own code by chaining together small gadgets in existing library code
- Mitigation: Stack canaries
  - Add a sacrificial value on the stack. If the canary has been changed, someone’s probably attacking our system
  - Defeats attacker overwriting the RIP with address of shellcode
  - Subversions
    - An attacker can write around the canary
    - The canary can be leaked by another vulnerability (e.g. format string vulnerability)
    - The canary can be brute-forced by the attacker
- Mitigation: Pointer authentication
  - When storing a pointer in memory, replace the unused bits with a pointer authentication code (PAC). Before using the pointer in memory, check if the PAC is still valid
  - Defeats attacker overwriting the RIP (or any pointer) with address of shellcode
- Mitigation: Address space layout randomization (ASLR)
  - Put each segment of memory in a different location each time the program is run
  - Defeats attacker knowing the address of shellcode
  - Subversions
    - Leak addresses with another vulnerability
    - Brute-force attack to guess the addresses
- Combining mitigations
  - Using multiple mitigations usually forces the attacker to find multiple vulnerabilities to exploit the program (defense-in-depth)



## 3. Cryptography

### 3.1 Intro

The most basic building block of any cryptographic system (or cryptosystem) is the key. The key is a secret value that helps us secure messages. Many cryptographic algorithms and functions require a key as input to lock or unlock some secret value.
- Symmetric key is shared between the sender and receiver for both encryption and decryption processes.
- Asymmetric key involves two distinct keys: a public key and a private key.

In cryptography, there are three main security properties that we want to achieve.

- Confidentiality is the property that prevents adversaries from reading our private data. If a message is confidential, then an attacker does not know its contents.

- Integrity is the property that prevents adversaries from tampering with our private data. If a message has integrity, then an attacker cannot change its contents without being detected.

- Authenticity is the property that lets us determine who created a given message. If a message has authenticity, then we can be sure that the message was written by the person who claims to have written it.


### 3.2 Kerckhoff’s Principle
Kerckhoffs's Principle, often referred to as "Kerckhoffs's Second Law," is a fundamental concept in the field of cryptography.

Kerckhoffs's Principle can be summarized as follows:

> A cryptosystem should remain secure even if everything about the system, except the key, is public knowledge.

In other words, the security of a cryptographic system should not rely on keeping the details of the encryption algorithm or system design secret. Instead, the security should be based on the secrecy of the cryptographic keys used in the system. The algorithm itself can be widely known and publicly analyzed without compromising the security of the system.

This principle has several implications and benefits:

- Open Design: Cryptographic algorithms and protocols should be open to public scrutiny. By allowing experts to analyze the algorithm, potential vulnerabilities can be identified and addressed more effectively.

- Key Management: The focus should be on securely managing the cryptographic keys. Even if an adversary knows the algorithm and the system's inner workings, they should not be able to decipher encrypted messages without the key.

- Security Through Simplicity: A complex and obscure encryption algorithm is not necessarily more secure. Simplicity in design is often favored, as it reduces the risk of hidden vulnerabilities.

- Interoperability: If the algorithm is public and widely accepted, it promotes interoperability between different systems and platforms.

- Longevity: Even if an algorithm becomes obsolete, the security of the system should remain intact as long as the keys are secure.


### 3.3 Symmetric-Key Encryption

#### 3.3.1  IND-CPA Security
In an IND-CPA attack, the adversary is allowed to choose two plaintexts and receive the encryption of one of them. The goal of the adversary is to determine which of the two plaintexts was encrypted. The encryption scheme is considered secure under IND-CPA if the adversary's success in distinguishing the encrypted plaintext from random noise is limited to a negligible advantage, even when the adversary can choose the plaintexts.

Formally, an encryption scheme is IND-CPA secure if, for every efficient adversary A, the advantage of A in distinguishing the encryption of two chosen plaintexts is negligible. In the IND-CPA game: The scheme is secure even if Eve can win with `probability ≤ 1/2 + Ɛ`, where Ɛ is negligible.

Advantage of the adversary should be exponentially small, based on the security parameters of the algorithm. Example: For an encryption scheme with a k-bit key, the advantage should be `$O((1/2)^{k})$`.For now, 280 is a reasonable threshold, but this will change over time!

#### 3.3.2 One-time Pads
Here's how the one-time pad works:

1. Key Generation:
Generate a truly random key of the same length as the plaintext. The key is used only once and should never be reused.

2. Encryption:
To encrypt the plaintext, perform a bitwise XOR operation between the plaintext and the key. This generates the ciphertext.

3. Decryption:
To decrypt the ciphertext, perform the same XOR operation between the ciphertext and the key. This recovers the original plaintext.


One-time pads are seldom used in practice due to the challenges associated with generating, managing, and distributing truly random keys. 

#### 3.3.3 Block Ciphers
Here's an overview of how block ciphers work:

1. Key Generation:
A secret key is generated by a trusted entity and shared securely between the sender and receiver. The length of the key depends on the block cipher algorithm being used.

2. Encryption:
Block ciphers operate on fixed-size blocks of plaintext. The plaintext block is combined with the secret key using a series of substitution and permutation operations. The output is the corresponding block of ciphertext.

3. Decryption:
To decrypt the ciphertext, the same secret key is used in reverse. The ciphertext block is processed through a series of decryption operations, effectively reversing the encryption process, resulting in the original plaintext block.

Block ciphers are used in various applications, such as secure communication, data protection, file encryption, and more. They form the basis for many cryptographic protocols and systems.

### 3.4 Cryptographic Hashes

Cryptographic hash functions have several useful properties. The most significant include the following:

#### 3.4.1 One-way Property

The hash function can be computed efficiently: Given `x`, it is easy to compute `H(x)`. However, given a hash output `y`, it is infeasible to find any input `x` such that `H(x) = y`. (This property is also known as "preimage resistant.") Intuitively, the one-way property claims that given an output of a hash function, it is infeasible for an adversary to find any input that hashes to the given output.

#### 3.4.2 Second Preimage Resistance

Given an input `x`, it is infeasible to find another input `x'` such that `x' ≠ x` but `H(x) = H(x')`. This property is closely related to preimage resistance; the difference is that here the adversary also knows a starting point, `x`, and wishes to tweak it to `x'` in order to produce the same hash—but cannot. Intuitively, the second preimage resistant property claims that given an input, it is infeasible for an adversary to find another input that has the same hash value as the original input.

#### 3.4.3 Collision Resistance

It is infeasible to find any pair of messages `x`, `x'` such that `x' ≠ x` but `H(x) = H(x')`. Again, this property is closely related to the previous ones. Here, the difference is that the adversary can freely choose their starting point, `x`, potentially designing it specially to enable finding the associated `x'`—but again cannot. Intuitively, the collision resistance property claims that it is infeasible for an adversary to find any two inputs that both hash to the same value. While it is impossible to design a hash function that has absolutely no collisions since there are more inputs than outputs (remember the pigeonhole principle), it is possible to design a hash function that makes finding collisions infeasible for an attacker.

### 3.5 Message Authentication Codes (MACs)

A Message Authentication Code (MAC) is a technique used to generate a fixed-length tag for an arbitrary-length message, providing data integrity and authenticity. Here are the key components and properties of a MAC system:

- KeyGen() → K: Generate a key K
- MAC(K, M) → T: Generate a tag T for the message M using key K
  - Inputs: A secret key and an arbitrary-length message
  - Output: A fixed-length tag on the message
- Properties
  - Correctness: Determinism
    - Note: Some more complicated MAC schemes have an additional Verify(K, M, T) function that don’t require determinism, but this is out of scope
  - Efficiency: Computing a MAC should be efficient
  - Security: EU-CPA (existentially unforgeable under chosen plaintext attack)

### 3.6 Authenticated Encryption
- Authenticated encryption: A scheme that simultaneously guarantees confidentiality and integrity (and authenticity) on a message
- First approach: Combine schemes that provide confidentiality with schemes that provide integrity and authenticity
  - MAC-then-encrypt: `Enc(K1, M || MAC(K2, M))`
  - Encrypt-then-MAC: `Enc(K1, M) || MAC(K2, Enc(K1, M))`
  - Always use Encrypt-then-MAC because it's more robust to mistakes
- Second approach: Use AEAD encryption modes designed to provide confidentiality, integrity, and authenticity
  - Drawback: Incorrectly using AEAD modes leads to losing both confidentiality and integrity/authentication




## 4. Web Security









## 5. Network Security

## 6. Miscellaneous Topics
