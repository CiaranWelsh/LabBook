C code for wrapping
======================


void fakeFun(double *d) {
    printf("I'm a double pointer from C: %f, %f\n", *d, *(d + 1));
}


void function_that_takes_a_function(f1 fn, double *input, double *output) {
    printf("From C: I'm a function that takes a function\n");
    printf("From C: input: %f\n", *input);
    printf("From C: output before callback %f\n", *output);

    (*fn)(input, output);
    printf("From C: output after callback: %f\n", *output);
}

void function_that_takes_f2(f2 fn, double *input, double *output, double *ignored) {
    printf("From C: I'm a function that takes a function\n");
    printf("From C: input: %f\n", *input);
    printf("From C: output before callback %f\n", *output);

    (*fn)(input, output, ignored);
    printf("From C: output after callback: %f\n", *output);
}

void function_that_takes_ESfcnFG(ESfcnFG fg) {
    printf("From C: hello from function_that_takes_ESfcnFG\n");
    double *x;
    x[0] = 1.0;
    x[1] = 2.0;
    double f = 1.0;
    double g = 0.0;
//    fg(x, &f, &g);
}


Point *makePoint(int x, int y) {
    /**
     *
     * https://stackoverflow.com/questions/38661635/ctypes-struct-returned-from-library
     */

    Point *point = (Point *) malloc(sizeof(Point));
    int *p1 = (int *) malloc(sizeof(int));
    int *p2 = (int *) malloc(sizeof(int));
    point->x = x;
    point->y = y;
    return point;
}

void freePoint(Point *point) {
    free(point);
}

Structures with ctypes
----------------------


class Point(ct.Structure):
    """Example of using structs with ctypes.
    https://stackoverflow.com/questions/65901925/how-to-create-a-c-struct-from-python-using-ctypes/65902099?noredirect=1#65902099
    On the C end we have:

        typedef struct Point {
            int x;
            int y;
        } Point ;

        Point* makePoint(int x, int y){
            Point *point = (Point*) malloc(sizeof (Point));
            point->x = x;
            point->y = y;
            return point;
        }

        void freePoint(Point* point){
            free(point);
        }

    Then in Python:

        >>> lib = ct.CDLL(library)
        >>> lib.makePoint.restype = ct.POINTER(Point) # aka this class
    Then you can do
        >>> lib.makePoint.restype = ct.POINTER(Point)
        >>> p = lib.makePoint(4, 5)
        >>> print(p.contents.x)
        >>> print(p.contents.y)
    Be exceptionally careful with types! int in C maps to c_int32 in ctypes.

    """
    _fields_ = [
        ("x", ct.c_int32),
        ("y", ct.c_int32),
    ]


Callbacks
------------




