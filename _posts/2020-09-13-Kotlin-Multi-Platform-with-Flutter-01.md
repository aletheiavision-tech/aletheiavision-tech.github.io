---
title: Flutter와 Kotlin Multiplatform으로 앱 개발의 효율성을 추구하기(1)
published: true
---

어떻게 모바일 앱 개발의 효율성을 추구할 것인가?  
빠르게 만들 수 있고, 유지 보수 비용도 줄 일 수 있다면, 그 시간을 Test나 설계 등에 투자해서,   
같은 시간을 들였을 때 보다 좋은 Product를 만들어 낼 수 있을 것이다.   
이에 대한 해답으로서, Kotlin Multiplatform과  Flutter의 조합을 살펴보려고 한다.   

1. [코틀린 멀티플랫폼](https://aletheiavision-tech.github.io/Kotlin-Multi-Platform-with-Flutter-01)  
2. [Flutter](#flutter)   
3. [Flutter와 코틀린 멀티플랫폼 연동](#flutter-with-kotlin-multi-platform)   
4. [아키텍쳐](#architecture)    
 
그 첫 번째로 살펴 볼 것은 Kotlin Muliti Platform이다.   

2019년 덴마크 코펜하겐에서 열린 Kotlin Conference에 참석한 기억을 떠올려 보자면,  
*(사실 일주일 내내 해가 없고, 3시에 어둑어둑 해지는 날씨가 가장 먼저 떠오르지만)*   
2일간 Conference에서 유독 강조 되었던 것은, **Muliti Platform**과 Coroutine이었고, 
가장 인상 깊었던 것은 Kotlin 1.4 버전에 대한 스포일러와 역시 **Multi Platform**이었다.   
그 만큼 코틀린의 미래에 Muti Platform이 차지하는 비중은 크다고 느껴졌고,   
Kotlin Update를 통해 지속적으로 지원의 강도를 높이고 있는 모습이다.   

먼저 개인적으로 xml에 동작 코드를 넣으면 가독성이 떨어지고,    
debugging이 어려워 지기 때문에 단순한 앱이나, UI 중심의 APP이 아닌 다음에는 지양하고 있기 때문에    
해당 부분은 고려하지 않았다.    

Multi Platform은 기본적으로 다음과 같은 계층 구조를 가지고 있다.
![그림](https://4d4f6p22cgml1ale382crgth-wpengine.netdna-ssl.com/wp-content/uploads/2020/01/slide-25.001.jpeg)

Web, Desktop, Mobile 까지의 멀티플랫폼을 지원한다.       
주로 사용하게 될 Android/iOS 조합은 Kotlin Native를 사용하고 있는 것을 확인할 수 있다.   
이를 통해서, ios의 최상위 레벨의 framework인 cocoa touch framework(foundation, uikit등)와 posix를 거의 대부분 호출할 수 있다.    
objective-c,c++과 swift 없이 ios app을 실행할 수 있다니 상당히 매력적이 아닐 수 없다.   
IntelliJ의 **kotlin/Native Plugin**이 바로 이 역활을 해준다.    

또 계층 구조는 각각의 Platform 별 main으로 구성된다.   
```
kotlin {
    sourceSets {
        desktopMain {
            dependsOn(commonMain)
        }
        linuxX64Main {
            dependsOn(desktopMain)
        }
        mingwX64Main {
            dependsOn(desktopMain)
        }
        macosX64Main {
            dependsOn(desktopMain)
        }
    }
```

이때 각각의 main들의 위치는  
```
src/commonMain  
src/desktopMain  
```
이런 식이 된다.   

즉 platform 별도 구현해야 할 function과 공통 로직을 분리하게 되는데,
commonMain()에서 expect keyword를 사용했다면, 하위 Main들에서 이를 각각 구현해 주어야 하고,
keyword를 사용하지 않으면 공통적으로 사용 가능하다는 것을 의미한다.
즉 commonMain()에 다음과 같이 선언되었다면,
```
expect fun souldImplementMe()
fun justFunc() {
}
```
jvmMain, jsMain, macosX64Main 등에서는
~~~~
actual fun souldImplemenatMe() {
}
~~~~
이렇게 되어야 한다는 의미.
이 키워드는 function 뿐만이 아니라, class에 대해서도 적용할 수 있다.   

multiplatform에서는 주요한 target shortcut이 미리 내장되어 있다.
이런 target에는 ios, watchos, tvos등이 있고,
ios target shortcut에는 iosArm64, iosX64와 같은 target들이 내장되어 있다.
(각각 iosMain, iosArm64Main, iosX64Main을 지닌고 있다.)
이런 hierarchical한 구조를 사용하고 싶다면, gradle.properties에 다음을 명시해야 한다.
```
kotlin.mpp.enableGranularSourceSetsMetadata=true
```

역시 이 부분을 Customizing하고 싶다면,    
다음과 같은 식으로 해준다.   
```
kotlin {
    sourceSets{
        iosMain {
            dependsOn(commonMain)
            iosX64Main.dependsOn(it)
            iosArm64Main.dependsOn(it)
        }
    }
}
```

platform dependent한 library들은 iosArm64나 iosX64에는 적용되지 않고,
부모인 iosMain에만 적용 가능하다.
부모에 적용한 library를 자식들에서 사용하게 하려면, **gralde.properties**에 다음을 추가해 주도록 한다.
```
kotlin.mpp.enableGranularSourceSetsMetadata=true
kotlin.native.enableDependencyPropagation=false


> Written with [StackEdit](https://stackedit.io/).
