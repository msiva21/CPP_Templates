### Introduction
- This is a series of articles(total of three) in which i have tried to describe the working virtual keyword thoroughly. Although I am not an expert but this is what I have learned so far from various sources & industry experience. So these articles are basically a collection of my connected dots on virtual keyword.
- Before learning All about the virtual keyword in C++, I would like to clarify two things
    1. Implementation of a virtual function is purely compiler dependent
    2. No C++ standard defined implementation. C++ standard only states behavior.
- I always ask my self before learning anything new "Why it needed in the first place?". So, let's start from there:
### Why we need a virtual function?
- Let us understand it with an example. Suppose you want to connect to the network or to other mobile using your smartphone.
- You have two choices Bluetooth or Wifi. Although these two are completely different technologies, still some things are common in them at an abstract level like both are communication protocol, both need authentication, etc.
- Let say we have a class of them as follows:
```
class wifi_t{
    private:
        char _pass[15];
        // storage ...
    public:
        void authenticate();
        void connect();
        // operations ...
};

class bluetooth_t{
    private:
        char _pass[15];
        // storage ...
    public:
        void authenticate();
        void connect();
        // operations ...
};
```
- Now, below is the main application in which you want to connect your device to others.
```
int main()
{
    wifi_t         *wifi = new wifi_t;
    bluetooth_t     *bluetooth = new bluetooth_t;

    int pt = selectProtocol();
    
    if(pt == BLUETOOTH){
        bluetooth->authenticate();
        bluetooth->connect();
    }
    else if(pt == WIFI){
        wifi->authenticate();
        wifi->connect();
    }
    return 0;
}
```
- If you observe above code then you will find that despite selecting any protocol some steps are same.
- In this case, you can leverage virtual keyword functionality of C++ as follows:

```
class protocol_t{
    private:
        uint8_t _type;
        // storage ...
    public:
        virtual void authenticate(){};
        virtual void connect(){};
        // operations ...
};

class wifi_t : public protocol_t{
    private:
        char _pass[15];
        // storage ...
    public:
        virtual void authenticate(){};
        virtual void connect(){};
        // operations ...
};

class bluetooth_t : public protocol_t{
    private:
        char _pass[15];
        // storage ...
    public:
        virtual void authenticate(){};
        virtual void connect(){};
        // operations ...
};

void makeConnection(protocol_t *protocol)
{
    protocol->authenticate();
    protocol->connect();
}    

int main()
{
    int pt = selectProtocol();
  
    makeConnection( (pt == WIFI) ? new wifi_t : new bluetooth_t);    // You wont be able to compile this line, but i have kept it for simplysity

    return 0;
}
```

As you can see there are some benefits we have achieved using virtual keywords are:
1. **Run time polymorphism**: Behavioural functions will be identified at runtime & would be called by their type like if `protocol` is wifi then execute `wifi_t::authenticate()` & `wifi_t::connect()`.
2. **Reusability of code**: Observe `makeConnection` function there is an only single call to behavioural functions we have removed the redundant code from main.
3. **Code would be compact**: Observe earlier `main` function & newer one.

### How virtual function works
- When you declare any function virtual, the compiler will transform(augment is the precise word here) some of your code at compile time.
- Like in our case class `protocol_t` class object will be augmented by a pointer called `_vptr` which points to the virtual table.
- This is nothing but a pointer(`_vptr`) which points to an array of a function pointer which includes offset/address of your virtual function. So that it can call your function through that table rather than calling it directly by adding offset to `this` pointer.

So if you call function `authenticate()` using a pointer of type `protocol_t` 
```
protocol_t *protocol;
protocol->authenticate();
```
then it would probably be augmented by compiler like this
```
( * protocol->vptr[ 1 ])( protocol ); 
```
Where the following holds:
1. `_vptr` represents the internally generated virtual table pointer inserted within each object whose class
declares or inherits one or more virtual functions. In practice, its name is mangled. There may be
multiple vptrs within a complex class derivation.
2. `1` in `_vptr[ 1 ]` is the index into the virtual table slot associated with `authenticate()`.
3. `protocol` in its second occurrence represents the `this` pointer.

- When we inherit `protocol_t` class to `wifi_t` class, this virtual table will be literally overridden with its respective overridden/polymorphic function slot. Each virtual function has a fixed index in the virtual table, no matter how long the inheritance hierarchy is.
- If `derived` class introduce a new virtual function not present in the base class, the virtual table will be grown by a slot and the address of the function is placed within that slot.
- If you want to summarize virtual keyword functionality in two words then its `indirect calling` of a polymorphic function.

