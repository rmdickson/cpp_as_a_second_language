---
layout: episode
title: "Operator overloading"
teaching: 10
exercises: 10
questions:
- "Can we define operators for our classes?"
- "Where can you define operators?"
- "What operators can we define?"
- "Does the order matter?"
objectives:
- ""
keypoints:
- ""
---

It would be nice to be able to add two of our `Vectors` together into one vector. We can use operator overloading to create a new `+` operator that will add two of our `Vectors` together.

Operator overloading works similar to how function overloading works.

To create a new operator function you need to use the `operator` keyword followed by the operator you wish to overload, for example `operator+`. The rest of the syntax for creating an operator is similar to how a function is created.

Let's add a new `+` operator to our `Vector` class. 

~~~
$ cp destructor.cpp operator.cpp
$ nano operator.cpp
~~~
<div class="gitfile" markdown="1">
~~~
#include <iostream>

class Vector{
public:
  int size;
  int* data;
  
  Vector operator+(Vector& rhs){
    Vector temp(size+rhs.size);
    int tempIndex;
    for(tempIndex=0;tempIndex<size;++tempIndex){
      temp.data[tempIndex]=data[tempIndex];
    }
    int rhsIndex=0;
    for(tempIndex=size;tempIndex<temp.size;++tempIndex){
      temp.data[tempIndex]=rhs.data[rhsIndex];
      ++rhsIndex;
    }
    return temp;
  }
  
  ~Vector(){...}
  Vector(){...}
  Vector(int size_in){...}
  Vector(const Vector& vec_in){...}
  void display(){...}
  void display(int num){...}
};

int main(){
  Vector a(10);
  a.data[9]=15;
  Vector b=a;
  b.data[8]=10;
  a.display();
  b.display();
  Vector c=a+b;
  c.display();
}
~~~
[operator.cpp](https://github.com/acenet-arc/cpp_as_a_second_language/blob/master/lesson_src/operator%2B.cpp)
</div>

This new `+` operator returns a `Vector` object and has a `Vector` object as an argument. In a statement like `Vector c=a+b;` where `a` and `b` are objects of type `Vector` you can think of it being like `Vector d=a.operator+(b);` and in fact you can actually use it like that if you wish. In this way `operator+` acts exactly like any other class member function, except that it can also be invoked directly like `a+b` rather that using the `.` and the full function name.

Lets try out the new operator
~~~
$ g++ operator.cpp -o operator
$ ./operator
~~~
{: .bash}
~~~
Vector: size=10, contents=(0,0,0,0,0,0,0,0,0,15)
Vector: size=10, contents=(0,0,0,0,0,0,0,0,10,15)
Vector: size=20, contents=(0,0,0,0,0,0,0,0,0,15,0,0,0,0,0,0,0,0,10,15)
~~~
{: .output}
As we can see it added the two vectors together, with `a` preceding `b`.

Some operators can be declared as class member functions and some must be declared outside the class and some can be declared either way depending on how we want to use them. There are quite a few different operators; combining those with the different constructors, different data types, and how the operators work together can lead to quite a few different ways to create operators. You can really spend a lot of time trying to create operators to cover all the possible ways you might want to use operators with your class. **Be careful not to spend too much time writing code you might not need**.

For a list of operators and where each can be declared see:
[https://www.cplusplus.com/doc/tutorial/templates/](https://www.cplusplus.com/doc/tutorial/templates/)

## operator=
We have so far used the `+` operator we defined like so, `Vector a,b; Vector c=a+b;`, however if we wanted to use it this way `Vector a,b,c; c=a+b;` we would have an issue. So far we have been relying on our copy constructor which gets called when we do `Vector c=`some other vector, however when we do `c=a+b` it will invoke the compiler created assignment operator (`operator=`) which will just copy our pointers over. Then when the destructors gets called for the two `Vector` objects with the same pointer it will cause a double free error. This exactly the same thing that would happen if we didn't create our copy constructor for exactly the same reason. Lets add an assignment operator now.

<div class="gitfile" markdown="1">
~~~
#include <iostream>

class Vector{
public:
  int size;
  int* data;
  
  void operator=(Vector rhs){
    if(&rhs!=this){
      
      delete[] data;
      data=new int[rhs.size];
      size=rhs.size;
      for(int i=0;i<rhs.size;++i){
        data[i]=rhs.data[i];
      }
    }
  }
  
  Vector operator+(Vector& rhs){...}
  ~Vector(){...}
  Vector(){...}
  Vector(int size_in){...}
  Vector(const Vector& vec_in){...}
  void display(){...}
  void display(int num){...}
};

int main(){
  Vector a(10);
  a.data[9]=15;
  Vector b=a;
  b.data[8]=10;
  a.display();
  b.display();
  Vector c;
  c=a+b;
  c.display();
}
~~~
[operator.cpp](https://github.com/acenet-arc/cpp_as_a_second_language/blob/master/lesson_src/operator%3D.cpp)
</div>
Notice that we check to see if the rhs has the same address as the current object (self assignment `a=a`), if so there is nothing to do. If we didn't do this check we would delete our data and then try to copy the new uninitialized data to itself, which doesn't seem like a good thing to do.


## Rule of Three
The Rule of Three is a general rule that states, if any one of the three following class member functions are defined, then they should all be defined.
* Destructor
* Copy constructor
* Assignment operator

In our case we saw why that is. We allocated memory in our constructor and when we created a new object from a previous object either using the compiler created `=` operator or copy constructor, we ended up with a double free when our destructor was called. The compiler created copy constructor and assignment operator only copied the pointer to the data, they didn't allocate the objects own memory and copy values over from the other object.

> ## Mixing types with operators
> You can add additional overloaded versions of an operator to allow different types to be added together. For example if we wanted to be able to add an `Vector` object and an `int` we could add the following member function to our `Vector` class:
> ~~~
> Vector operator+(int rhs){
>   Vector temp(size+1);
>   
>   //copy data into temp with a for loop
>   
>   temp.data[size]=rhs;
>   return temp;
> }
> ~~~
> 
> Then we could add them like so:
> ~~~
> Vector a;
> Vector b=a+21;
> ~~~
> However, the order is important here we could not do the following:
> ~~~
> Vector a;
> Vector b=21+a;
> ~~~
> as it would try to call the `operator+` for an integer with a RHS of type `Vector` and that operator doesn't exist. We could add that in a slightly different way with a stand alone function.
> ~~~
> Vector operator+(int lhs,Vector rhs){
>   Vector temp(rhs.size+1);
>   temp.data[0]=lhs;
>   
>   //copy data into temp with a for loop starting a 1
>   
>   return temp;
> }
> ~~~
{: .callout}
