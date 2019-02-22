---

layout: post

title: 'Android View的测量和布局'

subtitle: ''

date: 2017-03-01

categories: Android基础

tags: Android-View

---
简单讲一下View的测量,布局,绘制,和事件
---

# 绘制
## onMeasure 测量
  * ### ViewGroup的测量
      1. 遍历调用每个子view的measure()来计算view的尺寸.根据父视图的measureSpec和子视图的layoutParams来决定子视图的measureSpec.
      2. 计算子view的位置.并保存子view的位置和尺寸.
      3. 计算自己的尺寸并用setMeasuredDimension()保存.
    
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
//      super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int mode = MeasureSpec.getMode(widthMeasureSpec);
    int size = MeasureSpec.getSize(widthMeasureSpec);
    for (int i = 0; i < getChildCount(); i++) {
        View view = getChildAt(i);
        LayoutParams lp = view.getLayoutParams();
        int viewWidthSpec = 0;
        int viewHeightSpec = 0;
        switch (lp.width) {
            case LayoutParams.MATCH_PARENT:
                if (mode == MeasureSpec.EXACTLY || mode == MeasureSpec.AT_MOST) {
                    viewWidthSpec = MeasureSpec.makeMeasureSpec(size, MeasureSpec.EXACTLY);
                } else {
                    viewWidthSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
                }
                break;
            case LayoutParams.WRAP_CONTENT:
                if (mode == MeasureSpec.EXACTLY || mode == MeasureSpec.AT_MOST) {
                    viewWidthSpec = MeasureSpec.makeMeasureSpec(size, MeasureSpec.AT_MOST);
                } else {
                    viewWidthSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
                }
                break;
            default:
                viewWidthSpec = MeasureSpec.makeMeasureSpec(lp.width, MeasureSpec.EXACTLY);
                break;
        }
        view.measure(viewWidthSpec, viewHeightSpec);
    }
}	
```
  
## onlayout 布局  
## onDraw 绘制
### 两个方法
 1. Invalidate
      > 当view的appearance发生改变.如状态,焦点,背景只要是自身的都会引起invalidate操作.所以当我们改变了view的appearance,需要刷新时,需要调用invalidate(),viewGroup的话,重绘整个view树
      
2. RequestLayout
    > view的边界,也就是宽高发生变化.不适合现在的区域,则调用该方法.View执行requestLayout方法，会向上递归到顶级父View中，再执行这个顶级父View的requestLayout，所以其他View的onMeasure，onLayout也可能会被调用。
    
    > 调用invalidate方法只会执行onDraw方法；调用requestLayout方法只会执行onMeasure方法和onLayout方法，并不会执行onDraw方法

   
# 事件分发
  ![](http://ww1.sinaimg.cn/large/6ad807f3gy1fplqc6aydnj20jg0m4myi.jpg)
  
> 父视图分发到子视图.可以先拦截处理,也可以在子视图不处理后,再处理.
 
## 嵌套视图 
1. scroolview嵌套listview
    1. scrollview重写`onInterceptTouchEvent`,并返回false.意思是scrollview不拦截listview的事件,让其自身处理.(也就是遍历所有子view)找到子View调用dispatchTouchEnvent方法,让其自行处理.而没有被listview覆盖的部分,还会继续调用ScrollView的onTouchEvent方法.
    2. listview重写dispatchTouchEvent方法.加入`getParent().requestDisallowInterceptTouchEvent(true);`同理1.都是将事件交给子view.
    3. 设置setOnTouchListener监听
    
```java
//当用户按下的时候，我们告诉父组件，不要拦截我的事件（这个时候子组件是可以正常响应事件的），拿起之后就会告诉父组件可以阻止。
mListView.setOnTouchListener(new View.OnTouchListener() {
           @Override
           public boolean onTouch(View v, MotionEvent event) {
               switch (event.getAction()) {
                   case MotionEvent.ACTION_MOVE:                    
                       v.getParent().requestDisallowInterceptTouchEvent(true);
                       break;
                   case MotionEvent.ACTION_UP:
                   case MotionEvent.ACTION_CANCEL:                        
                       v.getParent().requestDisallowInterceptTouchEvent(false);
                       break;
               }
               return false;
           }
       }); 
}
```
  
  

