# SPI

参考了java的spi机制,提供插件进行灵活扩展功能。

采用微内核+插件方式，提供扩展功能。

#### java-spi

在META-INF/services目录下，创建一个文件，包含服务接口具体实现类，然后java-spi在使用到这个类时，查找具体实现类，进行实例化
