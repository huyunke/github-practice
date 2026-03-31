## 特性
- uniform是全局的(Global)。全局意味着uniform变量必须在每个**着色器程序对象**中都是独一无二的，而且它可以被着色器程序的任意着色器在任意阶段访问
- uniform值无论设置成什么，uniform会一直保存它们的数据，直到它们被重置或更新

## 声明
```
#version 330 core 
out vec4 FragColor; 

uniform vec4 ourColor; // 在OpenGL程序代码中设定这个变量 

void main() { 
	FragColor = ourColor; 
}
```

**如果声明了一个uniform却在GLSL代码中没用过，编译器会静默移除这个变量，导致最后编译出的版本中并不会包含它，可能会导致错误**

## 获取uniform的位置
```cpp
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
```
**如果 `glGetUniformLocation` 返回 `-1`，通常是因为你在着色器里声明了它但没使用，或者名字拼错了**
查询`uniform`地址不要求你之前使用过着色器程序，但是更新一个`uniform`之前你**必须**先使用程序（调用`glUseProgram`)

## 设置uniform
```cpp
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f; // 随时间周期性变化的颜色值
glUseProgram(shaderProgram); // 更新前必须先激活程序！
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```