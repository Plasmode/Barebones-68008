barebones 68008, BB68008, is a simple computer consisted of 68008 CPU, 128K/512K RAM, and 22V10 as boot ROM and glue logic. It is similar to barebones Z80 and 6502 in concept and prototyped using a BB6580 board.
Like barebones 6502 and Z80, BB68008 is bootstrapped over serial port every power cycle. The serial port is bit-bang 57600-N82. There is also a page flipflop that switches out ROM when location $40 is accessed. Most of 22V10 is devoted to bootstrap ROM for 68008. The bootstrap code is 48 bytes so which is near the limit of 22V10 sum-of-product matrix.
This is the serial bootstrap algorithm:  
At power up, 22V10-based ROM provides the program for 68008 until location $40 is access which will switch out the ROM and replaced with RAM. While ROM is enabled, the RAM is enabled and writable so the role of ROM bootstrap is to decode incoming serial data and write to itself starting from location $0. When location $40 is written, the ROM is replaced with RAM but the bootstrap program remained the same. When 512 bytes of serial data are received, the program starts at location $40.
SerRx    equ $fffffff4
page     equ $fffffffe
         org $0
;serial bootstrap with 22V10
;serial receive is hooked up to D[7]
;CPU clock is 8MHz
         dc.l $8000
         dc.l start
         org $14
start:           
         lea $0,a0                 ;start serial load from $0  
         move.w #$200,d3            ;write 512 bytes into RAM
setup:         
         moveq #7,d1               ;8 bits in serial receive          
startBit:         
         move.b SerRx,d2   
         bmi startBit
         movem d0-d7/a0-a3,(a7)     ;wait 1.5 bit time     
getSer:         
         asl SerRx                  ; shift into extend bit
         roxr.b #1,d2                 ; D2 holds the received value       
         movem d0-d7,(a7)           ;wait 1 bit time
         dbra d1,getSer
         move.b d2,(a0)+
         dbra d3,setup  

         bra.s bootend               ;prefetch this instruction
         nop
bootend: 
;this is location $40        
;loaded program resume execution here.
         end  0           
