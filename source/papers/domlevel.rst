DOM Level Fuzz
--------------------------------------------------

这是Nduja[3-5]实现的主要的思路。该方法认为基于DOM的Fuzz可以分为三个层次，第一层是常规的随机生成元素，这种思路是最普遍的，所以效率不高。作者认为应该从更高的层次去fuzz。比如基于DOM的第二（基于逻辑）第三层（基于事件）进行fuzz，以期获得更好的效果。

从第二层出发，更多考虑的是基于DOM逻辑进行变换。举例来说，在一棵DOM树上，存在的逻辑有firstChild/lastChild/nextNode/previousNode/nextSiebling/previousSiebling等，而节点则存在detach/attach/nextNode/previousNode等逻辑，可以基于这相关的逻辑进行变换。

第三层考虑的是DOM之间发生变化时相应触发的事件。利用事件之间的逻辑关系去fuzz，比如节点A发生detach时，会触发让该节点attach到节点B的动作，但是又会触发节点删除子节点的动作。用这样一些逻辑上的边界条件去尝试fuzz。
