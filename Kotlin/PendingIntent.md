### PendingIntent

> PendingIntent可以看作是对Intent的一个封装，但它不是立刻执行某个行为，而是满足某些条件或触发某些事件后才执行指定的行为。

对要执行的Intent和目标操作的描述。这个类的实例通过👇的方法创建：

> `getActivity(Context, int, Intent, int)`, 从系统取得一个用于启动一个Activity的PendingIntent对象
> `getActivities(Context, int, Intent[], int)`
>  `getBroadcast(Context, int, Intent, int)`从系统取得一个用于向Broadcast的发送广播的PendingIntent对象.
> `getService(Context, int, Intent, int)`从系统取得一个 用于启动一个Service的PendingIntent对象

返回的对象可以传递给其他的applications，以便于它们后续可以执行我们所描述的操作。

PendingIntent 的使用场景有三个：

- 使用 AlarmManager 设定闹钟
- 在系统状态栏显示 Notification
- 在桌面显示 Widget

PendingIntent执行的操作实质上是参数传进来的Intent的操作，但是使用PendingIntent的目的在于它所包含的Intent的操作的执行是需要满足某些条件的。

通过给另一个应用程序一个PendingIntent，我们就授予了它执行你指定操作的权限，就好比另一个应用程序就是我们自己一样（拥有相同的权限和身份）所以我们要注意如何构建PendingIntent：例如，往往我们提供的基础Intent应该将组件名称显式设置为我们自己的组件之一，以确保将它发送到那里。

一个PendingIntent本身简单来讲就是一个系统维护的token的引用，这个token描述了用于检索它的原始数据。这意味着，即使其拥有的应用程序的进程被杀死了，PendingIntent本身对在已赋予它的其他进程中依然是可用的。

可以调用cancel( )将PendingIntnet删除。

如果创建的应用程序以后重新检索相同类型的PendingIntent（相同的操作，相同的Intent操作，数据，类别和组件以及相同的标志），它将收到表示相同token的PendingIntent（如果该token仍然有效）。因此，为了检索一个PendingIntent，知道何时将两个Intent视为相同的是很重要的。人们普遍犯的一个错误就是，利用多个仅在“extra”内容中有所不同的Intents，创建了多个PendingIntent的对象，以期望每次都获得一个不同的PendingIntent。这是不可能发生的。用于匹配的Intent的部分与 Intent#filterEquals(Intent)判断是否相同的部分相同。 如果使用在Intent#filterEquals(Intent) 等效的两个Intent对象，则它们将获得相同的PendingIntent。

```java
public boolean filterEquals(Intent other) {
  if (other == null) {
    return false;
  }
  if (!Objects.equals(this.mAction, other.mAction)) 
    return false;
  if (!Objects.equals(this.mData, other.mData)) 
    return false;
  if (!Objects.equals(this.mType, other.mType)) 
    return false;
  if (!Objects.equals(this.mIdentifier, other.mIdentifier)) 
    return false;
  if (!Objects.equals(this.mPackage, other.mPackage)) 
    return false;
  if (!Objects.equals(this.mComponent, other.mComponent)) 
    return false;
  if (!Objects.equals(this.mCategories, other.mCategories)) 
    return false;
  return true;
}
```

有两种典型的方法可以解决这个：

如果真的需要同时激活多个不同的PendingIntent对象（例如用作两个同时显示的通知），那么需要确保它们有所不同，以便将它们与不同的PendingIntent关联。

+ 可能是Intent.fileterEquals(Intent)考虑的任意Intent属性，

+ 提供给`getActivity(),getActivities(),getBroadcast(),getService()` 的requestCode不同。

  + ```java
    public static PendingIntent getActivity(Context context, int requestCode, @NonNull Intent intent, @Flags int flags, @Nullable Bundle options);
    public static PendingIntent getActivity(Context context, int requestCode, Intent intent, @Flags int flags);
    public static PendingIntent getActivities(Context context, int requestCode, @NonNull Intent[] intents, @Flags int flags);
    public static PendingIntent getActivities(Context context, int requestCode, @NonNull Intent[] intents, @Flags int flags, @Nullable Bundle options);
    public static PendingIntent getBroadcast(Context context, int requestCode, Intent intent, @Flags int flags);
    public static PendingIntent getService(Context context, int requestCode, @NonNull Intent intent, @Flags int flags)
    ```

