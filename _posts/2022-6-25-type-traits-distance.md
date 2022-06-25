---
layout: post
title: C++ Type Traits and an interview question
date: 2022-6-22
---

In a C++ interview I was asked to implement [std::distance](https://en.cppreference.com/w/cpp/iterator/distance) function as optimally as possible.

Optimally meant:

- Time complexity for raw points should be O(1)
- Time complexity for contiguous data structures like `std::vector` should be O(1)
- Time complexity for non-contiguous data structures like `std::list` should be O(n)

I'll be using this to test my code:

```cpp
int main()
{
    std::vector<int> vec = { 1,2,3,4,5,6,7,8,9 };

    std::list<int> list = { 1,2,3,4,5,6,7,8,9 };

    int b[9] = { 1,2,3,4,5,6,7,8,9 };

    std::cout << my_distance(&b[0], &b[8]) << std::endl <<
        my_distance(std::next(vec.begin(), 0), std::next(vec.begin(), 8)) << std::endl <<
        my_distance(std::next(list.begin(), 0), std::next(list.begin(), 8)) << std::endl;
}
```

My first attempt was to write templates for all raw pointers, vectors, lists.  
By restricting the allowed types in function parameters I should be able to achieve that:

```cpp
// should be used for vectors
template <typename T>
int my_distance(typename std::vector<T>::iterator a, typename std::vector<T>::iterator b) {
    return b - a;
}

// should be used for raw pointers
template <typename T>
int my_distance(T* a, T* b) {
    return b - a;
}

// should be used for lists
template <typename T>
int my_distance(typename std::list<T>::iterator a, typename std::list<T>::iterator b) {
    int ctr = 0;
    for (typename std::list<T>::iterator i = a; i != b; i = i.next(), ctr++);
    return ctr;
}
```

But, I faced compilation errors

![Error]({{ site.baseurl }}/images/2022-6-25-type-traits-distance/error.png)  

So the compiler couldn't deduce `T` from any of the 3 specified templates for both usages of `my_distance`. That was weird but I can understand that it may be related to the internal implementation of iterators in the MSVC STL. [This SO thread](https://stackoverflow.com/questions/18138075/vector-iterator-parameter-as-template-c) vaguely points that what I'm doing should work. Looking at the Output window for more verbose logs, I found this corresponding to the `my_distance` call for `vector`:

```cpp
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(38,9): error C2672: 'my_distance': no matching overloaded function found
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(38,9): error C2783: 'int my_distance(std::list<T,std::allocator<_Ty>>::iterator,std::list<T,std::allocator<_Ty>>::iterator)': could not deduce template argument for 'T'
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(23): message : see declaration of 'my_distance'
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(38,73): error C2784: 'int my_distance(T *,T *)': could not deduce template argument for 'T *' from '_InIt'
1>        with
1>        [
1>            _InIt=std::_Vector_iterator<std::_Vector_val<std::_Simple_types<int>>>
1>        ]
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(17): message : see declaration of 'my_distance'
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(38,9): error C2783: 'int my_distance(std::vector<T,std::allocator<_Ty>>::iterator,std::vector<T,std::allocator<_Ty>>::iterator)': could not deduce template argument for 'T'
1>D:\my-projs\rand-cpp\type_traits\TypeTraits\ConsoleApplication1\ConsoleApplication1.cpp(11): message : see declaration of 'my_distance'
```

A line of interest would be

```cpp
1>            _InIt=std::_Vector_iterator<std::_Vector_val<std::_Simple_types<int>>>
```

This seems to indicate that out `int` is being wrapped into 2 more containers before reaching iterator. That may be somewhat related to the issue but I'd like help here :).
This code is available at: <https://github.com/sin3point14/TypeTraitsDistance/tree/30b8b716db0fbd4fd54748ce1cc70e19a323f43d>

Going ahead directly to the generalised solution, we need to somehow "detect" functionalities in our template parameters and conditionally give them a function defintion. For that we need:

### Type Traits

This C++ metaprogramming technique allows us to instantiate templates using a feature called SFINAE(Substitution Failure Is Not An Error). It gives us some features similar to reflection in languages like go, java, C# etc but a very major difference being that SFINAE is compile time and reflection is runtime.  

SFINAE means that for multiple template defintions for a function/class, gf the compiler detects an error when it substitutes our type, it won't be considered an error until it can't find ANY legal template instantiation for that type. Even in our previous case the compiler tries to instantiate all 3 template definitions but failed in all of them. [Refer this](https://riptutorial.com/cplusplus/example/3780/what-is-sfinae) for a nice primer on SFINAE.  

[`type_traits`](https://en.cppreference.com/w/cpp/header/type_traits) header file gives access to templates that allow us to check for type properties like `is_integral`, `is_array`, `is_const`. These checks can then be combined with `enable_if` to conditionally remove a template specialization.

Back to our problem, we need to find a way to detect a way to differentiate between contiguous and other iterators. For that I oppened `vector` header file and looked at the iterator class defined in it called `_Vector_iterator`.

![Error]({{ site.baseurl }}/images/2022-6-25-type-traits-distance/vector_iter.png)  

`iterator_concept` and `iterator_category` seem interesting. Concepts are a C++20 feature and I intend to solve this problem in C++17, so I'll investigate `iterator_category`. I seeked to definition of `random_access_iterator_tag`. Random access iterators seem to refer to whether an iterator can access an element in constant time. So such iterators _should_ also have `-` operator defined. Now The goal is to detect this `using` statement in an iterator class. After lots of trial and error and referring usages of `enable_if` on SO and cppreference I reached this solution:

```cpp
// should be used for vectors
template <typename T, std::enable_if_t<std::is_same<typename T::iterator_category, std::random_access_iterator_tag>::value, bool> = true>
typename int my_distance(T a, T b) {
    std::cout << "std::is_same\n";
    return b - a;
}

// should be used for raw pointers
template <typename T>
int my_distance(T* a, T* b) {
    std::cout << "pointer\n";
    return b - a;
}

// should be used for lists
template <typename T, std::enable_if_t<!std::is_same<typename T::iterator_category, std::random_access_iterator_tag>::value, bool> = true>
typename int my_distance(T a, T b) {
    std::cout << "!std::is_same\n";
    int ctr = 0;
    for (T i = a; i != b; i++, ctr++);
    return ctr;
}
```

I'll try to break the first template down, the second only is only the negation of the previous one.

```cpp
template <
    // our supplied type
    typename T
    // expands to enable_if<>::type, which resolves to the type in second arg
    // if first arg is true else, it fails the template substitution
    , std::enable_if_t<
        // this returns true if they match else false
        std::is_same<
            typename T::iterator_category
            , std::random_access_iterator_tag
        // the true/false value is stored in ::value
        >::value
        // this can be any type(void by default), we need this argument  
        // because  template arguments are types and std::enable_if_t 
        // is being supplied as a template argument. So std::enable_if_t
        // type in our case. We can't return void since void can't be
        // assigned a default value(see next)
        , bool
    // a default value for the returned bool type else we'll need to supply
    // a second type argument whenever we use this template, while the
    // second argument is just a placeholder used for detection
    > = true
>
```

The default argument may seem weird but it is due to function template equaivalence rule for default tepmplate arguments as mentioned in the `Notes` section [of the cppreference page to std::enable_if](https://en.cppreference.com/w/cpp/types/enable_if).

According to our requirement, default argument is necessary. We don't want to specify 2 types and the second argument is just a placeholder for our template magic anyway.  
Function template equivalence rules assign "signatures" to template declarations. Like functions, we cannot have two templates with the same signature. The rule says that default arguments are not accounted when making equivalence comparisons. So we cannot structure our template magic like:

```cpp
template <typename T, typename = std::enable_if_t<std::is_same<typename T::iterator_category, std::random_access_iterator_tag>::value>>

...

template <typename T, typename = std::enable_if_t<!std::is_same<typename T::iterator_category, std::random_access_iterator_tag>::value>>
```

Both of these templates would be equivalent and the latter would be treated as redefinition.

Find the final code at: [TypeTraitsDistance](https://github.com/sin3point14/TypeTraitsDistance)

PS: While writing this blog I found that [Iterator Tags](https://en.cppreference.com/w/cpp/iterator/iterator_tags) and [Random Access Iterator Tag](https://cplusplus.com/reference/iterator/RandomAccessIterator/) are part of the cpp standard and have proper documentation.
