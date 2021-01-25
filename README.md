## ctfbox 
**A box for CTF challenges with some sugar functions, Just enjoy it**

Current version: **1.2.1**

Please use python **3.6+**

## Install
All you need to do is
```sh
pip install ctfbox
```

## Usage

### Common
```python
from ctfbox import * # Will not import the pwn part, please check the Pwn Usage section below
# enjoy it
```

### Pwn Usage
```python
# Don't support windows
from pwn import * # import pwntools
# set pwntools config...
# context.os = 'linux'
# context.log_level = 'debug'
# context.arch = 'amd64'
from ctfbox.pwntools.config import Config # import confit for pwn part of ctfbox
# set necessary config 
"""
Attributes:
- local(bool) : connect to local binary / remote address, default: True
- bin(str)    : the binary path, e.g. './pwn'
- address(str): the remote address, e.g. '127.0.0.1:2333'
- pie(bool)   : whether the memory address is randomized, default: False
"""
Config.local = True
Config.address = "127.0.0.1:2333"
Config.bin = "./bin"
# import pwn part
from ctfbox.pwn import *
```
now you can use the below attributes/functions
```
slog // empty dictionary, you can set the leaked address and corresponding name. e.g. slog['libc'] = libc_addr
elf  // pwntools.ELF(binaray)
cn   // a connect to local binary or remote address
re   // lambda of cn.recv(m, t)
recv // lambda of cn.recv()
ru   // lambda of cn.recvuntil(x)
rl   // lambda of cn.recvline()
sd   // lambda of cn.send(x)
sl   // lambda of cn.sendline(x)
ia   // lambda of cn.interactive()
sla  // lambda of cn.sendlineafter(a, b)
sa   // lambda of cn.sendafter(a, b)
ft   // ft(arg, f=pwnlib.util.cyclic.de_bruijn(), l=None) lambda of flat(*arg, filler=f, length=l)
gdba // gdba(bps) debug, argument bps save the breakpoint address, breakpoint can also be automatically set when pie is turned on, need pmap command
slog_show // print all set slogs, in hexadecimal format
```


## Functions

### utils
Some functions with names similar to PHP, close to intuition
- url_encode(s: str, encoding: str = 'utf-8') -> str
- url_decode(s: str, encoding: str = 'utf-8') -> str
- base64_decode(s: str, encoding='utf-8') -> str
- base64_encode(s: str, encoding='utf-8') -> str
- json_encode(obj) -> object
- json_decode(data) -> str
- jwt_decode(token: str) -> bytes
- jwt_encode(header: dict, payload: dict, key=None, algorithm=None) -> str
- bin2hex(s: str) -> str
- hex2bin(s: str) -> str
- sha1(s: str, encoding='utf-8') -> str
- sha256(s: str, encoding='utf-8') -> str
- md5(s: str, encoding='utf-8') -> str
- random_int(minN: int = 0, maxN: int = 1024) -> int
- random_string(n: int = 32, alphabet: str = "") -> str

Some functions that may be used in web
- get_flask_pin(username: str,  absRootPath: str, macAddress: str, machineId: str, modName: str = "flask.app", appName: str = "Flask") -> str

- flask_session_helper(*Note: There is no flask dependency in ctfbox itself, the following two functions need to install the dependency by yourself*)
  - flask_session_encode(secret_key: str, payload: dict) -> str
  - flask_session_decode(session_data: str, secret_key: str) -> dict
  ```python
  # Here is example
  sc = '123'
  pl = {
  'user': 'admin',
  'info': 'test'
  }
  ss = 'eyJpbmZvIjoidGVzdCIsInVzZXIiOiJhZG1pbiJ9.YA2WEA.phDDlkaEQOaXthwvpENxAeiHfiE'
  print(flask_session_encode(sc, pl))
  print(flask_session_decode(ss, '123'))
  print(flask_session_decode(ss, '12345'))
  
  # Output
  eyJpbmZvIjoidGVzdCIsInVzZXIiOiJhZG1pbiJ9.YA2XHw.PSPjYFyj3hxsTNx-d2vjncAMJW4
  {'info': 'test', 'user': 'admin'}
  # raise a FlaskSessionHelperError
  ```

Some functions that may be used in reverse engineering
* print data in hex format: `printHex()`
* pack number into bytes: `p16()`, `p32()`, `p64()`
* unpack number from bytes: `u16()`, `u32()`, `u64()`

please refer to source code for functions signatures and usage



Some functions that may be used in misc

***TODO***

Some other functions
- od_parse(data: str) -> Dict[str, Union[str, list]]


### core
Some functions Write by ourselves
- Threader(number: int, timeout: int = None, retry: int = 2)
   ```
    A simple decorator function that can decorate the function to make it multi-threaded.
   ```
   Here is a example.
   ```Python
   from ctfbox import Threader, random_string, random_int
   from time import sleep

   @Threader(10)
   def exp(i: int):
       sleep(random_int(1, 5))
       return "%d : %s" % (i, random_string())
    
    tasks = [exp(i) for i in range(100)] # 100 tasks
    for task in tasks: 
        # task.result return when a task completed
        # task is a concurrent.futures.Future with some sugar attributes
        print('result: %s running: %s done: %s exception: %s' % (task.result, task.running, task.done, task.exception))
   ```