如果一次只需要为一个将要使用的Intent激活一个PendingIntent，则可以选择使用标志`FLAG_CANCEL_CURRENT`或者`FLAG_UPDATE_CURRENT`来取消或修改与您提供的Intent相关的任何当前的PendingIntent。

还要注意，诸如`FLAG_ONE_SHOT`或者`FLAG_IMMUTABLE`之类的标志描述了PendingIntent实例，因此被用来去标识它。检索或者修改使用这些标志创建的PendingInent的调用也需要这些标志与其他标志一起提供。例如：为了检索一个利用`FLAG_ONE_SHOT`标志创建的已存在的PendingIntent，同时需要提供`FLAG_ONE_SHOT`和`FLAG_NO_CREATE`。

#### Summary

##### 嵌套的classes

+ (class) PendingIntent.CanceledException
  + 尝试通过已被取消或无法再执行请求的PendingIntent发送时抛出异常
+ (interface) PendingIntent.OnFinished
  + 用于发现当一个发送操作已经完成的回调借口，主要与正在执行广播的PendingIntent一起使用，它提供的信息与通过最终的BroadcastReceiver调用Context.sendOrderedBroadcast(Intent, String, android.content.BroadcastReceiver, Handler, int, String, Bundle)相同。
  + Summary
    + abstract void onSendFinished(PendingIntent pendingIntent, Intent intent, int resultCode, String resultData, Bundle resultExtras)
      + 当一个发送操作完成时调用
      + Parameters
        + pendingIntent: 通过该操作发送的PendingIntent
        + intent: 发送的最初的Intent
        + resultCode: 由发送决定的最后结果代码
        + resultData: 通过广播收集的最终数据
        + resultExtras: 通过广播收集的最终extras

##### 常量

+ (int) FLAG_CANCEL_CURRENT
  + 用来表示描述的PendingIntent是否已经存在，当前的一个应该在生成一个新的之前被取消。为了和`getActivity(Context, int, Intent, int)`，`getBroadcast(Context, int, Intent, int)`，和 `getService(Context, int, Intent, int)`一起使用
  + 可以使用这个去检索一个新的PendingIntent当你只改变了Inent的Extra data；通过取消先前的pending intent，可以确保只有获得了新数据的实体才能启动它。如果这个断言不是一个问题，考虑使用`FLAG_UPDATE_CURRENT`。
+ (int) FLAG_IMMUTABLE
  + 用来表示创建的PendingIntent应是不变的。这意味着传递给send方法以填充此intent的为填充属性的其他intent参数将被忽略。
+ (int) FLAG_NO_CREATE
  + 指示如果所描述的PendingIntent还不存在，则仅返回null而不创建它。为了和`getActivity(Context, int, Intent, int)`，`getBroadcast(Context, int, Intent, int)`，和 `getService(Context, int, Intent, int)`一起使用
+ (int) FLAG_ONE_SHOT
  + 指示这个PendingIntent只可以被用一次。为了和`getActivity(Context, int, Intent, int)`，`getBroadcast(Context, int, Intent, int)`，和 `getService(Context, int, Intent, int)`一起使用。
  + 如果设置，则在调用了`send()`后，它将自动地取消，并且以后尝试通过它发送的任何尝试都会失败。
+ (int) FLAG_UPDATE_CURRENT
  + 指示如果所描述的PendingIntent已经存在，则保留它但用新Intent中的内容替换其extra data。

##### 继承的常量

+ from interface **android.os.Parcelable**

##### 字段

+ public static final Creator&lt;PendingIntent&gt;

##### 公有方法

###### public void cancel()

取消当前活动的PendingIntent。 只有拥有PendingIntent的原始应用程序才能取消它。

```java
public void cancel() {
  try {
    ActivityManager.getService().cancelIntentSender(mTarget);
  } catch (RemoteException e) {
  }
}
```

##### public int describeContents()

描述此Parcelable实例的编组表示中包含的特殊对象的种类。例如，如果对象将在writeToParcel(Parcel, int)的输出中包括文件描述符，则此方法的返回值必须包含CONTENTS_FILE_DESCRIPTOR位。

```java
/**
	@return 一个位掩码，指示此Parcelable对象实例编组的一组特殊对象类型。值为0或CONTENTS_FILE_DESCRIPTOR
*/
public int describeContents() {
  return 0;
}
```

##### public boolean equals(Object otherObj)

