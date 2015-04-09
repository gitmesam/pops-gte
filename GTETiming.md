# Introduction #

During reversing of Pops `CpuException()` call, I found reference to GTE timing table :)


# Details #

My Pops version: `branches/pops-352/src(r5285)`

Table resided at 0x0002CCD8 in PRX:

static u8 gteTiming`[32]` = {
|0x01 |0x0F |0x01 |0x01 |0x01 |0x01 |0x08 |0x01 |
|:----|:----|:----|:----|:----|:----|:----|:----|
|0x01 |0x01 |0x01 |0x01 |0x06 |0x01 |0x01 |0x01 |
|0x08 |0x08 |0x08 |0x13 |0x08 |0x01 |0x2C |0x01 |
|0x01 |0x01 |0x01 |0x11 |0x0B |0x01 |0x0E |0x01 |
|0x1F |0x01 |0x01 |0x01 |0x01 |0x01 |0x01 |0x01 |
|0x05 |0x0D |0x11 |0x01 |0x01 |0x05 |0x06 |0x01 |
|0x17 |0x01 |0x01 |0x01 |0x01 |0x01 |0x01 |0x01 |
|0x01 |0x01 |0x01 |0x01 |0x01 |0x05 |0x05 |0x27 |

Corresponding GTE instructions:
|psxBASIC| gteRTPS | psxNULL | psxNULL| psxNULL| psxNULL | gteNCLIP| psxNULL|
|:-------|:--------|:--------|:-------|:-------|:--------|:--------|:-------|
|psxNULL | psxNULL | psxNULL | psxNULL| gteOP  | psxNULL | psxNULL | psxNULL|
|gteDPCS | gteINTPL| gteMVMVA| gteNCDS| gteCDP | psxNULL | gteNCDT | psxNULL|
|psxNULL | psxNULL | psxNULL | gteNCCS| gteCC  | psxNULL | gteNCS  | psxNULL|
|gteNCT  | psxNULL | psxNULL | psxNULL| psxNULL| psxNULL | psxNULL | psxNULL|
|gteSQR  | gteDPCL | gteDPCT | psxNULL| psxNULL| gteAVSZ3|gteAVSZ4| psxNULL|
|gteRTPT | psxNULL | psxNULL | psxNULL| psxNULL| psxNULL | psxNULL |psxNULL|
|psxNULL | psxNULL | psxNULL | psxNULL| psxNULL| gteGPF  | gteGPL  | gteNCCT |

Result table:
|instruction|clocks (Pops)|Doomed/Padua gte.txt|Official doc|
|:----------|:------------|:-------------------|:-----------|
|RTPS |15|15|14 `*`|
|NCLIP|8 |8 |8 |
|OP  |6 |6 |6 |
|DPCS |8 |8 |8 |
|INTPL|8 |8 |8 |
|MVMVA|8 |8 |8 |
|NCDS|19|19|19|
|CDP|8 |13 `*`|13|
|NCDT |44|44|44|
|NCCS|17|17|17|
|CC|11|11|11|
|NCS|14|14|14|
|NCT|31|30 `*`|30|
|SQR|5 |5 |5 |
|DPCL|13|8 `*`|8 |
|DPCT|17|17|17|
|AVSZ3|5 |5 |5 |
|AVSZ4|6 |6 |6 |
|RTPT|23|23|22 `*`|
|GPF|5 |5 |5 |
|GPL|5 |5 |5 |
|NCCT|39|39|39|

`*` - GTE timing specified in gte.txt by Doomed/Padua differs from Pops timing.