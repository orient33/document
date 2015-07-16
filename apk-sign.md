http://developer.android.com/intl/zh-cn/tools/publishing/app-signing.html#considerations
apk签名的官方文档 -- 手动签名
1 用 keytool 生成私钥
 keytool -genkey -v -keystore release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
2 编译好apk...
3 使用jarsigner签名
 jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore release-key.keystore my.apk alias_name
 这里my.apk会被覆盖重写
4 验证签名
  jarsigner -verify -verbose -certs my.apk
5 4k对齐--apk对齐
  zipalign -v 4 my.apk my-aligned.apk
------------------------------------------------------------------------------------------------------------
http://blog.csdn.net/dacainiao007/article/details/17661413
签名过程--aosp源码 /build/tools/signapk/SignApk.java
签名后的apk中多了META-INF文件夹,有三个文件 MANIFEST.MF,CERT.SF,CERT.RSA,
1 生成MANIFEST.MF
  遍历apk包中所有非文件夹非签名文件的file,逐个生成SHA1数字签名信息，再用Base64编码
  private static Manifest addDigestsToManifest(JarFile jar)
2 生成CERT.SF文件
  对于1生成的MANIFEST.MF，使用 SHA1-RSA算法,用私钥签名
  Signature signature = Signature.getInstance("SHA1withRSA");
  .......
3 生成CERT.RSA文件
  CERT.RSA保存了公钥和所采用的加密算法等信息