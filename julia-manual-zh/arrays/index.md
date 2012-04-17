---
layout: manual
title:  Arrays
---

Julia, like most technical computing languages, provides a first-class array implementation. Most technical computing languages pay a lot of attention to their array implementation at the expense of other containers. Julia does not treat arrays in any special way. The array library is implemented almost completely in julia itself, and derives its performance from the compiler, just like any other code written in julia.

An array is a collection of objects stored in a multi-dimensional grid. In the most general case, an array may contain objects of type `Any`. For most computational purposes, arrays should contain objects of a more specific type, such as `Float64`, `Int32`, etc.

In general, unlike many other technical computing languages, Julia does not expect programs to be written in a vectorized style for performance. Julia's JIT compiler uses type inference and generates optimized code for scalar array indexing, allowing programs to be written in a style that is convenient and readable, without sacrificing performance, and using significantly lesser memory at times.

In Julia, all arguments to functions are passed by reference. Some technical computing languages pass arrays by value, and this is convenient in many cases. In Julia, modifications made to input arrays within a function will be visible in the parent function. The entire Julia array library ensures that inputs are not modified by library functions. User code, if it needs to exhibit similar behaviour, should take care to create a copy of inputs that it may modify.

## Basic Functions

1. `ndims(A)` — the number of dimensions of A
1. `size(A)` — a tuple containing the dimensions of A
1. `eltype(A)` — the type of the elements contained in A
1. `numel(A)` — the number of elements in A
1. `length(A)` — the size of the largest dimension of A
1. `nnz(A)` — the number of nonzero values in A
1. `stride(A,k)` — the size of the stride along dimension k
1. `strides(A)` — a tuple of the linear index distances between adjacent elements in each dimension

## Construction and Initialization

A broad variety of functions for constructing and initializing arrays are provided.
In the following list of such functions, calls with a `dims...` argument can either take a single tuple of dimension sizes or a series of dimension sizes passed as a variable number of arguments.

1. `Array(type, dims...)` — an uninitialized dense array
1. `cell(dims...)` — an uninitialized cell array (heterogeneous array)
1. `zeros(type, dims...)` — an array of all zeros of specified type
1. `ones(type, dims...)` — an array of all ones of specified type
1. `trues(dims...)` — a `Bool` array with all values `true`
1. `falses(dims...)` — a `Bool` array with all values `false`
1. `reshape(A, dims...)` — an array with the same data as the given array, but with different dimensions.
1. `fill(A, x)` — fill the array `A` with value `x`
1. `copy(A)` — copy `A`
1. `similar(A, element_type, dims...)` — an uninitialized array of the same generic type as the given array (dense, sparse, etc.), but with the specified element type and dimensions. The second and third arguments are both optional, defaulting to the element type and dimensions of `A` if omitted.
1. `reinterpret(type, A)` — Construct an array with the same binary data as the given array, but with the specified element type.
1. `rand(dims)` — random array with `Float64` uniformly distributed values in [0,1)
1. `randf(dims)` — random array with `Float32` uniformly distributed values in [0,1)
1. `randn(dims)` — random array with `Float64` normally distributed random values with a mean of 0 and standard deviation of 1
1. `eye(n)` — n-by-n identity matrix
1. `eye(m, n)` — m-by-n identity matrix
1. `linspace(start, stop, n)` — Construct a vector of `n` linearly-spaced elements from `start` to `stop`.

## Comprehensions

Comprehensions provide a general and powerful way to construct arrays. The comprehension syntax is similar to the set construction notation in mathematics:

    A = [ F(x,y,...) | x=rx, y=ry, ... ]

The meaning of this form is that `F(x,y,...)` is evaluated with the variables `x`, `y`, etc. taking on each value in their given list of values. Values can be specified as any iterable object, but will commonly be ranges like `1:n` or `2:(n-1)`, or explicit arrays of values like `[1.2, 3.4, 5.7]`. The result is an N-d dense array with dimensions that are the concatenation of the dimensions of the variable ranges `rx`, `ry`, etc. and each `F(x,y,...)` evaluation returns a scalar.

The following example computes a weighted average of the current element and its left and right neighbour along a 1-d grid.

    julia> const x = rand(10)
    [0.6017125321472665,0.55317268439850298,0.83375372173664064,0.20371170284589835,0.50800458572940888,0.52963052092498386,0.33042233578025493,0.49411133447814293,0.29570938193206264,0.81897111867503525]

    julia> [ 0.5*x[i-1] + x[i] + 0.5*x[i+1] | i=2:length(x)-1 ]
    [1.27090581134045655,1.21219591535884108,0.8745908565789231,0.87467569761484998,0.94884398167981576,0.84229326348181832,0.80717719333430171,0.95225060850865173]

In most high-level technical computing languages, this computation would be performed by computing three vectors (left-shifted, centre, and right-shifted) and then using vector arithmetic, which ends up using at least three times as much memory and perhaps more for temporary vectors that are created along the way. The Julia approach here computes the result vector directly. The code is closer to the math, and thus much more intuitive to understand.

NOTE: In the above example, `x` is declared as constant because type inference in Julia does not work on non-constant global variables.

## Indexing

The general syntax for indexing into an n-dimensional array A is:

    X = A[I_1, I_2, ..., I_n]

where each I_k may be:

1. A scalar value
1. A `Range` of the form `:`, `a:b`, or `a:b:c`
1. An arbitrary integer vector, including the empty vector `[]`

The result X has the dimensions `(size(I_1), size(I_2), ..., size(I_n))`, with location `(i_1, i_2, ..., i_n)` of X containing the value `A[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`.

