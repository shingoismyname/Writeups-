# Secure Password Database
- Đây là 1 bài mức độ medium, <br>
- I made a new password authentication program that even shows you the password you entered saved in the database! <br>
  Isn't that cool? [system.out](https://challenge-files.cylabacademy.net/library/c78772efebbd915d804bd7525802ffbf4dfbe0789663066904b37c730bc78498/system.out)
## Hướng đi 
- Trước tiên ta thấy đề bài cho ta 1 file đuôi *.out* là 1 file *.out*, nó được biên soạn bởi trình biên dịch. Hiểu đơn giản là 1 file có thể chạy được. <br>
Đến đây tôi chạy thử file hoặc kết nối với sever của đề bài để chạy thử thì: <br>
``` console
shingo333-academy@webshell:~$ nc xebec.cylabacademy.net 23755
Please set a password for your account:
hieu
How many bytes in length is your password?
4
You entered: 4
Your successfully stored password:
104 105 101 117 10 
Enter your hash to access your account!
d
system.out: heartbleed.c:69: main: Assertion `1 == 0' failed.
Aborted (core dumped)
```
- Chương trình yêu cầu tôi nhập password, số byte của password và cuối dùng là hash để có thể log vào tài khoản (có thể chứa flag)<br>
- Để xem chương trình làm gì, tôi sẽ sử dụng 1 công cụ là *Ghidra*, bạn có thể tham khảo ở [đây](https://github.com/nationalsecurityagency/ghidra)<br>
Tiến hành mở file trong *Ghidra* và nhấn phân tích khi *Ghidra* gợi ý phân tích, sau đó là đến hàm *main* mà *Ghidra* phát hiện ra tôi sẽ được một giao diện kiểu như:

