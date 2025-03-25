C++面向对象的设计原则主要包括**SOLID 原则**和一些**常见的设计模式指导原则**。这些原则可以帮助我们写出**高内聚、低耦合、可维护、可扩展**的代码。
# SOLID 设计原则
SOLID 是五大核心面向对象设计原则的缩写，包括：

S（单一职责原则，SRP）

O（开闭原则，OCP）

L（里氏替换原则，LSP）

I（接口隔离原则，ISP）

D（依赖倒置原则，DIP）
## 单一职责原则
一个类只应该有一个引起它变化的原因，即一个类**只负责一项职责**。
~~~
class ReportGenerator {
public:
    void generateReport() { std::cout << "生成报告" << std::endl; }
};
class ReportPrinter {
public:
    void printReport() { std::cout << "打印报告" << std::endl; }
};
~~~
每个类都有单一职责，如果修改 printReport()，不会影响 generateReport()。
## 开闭原则
对扩展开放，对修改关闭，即**扩展功能时不修改原有代码**，而是通过**新增代码**来实现。
~~~
class Shape {
public:
    virtual void draw() = 0; // 纯虚函数
};
class Rectangle : public Shape {
public:
    void draw() override { std::cout << "绘制矩形" << std::endl; }
};
class Circle : public Shape {
public:
    void draw() override { std::cout << "绘制圆形" << std::endl; }
};
class Painter {
public:
    void drawShape(Shape* shape) { shape->draw(); }
};
~~~
如果要增加三角形，只需继承 Shape，而不需要修改 Painter，实现**对修改关闭，对扩展开放**。
## 里氏替换原则
子类必须能替换基类，而不会影响程序正确性。
~~~
// 抽象鸟类
class Bird {
public:
    virtual void makeSound() const = 0;
    virtual ~Bird() {}
};
// 会飞的鸟类
class FlyingBird : public Bird {
public:
    virtual void fly() const = 0;
};
// 麻雀类，继承自会飞的鸟类
class Sparrow : public FlyingBird {
public:
    void makeSound() const override {
        std::cout << "Chirp!" << std::endl;
    }
    void fly() const override {
        std::cout << "Flying..." << std::endl;
    }
};
// 鸵鸟类，继承自鸟类
class Ostrich : public Bird {
public:
    void makeSound() const override {
        std::cout << "Boom!" << std::endl;
    }
};
~~~
## 接口隔离原则
不要强迫类实现不需要的接口，即 一个接口只包含必要的方法。
~~~
class IPrinter {
public:
    virtual void print() = 0;
};
class IScanner {
public:
    virtual void scan() = 0;
};
class Printer : public IPrinter {
public:
    void print() override { std::cout << "打印机打印" << std::endl; }
};
~~~
Printer 只实现 print()，不会被迫实现不需要的接口。
## 依赖倒置原则
高层模块不应该依赖低层模块，而应该依赖抽象（接口）。
~~~
class InputDevice {
public:
    virtual void input() = 0;
};
class Keyboard : public InputDevice {
public:
    void input() override { std::cout << "键盘输入" << std::endl; }
};
class Computer {
private:
    InputDevice* device;
public:
    Computer(InputDevice* d) : device(d) {}
    void use() { device->input(); }
};
~~~
Computer**依赖抽象**（InputDevice），可以轻松扩展新设备（如鼠标）。
