### RemoteViews

一个可以描述在另一个进程中显示的视图层次的类，继承Object。这个视图层次是从一个布局资源文件中填充的，这个类还提供了一些修改填充的视图层次的内容的基本操作。

#### RemoteViews被限制仅支持：

+ Layout
  + AdapterViewFlipper & ViewFlipper
    + 在添加的两个或多个视图之间动画的简单ViewAnimator（继承FrameLayout的可以在多个视图之间切换时执行动画的容器）。一次只显示一个子视图。如果需要，可以自动地按一定的时间间隔在每个子视图之间切换。
    + 两者区别
      + 使用ViewFlipper，通常会预先声明所有子项，并且没有回收概念。
      + 使用AdapterViewFlipper，可以Adapter像使用一样使用ListView，Spinner等等，这样可以即时确定子代，并且可以回收代表子代的视图。
      + 对于小的静态内容，ViewFlipper则更为简单。此外，它AdapterViewFlipper是在API级别11中添加的，因此不适用于Android上的旧版本。
  + FrameLayout
  + GridLayout
    + 一种将子元素放置在矩形网格中的布局。
  + GridView
    + 在二维滚动网格中显示项目的视图。网格中的项来自与此视图关联的ListAdapter。
  + LinearLayout
  + ListView
  + RelativeLayout
  + StackView
    + 也是AdapterViewAnimator的子类，也用于显示Adapter提供的系列View。SackView将会以“堆叠（Stack）”方式来显示多个列表项。
    + 为了控制StackView显示的View组件，StackView提供了如下两种控制方式。
      - 拖走StackView中处于顶端的View，下一个View将会显示出来。将上一个View拖进StackView，将使之显示出来。
      - 通过调用StackView的showNext()、showPrevious()控制显示上一个、下一个组件。
+ widget
  + AnalogClock
    + 表盘式的模拟时钟
    + 该类已在API级别23中弃用。
  + Button
  + Chronometer
    + 计时器
  + ImageButton
  + ImageView
  + ProgressBar
  + TextClock
    + 将当前日期和/或时间显示为格式化字符串显示
  + TextView
+ 不支持这些类的后代

#### 嵌套的类

+ **class** RemoteViews.ActionException

  + 当执行一个动作时出错抛出的异常

+ **class** RemoteViews.RemoteResponse

  + 表示对一个RemoteViews的任何元素执行的动作的响应的类

  + 方法：

    + ```java
      public RemoteResponse addSharedElement(int viewId, @NonNull String sharedElementName);
      /*
      	添加了一个共享元素，作为跨活动场景动画在活动之间转换的一部分。
      	第一个元素的位置将被用作出口转换的中心。	
      	相关共享元素在被启动的活动中的位置将是其进入过渡的中心位置。
      		@param viewId将作为转换的一部分共享视图的id
      		@param sharedElementName这个视图的共享元素名
      */
      
      public static RemoteResponse fromFillInIntent(@NonNull Intent fillIntent);
      /*
      	不建议在小部件中使用集合时（如ListView、StackView等）在各个item上设置PendingIntents，这样代价很大，但可以在集合上设置一个PendingIntent模板（RemoteViews.setPendingIntentTmplate(int, PedingIntent)）,并且可以通过在item上设置fillIntent来区分给定的item等各个单击动作。然后为了确定点击时最终哪一个intent会被执行，会将fillInIntent与PendingIntent模板组合在一起。
      	它的工作方式如下：在PendingIntent模板中保留为空但由fillIntent提供的任何字段都将会被重写，并会使用最总产生的PendingIntent。随后，设置在fillIntent中的关联字段会填充到PendingIntent模板的剩余部分中。（Intent.fillIn(Intent, int)）
      	创建一个发送一个挂起意图并且把这个intent作为其一部分的响应。
      	intent的源边界(Intent#getSourceBounds()) 将设置为屏幕空间中目标视图的边界。
      	请注意，与mPendingIntent关联的任何活动选项都可能在启动该意图之前被覆盖。
      		@param fillIntent 这个Intent将会与父母的PendingIntent结合在一起，以确定响应的行为。
      */
      
      public static RemoteResponse fromPendingIntent(@NonNull PendingIntent pendingIntent);
      /*
      	创建一个发送一个挂起意图并且把这个intent作为其一部分的响应。
        intent的源边界（Intent＃getSourceBounds（））将设置为屏幕空间中目标视图的边界。
        请注意，与mPendingIntent关联的任何活动选项都可能在启动该意图之前被覆盖。
        	@param pendingIntent 作为响应的一部分发送的PendingIntent
      */
      ```

