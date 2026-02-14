`cmake -P` 称为“脚本模式”，它告知 CMake 这个文件不打算包含 `project（）` 命令。我们不开发任何软件，只是把 CMake 当作命令解释器使用

- cmake里面的”true"
```text
"True"
"On"
"Yes"
"1"
"2"
"10"
"100"
```
- cmake里面的"false"
```text
"False"
"Off"
"No"
"0"
"Ignore"
"NotFound"
""   (空字符串)
```
- CMake 识别的是 **特殊的 `*-NOTFOUND` 失败值**
- 不是“字符串里随便出现 notfound 四个字母”