- provide(host: str = "0.0.0.0", port: int = 2005, isasync: bool = False, files: List[Tuple[Union[filepath, content], routePath, contentType]] = {})
   ```
   A simple and customizable http server.
   ```
   Here are some examples.
   ```python
   # provide a exist file named index.html
   provide(files=[('index.html',)])
   # Here is a trick if you provide only one file
   provide(files=['index.html'])
   # route /index.html provide content Hello world\n
   provide(files=[(b"Hello world\\n", "/index.html")])
   # provide some files
   provide(files=[("test.txt", ), ("index.html", )])
   ```
- hashAuth(startIndex: int = 0, endIndex: int = 5, answer: str = "", maxRange: int = 1000000, threadNum: int = 25, hashType: HashType = HashType.MD5) -> str
   ```
   A function used to blast the first few bits of the hash, often used to crack the ctf verification code
   ```
   Here are some examples.
   ```python
   ### HashType optional value: HashType.MD5, HashType.SHA1, HashType.SHA256, HashType.SHA512
   ### Crack the first five number MD5 type ctf verification codes
   print(hashAuth(answer="02fcf"))
   ### Crack the first five number SHA1 type ctf verification codes
   print(hashAuth(answer="d13ce", hashType=HashType.SHA1))
   #### Crack more quickly!!
   print(hashAuth(answer="c907773", endIndex=7, threadNum=50))
   ### Make the range bigger!!
   print(hashAuth(answer="59e711d", endIndex=7, maxRange=2000000))
   ```
- httpraw(raw: Union[bytes, str], **kwargs -> requests.Response)
   ```
   Send raw request by python-requests
   
   Allow kwargs:
   - proxies(dict) : requests proxies
   - timeout(float): requests timeout
   - verify(bool)  : requests verify
   - real_host(str): use real host instead of Host if set
   - ssl(bool)     : whether https
   ```
- gopherraw(raw: str, host: str = "",  ssrfFlag: bool = False) -> str
   ```
   Generate gopher requests URL form a raw http request
   ```
   Here is an example.
   ```python
   raw = """POST /admin HTTP/1.1
   Host: 127.0.0.1:5000
   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:84.0) Gecko/20100101 Firefox/84.0
   Content-Type: application/x-www-form-urlencoded
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
   Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
   Connection: close
   Cookie: isAdmin=1
   Upgrade-Insecure-Requests: 1
   Content-Length: 3

   a=b"""

   print(gopherraw(raw, ssrfFlag=False))
   # output is gopher://127.0.0.1:5000/_%0D%0APOST%20/admin%20HTTP/1.1%0D%0AHost%3A%20127.0.0.1%3A5000%0D%0AUser-Agent%3A%20Mozilla/5.0%20%28Windows%20NT%2010.0%3B%20Win64%3B%20x64%3B%20rv%3A84.0%29%20Gecko/20100101%20Firefox/84.0%0D%0AContent-Type%3A%20application/x-www-form-urlencoded%0D%0AAccept%3A%20text/html%2Capplication/xhtml%2Bxml%2Capplication/xml%3Bq%3D0.9%2Cimage/webp%2C%2A/%2A%3Bq%3D0.8%0D%0AAccept-Language%3A%20zh-CN%2Czh%3Bq%3D0.8%2Czh-TW%3Bq%3D0.7%2Czh-HK%3Bq%3D0.5%2Cen-US%3Bq%3D0.3%2Cen%3Bq%3D0.2%0D%0AConnection%3A%20close%0D%0ACookie%3A%20isAdmin%3D1%0D%0AUpgrade-Insecure-Requests%3A%201%0D%0AContent-Length%3A%203%0D%0A%0D%0Aa%3Db
   # curl this url directly
   ```
## Techniques
- [pdm](https://github.com/frostming/pdm)
- [version-helper](https://github.com/WAY29/version-helper/)

## Depends
- requests
- PyJWT

## Contributors
Syclover
   - [Longlone](https://github.com/way29)
   - [F4ded](https://github.com/F4ded)
   - [lingze](https://github.com/wlingze)
   - [pjx](https://github.com/pjx206)

Other
   - [Morouu](http://github.com/Morouu)

## Logs

### 1.3.0(TODO)
- some functions:
    - flask_session_encode
    - flask_session_decode
    - gopherraw
### 1.2.1
httpraw:
   - fix a bug that httpraw may not be able to send post request correctly
   - fix a bug that could not solve port
   - fix a bug that real_host could not use
   - fix a bug that may cause encoding error
### 1.2.0
- add dev dependencies: icecream
- add some functions:
    - od_parse
    - get_flask_pin
    - httpraw
    - p16 p32 p64 and uXX functions
    - Base32 and Base64 table getter
### v1.1.1
- move project to new directory
- update Readme.md, added missing functions
### v1.1.0
- add pwn part, please see Pwn Usage
- add some functions that may be used in reverse
- update hashAuth functions
  - error if startIndex is less than endIndex
  - if startIndex is zero and length of hash(endIndex - startIndex) is not equal to length of answer, endIndex will be set to length of answer
- update Readme.md, add usage and contributors, Supplementary dependency: PyJWT
### v1.0.2
- update Readme.md
### V1.0.1
- update Readme.md
### V1.0.0
- first commit