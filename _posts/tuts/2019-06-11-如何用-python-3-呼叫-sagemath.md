---
layout: post
title: 如何用 Python 3 呼叫 SageMath
date: 2019-06-11 11:50 +0800
tags:
  - Development
---

之前跟我的基友 [maojui](https://github.com/maojui) 想他的 crypto 懶人包要怎麼 call SageMath。因為 SageMath 本身是修改過的 Python 2，似乎也沒有提供 package 讓人用，所以就想說寫個簡單的 wrapper，感覺滿舒服的。

Python 有個套件方便你開 subprocess，而且可以與之溝通。直覺想到可以開個 SageMath 的子 process，透過 socket 丟程式去給他跑，用 pickle 包執行結果回來。

我會先建立一個暫時的 socket 檔案，把 Python 3 當成 server：

```python
import tempfile
import socket

sock_path = tempfile.mktemp()
server = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
server.bind(sock_path) # 開 unix socket 在暫存路徑上
server.listen(1)       # 只處理一個 client

# 建立 SageMath client 讓它連過來
# ...

client, _ = server.accept()          # 這裡等候 SageMath 連過來
client_file = client.makefile('rwb') # 用這個送程式碼給 SageMath 跑
```

接著來建立 subprocess，以下是我想讓 SageMath 跑的 Python 2 程式：

```python
def __main():
    import socket, pickle, base64
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.connect(SERVER_SOCKET)
    sock_file = sock.makefile('rwb')
    while True:
        code = eval(sock_file.readline())
        try:
            result = __execute(code)
            result = pickle.dumps(result)
            result = base64.b64encode(result)
        except:
            import traceback
            print traceback.format_exc()
            result = base64.b64encode(pickle.dumps(None))
        sock_file.write(result + '\n')
        sock_file.flush()

def __execute(code):
    __return_value = None
    exec(code)
    return __return_value

__main()
```

> 為求方便，如果程式碼能用一行來表示，那我們 client 只要用 readline 收資料就好了，剛好 python 有 `repr()` 可以把多行字串壓成一行，client 收到只要用 `eval()` 就能轉換回來。
> 
> 要讓 SageMath client 在收到程式碼之後可以執行，這邊是利用 `exec()`，`exec()` 執行的結果會存在 `__return_value`，之後是用 pickle 包好透過 socket 傳回 Python 3 server。
> 
> 因為不想要送過去的程式碼污染變數，所以放在一個 function 裡面執行，這樣區域變數在 function 結束之後就會消滅。

接著來開 SageMath 吧！

```python
sage_script = f'''
def __main():
    import socket, pickle, base64
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.connect({repr(sock_path)})
    sock_file = sock.makefile('rwb')
    while True:
        code = eval(sock_file.readline())
        try:
            result = __execute(code)
            result = pickle.dumps(result)
            result = base64.b64encode(result)
        except:
            import traceback
            print traceback.format_exc()
            result = base64.b64encode(pickle.dumps(None))
        sock_file.write(result + '\\n')
        sock_file.flush()
def __execute(code):
    __return_value = None
    exec(code)
    return __return_value
__main()
''')

import subprocess as sp
sage_proc = sp.Popen(['sage', '-c', sage_script], stderr=sp.DEVNULL)
```
> 最後一行開 SageMath 執行上一步的 code，他會連到 Python 3 server 等候程式碼。

然後我們就可以這樣呼叫 SageMath。

```python
code = '''
__return_value = factor(29229)
'''

code_bytes = (repr(code) + '\n').encode()

client_file.write(code_bytes)
client_file.flush()

import pickle, base64, dill

dill._dill._reverse_typemap['ObjectType'] = object
# https://rebeccabilbro.github.io/convert-py2-pickles-to-py3/

result = client_file.readline()
result = base64.b64decode(result)
result = pickle.loads(result, encoding='bytes')

print(result)
```

整個包好的 class 長這樣：

[https://github.com/djosix/codes/blob/master/other/sage_wrapper.py](https://github.com/djosix/codes/blob/master/other/sage_wrapper.py)

