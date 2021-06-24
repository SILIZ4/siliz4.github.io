---
layout: post
title: Read and write binary files in C++
author: Simon Lizotte
date: '2020-05-21 17:15'
category:
        - guides
        - tutorials
summary: An efficient way to read and write data in C++
thumbnail: binary-cpp.svg
---

## Binary files -- an efficient way to read and write data

We assume here that the reader is somewhat familiar with the different types and the C++ syntax.

### Why binary?
A common way to store data in file is to save it in plain text. In effect, this is simple and convenient. More often than not however, **this is inefficient**: the files are big, slow to write and slow to read. A faster and lighter way of storing data is to directly save its bytes from the memory.

Consider an array of 8 ``float`` values that all have 5 decimals
```cpp
float data[8] = {0.00001, 0.00002, 0.00003, 0.00004,
                 0.00005, 0.00006, 0.00007, 0.00008};
```
In a text file, this data could be saved by separating each value with a space and by writing the numbers in strings. If the file uses the ASCII character set, every character requires 1 byte. This way, each number would occupy 
{% raw %}
$$ 1 \; \frac{\text{byte}}{\text{character}} \times 7\; \frac{\text{characters}}{\text{number}} = 7 \;\frac{\text{bytes}}{\text{number}}.$$
{% endraw %}
In addition, each character is separated by a space, which adds 7 extra characters to the file. In total, the file size would be

$$ 7\;\frac{\text{bytes}}{\text{number}} \times 8\;\text{numbers} + 7\; \text{bytes} = 63 \;\text{bytes}.$$

In constrast, the ``float`` type only occupies 4 bytes of memory. In this manner, if the binary reprensation of the numbers is written in the file, its size is

$$ 8\; \text{numbers}\times 4\;\frac{\text{bytes}}{\text{number}}  = 32 \;\text{bytes}.$$

The file is almost **half the size** of its text counterpart!

### How to implement it?

The process involving writing to binary is more complex and tedious than the usual text file, but it is managable. The first difference is that the flag "binary" must be set to the file stream
```cpp
std::ofstream outputFileStream;
outputFileStream.open(fileName, std::ios::out|std::ios::binary);
```
In order to write the data in the file, it must be casted (transformed) into an array of characters. This can be done with a C-type cast
```cpp
(char*) &data
```
Note here that the **reference of the variable** is used. Here comes the trickier part: the number of bytes saved must be passed in argument to the ``write`` method. Even though there is a built-in function ``sizeof`` that returns the size of a type, one must proceed carefully because it does not always give directly the required size.

In the proposed exampled, there are two ways to go about the writing process. The most intuitive way is to loop on the values stored and write them one by one
```cpp
for(int i=0; i<8; i++)
    outputFileStream.write((char*) &data[i], sizeof(float));
```
The second way involves taking advantage of the contiguous memory used by arrays. When the array ``data`` was declared, 8 locations of 4 bytes were allocated to this variable (the 32 bytes that was shown in the last section). It turns out that these are stored one after the other in memory. Knowing that 32 bytes are required, it is possible to write the entire array with a single line of code

```cpp
outputFileStream.write((char*) data, 8*sizeof(float));
```
The principle to read the data is similar. The input stream is opened in binary
```cpp
std::ifstream inputFileStream;
inputFileStream.open(fileName, std::ios::in|std::ios::binary);
```
When the file is read with ``read``, the bytes are copied into the variable passed in argument. In the same manner as for the writing, the size read in bytes must be known. 

There are once again two options to read the data: fill each element of the array one by one with a loop
```cpp
float fileContent[8];
for(int i=0; i<8; i++)
    inputFileStream.read((char*) &fileContent[i], sizeof(float));
```
and fill the array all at once
```cpp
inputFileStream.read((char*) &fileContent, 8*sizeof(float));
```
Although it is possible to save more complex data type such as structures and classes in binary, this guide don't explain it this as it very quickly becomes complicated. A good place to start for a interested reader is (here)[serialization-cpp]. 

Here is a simple program that writes ``data`` into a binary file ``foo.bin``, reads it and output the values to the console.
```cpp
#include <fstream>
#include <iostream>


using namespace std;

int main(){
    float data[8] = {0.00001, 0.00002, 0.00003, 0.00004,
                     0.00005, 0.00006, 0.00007, 0.00008};
    ofstream outputFileStream("foo.bin", ios::out | std::ios::binary);
    for (int i=0; i<8; i++)
        outputFileStream.write((char*) &data[i], sizeof(float));

    // Alternative to write the data
    // outputFileStream.write((char*) data, 8*sizeof(float));

    outputFileStream.close();

    float fileData[8];
    ifstream inputFileStream("foo.bin", ios::in | ios::binary);
    cout << "File data is\n";
    for (int i=0; i<8; i++)
        inputFileStream.read((char*) &fileData[i], sizeof(float));

    // Alternative to read the data
    // inputFileStream.read((char*) &fileData, 8*sizeof(float));

    for (int i=0; i<8; i++)
        cout << fileData[i] << ", " << endl;
    cout << "\nFinished reading\n";
    inputFileStream.close();

    remove("foo.bin");
    return 0;
}
```

[serialization-cpp]: https://isocpp.org/wiki/faq/serialization
