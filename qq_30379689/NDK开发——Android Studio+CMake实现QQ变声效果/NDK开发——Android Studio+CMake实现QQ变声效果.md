[toc]

##��Ŀ��ʾ

![����дͼƬ����](http://img.blog.csdn.net/20170902172351113?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##Դ������

Github��https://github.com/AndroidHensen/NDKVoice

##��Ŀ����

��Ŀ����Fmod��Դ�⣬һ���ǳ���ͨ�õ���Ƶ���棬��ԭʼ����������Ч�Ĵ���������������Ч���������Ǳ�����Ƶ�Ĵ���

* ԭ����ֱ�Ӳ�����Ƶ�ļ�
* ���򣺶���Ƶ��߰˶�
* ���壺����Ƶ���Ͱ˶�
* ��㤣�������Ƶ�Ĳ���
* ��Ц��������Ƶ�Ĳ����ٶ�
* ���飺������Ƶ�Ļ���

##��������

1. ���Android Studio��SDKManager�����غ�ndk-build��cmake
2. �ٶ�fmod�Ĺ�������DownLoadҳ���У��ҵ�Androidƽ̨��Դ��������أ�������Ҫʹ��VPN������ϵ�ͷ�

##��Ŀ��ʼ

1������һ���µ���Ŀ����ѡ����C++

![����дͼƬ����](http://img.blog.csdn.net/20170902170637497?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2����ѡC++11��C++��������

![����дͼƬ����](http://img.blog.csdn.net/20170902170709297?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3���ҵ����غõ�fmodĿ¼�£���api/lowlevel/lib�У��������ļ������������Ŀ��libsĿ¼��

![����дͼƬ����](http://img.blog.csdn.net/20170902170819852?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4������Ŀ��Ӷ�fmod.jar����֧�֣�fmod.jar�Ҽ�Add As Libs

![����дͼƬ����](http://img.blog.csdn.net/20170902171030466?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5���ҵ����غõ�fmodĿ¼�£���api/lowlevel/inc�У��������ļ������������Ŀ��cpp(jni folder)Ŀ¼��

![����дͼƬ����](http://img.blog.csdn.net/20170902171306657?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

�����ú�����ͼ

![����дͼƬ����](http://img.blog.csdn.net/20170902171320417?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

6��׼����һ����Ƶ�ļ��ŵ�assets�ļ�����

![����дͼƬ����](http://img.blog.csdn.net/20170902172212909?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

7������app��gradle�ļ��е�cmake��ndk

```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.3"
    defaultConfig {
        applicationId "com.handsome.ndkvoice"
        minSdkVersion 16
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags "-std=c++11 -frtti -fexceptions"
                abiFilters "armeabi","arm64-v8a","armeabi-v7a","x86"
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets.main {
        jniLibs.srcDirs = ['libs']
        jni.srcDirs = []
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}
```

##�����д

1�����Ǵ���һ��Utils�࣬��д���ǵ�fmod��������

```
public class Utils {

    //��Ч����
    public static final int MODE_NORMAL = 0;
    public static final int MODE_LUOLI = 1;
    public static final int MODE_DASHU = 2;
    public static final int MODE_JINGSONG = 3;
    public static final int MODE_GAOGUAI = 4;
    public static final int MODE_KONGLING = 5;

    /**
     * ��Ч����2��ֵ
     *
     * @param path ��Ƶ�ļ�·��
     * @param type ��Ƶ�ļ�����
     */
    public native static void fix(String path, int type);

    static {
        System.loadLibrary("fmodL");
        System.loadLibrary("fmod");
        System.loadLibrary("sound");
    }
}
```

2��ͨ���նˣ�javah���ǵ��ļ�

![����дͼƬ����](http://img.blog.csdn.net/20170902171651325?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3���������õ�.h�ļ��Ž����ǵ�cppĿ¼�£�������sound.cpp

![����дͼƬ����](http://img.blog.csdn.net/20170902171742910?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4����дsound.cpp����

```
#include "inc/fmod.hpp"
#include <stdlib.h>
#include <unistd.h>
#include  "com_handsome_ndkvoice_Utils.h"

#include <jni.h>

#include <android/log.h>
#define LOGI(FORMAT,...) __android_log_print(ANDROID_LOG_INFO,"zph",FORMAT,##__VA_ARGS__);
#define LOGE(FORMAT,...) __android_log_print(ANDROID_LOG_ERROR,"zph",FORMAT,##__VA_ARGS__);

#define MODE_NORMAL 0
#define MODE_LUOLI 1
#define MODE_DASHU 2
#define MODE_JINGSONG 3
#define MODE_GAOGUAI 4
#define MODE_KONGLING 5

using namespace FMOD;

JNIEXPORT void JNICALL Java_com_handsome_ndkvoice_Utils_fix(JNIEnv *env,
        jclass jcls, jstring path_jstr, jint type) {
    //��������
    System *system;
    //����
    Sound *sound;
    //���ִ�����Ч��
    DSP *dsp;
    //���ڲ���
    bool playing = true;
    //���ֹ��
    Channel *channel;
    //�����ٶ�
    float frequency = 0;
    //��Ƶ��ַ
    const char* path_cstr = env->GetStringUTFChars(path_jstr, NULL);

    System_Create(&system);
    system->init(32, FMOD_INIT_NORMAL, NULL);

    try {
        //��������
        system->createSound(path_cstr, FMOD_DEFAULT, NULL, &sound);
        switch (type) {
            case MODE_NORMAL:
                //ԭ������
                system->playSound(sound, 0, false, &channel);
                break;
            case MODE_LUOLI:
                //�������߽���������һ����Ч
                system->createDSPByType(FMOD_DSP_TYPE_PITCHSHIFT, &dsp);
                //���������Ĳ���
                dsp->setParameterFloat(FMOD_DSP_PITCHSHIFT_PITCH, 1.8);
                //��ӽ���channel����ӽ����
                system->playSound(sound, 0, false, &channel);
                channel->addDSP(0, dsp);
                break;
            case MODE_DASHU:
                system->createDSPByType(FMOD_DSP_TYPE_PITCHSHIFT, &dsp);
                dsp->setParameterFloat(FMOD_DSP_PITCHSHIFT_PITCH, 0.8);
                system->playSound(sound, 0, false, &channel);
                channel->addDSP(0, dsp);
                break;
            case MODE_JINGSONG:
                system->createDSPByType(FMOD_DSP_TYPE_TREMOLO, &dsp);
                dsp->setParameterFloat(FMOD_DSP_TREMOLO_SKEW, 0.8);
                system->playSound(sound, 0, false, &channel);
                channel->addDSP(0, dsp);
                break;
            case MODE_GAOGUAI:
                //���˵�����ٶ�
                system->playSound(sound, 0, false, &channel);
                channel->getFrequency(&frequency);
                frequency = frequency * 2;
                channel->setFrequency(frequency);
                break;
            case MODE_KONGLING:
                system->createDSPByType(FMOD_DSP_TYPE_ECHO, &dsp);
                dsp->setParameterFloat(FMOD_DSP_ECHO_DELAY, 300);
                dsp->setParameterFloat(FMOD_DSP_ECHO_FEEDBACK, 20);
                system->playSound(sound, 0, false, &channel);
                channel->addDSP(0, dsp);
                break;
            }
    } catch (...) {
        LOGE("%s", "�����쳣");
        goto end;
    }
    system->update();

    //��λ��΢��
    //ÿ�����ж����Ƿ��ǲ���
    while (playing) {
        channel->isPlaying(&playing);
        usleep(1000);
    }
    goto end;

    //�ͷ���Դ
    end: env->ReleaseStringUTFChars(path_jstr, path_cstr);
    sound->release();
    system->close();
    system->release();
}
```

5������cmake�ļ�

```
cmake_minimum_required(VERSION 3.4.1)


find_library( # Sets the name of the path variable.
              log-lib
              log )

set(distribution_DIR ${CMAKE_SOURCE_DIR}/libs)

add_library( fmod
             SHARED
             IMPORTED )
set_target_properties( fmod
                       PROPERTIES IMPORTED_LOCATION
                       ${distribution_DIR}/${ANDROID_ABI}/libfmod.so )
add_library( fmodL
             SHARED
             IMPORTED )
set_target_properties( fmodL
                       PROPERTIES IMPORTED_LOCATION
                       ${distribution_DIR}/${ANDROID_ABI}/libfmodL.so )
add_library( sound
             SHARED
             src/main/cpp/sound.cpp )

include_directories(src/main/cpp/inc)

target_link_libraries( sound fmod fmodL
                       ${log-lib} )
```

6����д���ǵ�Activity����ÿ�����ʹ������¼������ñ��ط���

```
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private ExecutorService fixedThreadPool;
    private PlayerThread playerThread;
    private String path = "file:///android_asset/hensen.mp3";
    private int type;

    private LinearLayout normal, luoli, dashu, jingsong, gaoguai, kongling;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        normal = (LinearLayout) findViewById(R.id.normal);
        luoli = (LinearLayout) findViewById(R.id.luoli);
        dashu = (LinearLayout) findViewById(R.id.dashu);
        jingsong = (LinearLayout) findViewById(R.id.jingsong);
        gaoguai = (LinearLayout) findViewById(R.id.gaoguai);
        kongling = (LinearLayout) findViewById(R.id.kongling);
        normal.setOnClickListener(this);
        luoli.setOnClickListener(this);
        dashu.setOnClickListener(this);
        jingsong.setOnClickListener(this);
        gaoguai.setOnClickListener(this);
        kongling.setOnClickListener(this);

        fixedThreadPool = Executors.newFixedThreadPool(1);
        FMOD.init(this);
    }

    class PlayerThread implements Runnable {
        @Override
        public void run() {
            Utils.fix(path, type);
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.normal:
                type = Utils.MODE_NORMAL;
                break;
            case R.id.luoli:
                type = Utils.MODE_LUOLI;
                break;
            case R.id.dashu:
                type = Utils.MODE_DASHU;
                break;
            case R.id.jingsong:
                type = Utils.MODE_JINGSONG;
                break;
            case R.id.gaoguai:
                type = Utils.MODE_GAOGUAI;
                break;
            case R.id.kongling:
                type = Utils.MODE_KONGLING;
                break;
        }

        playerThread = new PlayerThread();
        fixedThreadPool.execute(playerThread);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        FMOD.close();
    }
}
```

##���̷���

* cmake�ļ��ķ���
* �������͵ķ���
* ʵ�ֱ���������̵ķ���

###cmake�ļ�����

1������cmake�汾

```
cmake_minimum_required(VERSION 3.4.1)
```

2������һ��·��������ָ�����ǵ�libsĿ¼��Ҳ����fmod.jar��so���Ŀ¼

```
set(distribution_DIR ${CMAKE_SOURCE_DIR}/libs)
```

3�����������Ҫ��fmod��fmodL��sound�Ŀ�

```
add_library( fmod
             SHARED
             IMPORTED )
set_target_properties( fmod
                       PROPERTIES IMPORTED_LOCATION
                       ${distribution_DIR}/${ANDROID_ABI}/libfmod.so )
add_library( fmodL
             SHARED
             IMPORTED )
set_target_properties( fmodL
                       PROPERTIES IMPORTED_LOCATION
                       ${distribution_DIR}/${ANDROID_ABI}/libfmodL.so )
add_library( sound
             SHARED
             src/main/cpp/sound.cpp )
```

������ͨ�����Ǹղ����õ�libsĿ¼�£��Ͷ�Ӧ��Gradle���õļܹ�����ƥ�䣬�ҵ�libfmod.so�ļ�

```
${distribution_DIR}/${ANDROID_ABI}/libfmod.so
```

Gradle���õļܹ�������ط�

![����дͼƬ����](http://img.blog.csdn.net/20170902173448296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



###ʵ�ֱ���������̵ķ���

�����ʵ����Ҫ���Ķ�fmod���ص����ӣ����������Ĵ����������Ч�Ĵ�������д��

![����дͼƬ����](http://img.blog.csdn.net/20170902173847118?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

��ŵ�����

1. ����fmod��ϵͳ��System_Create(&system);
2. ϵͳ���г�ʼ����system->init(32, FMOD_INIT_NORMAL, NULL);
3. ������Ƶ��system->createSound(path_cstr, FMOD_DEFAULT, NULL, &sound);
4. ������ͬ���͵���Ч��system->createDSPByType(FMOD_DSP_TYPE_PITCHSHIFT, &dsp);
5. ����������system->playSound(sound, 0, false, &channel);
6. ����Ч���������У�channel->addDSP(0, dsp);
7. �ر���Դ��end����
