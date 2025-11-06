[![Documented with Setinstone.io](https://img.shields.io/badge/⛰️Documented%20with-Setinstone.io-success?logo=book&logoColor=white)](https://calendly.com/set-in-stone-thomas-benoit/setinstone-demo)
[![CI](https://github.com/thomgit9/pfr/actions/workflows/ci.yml/badge.svg?branch=develop)](https://github.com/thomgit9/pfr/actions/workflows/ci.yml)
[![License: BSL-1.0](https://img.shields.io/badge/License-BSL--1.0-blue.svg)](LICENSE_1_0.txt)
![C++](https://img.shields.io/badge/C%2B%2B-14%2B-blue)
![Header-only](https://img.shields.io/badge/library-header--only-lightgrey)

# [Boost.PFR](https://boost.org/libs/pfr)

## Presentation

**Boost.PFR (Precise Function Reflection)** is a C++14 header-only library providing *std::tuple-like* functionalities for user-defined types, without requiring macros, boilerplate code, or external reflection metadata.

This makes it possible to access structure fields by index, perform serialization, comparison, or function application over struct members automatically — achieving reflection-like capabilities at compile-time.

The project `thomgit9/pfr` is a mirror implementation of `boostorg/pfr`, released under the [Boost Software License 1.0 (BSL-1.0)](https://boost.org/LICENSE_1_0.txt), and is completely independent of other Boost libraries.

Repository: [https://github.com/boostorg/pfr](https://github.com/boostorg/pfr)

---

## Installation

Boost.PFR is a **header-only** library.

To integrate it into your project:

1. Copy the content of the `include/` directory into your project’s include path.
2. Include the main header:

```cpp
#include "boost/pfr.hpp"
```

3. No additional configuration or macros are needed.  
   Works with standard-compliant C++14 (and newer) compilers.

For a namespace-free version (without `boost::` prefix), see [PFR Non-Boost](https://github.com/apolukhin/pfr_non_boost).

---

## Usage

Below are a few minimal motivating examples demonstrating reflection-like access to structure members.

### Example 0 - Field Access and Serialization

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "boost/pfr.hpp"

struct some_person {
  std::string name;
  unsigned birth_year;
};

int main(int argc, const char* argv[]) {
  some_person val{"Edgar Allan Poe", 1809};

  std::cout << boost::pfr::get<0>(val)
            << " was born in " << boost::pfr::get<1>(val);

  if (argc > 1) {
    std::ofstream ofs(argv[1]);
    ofs << boost::pfr::io(val);
  }
}
```

Output:
```
Edgar Allan Poe was born in 1809
```

[Run this sample](https://godbolt.org/z/PfYsWKb7v)

---

### Example 1 - Tuple-like Introspection

```cpp
#include <iostream>
#include "boost/pfr.hpp"

struct my_struct {
    int i;
    char c;
    double d;
};

int main() {
    my_struct s{100, 'H', 3.141593};
    std::cout << "my_struct has "
              << boost::pfr::tuple_size<my_struct>::value
              << " fields: " << boost::pfr::io(s) << "\n";
}
```

Output:
```
my_struct has 3 fields: {100, H, 3.14159}
```

---

### Example 2 - String and Integer Structs

```cpp
#include <iostream>
#include "boost/pfr.hpp"

struct my_struct {
    std::string s;
    int i;
};

int main() {
    my_struct s{{"Das ist fantastisch!"}, 100};
    std::cout << "my_struct has "
              << boost::pfr::tuple_size<my_struct>::value
              << " fields: " << boost::pfr::io(s) << "\n";
}
```

Output:
```
my_struct has 2 fields: {"Das ist fantastisch!", 100}
```

---

### Example 3 - Integration with Boost.Spirit.X3

```cpp
#include <iostream>
#include <string>
#include <boost/config/warning_disable.hpp>
#include <boost/spirit/home/x3.hpp>
#include <boost/fusion/include/adapt_boost_pfr.hpp>
#include "boost/pfr/io.hpp"

namespace x3 = boost::spirit::x3;

struct ast_employee {
    int age;
    std::string forename;
    std::string surname;
    double salary;
};

auto const quoted_string = x3::lexeme['"' >> +(x3::ascii::char_ - '"') >> '"'];

x3::rule<class employee, ast_employee> const employee = "employee";
auto const employee_def =
    x3::lit("employee") >> '{'
    >> x3::int_ >> ',' >> quoted_string >> ','
    >> quoted_string >> ',' >> x3::double_ >> '}';
BOOST_SPIRIT_DEFINE(employee);

int main() {
    std::string str = R"(employee{34, "Chip", "Douglas", 2500.00})";
    ast_employee emp;
    x3::phrase_parse(str.begin(), str.end(), employee, x3::ascii::space, emp);
    std::cout << boost::pfr::io(emp) << std::endl;
}
```

Output:
```
(34 Chip Douglas 2500)
```

---

## Functions and Classes Overview

| Name | File | Description | Inputs | Outputs |
|------|------|-------------|---------|----------|
| `boost::pfr::get<I>(T&&)` | `include/boost/pfr/core.hpp` | Returns the I-th field of an aggregate type. | Aggregate object `T`, index `I` | Reference or value of the field |
| `boost::pfr::for_each_field(T&&, F&&)` | `include/boost/pfr/for_each_field.hpp` | Iterates over all fields of a struct, applying a provided function. | Aggregate object `T`, callable `F` | None (executes function `F`) |
| `boost::pfr::io(T&&)` | `include/boost/pfr/io.hpp` | Converts structure values into human-readable stream representation. | Streamable aggregate object `T` | Serialized string or stream output |
| `boost::pfr::tuple_size<T>` | `include/boost/pfr/tuple_size.hpp` | Provides compile-time field count of a struct (`constexpr std::size_t`). | Type `T` | Field count as `std::integral_constant` |
| `boost::pfr::structure_to_tuple(T&&)` | `include/boost/pfr/core.hpp` | Returns all fields packed in a `std::tuple`. | Aggregate object `T` | `std::tuple` of the struct’s fields |
| `boost::pfr::make_flat_tuple_of_references(T&&)` | `include/boost/pfr/detail/make_flat_tuple_of_references.hpp` | Creates a tuple of references to flattened members, handling nested structs. | Aggregate object `T` | Tuple of references to all fields |

---

## Requirements and Limitations

See full developer documentation on [Boost.org](https://www.boost.org/doc/libs/develop/doc/html/boost_pfr.html).  
Compatible with **C++14**, **C++17**, and **C++20** standard compilers (GCC, Clang, MSVC).

---

## Test Results

| Branch | CI Build | Test Coverage | Details |
|--------|-----------|---------------|----------|
| develop | [![CI](https://github.com/boostorg/pfr/actions/workflows/ci.yml/badge.svg?branch=develop)](https://github.com/boostorg/pfr/actions/workflows/ci.yml) [![Build status](https://ci.appveyor.com/api/projects/status/0mavmnkdmltcdmqa/branch/develop?svg=true)](https://ci.appveyor.com/project/apolukhin/pfr/branch/develop) | [![Coverage Status](https://coveralls.io/repos/github/apolukhin/magic_get/badge.png?branch=develop)](https://coveralls.io/github/apolukhin/magic_get?branch=develop) | [details...](https://regression.boost.io/develop/developer/pfr.html) |
| master | [![CI](https://github.com/boostorg/pfr/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/boostorg/pfr/actions/workflows/ci.yml) [![Build status](https://ci.appveyor.com/api/projects/status/0mavmnkdmltcdmqa/branch/master?svg=true)](https://ci.appveyor.com/project/apolukhin/pfr/branch/master) | [![Coverage Status](https://coveralls.io/repos/github/apolukhin/magic_get/badge.png?branch=master)](https://coveralls.io/github/apolukhin/magic_get?branch=master) | [details...](https://regression.boost.io/master/developer/pfr.html) |

---

## License

Distributed under the [Boost Software License, Version 1.0](https://boost.org/LICENSE_1_0.txt).

---
