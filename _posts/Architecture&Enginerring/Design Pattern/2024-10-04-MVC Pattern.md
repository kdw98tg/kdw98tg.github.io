---
layout: post

title: "[Design Pattern] MVC 패턴 (MVC Pattern)"

categories:
  -  Architecture&Engineering
  
tags:
  - Java
  - Head-First-Design-Pattern
  - Book

toc: true
toc_sticky: true

date: 2024-10-04
last_modified_at: 2024-10-04
---

## MVC 패턴 이란?

MVC 은 Model, View, Controller 의 앞글자만 따서 만든 패턴의 이름 입니다. MVC 패턴은 전략패턴, 옵저버 패턴, 컴포지트 패턴 등이 어우러진 복합 패턴 입니다.

MVC가 없을 때는 사용자에게 보여지는 View 코드와, 엔티티의 속성을 담당하는 Model 과, 그것들을 제어하는 Controller 코드가 한 스크립트에 합쳐져 있어서, 유지보수하기가 힘들고 분업을 하기가 힘들었습니다.

그래서 과거에 만들어진 소프트웨어들은 디자인 보다는 기능 중심이 많았습니다.

하지만 MVC 패턴이 등장하고, 각 기능에 대한 역할분담이 이루어지면서, 소프트웨어들의 디자인 및 기능이 확연하게 발전하였습니다. 

MVC 패턴의 동작과정을 살펴보면, 사용자가 View를 통해 상호작용을 합니다. View 는 사용자가 상호작용 했다는 사실을 Controller 에게 알려 줍니다. 그러면 Controller 는 해당 엔티티의 정보를 가지고 있는 Model 에게 상태를 바꿔달라고 요청을 합니다. Model 에서 상태가 바뀌면, View 에게 상태가 바뀐것을 알려주고, View 를 갱신하게 됩니다.

다시한번 보기좋게 정리한다면 다음과 같습니다.

1. 사용자가 뷰와 상호작용 합니다.
2. 컨트롤러는 모델에게 상태를 변경하라고 요청합니다.(컨트롤러가 뷰를 바꿔달라고 요청할 수도 있음)
3. 모델에서 상태가 변경되면 모델이 뷰에게 그 사실을 알립니다.

이를 그림으로 표현하면 다음과 같습니다.

MVC 패턴을 표현하는 많은 그림들이 있지만, 이 그림이 가장 적절하다고 생각합니다.

![](/assets/images/posts/Pasted%20image%2020241004122611.png)

MVC 패턴은 다양한 웹 프레임워크에 적용되어 있습니다. 각 프레임워크마다 MVC 를 구현한 방법은 조금씩 다르긴 합니다. 대표적인 예로 Spring Boot, ASP.net 이 있습니다. 

## 예제

MVC 패턴을 활용해서 BPM 제어 도구를 만드는 예제를 만들어 봅니다.

(이번 예제는 View 를 생성하는 라이브러리를 사용합니다. 뷰를 만드는 과정은 주석으로 대체 하겠습니다.)

### Model 만들기

우선, 모델클래스를 먼저 만듭니다. 모델클래스는 위에서 말했던 대로, 각 상태들(데이터)를 가지고 있고, 상태가 변경되었을 때, View 에 알려줘야 하기 때문에, `Observer` 패턴을 사용하여 구현합니다.

모델을 만들 때, 재사용성과 확장성을 고려하여 Interface로 틀을 잡습니다. 또한, 옵저버 패턴을 구현하기 위한 Observer 인터페이스도 만듭니다.

🗅 **<span style="color: #c03a92">interface BeatModelInterface</span>**

```java
package src.combine_mvc.model;  
  
public interface BeatModelInterface {  
    public void initialize();  
    public void on();  
    public void off();  
    public void setBPM(int _bpm);  
    public int getBPM();  
    public void registerObserver(BeatObserver _observer);  
    public void removeObserver(BeatObserver _observer);  
    public void registerObserver(BPMObserver _observer);  
    public void removeObserver(BPMObserver _observer);  
}
```

🗅 **<span style="color: #c03a92">interface BeatObserver</span>**