+ **@interface** RemoteViews.RemoteView (Implements Annotation)

  + 此批注指示允许将View的子类与RemoteViews机制一起使用。
  + 需要实现Annotation接口的方法：
    + Class<? extends Annotation> annotationType();

#### 常量

+ String EXTRA_SHARED_ELEMENT_BOUNDS
  + 包含所有共享元素范围的Intent Extra。

#### 字段

+ public static final Creator&lt;RemoteViews&gt;
  + 实例化RemoteViews对象的Parcelable.Creator

#### 公有构造器

##### RemoteViews(String packageName, int layoutId)

```java
/**
* 创建一个将会展示特定布局文件内的视图的RemoteViews对象。
* @param packageName 包含布局资源的包名
* @param layoutId 布局资源的id
*/
public RemoteViews(String packageName, int layoutId) {
  this(getApplicationInfo(packageName, UserHandle.myUserId()), layoutId);
}

/**
* 创建一个新的RemoteViews对象，这个对象将会展示特定的布局文件内的视图
* @param application 一个应用程序，这个程序的内容是通过视图展示的。
* @param layoutId 布局资源的id
*/
protected RemoteViews(ApplicationInfo application, @LayoutRes int layoutId) {
  mApplication = application;
  mLayoutId = layoutId;
  mBitmapCache = new BitmapCache();
  mClassCookies = null;
}
```

##### RemoteViews(RemoteViews landscape, RemoteViews portrait)

```java
/*
* 创建一个新的RemoteViews对象，该对象将根据当前配置填充为指定的横向和纵向RemoteViews。
* @param landscape 横向布局的RemoteViews
* @param portrait 纵向布局的RemoteViews
*/
public RemoteViews(RemoteViews landscape, RemoteViews portrait) {
  if (landscape == null || portrait == null) {
    throw new RuntimeException("Both RemoteViews must be non-null");
  }
  if (!landscape.hasSameAppInfo(portrait.mApplication)) {
    throw new RuntimeException("Both RemoteViews must share the same package and user");
  }
  mApplication = portrait.mApplication;
  mLayoutId = portrait.mLayoutId;
  mLightBackgroundLayoutId = portrait.mLightBackgroundLayoutId;
  
  mLandscape = landscape;
  mPortrait = portrait;

  mBitmapCache = new BitmapCache();
  configureRemoteViewsAsChild(landscape);
  configureRemoteViewsAsChild(portrait);

  mClassCookies = (portrait.mClassCookies != null) ? portrait.mClassCookies : landscape.mClassCookies;
}
```

##### RemoteViews(RemoteViews src)

```java
/**
* 创建另一个RemoteViews的副本
*/
public RemoteViews(RemoteViews src) {
  mBitmapCache = src.mBitmapCache;
  mApplication = src.mApplication;
  mIsRoot = src.mIsRoot;
  mLayoutId = src.mLayoutId;
  mLightBackgroundLayoutId = src.mLightBackgroundLayoutId;
  mApplyFlags = src.mApplyFlags;
  mClassCookies = src.mClassCookies;
  
  if (src.hasLandscapeAndPortraitLayouts()) {
    mLandscape = new RemoteViews(src.mLandscape);
    mPortrait = new RemoteViews(src.mPortrait);
  }
  
  if (src.mActions != null) {
    Parcel p = Parcel.obtain();
    p.putClassCookies(mClassCookies);
    src.writeActionsToParcel(p);
    p.setDataPosition(0);
    // Since src is already in memory, we do not care about stack overflow as it has
    // already been read once.
    readActionsFromParcel(p, 0);
    p.recycle();
  }
  
  // Now that everything is initialized and duplicated, setting a new BitmapCache will re-initialize the cache.
  setBitmapCache(new BitmapCache());
}
```

