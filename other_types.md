# Other MATLAB Datatypes
Originally, MATLAB arrays only went up to two dimensions and contained values stored as double precision floating point.
However, the language evolved to work with arrays of more dimensions, with different datatypes.
Other data structures were also introduced, such as ``struct`` and ``cell``, which makes some things easier.

## Representing numbers
### Floating point
Floating point numbers are those that can contain fractions, and are not limited to whole numbers.
These can be double precision or single precision.  
#### Double precision floating point
These are the most accurate representations of floating point numbers, but they need more memory than single precision.
When numerical precision is needed, then double precision is preferred. Each number is represented by 8 bytes of memory.
The arrays we have generated so far have used double precision.  Other representations can be converted to double precision using the ``double`` function.

#### Single precision floating point
This representation of floating point numbers uses four bytes, and is less precise than double precision.
A double precision array can be converted to single precision using the ``single`` function.
```matlab
d = [1e-8 1 -1];
s = single(d);
```
One way that we can look at the effects of numerical precision is by adding numbers together.  Try comparing the results of the following:
```matlab
sum(d)
sum(s)
sum(d(3:-1:1))
sum(s(3:-1:1))
```
The ``sum(s)`` command produced a value of zero.  This is because in single precision, a value of 1 is indistinguishable from a value of 1.00000005.

### Integers
Many types of whole number can be represented in MATLAB. These are determined by the number of bytes used to encode them and whether they are signed or unsigned.
Unsigned representations don't allow negative values to be represented, whereas signed representations allow negative values.
Available integer types are:
* **uint8** Unsigned 8 bit integers. One byte. Values can range from 0 to 255.
* **int8** Signed 8 bit integers. One byte. Values can range from -128 to 127.
* **uint16** Unsigned 16 bit integers. Two bytes. Values range from 0 to 65535.
* **int16** Signed 16 bit integers. Two bytes. Values range from -32768 to 32767.
* **uint32** Unsigned 32 bit integers. Four bytes. Values range from 0 to 4294967295.
* **int32** Signed 32 bit integers. Four bytes. Values range from -2147483648 to 2147483647.
* **uint64** Unsigned 64 bit integers. Eight bytes. Values range from 0 to 18446744073709551615.
* **int64** Signed 64 bit integers. Eight bytes. Values range from -9223372036854775808 to 9223372036854775807.

Images, such as 
