<title>HireMe.c | death_by_code()</title>

# HireMe.c

HireMe.c is a challenge by Nintendo's Euroepan Research & Development organisation, available at https://www.nerd.nintendo.com/files/HireMe.html.

Solve the puzzle and you might get hired, basically.

Seems really difficult, but I'll give it a shot anyway. I don't expect to get very far, but close is good enough. I'll be extra verbose so I can easily pick up where I left off, in case I decide to take another one of those 5-month pauses between commits o_o

While doing this, I did my best not to do any research specific to this particular challenge. Whatever I come up, came from my brain and the gods of Stack Overflow.

***

First, taking a look at the comment.

> The solutions to this challenge belong to different levels :
>
> Level 1 : an iterative algorithm which typically takes more than a second to find a solution (for any given output). 
>
> Most people stop here, which is fine, but if you want to go further, there is :
>
> Level 2 : a non-iterative algorithm which typically takes less than a millisecond to find a solution (for any given output).
>
> Few people have reached this level. But if you want to beat it completely, there's yet another castle...
>
> Level 3 : an algorithm which can provide any of the 2^128 solutions (for any given output).
>
> Even fewer people have reached this final level. Congratulations to them!

So we basically have to build an algorithm which can provide our input array `u8 input[32]` for any target. Then a `memcmp()` is used to check for the first `size_t 16` bytes. If `memcmp` returns a non-zero number (e.g 0x0A) then the output (in essence, parsed input), which was diffused and confused through the random values, does not match the target.

Firstly, inspecting the `Forward()` function. There are four `for` loops, 3 of which use the `u8 confusion[512]` array and 1 of which interact with the `u32 diffusion[32] array` 

```c
typedef unsigned char u8;
typedef unsigned int u32;

...
//            input     output  confusion diffusion 
void Forward(u8 c[32],u8 d[32],u8 s[512],u32 p[32])
{
    for(u32 i=0;i<256;i++)
    {
        for(u8 j=0;j<32;j++)
        {
            d[j]=s[c[j]];
            c[j]=0;
        }

        for(u8 j=0;j<32;j++)
            for(u8 k=0;k<32;k++)
                c[j]^=d[k]*((p[j]>>k)&1);
    }
    for(u8 i=0;i<16;i++)
        d[i]=s[c[i*2]]^s[c[i*2+1]+256];
}

```

## Breaking the `void Forward()` function down

For easy reference, I want to break `void Forward()` down into each of the four `for()` iterations. **Hex values are random**, as they are just there for just for illustration.

### `d[j]=s[c[j]];`

- runs for all 32 elements of `input[32]`

```c
        for(u8 j=0;j<32;j++)
        {
            output[j]=confusion[input[j]];
            input[j]=0; // at the end of the loop, input[*] = 0
        }
```

I would now like to use some tasty examples

```c
u8 input[32];

int j = 3;

// hence

output[3] = confusion[input[3]]; 

// results

// input[3] = 0x4e
// output[3] = confusion[input[3]] = confusion[0x4e] = 0x59
```

### `c[j]^=d[k]*((p[j]>>k)&1);`

- runs for all 32 elements of `input[32]`
- this one's a bit of a [monster](https://open.spotify.com/track/06XQvnJb53SUYmlWIhUXUi?autoplay=true)
- I'm actually listening to that song right now, huh
- what a frickin nightmare... I have to reverse engineer this :sob:

```c
        for(u8 j=0;j<32;j++)
            for(u8 k=0;k<32;k++)
                input[j]^=output[k]*((diffusion[j]>>k)&1);
```

For `input[j]^=output[k]*((diffusion[j]>>k)&1);`

- let **potato** = `((diffusion[j]>>k)&1);` be:
	- **eggplant** = take the `j`th element of `diffusion[32]`, then right shift by `k` bits
	- perform AND between the LSB of **eggplant** and an int literal = 1
- `output[k]` XOR **potato**

#### Endianness

x86 machines are **little-endian** with the LSB at the lowest address (see https://docs.oracle.com/cd/E23824_01/html/819-3196/hwovr-35423.html)

```

+--------+--------+--------+--------+
| byte 3 | byte 2 | byte 1 | byte 0 |
+--------+--------+--------+--------+
| MSB    |        |        | LSB    |
+--------+--------+--------+--------+
```






