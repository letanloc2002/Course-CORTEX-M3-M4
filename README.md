# Course-CORTEX-M3-M4

------------------- Features of the cortex M4 processor --------------------

1. operational mode of the processor
2. the different access levels of the processor
3. the register set of
4. the banked stack design of the processor
5. exceptions and exception handling
6. interupt handling
7. bus interfaces and bus matrix
8. memory architecture of the processor, bit bading, memory
9. endinaness
10. aligned and unaligned data transfer
11. bootloader and IAP.

## --------------------- Operational Mode of the cortex M Processor.

1. Thread mode
2. Handler mode
3. all you application code will excute under "Thread mode" of the processor. This is also called as "User mode"
4. All the exceptionn handlers or interrupt handlers will run under the "Handler mode" of the processor.
5. Processor always starts with "Thread mode"
6. whenever the core meets this the system exception or any extern interrupts then the core will change its mode to handler mode in order to service the ISR associated with that system exception or the interrupt

---

## Operation modes and access levels

---

- when you're in "handler mode", you will can access all resources, you can change anything you want
- \*\*\* ISR_NUMBER nếu bằng 0 thì sẽ đang ở trong chế độ "Thread mode", nếu khác 0 thì sẽ ở trong chế độ handler, tùy vào giá trị ISR_NUMBER sẽ xác định được là handler gì

## Access levels of the processor

1.  processor offers 2 access levels

-privileged access level (PAL)
-non-privileged access level (NPAL)
-by default, your code wil run in PAL
-when in thread mode can change to NPAL, but when in NPAL you can change back PAL, but its imposible when you in handler mode
-handler mode code excution is always with PAL

-use the Control register of the processor if you want to switch between the access level.
\*\*\* CORE REGISTERS

- R0->R12 registers are for general purpose.(32bit)(data upgrade, data storage)
- the register R13 are SP (Stack pointer) use to track the memory, trỏ đến vùng nhớ để lưu trữ
  - PSP: processor stack pointer
  - MSP: main stack pointer

-Link register (R14), It stores the return information for subrountines, function call and exceptions. On the reset, the processor sét the LR value to 0xFFFFFFFF

- Khi chạy chương trình với fun1() và fun2(), với fun1() call fun2() thì tại thời điểm call thì LR(R13) sẽ được nạp pointer địa chỉ của cậu lệnh tiếp theo trong fun1() và PC sẽ được nạp địa chỉ của fun2()
  -Program counter (PC)(R15). It contains the current program address. Reset, PC is loaded reset vector, which address 0x00000004, trỏ đến địa chỉ của câu lệnh kế tiếp

- Program status register combine:
  -application program status register (APSR) contains conditional flag
  -interrrupt program status resgister(IPSR)
  -excution program status resgister(EPSR)

- T bit is used to change between ARM-ISA(0) and Thumb(1) state but in cortex Mx just support Thumb state so if set T bit is 0, that's not illegal

- R0-R15 don't have unique address so they aren't a part of memory map, if you want to access it, you have to use assembly code.

-------------ARM GCC inline assembly code usage ---------------

- \_\_asm volatile(code:output operand list: input operand list: clobber list);
- \_\_asm volatile("MOV R0,R1");
- \_\_asm volatile("MOV R0,R1":::);

  --------------Reset sequence of the processor------------------

1. When you reset the processor, The PC iss loaded with the value 0x0000_0000
2. Then rpocessor reads the value memory location 0x0000_0000 in to MSP
   MSP == value@0x0000_0000
   MSP is a Main Stack pointer register
   that means, processor first initializes the Stack pointer
3. After that, Processor reads the value @ memory location 0x0000_0004 in to PC
   That value is actually address of the reset handler
4. PC jumps to the reset handler.
5. A reset handler is just a C or assembly function written by you to carry out any initializations required.
6. From reset handler you call your main() function of the application
   -----------------Access level and T bit -----------------

