---
layout : single
title : "[디자인패턴] flux"
categories: DesignPattern
tag : [CS, DesignPattern]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

# **Flux 패턴: 단방향 데이터 흐름 관리**

## **Flux 패턴이란?**

Flux는 **단방향 데이터 흐름**을 통해 애플리케이션 상태를 관리하는 디자인 패턴

**페이스북**이 개발한 이 패턴은 복잡한 애플리케이션에서 데이터 일관성을 유지하고 버그를 줄이는 데 유용하다.

1. **Flux 패턴의 필요성**
    - **기존 패턴의 문제**
        - MVC 같은 양방향 데이터 흐름 패턴에서는 모델과 뷰 사이의 관계가 복잡해져 데이터 일관성이 깨질 가능성이 높음.
        - 데이터 흐름을 추적하기 어려워 디버깅 및 유지보수가 힘듦
        - 복잡한 로직으로 인해 상태 관리가 어려워짐
2. **Flux의 해결책**
    - 데이터가 **단방향**으로만 흐르도록 강제하여, 상태 변화를 예측 가능하게 만듦
    - 모든 데이터 변경은 **Action → Dispatcher → Store → View**를 통해 이루어짐

### **Flux 패턴의 구성 요소**

1. **Action**
    - 사용자의 이벤트를 캡처하여 이를 설명하는 **객체**
    - ex ) 클릭, 입력 등과 같은 사용자 상호작용
    - **역할**
        - Dispatcher에게 데이터와 액션 타입 전달
2. **Dispatcher**
    - Flux의 중심 역할을 하는 계층
    - Action 객체를 받아서, 적절한 로직을 Store에 전달
    - **특징**
        - 모든 Action이 Dispatcher를 거쳐야 함
        - Store에 데이터 업데이트를 요청
3. **Store**
    - **애플리케이션의 상태**를 저장하고 관리
    - 데이터를 변경할 수 있는 유일한 계층
    - View가 상태를 읽을 수 있도록 제공
4. **View**
    - 사용자 인터페이스(UI)를 나타냄
    - Store로부터 상태를 읽어와 사용자에게 렌더링
    - 사용자 입력 이벤트를 Action으로 변환해 Dispatcher로 전달

### **Flux 패턴의 동작 흐름**

1. **사용자가 이벤트를 트리거** (ex ) 버튼 클릭)
2. **Action** 객체가 생성되고, Dispatcher로 전달
3. **Dispatcher**는 Action의 타입을 기반으로 적절한 로직을 **Store**에 전달
4. **Store**는 상태를 업데이트하고, 상태 변화에 따라 View에 알림을 보냄
5. **View**는 Store에서 데이터를 읽어와 UI를 업데이트

## **Flux 패턴의 장점**

1. **데이터 일관성**
    - 모든 상태 변경이 한 방향으로만 흐르므로 데이터 일관성을 유지
2. **디버깅 용이**
    - 데이터 흐름이 예측 가능해 상태 변화를 쉽게 추적 가능
3. **유지보수 용이**
    - 각 계층이 명확히 분리되어 있어, 단위 테스트와 코드 수정이 간단

## **Java로 구현한 Flux 패턴**

- Flux 패턴은 Java에서도 적용 가능하며, 아래는 간단한 예제
    
    ```java
    import java.util.ArrayList;
    import java.util.List;
    
    // Action
    class Action {
        String type;
        Object payload;
    
        public Action(String type, Object payload) {
            this.type = type;
            this.payload = payload;
        }
    
        public String getType() {
            return type;
        }
    
        public Object getPayload() {
            return payload;
        }
    }
    
    // Dispatcher
    class Dispatcher {
        private Store store;
    
        public Dispatcher(Store store) {
            this.store = store;
        }
    
        public void dispatch(Action action) {
            store.update(action);
        }
    }
    
    // Store
    class Store {
        private List<String> todos = new ArrayList<>();
        private List<ChangeListener> listeners = new ArrayList<>();
    
        public void update(Action action) {
            switch (action.getType()) {
                case "ADD_TODO":
                    todos.add((String) action.getPayload());
                    break;
                case "REMOVE_TODO":
                    todos.remove(action.getPayload());
                    break;
            }
            notifyListeners();
        }
    
        public List<String> getTodos() {
            return todos;
        }
    
        public void addChangeListener(ChangeListener listener) {
            listeners.add(listener);
        }
    
        private void notifyListeners() {
            listeners.forEach(ChangeListener::onChange);
        }
    }
    
    // Change Listener Interface
    interface ChangeListener {
        void onChange();
    }
    
    // View
    class TodoView implements ChangeListener {
        private Store store;
    
        public TodoView(Store store) {
            this.store = store;
            this.store.addChangeListener(this);
        }
    
        public void render() {
            System.out.println("Current Todos: " + store.getTodos());
        }
    
        @Override
        public void onChange() {
            render();
        }
    }
    
    // Main
    public class FluxPattern {
        public static void main(String[] args) {
            Store store = new Store();
            Dispatcher dispatcher = new Dispatcher(store);
            TodoView view = new TodoView(store);
    
            // Initial Render
            view.render();
    
            // Dispatch Actions
            dispatcher.dispatch(new Action("ADD_TODO", "Learn Java"));
            dispatcher.dispatch(new Action("ADD_TODO", "Study Flux Pattern"));
            dispatcher.dispatch(new Action("REMOVE_TODO", "Learn Java"));
        }
    }
    
    ```
    
    1. **Action**
        - "ADD_TODO"와 "REMOVE_TODO" 타입을 정의하여, 각각 할 일을 추가/삭제
    2. **Dispatcher**
        - Action 객체를 Store로 전달
    3. **Store**
        - 애플리케이션 상태(할 일 목록)를 관리
        - 상태 변화가 발생하면 View에 알림
    4. **View**
        - Store에서 데이터를 읽어와 사용자에게 렌더링

### **Flux 패턴이 적용된 Redux**

- Flux 패턴은 React의 상태 관리 라이브러리인 **Redux**로 발전
- Redux는 Flux와 유사하지만, 단일 Store를 강제하며, **Reducer**를 통해 상태를 업데이트

---

- Flux 패턴은 데이터 흐름을 단방향으로 강제하여 애플리케이션의 상태를 효율적으로 관리.
- 데이터 일관성과 유지보수성이 뛰어나, 대규모 프로젝트에서 유용.
- Redux와 같은 프레임워크에서 Flux의 철학을 쉽게 활용할 수 있음.


<br>
<br><br><br>