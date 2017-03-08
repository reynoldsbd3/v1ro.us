---
author:
  description: Software Engineer
  email: reynoldsbd3@hotmail.com
  github: https://github.com/reynoldsbd3
  image: /images/avatar-64x64.png
  name: Bobby Reynolds
  website: https://www.reynoldsbd.net/
cardbackground: '#263238'
cardheaderimage: /images/default.jpg
cardthumbimage: /images/default.jpg
categories:
- presentation
date: 2017-03-07T15:17:00Z
description: Modules and features that make Python a perfect language for hackers
tags:
- python
title: The Python Hacker's Toolkit
---

# The Python Hacker's Toolkit

Modules and features that make Python a perfect language for hackers

???

Python is an ideal language for all sorts of things, including CTF problems and other cybersecurity applications. This
is not just because of it's succinct syntax and easy manipulation of binary data, but also because of the huge amount of
first- and third-party libraries available.

This presentation demonstrates some of what I think are the most useful of these libraries, those that have and will
continue to be incredibly useful for working on CTFs and in Cybersecurity.

---

# Struct

???

The first module I want to talk about is `struct`. Python `struct`'s are a really simple way to convert between binary
data formats and usable Python data types. I'll just jump straigt into an example.

--

Data format:

```c
struct message {
    int msg_no;
    unsigned sender_id;
    unsigned dest_id;
    char subject[32];
    char body[256];
};
```

???

Suppose we're dealing with a program that sends messages over the network. The messages are sent using this format,
which includes some numeric information about the message as well as the subject and body of the message.

--

Intercepted data:

```python
raw_data = b'\x00\x00\x00\x01\x00\x00\x00\x02\x00\x00\x00\x03hello\x00...' \
        b'\x00this is the message body\x00...'
```

???

Now suppose we've done a packet capture and retrieved this raw data, shown here as a Python byte array, and we want to
write a program that can take these raw bytes and print the message to the screen. This is exactly what the `struct`
module makes easy.

---

# Struct

```python
import struct

msg_struct = struct.Struct('!i2I32s256s')

msg = msg_struct.unpack(raw_data)
subj = msg[3].decode().strip("\x00")
body = msg[4].decode().strip("\x00")

print(f'Message number: {msg[0]}')
print(f'Sender ID: {msg[1]}')
print(f'Receiver ID: {msg[2]}')
print(f'Subject: {subj}')
print(f'Message:\n\n{body}')
```

???

The module lets you create an instance of a `Struct` object with this kooky-looking format string. Then, you can call
the `.unpack()` method on that object with the raw binary data and get back a tuple object containing Python data types.

Number values can be used right away without any more work. Strings, however, need a little more love before we can
print them to the string.

---

# Struct

```python
'!i2I32s256s'
```

Format string provides instructions for interpreting raw bytes:

???

So let's pick apart the struct format string and understand exactly how it's used.

--

* "!" -> network byte order

???

The first character tells `struct` that we want to interpret integers using network byte order. There are other
characters we can use to specify little- or big-endian.

--
* "i" -> `int msg_no;`

???

The "i" encodes a single 4-byte integer.

--
* "2I" -> `unsigned sender_id; unsigned dest_id;`

???

A capital "I" encodes an *unsigned* integer, and the number 2 before it means there are 2 of them to decode.

--
* "32s" -> `char subject[32];`

???

The little "s" character is used to encode `char` arrays. In this case, the number beforehand gives us the length of the
array, rather than the number of arrays.

--
* "256s" -> `char body[256];`

???

Same deal here, just a bigger array to hold the body.

--

Reading strings:

```python
>>> msg = msg_struct.unpack(raw_data)
>>> type(msg[3])
<class 'bytes'>
>>> print(msg[3])
b'hello\x00\x00\x00\x00\x00\x00\x00\x00\x00...'
>>> print(msg[3].decode().strip('\x00'))
hello
```

???

There are a million and one ways to interpret a string, but the `struct` module does not attempt to do any
interpretation. Instead, it just passes back the bytes that make up the string, and it's up to you to decode things.

--