两个PendingIntent对象上的比较运算符，例如返回true，则它们都表示来自同一包的相同操作。 这样，您可以多次使用`getActivity(Context，int，Intent，int)，getBroadcast(Context，int，Intent，int)或getService(Context，int，Intent，int)`多次（甚至跨进程被杀死），结果 在不同的PendingIntent对象中，但是其equals()方法将它们标识为相同的操作。

```java
/**
	@param otherObj 要比较多另一个对象
	@return true如果这个对象等于obj参数；false反之
*/
@Override
public boolean equals(Object otherObj) {
  if (otherObj instanceof android.app.PendingIntent) {
    return  mTarget.asBinder().equals(((android.app.PendingIntent)otherObj).mTarget.asBinder());
  }
  return false;
}
```

##### public static PendingIntent getActivities(Context context, int requestCode, Intent[] intents, int flags, Bundle options)

类似于getActivity(android.content.Context, int, android.content.Intent, int)，但允许提供一个Intent数组。 数组中的最后一个Intent被用作PendingIntent的主键，就像赋予getActivity(android.content.Context, int, android.content.Intent, int)的单个Intent一样。 发送结果PendingIntent后，将所有Intent以与将它们传递给Context＃startActivities(Intent[])相同的方式启动。

> 数组中的第一个Intent将在现有活动的上下文之外启动，因此您必须在Intent中使用Intent＃FLAG_ACTIVITY_NEW_TASK启动标志。 （数组中第一个活动之后的活动是在数组中上一个活动的上下文中启动的，因此不需要FLAG_ACTIVITY_NEW_TASK。）

> 数组中的最后一个intent表示PendingIntent的键。 换句话说，它是匹配的重要元素（就像对getActivity(android.content.Context, int, android.content.Intent, int)提供的单个intent一样，其内容将由send(android.content.Context, int, android.content.Intent)和FLAG_UPDATE_CURRENT等替换。这是因为它是所提供的意图中最具体的，也是用户在意图启动时实际看到的UI。

> 出于安全原因，在此处提供的Intent对象几乎应始终是显式intent，即指定要通过Intent＃setClass(android.content.Context，Class)传递给的显式组件。

```java
/**
* @param context 这个PendingIntent要开启acitivity的Context
* @param requestCode 给发送者的私有请求码
* @param intents 要启动的acitivities的Intent的数组
* @param flags 可能是FLAG_ONE_SHOT，FLAG_NO_CREATE， FLAG_CANCEL_CURRENT， FLAG_UPDATE_CURRENT， FLAG_IMMUTABLE 或者Intent.fillIn()支持的任何标志，以控制实际发送发生时可以提供的哪些未指定的Intent部分。
* @return 返回与给定参数匹配的现有或新的PendingIntent。仅当提供了FLAG_NO_CREATE时才可以返回null。
*/
public static PendingIntent getActivities(Context context, int requestCode, @NonNull Intent[] intents, @Flags int flags, @Nullable Bundle options) {
  String packageName = context.getPackageName();
  String[] resolvedTypes = new String[intents.length];
  for (int i=0; i<intents.length; i++) {
    intents[i].migrateExtraStreamToClipData();
    intents[i].prepareToLeaveProcess(context);
    resolvedTypes[i] =  intents[i].resolveTypeIfNeeded(context.getContentResolver());
  }
  try {
    IIntentSender target = ActivityManager.getService().getIntentSender(ActivityManager.INTENT_SENDER_ACTIVITY, packageName, null, null, requestCode, intents, resolvedTypes, flags, options, context.getUserId());
    return target != null ? new PendingIntent(target) : null;
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

##### public static PendingIntent getActivities (Context context, int requestCode, Intent[] intents, int flags)

类似于getActivity(android.content.Context, int, android.content.Intent, int)，但允许提供一个Intent数组。 数组中的最后一个Intent被用作PendingIntent的主键，就像赋予getActivity(android.content.Context, int, android.content.Intent, int)的单个Intent一样。 发送结果PendingIntent后，将所有Intent以与将它们传递给Context＃startActivities(Intent[])相同的方式启动。

```java
public static android.app.PendingIntent getActivities(Context context, int requestCode, @NonNull Intent[] intents, @Flags int flags) {
  return getActivities(context, requestCode, intents, flags, null);
}
```

##### public static PendingIntent getActivity(Context context, int requestCode, Intent intent, int flags)

检索将启动新活动的PendingIntent，例如调用Context＃startActivity(Intent)。 请注意，该活动将在现有活动的上下文之外启动，因此您必须在Intent中使用Intent＃FLAG_ACTIVITY_NEW_TASK启动标志。

> 出于安全原因，您在此处提供的Intent几乎应该始终是显式Intent，即指定要通过Intent＃setClass(android.content.Context, Class)传递给的显式组件。

```java
public static android.app.PendingIntent getActivity(Context context, int requestCode, Intent intent, @Flags int flags) {
  return getActivity(context, requestCode, intent, flags, null);
}
```

##### public static PendingIntent getActivity(Context context, int requestCode, Intent intent, int flags, Bundle options)

检索一个可以打开一个新Activity的PendingIntent，就像Context.startActivity(Intent intent)那样。要注意这个活动将会开启在一个已经存在的activity的环境之外，所以我们一定要在Intent中使用**Intent.FLAG_ACTIVITY_NEW_TASK**的启动标志。

>出于安全原因，我们在此处提供的Intent应该基本上都是一个显式的intent，即指定一个显式组建通过Intent.setClass(android.content.class)传递。

```java
/*
* @param context 这个PendingIntent开启Activity的环境
* @param requestCode 给发送者的私有的请求码
* @param intent 要启动的activity的Intent
* @param flags 可能是FLAG_ONE_SHOT, FLAG_NO_CREATE_CURRENT, FLAG_UPDATE_CURRENT,或者Intent#fillIn支持的任何标志，用于控制在实际发送发生时可以提供Intent的未指定部分。
* @param options 有关如何启动activity的其他选项，如果没有可能是null
* 
* @returns 返回一个已经存在的或者一个新的和所给参数相匹配的PendingIntent，只有当FLAG_NO_CREATE标志提供时可能会返回null
*/

