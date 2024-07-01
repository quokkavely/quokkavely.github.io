---
layout : single
title : "[JAVA] 미니 텍스트 게임 만들기 "
categories: JAVA-Practice
tag : [JAVA, 실습]
toc : true
toc_sticky : true
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## 미니 텍스트 게임 만들기

- unit1과 unit2의 정보를 입력 받은 후 게임을 실행한다.
- 조건문, 반복문, 연산자가 주요 기능으로 사용됨.

### 처음 시도

- 내가 처음 구현한 코드는 캐릭터에 대하 정보를 입력받는 클래스와 게임이 진행되는 클래스 총 2개로 나누고 캐릭터 클래스에  플레이어 정보에 대해 담고 플레이어 생성과 플레이어 정보 출력까지 했는데 공격 메서드를 구현하기 힘들었음
- 문제를 이해못한건지 내가 구현을 못한건지… 아마 그 두개 다가 맞음…
- 일단 공격을 어떻게 진행해야할지 몰랐음. 내가 공격버튼을 눌러야 공격을 하는게 아닌가 싶었고, 그럴려면 계속 입력받아야할텐데… 그럼 다른 플레이어는 고정값이 있는건지? 랜던값을 넣어서 공격이 진행되는건가 싶었음….
1. LoL_char 클래스
    
    ```java
    import java.util.Scanner;
    
    class LoL_char {
      public String unitName;
      public String attackForce;
      public String defensiveForce;
      public String strength;
    
    	void user_info(String name, String af, String def, String hp) {
    
        this.unitName = name;
        this.attackForce = af;
        this.defensiveForce = def;
        this.strength = hp;
    
      }
    
     // 유닛을 생성하는 메서드
      String[] user_create() {
        Scanner sc = new Scanner(System.in);
        System.out.print("[시스템] 유닛 [이름] 을 입력해 주세요: ");
        unitName= sc.nextLine();
        System.out.print("[시스템] 유닛 [공격력] 을 입력해 주세요 : ");
        attackForce= sc.nextLine();
        System.out.print("[시스템] 유닛 [방어력] 을 입력해 주세요 : ");
        defensiveForce= sc.nextLine();
        System.out.print("[시스템] 유닛 [체력] 을 입력해 주세요 : ");
        strength= sc.nextLine();
        System.out.println("============================================");
        String[] userOne={unitName,attackForce,defensiveForce,strength};
        user_info(unitName,attackForce,defensiveForce,strength);
        return userOne;
      }
      
      void user_print(String[] user) {
    
        System.out.printf("[안내] 생성된 유닛 정보는 다음과 같습니다.%n");
        System.out.printf("[안내] %s 유닛이 게임에 참여하였습니다.%n", this.unitName);
        System.out.printf("[공격력] : %s%n", this.attackForce);
        System.out.printf("[방어력] : %s%n", this.defensiveForce);
        System.out.printf("[체력] : %s%n", this.strength);
        System.out.println("============================================");
      }
    
        int[] user_info_int(String[]info){
    
          info=user_create();
          int[] newArr = new int[info.length-1];
          for (int i = 0; i < info.length-1; i++) {
            newArr[i] = Integer.parseInt(info[i+1]);
          }
          return newArr;
        }
    
     //공격을 수행 메서드
      
      void attack(int[] me_info_int, int[] enemy) {
        // 조건 1. 적군의 체력이 0 이하면 유닛 제거됨
            //TODO:
            enemy[3]-=(me_info_int[2]/enemy[2]);
          if(enemy[3]==0){
    
            System.out.println("상대 유닛의 남은 [체력]은 0입니다.");
            System.out.println("[안내] 더 이상 공격할 수 없습니다.");
            System.out.println("상대 유닛이 제거되었습니다.");
          }else{
    
            System.out.printf("[안내] [레이스]유닛이 [공격] 하였습니다.%n");
            System.out.printf("[안내] 상대 유닛의 남은 [체력]은 %d 입니다.%n",enemy[3]);
    }}}
    ```
    
    공격은 구현 똑바로 못함..
    
