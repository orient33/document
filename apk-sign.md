http://developer.android.com/intl/zh-cn/tools/publishing/app-signing.html#considerations
apkǩ���Ĺٷ��ĵ� -- �ֶ�ǩ��
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
http://blog.csdn.net/dacainiao007/article/details/17661413
ǩ������--aospԴ�� /build/tools/signapk/SignApk.java
ǩ�����apk�ж���META-INF�ļ���,�������ļ� MANIFEST.MF,CERT.SF,CERT.RSA,
1 ����MANIFEST.MF
  ����apk�������з��ļ��з�ǩ���ļ���file,�������SHA1����ǩ����Ϣ������Base64����
  private static Manifest addDigestsToManifest(JarFile jar)
2 ����CERT.SF�ļ�
  ����1���ɵ�MANIFEST.MF��ʹ�� SHA1-RSA�㷨,��˽Կǩ��
  Signature signature = Signature.getInstance("SHA1withRSA");
  .......
3 ����CERT.RSA�ļ�
  CERT.RSA�����˹�Կ�������õļ����㷨����Ϣ