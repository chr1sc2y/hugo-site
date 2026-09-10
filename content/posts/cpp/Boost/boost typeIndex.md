---
title: "Exploring boost::typeindex"
date: 2020-07-31T20:31:06+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on Exploring boost::typeindex, preserving the examples and context of the original article."
---
# Exploring boost::typeindex

> Originally published in Chinese on 2020-07-31; this English edition preserves the original scope and technical context.

## boost::typeIndex's Related Exploration

Effective Modern C++ Item 4: Know how to view deduced types. mentions the use of `Boost::typeindex`, but does not discuss its implementation.

## 1. [typeid](https://en.cppreference.com/w/cpp/language/typeid) Operator

typeid is a operator in C++ used to obtain information about a type. It is often used where a known dynamic type of polymorphic objects is needed, or to identify static types.

We can write a simple demo to get information related to object types, requiring the inclusion of the `tepyinfo` header file:
```c++
#include <iostream>
#include <typeinfo>

using namespace std;

class Foo {};

int main()
{
    cout << "1: " << typeid(1).name() << endl;
cout << "int: " << typeid(int).name() << endl; // Similar to the sizeof operator, typeid can also directly operate on data types (like int).

    cout << "typeid: " << typeid(typeid(int)).name() << endl;
    cout << "typeid: " << typeid(const type_info &).name() << endl;

    const Foo *foo = new Foo();
    cout << "foo: " << typeid(foo).name() << endl;
    cout << "*foo: " << typeid(*foo).name() << endl;
    cout << "Foo: " << typeid(Foo).name() << endl;
}

```
```bash
[joelzychen@DevCloud ~/typeid]$ g++ -std=c++11 -otypeid_test typeid_test.cpp
[joelzychen@DevCloud ~/typeid]$ ./typeid_test
1: i
int: i
typeid: N10__cxxabiv123__fundamental_type_infoE
typeid: St9type_info
foo: PK3Foo
*foo: 3Foo
Foo: 3Foo
```

The `std::type_info::name()` function returns strings in GCC and Clang implementations where 'i' represents int, 'P' represents pointer, and 'K' represents const; the numbers following these indicate the number of characters that follow. We can observe a more intuitive output by compiling and running this code with Microsoft's MSVC compiler.

```bash
1: int
int: int
typeid: class type_info
typeid: class type_info
foo: class Foo const *
*foo: class Foo
Foo: class Foo
```
One can see that most results align with our expectations, but the invocation of `typeid(const type_info &).name()` returns a result that is not as expected, `const type_info &`, where the `const` and `reference` characteristics are not preserved. Let's consider a simpler example:
```c++
#include <iostream>
#include <typeinfo>

using namespace std;

template<typename T>
static void PrintType(const T &t)
{
    std::cout << "T: " << typeid(T).name() << std::endl;
    std::cout << "t: " << typeid(t).name() << std::endl;
}

int main()
{
    const int *p_i;
    PrintType(p_i);
}

```
```bash
[joelzychen@DevCloud ~/typeid]$ g++ -std=c++11 -otypeid_test typeid_test.cpp
[joelzychen@DevCloud ~/typeid]$ ./typeid_test
T: PKi
t: PKi
```
`PrintType` template receives `T` as `PKi` (const int\*) type, similar to the previous examples, the `const reference` characteristic of `t` is not preserved.

## 2. Use `boost::typeindex::type_id_with_cvr` instead of `typeid`

The `boost` library provides a function similar to the `typeid` operator, `boost::typeindex::type_id_with_cvr`, which can be used to obtain the object type. We can utilize this template function to obtain a more precise type:
```c++
#include <iostream>
#include <typeinfo>
#include <boost/type_index.hpp>

using namespace std;

template<typename T>
static void PrintType(const T &t)
{
    cout << "T: " << boost::typeindex::type_id_with_cvr<T>().pretty_name() << endl;
    cout << "t: " << boost::typeindex::type_id_with_cvr<decltype(t)>().pretty_name() << endl;
    cout << "typeid: " << boost::typeindex::type_id_with_cvr<decltype(typeid(int))>().pretty_name() << endl;
}

int main()
{
    const int *p_i{ nullptr };
    PrintType(p_i);
}

```
```bash
[joelzychen@DevCloud ~/typeid]$ g++ -std=c++11 -otypeid_test typeid_test.cpp -I/usr/include/boost-1.73.0/gcc-head/include
[joelzychen@DevCloud ~/typeid]$ ./typeid_test
T: int const*
t: int const* const&
typeid: std::type_info const&
```
`typeid` returns the value type as `std::type_info const&`, whereas `boost::typeindex::type_id_with_cvr` retains its `const` and `reference` traits through the `pretty_name()` function, which outputs the result as a string. Unlike `typeid`, the `type_id_with_cvr` function can only take template parameters or types derived with `decltype`, and cannot accept a variable.

