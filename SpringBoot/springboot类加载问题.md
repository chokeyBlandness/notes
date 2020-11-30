# SpringBoot类加载问题

### 问题描述

SpringBoot中的类在开发环境和生产环境被加载到不同的类加载器中。

SpringBoot实现了自己的类加载器机制，因此开发环境中AppClassLoader能够加载到SpringBoot的类，但是打包后，Spring类就无法在AppClassLoader中加载到。