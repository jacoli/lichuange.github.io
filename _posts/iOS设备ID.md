---
title: iOS设备ID
date: 2016-04-27
---

NSString *identifierForVendor = [[UIDevice currentDevice].identifierForVendor UUIDString]; 
NSString *identifierForAdvertising = [[ASIdentifierManager sharedManager].advertisingIdentifier UUIDString]; 

设备标识
identifierForVendor，
app在一个device上的标识，从ios6开始支持。一个设备上相同vendor的app获取到相同值，不同vendor则不同值。
app如果从App Store直接下载，该值由App Store分配，如果app是企业版、Ad Hoc、或开发版，则该值由bundle identifier计算。
设备上删除所有该Vendor的apps后，下次再安装app，值会发生变化。
If the value is nil, wait and get the value again later. This happens, for example, after the device has been restarted but before the user has unlocked the device.

advertisingIdentifier
一个设备上所有app获取到都是相同的值。
If the value is nil, wait and get the value again later. This happens, for example, after the device has been restarted but before the user has unlocked the device.





### iOS获取设备唯一标识的各种方法？IDFA、IDFV、UDID分别是什么含义？


[摘要：1、UDID (Unique Device Identifier) UDID的齐称是Unique Device Identifier，望文生义，它便是苹果IOS装备的独一辨认码，它由40个字符的字母战数字构成。正在良多须要限定] 


一、UDID (Unique Device Identifier)

	UDID的全称是Unique Device Identifier，顾名思义，它就是苹果IOS设备的唯一识别码，它由40个字符的字母和数字组成。在很多需要限制一台设备一个账号的应用中经常会用到。在iOS5中可以获取到设备的UDID，后来被苹果禁止了。
二、UUID（Universally Unique Identifier）

	UUID是Universally Unique Identifier的缩写，中文意思是通用唯一识别码。它是让分布式系统中的所有元素，都能有唯一的辨识资讯，而不需要透过中央控制端来做辨识资讯的指定。这样，每个人都可以建立不与其它人冲突的 UUID。在此情况下，就不需考虑数据库建立时的名称重复问题。苹果公司建议使用UUID为应用生成唯一标识字符串。
	开发者可以在应用第一次启动时调用一 次，然后将该串存储起来，替代UDID来使用。但是，如果用户删除该应用再次安装时，又会生成新的字符串，所以不能保证唯一识别该设备。使用UUID，就要考虑应用被删除后再重新安装时的处理。一个解决的办法是：UUID一般只生成一次，保存在iOS系统里面，如果应用删除了，重装应用之后它的UUID还是一样的，除非系统重置 。但是不能保证在以后的系统升级后还能用（如果系统保存了该信息就能用）。
三、MAC Address

用来表示互联网上每一个站点的标识符，采用十六进制数表示，共六个字节（48位）。其中，前三个字节是由IEEE的注册管理机构 RA负责给不同厂家分配的代码(高位24位)，也称为“编制上唯一的标识符” （Organizationally Unique Identifier)，后三个字节(低位24位)由各厂家自行指派给生产的适配器接口，称为扩展标识符（唯一性）。

MAC地址在网络上用来区分设备的唯一性，接入网络的设备都有一个MAC地址，他们肯定都是不同的，是唯一的。一部iPhone上可能有多个MAC地址，包括WIFI的、SIM的等，但是iTouch和iPad上就有一个WIFI的，因此只需获取WIFI的MAC地址就好了，也就是en0的地址。
MAC地址就如同我们身份证上的身份证号码，具有全球唯一性。这样就可以非常好的标识设备唯一性，类似与苹果设备的UDID号，通常的用途有：1）用于一些统计与分析目的，利用用户的操作习惯和数据更好的规划产品；2）作为用户ID来唯一识别用户，可以用游客身份使用app又能在服务器端保存相应的信息，省去用户名、密码等注册过程。
使用Mac地址生成设备的唯一标识主要分三种：
1、直接使用“MAC Address” 
2、使用“MD5(MAC Address)”
3、使用“MD5(Mac Address+bundle_id)”获得“机器＋应用”的唯一标识（bundle_id 是应用的唯一标识）
在iOS7之后，如果请求Mac地址都会返回一个固定值。
四、IDFA（identifierForIdentifier）