Indexing syntax is equivalent to a call to `ref`:

    X = ref(A, I_1, I_2, ..., I_n)

Example:

    julia> x = reshape(1:16, 4, 4)
    4x4 Int64 Array
    1 5 9 13
    2 6 10 14
    3 7 11 15
    4 8 12 16

    julia> x[2:3, 2:end-1]
    2x2 Int64 Array
    6 10
    7 11

## Assignment

The general syntax for assigning values in an n-dimensional array A is:

    A[I_1, I_2, ..., I_n] = X

where each I_k may be:

1. A scalar value
1. A `Range` of the form `:`, `a:b`, or `a:b:c`
1. An arbitrary integer vector, including the empty vector `[]`

The size of X should be `(size(I_1), size(I_2), ..., size(I_n))`, and the value in location `(i_1, i_2, ..., i_n)` of A is overwritten with the value `X[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`.

Index assignment syntax is equivalent to a call to `assign`:

      A = assign(A, X, I_1, I_2, ..., I_n)

Example:

    julia> x = reshape(1:9, 3, 3)
    3x3 Int64 Array
    1 4 7
    2 5 8
    3 6 9

    julia> x[1:2, 2:3] = -1
    3x3 Int64 Array
    1 -1 -1
    2 -1 -1
    3 6 9

## Concatenation

Arrays can be concatenated along any dimension using the following syntax:

1. `cat(dim, A...)` — concatenate input n-d arrays along the dimension `dim`
1. `vcat(A...)` — Shorthand for `cat(1, A...)`
1. `hcat(A...)` — Shorthand for `cat(2, A...)`
1. `hvcat(A...)`

Concatenation operators may also be used for concatenating arrays:

1. `[A B C...]` — calls `hcat`
1. `[A, B, C, ...]` — calls `vcat`
1. `[A B; C D; ...]` — calls `hvcat`

## Vectorized Operators and Functions

The following operators are supported for arrays. In case of binary operators, the dot version of the operator should be used when both inputs are non-scalar, and any version of the operator may be used if one of the inputs is a scalar.

1. Unary Arithmetic — `-`
1. Binary Arithmetic — `+`, `-`, `*`, `.*`, `/`, `./`, `\`, `.\`, `^`, `.^`, `div`, `mod`
1. Comparison — `==`, `!=`, `<`, `<=`, `>`, `>=`
1. Unary Boolean or Bitwise — `~`
1. Binary Boolean or Bitwise — `&`, `|`, `$`
1. Trigonometrical functions — `sin`, `cos`, `tan`, `sinh`, `cosh`, `tanh`, `asin`, `acos`, `atan`, `atan2`, `sec`, `csc`, `cot`, `asec`, `acsc`, `acot`, `sech`, `csch`, `coth`, `asech`, `acsch`, `acoth`, `sinc`, `cosc`, `hypot`
1. Logarithmic functions — `log`, `log2`, `log10`, `log1p`, `logb`, `ilogb`
1. Exponential functions — `exp`, `expm1`, `exp2`, `ldexp`
1. Rounding functions — `ceil`, `floor`, `trunc`, `round`, `ipart`, `fpart`
1. Other mathematical functions — `min`, `max,` `abs`, `pow`, `sqrt`, `cbrt`, `erf`, `erfc`, `gamma`, `lgamma`, `real`, `conj`, `clamp`

## Implementation

The base array type in Julia is the abstract type `AbstractArray{T,n}`. It is parametrized by the number of dimensions `n` and the element type `T`. `AbstractVector` and `AbstractMatrix` are aliases for the 1-d and 2-d cases. Operations on `AbstractArray` objects are defined using higher level operators and functions, in a way that is independent of the underlying storage class. These operations are guaranteed to work correctly as a fallback for any specific array implementation.

The `Array{T,n}` type is a specific instance of `AbstractArray` where elements are stored in column-major order. `Vector` and `Matrix` are aliases for the 1-d and 2-d cases. Specific operations such as scalar indexing, assignment, and a few other basic storage-specific operations are all that have to be implemented for `Array`, so that the rest of the array library can be implemented in a generic manner for `AbstractArray`.

`SubArray` is a specialization of `AbstractArray` that perform indexing by reference rather than by copying. A `SubArray` is created with the `sub` function, which is called the same way as `ref` (with an array and a series of index arguments). The result of `sub` looks the same as the result of `ref`, except the data is left in place. `sub` stores the input index vectors in a `SubArray` object, which can later be used to index the original array indirectly.

`StridedVector` and `StridedMatrix` are convenient aliases defined to make it possible for Julia to call a wider range of BLAS and LAPACK functions by passing them either `Array` or `SubArray` objects, and thus saving inefficiencies from indexing and memory allocation.

The following example computes the QR decomposition of a small section of a larger array, without creating any temporaries, and by calling the appropriate LAPACK function with the right leading dimension size and stride parameters.

    julia> a = rand(10,10);

    julia> b = sub(a, 2:2:8,2:2:4)
    4x2 SubArray of 10x10 Float64 Array
    0.48291296659328276 0.31639301252254248
    0.11191852765878418 0.80311033863988501
    0.34377272170384798 0.12998312467801409
    0.75207724893767547 0.48974544536835718

    julia> (q,r,p) = qr(b);

    julia> q
    4x2 Float64 Array
    -0.31610281030340204 0.38994108897230212
    -0.80237370921615103 -0.5848318975546335
    -0.12986390146593485 0.36571345172816944
    -0.48929624071011685 0.61005841520202764

    julia> r
    2x2 Float64 Array
    -1.00091806276211814 -0.65508286752651457
    0.0 0.70738744643074303

    julia> p
    [2,1]
