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
- Ta có thể ssh vào trước, kiểm tra đường dẫn hiện tại bằng `pwd` sau đó mới tải về.
- 