## type_id_with_cvr() Implementation

`type_id_with_cvr` this template function is defined in `boost/type_index.hpp`. It actually calls the static template function `type_id_with_cvr` of the `stl_type_index` class:
```c++
// boost/type_index.hpp
namespace boost { namespace typeindex {
template <class T>
inline type_index type_id_with_cvr() BOOST_NOEXCEPT {
    return type_index::type_id_with_cvr<T>();
}
}

// boost/type_index/stl_type_index.hpp
namespace boost {
class stl_type_index : public type_index_facade<stl_type_index, std::type_info> // omitted BOOST_NO_STD_TYPEINFO macro judgment
{
public:
   typedef std::type_info type_info_t; // omitted judgment of BOOST_NO_STD_TYPEINFO macro
private:
    const type_info_t* data_;
public:
    inline stl_type_index(const type_info_t& data) BOOST_NOEXCEPT
       : data_(&data) // Utilize the `typeid` operator to return a `const type_info_t&` object for construction.
    {}

    template <class T>
    inline static stl_type_index type_id_with_cvr() BOOST_NOEXCEPT;
}
}
```
`boost::typeindex::type_id_with_cvr` function invokes the `typeid` operator with its second template parameter `detail::cvr_saver<T>` and constructs an `stl_type_index` object using the returned `const type_info_t&` object;

`detail::cvr_saver` is a template class that contains information about its template parameter `<class T>`, and it can be used to obtain the `type_info` via `typeid`.
```c++
// boost/type_index/stl_type_index.hpp
namespace boost {
template <class T>
inline stl_type_index stl_type_index::type_id_with_cvr() BOOST_NOEXCEPT {
    typedef BOOST_DEDUCED_TYPENAME boost::conditional<
        boost::is_reference<T>::value ||  boost::is_const<T>::value || boost::is_volatile<T>::value,
        detail::cvr_saver<T>,
        T
>::type type; // Equivalent to using type = boost::conditional<...>

    return typeid(type);
}
}

// boost/type_traits/conditional.hpp
namespace detail {
    template <class T> class cvr_saver{};
}

namespace boost {
template <bool b, class T, class U> struct conditional { typedef T type; };
}

```
## 4 class stl_type_facade