2. 게임 프로그램 클래스
    
    ```java
    public class LOL_Program {
      public static void main(String[] args) {
        //TODO:
        System.out.println("[안내] TRPG 스타크래프트 시작합니다.");
        LoL_char jerry = new LoL_char();
        LoL_char TOM  = new LoL_char();
        System.out.println("[안내] 자신의 유닛 정보를 입력해 주세요.");
        jerry.user_print(jerry.user_create());
        System.out.println("[안내] 상대 유닛 정보를 입력해 주세요.");
        TOM.user_print(TOM.user_create());
        jerry.attack();
    
      }
    }
    ```
    
    - 캐릭터 변수명을 톰과 제리로 만든 후  두 플레이어가 생성되면 유닛 저보들을 입력 후  동시에 싸움을 할 수 있도록 하려고 했음.
    - 공격 로직은 도무지 이해가 안가서 강사님께 메서드나 공격논리에 대해 듣고
    - 일부분만 수정해봄.

### 2번째 시도

1. 게임 프로그램 클래스
    
    ```java
       String[] user_create() {
        String[] userOne = new String[4];
    
        Scanner sc = new Scanner(System.in);
        System.out.print("[시스템] 유닛 [이름] 을 입력해 주세요 : ");
        userOne[0] = sc.nextLine();
        System.out.print("[시스템] 유닛 [공격력] 을 입력해 주세요 :");
        userOne[1] = sc.nextLine();
        System.out.print("[시스템] 유닛 [방어력] 을 입력해 주세요 :");
        userOne[2] = sc.nextLine();
        System.out.print("[시스템] 유닛 [체력] 을 입력해 주세요 :");
        userOne[3] = sc.nextLine();
        System.out.println("=========================================");
    
        user_info(userOne[0], userOne[1], userOne[2], userOne[3]);
        return userOne;
      }
    
      void attack(int[] me_info_int, int[] enemy) {
        //서로 싸우려면 메서드 하나 더만들어야하는지?
    
        do {
          if ((enemy[2] - me_info_int[0] / enemy[1]) < 0) {
            System.out.printf("[안내] [%s]유닛이 [공격] 하였습니다.%n", this.name);
            System.out.println("[안내] 상대 유닛의 남은 [체력]은 0 입니다.");
            System.out.println("--------------------------");
            System.out.println("[안내] 더 이상 공격할 수 없습니다.");
            System.out.println("[안내] 상대 유닛이 제거되었습니다.");
          } else {
            System.out.printf("[안내] [%s]유닛이 [공격] 하였습니다.%n", this.name);
            System.out.printf("[안내] 상대 유닛의 남은 [체력]은 %d 입니다.%n", enemy[2] - me_info_int[0] / enemy[1]);
            enemy[2] -= me_info_int[0] / enemy[1];
          }
        } while (enemy[2] != 0);
    
      }
    }
    ```
    
    - 그래서 프로그램은 마지막에 do-while문 사용해서 죽을 때 까지 반복하는 방식으로 구현함.
    - 둘 중에 체력이 다 소진되는 사람이 먼저 죽는 걸로 셋팅함.
2. 게임프로그램 클래스
    
    ```java
     LoL_char user1 =new LoL_char();
     String[]userOne=user1.user_create();
     user1.user_print(userOne);
    
     System.out.println("[안내] 상대 유닛 정보를 입력해 주세요.");
     LoL_char user2 =new LoL_char();
     String[]userTwo=user2.user_create();
     user2.user_print(userTwo);
    
     int[]userOneNum=user1.user_info_int(userOne);
     int[]userTwoNum=user2.user_info_int(userTwo);
     user1.attack(userOneNum, userTwoNum);
    ```
    
    - user_info_int() 메서드 호출 추가하여 배열로 받음