```java
package src.combine_mvc.model;  
  
public interface BeatObserver {  
    public void updateBeat();  
}
```

🗅 **<span style="color: #c03a92">interface BPMObserver</span>**

```java
package src.combine_mvc.model;  
  
public interface BPMObserver {  
    public void updateBPM();  
}
```

인터페이스들을 만든 뒤, `BeatModel`클래스를 만들어서, `BeatModelInterface`를 상속시켜 줍니다.

해당 클래스는 `Observer` 들을 담을 수 있는 리스트가 있고, 데이터가 변경되면 `notifyObservers()`함수를 통해서 `View` 에게 상태가 변경되었음을 알려줍니다.

🗅 **<span style="color: #c03a92">class BeatModel</span>**

```java
package src.combine_mvc.model;  
  
import javax.sound.sampled.AudioSystem;  
import javax.sound.sampled.Clip;  
import javax.sound.sampled.Line;  
import java.io.File;  
import java.util.ArrayList;  
import java.util.List;  
  
public class BeatModel implements BeatModelInterface, Runnable {  
  
    private List<BeatObserver> beatObservers = new ArrayList<BeatObserver>();  
    private List<BPMObserver> bpmObservers = new ArrayList<BPMObserver>();  
    private int bpm = 90;  
    private Thread thread;  
    private Clip clip;  
    private boolean stop = false;  
  
    @Override  
    public void initialize() {  
        try {  
            File resource = new File("clap.wav");  
            clip = (Clip) AudioSystem.getLine(new Line.Info(Clip.class));  
        } catch (Exception ex) {  
            System.out.println("Error: Can't load clip");  
            System.out.println(ex);  
        }  
    }  
  
    @Override  
    public void on() {  
        bpm = 90;  
        notifyBPMObservers();  
        thread = new Thread(this);  
        stop = false;  
        thread.start();  
    }  
  
    @Override  
    public void off() {  
        stopBeat();  
        stop = true;  
    }  
  
    @Override  
    public void setBPM(int _bpm) {  
        this.bpm = _bpm;  
        notifyBPMObservers();  
    }  
  
    @Override  
    public int getBPM() {  
        return bpm;  
    }  
  
    @Override  
    public void registerObserver(BeatObserver _observer) {  
        beatObservers.add(_observer);  
    }  
  
    @Override  
    public void removeObserver(BeatObserver o) {  
        int i = beatObservers.indexOf(o);  
        if (i >= 0) {  
            beatObservers.remove(i);  
        }  
    }  
  
    public void notifyBeatObservers() {  
        for (BeatObserver beatObserver : beatObservers) {  
            ((BeatObserver) beatObserver).updateBeat();  
        }  
    }  
  
    @Override  
    public void registerObserver(BPMObserver _observer) {  
        for (BPMObserver bpmObserver : bpmObservers) {  
            ((BPMObserver) bpmObserver).updateBPM();  
        }  
    }  
  
    @Override  
    public void removeObserver(BPMObserver _observer) {  
        int i = bpmObservers.indexOf(_observer);  
        if (i >= 0) {  
            bpmObservers.remove(i);  
        }  
    }  
  
    public void notifyBPMObservers() {  
        for (BPMObserver bpmObserver : bpmObservers) {  
            bpmObserver.updateBPM();  
        }  
    }  
  
    @Override  
    public void run() {  
        while (!stop) {  
            playBeat();  
            notifyBeatObservers();  
            try {  
                Thread.sleep(60000 / getBPM());  
            } catch (Exception ex) {  
                System.out.println(ex.getMessage());  
            }  
        }  
    }  
  
    public void playBeat() {  
        clip.setFramePosition(0);  
        clip.start();  
    }  
  
    public void stopBeat() {  
        clip.setFramePosition(0);  
        clip.stop();  
    }  
}
```

### View 만들기

`Model` 을 만들었으니, 해당 데이터를 사용자에게 보여줄 `View` 객체를 만듭니다. 해당 객체는 Java 에서 제공하는 UI 프레임워크를 사용하는데, MVC 패턴과 직접적인 연관이 없어 주석으로 대체 하겠습니다.

