# Bypass me 
- Your task is to analyze and exploit a password-protected binary called bypassme.bin and binary performs input sanitization.
- However, instead of guessing the password, you are expected to reverse engineer or debug the program to bypass the authentication logic and retrieve the hidden flag.
- You'll need to think like an attacker using tool like (LLDB)[https://lldb.llvm.org/use/tutorial.html] to uncover how the binary works under the hood and leak the correct password.
- ## Hướng đi
- Trước tiên ta cần tìm hiểu 1 chút về *lldb*, đây là 1 tool có chức năng tương tự *gdb*, dùng để debugging 1 chương trình <br>
 Đề bài yêu cầu chúng ta xử lý file bypassme.bin, và sẽ kết nối thông qua ssh, tuy nhiên để tiện thì tôi muốn tải nó về kết hợp dùng *ghidra* và *lldb* để giải quyết <br>
 Dùng *scp*
```console
scp username@remote_host:/path/to/remote/file.tx
```
- Ta có thể ssh vào trước, kiểm tra đường dẫn hiện tại bằng `pwd` sau đó mới tải về.<br>
Sau đó ta sẽ được: <br>
![Ghidra](ghidra2.png)

- Giờ ta sẽ vào phân tích <br>
```console
    printf("\n[%d tries left] Enter password: ",(ulong)(uint)attempts);
    fflush(stdout);
    fgets(buf,0x80,stdin);
    sVar3 = strcspn(buf,"\n");
    buf[sVar3] = '\0';
    sanitize(buf,sanitized);
    printf("\nRaw Input:      [%s]\n",buf);
    printf("Sanitized Input:[%s]\n",sanitized);
    puts("Hint: Input must match something special...");
```
- Chú ý ngay đến đoạn này, có vẻ như chúng ta cần nhập vào 1 password, nó sẽ **match** với 1 cái gì đó.

```console
    iVar2 = strcmp(buf,password);
    if (iVar2 == 0) {
      auth_sequence();
      __stream = fopen("../../root/flag.txt","r");
```

