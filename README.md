# root-me-ELF-Cpp-0-protection

## Research Log

---

### [00:00] Setup & Tools

- **Ghidra** / **IDA**: For slicing and dicing binaries.
- **VM (Kali Linux)**: Because running random binaries on your host is a bad idea.
- **MobaXterm**: SSH like a boss.
- **Music Playlist**: Because working without music is a crime. [YouTube Music](https://music.youtube.com/playlist?list=PLsaTaTh7fDWEq_JMpTKHqwNzmM3tDFQFy&si=RsgwNSH1G1gUVFfa)
- **Energy Drinks**: XL/Blue/RedBull/Monster or anything that makes your heart race.

---

### [00:05] First Contact

Let’s see what this binary wants from us:

```
┌──(bash㉿bash-linux)-[~/Desktop/cpp_no_protection]
└─$ ./ch25.bin
usage : ./ch25.bin password

┌──(bash㉿bash-linux)-[~/Desktop/cpp_no_protection]
└─$ ./ch25.bin 12345
Password incorrect.
```

Yup, classic. Wants a password as an argument. Time to fire up IDA.

---

### [00:10] Into the C++ Abyss

As expected, C++ code is a spaghetti bowl of structs and objects. Here’s the decompiled main (brace yourself):

```cpp
// ...existing code...
int __cdecl main(int argc, const char **argv, const char **envp)
{
  // ...so many variables...
  if ( argc > 1 )
  {
    // ...constructors, allocators, destructors, oh my...
    plouf(v13);
    // ...more destructors...
    if ( (unsigned __int8)std::operator==<char>((std::string *)v13, (char *)argv[1]) )
    {
      // Success messages
    }
    else
    {
      // Failure message
    }
    // ...cleanup...
  }
  else
  {
    // Print usage
    return 5;
  }
  return v7;
}
```

C++ is object-oriented, which means everything is wrapped in structs and classes. Makes reversing a bit more... fun.

---

### [00:20] Assembly Time

Here’s the juicy part in assembly:

```
.text:08048B80 mov     eax, [ebx+4]
.text:08048B83 add     eax, 4
.text:08048B86 mov     eax, [eax]
.text:08048B88 mov     [esp+4], eax    ; char *
.text:08048B8C lea     eax, [ebp+var_14]
.text:08048B8F mov     [esp], eax      ; std::string *
.text:08048B92 call    _ZSteqIcSt11char_traitsIcESaIcEEbRKSbIT_T0_T1_EPKS3_ ; std::operator==<char>(std::string const&,char const*)
.text:08048B97 test    al, al
.text:08048B99 jz      short loc_8048BE5
```

It calls a function to compare two strings, then checks if they’re equal. If yes, you win. If not, you get the sad trombone.

---

### [00:25] Digging Deeper

Let’s peek into the comparison function:

```cpp
bool __cdecl std::operator==<char>(std::string *a1, char *a2)
{
  return std::string::compare(a1, a2) == 0;
}
```

It’s just a wrapper around `std::string::compare`. Nothing fancy, but now we know where the magic happens.

---

### [00:30] Debugger Magic Trick

Let’s do some runtime snooping. Set a breakpoint right before the comparison:

```
b *0x08048B92
```

Run the program with a dummy password:

```
r 12345
```

When we hit the breakpoint, let’s see what EAX points to:

```
x/wx $eax
```

Now, let’s grab the string at that address:

```
x/s <found address>
```

---

### [00:35] The Payoff

Here’s what we see in GDB:

```
(gdb) b *0x08048B92
Breakpoint 1 at 0x8048b92
(gdb) r 12345
Starting program: /home/bash/Desktop/cpp_no_protection/ch25.bin 12345
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x08048b92 in main ()
(gdb) x/wx $eax
0xffffd164:     0x0804f4cc
(gdb) x/s 0x0804f4cc
0x804f4cc:      "Here_you_have_to_understand_a_little_C++_stuffs"
(gdb)
```

Yessss! We got the code.

---

### [00:40] Victory Lap

yessss we got the code  
let's make sure it's the password:

```
└─$ ./ch25.bin Here_you_have_to_understand_a_little_C++_stuffs
Bravo, tu peux valider en utilisant ce mot de passe...
Congratz. You can validate with this password...
```

**Mission accomplished!**  
Password cracked, C++ shenanigans defeated, and another energy drink justified. 😎  