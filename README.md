# Owl - An OCaml Math Library
Owl is an OCaml math library. It supports both dense and sparse matrix operations, linear algebra, regressions, and many advanced mathematical and statistical functions (such as Markov chain Monte Carlo methods).

## Installation

Installation is rather trivial. First, you need to clone the repository.

```bash
git clone git@github.com:ryanrhymes/owl.git
```

Then you need to install all the dependencies. Owl depends on gsl, ctypes, plplot, and several other modules. Check the `_oasis` file in the `owl` folder to find out the complete list of required modules.

Next, you can compile and install the module with the following command.

```bash
make oasis
make && make install
```

If you want `utop` to automatically load Owl for you, you can also edit `.ocamlinit` file in your home folder by adding the following lines.

```bash
#require "Owl"
#require "ctypes"
#require "ctypes.foreign"
```

Owl is well integrated with `utop`. Now you can start `utop` and continue this tutorial to do some experiments.


## Create Matrices

To start, we can use `Dense.uniform_int` to create a 5x5 random dense matrix.

```ocaml
let x = Dense.uniform_int 5 5;;
```

You should see the following output in `utop`.

```bash
   C0 C1 C2 C3 C4
R0 25  2 77 85 72
R1 71 92 98 87 53
R2 35 29 82 65 20
R3  2 29 66 42 12
R4 99 72 78 30 11
val x : Dense.dsmat =
```

`Dense` module also provides other functions to create various matrices, e.g., as below.

```ocaml
let x = Dense.eye 5;;             (* identity matrix *)
let x = Dense.zeros 5 5;;         (* all elements are zeros *)
let x = Dense.ones 5 5;;          (* all elements are ones *)
let x = Dense.uniform 5 5;;       (* random matrix of uniform distribution *)
let x = Dense.gaussian 5 5;;      (* random matrix of gaussian distribution *)
...
```

Combined with `Stats` module, you can also create any matrices of many distributions. E.g., the following code first creates an empty dense matrix, then initialise the elements with Bernoulli distribution. Test it in `utop`, you should get a dense matrix where half of the elements are zeros.

```ocaml
let x = Dense.empty 8 8 |> Dense.map (fun _ -> Stats.Rnd.bernoulli 0.5 |> float_of_int);;
```

Or create a matrix where the elements follow Laplace distribution.

```ocaml
let x = Dense.empty 8 8 |> Dense.map (fun _ -> Stats.Rnd.laplace 0.2);;
```

With `Dense` module, you can also generate linearly spaced interval and meshgrids, e.g.,

```ocaml
let x = Dense.linspace 0. 5. 6;;
```
which will return a 1x5 row vector as below
```bash
   C0  C1  C2  C3  C4  C5
R0  0   1   2   3   4   5
val x : Dense.dsmat =
```

Matrices can be saved to and loaded from a file.

```ocaml
Dense.save x 'matrix_01.data';;  (* save the matrix to a file *)
Dense.load 'matrix_01.data';;    (* load the matrix from a file *)
```


## Access Elements, Rows, and Columns

Both `Dense` and `Sparse` modules provide a wide range of operations to access the elements, rows, and columns of a matrix. You can use `Dense.set` and `Dense.get` to manipulate individual element.

```ocaml
Dense.set x 0 1 2.5;;
Dense.get x 0 1;;
```

Equivalently, there are shorthands for `Dense.get` and `Dense.set`.

```ocaml
x.{0,1} <- 2.5;;  (* Dense.set x 0 1 2.5 *)
x.{0,1};;         (* Dense.get x 0 1 *)
```

We can use `Dense.row` and `Dense.col` to retrieve a specific row or column of a matrix, or use `Dense.rows` and `Dense.cols` to retrieve multiple of them.

```ocaml
Dense.row x 5;;        (* retrieve the fifth row *)
Dense.cols x [|1;3;2|] (* retrieve the column 1, 3, and 2 *)
```

E.g., the following code generates a random matrix, then scales up each element by a factor of 10 using `Dense.map` function.

