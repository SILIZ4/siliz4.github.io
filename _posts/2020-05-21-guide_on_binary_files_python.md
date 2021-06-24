---
layout: post
title: Read and write binary files in Python
author: Simon Lizotte
date: '2020-05-21 17:15'
thumbnail: binary-python.svg
---

## Binary files -- an efficient way to read and write data
In general, inputing and outputing information on a data storage is a rather slow process that should be avoided as much as possible. However, there are many cases where it is necessary. When the files are reasonably small, the difference between an efficient and an inefficient process is not noticeable. But when the files grow bigger or when the reading and writing are done oftenly, there can be a significant loss of performance by using text files. This guide will demonstrate with an example how one could improve the efficiency of a program that needs to do such operations.

## Why care about binary?
When data is stored in a file, it is actually always written in a binary format. However, in a text file, each character is kept separately and, most often than not, this ends up occupying more space than it should.

Consider a dataset composed of a sequence of 20 numbers between 0 and 0.3.


```python
import numpy

data = numpy.linspace(0, 0.3, 20).astype(numpy.float32)
data
```




    array([0.        , 0.01578947, 0.03157895, 0.04736842, 0.06315789,
           0.07894737, 0.09473684, 0.11052632, 0.12631579, 0.14210527,
           0.15789473, 0.17368421, 0.18947369, 0.20526315, 0.22105263,
           0.23684211, 0.25263157, 0.26842105, 0.28421053, 0.3       ],
          dtype=float32)



A easy and convenient way to store ``data`` is to write the values in a regular file by separating each number by a space.


```python
def save_array_in_text(array, file_name):
    with open(file_name, 'w') as file_stream:
        for value in array:
            file_stream.write(str(value))
            file_stream.write(' ')


save_array_in_text(data, "data.txt")
with open("data.txt", 'r') as file_stream:
    print(file_stream.readline())
```

    0.0 0.015789473 0.031578947 0.047368422 0.06315789 0.078947365 0.094736844 0.110526316 0.12631579 0.14210527 0.15789473 0.17368421 0.18947369 0.20526315 0.22105263 0.23684211 0.25263157 0.26842105 0.28421053 0.3 


The data is simply retrieved by parsing the text file.


```python
with open("data.txt", 'r') as file_stream:
    file_data = file_stream.readline().strip()  # Strip removes the trailing white space

file_data = file_data.split(' ')
numpy.array(file_data, dtype=float)
```




    array([0.        , 0.01578947, 0.03157895, 0.04736842, 0.06315789,
           0.07894737, 0.09473684, 0.11052632, 0.12631579, 0.14210527,
           0.15789473, 0.17368421, 0.18947369, 0.20526315, 0.22105263,
           0.23684211, 0.25263157, 0.26842105, 0.28421053, 0.3       ])



Note the size of the file in bytes.


```python
from os import path, remove

path.getsize("data.txt")
```




    212



## How to save into binary?

The first step consists of transforming the array in bytes. This is simply accomplished by the built-in function ``bytearray``.


```python
bytearray(data)
```




    bytearray(b'\x00\x00\x00\x00\xedX\x81<\xedX\x01=d\x05B=\xedX\x81=(\xaf\xa1=d\x05\xc2=\x9f[\xe2=\xedX\x01>\x0b\x84\x11>(\xaf!>F\xda1>d\x05B>\x810R>\x9f[b>\xbd\x86r>\xedX\x81>|n\x89>\x0b\x84\x91>\x9a\x99\x99>')



Now, one simply needs to add the letter "b" in the ``open`` built-in function to save in a binary format.


```python
def save_array_in_binary(array, file_name):
    with open(file_name, "wb") as file_stream:
        file_stream.write(bytearray(array))


save_array_in_binary(data, "data.bin")
with open("data.bin", "rb") as file_stream:
    print(file_stream.readline())
```

    b'\x00\x00\x00\x00\xedX\x81<\xedX\x01=d\x05B=\xedX\x81=(\xaf\xa1=d\x05\xc2=\x9f[\xe2=\xedX\x01>\x0b\x84\x11>(\xaf!>F\xda1>d\x05B>\x810R>\x9f[b>\xbd\x86r>\xedX\x81>|n\x89>\x0b\x84\x91>\x9a\x99\x99>'


Let's now inspect the size of this file.


```python
path.getsize("data.bin")
```




    80



This is a significant difference: the file is around **38% the size of the other!**

The values in binary can be read with the function ``fromfile`` of the ``numpy`` library.


```python
with open("data.bin", "rb") as file_stream:
    loaded_from_binary = numpy.fromfile(file_stream)
loaded_from_binary
```




    array([3.00928836e-17, 1.28047338e-13, 8.04184663e-12, 1.33571292e-10,
           1.01955355e-09, 4.15664034e-09, 1.69402660e-08, 6.90159393e-08,
           1.89480061e-07, 3.81469873e-07])



This doesn't seem correct, right? This is one of the drawback of using binary files: if the type is incorrect, the data won't be retrieved properly. The data was saved in the type ``float32``, but by default, ``numpy`` assumes the data is in the ``float64`` type.


```python
print("Saved data type:", data.dtype)
print("Loaded data type:", loaded_from_binary.dtype)
```

    Saved data type: float32
    Loaded data type: float64


This problem is solved by specifying to ``numpy`` the correct data type.


```python
with open("data.bin", "rb") as file_stream:
    loaded_from_binary = numpy.fromfile(file_stream, dtype=numpy.float32)
loaded_from_binary
```




    array([0.        , 0.01578947, 0.03157895, 0.04736842, 0.06315789,
           0.07894737, 0.09473684, 0.11052632, 0.12631579, 0.14210527,
           0.15789473, 0.17368421, 0.18947369, 0.20526315, 0.22105263,
           0.23684211, 0.25263157, 0.26842105, 0.28421053, 0.3       ],
          dtype=float32)



## Important note and further discussion
Even though here it was shown that writing ``data`` in a binary file is more efficient, it is not always the case. For instance, if the data had been saved in ``float64``, the size of the binary file would be doubled. Furthermore, the elements of ``data`` have a lot of decimals and each decimal has to be represented by a character in the text file. By using numbers with fewer decimals, the text format would actually fare better:


```python
data = numpy.linspace(0, 9.5, 20).astype(numpy.float64)
save_array_in_text(data, "data.txt")
save_array_in_binary(data, "data.bin")
print("data:", data)
print("Size of text file", path.getsize("data.txt"))
print("Size of binary file", path.getsize("data.bin"))

remove("data.txt")
remove("data.bin")
```

    data: [0.  0.5 1.  1.5 2.  2.5 3.  3.5 4.  4.5 5.  5.5 6.  6.5 7.  7.5 8.  8.5
     9.  9.5]
    Size of text file 80
    Size of binary file 160


In order to maximise the saved space and the effiency benefits, one must chose wisely the type used to save. In these examples, each character saved in the text file requires 1 byte and each number is seperated by a space. Each element in the data then occupies at minimum 2 bytes. But this size can be surpassed easily by using a ``float16`` which always occupies 2 bytes of memory.

For those interested, the standard and most common types are listed [here][ctypes].

[ctypes]: https://www.tutorialspoint.com/cprogramming/c_data_types.htmhere
