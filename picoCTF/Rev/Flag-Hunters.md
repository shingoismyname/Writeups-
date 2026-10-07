# Flag Hunters
- Đây là 1 bài mức độ easy.
**Đề bài như sau:**
- Lyrics jump from verses to the refrain kind of like a subroutine call. There is a hidden refrain this program doesn not print by default. Can you get it to print it? There might be something in it for you.<br>
The program`s source code can be downloaded [here](https://challenge-files.cylabacademy.net/library/f80370282cd4baf0ad8e8ed41402ccddb55aea0038d174673324709755639912/lyric-reader.py)<br>
## Giải
- Sau khi tải về tôi thấy đây là 1 file python, dùng vs code để mở ra:
```python
import re
import time


# Read in flag from file
flag = open('flag.txt', 'r').read()

secret_intro = \
'''Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, '''\
+ flag + '\n'


song_flag_hunters = secret_intro +\
'''

[REFRAIN]
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
CROWD (Singalong here!);
RETURN

[VERSE1]
Command line wizards, we’re starting it right,
Spawning shells in the terminal, hacking all night.
Scripts and searches, grep through the void,
Every keystroke, we're a cypher's envoy.
Brute force the lock or craft that regex,
Flag on the horizon, what challenge is next?

REFRAIN;

Echoes in memory, packets in trace,
Digging through the remnants to uncover with haste.
Hex and headers, carving out clues,
Resurrect the hidden, it's forensics we choose.
Disk dumps and packet dumps, follow the trail,
Buried deep in the noise, but we will prevail.

REFRAIN;

Binary sorcerers, let’s tear it apart,
Disassemble the code to reveal the dark heart.
From opcode to logic, tracing each line,
Emulate and break it, this key will be mine.
Debugging the maze, and I see through the deceit,
Patch it up right, and watch the lock release.

REFRAIN;

Ciphertext tumbling, breaking the spin,
Feistel or AES, we’re destined to win.
Frequency, padding, primes on the run,
Vigenère, RSA, cracking them for fun.
Shift the letters, matrices fall,
Decrypt that flag and hear the ether call.

REFRAIN;

SQL injection, XSS flow,
Map the backend out, let the database show.
Inspecting each cookie, fiddler in the fight,
Capturing requests, push the payload just right.
HTML's secrets, backdoors unlocked,
In the world wide labyrinth, we’re never lost.

REFRAIN;

Stack's overflowing, breaking the chain,
ROP gadget wizardry, ride it to fame.
Heap spray in silence, memory's plight,
Race the condition, crash it just right.
Shellcode ready, smashing the frame,
Control the instruction, flags call my name.

REFRAIN;

END;
'''

MAX_LINES = 100

def reader(song, startLabel):
  lip = 0
  start = 0
  refrain = 0
  refrain_return = 0
  finished = False

  # Get list of lyric lines
  song_lines = song.splitlines()
  
  # Find startLabel, refrain and refrain return
  for i in range(0, len(song_lines)):
    if song_lines[i] == startLabel:
      start = i + 1
    elif song_lines[i] == '[REFRAIN]':
      refrain = i + 1
    elif song_lines[i] == 'RETURN':
      refrain_return = i

  # Print lyrics
  line_count = 0
  lip = start
  while not finished and line_count < MAX_LINES:
    line_count += 1
    for line in song_lines[lip].split(';'):
      if line == '' and song_lines[lip] != '':
        continue
      if line == 'REFRAIN':
        song_lines[refrain_return] = 'RETURN ' + str(lip + 1)
        lip = refrain
      elif re.match(r"CROWD.*", line):
        crowd = input('Crowd: ')
        song_lines[lip] = 'Crowd: ' + crowd
        lip += 1
      elif re.match(r"RETURN [0-9]+", line):
        lip = int(line.split()[1])
      elif line == 'END':
        finished = True
      else:
        print(line, flush=True)
        time.sleep(0.5)
        lip += 1



reader(song_flag_hunters, '[VERSE1]')

```
- Nhận thấy đây có vẻ như là 1 chương trình in ra 1 đoạn nhạc theo 1 trật tự nào đó, tôi đã đọc và đây là cách chương trình hoạt động:
- Bài hát này gồm Intro (là đoạn chứa flag), điệp khúc, và khổ 1
- Khi chạy, nó sẽ bắt đầu in từ đoạn bắt đầu là [Verse 1], sau đó tiếp tục và cứ mỗi lần chuyển đoạn (gặp chữ "REFRAIN" thì nó lại nhảy đến đoạn [REFRAIN] để in) sau đó quay lại---> Lặp lại cho đến hết bài.<br>
Tôi cũng để ý thấy ở đoạn [REFRAIN] này có yêu cầu input từ người dùng: 
```python
if line == 'REFRAIN':
        song_lines[refrain_return] = 'RETURN ' + str(lip + 1)
        lip = refrain
      elif re.match(r"CROWD.*", line):
        crowd = input('Crowd: ')
        song_lines[lip] = 'Crowd: ' + crowd
        lip += 1
```
Và 1 chú ý quan trọng nữa:
```python
elif re.match(r"RETURN [0-9]+", line):
        lip = int(line.split()[1])
```

- Ở đây, biến lip chính là số thứ tự (vị trí) của dòng mà chương trình in ra, nó luôn được thay đổi suốt quá trình chạy.
Với dòng elif này thực chất là: 'Nếu gặp dòng có từ RETURN + 1 số thì nhảy đến dòng có thứ tự đó để in nhé' <br>
Từ đây chúng ta hoàn toàn có thể nghĩ đến việc từ input của người dùng, làm sao đó để nhập 1 dòng dạng RETURN + 1 số (dòng chứa flag) để chương trình thực thi.<br>
Nhưng có 1 vấn đề:
```python
re.match(r"RETURN [0-9]+", line)
```
- Hàm match() chỉ hoặc động nếu đoạn "RETURN + 1 số" nằm ở 'ĐẦU DÒNG', trong khi input của chúng ta đứng sau từ 'Crowd':
```python
crowd = input('Crowd: ')
song_lines[lip] = 'Crowd: ' + crowd
```
- Đến đây thì không biết làm gì do đó ta sẽ xem xét kĩ đoạn code hơn nhé:
```python
for line in song_lines[lip].split(';'):
```
- Nhìn lại dòng for này ta thấy chương trình tách dòng nếu nó được ngăn cách với nhau bởi ';', và đây cũng là mấu chốt của bài này.<br>
Do đó nếu input của ta có dạng: ;RETURN 1 <br>
Thì chương trình sẽ coi RETURN 1 này là 1 dòng mới, và hàm 'match' sẽ thực thi từ đó nhảy đến dòng 1.<br>
## Lấy flag: 
- OK, Giờ thấy launch instance và ghi input vào thôi. Đợi 1 xíu ta sẽ ra được flag (tự tìm đi nhé ^^ ) <br>
<Lưu ý: ở đây ta có thể RETURN luôn vào dòng chứa flag, hoặc các dòng phía trên vì nó vẫn sẽ tự in tiếp đến dòng chứa flag>