public static android.app.PendingIntent getActivity(Context context, int requestCode, @NonNull Intent intent, @Flags int flags, @Nullable Bundle options) {
  String packageName = context.getPackageName();
  String resolvedType = intent != null ? intent.resolveTypeIfNeeded(context.getContentResolver()) : null;
  try {
    intent.migrateExtraStreamToClipData();
    intent.prepareToLeaveProcess(context);
    IIntentSender target = ActivityManager.getService().getIntentSender(ActivityManager.INTENT_SENDER_ACTIVITY, packageName, null, null, requestCode, new Intent[] { intent }, resolvedType != null ? new String[] { resolvedType } : null, flags, options, context.getUserId());
    return target != null ? new android.app.PendingIntent(target) : null;
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

```java
//将other的内容复制到此对象中，但仅复制未由此对象定义的字段。
@FillInFlags
public int fillIn(@NonNull Intent other, @FillInFlags int flags) {
  int changes = 0;
  boolean mayHaveCopiedUris = false;
  if (other.mAction != null && (mAction == null || (flags&FILL_IN_ACTION) != 0)) {
    mAction = other.mAction;
    changes |= FILL_IN_ACTION;
  }
  if ((other.mData != null || other.mType != null) && ((mData == null && mType == null) || (flags&FILL_IN_DATA) != 0)) {
    mData = other.mData;
    mType = other.mType;
    changes |= FILL_IN_DATA;
    mayHaveCopiedUris = true;
  }
  if (other.mIdentifier != null && (mIdentifier == null || (flags&FILL_IN_IDENTIFIER) != 0)) {
    mIdentifier = other.mIdentifier;
    changes |= FILL_IN_IDENTIFIER;
  }
  if (other.mCategories != null && (mCategories == null || (flags&FILL_IN_CATEGORIES) != 0)) {
    if (other.mCategories != null) {
      mCategories = new ArraySet<String>(other.mCategories);
    }
    changes |= FILL_IN_CATEGORIES;
  }
  if (other.mPackage != null && (mPackage == null || (flags&FILL_IN_PACKAGE) != 0)) {
    // Only do this if mSelector is not set.
    if (mSelector == null) {
      mPackage = other.mPackage;
      changes |= FILL_IN_PACKAGE;
    }
        }
        // Selector is special: it can only be set if explicitly allowed,
        // for the same reason as the component name.
        if (other.mSelector != null && (flags&FILL_IN_SELECTOR) != 0) {
            if (mPackage == null) {
                mSelector = new Intent(other.mSelector);
                mPackage = null;
                changes |= FILL_IN_SELECTOR;
            }
        }
        if (other.mClipData != null
                && (mClipData == null || (flags&FILL_IN_CLIP_DATA) != 0)) {
            mClipData = other.mClipData;
            changes |= FILL_IN_CLIP_DATA;
            mayHaveCopiedUris = true;
        }
        // Component is special: it can -only- be set if explicitly allowed,
        // since otherwise the sender could force the intent somewhere the
        // originator didn't intend.
        if (other.mComponent != null && (flags&FILL_IN_COMPONENT) != 0) {
            mComponent = other.mComponent;
            changes |= FILL_IN_COMPONENT;
        }
        mFlags |= other.mFlags;
        if (other.mSourceBounds != null
                && (mSourceBounds == null || (flags&FILL_IN_SOURCE_BOUNDS) != 0)) {
            mSourceBounds = new Rect(other.mSourceBounds);
            changes |= FILL_IN_SOURCE_BOUNDS;
        }
        if (mExtras == null) {
            if (other.mExtras != null) {
                mExtras = new Bundle(other.mExtras);
                mayHaveCopiedUris = true;
            }
        } else if (other.mExtras != null) {
            try {
                Bundle newb = new Bundle(other.mExtras);
                newb.putAll(mExtras);
                mExtras = newb;
                mayHaveCopiedUris = true;
            } catch (RuntimeException e) {
                // Modifying the extras can cause us to unparcel the contents
                // of the bundle, and if we do this in the system process that
                // may fail.  We really should handle this (i.e., the Bundle
                // impl shouldn't be on top of a plain map), but for now just
                // ignore it and keep the original contents. :(
                Log.w(TAG, "Failure filling in extras", e);
            }
        }
        if (mayHaveCopiedUris && mContentUserHint == UserHandle.USER_CURRENT
                && other.mContentUserHint != UserHandle.USER_CURRENT) {
            mContentUserHint = other.mContentUserHint;
        }
        return changes;
    }
```

##### public static PendingIntent getBroadcast(Context context, int requestCode, Intent intent, int flags)

检索一个将会执行广播的PendingIntent，就像Context.sendBroadcast(Intent)一样。

```java
/*
* @param context PendingIntent要执行广播的环境。
* @param requestCode 给sender的私有请求码
* @param intent 要广播的Intent
* @param flags 可能是FLAG_ONE_SHOT，FLAG_NO_CREATE，FLAG_CANCEL_CURRENT，FLAG_UPDATE_CURRENT，FLAG_IMMUTABLE或Intent＃fillIn支持的任何标志，以控制实际发送发生时可以提供的哪些未指定的Intent部分。
* @return 返回与给定参数匹配的现有或新的PendingIntent。 仅当提供了FLAG_NO_CREATE时才可以返回null。
*/
public static android.app.PendingIntent getBroadcast(Context context, int requestCode, Intent intent, @Flags int flags) {
  return getBroadcastAsUser(context, requestCode, intent, flags, context.getUser());
}
```

##### public String getCreatorPackage()

返回创建此PendingIntent的应用程序的程序包名称，即您将实际发送该Intent的标识。 返回的字符串由系统提供，因此应用程序无法欺骗其程序包。

>请谨慎使用。 这一切告诉您是谁创建了PendingIntent。 它不会告诉您是谁将PendingIntent交给您的：也就是说，PendingIntent对象旨在在应用程序之间传递，因此您从一个应用程序收到的PendingIntent实际上可能是它从另一个应用程序收到的，这意味着您在此处得到的结果将是 标识原始应用程序。 因此，您仅应使用此信息来标识您希望通过send()调用与谁进行交互，而不是由谁为您提供PendingIntent。

```java
/*
* @return PendingIntent的包名，或者如果没有关联会返回null
*/
@Nullable
public String getCreatorPackage() {
  try {
    return ActivityManager.getService().getPackageForIntentSender(mTarget);
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

##### public int getCreatorUid()

返回创建此PendingIntent的应用程序的uid，即您实际将在其下发送Intent的标识。 返回的整数由系统提供，因此应用程序无法发送其uid。

```java
/*
* @return PendingIntent的uid，或者如果没有关联的就会返回-1
*/
public int getCreatorUid() {
  try {
    return ActivityManager.getService().getUidForIntentSender(mTarget);
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

##### public UserHandle getCreatorUserHandle()

返回创建此PendingIntent的应用程序的用户句柄，即您将实际在其下发送Intent的用户。 返回的UserHandle由系统提供，因此应用程序无法欺骗其用户。 有关用户句柄的更多说明，请参见Process.myUserHandle()。

```java
/*
* @return PendingIntent的用户句柄，或者如果没有关联的就会返回null
*/
@Nullable
public UserHandle getCreatorUserHandle() {
  try {
    int uid = ActivityManager.getService().getUidForIntentSender(mTarget);
    return uid > 0 ? new UserHandle(UserHandle.getUserId(uid)) : null;
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

##### public static PendingIntent getForegroundService(Context context, int requestCode, Intent intent, int flags)

检索将启动前台服务的PendingIntent，例如调用Context.startForegroundService。 提供给服务的起始参数将来自Intent的其他内容。

```java
/*
* @param context 此PendingIntent应该在其中启动服务的Context。
* @param requestCode 发送者的私有请求码
* @param intent 描述要启动的服务的Intent。 该值不能为空。
* @param flags 可以是FLAG_ONE_SHOT，FLAG_NO_CREATE，FLAG_CANCEL_CURRENT，FLAG_UPDATE_CURRENT，FLAG_IMMUTABLE或Intent＃fillIn支持的任何标志，以控制实际发送发生时可以提供的哪些未指定的Intent部分。
* @return 返回与给定参数匹配的现有或新的PendingIntent。 仅当提供了FLAG_NO_CREATE时才可以返回null。
*/
public static PendingIntent getForegroundService(Context context, int requestCode, @NonNull Intent intent, @Flags int flags) {
  return buildServicePendingIntent(context, requestCode, intent, flags, ActivityManager.INTENT_SENDER_FOREGROUND_SERVICE);
}
```

##### public IntentSender getIntentSender()

检索包装PendingIntent的现有发送方的IntentSender对象

```java
/* 
* @return 返回包装PendingIntent的发送者的IntentSender对象
*/
public IntentSender getIntentSender() {
  return new IntentSender(mTarget, mWhitelistToken);
}
```

##### public static PendingIntent getService(Context context, int requestCode, Intent intent, int flags)

检索将启动服务的PendingIntent，例如调用Context.startService。 提供给服务的起始参数将来自Intent的其他内容。

```java
/*
* @param context 此PendingIntent应该在其中启动服务的context。
* @param requestCode 发送者的私有请求码
* @param intent 描述要启动的服务的Intent。 该值不能为空。
* @param flags 可以是FLAG_ONE_SHOT，FLAG_NO_CREATE，FLAG_CANCEL_CURRENT，FLAG_UPDATE_CURRENT，FLAG_IMMUTABLE或Intent＃fillIn支持的任何标志，以控制实际发送发生时可以提供的哪些未指定的Intent部分。
* @return 返回与给定参数匹配的现有或新的PendingIntent。 仅当提供了FLAG_NO_CREATE时才可以返回null。
*/
public static PendingIntent getService(Context context, int requestCode, @NonNull Intent intent, @Flags int flags) {
  return buildServicePendingIntent(context, requestCode, intent, flags, ActivityManager.INTENT_SENDER_SERVICE);
}
```

##### public static PendingIntent readPendingIntentOrNullFromParcel (Parcel in)

用于从一个Parcel中读取PendingIntent或空指针的便利功能。 必须事先使用writePendingIntentOrNullToParcel(PendingIntent, Parcel)编写PendingIntent。

```java
/*
* @param in 包含写入PendingIntent的Parcel
* @return 从Parcel读取PendingIntent返回，如果已经写入null则返回null
*/
@Nullable
public static PendingIntent readPendingIntentOrNullFromParcel(@NonNull Parcel in) {
  IBinder b = in.readStrongBinder();
  return b != null ? new PendingIntent(b, in.getClassCookie(PendingIntent.class)) : null;
}
```

##### public void send(Context context, int code, Intent intent, PendingIntent.OnFinished onFinished, Handler handler, String requiredPermission, Bundle options)

执行与此PendingIntent关联的操作，允许调用方指定有关要使用的Intent的信息，并在发送完成时得到通知。

对于intent参数，PendingIntent通常会限制在此处提供哪些字段，根据如何在getActivity(Context, int, Intent, int)，getBroadcast(Context, int, Intent, int)或getService(Context, int, Intent, int)中检索PendingIntent的方式。

```java
/*
* @param context 调用者的Context。 如果intent也为null，则可以为null。
* @param code 返回代码提供回PendingIntent的目标。
* @param intent 其他Intent数据。有关如何将其应用于原始Intent的信息，请参见Intent.fillIn。使用null不会修改原始Intent。如果在创建此PendingIntent时设置了标志FLAG_IMMUTABLE，则将忽略此参数。该值可以为空。
* @param onFinished 发送完成后要回调的对象；如果没有回调，则返回null。
* @param handler 标识应该发生回调的线程的Handler。如果为null，则回调将从进程的线程池发生。
* @param requiredPermission PendingIntent的接收者必须持有的许可名称。这仅对广播Intent有效，并且对应于Context.sendBroadcast(Intent, String)中的权限参数。如果为null，则不需要权限。
* @param options 调用者想提供的可以修改发送行为的其他选项。可以从ActivityOptions构建，以应用于活动开始。
* @throws 如果PendingIntent不再允许通过它发送更多的Intent，则抛出CanceledException。
*/
public void send(Context context, int code, @Nullable Intent intent, @Nullable OnFinished onFinished, @Nullable Handler handler, @Nullable String requiredPermission, @Nullable Bundle options) throws CanceledException {
  if (sendAndReturnResult(context, code, intent, onFinished, handler, requiredPermission, options) < 0) {
    throw new CanceledException();
  }
}
```

##### public void send()

执行与此PendingIntent相关的操作。

```java
public void send() throws CanceledException {
  send(null, 0, null, null, null, null, null);
}
```

##### public void send (Context context, int code, Intent intent, PendingIntent.OnFinished onFinished, Handler handler)

```java
public void send(Context context, int code, @Nullable Intent intent, @Nullable OnFinished onFinished, @Nullable Handler handler) throws CanceledException {
  send(context, code, intent, onFinished, handler, null, null);
}
```

##### public void send(Context context, int code, Intent intent, PendingIntent.OnFinished onFinished, Handler handler, String requiredPermission)

```java
public void send(Context context, int code, @Nullable Intent intent, @Nullable OnFinished onFinished, @Nullable Handler handler, @Nullable String requiredPermission) throws CanceledException {
  send(context, code, intent, onFinished, handler, requiredPermission, null);
}
```

##### public void send(int code, PendingIntent.OnFinished onFinished, Handler handler)

执行与此PendingIntent关联的操作，以便在发送完成后通知调用者。

```java
public void send(int code, @Nullable OnFinished onFinished, @Nullable Handler handler) throws CanceledException {
  send(null, code, null, onFinished, handler, null, null);
}
```

##### public void send(Context context, int code, Intent intent)

执行与此PendingIntent关联的操作，允许调用方指定有关要使用的Intent的信息。

```java
public void send(Context context, int code, @Nullable Intent intent) throws CanceledException {
  send(context, code, intent, null, null, null, null);
}
```

##### public void send(int code)

执行与此PendingIntent相关的操作。

```java
public void send(int code) throws CanceledException {
  send(null, code, null, null, null, null, null);
}
```

##### public static void writePendingIntentOrNullToParcel(PendingIntent sender, Parcel out)

用于将PendingIntent或null指针写入Parcel的便捷功能。 您必须将其与readPendingIntentOrNullFromParcel(Parcel)结合使用，以便稍后读取。

```java
/*
* @param sender 要写入的PendingIntent，或者为null
* @param out PendingIntent要写入哪里
*/
public static void writePendingIntentOrNullToParcel(@Nullable PendingIntent sender, @NonNull Parcel out) {
  out.writeStrongBinder(sender != null ? sender.mTarget.asBinder() : null);
  if (sender != null) {
    OnMarshaledListener listener = sOnMarshaledListener.get();
    if (listener != null) {
      listener.onMarshaled(sender, out, 0 /* flags */);
    }
  }
}
```

##### public void writeToParcel(Parcel out, int flags)

将此对象展平到Parcel中。

```java
/*
* @param out 写对象的Parcel。
* @param flags 有关应如何写入对象的附加标志。 可能为0或Parcelable.PARCELABLE_WRITE_RETURN_VALUE。 值可以是0，也可以是Parcelable.PARCELABLE_WRITE_RETURN_VALUE和android.os.Parcelable.PARCELABLE_ELIDE_DUPLICATES的组合
*/
public void writeToParcel(Parcel out, int flags) {
  out.writeStrongBinder(mTarget.asBinder());
  OnMarshaledListener listener = sOnMarshaledListener.get();
  if (listener != null) {
    listener.onMarshaled(this, out, flags);
  }
}
```

