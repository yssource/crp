# Moved members

One common use of null pointers in modern C++ is as values for the members of
moved objects so that the destructor can still safely be called. E.g.,

```c++,hidelines=#
# #include <cstdlib>
# #include <cstring>
#
// widget.h
struct widget_t;
widget_t *alloc_widget();
void free_widget(widget_t*);
void copy_widget(widget_t* dst, widget_t* src);

// widget.cc
class Widget {
    widget_t* widget;
public:
#    Widget() : widget(alloc_widget()) {}
#
#    Widget(const Widget &other) : widget(alloc_widget()) {
#        copy_widget(widget, other.widget);
#    }
#
    Widget(Widget &&other) : widget(other.widget) {
        other.widget = nullptr;
    }

    ~Widget() {
        free_widget(widget);
    }
};
```

Rust's notion of moving objects does not involve leaving behind an object on
which a destructor will be called, and so this use of null does not have a
corresponding idiom. See the chapter on [copy and move
constructors](/idioms/constructors/copy_and_move_constructors.md) for more
details.
