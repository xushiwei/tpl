tpl
===

TPL is `Text Processing Language`


## Field Extraction

### Example 1

```c++
#include <vector>               // std::vector
#include <iostream>     // std::cout
#include <tpl/regex/Match.h> // minimization of including tpl

using namespace tpl;

void main()
{
  std::vector<double> values; // you can change vector to other stl containers.

  if ( "-.1 -0.1 +32. -22323.2e+12" >> real()/append(values) % ws() )
  {
    for (std::vector<double>::iterator it = values.begin(); it != values.end(); ++it)
    {
      std::cout << *it << "\n";
    }
  }
}
```

Output:

```c++
-0.1
-0.1
32
-2.23232e+16
```

What we use:

* Rules: real(), ws(), operator %
* Actions: /append(Container)
* Matching: operator >> 

### Example 2

```c++
#include <vector>               // std::vector
#include <iostream>     // std::cout
#include <tpl/regex/Match.h> // minimization of including tpl

using namespace tpl;

void main()
{
  std::vector<double> values; // you can change vector to other stl containers.
        
  if ( " -.1 , -0.1 , +32. , -22323.2e+12 " >> skipws_[real()/append(values) % gr(',')] )
  {
    for (std::vector<double>::iterator it = values.begin(); it != values.end(); ++it)
    {
      std::cout << *it << "\n";
    }
  }
}
```

Output:

```c++
-0.1
-0.1
32
-2.23232e+16
```

What we use:

* Rules: real(), ','
* Grammars: gr(Rule), operator %
* Skipper: skipws_[] (syntax: Skipper[Grammar])
* Actions: /append(Container)
* Matching: operator >> 

## Data Validation

### Example 1

```c++
#include <iostream>     // std::cout
#include <tpl/regex/Match.h> // minimization of including tpl

using namespace tpl;

void validation_example_1(const char* str)
{
  if (str == integer())
    std::cout << "  " << str << " is an integer value.\n";
  else if (str == real())
    std::cout << "  " << str << " is a real value.\n";
  else
    std::cout << "  " << str << " isn't a numeric value.\n";
}

void main()
{
  validation_example_1("-135");
  validation_example_1("+.23e-23");
  validation_example_1("-.1.e23");
}
```

Output:

```c++
  -135 is an integer value.
  +.23e-23 is a real value.
  -.1.e23 isn't a numeric value.
```

What we use:

* Rules: integer(), real()
* Matching: operator == 

### Example 2

```c++
#include <iostream>     // std::cout
#include <tpl/regex/Match.h> // minimization of including tpl

using namespace tpl;

void validation_example_2(const char* str)
{
  int val;
  if (str == integer()/assign(val) && val >= 1 && val <= 12)
    std::cout << "  " << str << " is an integer value between 1 to 12.\n";
  else
    std::cout << "  " << str << " is not valid data.\n";
}

void main()
{
  validation_example_2("3");
  validation_example_2("13");
  validation_example_2("-135");
  validation_example_2("+.23e-23");
}
```

Output:

```c++
  3 is an integer value between 1 to 12.
  13 is not valid data.
  -135 is not valid data.
  +.23e-23 is not valid data.
```

What we use:

* Rules: integer()
* Actions: /assign(Variable)
* Matching: operator == 

### Example 3

```c++
#include <iostream>     // std::cout
#include <tpl/regex/Match.h> // minimization of including tpl
#include <boost/spirit/phoenix.hpp>

using namespace phoenix;
using namespace tpl;

void validation_example_3(const char* str)
{
  if (str == integer()/meet(arg1 >= 1 && arg1 <= 12))
    std::cout << "  " << str << " is an integer value between 1 to 12.\n";
  else
    std::cout << "  " << str << " is not valid data.\n";
}

void main()
{
  validation_example_3("3");
  validation_example_3("13");
  validation_example_3("-135");
  validation_example_3("+.23e-23");
}
```

Output:

```c++
  3 is an integer value between 1 to 12.
  13 is not valid data.
  -135 is not valid data.
  +.23e-23 is not valid data.
```

What we use:

* Rules: integer()
* Predicate: /meet(Condition)
* Matching: operator ==
* Boost Phoenix: arg1 >= 1 && arg1 <= 12 

If you don't want to use Boost Phoenix, you can:

```c++
void validation_example_3(const char* str)
{
  if (str == integer()/(ge(1) && le(12)))
    std::cout << "  " << str << " is an integer value between 1 to 12.\n";
  else
    std::cout << "  " << str << " is not valid data.\n";
}
```

What we use:

