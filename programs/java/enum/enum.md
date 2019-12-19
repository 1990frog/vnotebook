[TOC]

```java
public enum Weekplus {

    /**
     * 1. 枚举值必须要定义在前面
     */
    MONDAY(1,"星期一"),
    TUESDAY(2,"星期二"),
    WEDNESDAY(3,"星期三"),
    THURSDAY(4,"星期四"),
    FRIDY(5,"星期五"),
    SATURDAY(6,"星期六"),
    SUNDAY(7,"星期天");

    /**
     * 2. 可以有属性
     */
    private int value;
    private String desc;

    /**
     * 3. 构造函数必须是私有的
     */
    private Weekplus(int value, String desc){
        this.value = value;
        this.desc = desc;
    }
    public int getValue() {
        return value;
    }
    public String getDesc() {
        return desc;
    }
}
```