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
```kotlin
package changhwan.experiment.sopthomework

import android.content.Intent
import android.net.Uri
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.databinding.DataBindingUtil
import androidx.lifecycle.MutableLiveData
import changhwan.experiment.sopthomework.databinding.ActivityHomeBinding

class HomeActivity : AppCompatActivity() {

    private lateinit var binding: ActivityHomeBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_home)


        val introduce = Introduce(
            liveName = MutableLiveData<String>().apply { value = "이창환" },
            LiveAge = MutableLiveData<Int>().apply { value = 26 },
            liveMbti = MutableLiveData<String>().apply { value = "외워야겠다 ..." },
            liveIntroduction = MutableLiveData<String>().apply { value = "현재시각 새벽 2시에 푸라닭을 결국 먹고있는 저는 돼지입니다 제발 살은 안쪘으면 좋겠네요" },
            resorce = R.drawable.pig
        )

        binding.introduce = introduce
        binding.lifecycleOwner = this

        binding.homeToGit.setOnClickListener {
            val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://github.com/2chang5"))
            startActivity(intent)
        }

    }
}
```
이렇게 하면 원하던 바인 Databinding과 LiveData를 적용한것인데

이를 통해서 얻은것은 data 만 조작해주면 실시간으로 UI는 바뀌기 때문에 UI관련 코드를 생략할수있고

데이터 관리에만 신경쓸수있습니다.

## 프래그먼트에서의 lifecycleowner
프래그먼트에서는 this를 넘겨주면 안되고 

추가적으로 LifeCycleOwner에 대해 알아보자면

binding.lifecycleowner = viewLifecycleOwner 이런식으로 넘겨줘야합니다. 자세한 사항은 밑에 적어놓았습니다.

### LifeCycleOwner
Observer는 UI 컨트롤러(액티비티 or 프래그먼트)의 생명주기를 따릅니다.

Observer가 어떤 액티비티 혹은 어떤 프래그먼트의 수명주기를 따를지 정해주어야 합니다.

액티비티 같은 경우는 this 키워드를 이용해 현재 액티비티를 지정해주면 되는데

프래그먼트 같은 경우는 this를 사용하면 안 됩니다.

[그 이유](https://pluu.github.io/blog/android/2020/01/25/android-fragment-lifecycle/)

-> ViewModel과 혼재되어 설명되어있으므로 [ViewModel]()을 익히고 다시 보자

3줄 요약해보자면

1. 기존의 프래그먼트 생명주기를 사용하면 복수의 Observer가 호출될 가능성이 있다.

2. 구글이 실수한 부분이고, 이를 개선하기 위해 새로운 프래그먼트 생명주기가 도입되었다.

3. this 대신에 viewLifecycleOwner를 사용하면 된다.

### observeForever
위 에서 언급한 것처럼 Observer는 UI 컨트롤러의 생명주기를 따릅니다.

액티비티가 실행되면 관찰자도 감시를 시작하며 알림을 받을 수 있는 상태가 되고

액티비티가 정지되면 관찰자도 감시를 중단하며 알림을 받을 수 없는 상태가 됩니다.

그런데 lifeCycleOwner와 상관없이 항상 알림을 받을 수 있는 방법이 있는데 그게 바로 observerForever 메서드입니다.

이 메서드를 사용하면 lifeCycleOwner가 없어도 관찰자를 생성하고 LiveData와 연결할 수 있으며

UI 컨트롤러의 생명주기와 상관없이 항상 알림을 받을 수 있는 상태가 됩니다.

```kotlin
        // 일반적인 Observer 생성
        model.getAll().observe(this, Observer{ notice ->
 
        })
 
        // observerForever를 통한 생성 
        model.getAll().observeForever(Observer{ notice ->
 
        })
```
이렇게 observe대신 observerForever를 사용하면 되고 this를 넣어주지 않아도 됩니다.

대신 관찰자를 삭제할 때는 removeObserver 메서드를 사용하여 직접 삭제해주어야 합니다.