`class type_index_facade` is the base class of `class stl_type_index`, and its source code is in `type_index_facade.hpp`, using the facade design pattern.
```c++
// boost/type_index/stl_type_index.hpp
// Will derive class Derived as a template parameter
template <class Derived, class TypeInfo>
class type_index_facade {
public:
    typedef TypeInfo                                type_info_t;

// Call the raw_name() of the subclass using a non-virtual function via a template to achieve static polymorphism.
    inline const char* name() const BOOST_NOEXCEPT {
        return derived().raw_name();
    }

python
# Return a human-readable string, invoking the sub-function `name()`.


Original ASCII Diagram:

+-------------------+
|       Human-Read   |
|       able String   |
+-------------------+

    inline std::string pretty_name() const {
        return derived().name();
    }

// Compare the `raw_name()` of derived classes classes, requiring the derived classes classes to implement the `raw_name()` function.
    inline bool equal(const Derived& rhs) const BOOST_NOEXCEPT {
        const char* const left = derived().raw_name();
        const char* const right = rhs.raw_name();
        return left == right || !std::strcmp(left, right);
    }

// Compare the `raw_name()` of derived classes classes, requiring the derived classes classes to implement the `raw_name()` function.
    inline bool before(const Derived& rhs) const BOOST_NOEXCEPT {
        const char* const left = derived().raw_name();
        const char* const right = rhs.raw_name();
        return left != right && std::strcmp(left, right) < 0;
    }

// GET THE HASH VALUE OF A TYPE BY DEFAULT HASHING raw_name() OF DERIVED CLASSES
    inline std::size_t hash_code() const BOOST_NOEXCEPT {
        const char* const name_raw = derived().raw_name();
        return boost::hash_range(name_raw, name_raw + std::strlen(name_raw));
    }
}
```
Furthermore, the base class `class type_index_facade` overloads various comparison operators, output stream operators, and the class hash value algorithm.
```c++
// boost/type_index/stl_type_index.hpp
// Omitted other types of comparison operators
template <class Derived, class TypeInfo>
inline bool operator == (const TypeInfo& lhs, const type_index_facade<Derived, TypeInfo>& rhs) BOOST_NOEXCEPT {
   return Derived(lhs) == rhs;	// needs a derived class implementation to implement a constructoring function taking const TypeInfo&
}

// Overload output stream operator
template <class CharT, class TriatT, class Derived, class TypeInfo>
inline std::basic_ostream<CharT, TriatT>& operator<<(
    std::basic_ostream<CharT, TriatT>& ostr,
    const type_index_facade<Derived, TypeInfo>& ind)
{
    ostr << static_cast<Derived const&>(ind).pretty_name();
    return ostr;
}

// Class hash value algorithm
template <class Derived, class TypeInfo>
inline std::size_t hash_value(const type_index_facade<Derived, TypeInfo>& lhs) BOOST_NOEXCEPT {
    return static_cast<Derived const&>(lhs).hash_code();
}
```
If you want to perform all operations of the base class `class type_index_facade`, you must also derive a subclass and implement the following two functions:

1. `raw_name()`，base class many functions depend on the derived class's function
2. `Derived(const TypeInfo&)`，a constructor that takes `const TypeInfo&` as a parameter, used for comparison with `TypeInfo` object.

## 5 class type_type_index

`stl_type_index` is a derived class of `stl_type_facade`. Its private member variable `type_info_t` is defined through `typedef`. `BOOST_NO_STD_TYPEINFO` means that the `std` namespace does not have a `type_info` type, in which case the global namespace `type_info` is defined as `type_info_t`.
```c++
public:
#ifdef BOOST_NO_STD_TYPEINFO
    typedef type_info type_info_t;
#else
    typedef std::type_info type_info_t;
#endif

private:
    const type_info_t* data_;
```
For clarity, the definition of the `BOOST_NO_STD_TYPEINFO` macro is omitted temporarily; the declaration of the derived class `stl_type_index` is as follows:
```c++
class stl_type_index : public type_index_facade<stl_type_index, std::type_info>
{
public:
    typedef std::type_info type_info_t;

private:
const type_info_t* data_; // Unique private member const type_info_t*

public:
    inline stl_type_index() BOOST_NOEXCEPT
        : data_(&typeid(void))
    {}

    inline stl_type_index(const type_info_t& data) BOOST_NOEXCEPT
:data_(data) // Constructor taking const TypeInfo& as a parameter, which is used by the comparison operator and type_id_with_cvr() function. This constructor is also relied upon by these functions.
    {}

`inline const type_info& type_info() const BOOST_NOEXCEPT; // Get private member data`

   inline const char* raw_name() const BOOST_NOEXCEPT; // raw_name() function
    inline const char*  name() const BOOST_NOEXCEPT;
    inline std::string  pretty_name() const;

    inline std::size_t  hash_code() const BOOST_NOEXCEPT;
    inline bool         equal(const stl_type_index& rhs) const BOOST_NOEXCEPT;
    inline bool         before(const stl_type_index& rhs) const BOOST_NOEXCEPT;

    template <class T>
    inline static stl_type_index type_id() BOOST_NOEXCEPT;

    template <class T>
    inline static stl_type_index type_id_with_cvr() BOOST_NOEXCEPT;

    template <class T>
    inline static stl_type_index type_id_runtime(const T& value) BOOST_NOEXCEPT;
};

