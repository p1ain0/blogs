# Taichi

## 安装

pip install taichi

## Taichi程序的组成

Data ---> Computation ---> Visualization

### 初始化

```python
import taichi as ti
ti.init(arch=ti.gpu) # The entry point for All Taichi project
```

arch:
| | CPU | CUDA | OpenGL | Apple Metal | Vulkan |
|-| -- | -- | -- | -- | -- |
| Window | Yes | Yes | Yes | No | WIP |
| Linux | Yes | Yes | Yes | No | WIP |
| MacOS | Yes | No  | No | Yes | WIP |

- taichi-scope
 - Everthing decorated by @ti.kernel or @ti.func is in the Taichi-scope

```python
import taichi as ti
ti.init(arch=ti.gpu)

@ti.kernel  #Taichi scope 
def fun():
    print("This is now a Tachi kernel")
fun()
```



### taichi Data

基础类型：
signed integers : ti.i8, ti.i16, ti.i32, ti.i64
unsigned intergers : ti.u8, ti.u16, ti.u32, ti.u64
floating points : ti.f32, ti.f64

| Type | i8 | i16 | i32 | i64 | u8 | u16 | u32 | u64 | f32 | f64 |
| - | - | - | - | - | - | - | - | - | - | - |
| CPU/CUDA | yes | yes | yes | yes | yes | yes | yes | yes | yes | yes |
| Metal | yes | yes | yes | No | yes | yes | yes | No | yes | No |
| OpenGL | No | No | Yes | Ext | No | No | No | No | yes | yes |

Default types
- using int or float for default types
- can be changed via ti.init

```
ti.init(default_fp=ti.f32) # float = ti.f32
ti.init(default_fp=ti.f64) # float = ti.f64

ti.init(default_fp=ti.i32) # int = ti.i32
ti.init(default_fp=ti.i64) # int = ti.i64
```

Type promiotions

- Taichi picks the more precise type to store the algebraic results
 - i32+f32=f32
 - i32+i64=i64

#### type cast

- Implicit cast 
  - static types within the Taichi scope
  
```
import taichi as tiles
ti.init(arch=ti.cpu)
def foo()
	a=1
	a=2.7
	print(a)
foo() #2.7

import taichi as tiles
ti.init(arch=ti.cpu)
@ti.kernel
def foo()
	a=1
	a=2.7
	print(a)
foo() #2
```
不同类型之间做转换：
variable = ti.cast(variable, type)

#### Compound type

- Using ti.types to create compound types including:
 - vector / matrix / struct
 
- Access compound elements using [i,j,k,...] indexing

```python
vec3f = ti.types.vector(3, ti.f32)
mat2f = ti.types.matrix(2, 2, ti.f32)
ray = ti.types.struct(ro=vec3f, rd=vec3f, l=ti.f32)

也可以使用taichi里边内置的类型：
ti.Vector()
ti.Matrix()
ti.Struct()
```

ti.field
- a global N-d array of elements
  - global:can be read/written from both the Taichi-scope and the Python-scope
  - N-d:(Scalar:N=0),(Vector:N=1), (Matrix:N=2),(N = 3,4,5,...)
  - elements: scalar, vector, matrix, struct
  - access elements in a field using [i,j,k,...] indexing
	- Special case,access a zero-d field using [None]
	
ti.field examples

- 3D gravitational field in a 256*256*128 room

```
gravitational_field = ti.Vector.field(n = 3,dtype=ti.f32, shape=(256,256,128))
```

- 2D strain-tensor field in a 64*64 grid

```
strain_tensor_field = ti.Matrix.field(n=2, m=2, dtype=ti.f32, shape=(64,64))
```

- a global scalar that I want to access in a Taichi kernel

```
global_scalar = ti.field(dtype=ti.f32, shape=())
```



heat_field = ti.field(dtype=ti.f32, shape=(256,256))

### taichi计算核

Kernels

- A python function decorated by @ti.kernel is a Taichi kernel 
 - Taichi kernels can only be called from the Python scope

```
import taichi as ti
ti.init(arch=ti.cpu)
@ti.kernel
def foo():
	pass

@ti.kernel
def bar()
	pass
	
#OK
foo()
bar()

#OK
def foo2()
	foo()

#not OK
@ti.kernel
def bar2():
	foo()

```

For-loops in a @ti.kernel

- For loops at the outermost scope in a Taichi kernel is automatically parallelized
最外层的循环会并行计算

- Design your for loops for best performance

- Race condition
  - Taichi uses += as an atomic add
  - The compiler optimizes for unnecessary atomic opetations

- Type of for-loops in Taichi
  - range-for: loops over a range, identical to Python range-for
  - struct-for:loops over a ti.field, only lives at the outermost scope


参数：
1.最多8个参数
2.从python作用域传到taichi作用域
3.必须是type-hinted
4.scalar only
5.pass by value

返回值：
1.可以返回不返回
2.从python作用域传到taichi作用域
3.必须是type-hinted

Functions
- A Python function decorated by @ti.func is a Taichi function 
  - Taichi functions can only be called from the Taichi scope

Taichi funcitons are force-inlined(不支持递归)

