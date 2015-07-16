apk签名的官方文档 http://developer.android.com/intl/zh-cn/tools/publishing/app-signing.html#considerations
手动签名
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
  