class Test(unittest.TestCase):

    def setUp(self) -> None:
        pass

    def test_ctypes_callback_fn_example(self):
        from ctypes import cdll

        libc = cdll.msvcrt

        IntArray5 = ct.c_int * 5
        ia = IntArray5(5, 1, 7, 33, 99)
        qsort = libc.qsort
        qsort.restype = None

        CALLBACK_FN = ct.CFUNCTYPE(ct.c_int, ct.POINTER(ct.c_int), ct.POINTER(ct.c_int))

        def py_cb(a, b):
            print("Pycb", a[0], b[0])
            return 0

        qsort(ia, len(ia), ct.sizeof(ct.c_int), CALLBACK_FN(py_cb))

    def testPassingDoubleArray(self):
        lb = ct.pointer(sres.DoubleArrayLen2(0.1, 0.1))  # double *lb,
        sres.fakeFun(lb)

    def testPassingDoubleArrayUsingWrapperFn(self):
        lb = sres._makeDoubleArrayPtr([0.1, 0.1])  # double *lb,
        sres.fakeFun(lb)

    def testFunctionPointerInIsolation(self):
        import ctypes as ct

        lib = ct.CDLL("SRES")

        F1_CALLBACK = ct.CFUNCTYPE(None, ct.POINTER(ct.c_double * 2), ct.POINTER(ct.c_double), ct.POINTER(ct.c_double))
        lib.function_that_takes_a_function.argtypes = [
            F1_CALLBACK, ct.POINTER(ct.c_double * 2),
            ct.POINTER(ct.c_double),
            ct.POINTER(ct.c_double)
        ]
        lib.function_that_takes_a_function.restype = None

        def func_to_pass_in(d1, d2, d3):
            print("hello from Python: ")

        # function_that_takes_a_function(func_to_pass_in, sres._makeDoubleArrayPtr([0.1, 0.1]))
        lib.function_that_takes_a_function(F1_CALLBACK(func_to_pass_in), sres._makeDoubleArrayPtr([0.1, 1.2]),
                                           ct.pointer(ct.c_double(4.0)), ct.pointer(ct.c_double(6.0)))

    def testFunctionPointerInIsolationAndUpdateAValue(self):
        import ctypes as ct

        lib = ct.CDLL("SRES")

        F1_FUNCTION_PTR = ct.CFUNCTYPE(None, ct.POINTER(ct.c_double), ct.POINTER(ct.c_double))

        lib.function_that_takes_a_function.argtypes = [
            F1_FUNCTION_PTR, ct.POINTER(ct.c_double), ct.POINTER(ct.c_double)
        ]
        lib.function_that_takes_a_function.restype = None

        def func_to_pass_in(x, y):
            print("From Python: hello from Python: ")
            print("From Python: x, y: ", x.contents, y.contents)
            new_value = x.contents.value + y.contents.value
            new_value_double_ptr = ct.pointer(ct.c_double(new_value))

            ct.memmove(ct.cast(y, ct.c_void_p).value,
                       ct.cast(new_value_double_ptr, ct.c_void_p).value,
                       ct.sizeof(ct.c_double))

        # function_that_takes_a_function(func_to_pass_in, sres._makeDoubleArrayPtr([0.1, 0.1]))
        input = ct.c_double(4.0)
        output = ct.c_double(1.0)
        input_ptr = ct.pointer(input)
        output_ptr = ct.pointer(output)
        lib.function_that_takes_a_function(F1_FUNCTION_PTR(func_to_pass_in), input_ptr, output_ptr)

    def testFunctionPointerInIsolationAndUpdateAValue2(self):
        import ctypes as ct

        lib = ct.CDLL("SRES")

        ESFcnFG_FUNCTION_PTR = ct.CFUNCTYPE(None, ct.POINTER(ct.c_double * 2), ct.POINTER(ct.c_double),
                                            ct.POINTER(ct.c_double))

        lib.function_that_takes_f2.argtypes = [
            ESFcnFG_FUNCTION_PTR, ct.POINTER(ct.c_double), ct.POINTER(ct.c_double), ct.POINTER(ct.c_double)
        ]

        lib.function_that_takes_f2.restype = None

        def func_to_pass_in(x, y, z):
            print("hello from Python: ")
            print("x, y: ", x.contents[0], y.contents)
            new_value = x.contents[0] + x.contents[1] + y.contents.value
            new_value_double_ptr = ct.pointer(ct.c_double(new_value))

            ct.memmove(
                ct.cast(y, ct.c_void_p).value,
                ct.cast(new_value_double_ptr, ct.c_void_p).value,
                ct.sizeof(ct.c_double))

        # function_that_takes_a_function(func_to_pass_in, sres._makeDoubleArrayPtr([0.1, 0.1]))
        input = ct.pointer(ct.c_double(4.0))
        output = ct.pointer(ct.c_double(1.0))
        ignored = ct.pointer(ct.c_double(2.0))
        lib.function_that_takes_f2(ESFcnFG_FUNCTION_PTR(func_to_pass_in), input, output, ignored)

    def test_use_the_problematic_function_pointer_outside_context_of_SRES(self):
        import ctypes as ct

        lib = ct.CDLL("SRES")

        ESfcnFG_TYPE = ct.CFUNCTYPE(None, ct.POINTER(ct.c_double * 2), ct.POINTER(ct.c_double), ct.POINTER(ct.c_double))

        lib.function_that_takes_ESfcnFG.argtypes = [
            ESfcnFG_TYPE
        ]
        lib.function_that_takes_ESfcnFG.restype = None

        def cost_fun(x, f, g):
            print("hello from cost_fun")
            sim = generateData(x.contents[0], x.contents[1])
            cost = 0
            for i in range(10):
                cost += (EXP_DATA[i] - sim[i]) ** 2
            cost_dbl_ptr = ct.pointer(ct.c_double(cost))

            # copy the value from Python to C. If we don't do this, the value gets deleted.
            ct.memmove(ct.cast(f, ct.c_void_p).value, ct.cast(cost_dbl_ptr, ct.c_void_p).value, ct.sizeof(ct.c_double))

        lib.function_that_takes_ESfcnFG(ESfcnFG_TYPE(cost_fun))