3. 추가 구현해야 할 기능
    1. 객체지향 적으로 풀어야 함, 하나의 메서드는 하나의 기능만  그리고 클래스도 비슷한 기능의 메서드만 집합한 것으로. (프로그램 진행되는 클래스 / 프로그램 내부관련 클래스/플레이어 정보에 대한 클래스)
    2. 게임이 시작하면 바로 끝나버림 → 때릴 때마다 얼마 때리고 체력 얼마 남았고 이런 기능이 있어야 함.

### 3번째 시도

1. Player class
    
    ```java
    public class Player {
            public String name;
            public int af;
            public int def;
            public int hp;
    
        public Player(String name,int af,int def, int hp){
            this.name=name;
            this.af=af;
            this.def=def;
            this.hp=hp;
        }
    
        public String getName(){
            return name;
        }
    
        public int getAf(){
            return af;
        }
    
        public int getDef(){
            return def;
        }
        public void setHp(int hp){
            this.hp=hp;
        }
        public int getHp(){
            return hp;
        }
    
    }
    
    ```
    
    - player class에서는 player의 입력받을 정보들을 선언 후 생성자를 통해 초기화 함.
    - 네 개 정보를 getter 사용 그 다음  체력만 수정이 필요해서 setter도 사용,
2. RPGgameApplicatio class
    
    ```java
    import java.util.Scanner;
    
    public class LOL_Program {
    public static Scanner sc= new Scanner(System.in);
     public static void main(String[] args) {
    
          String str="[안내] 자신의 유닛 정보를 입력해 주세요.";
    
          Player player1 = inputPlayerInfo(str)
          displayPlayerInfo(player1);
    
          Player player2=inputPlayerInfo(str);
          displayPlayerInfo(player2);
          
          playGame(player1,player2);
    
      }
    
      public static void playGame(Player player1,Player player2) {
          LOL_Management game = new LOL_Management(player1, player2);
                game.playGame();
             }
    
        public static Player inputPlayerInfo(String str) {
            System.out.println("=".repeat(50));
            System.out.println(str);
            System.out.println("[시스템] 유닛 [이름] 을 입력해 주세요 :");
            String unitName=sc.nextLine();
            System.out.println("[시스템] 유닛 [공격력] 을 입력해 주세요 :");
            int ap=Integer.parseInt(sc.nextLine());
            System.out.println("[시스템] 유닛 [방어력] 을 입력해 주세요 :");
            int def=Integer.parseInt(sc.nextLine());
            System.out.println("[시스템] 유닛 [체력] 을 입력해 주세요 :");
            int hp= Integer.parseInt(sc.nextLine()
            System.out.println("=".repeat(50));
            return new Player(unitName,ap,def,hp);
    }
    
        publi static void displayPlayerInfo(Player player){
            System.out.println("=".repeat(50));
            System.out.println("[안내] 생성된 유닛 정보는 다음과 같습니다.");
            System.out.printf("[안내] %s 유닛이 게임에 참여하였습니다.",player.getName());
            System.out.printf("[공격력] :%d",player.getAf());
            System.out.printf("[방어력] :%d",player.getDef());
            System.out.printf("[체력] :%d",player.getHp());
            System.out.println("=".repeat(50));
        }
    
    }
    ```
    
    - 유닛 정보를 입력 및 출력 그리고 게임 실행하는 메서드를 application을 따로 만들어서 구현해봤음.
    - player에 입력한 정보를 객체(Player)로 저장하고 getter를 이용하여 정보를 받아옴
    - 기존에는 배열을 만들고 배열로 리턴 했다면 이번에는 객체로 리턴함
        
        (객체지향적으로 풀기!, 사실 잘 몰라서 강사님 풀이를 외우고 구현해보니 이해가 감…)
        
    