###### FAQ

**Q**. How do we know at runtime that pointer `protocol` will execute a right function(of the object pointed to)?

**A**. In general, we don't know the exact type of the object `protocol` addresses at each invocation of `authenticate()`. We do know, however, that through `protocol` we can access the virtual table associated with the object's class. And the offset of `_vptr` is fixed throughout the inheritance hierarchy. Again we also that index of function `authenticate()` in a virtual table is fixed throughout the inheritance hierarchy.
This way right `authenticate()` function execution will be guaranteed. 

**Q**. Where & how this code augment by the compiler?

**A**. The code necessary to fill virtual table slot is generated by the compiler in constructors right before user-written code.

**Q**. What if there is a `derived` class having more than one base class?

**A**. We will discuss this scenario in the subsequent topic.

### How pure virtual function works
- When you declare any function as pure virtual, the compiler automatically fills the slot of that pure virtual function with dummy function or so-called place holder `pure_virtual_called()` library instance. And run time exception is placed if somehow this place holder will be called.  
- Rest of calling & virtual table slot mechanism would be the same as a normal virtual function.
### Virtual function support under multiple inheritances.
- Now with multiple inheritance things will get a little bit tricky. To understand this behaviour let us take another simplified example as follow :

```
class base1{
  public: 
    int base1_var;

    virtual void base1_func(){}
            
    virtual base1* print(){
      cout<<"print base1\n";
      return this;
    }
};

class base2{
  public: 
    int base2_var;

    virtual void base2_func(){}

    virtual base2* print(){
      cout<<"print base2\n";
      return this;
    }
};

class derived : public base1, public  base2{
  public:
    int derived_var;

    virtual derived* print(){
      cout<<"print derived\n";
      return this;
    }
};
```
- Here we have `derived` class with two base classes. In this case, when we declare an object of the `derived` class, two virtual tables will be created in the `derived` class object. One for `base1` & other for `base2`, which are overridden by `derived` polymorphic functions. 
```
|                        |          
|------------------------| <------ derived object memory layout
|  base1::base1_var      |          
|------------------------|          |----------|----------------------|
|  base1::_vptr_base1    |----------|          |   type_info derived  |
|------------------------|                     |----------------------|
|  base2::base2_var      |                     |   base1::base1_func  |
|------------------------|                     |----------------------|
|  base2::_vptr_base2    |----------|          |    derived:::print   |
|------------------------|          |          |----------------------|
|  derived::derived_var  |          |
|------------------------|          |----------|----------------------|
|                        |                     |   type_info derived  |
|                        |                     |----------------------|
|                        |                     |   base2::base2_func  |
|                        |                     |----------------------|
                                               |    derived::print    |
                                               |----------------------|
```
- To understand that, first let's assign a `base2` pointer the address of a `derived` class object allocated on the heap:

```
base2 *pb = new derived;
```
- The address of the new `derived` object must be adjusted to address its `base2` subobject. The code to accomplish this is generated at compile time:
```
// transformation to support second base class
derived *temp = new derived;
base2 *pb = temp ? temp + sizeof( base1 ) : 0;
```
- Visualizing memory object of above adjustment.
```
        |                        |          
        |------------------------| <------ derived object memory layout
        |  base1::base1_var      |          
        |------------------------|          |----------|----------------------|
        |  base1::_vptr_base1    |----------|          |   type_info derived  |
pb ---> |------------------------|                     |----------------------|
        |  base2::base2_var      |                     |   base1::base1_func  |
        |------------------------|                     |----------------------|
        |  base2::_vptr_base2    |----------|          |    derived:::print   |
        |------------------------|          |          |----------------------|
        |  derived::derived_var  |          |
        |------------------------|          |----------|----------------------|
        |                        |                     |   type_info derived  |
        |                        |                     |----------------------|
        |                        |                     |   base2::base2_func  |
        |                        |                     |----------------------|
                                                       |    derived:::print   |
                                                       |----------------------|
```
- Without this adjustment, any nonpolymorphic use of the pointer would fail, such as
```
pb->base2_var;
```
- The call to polymorphic function `print()` 
```
pb->print();
```
would probably be transformed into
```
( * pb->_vptr_base2[ 2 ])( pb ); 
```

NOTE: There is one extraordinary case of virtual destructor which we will discuss in depth in another article.


### Reference 
- http://www.avabodh.com/cxxin/virtualbase.html
- Book: Inside C++ Object Model By Lippman
- 
