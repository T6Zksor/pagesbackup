# Modern C++ get buffer
2017-07-12

``` cpp
std::unique_ptr<char[]> buffer(new char[n]);
buffer.get()
```