3. RPGgame class
    
    ```java
    public class LOL_Management {
        private final Player player1;
        private final Player player2;
    
        public LOL_Management(Player player1,Player player2){
            this.player1=player1;
            this.player2=player2;
        }
    
        //게임플레이
        public void playGame() {
    
            while (player1.getHp() > 0 && player2.getHp() > 0) {
                int healthPowerPlayer1 = player1.getHp();
                int healthPowerPlayer2 = player2.getHp();
                int defensePowerPlayer1 = player1.getDef();
                int defensePowerPlayer2 = player2.getDef();
                int attackPowerPlayer1 = player1.getAf();
                int attackPowerPlayer2 = player2.getAf();
    
                System.out.println("=".repeat(50));
                System.out.printf("[안내] [%s]유닛이 [%s]유닛을 [공격] 하였습니다.%n", player1.getName(), player2.getName());
                player2.setHp(healthPowerPlayer2 - getAttackPoint(attackPowerPlayer1, defensePowerPlayer2));
                if (player1.getHp() > 0) {
                    System.out.printf("[안내] %s 유닛의 남은 [체력]은 %d입니다.%n", player2.getName(), player2.getHp());
                     }
                System.out.println("=".repeat(50));
                System.out.printf("[안내] [%s]유닛이 [%s]유닛을 [공격] 하였습니다.%n", player2.getName(), player1.getName());
                player1.setHp(healthPowerPlayer1 - getAttackPoint(attackPowerPlayer2, defensePowerPlayer1));
                if (player1.getHp() > 0) {
                    System.out.printf("[안내] %s 유닛의 남은 [체력]은 %d입니다.%n", player1.getName(), player1.getHp());
                    }
            }
            //죽었을때
            System.out.println("=".repeat(50));
            String currentDeathPlayerName = player1.getHp() < 0 ? player1.getName() : player2.getName();
            System.out.printf("[안내] %s 유닛이 사망했습니다.%n", currentDeathPlayerName);
            System.out.println("[안내] 게임이 종료되었습니다.");
        }
        //공격력방어력구현
            private int getAttackPoint(int player1Af, int player2Def){
            return player1Af/player2Def;
        }
    }
    
    ```
    
    - 게임 진행에 관련된 플레이어 1,2의 정보(체력,공격력…) 를 getter 사용하여 모두 받아온 후 player1이 먼저 공격하고 그 다음 2가 공격해서 남은 체력이  모두 없어지면 죽는다, 그리고 while문이 끝났다는 건 죽었다는 뜻이므로 while문 밖에서 삼항연산자로 죽은 player를 알림.
    - 공격으로 인한 체력 데미지를 계산하는 값은 따로 메서드로 만들어서 구현.

 

### Comment

와 처음에는 진짜 너무 모르겠어서 한참 헤맸다…다들 타자소리는 많이 나는데 나는 하나도 못치고. 속상해서 집에서도 계속 생각해도 모르겠어서 강사님께 도저히 이해가 안간다고도 해보고 했는데

다음날 강사님 풀이듣는데 금방 구현해내는 강사님 보면서,, 아 저거 나도 할 줄 아는데 싶었다...

아직은 익숙하지 않아서 코드를 구현해내는게 오래걸리고

어떻게 써야하는지 막막하지만 연습만이 답이겠지.

메서드라는 개념이 이제야 조금 잡히고 클래스라는게 이제야 어떤식으로 사용하는건지 감이 잡히고

하다보니 조금씩 풀었던 것 같다. 강사님께서 getter,setter를 사용하는 걸 알려주셨고 왜 쓰는지는 바로 이해 못했지만 조금 더 연습해봐야 할 것 같다

강사님 풀이보고 3번째 시도한거라 강사님 코드와 거의 유사하게 풀었지만 이걸보고 완벽히 이해하고 백지에서도 새롭게 구현해내는 것을 연습해야겠지!

아직 많이 안해봐서가 맞으니 너무 낙심하지말자

<br/>
<br/>

---

<br/>
<br/>
<br/>
<br/>
<br/>