# DavidBounceScrollView
##【Android】Android开发实现带有反弹效果，仿IOS反弹scrollview详解教程

作者：程序员小冰，

新浪微博：http://weibo.com/mcxiaobing 

首先给大家看一下我们今天这个最终实现的效果图：

![这里写图片描述](http://img.blog.csdn.net/20160908095941142)

这个是ios中的反弹效果。当然我们安卓中如果想要实现这种效果，

感觉不会那么生硬，滚动到底部或者顶部的时候。当然

使用scrollview是无法实现的。所以我们需要新建一个view继承ScrollView

```
package davidbouncescrollview.qq986945193.com.davidbouncescrollview;

import android.annotation.SuppressLint;
import android.content.Context;
import android.graphics.Rect;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.view.animation.TranslateAnimation;
import android.widget.ScrollView;

/**
 * @author ：程序员小冰
 * @新浪微博 ：http://weibo.com/mcxiaobing
 * @GitHub:https://github.com/QQ986945193
 * @CSDN博客: http://blog.csdn.net/qq_21376985
 * @交流Qq ：986945193
 * 类名：带有反弹效果的scrollview
 */
public class BounceScrollView extends ScrollView {
    private View inner;// 孩子View

    private float y;// 点击时y坐标

    private Rect normal = new Rect();// 矩形(这里只是个形式，只是用于判断是否需要动画.)

    private boolean isCount = false;// 是否开始计算

    public BounceScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    /***
     * 根据 XML 生成视图工作完成.该函数在生成视图的最后调用，在所有子视图添加完之后. 即使子类覆盖了 onFinishInflate
     * 方法，也应该调用父类的方法，使该方法得以执行.
     */
    @SuppressLint("MissingSuperCall")
    @Override
    protected void onFinishInflate() {
        if (getChildCount() > 0) {
            inner = getChildAt(0);
        }
    }

    /***
     * 监听touch
     */
    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        if (inner != null) {
            commOnTouchEvent(ev);
        }

        return super.onTouchEvent(ev);
    }

    /***
     * 触摸事件
     *
     * @param ev
     */
    public void commOnTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_UP:
                // 手指松开.
                if (isNeedAnimation()) {
                    animation();
                    isCount = false;
                }
                break;
            /***
             * 排除出第一次移动计算，因为第一次无法得知y坐标， 在MotionEvent.ACTION_DOWN中获取不到，
             * 因为此时是MyScrollView的touch事件传递到到了LIstView的孩子item上面.所以从第二次计算开始.
             * 然而我们也要进行初始化，就是第一次移动的时候让滑动距离归0. 之后记录准确了就正常执行.
             * https://github.com/QQ986945193
             */
            case MotionEvent.ACTION_MOVE:
                final float preY = y;// 按下时的y坐标
                float nowY = ev.getY();// 时时y坐标
                int deltaY = (int) (preY - nowY);// 滑动距离
                if (!isCount) {
                    deltaY = 0; // 在这里要归0.
                }

                y = nowY;
                // 当滚动到最上或者最下时就不会再滚动，这时移动布局
                if (isNeedMove()) {
                    // 初始化头部矩形
                    if (normal.isEmpty()) {
                        // 保存正常的布局位置
                        normal.set(inner.getLeft(), inner.getTop(),
                                inner.getRight(), inner.getBottom());
                    }
//				Log.e("jj", "矩形：" + inner.getLeft() + "," + inner.getTop()
//						+ "," + inner.getRight() + "," + inner.getBottom());
                    // 移动布局
                    inner.layout(inner.getLeft(), inner.getTop() - deltaY / 2,
                            inner.getRight(), inner.getBottom() - deltaY / 2);
                }
                isCount = true;
                break;

            default:
                break;
        }
    }

    /***
     * 回缩动画
     */
    public void animation() {
        // 开启移动动画
        TranslateAnimation ta = new TranslateAnimation(0, 0, inner.getTop(),
                normal.top);
        ta.setDuration(200);
        inner.startAnimation(ta);
        // 设置回到正常的布局位置
        inner.layout(normal.left, normal.top, normal.right, normal.bottom);

//		Log.e("jj", "回归：" + normal.left + "," + normal.top + "," + normal.right
//				+ "," + normal.bottom);

        normal.setEmpty();

    }

    // 是否需要开启动画
    public boolean isNeedAnimation() {
        return !normal.isEmpty();
    }

    /***
     * 是否需要移动布局 inner.getMeasuredHeight():获取的是控件的总高度
     * <p/>
     * getHeight()：获取的是屏幕的高度
     * <p/>
     * https://github.com/QQ986945193
     *
     * @return
     */
    public boolean isNeedMove() {
        int offset = inner.getMeasuredHeight() - getHeight();
        int scrollY = getScrollY();
//		Log.e("jj", "scrolly=" + scrollY);
        // 0是顶部，后面那个是底部
        if (scrollY == 0 || scrollY == offset) {
            return true;
        }
        return false;
    }
}

```

然后他的用法就是和ScrollView用法一样了。比如直接在布局中引用：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"

    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="davidbouncescrollview.qq986945193.com.davidbouncescrollview.MainActivity">

    <davidbouncescrollview.qq986945193.com.davidbouncescrollview.BounceScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />

            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="@android:color/black" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:text="大家好，我是程序员小冰" />
        </LinearLayout>

    </davidbouncescrollview.qq986945193.com.davidbouncescrollview.BounceScrollView>

</LinearLayout>

```

最后直接运行即可看到上面的效果。如果对您有帮助，欢迎star，fork。。。
