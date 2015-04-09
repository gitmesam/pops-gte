# Introduction #

Experimental wiki-based reversing (no SVN).

## Instruction encoding ##

```

    AVSZ3  cop2 0x158002d = 0100.10 01.0101. 1000.0000.00 00.00 10.1101     25

    0x4958002D

```


## RAW Pops dissasembly ##

```
        0x0000E5D0: lhu        $a0, 68($gp)
                    lhu        $a1, 72($gp)
                    lhu        $a2, 76($gp)
                    lh         $v0, 244($gp)
                    addu       $a0, $a0, $a1
                    addu       $a0, $a0, $a2
                    mult       $a0, $v0
                    li         $v0, 0xFFFF
                    mflo       $a0
                    sw         $a0, 96($gp)
                    sra        $a0, $a0, 12
                    sltu       $a1, $v0, $a0
                    max        $a0, $zr, $a0
                    min        $a0, $v0, $a0
                    sll        $a1, $a1, 18
                    mtv        $a1, S330
                    jr         $ra
                    sh         $a0, 28($gp)
```

## Reversing ##

```

    a0 = gteSZ1
    a1 = gteSZ2
    a2 = gteSZ3

    v0 = gteZSF3

    a0 = gteSZ1 + gteSZ2
    a0 = gteSZ1 + gteSZ2 + gteSZ3

    Acc = (gteSZ1 + gteSZ2 + gteSZ3) * gteZSF3

    v0 = 0xFFFF

    a0 = lo

    gteMAC0 = a0

    a0 = a0 >> 12
    a1 = 0xFFFF < a0
    a0 = max (0, a0)
    a0 = min (0xFFFF, a0)
    
    a1 = a1 << 18
    VFPU.S330 = a1
    gteOTZ = a0
```

## Official doc ##

```
Calculations:
    (1.31.12)       [OOTZ] = ZSF3*SZ0(1)
                             + ZSF3*SZ1(2)
                             + ZSF3*SZ2(3); <4>
    (0.16. 0)       OTZ = limC([OOTZ]);
    (1.31. 0)       *MAC0* = [OOTZ];
```

## Pops C ##

```
AVSZ3
{
    u32     acc = gteSZ1 + gteSZ2 + gteSZ3;
    s64     mac = acc * gteZSF3;
    s32     prod = (s32)mac;      // get lower part of 64-bit result
    gteMAC0 = prod;
    prod = prod >> 12;     // keep sign while shifting
    gteOTZ = min (0xFFFF, max (0, prod));
    FLAG[18] = (u32)prod > 0xFFFF;       // since this is unsigned comparsion, all negative numbers are also checked
}
```

Pops missing emulation of test result `<`4`>` check :

**FLAG**:
|bit|meaning|
|:--|:------|
|16 |1: Calculation test result #4 overflow generated (2`^`31 or more)|
|15 |1: Calculation test result #4 underflow generated (less than -2`^`31)|

## Pcsx-r C ##

```

#define limD(a) LIM((a), 0xffff, 0x0000, (1 << 31) | (1 << 18))
#define F(a) BOUNDS((a), 0x7fffffff, (1 << 31) | (1 << 16), -(s64)0x80000000, (1 << 31) | (1 << 15))


void gteAVSZ3() {
    gteFLAG = 0;
    gteMAC0 = F((s64)(gteZSF3 * gteSZ1) + (gteZSF3 * gteSZ2) + (gteZSF3 * gteSZ3));
    gteOTZ = limD(gteMAC0 >> 12);
}
```

<font color='red'><b>Pops has issue?</b></font><br>
Why Pops doesn't make overflow/underflow check of MAC0 calculation (FLAG<code>[</code>15<code>]</code> and FLAG<code>[</code>16<code>]</code>) ??? Need to check on real hardware.<br>
<br>
<h2>Test on real hardware</h2>

<img src='http://ogamespec.com/imgstore/whc4e53af0ab1598.jpg'>

Test on real hardware show, that overflow/underflow calculations are performed by GTE, so <b>Pops has bug in AVSZ3</b>.<br>
<br>
Also many emulators has flaw in AVSZ3, for example ePSXe 1.6.0 :<br>
<img src='http://ogamespec.com/imgstore/whc4e53afce3c12f.jpg'>

Pcsx-Reloaded has correct AVSZ3 :<br>
<img src='http://ogamespec.com/imgstore/whc4e53adc6b5578.jpg'>