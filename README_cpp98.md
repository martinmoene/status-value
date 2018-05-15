status_value cpp98 - a class for status and optional value for C++98 and later
------------------------------------------------------------------------------
[![Language](https://img.shields.io/badge/language-C++-blue.svg)](https://isocpp.org/)  [![Standard](https://img.shields.io/badge/c%2B%2B-98-orange.svg)](https://en.wikipedia.org/wiki/C%2B%2B#Standardization) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![Build Status](https://travis-ci.org/martinmoene/status_value.svg?branch=master)](https://travis-ci.org/martinmoene/status_value)  [![Version](https://badge.fury.io/gh/martinmoene%2Fstatus_value.svg)](https://github.com/martinmoene/status_value/releases) [![download](https://img.shields.io/badge/latest%20version%20%20-download-blue.svg)](https://raw.githubusercontent.com/martinmoene/status_value/master/include/nonstd/status_value_cpp98.hpp)

status_value is a single-file header-only library for objects that represent a status and an optional value. This variant is intended for use with C++98 and later. The library is based on the proposal for status_value [[1](#ref1)].

**Contents**  
- [Example usage](#example-usage)
- [In a nutshell](#in-a-nutshell)
- [License](#license)
- [Dependencies](#dependencies)
- [Installation](#installation)
- [Synopsis](#synopsis)
- [Comparison with like types](#comparison)
- [Reported to work with](#reported-to-work-with)
- [Implementation notes](#implementation-notes)
- [Notes and references](#notes-and-references)
- [Appendix](#appendix)


Example usage
-------------
```C++
#include "status_value_cpp98.hpp"

#include <cstdlib>
#include <iostream>
#include <string>

using namespace nonstd;

status_value< std::string, int >
to_int( char const * const text ) 
{
    char * pos = NULL;
    long value = strtol( text, &pos, 0 );

    if ( pos != text ) return status_value< std::string, int >( "Excellent", value );
    else               return status_value< std::string, int >( std::string("'") + text + "' isn't a number" );
}

int main( int argc, char * argv[] )
{
    char const * const text = argc > 1 ? argv[1] : "42";

    status_value<std::string, int> svi = to_int( text );

    if ( svi ) std::cout << svi.status() << ": '" << text << "' is " << *svi << ", ";
    else       std::cout << "Error: " << svi.status();

    return 0; // VC6
}
```
### Compile and run
```
prompt> g++ -std=c++98 -Wall -I../include/nonstd -o 01-basic_cpp98.exe 01-basic_cpp98.cpp && 01-basic_cpp98.exe 123 && 01-basic_cpp98.exe abc
Excellent: '123' is 123, Error: 'abc' isn't a number
```

In a nutshell
-------------
**status_value** is a single-file header-only library to represent objects that contain a status and an optional value. The library is an implementation of the  proposal for [std:&#58;status_value](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4233.html) [[1](#ref1),[2](#ref2)] for use with C++98 and later.

**Features and properties of status_value** are ease of installation (single header), construction of a status, or a status and a value from a value that is convertible to the underlying type, move-construction (C++11) / copy-construction (C++98) from another status_value of the same type, testing for the presence of a value, operators and method value() for *checked* access to the value and access to the status.  

**Not provided** are copy-construction of a status_value under C++11 (by design), ... .

For more examples, see [[1](#ref1)].


License
-------
*status_value* is distributed under the [Boost Software License](https://github.com/martinmoene/span-lite/blob/master/LICENSE.txt).


Dependencies
------------
*status_value* has no other dependencies than the [C++ standard library](http://en.cppreference.com/w/cpp/header).


Installation
------------
*status_value* is a single-file header-only library. Put `status_value.hpp` directly into the project source tree or somewhere reachable from your project.


Synopsis
--------

**Contents**  
- [Interface of status_value](#interface-of-status_value)  
- [Configuration macros](#configuration-macros)
- [Macros to control alignment](#macros-to-control-alignment)  

### Interface of status_value

| Kind         | Method                                                       | Result |
|--------------|--------------------------------------------------------------|--------|
| Construction | status_value() = delete &ensp; *or* &ensp; private           | disallow default construction |
| C++11        | status_value( status_value && other )                        | move-construct from other |
| C++98        | status_value( status_value const & other )                   | copy-construct from other |
| &nbsp;       | status_value( status_type const & s )                        | copy-construct from status |
| C++11        | status_value( status_type const & s, value_type && v )       | copy-construct from status,<br>move construct from value |
| &nbsp;       | status_value(  status_type const & s, value_type const & v ) | copy-construct from status and value |
| Destruction  | ~status_value()                                              | status, value destroyed if present|
| Observers    | operator bool() const                                        | true if contains value |
| &nbsp;       | bool has_value() const                                       | true if contains value |
| &nbsp;       | status_type const & status() const                           | the status |
| &nbsp;       | value_type const & value() const                             | the value (const ref);<br>see [note 1](#note1) |
| &nbsp;       | value_type & value()                                         | the value (non-const ref);<br>see [note 1](#note1) |
| &nbsp;       | value_type const & operator *() const                        | the value (const ref);<br>see [note 1](#note1) |
| &nbsp;       | value_type & operator *()                                    | the value (non-const ref);<br>see [note 1](#note1) |

<a id="note1"></a>Note 1: checked access: if no content, throws status value.

### Configuration macros

\-D<b>nsel\_CONFIG\_CONFIRMS\_COMPILATION\_ERRORS</b>=0  
Define this macro to 1 to experience the by-design compile-time errors of the library in the test suite. Default is 0.

### Macros to control alignment

If *status_value* is compiled as C++11 or later, C++11 alignment facilities are used for storage of the underlying object. When compiled as pre-C++11, *status_value* tries to determine proper alignment itself. If this doesn't work out, you can control alignment via the following macros. See also section [Implementation notes](#implementation-notes).

-D<b>nssv_FEATURE_MAX_ALIGN_HACK</b>=0  
Define this to 1 to use the *max align hack* for alignment. Default is 0.

-D<b>nssv_FEATURE_ALIGN_AS</b>=*pod-type*  
Define this to the *pod-type* you want to align to (no default).

-D<b>nssv_FEATURE_ALIGN_AS_FALLBACK</b>=*pod-type*  
Define this to the *pod-type* to use for alignment if the algorithm of *status_value* cannot find a suitable POD type to use for alignment. Default is double.


Reported to work with
---------------------
*status_value_cpp98* is reported to work with the following compilers: 
- Visual C++ 6 SP6 (VS6), VC10, (VS2010), VC11 (VS2012), VC12 (VS2013), VC14 (VS2015)
- GNUC 5.2.0 with -std=c++98, -std=c++03, -std=c++11, -std=c++14, -std=c++1y 
- clang 3.6, 3.7 with -std=c++03, -std=c++11 (on Travis)


Implementation notes
--------------------

### Object allocation and alignment

*status_value* reserves POD-type storage for an object of the underlying type inside a union to prevent unwanted construction and uses placement new to construct the object when required. Using non-placement new (malloc) to  obtain storage, ensures that the memory is properly aligned for the object's type, whereas that's not the case with placement new.

If you access data that's not properly aligned, it 1) may take longer than when it is properly aligned (on x86 processors), or 2) it may terminate the program immediately (many other processors).

Although the C++ standard does not guarantee that all user-defined types have the alignment of some POD type, in practice it's likely they do [6, part 2].

If *status_value* is compiled as C++11 or later, C++11 alignment facilities are used for storage of the underlying object. When compiling as pre-C++11, *status_value* tries to determine proper alignment using meta programming. If this doesn't work out, you can control alignment via three macros. 

*status_value* uses the following rules for alignment:

1. If the program compiles as C++11 or later, C++11 alignment facilities  are used.

2. If you define -D<b>nssv_FEATURE_MAX_ALIGN_HACK</b>=1 the underlying type is aligned as the most restricted type in `struct max_align_t`. This potentially wastes many bytes per optional if the actually required alignment is much less, e.g. 24 bytes used instead of the 2 bytes required.

3. If you define -D<b>nssv_FEATURE_ALIGN_AS</b>=*pod-type* the underlying type is aligned as *pod-type*. It's your obligation to specify a type with proper alignment.

4. If you define -D<b>nssv_FEATURE_ALIGN_AS_FALLBACK</b>=*pod-type* the fallback type for alignment of rule 5 below becomes *pod-type*. It's your obligation to specify a type with proper alignment.

5. At default, *status_value* tries to find a POD type with the same alignment as the underlying type. 

	The algorithm for alignment of 5. is:
	- Determine the alignment A of the underlying type using `alignment_of<>`.
	- Find a POD type from the list `alignment_types` with exactly alignment A.
	- If no such POD type is found, use a type with a relatively strict alignment requirement such as double; this type is specified in  `nssv_FEATURE_ALIGN_AS_FALLBACK` (default double).

Note that the algorithm of 5. differs from the one Andrei Alexandrescu uses in [6, part 2].

The class template `alignment_of<>` is gleaned from [Boost.TypeTraits, alignment_of](http://www.boost.org/doc/libs/1_57_0/libs/type_traits/doc/html/boost_typetraits/reference/alignment_of.html) [10]. The storage type `storage_t<>` is adapted from the one I created for [optional lite](https://github.com/martinmoene/optional-lite) [10].

For more information on constructed unions and alignment, see [5-9].


Notes and references
--------------------

<a id="ref1"></a>[1] Lawrence Crowl and Chris Mysen. [N4233 - A Class for Status and Optional Value](http://wg21.link/n4233). 10 October 2014.

<a id="ref2"></a>[2] Lawrence Crowl. [P0157R0 - Handling Disappointment in C++](http://wg21.link/p0157r0). 7 July 2015.

<a id="ref3"></a>[3] Vicente J. Botet Escriba. [Dxxxxr0 - A proposal to add a utility class to represent expected monad (Revision 2)](https://github.com/viboes/std-make/blob/master/doc/proposal/expected/DXXXXR0_expected.pdf) (PDF). 12 March 2016.

<a id="ref4"></a>[4] Fernando Cacciola and Andrzej Krzemieński. [N3793 - A proposal to add a utility class to represent optional objects (Revision 5)](http://wg21.link/n3793). 2013-10-03.  

<a id="ref5"></a>[5] Andrei Alexandrescu. [Generic<Programming>: Discriminated Unions part 1](http://collaboration.cmc.ec.gc.ca/science/rpn/biblio/ddj/Website/articles/CUJ/2002/cexp2004/alexandr/alexandr.htm), [part 2](http://collaboration.cmc.ec.gc.ca/science/rpn/biblio/ddj/Website/articles/CUJ/2002/cexp2006/alexandr/alexandr.htm), [part 3](http://collaboration.cmc.ec.gc.ca/science/rpn/biblio/ddj/Website/articles/CUJ/2002/cexp2008/alexandr/alexandr.htm). April 2002. 

<a id="ref6"></a>[6] Herb Sutter. [Style Case Study #3: Construction Unions](http://www.gotw.ca/gotw/085.htm). GotW #85. 2009

<a id="ref7"></a>[7] Kevin T. Manley. [Using Constructed Types in C++ Unions](http://collaboration.cmc.ec.gc.ca/science/rpn/biblio/ddj/Website/articles/CUJ/2002/0208/manley/manley.htm). C/C++ Users Journal, 20(8), August 2002.

<a id="ref8"></a>[8] StackOverflow. [Determining maximum possible alignment in C++](http://stackoverflow.com/a/3126992).

<a id="ref9"></a>[9] [Boost.TypeTraits, alignment_of](http://www.boost.org/doc/libs/1_57_0/libs/type_traits/doc/html/boost_typetraits/reference/alignment_of.html) ( [code](http://www.boost.org/doc/libs/1_57_0/boost/type_traits/alignment_of.hpp) ).

<a id="ref5"></a>[10] Martin Moene. [optional lite](https://github.com/martinmoene/optional-lite) ([optional.hpp](https://github.com/martinmoene/optional-lite/blob/master/optional.hpp)).


Appendix
--------
### A.1 status_value test specification

```
status_value<>: Disallows default construction
status_value<>: Allows construction from only status
status_value<>: Allows construction from status and non-default-constructible value
status_value<>: Allows construction from copied status and moved value (C++11)
status_value<>: Allows construction from copied status and copied value
status_value<>: Allows copy-construction from other status_value of the same type (pre C++11)
status_value<>: Allows move-construction from other status_value of the same type (C++11)
status_value<>: Allows to observe its status
status_value<>: Allows to observe the presence of a value (has_value())
status_value<>: Allows to observe the presence of a value (operator bool)
status_value<>: Allows to observe its value
status_value<>: Throws status when observing a non-present value```
