apkǩ���Ĺٷ��ĵ� http://developer.android.com/intl/zh-cn/tools/publishing/app-signing.html#considerations
�ֶ�ǩ��
1 �� keytool ����˽Կ
 keytool -genkey -v -keystore release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
2 �����apk...
3 ʹ��jarsignerǩ��
 jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore release-key.keystore my.apk alias_name
 ����my.apk�ᱻ������д
4 ��֤ǩ��
  jarsigner -verify -verbose -certs my.apk
5 4k����--apk����
  zipalign -v 4 my.apk my-aligned.apk
------------------------------------------------------------------------------------------------------------
  