##### RemoteViews(Parcel parcel)

```java
/*
* 从parcel中读取RemoteViews
* @param parcel
*/
public RemoteViews(Parcel parcel) {
  this(parcel, null, null, 0, null);
}

private RemoteViews(Parcel parcel, BitmapCache bitmapCache, ApplicationInfo info, int depth, Map<Class, Object> classCookies) {
  if (depth > MAX_NESTED_VIEWS && (UserHandle.getAppId(Binder.getCallingUid()) != Process.SYSTEM_UID)) {
    throw new IllegalArgumentException("Too many nested views.");
  }
  depth++;
  
  int mode = parcel.readInt();
  
  // 只在根RemoteViews中存贮bitmap缓存
  if (bitmapCache == null) {
    mBitmapCache = new BitmapCache(parcel);
    // 存储类cookies以便于我们克隆这个RemoteViews的时候它们是可用的
    mClassCookies = parcel.copyClassCookies();
  } else {
    setBitmapCache(bitmapCache);
    mClassCookies = classCookies;
    setNotRoot();
  }
  
  if (mode == MODE_NORMAL) {
    mApplication = parcel.readInt() == 0 ? info : ApplicationInfo.CREATOR.createFromParcel(parcel);
    mLayoutId = parcel.readInt();
    mLightBackgroundLayoutId = parcel.readInt();
    
    readActionsFromParcel(parcel, depth);
  } else {
    // MODE_HAS_LANDSCAPE_AND_PORTRAIT
    mLandscape = new RemoteViews(parcel, mBitmapCache, info, depth, mClassCookies);
    mPortrait = new RemoteViews(parcel, mBitmapCache, mLandscape.mApplication, depth, mClassCookies);
    mApplication = mPortrait.mApplication;
    mLayoutId = mPortrait.mLayoutId;
    mLightBackgroundLayoutId = mPortrait.mLightBackgroundLayoutId;
  }
  mApplyFlags = parcel.readInt();
}
```

#### 公有方法

##### public void addView(int viewId, RemoteViews nestedView)

等同于填充给定的RemoteViews之后调用ViewGroup.addView(View)。 这使用户可以构建“嵌套的” RemoteView。 如果RemoteViews的使用者可以回收布局，请使用removeAllViews(int)清除所有现有子项。

```java
/*
* @param viewId 要添加子视图的父视图的id
* @param nestedView 描述子视图的RemoteViews
*/
public void addView(int viewId, RemoteViews nestedView) {
  addAction(nestedView == null
            ? new ViewGroupActionRemove(viewId)
            : new ViewGroupActionAdd(viewId, nestedView));
}

```

##### public Views apply(Context context, ViewGroup parent)

扩展此对象表示的视图层次结构并应用所有操作。

```java
/**
* Caller beware: this may throw
* @param context Default context to use
* @param parent 生成的视图层次结构将附加到的父级。此方法不附加层次结构。调用者应在适当的时候这样做。
* @return 填充的视图层次结构
*/
public View apply(Context context, ViewGroup parent) {
  return apply(context, parent, null);
}
/** @hide */
public View apply(Context context, ViewGroup parent, OnClickHandler handler) {
  RemoteViews rvToApply = getRemoteViewsToApply(context);
  
  View result = inflateView(context, rvToApply, parent);
  rvToApply.performApply(result, parent, handler);
  return result;
}
```

##### public int describeCntents()

描述此Parcelable实例的编组表示中包含的特殊对象的种类。 例如，如果对象将在writeToParcel(android.os.Parcel, int)的输出中包括文件描述符，则此方法的返回值必须包含CONTENTS_FILE_DESCRIPTOR位。

```java
public int describeContents() {
  return 0;
}
```

##### public int getLayoutId()

返回与此RemoteViews关联的根布局的布局ID。 如果RemoteViews同时具有横向和纵向根，则将返回与纵向布局关联的布局ID。

```java
public int getLayoutId() {
  return hasFlags(FLAG_USE_LIGHT_BACKGROUND_LAYOUT) && (mLightBackgroundLayoutId != 0) ? mLightBackgroundLayoutId : mLayoutId; 
}
```

##### public String getPackage()

