---
layout : single
title : "[JAVA] kiosk 프로그램 실습"
categories: JAVA-Learn
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: true
---

## 처음 짠 코드

1. kiosk class
    
    ```java
    public class Kiosk {
        Scanner sc = new Scanner(System.in);
        private static final MenuItem menuItem1 = new MenuItem("김밥", 1000);
        private static final MenuItem menuItem2 = new MenuItem("계란 김밥", 1500);
        private static final MenuItem menuItem3 = new MenuItem("충무 김밥", 1000);
        private static final MenuItem menuItem4 = new MenuItem("떡볶이", 2000);
    
        public void orderInfo(){
                System.out.println("=".repeat(50));
                System.out.println("[안내] : Welcome ~ 제리김밥에 오신 것을 환영합니다.");
                System.out.println("주문을 시작하시겠습니까?");
               // System.out.println("주문을 원하지 않으시면 1을 눌러주세요");
                System.out.println("=".repeat(50));
    
        }
    
        public void selectMenu() {
                System.out.println("[안내] : 원하시는 메뉴의 번호를 입력하여 주세요.숫자만 입력해주세요");
                System.out.println("1) 김밥(1000원) 2) 계란 김밥(1500원) 3) 충무 김밥(1000원) 4) 떡볶이(2000원)");
        }
        
       public String orderMenu(int num){
                String selectMenu="";
                switch(num){
                    case 1: selectMenu=menuItem1.getName();
                            break;
                    case 2: selectMenu=menuItem2.getName();
                            break;
                    case 3: selectMenu=menuItem3.getName();
                            break;
                    case 4: selectMenu=menuItem4.getName();
                            break;
                }
            return selectMenu;
        }
    
        // 주문음식 수량입력
        public void menuQuant(){
                System.out.printf("[안내]:선택하신 [메뉴]의 주문 수량을 입력하여 주세요.%n");
                System.out.println("(※ 최대 주문 가능 수량 : 99)");
    
        }
           //선택한 메뉴의 가격 Method
         public int orderMenuPrice(int num){
                int price=0;
                switch(num){
                    case 1: price = menuItem1.getPrice();
                    break;
                    case 2: price = menuItem2.getPrice();
                    break;
                    case 3: price = menuItem3.getPrice();
                    break;
                    case 4: price = menuItem4.getPrice();
                    break;
                }
                return price;
            }
    
        // 주문 결과를 출력
        public void orderDisplay(int menuNum,int selectQuant){
            System.out.println("=".repeat(15));
            System.out.println("주문하신 결과는 아래와 같습니다.");
            System.out.printf("주문하신 메뉴는 %s이며,수량은 %d개 입니다.%n",orderMenu(menuNum),selectQuant);
            System.out.printf("총 가격은 %d원 입니다.%n",orderMenuPrice(menuNum)*selectQuant);
            System.out.println("결제를 진행해주세요. 감사합니다.");
    
        }
    }
    ```
    
    - 사용자에게 주문할 때 사용자에게 보여주기 위해 내부에서 구현되는 클래스로 만듬
    - 또한 리턴 값이 모두 원시타입으로, 객체로 반환해서 더 객체지향적으로 코드를 짜야 할 필요가 있음. ⇒ 아래 메뉴아이템 클래스가 있으나 활용도가 낮은 것 같음
2. menu class
    
    ```java
    public class MenuItem {
        private String name;
        private int price;
    
        public MenuItem(String name, int price) {
            this.name = name;
            this.price = price;
        }
    
        public String getName() {
            return name;
        }
    
        public int getPrice() {
            return price;
        }
    }
    ```
    
    - 메뉴에 대한 클래스로 이름과 가격을 getter를 사용하여 호출 가능
3. application class
    
    ```java
    public class KioskApplication {
        Kiosk order = new Kiosk();
    
        public static void main(String[] args) {
            Scanner sc = new Scanner(System.in);
            Kiosk kioskfirst = new Kiosk();
            kioskfirst.orderInfo();
            
    				//메뉴선택 Method
            int selectNum;
            boolean result;
            do{
            kioskfirst.selectMenu();
            selectNum = Integer.parseInt(sc.nextLine().trim());
            kioskfirst.orderMenu(selectNum);
                if(!isValidNumber(selectNum)){
                System.out.println("[안내] : 메뉴에 포함된 번호를 입력하여 주세요.");
                    result=true;
                }else{
                    result=false;
                }
            }while(result);
            
    			 //수량입력 Method
            int selecQuant;
            do {
                kioskfirst.menuQuant();
                selecQuant = Integer.parseInt(sc.nextLine().trim());
    
                if(selecQuant>=100){
                    System.out.println("[경고] 100개는 입력하실 수 없습니다.");
                    System.out.println("[경고] 수량 선택 화면으로 돌아갑니다.");
                }
            } while(selecQuant>=100);
    
            //결제금액 출력 Method
            kioskfirst.orderDisplay(selectNum, selecQuant);
    
        }
    
        public static boolean isValidNumber(int number){
            if(number!=0){
                    if(1<=number&&number<=4){
                        return true;
                    }
               }
           return false;
        }
    ```
    
