---
title:  "[Android/Kotlin] Android MVVM 패턴 적용하기" 

categories:
  -  Android
tags:
  - [Android, Kotlin, MVVM]

toc: true
toc_sticky: true

date: 2025-05-16
last_modified_at: 2025-05-16
---

## MVVM 패턴이란 

MVC 패턴에서는 View와 Model 간의 직접적인 의존성이 발생하는 문제가 있었습니다. 이로 인해 하나의 변경이 다른 컴포넌트에 영향을 주어 유지보수가 어려워지는 단점이 있었습니다. 이를 개선하기 위해 등장한 MVP 패턴은 View와 Model 을 분리하고, Presenter가 이둘 사이의 중재자 역할을 수행하도록 설계되었습니다. 그러나, MVP 패턴에서는 Presenter가 View를 참조하고 있어, 의존성이 여전히 존재하며, 복잡한 화면일수록 Presenter의 책임이 과하게 커지는 문제점이 있었습니다.

이러한 구조적 한계를 보완하기 위해 MVVM 패턴이 등장했습니다. MVVM에서 ViewModel은 View를 직접 참조하지 않으며, 데이터 바인딩이나 옵저버 패턴을 활용하여 View와 간접적으로 연결됩니다. 이로 인해, View와 ViewModel 간의 의존성이 제거되며, UI로직과 비지니스 로직의 명확한 분리가 가능해졌습니다.

안드로이드에서는 Jetpack의 ViewModel과 LiveData, DataBinding을 통해 MVVM 아키텍쳐를 자연스럽게 구현할 수 있습니다. ViewModel은 UI에 필요한 데이터를 제공하고, LiveData는 데이터의 변경 사항을 View에 알리며, View는 이를 바인딩하여 화면을 구성합니다. 이 구조는 테스트 용이성, 유지보수성, 코드 재사용성 측면에서 큰 이점을 제공합니다.

## LiveData

LiveData는 MVVM을 구현하기 위해서 Android가 제공하는 관찰 가능한 데이터 홀더 클래스입니다. 일반 관찰 가능한 클래스와 달리 LiveData는 수명주기를 인식합니다. 즉, 액티비티, 프래그먼트, 서비스 등의 다른 앱 구성요소의 수명주기를 고려합니다.

ViewModel 클래스에 ViewModel() 클래스를 상속받으면, 해당 기능을 통해 Android 에서 MVVM패턴을 구현할 수 있습니다.


## MVVM 적용

MVVM 패턴 예제로는 `+`, `-` 버튼을 활용하여 숫자를 올리고 내리는 기능을 만들어 봅니다.

우선, MainActivity의 `activity_main.xml`을 열어서 다음과 같이 작성합니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textViewNumber"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="0"
        android:textSize="40sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/buttonMinus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="52dp"
        android:text="-"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/textViewNumber"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/buttonPlus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="52dp"
        android:text="+"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/textViewNumber"
        app:layout_constraintTop_toTopOf="parent" />


</androidx.constraintlayout.widget.ConstraintLayout>
```


![](/images/Pasted%20image%2020250506200632.png)

간단하게 버튼 2개와 텍스트 1개로 구성하였습니다.

그 후에, `MainViewModel`이라는 클래스를 만들고, ViewModel() 클래스를 상속시켜 줍니다.

```kotlin
package com.example.mvvmstructure

import androidx.lifecycle.ViewModel

class MainViewModel : ViewModel()
{
    
}
```

이제 해당 ViewModel에서 앱의 비지니스 로직을 작성할것입니다.

지금 `-`, `+`버튼 2개를 눌릴때 이벤트가 발생하기 때문에, `addCount()` `subCount()` 2개의 메서드를 viewModel에 정의해 줍니다. 더해줄 모델은 LiveData를 사용하여 만든 객체인 `count`로 합니다.

```kotlin
package com.example.mvvmstructure

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

class MainViewModel : ViewModel()
{
    private val _count = MutableLiveData<Int>()
    val count: LiveData<Int>
        get() = _count

    fun addCount()
    {
        _count.value = (_count.value ?: 0) + 1
    }

    fun subCount()
    {
        val currentNumber = count.value ?: 0
        if (currentNumber > 0)
        {
            _count.value = currentNumber - 1
        }
    }
}
```

이렇게 하고, MainActivity로 가서, viewModel을 멤버변수로 등록합니다. viewModel 은 `by viewModel()` 코드를 통해서 초기화 됩니다.

`by viewModel()`은 Kotlin의 위임 속성과 Android Jetpack의 일부 라이브러리를 사용한 것으로 ViewModel 인스턴스를 생성하고 관리를 위임한다는 의미를 내포하고 있습니다. 또한, 생명주기도 관리가 됩니다.

MainActivity는 `View`코드 이기 때문에, xml의 View들을 전부 알아도 됩니다. 여기서 `View`객체들을 가져와서 이벤트를 연결해 줍니다. 이벤트가 발생했을 때의 로직은 `ViewModel`에서 정의하게 됩니다.

그리고, 결과적으로 count가 바뀌면, 해당 값이 `View`의 텍스트에 반영되어야 하기 때문에, 텍스트는 count를 관찰 즉, `observe` 해야 합니다.

```kotlin
package com.example.mvvmstructure

import android.app.Activity
import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.example.mvvmstructure.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity()
{
    private lateinit var _binding: ActivityMainBinding
    private val _viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?)
    {
        super.onCreate(savedInstanceState)
        _binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(_binding.root)

        _viewModel.count.observe(this) { count ->
            _binding.textViewNumber.text = count.toString()
        }

        _binding.buttonMinus.setOnClickListener {
            _viewModel.subCount()
        }

        _binding.buttonPlus.setOnClickListener {
            _viewModel.addCount()
        }
    }
}
```

이렇게 하면, 간단하게 MVVM 패턴을 완성시킨 것입니다. 그리고 저는 각 행동을 함수화 시켜서 해당 코드가 어떤일을 하는지 함수명으로 나타내는것을 선호하기 때문에, 코드를 다음과 같이 바꿔줍니다. (이렇게하면 변경사항이 생겼을 때, 주석없이 바로 변경사항이 생길지점을 찾을 수 있습니다.)

```kotlin
package com.example.mvvmstructure

import android.app.Activity
import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.example.mvvmstructure.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity()
{
    private lateinit var _binding: ActivityMainBinding
    private val _viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?)
    {
        super.onCreate(savedInstanceState)
        _binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(_binding.root)

        init()
    }

    private fun init()
    {
        initObserve()
        initEvents()
    }

    private fun initEvents()
    {
        initButtonPlusEvent()
        initButtonMinusEvent()
    }

    private fun initObserve()
    {
        _viewModel.count.observe(this) { count ->
            _binding.textViewNumber.text = count.toString()
        }
    }

    private fun initButtonPlusEvent()
    {
        _binding.buttonMinus.setOnClickListener {
            _viewModel.subCount()
        }
    }

    private fun initButtonMinusEvent()
    {
        _binding.buttonPlus.setOnClickListener {
            _viewModel.addCount()
        }
    }
}
```


![](/images/Pasted%20image%2020250506203519.png)

정상적으로 잘 작동합니다.