```java
public String getPackage() {
  return (mApplication != null) ? mApplication.packageName : null;
}
```

##### public void reapply(Context context, View v)

将所有操作应用于提供的视图。

```java
/*
* @param v 要对其执行操作的视图。 这应该是apply(android.content.Context, android.view.ViewGroup)调用的结果。
*/
public void reapply(Context context, View v) {
  reapply(context, v, null);
}

/** @hide */
public void reapply(Context context, View v, OnClickHandler handler) {
  RemoteViews rvToApply = getRemoteViewsToApply(context);
  // 如果视图在一个方向上应用了此RemoteViews，并且在方向变化中持久存在，并且在新的方向上重新应用了RemoteViews，则我们将抛出一个例外，因为布局可能是完全不相关的。
  if (hasLandscapeAndPortraitLayouts()) {
    if ((Integer) v.getTag(R.id.widget_frame) != rvToApply.getLayoutId()) {
      throw new RuntimeException("Attempting to re-apply RemoteViews to a view that" + " that does not share the same root layout id.");
    }
  }
  rvToApply.performApply(v, (ViewGroup) v.getParent(), handler);
}
```

##### public void remoeAllViews(int viewId)

相当于调用了ViewGroup#removeAllViews()

```java
/**
* @param viewId 要从中移除所有子级的父级ViewGroup的ID。
*/
public void removeAllViews(int viewId) {
  addAction(new ViewGroupActionRemove(viewId));
}
```

##### public void setAccessibilityTraversalAfter(int viewId, int nextId)

相当于调用View.setAccessibilityTraversalAfter(int)

```java
/*
* @param viewId 要设置其可访问性遍历后视图的视图的ID。
* @param nextId 可访问性遍历中的下一个ID。
*/
public void setAccessibilityTraversalAfter(int viewId, int nextId) {
  setInt(viewId, "setAccessibilityTraversalAfter", nextId);
}
```

##### public void setAccessibilityTraversalBefore()

相当于调用View.setAccessibilityTraversalBefore(int)

```java
/*
* @param viewId 要设置其可访问性遍历后视图的视图的ID。
* @param nextId 可访问性遍历中的下一个ID。
*/
public void setAccessibilityTraversalBefore(int viewId, int nextId) {
  setInt(viewId, "setAccessibilityTraversalBefore", nextId);
}
```

##### public void setBitmap(int viewId, String methodName, Bitmap value)

调用此方法在此RemoteView的布局中的视图上获取一个Bitmap。

```java
/**
* @param viewId 要在其上调用方法的视图的ID。
* @param methodName 调用方法的名称。
* @param value 传递给方法的值。
*/
public void setBitmap(int viewId, String methodName, Bitmap value) {
  addAction(new BitmapReflectionAction(viewId, methodName, value));
}

```

##### public void setBoolean(int viewId, String methodName, boolean value)

调用此方法在此RemoteView的布局中的视图上采用一个布尔值。

```java
/**
* @param viewId 要在其上调用方法的视图的ID。
* @param methodName 调用方法的名称。
* @param value 传递给方法的值。
*/
public void setBoolean(int viewId, String methodName, boolean value) {
  addAction(new ReflectionAction(viewId, methodName, ReflectionAction.BOOLEAN, value));
}
```

##### public void setBundle(int viewId, String methodName, boolean value)

调用此方法在此RemoteView的布局中的视图上获取一个Bundle。

```java
/**
* @param viewId 要在其上调用方法的视图的ID。
* @param methodName 调用方法的名称。
* @param value 传递给方法的值。
*/
public void setBundle(int viewId, String methodName, Bundle value) { 
  addAction(new ReflectionAction(viewId, methodName, ReflectionAction.BUNDLE, value));
}
```

##### public void setByte(int viewId, String methodName, boolean value)

调用此方法在此RemoteView的布局中的视图上获取一个Byte。

```java
/**
* @param viewId 要在其上调用方法的视图的ID。
* @param methodName 调用方法的名称。
* @param value 传递给方法的值。
*/
public void setByte(int viewId, String methodName, byte value) {
  addAction(new ReflectionAction(viewId, methodName, ReflectionAction.BYTE, value));
}
```

