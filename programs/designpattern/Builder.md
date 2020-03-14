[TOC]

# 概览
解决具有大量参数的构造函数不好用的问题，初始化参数太多太复杂就不好了

创建者模式又叫建造者模式，是将一个复杂的对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。创建者模式隐藏了复杂对象的创建过程，它把复杂对象的创建
过程加以抽象，通过子类继承或者重载的方式，动态的创建具有复合属性的对象。

# 组成
+ 建造者（Builder）：对复杂对象的创建过程加以抽象，给出一个抽象接口，以规范产品对象的各个组成部分的建造。
+ 具体创建者（ConcreateBuilder）：实现Builder接口，针对不同的业务逻辑，具体化复杂对象的各个部分的创建。在建造过程完成后，提供产品的实例
+ 指导者（Dierctor）：调用具体建造者来创建复杂对象的各个部分，在指导者中不设计具体产品的信息，只负责保证对象各部分完整创建或者按某种顺序创建。
+ 产品（Product）：要创建的复杂对象，一般来说包含多个部分。

# 优点
+ 建造模式的使用使得产品的内部表象可以独立地变化。使用建造模式可以使客户端不必知道产品内部组成的细节。
+ 每一个Builder都相对独立，而与其他的Builder无关。
+ 模式所建造的最终产品更易于控制。

# 缺点
+ 建造者模式的"加工工艺"是暴露的，这样使得建造者模式更加灵活，也使得工艺变得对客户不透明。

# 适用场景
+ 隔离复杂对象的创建和使用，相同的方法，不同执行顺序，产生不同事件结果
+ 多个部件都可以装配到一个对象中，但产生的运行结果不相同
+ 产品类非常复杂或者产品类因为调用顺序不同而产生不同作用
+ 初始化一个对象时，参数过多，或者很多参数具有默认值
+ Builder模式不适合创建差异性很大的产品类
+ 产品内部变化复杂，会导致需要定义很多具体建造者类实现变化，增加项目中类的数量，增加系统的理解难度和运行成本
+ 需要生成的产品对象有复杂的内部结构，这些产品对象具备共性；

# 例子
+ 工厂（建造者模式）：负责制造汽车（组装过>程和细节在工厂内）
+ 汽车购买者（用户）：你只需要说出你需要的>型号（对象的类型和内容），然后直接购买就可>>以使用了（不需要知道汽车是怎么组装的（车轮、车门、>发动机、方向盘等等））

# 代码
```java
class Cpu{}
class Memory{}
class Ssd{}
class Display{}

public class PcBuilder {
    private Cpu cpu;
    private Memory memory;
    private Ssd ssd;
    private Display display;

    PcBuilder(Cpu cpu, Memory memory, Ssd ssd, Display display) {
        this.cpu = cpu;
        this.memory = memory;
        this.ssd = ssd;
        this.display = display;
    }

    public static PcBuilderBuilder builder() {
        return new PcBuilderBuilder();
    }

    public static class PcBuilderBuilder {
        private Cpu cpu;
        private Memory memory;
        private Ssd ssd;
        private Display display;

        PcBuilderBuilder() {
        }

        public PcBuilderBuilder cpu(Cpu cpu) {
            this.cpu = cpu;
            return this;
        }

        public PcBuilderBuilder memory(Memory memory) {
            this.memory = memory;
            return this;
        }

        public PcBuilderBuilder ssd(Ssd ssd) {
            this.ssd = ssd;
            return this;
        }

        public PcBuilderBuilder display(Display display) {
            this.display = display;
            return this;
        }

        public PcBuilder build() {
            return new PcBuilder(this.cpu, this.memory, this.ssd, this.display);
        }

        public String toString() {
            return "PcBuilder.PcBuilderBuilder(cpu=" + this.cpu + ", memory=" + this.memory + ", ssd=" + this.ssd + ", display=" + this.display + ")";
        }
    }

    public static void main(String[] args) {
        PcBuilder pc = builder()
                .cpu(new Cpu())
                .memory(new Memory())
                .ssd(new Ssd())
                .display(new Display())
                .build();
    }

}
```