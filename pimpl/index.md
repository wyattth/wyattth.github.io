pImpl Lite
==========

When you know that your class will be used as heap objects, it is often simpler
to use an explicit smart pointer ```std::unique_ptr<Widget>``` rather than a
fully wrapped value-type piple. Compare:

```c++
class Gadget_simple {
    std::unique_ptr<SimpleWidget> widget_;
    
    void foo() {
        widget_->doSomething();
    }
};

class Gadget_pImpl {
    PimplWidget widget_;
    
    void foo() {
        widget_.doSomething();
    }
};
```

This approach allows the type of memory management (i.e. ```std::unique_ptr<Widget>``` 
vs ```std::shared_ptr<Widget>``` vs ```std::weak_ptr<Widget>```).

Define a header file ```Widget.h```...

```c++
#ifndef Widget_h
#define Widget_h

#include <memory>  // for unique_ptr<>

namespace io { namespace Core {
    class Widget;
} }

class io::Core::Widget {
    class Implementation;  // defined in Widget.cpp
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
    int id;
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

void foo() {
    std::unique_ptr<io::Core::Widget> w = io::Core::Widget::make(42);
    w->zip(100);
}
```