广告标示符，适用于对外：例如广告推广，换量等跨应用的用户追踪等。

	是iOS 6中另外一个新的方法，提供了一个方法advertisingIdentifier，通过调用该方法会返回一个NSUUID实例，最后可以获得一个UUID，由系统存储着的。不过即使这是由系统存储的，但是有几种情况下，会重新生成广告标示符。如果用户完全重置系统（(设置程序 -> 通用 -> 还原 -> 还原位置与隐私) ，这个广告标示符会重新生成。另外如果用户明确的还原广告(设置程序-> 通用 -> 关于本机 -> 广告 -> 还原广告标示符) ，那么广告标示符也会重新生成。关于广告标示符的还原，有一点需要注意：如果程序在后台运行，此时用户“还原广告标示符”，然后再回到程序中，此时获取广 告标示符并不会立即获得还原后的标示符。必须要终止程序，然后再重新启动程序，才能获得还原后的广告标示符。
在同一个设备上的所有App都会取到相同的值，是苹果专门给各广告提供商用来追踪用户而设的，用户可以在 设置|隐私|广告追踪 里重置此id的值，或限制此id的使用，故此id有可能会取不到值，但好在Apple默认是允许追踪的，而且一般用户都不知道有这么个设置，所以基本上用来监测推广效果，是戳戳有余了。

注意：由于idfa会出现取不到的情况，故绝不可以作为业务分析的主id，来识别用户。

代码：

#import <AdSupport/AdSupport.h>

  NSString *adId = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
五、IDFV（identifierForVendor）

Vindor标示符，适用于对内：例如分析用户在应用内的行为等。

是给Vendor标识用户用的，每个设备在所属同一个Vender的应用里，都有相同的值。其中的Vender是指应用提供商，但准确点说，是通过BundleID的DNS反转的前两部分进行匹配，如果相同就是同一个Vender，例如对于com.somecompany.appone,com.somecompany.apptwo 这两个BundleID来说，就属于同一个Vender，共享同一个idfv的值。和idfa不同的是，idfv的值是一定能取到的，所以非常适合于作为内部用户行为分析的主id，来标识用户，替代OpenUDID。

注意：如果用户将属于此Vender的所有App卸载，则idfv的值会被重置，即再重装此Vender的App，idfv的值和之前不同。

代码：

  NSString *idfv = [[[UIDevice currentDevice] identifierForVendor] UUIDString];
六、OPEN UDID

	每台iOS设备的OpenUDID是通过第一个带有OpenUDID SDK包的App生成，如果你完全删除全部带有OpenUDID SDK包的App（比如恢复系统等），那么OpenUDID会重新生成，而且和之前的值会不同，相当于新设备；
	优点是没有用到MAC地址；不同设备能够获取各自唯一的识别码，保证了唯一性，可以用于以往UDID的相关用途；从代码分析OpenUDID的获取，识别码获取方便并且保存谨慎。缺点是当将设备上所有使用了OpenUDID方案的应用程序删除，且设备关机重启，xcode彻底清除并重启，重装应用程序去获取OpenUDID，此时OpenUDID变化，与之前不一样了，所有OpenUDID应用卸载后，由UIPasteboard保存的数据即被清除，重装故会重新获取新的OpenUDID。
那么当因为用户干预或者恶意程序，致使UIPasteboard数据清除，从而导致OpenUDID被删除，重装也会获取新的OpenUDID。

	OpenUDID生成唯一识别码的代码：	

unsigned char result[16];  
        const charchar *cStr = [[[NSProcessInfo processInfo] globallyUniqueString] UTF8String];  
        CC_MD5( cStr, strlen(cStr), result );  
        _openUDID = [NSString stringWithFormat:  
                     @"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%08x",  
                     result[0], result[1], result[2], result[3],   
                     result[4], result[5], result[6], result[7],  
                     result[8], result[9], result[10], result[11],  
                     result[12], result[13], result[14], result[15],  
                     arc4random() % 4294967295]; 