```ocaml
let x = Dense.(uniform 6 6 |> map (fun x -> x *. 10.));;
```

We can iterate a matrix row by row, or column by column. The following code calculates the sum of each row by calling `Dense.map_rows` function.

```ocaml
let x = Dense.(uniform 6 6 |> map_rows sum);;
```

We can fold elements by calling `Dense.fold`, fold rows by calling `Dense.fold_rows`. Similarly, there are also functions for `filter` operations. The following code filters out the elements not greater than 0.1 in x.

```ocaml
Dense.filter ((>) 0.1) x;;
```

We can also do something more complicated, e.g., by filtering out the rows whose summation is greater than 3.

```ocaml
Dense.filter ((>) 0.1) x;;    (* not greater than 0.1 in x *)
```

Shuffle the rows and columns, or draw some of them from a matrix.

```ocaml
Dense.shuffle_rows x;;                (* shuffle the rows in x *)
Dense.draw_cols x 3;;                 (* draw 3 columns from x with replacement *)
...
```

Practically, `Sparse` module provides all the similar operations for sparse matrices. In addition, `Sparse` module also has extra functions such as only iterating non-zero elements `Sparse.iter_nz`, and etc. Please read the full documentation for details.


## Linear Algebra

Simple matrix mathematics like add, sub, multiplication, and division are included in `Dense` module. Moreover, there are predefined shorthands for such operations. E.g., the following code creates two random matrices then compare which is greater.

```ocaml
let x = Dense.uniform 6 6;;
let y = Dense.uniform 6 6;;
Dense.(x >@ y)                  (* is x greater than y? *)
Dense.(x =@ y)                  (* is x equal to y? *)
...
```

Some basic math operations includes:

```ocaml
Dense.(x +@ y)                  (* add two matrices *)
Dense.(x *@ y)                  (* multiply two matrices, element-wise *)
Dense.(x $@ y)                  (* dot product of two matrices *)
Dense.(x +$ 2.)                 (* add a scalar to all elements in x *)
...
```

Apply various functions in `Maths` module to every element in x

```ocaml
Dense.(Maths.sin @@ x);;        (* apply sine function *)
Dense.(Maths.exp @@ x);;        (* apply exponential function *)
...
```

Concat two matrices, vertically or horizontally by

```ocaml
Dense.(x @= y);;                (* equivalent to Dense.concat_vertical *)
Dense.(x @|| y);;               (* equivalent to Dense.concat_horizontal *)
```

More advanced linear algebra operations such as `svd`, `qr`, and `cholesky` decomposition are included in `Linalg` module.

```ocaml
let u,s,v = Linalg.svd x   (* singular value decomposition *)
let q,r = Linalg.qr x      (* QR decomposition *)
let l = Linalg.cholesky x  (* cholesky decomposition *)
...
```


## Regression

`Regression` module currently includes `linear`, `exponential`, `nonlinear`, `ols`, `ridge`, `lasso`, `svm`, and etc. Most of them are based on a stochastic gradient descent algorithm implemented in `Optimise` module.

In the following, let's use an example to illustrate the simplest linear regression in `Regression` module. First, let's generate the measurement x which is a 1000 x 3 matrix. Each row of x is an independent measurement.

```ocaml
let x = Dense.uniform 1000 3;;
```

Next let's define the parameter of a linear model, namely p, a 3 x 1 matrix.

```ocaml
let p = Dense.of_array [|0.2;0.4;0.8|] 3 1;;
```

Then we generate the observations y from x and p by

```ocaml
let y = Dense.(x $@ p);;
```

Now, assume we only know x and y, how can we fit x and y into a linear model? It is very simple.

```ocaml
let p' = Regression.linear x y;;
```

From `utop`, you can see that p' equals `[|0.2; 0.4; 0.8|]` which is exactly the same as p. For other regression such as `lasso` and `svm`, the operation is more or less the same, please read Owl document for details.


## Plotting



## Conclude