Arguments and return values
- Do not need to be type-hinted
- Arguments pass by value


函数调用：
taichi函数只能从taichi scope中被调用

Anything in @ti.kernel and @ti.func are in the Taichi scope

Static data type in the Taichi scope
Static lexical scope in the Taichi scope

The @ti.kernes and @ti.func will be compiled JIT.
The Taichi scope variables can not see the Python scope variables at run time

Math ops：

```
ti.sin(x)
ti.cos(x)
ti.asin(x)
ti.acos(x)
ti.atan2(y, x)
ti.sqrt(x)
ti.cast(x, data_type)


ti.floor(x)
ti.ceil(x)
ti.inv(x)
ti.tan(x)
ti.tanh(x)
ti.exp(x)
ti.log(x)

ti.random(data_type)
abs(x)
int(x)
float(x)
max(x, y, ...)
min(x, y, ...)
x ** y

A.transpose()
A.inverse()
A.trace()
A.determinant()
v.normalized()
A + B, A * B, A @ B, ...

R, S = ti.polar_decompose(A, ti.f32)
U, sigma , V = ti.svd(A, ti.f32)
lam, V = ti.eig(A, ti.f32)
u.dot(v) # scalar
u.cross(v) # vector
u.outer_product(v) # matrix

```

### taichi可视化

- The print() in GPU is not likely to show until seeing ti.sync()


2D：ti.GUI

    set your window: gui=ti.GUI("Title", res=(1024,768))
    paint on a window: gui.set_image()
    elements: gui.circles(),gui.lines(), gui.rects(), gui.triangles()...
    widgets: gui.button(), gui.slider(), gui.text()...
    event: gui.get_events(), gui.get_key_event(), gui.running...

### 面向对象编程

```python
@ti.data_oriented
class ...
```

## 程序动画

步骤：

    1.setup your canvas
    2.put some colors on your canvas
    3.Draw a basic unit
    4.Repeat the basic units:tiles and fractals
    5.Animate your picture
    6.Introduce some randomness

Remark
- Taichi scope v.s. Python scope
 - Taichi scope is "on device", "statically-typed", "strongly-typed"
- @ti.kernel
 - “__global__” functions in CUDA
 - The outermost scope for-loop in a @ti.kernel is parallelized
- @ti.func
 - “__device__” functions in CUDA
 - force-inlined

## 提高代码质量

Reusability、Extensibility、Maintainability

### 元编程(metaprogramming)

```python
@ti.kernel
def copy_4(src:ti.template(), dst:ti.template()):
    for i in range(4):
        dst[i] = src[i]
a = ti.field(ti.f32, 4)
b = ti.field(ti.f32, 4)
c = ti.field(ti.f32, 12)
d = ti.field(ti.f32, 12)

copy(a, b, 4)
copy(c, d, 12)
```

使用ti.template()传递参数，

- 允许你传递任何taichi支持的东西：
  - 基础的taichi类型ti.f32、ti.i32、ti.f64 ...
  - 复合的taichi类型ti.Vector(),ti.Matrix()
  - taichi field: ti.field(), ti.Vector.field(), ti.Matrix.field(), ti.Struct.field()
  - Taichi classes:@ti.data_oriented
- Pass-by-reference, use with cautions
  - Computations in the Taichi scope can NOT modify Python scope data
  - Computations in the Taichi scope can modify Taichi fields
  - Computations in the Taichi scope can modify Taichi scope 

Copy a Taichi field to another

Metadata描述数据的数据

- Field:
  - field.dtype:type of a field
  - field.shape:shape of a field
- Matrix/Vector
  - matrix.n:rows of a mat
  - maxtrix.m:cols of a mat/vec

Use ti.template() with caution

Taichi kernels are instantiated whenever seeing a new parameter(even same typed)

Metaprogramming in Taichi

Unify the development of dimensionality-dependent code, such as 2D/3D physical simulations

Improve run-time performance by taking run-time costs to compile time

ti.static()

Compile-time branching

### 面向对象编程(Object-oriented programming)

Python OOP + Taichi DOP = ODOP
 Objective data-oriented programming(ODOP)


## 大规模计算的关键-高级数据结构

Taichi 是一个面向数据的编程语言

before we go: packed mode
- Initialized in ti.init()
- Decides whether to pad the data to the power of two
  - Default choice:packed=False. will do the padding
  - We assume packed=True in this class for simplicity
  
```
ti.init() # default: packed=False
a = ti.field(ti.i32, shape=(18,65))) #padded to (32, 128)

ti.init(packed=True)
a = ti.field(ti.i32, shape=(18,65))) # no padding
```

Taichi: optimized for data-access

```
x=ti.field(ti.i32, shape=16)

@ti.kernel
def fill()
	for i in x:
		x[i] = i
fill()
```

Layout101: from shape to ti.root

```
x=ti.Vector.field(3.ti.f32,shape = 16)
                 |
				 |
x = ti.Vector(3, ti.f32)
ti.root.dense(ti.i, 16).place(x)
```

Each cell of root has a dense container with 16 cells along the ti.i axis. Each cell of a dense container has field x



