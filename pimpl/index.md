---
title: simple site
tagline: Easy websites with GitHub Pages
description: Minimal tutorial on making a simple website with GitHub Pages
---

The **pImpl** idiom creates light-weight **value** types that hide their size and content
details in a privately and exclusively-owned heap object. In C++11, this uses as ```std::unique_ptr<T>``` 
as described in Scott Meyer's "Effective Modern C++". Methods are externally-linked non-virtual functions.

An simpler alternative offering greater transparency, a simpler implementation and more control by the client
is to provide a **factory** method returning ```unique_ptr```'s to a derived implementation of the public class.

When you know that your class will be used as heap objects, it is often simpler
to use an explicit smart pointer ```std::unique_ptr<Widget>``` rather than a
fully wrapped value-type piple. Compare:

```c++
class Gadget_simple {
    std::unique_ptr<SimpleWidget>  widget_ = SimpleWidget::make(44);
public:  
    void foo() {
        widget_->doSomething();
    }
};

class Gadget_pImpl {
    PimplWidget  widget_ { 44 };
public:    
    void foo() {
        widget_.doSomething();
    }
};
```

This approach allows the client (rather than the ```Widget``` object itself) to control the ownership/lifespan of the ```Widget```
(choosing ```std::unique_ptr<Widget>```, ```std::shared_ptr<Widget>``` or ```std::weak_ptr<Widget>```).
With a pImpl class, the ownership hidden (and fixed) inside the private definition.
The result, every pImpl class needs a significant amount of boiler-plate code:
 * a constructor that manually constructs the implementation
 * a default move constructor in class definition and implementation
 * a default move assignment in class definition and implementation
 * a default copy constructor (if appropriate) in class definition and implementationx
 * a default copy assignment (if appropriate) in class definition and implementation
 * a default destructor in class definition and implementation
 * a manual forwarding of every method from the pImpl object to its implementation

Define a header file ```Widget.h```...

```c++
#ifndef Widget_h
#define Widget_h

#include <memory>  // for unique_ptr<>

namespace io { namespace Core {  // define class in suitable namsepace
    class Widget;
} }

class io::Core::Widget {   
    class Implementation;  // private implementation defined in Widget.cpp
public:
    static std::unique_ptr<Widget> make(int a);  // factory
    virtual ~Widget() {}  // allow destruction via reference to Widget
// public API
    virtual void zip(int) = 0;
};

#endif /* Widget_h */
```

Now do the implementation itself in ```Widget.cpp```...

```c++
#include "Widget.h"
#include <iostream>

using io::Core::Widget;
using namespace std;

struct Widget::Implementation : Widget {
    int  id;
    Implementation(int a);
    void zip(int) override;
};

void Widget::Implementation::zip(int n) {
    cout << "Widget#" << id << " zips " << n << endl;
}

Widget::Implementation::Implementation(int a) : id(a) { }

unique_ptr<Widget> Widget::make(int a) {
    return unique_ptr<Widget> { new Implementation(a) };
};
```

So now we can use the ```Widget``` as follows

```c++
#include "Widget.h"

using io::Core::Widget;
using namespace std;

void foo() {
    unique_ptr<Widget> w = Widget::make(42);
    w->zip(100);
}
```
