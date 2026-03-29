## 使用
```cpp
glUseProgram(shaderProgram); glBindVertexArray(VAO); glDrawArrays(GL_TRIANGLES, 0, 3);
```
- 第一个参数：打算绘制的OpenGL图元的类型，`GL_TRIANGLES`三角形
- 第二个参数：顶点数组的起始索引
- 第三个参数：绘制的顶点数量