
1、dragListener
1.1 v.startDrag(null, new DragShadowBuilder(v), v, 0);
1.2 child.setOnDragListener(dragListener);

可以跨进程传递，再onDragEvent()中可以收到其他进程的view
api11就有，重点是内容的移动，不是view的移动，所以可以传递数据。

2、ViewDragHelper
2.1

drawerLayout
slidingMenu

2015v4包，重点是拖view