```
class stl_type_index {
public:
    template<typename T>
    struct equal {
        static bool get(const T& lhs, const T& rhs) {
            return lhs == rhs;
        }
    };

    template<typename T>
    struct before {
        static T get(const T& lhs, const T& rhs) {
            return lhs < rhs ? lhs : rhs;
        }
    };

    template<typename T>
    struct hash_code {
        static size_t get(const T& value) {
            return raw_name(value);
        }
    };

private:
    template<typename T>
    static std::string raw_name(const T& value) {
        // Implementation details
    }
};


For these operations, the `equal`, `before`, and `hash_code` objects are obtained through `raw_name()`. `raw_name()` is a private method that retrieves the specific name of the type.
```c++
inline std::size_t stl_type_index::hash_code() const BOOST_NOEXCEPT {
#ifdef BOOST_TYPE_INDEX_STD_TYPE_INDEX_HAS_HASH_CODE
    return data_->hash_code();
#else
    return boost::hash_range(raw_name(), raw_name() + std::strlen(raw_name()));
#endif
}

inline bool stl_type_index::equal(const stl_type_index& rhs) const BOOST_NOEXCEPT {
#ifdef BOOST_TYPE_INDEX_CLASSINFO_COMPARE_BY_NAMES
    return raw_name() == rhs.raw_name() || !std::strcmp(raw_name(), rhs.raw_name());
#else
    return !!(*data_ == *rhs.data_);
#endif
}

inline bool stl_type_index::before(const stl_type_index& rhs) const BOOST_NOEXCEPT {
#ifdef BOOST_TYPE_INDEX_CLASSINFO_COMPARE_BY_NAMES
    return raw_name() != rhs.raw_name() && std::strcmp(raw_name(), rhs.raw_name()) < 0;
#else
    return !!data_->before(*rhs.data_);
#endif
}
```
`name()` and `raw_name()` both invoke the private member function `name()` of `std::type_info`, namely `std::type_info::name()`.
```c++
inline const char* stl_type_index::raw_name() const BOOST_NOEXCEPT {
#ifdef _MSC_VER // Different compilers implement typeid differently, so the boost library implements both raw_name() and name() functions
    return data_->raw_name();
#else
    return data_->name();
#endif
}

inline const char* stl_type_index::name() const BOOST_NOEXCEPT {
    return data_->name();
}

```
In the `pretty_name()` function prototype, which was called in Part 2, the function prototype is as follows:

python
def pretty_name(obj: Any) -> str:
    pass

```c++
inline std::string stl_type_index::pretty_name() const {
    static const char cvr_saver_name[] = "boost::typeindex::detail::cvr_saver<";
    static BOOST_CONSTEXPR_OR_CONST std::string::size_type cvr_saver_name_len = sizeof(cvr_saver_name) - 1;

// For GCC and Clang, the demangled_name function performs demangling; for MSVC, since std::type_info::name() returns the already demangled string, no demangling is performed in the function.
    const boost::core::scoped_demangled_name demangled_name(data_->name());

// begin is the full string of the svr_saver object of type obtained via `demangled_name.get()`. Printing it at this GDB breakpoint will show its contents.
    // (gdb) p begin
	// $1 = 0x605010 "boost::typeindex::detail::cvr_saver<int const> ()"
    const char* begin = demangled_name.get();
    if (!begin) {
        boost::throw_exception(std::runtime_error("Type name demangling failed"));
    }

    const std::string::size_type len = std::strlen(begin);
    const char* end = begin + len;

// Character string comparison, trim the extra characters from both ends.
    if (len > cvr_saver_name_len) {
        const char* b = std::strstr(begin, cvr_saver_name);
        if (b) {
            b += cvr_saver_name_len;

            // Trim leading spaces
            while (*b == ' ') {         // the string is zero terminated, we won't exceed the buffer size
                ++ b;
            }

            // Skip the closing angle bracket
            const char* e = end - 1;
            while (e > b && *e != '>') {
                -- e;
            }

            // Trim trailing spaces
            while (e > b && *(e - 1) == ' ') {
                -- e;
            }

            if (b < e) {
                // Parsing seems to have succeeded, the type name is not empty
                begin = b;
                end = e;
            }
        }
    }

    return std::string(begin, end);
}

```
Here, `demangled_name` function aside, all the implementation details are understood. It is not difficult to understand that the `stl_type_index` class is a wrapper for the `std::type_info` class. The `type_id_with_cvr` and `pretty_name` functions respectively refine the `typeid` operator and `std::type_info::name()` function.
