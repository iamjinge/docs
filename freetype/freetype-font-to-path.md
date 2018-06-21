# 使用freetype提取字形

> Android使用freetype绘制文字大概有两种思路，一种是提取文字的**bitmap**进行绘制，一种是提取文字**路径**进行绘制，后者能更方便对文字绘制做一些变化。这里是基于freetype的[example](https://www.freetype.org/freetype2/docs/tutorial/example5.cpp)，将文字字形提取到安卓的[Path](https://developer.android.com/reference/android/graphics/Path)里面进行文字绘制。

[example](https://www.freetype.org/freetype2/docs/tutorial/example5.cpp)里面先加载字体，然后提取文字字形，再将文字路径输出到一个svg文件中。

### 安卓freetype初始化
[这里](https://juejin.im/post/5b24dd55f265da597c771f17)是从源码编译freetype并初始化安卓jni工程的例子。

## 初始化字体
```cpp
FT_Library library;
FT_Face face;
FT_Error error = FT_Init_FreeType(&library);
error = FT_New_Face(library, "/sdcard/temp/font.ttf", 0, &face);
error = FT_Set_Pixel_Sizes(face, 0, 16);
```
设置字体大小的时候需要注意，根据[文档](https://www.freetype.org/freetype2/docs/glyphs/glyphs-6.html)提取的路径会是字体大小的64倍，所以需要在适当的时候除以64。

## 加载字形
```cpp
FT_UInt glyph_index = FT_Get_Char_Index(face, static_cast<FT_ULong>(wChar));
error = FT_Load_Glyph(face, glyph_index, FT_LOAD_DEFAULT);
```

## 翻转文字
字体文件中的Y轴正方向和我们要绘制的Y轴正方向相反，这里需要翻转一下。
```cpp
const FT_Fixed multiplier = 65536L;
FT_Matrix matrix;
matrix.xx = 1L * multiplier;
matrix.xy = 0L * multiplier;
matrix.yx = 0L * multiplier;
matrix.yy = -1L * multiplier;
FT_GlyphSlot slot = face->glyph;
FT_Outline &outline = slot->outline;
FT_Outline_Transform(&outline, &matrix);
```

## 解析字形到Path
```cpp
FT_Outline_Funcs callbacks;
callbacks.move_to = MoveToFunction;
callbacks.line_to = LineToFunction;
callbacks.conic_to = ConicToFunction;
callbacks.cubic_to = CubicToFunction;
callbacks.shift = 0;
callbacks.delta = 0;

FT_GlyphSlot slot = face->glyph;
FT_Outline &outline = slot->outline;
FT_Error error = FT_Outline_Decompose(&outline, &callbacks, jPath);
```
`FT_Outline_Decompose`函数有三个参数，`outline`带有字形；`callback`解析字形的回调。这四个函数在[Path](https://developer.android.com/reference/android/graphics/Path)有一一对应的方法
| FT_Outline_Funcs | Path |
| -------- | ------ |
| move_to | moveTo |
|line_to | lineTo |
|conic_to| quadTo|
|cubic_to | cubicTo |
第三个参数是void类型的指针，这个指针传递到`callback`的回调函数中，这里的`jPath`是一个从Java传进来的Path对象，后面在`callback`的回调里面使用反射调用path里面的函数。例如MoveToFunction中的内容
```cpp
int MoveToFunction(const FT_Vector *to, void *user)
{
    JPath *jPath = static_cast<JPath *>(user);

    FT_Pos x = to->x;
    FT_Pos y = to->y;

    jclass pathClass = jPath->env->GetObjectClass(jPath->path);
    jmethodID method = jPath->env->GetMethodID(pathClass, "moveTo", "(FF)V");
    jPath->env->CallVoidMethod(jPath->path, method, (float) x, (float) y);
    return 0;
}
```

## 绘制
解析字形到Path之后，画到Canvas之前将Path缩小为原来的1/64，然后直接使用调用`Canvas.drawPath`就可以了。

[一个提取文字路径到Path例子](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fiamjinge%2FAndroidFreetypeSample)