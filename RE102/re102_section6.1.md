---
layout: default
permalink: /RE102/section6.1/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 6.1: The Unpacking Script #

Here is the full unpacking script. Extract the raw resource 1000 as a binary file as the first argument. It will output the exe as decrypted_payload.exe. You can also find this script here: [re102_unpacker.py](https://github.com/securedorg/nobarxtx.github.io/blob/master/RE102/%5Cnobarxtx.github.io/RE102/re102_unpacker.py)

```
import os
import sys

# key sizes
key1size = 32  # 0x20
key2size = 64  # 0x40

# offset to key2
key2_offset = 720  # 2D0

# offset to payload
payload_offset = 792  # 0x318

# offset to first instruction
junk_char_length_offset = 100  # 0x64
good_char_length_offset = 168  # 0xA8

# header bytes
header = 'MZ'


def key_schedule(_key):
    key = _key
    if not isinstance(_key, list):
        key = list(_key)
    keylength = len(key)
    S = range(256)
    j = 0
    for i in range(256):
        k = key[i % keylength]
        if not isinstance(key[i % keylength], int):
            k = ord(key[i % keylength])
        j = (j + S[i] + k) % 256
        S[i], S[j] = S[j], S[i]  # swap
    return S, j


def RC4_decrypt(_key, _enc):
    result = []
    S, j = key_schedule(_key)
    enc = _enc
    if not isinstance(_enc, list):
        enc = list(_enc)
    m = len(enc)
    i = 0
    for c in enc:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        k = S[(S[i] + S[j]) % 256]
        m -= 1
        if not isinstance(c, int):
            result.append(ord(c) ^ k)
        else:
            result.append(c ^ k)
    return result


def deflate_payload(junk, good_data, _data):
    data = list(_data)
    new_data = []
    j = 0
    k = 0
    new_data.append(ord(data[0]))
    i = 1
    while i < len(data):
        if j != junk:
            j = junk
            i += j
        if j == junk:
            if k < good_data:
                new_data.append(ord(data[i]))
                k += 1
                i += 1
            else:
                j = 0
                k = 0
    return new_data


with open(sys.argv[1], 'rb') as encrypted_file:

    # Get the file size
    file_size = os.path.getsize(sys.argv[1])

    # Get the first key of 0x20 bytes
    key1 = encrypted_file.read(key1size)

    # Get the start of deflate instructions
    encrypted_info = encrypted_file.read(payload_offset - key1size)

    # Get the bytes for the payload
    encrypted_payload = encrypted_file.read(file_size-payload_offset)

    # 1. Decrypt the deflation instructions
    decrypted_info = RC4_decrypt(key1, encrypted_info)

    # 2. Get instructions from the decrypted data
    junk_char_length = decrypted_info[junk_char_length_offset-key1size]
    good_char_length = decrypted_info[good_char_length_offset-key1size]

    # 3. deflate payload
    deflated_payload = deflate_payload(junk_char_length, good_char_length, encrypted_payload)

    # 4. get key2
    key2 = []
    for i in range(key2size):
        key2.append(decrypted_info[(key2_offset-key1size)+i])

    # 5. decrypt enc3
    decrypted_payload = RC4_decrypt(key2, deflated_payload)

    # 6. fix header
    for i in range(len(header)):
        decrypted_payload[i] = ord(header[i])

    # write to file
    with open("decrypted_payload.exe", 'wb') as out:
        for i in decrypted_payload:
            out.write(chr(i))
        out.close()
```

[Section 6 <- Back](https://nobarxtx.github.io/RE102/section6) | [Next -> Extra Fun](https://nobarxtx.github.io/RE102/section7)
