#### My Blog：[zhangmiao.cc](https://zhangmiao.cc/posts/9f23975a.html)

#### SMS涉及的主要类SmsManager

实现SMS主要用到SmsManager类，该类继承自java.lang.Object类，下面我们介绍一下该类的主要成员。

 **公有方法：**

- ArrayList<String> **divideMessage**(String text) 
  当短信超过SMS消息的最大长度时，将短信分割为几块。 
  参数：text——初始的消息，不能为空 
  返回值：有序的ArrayList<String>，可以重新组合为初始的消息

- static SmsManager **getDefault**() 
  获取SmsManager的默认实例。 
  返回值：SmsManager的默认实例

  <!-- more -->

- void **SendDataMessage**(String destinationAddress**,** String scAddress**,** short destinationPort**,** byte[] data**,**PendingIntent sentIntent, PendingIntent deliveryIntent) 
  发送一个基于SMS的数据到指定的应用程序端口。 
  参数： 
  1)、destinationAddress——消息的目标地址 
  2)、scAddress——服务中心的地址or为空使用当前默认的SMSC 3)destinationPort——消息的目标端口号 
  4)、data——消息的主体，即消息要发送的数据 
  5)、sentIntent——如果不为空，当消息成功发送或失败这个PendingIntent就广播。结果代码是Activity.RESULT_OK表示成功，或RESULT_ERROR_GENERIC_FAILURE、RESULT_ERROR_RADIO_OFF、RESULT_ERROR_NULL_PDU之一表示错误。对应RESULT_ERROR_GENERIC_FAILURE，sentIntent可能包括额外的“错误代码”包含一个无线电广播技术特定的值，通常只在修复故障时有用。 
  每一个基于SMS的应用程序控制检测sentIntent。如果sentIntent是空，调用者将检测所有未知的应用程序，这将导致在检测的时候发送较小数量的SMS。 
  6)、deliveryIntent——如果不为空，当消息成功传送到接收者这个PendingIntent就广播。
  异常：如果destinationAddress或data是空时，抛出IllegalArgumentException异常。

- void **sendMultipartTextMessage**(String destinationAddress**,** String scAddress**,** ArrayList<String> parts**,**ArrayList<PendingIntent> sentIntents, ArrayList<PendingIntent>  deliverIntents) 
  发送一个基于SMS的多部分文本，调用者应用已经通过调用**divideMessage**(String text)将消息分割成正确的大小。 
  参数： 
  1)、destinationAddress——消息的目标地址 
  2)、scAddress——服务中心的地址or为空使用当前默认的SMSC 
  3)、parts——有序的ArrayList<String>，可以重新组合为初始的消息 
  4)、sentIntents——跟**SendDataMessage**方法中一样，只不过这里的是一组PendingIntent 
  5)、deliverIntents——跟**SendDataMessage**方法中一样，只不过这里的是一组PendingIntent 
  异常：如果destinationAddress或data是空时，抛出IllegalArgumentException异常。

- void **sendTextMessage**(String destinationAddress, String scAddress, String text, PendingIntent sentIntent,PendingIntent deliveryIntent) 
  发送一个基于SMS的文本。参数的意义和异常前面的已存在的一样，不再累述。

**常量：**

- public static final int **RESULT_ERROR_GENERIC_FAILURE** 表示普通错误，值为1(0x00000001)

- public static final int **RESULT_ERROR_NO_SERVICE** 
  表示服务当前不可用，值为4 (0x00000004)

- public static final int **RESULT_ERROR_NULL_PDU** 
  表示没有提供pdu，值为3 (0x00000003)

- public static final int **RESULT_ERROR_RADIO_OFF** 
  表示无线广播被明确地关闭，值为2 (0x00000002)

- public static final int **STATUS_ON_ICC_FREE** 
  表示自由空间，值为0 (0x00000000)

- public static final int **STATUS_ON_ICC_READ** 
  表示接收且已读，值为1 (0x00000001)

- public static final int **STATUS_ON_ICC_SENT** 
  表示存储且已发送，值为5 (0x00000005)

- public static final int **STATUS_ON_ICC_UNREAD** 
  表示接收但未读，值为3 (0x00000003)

- public static final int **STATUS_ON_ICC_UNSENT** 
  表示存储但为发送，值为7 (0x00000007)


**第一：调用系统短信接口直接发送短信；主要代码如下：** 

```java
/**
     * 直接调用短信接口发短信
     * 
     * @param phoneNumber
     * @param message
     */
    public void sendSMS(String phoneNumber, String message) {
        // 获取短信管理器
        android.telephony.SmsManager smsManager = android.telephony.SmsManager
                .getDefault();
        // 拆分短信内容（手机短信长度限制）
        List<String> divideContents = smsManager.divideMessage(message);
        for (String text : divideContents) {
            smsManager.sendTextMessage(phoneNumber, null, text, sentPI,deliverPI);
        }
    }
```

**第二：调起系统发短信功能；主要代码如下：** 

```java
   /**
     * 调起系统发短信功能
     *
     * @param phoneNumber
     * @param message
     */
    public void doSendSMSTo(String phoneNumber, String message) {
        if (PhoneNumberUtils.isGlobalPhoneNumber(phoneNumber)) {
            Intent intent = new Intent(Intent.ACTION_SENDTO, Uri.parse("smsto:" + phoneNumber));
            intent.putExtra("sms_body", message);
            startActivity(intent);
        }
    }
```

下面来主要讲解第一种方法，第一种方法可以监控发送状态和对方接收状态使用的比较多。

 处理返回的状态代码如下: 

```java
//处理返回的发送状态 
        String SENT_SMS_ACTION = "SENT_SMS_ACTION";
        Intent sentIntent = new Intent(SENT_SMS_ACTION);
        sentPI= PendingIntent.getBroadcast(this, 0, sentIntent,
                0);
        // register the Broadcast Receivers
        this.registerReceiver(new BroadcastReceiver() {
            @Override
            public void onReceive(Context _context, Intent _intent) {
                switch (getResultCode()) {
                case Activity.RESULT_OK:
                    Toast.makeText(MainActivity.this,
                "短信发送成功", Toast.LENGTH_SHORT)
                .show();
                break;
                case SmsManager.RESULT_ERROR_GENERIC_FAILURE:
                break;
                case SmsManager.RESULT_ERROR_RADIO_OFF:
                break;
                case SmsManager.RESULT_ERROR_NULL_PDU:
                break;
                }
            }
        }, new IntentFilter(SENT_SMS_ACTION));

        
        //处理返回的接收状态 
        String DELIVERED_SMS_ACTION = "DELIVERED_SMS_ACTION";
        // create the deilverIntent parameter
        Intent deliverIntent = new Intent(DELIVERED_SMS_ACTION);
        deliverPI = PendingIntent.getBroadcast(this, 0,deliverIntent, 0);
        this.registerReceiver(new BroadcastReceiver() {
           @Override
           public void onReceive(Context _context, Intent _intent) {
               Toast.makeText(MainActivity.this,"收信人已经成功接收", Toast.LENGTH_SHORT)
               .show();
           }
        }, new IntentFilter(DELIVERED_SMS_ACTION));
```

**别忘了权限的问题：** 

```html
<uses-permission android:name="android.permission.SEND_SMS" /> 
```
