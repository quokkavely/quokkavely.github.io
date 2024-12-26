---
layout : single
title : "[디자인패턴] MVC, MVP, MVVM"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# **MVC, MVP, MVVM Pattern**

| 특징 | MVC | MVP | MVVM |
| --- | --- | --- | --- |
| 관계 | Controller ↔ 여러 View | Presenter ↔ 단일 View | ViewModel ↔ 여러 View |
| 역할 분리 | Model-View-Controller | Model-View-Presenter | Model-View-ViewModel |
| 데이터 전달 방식 | Controller ↔ View, Model | Presenter ↔ View, Model | Data Binding |
| 대표 기술 | Spring MVC | Android MVP | Angular, Vue.js |

## **1️⃣ MVC 패턴**

- MVC(Model-View-Controller)는 애플리케이션을 **모델(Model)**, **뷰(View)**, **컨트롤러(Controller)**로 나누어 개발하는 디자인 패턴

### **구성 요소**

1. **Model**: 애플리케이션의 데이터와 비즈니스 로직을 담당
    - 데이터베이스, 상수, 변수 등 포함
    - 데이터 변경 시 뷰(View)에 알림을 보냄
    - 예: 글자 정보, 크기, 위치 등
2. **View**: 사용자 인터페이스(UI)를 담당
    - 사용자가 볼 수 있는 화면
    - 모델 데이터 기반으로 UI를 구성하며, 데이터 저장은 하지 않음
    - 변경 이벤트를 컨트롤러(Controller)에 전달
3. **Controller**: 모델과 뷰 사이에서 중재 역할
    - 입력 이벤트를 처리하고 모델을 업데이트하거나 뷰를 갱신

### **MVC 패턴의 장단점**

- **장점**
    1. 역할 분리가 명확하여 코드 재사용성과 유지보수성이 뛰어남
    2. 확장성과 모듈화 용이
- **단점**
    1. 애플리케이션이 커질수록 모델과 뷰의 의존성이 증가
    2. 이벤트 흐름이 복잡해질 수 있음

## **2️⃣ MVP 패턴**

- **MVP(Model-View-Presenter)**는 MVC 패턴의 변형으로, **Controller** 대신 **Presenter**를 사용

### **구성 요소**

1. **Model**: 데이터 및 비즈니스 로직 처리 (MVC와 동일)
2. **View**: 사용자 인터페이스(UI)
    - Presenter와 1:1 관계를 유지하며, Presenter를 호출하여 데이터를 갱신
3. **Presenter**: 뷰와 모델 간의 중재자 역할
    - 뷰에 의존성이 생기므로 뷰와 강한 결합을 가짐

### **MVP 패턴의 특징**

- **View** 와 **Presenter 1:1 관계**를 가짐
- **View**는 **Presenter**를 참조하며, **Presenter**는 모델 데이터를 갱신하고 **View**에 결과를 전달

## **3️⃣ MVVM 패턴**

- **MVVM(Model-View-ViewModel)**는 MVC 패턴의 또 다른 변형으로, **Controller** 대신 **ViewModel**을 사용

### **구성 요소**

1. **Model**: 데이터 및 비즈니스 로직 처리
2. **View**: 사용자 인터페이스(UI)
    - 데이터 바인딩(Data Binding)을 통해 ViewModel과 연결
3. **ViewModel**: 뷰의 상태와 동작을 추상화
    - 뷰와 1:N 관계를 가질 수 있음
    - 커맨드 패턴을 통해 이벤트 처리

### **MVVM 패턴의 특징**

- **데이터 바인딩**: UI와 데이터 상태가 자동으로 동기화됨
- **커맨드 패턴**: 사용자 입력을 처리하기 위한 메서드 추상화

## **Java로 구현한 MVC, MVP, MVVM 예제**

### **1. MVC 패턴 예제**

```java
// Model
class Product {
    private String name;

    public Product(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

// View
class ProductView {
    public void displayProduct(String productName) {
        System.out.println("Product: " + productName);
    }
}

// Controller
class ProductController {
    private Product model;
    private ProductView view;

    public ProductController(Product model, ProductView view) {
        this.model = model;
        this.view = view;
    }

    public void updateView() {
        view.displayProduct(model.getName());
    }
}

// Main
public class MVCPattern {
    public static void main(String[] args) {
        Product product = new Product("Laptop");
        ProductView view = new ProductView();
        ProductController controller = new ProductController(product, view);

        controller.updateView();
    }
}

```

### **2. MVP 패턴 예제**

```java
// Model
class User {
    private String name;

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

// View
interface UserView {
    void showUserName(String name);
}

// Presenter
class UserPresenter {
    private User model;
    private UserView view;

    public UserPresenter(User model, UserView view) {
        this.model = model;
        this.view = view;
    }

    public void loadUser() {
        view.showUserName(model.getName());
    }
}

// Main
public class MVPPattern {
    public static void main(String[] args) {
        User user = new User("Alice");
        UserView view = name -> System.out.println("User: " + name);
        UserPresenter presenter = new UserPresenter(user, view);

        presenter.loadUser();
    }
}

```

### **3. MVVM 패턴 예제**

```java
import java.beans.PropertyChangeListener;
import java.beans.PropertyChangeSupport;

// Model
class Task {
    private String name;

    public Task(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

// ViewModel
class TaskViewModel {
    private Task model;
    private PropertyChangeSupport support;

    public TaskViewModel(Task model) {
        this.model = model;
        this.support = new PropertyChangeSupport(this);
    }

    public String getTaskName() {
        return model.getName();
    }

    public void updateTaskName(String newName) {
        String oldName = model.getName();
        model = new Task(newName);
        support.firePropertyChange("taskName", oldName, newName);
    }

    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }
}

// View
public class MVVMPattern {
    public static void main(String[] args) {
        Task task = new Task("Write Code");
        TaskViewModel viewModel = new TaskViewModel(task);

        viewModel.addPropertyChangeListener(evt -> {
            System.out.println("Task updated: " + evt.getOldValue() + " -> " + evt.getNewValue());
        });

        System.out.println("Initial Task: " + viewModel.getTaskName());
        viewModel.updateTaskName("Test Code");
    }
}

```

---

1. **MVC**: 가장 기본적이며 역할 분리가 명확
2. **MVP**: 뷰와 강한 결합, 단순 애플리케이션에 적합
3. **MVVM**: 데이터 바인딩으로 효율적이며, 대규모 애플리케이션에서 사용

위 패턴 중 애플리케이션의 **복잡성**, **확장성**, **유지보수성**을 고려해 적합한 패턴을 선택해야 한다.


<br>
<br><br><br>