`View`는 `Model` 의 옵저버 이므로, Observer 인터페이스를 상속시켜 줍니다.

🗅 **<span style="color: #c03a92">class DJView</span>**

```java
package src.combine_mvc.view;  
  
import src.combine_mvc.controller.ControllerInterface;  
import src.combine_mvc.model.BPMObserver;  
import src.combine_mvc.model.BeatModelInterface;  
import src.combine_mvc.model.BeatObserver;  
  
import javax.swing.*;  
import java.awt.*;  
import java.awt.event.ActionEvent;  
import java.awt.event.ActionListener;  
  
public class DJView implements ActionListener,  BeatObserver, BPMObserver  {  
    BeatModelInterface model;  
    ControllerInterface controller;  
    JFrame viewFrame;  
    JPanel viewPanel;  
    BeatBar beatBar;  
    JLabel bpmOutputLabel;  
    JFrame controlFrame;  
    JPanel controlPanel;  
    JLabel bpmLabel;  
    JTextField bpmTextField;  
    JButton setBPMButton;  
    JButton increaseBPMButton;  
    JButton decreaseBPMButton;  
    JMenuBar menuBar;  
    JMenu menu;  
    JMenuItem startMenuItem;  
    JMenuItem stopMenuItem;  
  
    public DJView(ControllerInterface controller, BeatModelInterface model) {  
        this.controller = controller;  
        this.model = model;  
        model.registerObserver((BeatObserver)this);  
        model.registerObserver((BPMObserver)this);  
    }  
  
    public void createView() {  
		//View를 만드는 함수 
    }  
  
    public void createControls() {  
        //제어 요소를 구성하는 함수
    }  
  
    public void enableStopMenuItem() {  
        stopMenuItem.setEnabled(true);  
    }  
  
    public void disableStopMenuItem() {  
        stopMenuItem.setEnabled(false);  
    }  
  
    public void enableStartMenuItem() {  
        startMenuItem.setEnabled(true);  
    }  
  
    public void disableStartMenuItem() {  
        startMenuItem.setEnabled(false);  
    }  
  
    public void actionPerformed(ActionEvent event) {  
        if (event.getSource() == setBPMButton) {  
            int bpm = 90;  
            String bpmText = bpmTextField.getText();  
            if (bpmText == null || bpmText.contentEquals("")) {  
                bpm = 90;  
            } else {  
                bpm = Integer.parseInt(bpmTextField.getText());  
            }  
            controller.setBPM(bpm);  
        } else if (event.getSource() == increaseBPMButton) {  
            controller.increaseBPM();  
        } else if (event.getSource() == decreaseBPMButton) {  
            controller.decreaseBPM();  
        }  
    }  
  
    public void updateBPM() {  
        if (model != null) {  
            int bpm = model.getBPM();  
            if (bpm == 0) {  
                if (bpmOutputLabel != null) {  
                    bpmOutputLabel.setText("offline");  
                }  
            } else {  
                if (bpmOutputLabel != null) {  
                    bpmOutputLabel.setText("Current BPM: " + model.getBPM());  
                }  
            }  
        }  
    }  
  
    public void updateBeat() {  
        if (beatBar != null) {  
            beatBar.setValue(100);  
        }  
    }  
}
```

### Controller 만들기

다음은 `View`와 `Model` 사이에서 중재자 역할을 할 Controller 를 만들어 보겠습니다.

Controller 역시 재사용성과 확장성을 고려하여 Interface로 틀을 먼저 만듭니다.

🗅 **<span style="color: #c03a92">interface ControllerInterface</span>**

```java
package src.combine_mvc.controller;  
  
public interface ControllerInterface {  
    void start();  
    void stop();  
    void increaseBPM();  
    void decreaseBPM();  
    void setBPM(int bpm);  
}
```

그 후에, 해당 Interface 를 상속하여 Controller 클래스를 만듭니다.

Controller 는 생성자에서 View 를 만드는 코드를 가지고 있습니다. 해당 클래스는 View 와 Model 에게 명령을 전달하는 명령 집합체 라고 생각하시면 됩니다.