4. 부족한 부분
    - 키오스크가 해야할 역할 , 어플리케이션이 해야할 역할 구분하기
    - 객체 생성 이용하기. (ex) 키오스크의 MenuItem으로 반환하는 메서드 만들기
    - 먹고갈건지 포장할건지 , 주문을 안할거면 돌아가기 기능 추가해보고 싶음

## 다시 푼 문제

1. kiosk class
    - selectMenu 메서드 수정 - 유효성 검증 기능을 추가하고 객체로 리턴함
    - menuQuant 메서드 수정 - 기존에는 수량만 입력받고 99개 이상일 경우 다시 입력은 kioskapplication에서 역할을 진행했다면 수정은 kiosk에서 다할 수 있도록 함. kiosk 클래스는 관리자가 수행하는 것이라면 kiosk application은 고객과 키오스크가 다루는 것으로 분리시켜서 생각해야 함. 고로 유효성 검증도 kiosk에 있는 것이 맞음.
    - displayorder 메서드 수정 - 메뉴의 이름과 가격을 getter로 가져옴
    - application의 isValidNumber 메서드 (숫자 유효 검사 기능) 이동.
    
    ```java
     public MenuItem selectMenu() {
            int selectNum;
            boolean isTrue;
            MenuItem menuItem=null;
            do {
                System.out.println("[안내] : 원하시는 메뉴의 번호를 입력하여 주세요.숫자만 입력해주세요");
                System.out.println("1) 김밥(1000원) 2) 계란 김밥(1500원) 3) 충무 김밥(1000원) 4) 떡볶이(2000원)");
                //선택한 숫자가 유효한 숫자인지 검증하기
                selectNum = Integer.parseInt(sc.nextLine().trim());
                if(!isValidNumber(selectNum)){
                    System.out.println("[안내] : 메뉴에 포함된 번호를 입력하여 주세요.");
                    isTrue=false;
                }else{
                    isTrue = true;
                    switch(selectNum) {
                        case 1:
                            menuItem=menuItem1;
                        break;
                        case 2:
                            menuItem=menuItem2;
                        break;
                        case 3:
                            menuItem=menuItem3;
                        break;
                        case 4:
                            menuItem=menuItem4;
                        break;
                    }
                }
    
            }while(!isTrue);
    
           return menuItem;
        }
    
    	public int menuQuant(){
            int selectQuant;
            do {
                System.out.printf("[안내]:선택하신 [메뉴]의 주문 수량을 입력하여 주세요.%n");
                System.out.println("(※ 최대 주문 가능 수량 : 99)");
                selectQuant = Integer.parseInt(sc.nextLine().trim());
    
                if(selectQuant>=100){
                    System.out.println("[경고] 100개는 입력하실 수 없습니다.");
                    System.out.println("[경고] 수량 선택 화면으로 돌아갑니다.");
                }
            } while(selectQuant>=100);
    
            return selectQuant;
    	}
    	
     public void displayOrder(MenuItem menuItem,int menuCount){
            System.out.println("=".repeat(50));
            System.out.println("주문하신 결과는 아래와 같습니다.");
            System.out.printf("주문하신 메뉴는 %s이며,수량은 %d개 입니다.%n",menuItem.getName(),menuCount);
            System.out.printf("총 가격은 %d원 입니다.%n",menuItem.getPrice()*menuCount);
            System.out.println("결제를 진행해주세요. 감사합니다.");
        }
      
     //유효성 검증
     public boolean isValidNumber(int number){
            if(number!=0){
                if(1<=number&&number<=4){
                    return true;
                }
            }
            return false;
     }
    
    ```
    
2. menu class = 변경사항 없음
3. application class
    - 외부 고객에게 보여주는 코드가 간략해짐, 내부 기능을 가진 메서드는 kiosk 클래스로 이동시킴
    
    ```java
    public class KioskApplication {
        public static void main(String[] args) {
            Kiosk kiosk=new Kiosk();
    
            //주문시작 + 주문메세지
            kiosk.orderInfo();
            MenuItem menuItem= kiosk.selectMenu();
            int menuCount=kiosk.menuQuant();
    
            //주문 결과 출력
            kiosk.displayOrder(menuItem,menuCount);
    
        }
    }
    
    ```
    

## Comment

강사님 풀이는 V1~V5까지 단계별로 주셨고

내가 푼 풀이는 V3 정도 인 것 같다.

V5는 개발하다 오신 분들도 풀기 힘들거라고 하셨고 V5 로 푼다면

수업듣지말고 이력서부터 쓰라고 하심.ㅠㅠㅠ부럽...그런사람은 없었지만

V3은 내가 푼 풀이랑 거의 비슷한데

나는 메서드의 기능을 다 분리하진 않았다

강사님은 간단한 기능도 다 분리해서 화면에 출력되는 건 다 따로 빼고

계산하는 기능, 사용자가 입력하는 기능 까지 모조리 분리하였는데

그것만하면 V3랑 거의 비슷한듯.

메뉴는 상수로 만들었는데 만약 메뉴가 많아질 경우 일일이 추가해야 된다는 단점이 있음..

또한 메뉴를 추가하는 것은 쉬운 일이나 메뉴를 수정해야 할 경우switch문을 변경하는 번거로움 도 발생해서 배운 것 중에 메뉴를 배열로 받는 방법을 활용하거나 Enum을 활용하는 방법도 있으니 생각해보기!