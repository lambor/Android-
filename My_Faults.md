# My Silly Faults 我SB了

* ArrayList add / HashMap put reference or new copy?  ArrayList HashMap容器中存储的是原对象引用还是副本？
  + 有一次使用ArrayList保存一些checkbox时发生了一个超出我意料的事情，发现ArrayList add存储的好像是对象的副本，然后发现HashMap保存的是原对象的引用。我当时就查了ArrayList的源码，发现其使用System.copyarray实现的。所以我就自以为然地认为ArrayList在自己容量不够时会建扩建，并使用System.copy来拷贝一个原来的副本。所以我就认为ArrayList的效率也太低了吧。
  + 直到我看到《think in java》中数组那一章中说**System.copyarray拷贝的是引用**，即浅拷贝。当时就懵逼了，然后就各种测试，发现确实是。然后就查原来的代码，然而却没有找到...

