# data binding 과 live data 동시 적용해보자
## 이글은 멧돼지님의 블로그를 보고 작성하였습니다.
LiveData와 DataBinding을 동시에 같이 사용한다면 LiveData의 값이 변경될때 View의 Data가 자동으로 바뀌어 

UI가 저절로 바뀌기 때문에 우리는 Data의 관리만 신경쓰면되기에 아주 편해집니다.

자 우선 기존의 방법과 같이 Databinding 과 Livedata를 사용했던대로 적용해봅시다.

## App 수준의 build.gradle
```kotlin
android {
    ...
    dataBinding {
        enabled = true
    }
}
```
```kotlin
implementation 'androidx.appcompat:appcompat:1.1.0'
```
## 레이아웃
```kotlin
//activity_home.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="introduce"
            type="changhwan.experiment.sopthomework.Introduce" />

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".HomeActivity">
 // 중략
 
 <TextView
                    android:id="@+id/homeMbti"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="30dp"
                    android:gravity="center"
                    android:text="@{`MBTI : ` + introduce.liveMbti}"
                    android:textSize="25dp"
                    android:textStyle="bold"
                    tools:text="mbti" />

                <TextView
                    android:id="@+id/homeIntro"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="30dp"
                    android:gravity="center"
                    android:text="@{`자기소개 : ` + introduce.liveIntroduction}"
                    android:textSize="25dp"
                    android:textStyle="bold"
                    tools:text="자기소개" />

                <Button
                    android:id="@+id/homeToGit"
                    android:layout_width="100dp"
                    android:layout_height="100dp"
                    android:layout_gravity="center"
                    android:layout_marginTop="30dp"
                    android:background="@drawable/gitlogo" />


            </LinearLayout>
        </ScrollView>
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```
이런식으로 직빵으로 변수 xml에 꽂아줍니다.

## dataclass 만들어 주기(안해도됨)
전에도 데이터 클래스에 변수들을 넣어줬기에 이번에도 데이터클래스를 만들고 거기다 

MutableLiveData 로 변수들을 쭉 만들어줄것입니다.

나중에 ViewModel 까지 사용한다면 dataclass가 아닌 ViewModel에 livedata들을 넣어서사용합니다.
```kotlin
//introduce.kt
package changhwan.experiment.sopthomework

import android.graphics.drawable.Drawable
import androidx.lifecycle.MutableLiveData

data class Introduce(
    var liveName: MutableLiveData<String>,
    var LiveAge: MutableLiveData<Int>,
    val liveMbti: MutableLiveData<String>,
    val liveIntroduction: MutableLiveData<String>,
    val resorce : Int,
)
```
## DataBinding 하려는 액티비티에서 데이터 연결 + lifecycleOwner넘겨주기
DataBinding을 사용하려는 액티비티에서 기존에 했던것처럼 

binding을 setting 하고 

DataClass를 초기화 할때 livedata를 초기화 시켜서 넣어줍니다. livedata 초기화 시키는 방법은 livedata 글에다 적어놨는데

여기선 .apply를 통해서 적용했습니다.

다음 binding변수에 데이터 클래스 연결해주고

binding 변수에 lifecycleOwner에 this를 넘겨주어 이 액티비티의 라이프 사이클을 따른다는 것을 명시합니다.