- Thật vậy nó kiểm tra `iVar2 = strcmp(buf,password)` và đưa ra flag nếu đúng `__stream = fopen("../../root/flag.txt","r");` <br>
Chúng ta cùng đi vào xem nó kiểm tra với cái gì nhé: <br>
- Ở đây: `blur` chính là *input* của chúng ta, nó được so sánh với *password*, ta sẽ lần theo biến này. <br>
```console
char password [128]
decode_password(password);
```
- Vậy ra *password* được xử lý trong hàm `decode_password`, nhấp vào hàm này để xem: <br>
```console

void decode_password(char *out)

{
  long lVar1;
  long in_FS_OFFSET;
  char *out_local;
  int i;
  uchar enc [11];
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  enc[0] = 0xf9;
  enc[1] = 0xdf;
  enc[2] = 0xda;
  enc[3] = 0xcf;
  enc[4] = 0xd8;
  enc[5] = 0xf9;
  enc[6] = 0xcf;
  enc[7] = 0xc9;
  enc[8] = 0xdf;
  enc[9] = 0xd8;
  enc[10] = 0xcf;
  for (i = 0; (uint)i < 0xb; i = i + 1) {
    out[i] = enc[i] ^ 0xaa;
  }
  out[0xb] = '\0';
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```
- Ta thấy chương trình khởi tạo `enc` gồm 11 phần tử, sau đó dùng vòng `for` để xử lý từng phần từ, cuối cùng `return` ra `out`, cũng chính là **pasword** cần tìm.
- Nhìn vào vòng `for` <br>
```console
for (i = 0; (uint)i < 0xb; i = i + 1) {
    out[i] = enc[i] ^ 0xaa;
  }
```
- Thao tác xử lý sẽ là `XOR` từng phần tử của `enc` với `0xaa` rồi đưa vào `out`. <br>
Vậy là ta đã nắm được các xử lý của chương trình này.<br>
Đến đây có 2 hướng: <br>
- Viết 1 trương trình python để `XOR` luôn `enc` vì ta đã biết được tất cả các phần tử.
- Hoặc dùng `lldb` như yêu cầu của đề bài <br>
Tôi sẽ đi theo cách 2 cho phù hợp với đề nhé: <br>
Tôi tiến hành kết nối với server: <br>
```console
ctf-player@academy-chall$ ls
bypassme.bin
ctf-player@academy-chall$
```
- Kiểm tra ta thấy file `bypassme.bin`, giờ ta sẽ chạy file này bằng `lldb`: <br>
```console
ctf-player@academy-chall$ lldb bypassme.bin
(lldb) target create "bypassme.bin"
Current executable set to '/home/ctf-player/bypassme.bin' (x86_64).
(lldb)
```
- Ta sẽ dịch nó sang assembly nhé: <br>
Dùng disassemble -n <tên hàm> cụ thể là hàm main nhé:<br>
```console
(lldb) disassemble -n main
bypassme.bin`main:
bypassme.bin[0x164e] <+0>:   endbr64 
bypassme.bin[0x1652] <+4>:   pushq  %rbp
bypassme.bin[0x1653] <+5>:   movq   %rsp, %rbp
bypassme.bin[0x1656] <+8>:   subq   $0x220, %rsp ; imm = 0x220 
bypassme.bin[0x165d] <+15>:  movq   %fs:0x28, %rax
bypassme.bin[0x1666] <+24>:  movq   %rax, -0x8(%rbp)
bypassme.bin[0x166a] <+28>:  xorl   %eax, %eax
bypassme.bin[0x166c] <+30>:  movl   $0x3, -0x21c(%rbp)
bypassme.bin[0x1676] <+40>:  leaq   -0x110(%rbp), %rax
bypassme.bin[0x167d] <+47>:  movq   %rax, %rdi
bypassme.bin[0x1680] <+50>:  callq  0x1334         ; decode_password at bypassme.c:16:33
bypassme.bin[0x1685] <+55>:  callq  0x14c5         ; intro_sequence at bypassme.c:45:23
bypassme.bin[0x168a] <+60>:  jmp    0x1837         ; <+489> at bypassme.c:82:20
bypassme.bin[0x168f] <+65>:  movl   -0x21c(%rbp), %eax
bypassme.bin[0x1695] <+71>:  addl   $0x1, %eax
bypassme.bin[0x1698] <+74>:  movl   %eax, %esi
bypassme.bin[0x169a] <+76>:  leaq   0x10f7(%rip), %rax
bypassme.bin[0x16a1] <+83>:  movq   %rax, %rdi
bypassme.bin[0x16a4] <+86>:  movl   $0x0, %eax
bypassme.bin[0x16a9] <+91>:  callq  0x1150         ; ___lldb_unnamed_symbol70 + 64
bypassme.bin[0x16ae] <+96>:  movq   0x295b(%rip), %rax ; stdout@GLIBC_2.2.5
bypassme.bin[0x16b5] <+103>: movq   %rax, %rdi
bypassme.bin[0x16b8] <+106>: callq  0x1190         ; ___lldb_unnamed_symbol70 + 128
bypassme.bin[0x16bd] <+111>: movq   0x295c(%rip), %rdx ; stdin@GLIBC_2.2.5
bypassme.bin[0x16c4] <+118>: leaq   -0x210(%rbp), %rax
bypassme.bin[0x16cb] <+125>: movl   $0x80, %esi
bypassme.bin[0x16d0] <+130>: movq   %rax, %rdi
bypassme.bin[0x16d3] <+133>: callq  0x1170         ; ___lldb_unnamed_symbol70 + 96
bypassme.bin[0x16d8] <+138>: leaq   -0x210(%rbp), %rax
bypassme.bin[0x16df] <+145>: leaq   0x10d4(%rip), %rdx
bypassme.bin[0x16e6] <+152>: movq   %rdx, %rsi
bypassme.bin[0x16e9] <+155>: movq   %rax, %rdi
bypassme.bin[0x16ec] <+158>: callq  0x1160         ; ___lldb_unnamed_symbol70 + 80
bypassme.bin[0x16f1] <+163>: movb   $0x0, -0x210(%rbp,%rax)
bypassme.bin[0x16f9] <+171>: leaq   -0x190(%rbp), %rdx
bypassme.bin[0x1700] <+178>: leaq   -0x210(%rbp), %rax
bypassme.bin[0x1707] <+185>: movq   %rdx, %rsi
bypassme.bin[0x170a] <+188>: movq   %rax, %rdi
bypassme.bin[0x170d] <+191>: callq  0x13c2         ; sanitize at bypassme.c:24:48
bypassme.bin[0x1712] <+196>: leaq   -0x210(%rbp), %rax
bypassme.bin[0x1719] <+203>: movq   %rax, %rsi
bypassme.bin[0x171c] <+206>: leaq   0x1099(%rip), %rax
bypassme.bin[0x1723] <+213>: movq   %rax, %rdi
bypassme.bin[0x1726] <+216>: movl   $0x0, %eax
bypassme.bin[0x172b] <+221>: callq  0x1150         ; ___lldb_unnamed_symbol70 + 64
bypassme.bin[0x1730] <+226>: leaq   -0x190(%rbp), %rax
bypassme.bin[0x1737] <+233>: movq   %rax, %rsi
bypassme.bin[0x173a] <+236>: leaq   0x1092(%rip), %rax
bypassme.bin[0x1741] <+243>: movq   %rax, %rdi
bypassme.bin[0x1744] <+246>: movl   $0x0, %eax
bypassme.bin[0x1749] <+251>: callq  0x1150         ; ___lldb_unnamed_symbol70 + 64
bypassme.bin[0x174e] <+256>: leaq   0x109b(%rip), %rax
bypassme.bin[0x1755] <+263>: movq   %rax, %rdi
bypassme.bin[0x1758] <+266>: callq  0x1120         ; ___lldb_unnamed_symbol70 + 16
bypassme.bin[0x175d] <+271>: leaq   -0x110(%rbp), %rdx
bypassme.bin[0x1764] <+278>: leaq   -0x210(%rbp), %rax
bypassme.bin[0x176b] <+285>: movq   %rdx, %rsi
bypassme.bin[0x176e] <+288>: movq   %rax, %rdi
bypassme.bin[0x1771] <+291>: callq  0x1180         ; ___lldb_unnamed_symbol70 + 112
bypassme.bin[0x1776] <+296>: testl  %eax, %eax
bypassme.bin[0x1778] <+298>: jne    0x1828         ; <+474> at bypassme.c:110:19
bypassme.bin[0x177e] <+304>: callq  0x1453         ; auth_sequence at bypassme.c:34:22
bypassme.bin[0x1783] <+309>: leaq   0x1092(%rip), %rax
bypassme.bin[0x178a] <+316>: movq   %rax, %rsi
bypassme.bin[0x178d] <+319>: leaq   0x108a(%rip), %rax
bypassme.bin[0x1794] <+326>: movq   %rax, %rdi
bypassme.bin[0x1797] <+329>: callq  0x11b0         ; ___lldb_unnamed_symbol70 + 160
bypassme.bin[0x179c] <+334>: movq   %rax, -0x218(%rbp)
bypassme.bin[0x17a3] <+341>: cmpq   $0x0, -0x218(%rbp)
bypassme.bin[0x17ab] <+349>: je     0x1812         ; <+452> at bypassme.c:106:23
bypassme.bin[0x17ad] <+351>: movq   -0x218(%rbp), %rdx
bypassme.bin[0x17b4] <+358>: leaq   -0x90(%rbp), %rax
bypassme.bin[0x17bb] <+365>: movl   $0x80, %esi
bypassme.bin[0x17c0] <+370>: movq   %rax, %rdi
bypassme.bin[0x17c3] <+373>: callq  0x1170         ; ___lldb_unnamed_symbol70 + 96
bypassme.bin[0x17c8] <+378>: testq  %rax, %rax
bypassme.bin[0x17cb] <+381>: setne  %al
bypassme.bin[0x17ce] <+384>: testb  %al, %al
bypassme.bin[0x17d0] <+386>: je     0x17f2         ; <+420> at bypassme.c:102:27
bypassme.bin[0x17d2] <+388>: leaq   -0x90(%rbp), %rax
bypassme.bin[0x17d9] <+395>: movq   %rax, %rsi
bypassme.bin[0x17dc] <+398>: leaq   0x104f(%rip), %rax
bypassme.bin[0x17e3] <+405>: movq   %rax, %rdi
bypassme.bin[0x17e6] <+408>: movl   $0x0, %eax
bypassme.bin[0x17eb] <+413>: callq  0x1150         ; ___lldb_unnamed_symbol70 + 64
bypassme.bin[0x17f0] <+418>: jmp    0x1801         ; <+435> at bypassme.c:104:23
bypassme.bin[0x17f2] <+420>: leaq   0x1048(%rip), %rax
bypassme.bin[0x17f9] <+427>: movq   %rax, %rdi
bypassme.bin[0x17fc] <+430>: callq  0x1120         ; ___lldb_unnamed_symbol70 + 16
bypassme.bin[0x1801] <+435>: movq   -0x218(%rbp), %rax
bypassme.bin[0x1808] <+442>: movq   %rax, %rdi
bypassme.bin[0x180b] <+445>: callq  0x1130         ; ___lldb_unnamed_symbol70 + 32
bypassme.bin[0x1810] <+450>: jmp    0x1821         ; <+467> at bypassme.c:108:20
bypassme.bin[0x1812] <+452>: leaq   0x103c(%rip), %rax
bypassme.bin[0x1819] <+459>: movq   %rax, %rdi
bypassme.bin[0x181c] <+462>: callq  0x1120         ; ___lldb_unnamed_symbol70 + 16
bypassme.bin[0x1821] <+467>: movl   $0x0, %eax
bypassme.bin[0x1826] <+472>: jmp    0x1867         ; <+537> at bypassme.c:116:1
bypassme.bin[0x1828] <+474>: leaq   0x103b(%rip), %rax
bypassme.bin[0x182f] <+481>: movq   %rax, %rdi
bypassme.bin[0x1832] <+484>: callq  0x1120         ; ___lldb_unnamed_symbol70 + 16
bypassme.bin[0x1837] <+489>: movl   -0x21c(%rbp), %eax
bypassme.bin[0x183d] <+495>: leal   -0x1(%rax), %edx
bypassme.bin[0x1840] <+498>: movl   %edx, -0x21c(%rbp)
bypassme.bin[0x1846] <+504>: testl  %eax, %eax
bypassme.bin[0x1848] <+506>: setne  %al
bypassme.bin[0x184b] <+509>: testb  %al, %al
bypassme.bin[0x184d] <+511>: jne    0x168f         ; <+65> at bypassme.c:83:15
bypassme.bin[0x1853] <+517>: leaq   0x1026(%rip), %rax
bypassme.bin[0x185a] <+524>: movq   %rax, %rdi
bypassme.bin[0x185d] <+527>: callq  0x1120         ; ___lldb_unnamed_symbol70 + 16
bypassme.bin[0x1862] <+532>: movl   $0x1, %eax
bypassme.bin[0x1867] <+537>: movq   -0x8(%rbp), %rdx
bypassme.bin[0x186b] <+541>: subq   %fs:0x28, %rdx
bypassme.bin[0x1874] <+550>: je     0x187b         ; <+557> at bypassme.c:116:1
bypassme.bin[0x1876] <+552>: callq  0x1140         ; ___lldb_unnamed_symbol70 + 48
bypassme.bin[0x187b] <+557>: leave  
bypassme.bin[0x187c] <+558>: retq   
(lldb)
```
- Được rồi, ta đã thấy hàm `decode_password`, ta sẽ tiếp tục xem nó nhé: <br>
```console
settings set target.x86-disassembly-flavor intel
```
- Nếu muốn rõ hơn ta có thể dùng lệnh này để assembly giống bên ghidra. <br>
- Giờ chú ý kỹ đoạn này nhé: <br>
```console
0x5582e1f593a0 <+108>: mov    rax, qword ptr [rbp - 0x28]
0x5582e1f593a4 <+112>: add    rax, 0xb
0x5582e1f593a8 <+116>: mov    byte ptr [rax], 0x0
```
- Đối chiếu với các thao tác của chương trình mà mình đã phân tích ở `ghidra`, ta nhận thấy đoạn này chính là `out[0xb] = '\0';` hay `out[12] = '\0';` <br>
Nên đoạn `byte ptr [rax], 0x0` chính là chìa khóa cần thiết để giải bài, địa chỉ của thanh ghi `rax` đang giữ chính là địa chỉ của phần tử `out[12]` và đoạn trước đó `add rax, 0xb` dùng để đưa `rax` đến vị trí `out[12]` này. <br>
- Vậy nên ý tưởng chính là đặt 1 breakpoint ngay tại đoạn <+112> này, sao đó chỉ cần đọc địa chỉ ở thanh `rax` là xong. Ok ta sẽ bắt tay vào làm: <br>
Đặt breakpoint tại hàm `decode_password`:
```console
(lldb) b -n decode_password
Breakpoint 3: where = bypassme.bin`decode_password(char*) + 31 at bypassme.c:17:19, address = 0x00005ea9be393353
(lldb) run
Process 111 launched: '/home/ctf-player/bypassme.bin' (x86_64)
Process 111 stopped
* thread #1, name = 'bypassme.bin', stop reason = breakpoint 3.1
    frame #0: 0x00005582e1f59353 bypassme.bin`decode_password(out="\xf9\U00000002") at bypassme.c:17:19
```
Tiếp theo là đặt breakpoint ở đoạn <+112> nhé: <br>
```console
(lldb) breakpoint set --address "decode_password + 112"
```
- Và chạy thôi : <br>
```console
(lldb) c
Process 111 resuming
Process 111 stopped
* thread #1, name = 'bypassme.bin', stop reason = breakpoint 7.1
    frame #0: 0x00005582e1f593a4 bypassme.bin`decode_password(out="SuperSecure") at bypassme.c:21:20
```
- Giờ ta sẽ đọc dữ liệu ở địa chỉ mà thanh ghi `rax` đang giữ nhé:
```console
(lldb) x/s $rax
0x7ffe3299eaf0: "SuperSecure"
```
- `x/s` sẽ đọc dữ liệu ở dạng `strings` cho đến `\0`, kết quả nhận được chính là password cần tìm. <br>
- Giờ thì chạy file và nhập password để lấy flag nhé: `./bypassme.bin`