[Official Struct Documentation](https://docs.python.org/3/library/struct.html)

???

Since `struct` is part of the Python standard library, the entire specification of these format strings is available in
the official Python docs.

---

# Third-Party Modules

Installing packages with `pip`:

```bash
$ pip search crypto
...
$ pip install pycrypto
...
$ pip uninstall pycrypto
```

[Documentation](https://packaging.python.org/installing/)

???

It's possible to extend Python's out of the box functionality with 3rd-party `import`-able modules using the `pip`
package manager. This tool is used to download and install 3rd party packages from the Python package index.

Note that if your system calls the Python command `python3` then you'll probably have to use `pip3` here.

---

# Requests

```bash
$ pip install requests
```

[Official Docs](http://docs.python-requests.org/en/master/)

???

The `requests` module is used to make HTTP requests across the internet. It doesn't necessarily add functionality to
Python's built-in `urllib` module, but it presents a way more user friendly API.

I'm going to go through a few examples of what you can do with either, demonstrating how the API's differ and why this
module should be part of your toolkit.

---

# Requests

Simple GET using `urllib`:

```python
>>> import urllib.request
>>> r = urllib.request.urlopen('http://python.org/')
>>> print(r.read(50))
b'<!doctype html>\n<!--[if lt IE 7]>   <html class="n'
```

???

Using the built-in `urllib`, this is how you would issue an HTTP GET request and read the result.

--

The same, using `requests`:

```python
>>> import requests
>>> print(requests.get('http://python.org/').text[:50])
<!doctype html>
<!--[if lt IE 7]>   <html class="n
```

???

The same operation using `requests` is a bit less verbose, and has the advantage that the result is already interpreted
as a unicode string rather than raw bytes.

---

# Requests

GET request with parameters in `urllib`:

```python
>>> import urllib.parse
>>> import urllib.request
>>> data = {}
>>> data['name'] = 'this is my name'
>>> data['password'] = '12345'
>>> params = urllib.parse.urlencode(data)
>>> print(params)
name=this+is+my+name&password=12345
>>> r = urllib.request.urlopen('http://python.org/' + '?' + params)
```

???

This is the standard way of sending GET requests with parameters. That SHA-1 problem for Boston Key Party accepted input
this way.

--

GET with parameters in `requests`:

```python
>>> import requests
>>> params = {'name': 'this is my name', 'password': '12345'}
>>> r = requests.get('http://python.org/', params=params)
```

???

This is the same operation using `requests`. It's significantly less verbose!

---

# Twisted

```bash
$ pip install twisted
```

[Official Website](https://twistedmatrix.com/trac/)

???

Twisted is a pretty extensive framework for writing network applications. It minimizes the amount of work necessary to
set up and use network sockets or deal with binary data, allowing you to focus on the business logic of your network
program.

It's kinda tough to give bite-sized examples for `twisted`, so instead I'm going to walk through a working demo of two
network programs to demonstrate how `twisted` makes things easier.

--

[**Excellent** Tutorial](http://krondo.com/an-introduction-to-asynchronous-programming-and-twisted/)

???

I could do a whole talk on Twisted; there are so many things you can do with it. For a deep dive, I've linked to an
excellent series of articles about Twisted and it's asynchronous programming model.

---

# PyCrypto

Non-Windows:

```bash
$ pip install pycrypto
```

???

PyCrypto implements a variety of cryptography-related functions and classes. It's pretty much a one-stop-shop for
anything crypto.

Unfortunately, it doesn't quite build properly on Windows; I think there's an installer available somewhere, but I
wouldn't really trust it.

---

# PyCrypto

Encrypting data with AES:

```python
>>> from Crypto import Random
>>> from Crypto.Cipher import AES

>>> plaintext = 'hello, friend!'.encode().ljust(AES.block_size, b'\x00')
>>> key = b'thisismykeypleasedonotstealitok?'
>>> iv = Random.new().read(AES.block_size)

>>> ciphertext = AES.new(key, AES.MODE_CBC, iv).encrypt(plaintext)
```

???

Here's an example of encrypting some arbitrary data using the AES algorithm. It's pretty straightforward, you just
build a cipher using the key and the IV, then use that cipher to encrypt the plaintext.

The plaintext needs to be padded to a length that's a multiple of the AES block size. In real life, you'll probably need
to do some math to determine the right way to pad; here, I've just assumed that the message fits in one block.

Also notice that my key is exactly 32 bytes long. This tells PyCrypto that we want to use 256-bit AES. In general, the
key needs to be a power of 2.

--

Decryption:

```python
>>> data = AES.new(key, AES.MODE_CBC, iv).decrypt(ciphertext)
>>> print(data.decode().strip('\x00'))
hello, friend!
>>>
```

???

And decryption is just as easy!

---

# PyCrypto

Generating an RSA key:

```python
>>> from Crypto.PublicKey import RSA

>>> keypair = RSA.generate(2048)
>>> pubkey = keypair.publickey()
```

???

PyCrypto also handles assymetric encryption like a champ. This code generates a fresh new RSA key pair.

--

Encrypting:

```python
>>> ciphertext = pubkey.encrypt(plaintext, 0)
```

???

Again, encryption is as easy as calling encrypt with the public key. In this case, no padding or IV is required.

--

Decrypting:

```python
>>> data = keypair.decrypt(ciphertext)
>>> print(data.decode().strip('\x00'))
hello, friend!
```

???

Then, you can use the private key to decrypt the message. Notice that once decrypted, there will be some null-byte
padding that needs to be stripped off.

---

# PyCrypto

Exporting RSA keys:

```python
>>> with open('key.pem', 'wb') as f:
        f.write(keypair.exportKey(passphrase='passw0rd'))
```

???

Exporting a key to a PEM-encoded file is trivial. PyCrypto even allows you to specify a passphrase to encrypt the key.

--

```python
>>> with open('key.pem') as f:
        keypair = RSA.importKey(f.read(), passphrase='passw0rd')
```

???

Importing an RSA key is just as easy.
