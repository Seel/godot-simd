# godot-simd
Godot module adding support for SIMD instructions from gdscript etc.

# About
This is a proof of concept module for high performance coding from gdscript and other scripting languages.

You can use these arrays to perform basic math but considerably faster than gdscript would normally perform, I have profiled at usually between 100x and 1000x faster.

The speedup is due to:
1) Linear ideally packed aligned layout in memory
2) Designed for easy compiler optimization
3) Autovectorization to SSE or Neon instructions where available
4) SSE / Neon intrinsics when CPU support is detected

### New classes
## FastArray_4f32
This is an array made up of 4 32bit floats, corresponding to the `__m128` SSE type and `float32x4_t` in Neon. Instead of having a loop in gdscript and calling a function multiple times, instead here the math functions take a `from` and `to` argument to pass the array range to apply the function to. This can either be used for either full 4 float data types (e.g. Quat, Plane) or Vector3 operations.

## FastArray_2f32
An array of pairs of 32bit floats, for use with e.g. Vector2. The functions are all ranged Vector2 equivalents.

## FastArray_4i32
4 32bit integers.

## Vec4_i32
Simple Vector4 integer class. Can be used for passing values to FastArray_4i32 or for your own use.

# ToDo
I have started making optional SSE versions of the functions. There is CPU detection code to detect CPU caps at startup, and dynamic dispatch to the fastest path supported by the hardware. The SSE code can be compiled out by switching out defines.

The architecture specific code is compiled in separate translation units, with specific compiler flags set in the scons SCsub. This is to prevent higher version code being generated by the compiler and used in hardware that does not support it (which would cause a crash). This is indicated by the gcc docs and the technique used by google chrome.

See:

https://gcc.gnu.org/onlinedocs/gcc-4.5.3/gcc/i386-and-x86_002d64-Options.html
https://randomascii.wordpress.com/2016/12/05/vc-archavx-option-unsafe-at-any-speed/

# FastArray_4f32 Functions
Functions include:
* read / writing elements and reserving array
* read / writing as pool arrays
* add, subtract, multiply, divide
* sqrt, inverse sqrt, reciprocal
* vec3: dot, cross, unit cross, normalize
* vec3: length, length squared
* vec3: xform, inverse xform

# Example
```
const size = 5000000

func TestSIMD():

	# create SIMD array
	var arr = FastArray_4f32.new()
	arr.reserve(size)

	# fill
	for i in range (size):
		arr.write(i, Quat(1, 1, 1, 1))

	# perform ranged function (single instruction, multiple data)
	arr.value_add(Quat(1, 1, 1, 1), 0, size)

	# read
	var q : Quat
	for i in range (size):
		q = arr.read(i)
		# do something with q
```

# FastArray_2f32 Functions
* read / write, reserve
* add, subtract, multiply, divide
* sqrt, inverse sqrt, reciprocal
* dot, cross, normalize
* length, length squared

# ToDo
* Thread safety
* Alignment
* Strided generic versions of functions (for use on external arrays)

# Notes
As yet some operations are difficult / impossible to autovectorize and require compiling without disabling intrinsics for maximum speed. These include sqrt, inverse sqrt and operations which depend on these.