* Rules: integer()
* Predicate: ge(Value) (GreatThan or EqualTo), le(Value) (LessThan or EqualTo), operator &&
* Matching: operator == 

## Data Extraction (DOM)

```c++
#include <tpl/c/Lex.h>
#include <tpl/regex/DOM.h>

using namespace tpl;

void main()
{
  typedef DOM<> dom;

  const char source[] = "class Foo : public Base1, Base2 {};";
        
  dom::Mark tagName("name");
  dom::NodeMark tagBase("base", true);
    dom::Mark tagAccess("access");
    //dom::Mark tagName("name");
        
  dom::Allocator alloc;
  dom::Document doc(alloc);

  if (
    source >> cpp_skip_
      [
        gr(c_symbol()/eq("class")) + c_symbol()/tagName + ':' +
        (
          !gr(c_symbol()/(eq("public")||eq("private"))/tagAccess) + c_symbol()/tagName
        )/tagBase % ',' +
        '{' + '}' + ';'
      ]/doc
    )
  {
    NS_STDEXT::OutputLog log;
    json_print(alloc, log, doc);
  }
}
```

Output:

```c++
{
  "name": "Foo",
  "base": 
  [
    {
      "access": "public",
      "name": "Base1"
    },
    {
      "name": "Base2"
    }
  ]
}
```

What we use:

* Rules: c_symbol(), ':'
* Grammars: gr(Rule), operator %, operator +, operator !
* Skipper: cpp_skip_[] (syntax: Skipper[Grammar])
* Predicate: /eq(Value), operator||
* Mark: /Mark, /NodeMark, /Document
* Matching: operator >>
* DOM Formatter: json_print 

## Demo: Calculator

See http://winx.googlecode.com/svn/trunk/tpl/examples/calculator/Calc.cpp.

```c++
#define TPL_USE_AUTO_ALLOC
#include <cmath>                // sin, cos, pow
#include <deque>                // std::deque
#include <iostream>     // std::cout
#include <functional>   // std::plus, std::minus, etc
#include <algorithm>    // std::max_element
#include <tpl/ext/Calculator.h>

using namespace tpl;

double max_value(const double x[], int count)
{
  return *std::max_element(x, x+count);
}

void calculator(const char* szExpr)
{
  typedef SimpleImplementation impl;

  // ---- define rules ----

  std::deque<double> stk;

  impl::Grammar::Var rFactor;

  impl::Grammar rTerm =
    rFactor + *(
      '*' + rFactor/calc<std::multiplies>(stk) | 
      '/' + rFactor/calc<std::divides>(stk) );

  impl::Grammar rExpr =
    rTerm + *(
      '+' + rTerm/calc<std::plus>(stk) |
      '-' + rTerm/calc<std::minus>(stk) );

  int arity;
  impl::Rule rFun =
    "sin"/calc(stk, sin, arity) | "cos"/calc(stk, cos, arity) |
    "pow"/calc(stk, pow, arity) | "max"/calc(stk, max_value, arity);

  rFactor =
    real()/append(stk) |
    '-' + rFactor/calc<std::negate>(stk) |
    '(' + rExpr + ')' |
    (gr(c_symbol()) + '(' + rExpr % ','/assign(arity) + ')')/(gr(rFun) + '(') |
    '+' + rFactor;

  // ---- do match ----

  try {
    if ( szExpr != skipws_[rExpr] )
      std::cerr << ">>> ERROR: invalid expression!\n";
    else
      std::cout << stk.back() << '\n';
  }
  catch (const std::logic_error& e) {
    std::cerr << ">>> ERROR: " << e.what() << '\n';
  }
}

int main(int argc, const char* argv[])
{
  if (argc == 2) {
    calculator(argv[1]);
    return 0;
  }
  else {
    for (;;)
    {
      std::string strExp;
      std::cout << "input an expression (q to quit): ";
      if (!std::getline(std::cin, strExp) || strExp == "q") {
        std::cout << '\n';
        return 0;
      }
      calculator(strExp.c_str());
    }
  }
}
```

Input/Outputs:

Input: 2 - 3 * 4
Output: -10

Input: max(1, pow(3, 2), cos(-2) * 0.2 - 1)
Output: 9

What we use:

* Rules: real(), c_symbol(), "sin", '*'
* Grammars: gr(Rule), operator %, operator +, operator |, unary operator *
* Skipper: skipws_[] (syntax: Skipper[Grammar])
* Actions: /append(Container), /assign(Variable), /calc<Functor>(Stack), /calc(Stack, Function, Arity)
* Restriction: /(gr(rFun) + '(') (syntax: Grammar/Grammar)
* Reference: impl::Grammar::Var
* Matching: operator !=