예를들어, 어플리케이션이 Stop 되어야 한다면, Stop 시에 model 과 view 가 해야할 행동들이 있을 겁니다. 해당 행동들을 함수로 만들어 놓았다고 생각하면 됩니다.

🗅 **<span style="color: #c03a92">class BeatController</span>**

```java
package src.combine_mvc.controller;  
  
import src.combine_mvc.model.BeatModelInterface;  
import src.combine_mvc.view.DJView;  
  
public class BeatController implements ControllerInterface{  
    private BeatModelInterface model;  
    private DJView view;  
  
    public BeatController(BeatModelInterface model) {  
        this.model = model;  
        view = new DJView(this, model);  
        view.createView();  
        view.createControls();  
        view.disableStopMenuItem();  
        view.enableStartMenuItem();  
        model.initialize();  
    }  
  
    public void start() {  
        model.on();  
        view.disableStartMenuItem();  
        view.enableStopMenuItem();  
    }  
  
    public void stop() {  
        model.off();  
        view.disableStopMenuItem();  
        view.enableStartMenuItem();  
    }  
  
    public void increaseBPM() {  
        int bpm = model.getBPM();  
        model.setBPM(bpm + 1);  
    }  
  
    public void decreaseBPM() {  
        int bpm = model.getBPM();  
        model.setBPM(bpm - 1);  
    }  
  
    public void setBPM(int bpm) {  
        model.setBPM(bpm);  
    }  
}
```


### Main 에서 실행

이제 MVC 패턴을 완성 하였으니 Main 에서 실행을 해봅니다.

위에서 말했듯, Controller 의 생성자에서 view 를 생성하도록 만들어 놨기 때문에, view를 따로 생성하거나, Model 을 초기화 하는 코드는 Main에서 없습니다.

🗅 **<span style="color: #c03a92">class DJTestDrive</span>**

```java
package src.combine_mvc;  
  
import src.combine_mvc.controller.BeatController;  
import src.combine_mvc.controller.ControllerInterface;  
import src.combine_mvc.model.BeatModel;  
import src.combine_mvc.model.BeatModelInterface;  
  
public class DJTestDrive {  
    public static void main(String[] args) {  
        BeatModelInterface model = new BeatModel();  
        ControllerInterface controller = new BeatController(model);  
    }  
}
```

모델과 컨트롤러를 만들고, Controller 에 모델을 DI 해주면 끝입니다.

## 정리

언젠가 잠깐 공부해본 `Spring Boot` 라는 프레임워크에서 MVC 패턴을 구현하면서 Model과 Controller에 인터페이스를 사용하는 이유를 고민해보았습니다. 그때는 이 짓거리를 왜 하나? 했었는데, 직접 MVC 를 만들어보니 그 이유를 조금은 알 것 같습니다.

 결합도를 낮추고 유지 보수를 용이하게 하는 데 인터페이스가 중요한 역할을 한다는 점은 이해가 됩니다. 하지만, 여전히 궁금한 점은, 어디까지 인터페이스로 나누어야 하고, 어떤 부분은 구체적인 클래스로 작성하는 것이 좋은지입니다.
 
책에서, 위 궁금증과 관련된 예제를 만들어 놓긴 했습니다. 음악의 Beat 가 아닌 사람의 심장 Beat를 측정하는 어플리케이션을 만들어놓았던 `BeatModelInterface`와 `ControllerInterface`를 활용해서 만드는 과정이었습니다. 책에서는 아주 간단하게 구현을 해놓아서, 만들어놓았던 인터페이스들을 사용하는데 무리가 없어 보였지만, 인터페이스에 보면 `on()` 과 `off()` 라는 함수가 있는데, 음악의 Beat는 끌 수 있고, 사람의 심장 Beat는 끌 수 없다라는 점에서 인터페이스를 굳이 사용해야 하는가? 에 대한 의문이 들었습니다.

프로젝트의 요구사항과 규모에 따라 달라지겠지만, 이러한 선택의 기준에 대해서는 더 많은 경험이 필요할 것 같습니다.