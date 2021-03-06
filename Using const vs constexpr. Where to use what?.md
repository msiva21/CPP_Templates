### Intro
- While introducing myself to C++ & its new features introduced in C++11 & C++14, I have completely neglected this keyword `constexpr`. 
- Initially, it was a bit confusing & comparing `constexpr` with `const` which was not allowing new thought & thinking in my mind about how this `constexpr` works & differ with `const`. So, I have studied this from different sources & here is the consolidation of it:
### `constexpr` with primitive variables
```c++
int varA = 3;
const int varB = 5;
constexpr int varC = 7;
```
- All of the above variable having a value which is known at compile time. `varA` is normal scenario while `varB` & `varC` will not take further value or assignment. So they are fixed at compile time if we have defined them like above.
- But, `varB` is not the right way(in some situation) of declaring constant value at compile time. For example, if I declare them as follows:
```c++
int getRandomNo()
{
  return rand() % 10;
}

int main()
{
    const int varB = getRandomNo();       // OK
    constexpr int varC = getRandomNo();   // not OK! compilation error

    return 0;
}
```
- Value of `varB` would not be anymore compile time. While statement with `varC` will throw compilation error. The reason is `constexpr` will always accept strictly compile time value.
### Function as `constexpr`
```c++
constexpr int sum(int x, int y)
{
    return x + y;
}

int main()
{
    const int result = sum(10, 20);     // Here, you can use constexpr as well
    cout << result;
    return 0;
}
```
- `constexpr` specifies that the value of an object, variable or a function can be evaluated strictly at compile time and the expression can be used in other constant expressions. 
```
+--------------------------------------+--------------------------------------+
|       int result = sum(10, 20);      |    const int result = sum(10, 20);   |
+--------------------------------------+--------------------------------------+
|  1   main:                           |      main:                           |
|  2   ....                            |      ....                            |
|  3   ....                            |      ....                            |
|  4   ....                            |      ....                            |
|  5           subl    $20, %esp       |                subl    $20, %esp       |
|  6           subl    $8, %esp        |              movl    $30, -12(%ebp)  | <----- Direct result substitution
|  7           pushl   $20             |              subl    $8, %esp        |
|  8           pushl   $10             |              pushl   $30             |
|  9           call    _Z3sumii        |              pushl   $_ZSt4cout      |
|  10          addl    $16, %esp       |              call    _ZNSolsEi       |
|  11          movl    %eax, -12(%ebp) |      ....                            |
|  12          subl    $8, %esp        |      ....                            |
|  13          pushl   -12(%ebp)       |      ....                            |
|  14          pushl   $_ZSt4cout      |                                      |
|  15          call    _ZNSolsEi       |                                      |
|  16  ....                            |                                      |
|  17  ....                            |                                      |
|  18  ....                            |                                      |
+--------------------------------------+--------------------------------------+
```
- If you observe above code, you can see that when you catch result as `const` or `constexpr`, call to function `sum` is not there in assembly rather compiler will execute that function by it self at compile time & substitute the result with function.
- By specifying `constexpr`, we suggest compiler to evaluate function `sum` at compile time.
### `constexpr` with constructors
```c++
class INT 
{ 
    int _no; 
public: 
    constexpr INT (int no) : _no(no){}       
    constexpr int getInt ()  {   return _no; } 
}; 
  
int main() 
{ 
    constexpr INT obj(INT(5).getInt()); 
    cout << obj.getInt(); 
    return 0; 
} 
```
- Above code is simple & self-explanatory.
### `constexpr` vs `const`
- They serve different purposes. `constexpr` is mainly for optimization while `const` is for practically `const` objects like the value of `Pi`.
- Both of them can be applied to member methods. Member methods are made `const` to make sure that there are no accidental changes by the method. On the other hand, the idea of using `constexpr` is to compute expressions at compile time so that time can be saved when the code is running.
- `const` can only be used with non-static member function whereas `constexpr` can be used with member and non-member functions, even with constructors but with condition that argument and return type must be of literal types.

### Where to use what?
- Where you need value not often & calculating it would be a bit complex, then that is the place you need `constexpr`. Otherwise, things are fine with older buddy `const`. For example, Fibonacci number, factorial, etc.
```c++
constexpr unsigned int factorial(unsigned int n)
{
    return (n <= 1) ? 1 : (n * factorial(n - 1));
}

static constexpr auto magic_value = factorial(5);
```
- Often programmer would suggest using `constexpr` instead of a macro. 
- Sometimes, you have something that can be evaluated down to a constant while maintaining good readability and allowing slightly more complex processing than just setting a constant to a number. For example:
```c++
template< typename Type > 
constexpr Type max( Type a, Type b ) 
{ 
    return a < b ? b : a; 
}
```
Its a pretty simple choice there but it does mean that if you call `max` with constant values it is explicitly calculated at compile time and not at runtime.
- Another good example is converting units like
```c++
const float rupee = dollarToRupee( 9.4 );
```
Here you can use `constexpr`.


### References
- https://blog.quasardb.net/2016/11/22/demystifying-constexpr
- https://en.cppreference.com/w/cpp/language/constexpr
- https://www.geeksforgeeks.org/understanding-constexper-specifier-in-c/
