```cpp
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```
- 第一个参数指定了我们绘制的模式
- 第二个参数是我们打算绘制顶点的个数
- 第三个参数是索引的类型，这里是GL_UNSIGNED_INT
- 最后一个参数指定EBO中的偏移量（或者传递一个索引数组，但是这是当你不在使用索引缓冲对象的时候）