##apk��װ����##

####1 aapt ����R.java �ļ�####
aapt package -f -M AndroidManifest.xml -I D:\pro\adt\sdk\platforms\android-21\android.jar -S res  -J genn -m

####2 ����dex�ļ� (����src/*.java  R.java ��������jar)####
jack --classpath "$ANDROID_HOME/platforms/android-21/android.jar"  --import libs/okhttp-3.2.0.jar --import libs/picasso-2.5.2.jar --import libs/gson-2.7.jar  --output-dex  outt/ java/ genn/

####3 ���apk####
aapt package -f -M AndroidManifest.xml  -I "$ANDROID_HOME/platforms/android-21/android.jar" -S res -F outt/a.apk
cd outt/ && aapt add  a.apk classes.dex

[4k���� ��ѡ zipalign 4 outt/a.apk  outt/a-align.apk]
####4 ǩ��####
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ..\..\android.keystore  --storepass 44165481 -keypass 44165481 outt\a-align.apk android.keystore

- �ο� http://mp.weixin.qq.com/s?__biz=MzA4NDM2MjAwNw==&mid=2650575963&idx=1&sn=98c880d03e5f8204e40cf61c6f551397&scene=0#rd
