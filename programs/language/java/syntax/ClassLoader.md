[TOC]

从文档中对ClassLoader类的介绍可以总结出这个类的作用就是根据一个指定的类的全限定名,找到对应的Class字节码文件,然后加载它转化成一个java.lang.Class类的一个实例.

因为它是加载字节码，所有在运行期间用的特别多（动态代理）