1.Various ARM processors support ARM-Thumb interworking, that mean the ability to switch between ARM and Thumb state
2.The processor must be in ARM state to excute instructions which are from ARM ISA anf the processor must be in Thumb state to excute instructions of Thumb ISA.
3.If 'T' bit of the EPSR is set (1), rpocessor thinks that the next instruction which it is about to excute is from Thum ISA
4.If 'T' bit of the EPSR is set (0), rpocessor thinks that the next instruction which it is about to excute is from ARM ISA
5.The cortex Mx processor does not support the ARM state.
Hence, the value of 'T' bit must always be 1.
Failing to maintain this is illegal and this will result in the "Usage fault" exception.
6.The lsb(bit 0) of the program counter (PC) is linked to this 'T' bit.
When you load a value or an address in to PC the bit(0) of the value is loaded into the T-bit
7.Hence, any address you place in the PC must have its 0th bit as 1.
this is usually taken care by the compiler and programmers need not to worry most of the time. 8. This is the reason why you see all vector addressed are incremented by 1 in the vector table

---------------------- memory Map and Bus interfaces----------

- memory map explains mapping of different peripheral registers and memories in the processor addressable memory location range
- the processor, addressable memory location range, depends upon the size of the address bus
- the mapping of different regions in the addressable memory location range is called 'memory map'

\*\*\* Bus interface

- AHB Lite buss iss mainly used for the amin buss interfaces
- APB buss is used for PPB access and some on-chip peripheral access using an AHB-APB brigde
- AHB Lite buss majorly used for high-speed communication with peripherals that demand high operation speed
- APB buss is used for low- speed communication compared to AHB. Mosst off the peripherals which don't require operation speed are connected this bus
- D-bus and I-bus is used to communicate wwith memory code to load instruction and data \* Bit Banding

\*\* Stack memory

- Stack memory is part of the main memory (Internal RAM or external RAM) reserved for the temporary storage of data
- Mainly used during function, interrupt/exception handling
- stack memory iss accessed in last in first out fashion (LIFO)
- The stack can be accessed using PUSH and POP instructiongs or using any memory manipulation instruction (LD,STR)
- The stack is traced using a stack pointer (SP) register. PUSH and POP instructiong affect (Decrement or increment) stack pointer register (Sp,R13)
- the temporary storage of processor register values
- the temporary storage of local variables of function
- during system exception or interrupt, stack memory will be used to save the context of the currently executing code

-Stack operation model: in ARM cortex Mx processor stack consumption model iss Full Descending

----MSP, PSP summary

1.  Physically there are 2 stack pointer registers in Cortex M Proxessors
2.  main Stack Pointer (MSP): thiss iss the default stack pointer used after reset, and is used for all exception/interrupt handlers and for codes which run in thread mode.
3.  Process Stack Pointer (PSP): this is an alternate stack pointer that can only be used in thread mode. It is usually used for application task in embedded systems and embedded 1.
4.  After power-up, processor automatically initalizes the MSP by reading the first location of the vector table

-----Changing SP:

- To access MSP anf PSP in assembly coded, you can use the MSR and MRS instructiongs
- In a C program you can wirte a naked function ('C' like assembly function which doesn't have epilogue and prologue sequences) to change the currently selected stack pointer

\*\*\* function CALL and AAPCS standard
\*\*\* System exception control registers

- The address map of the Private Peripheral Bus (PPB)

- NVIC discission

* NVIC is one of the peripheral of the Cortex M processor core
* It is used to configure the 240 interrupts
* using NVIC registers you can enable/ disable/Pend various interrrupts and read the status of the active and pending interrupts
* you can config the priority and priority grouping of various interrupt
* Its is called as "Nest" because, it supports Pre-empting a lower priority interrupt handler when higher priority interrrupt arrives
  \*\* what is priority>?
  STM32F4 hass 16 different priority levels
  \*\* pre-emtp priority and sub priority
  if value of pre-emtp priority is equal, then sub priority will be consider
  if 2 interrupts of the same pre-empt priority and suv priority hit the processor at the same time, Interrupt with the lowest IRQ will be allowed first
  \*\* what is a fault?

- the fault is an exceptiongenerated by the processor to indicate an error.
  \*\* why fault happens

\*\*\*\* Sheduling policy selection
