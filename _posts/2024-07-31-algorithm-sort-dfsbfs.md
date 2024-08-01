---
layout : single
title : "[Algorithm] Sort, DFS & BFS"
categories: Algorithm
tag : [python, 실습, 개념]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}


## 정렬알고리즘

### 버블정렬

버블 정렬의 최악의 경우 시간 복잡도는 O(n^2)

<img src="https://github.com/user-attachments/assets/3b5b317b-2be6-43a8-9e03-a295523e0bcd" width=500/>

버블정렬은 사용안하는 것이 좋음  → 효율이 안 좋음

반복문 2번 사용, 실무에서 사용하지 않음

- 아래 값들을 정렬 할 수 있는가 ?  X ⇒ stable하지 않다. but 기준을 정해주면 정렬 가능
    <img src="https://github.com/user-attachments/assets/958d5c45-c9d9-4ec7-b9bc-a6f4e4480d4b" width=300/>
    
- 자바로 구현한 버블 정렬
    
    ```java
    public static int[] bubbleSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
        // 외부 루프for (int j = 0; j < n - i - 1; j++) {
        // 내부 루프if (arr[j] > arr[j + 1]) {
        // 현재 요소와 다음 요소를 비교
        // 위치 교환int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
        return arr;
    }
    
    ```
    

### 선택정렬

주어진 목록에서 가장 작은 값을 찾아 정렬된 부분과 교환하는 정렬 방법

<img src="https://github.com/user-attachments/assets/600c115c-2263-4fef-a393-cf34635ba2e2" width=400/>

- 시간 복잡도: 선택 정렬은 항상 O(n^2)의 시간 복잡도를 가지며, 정렬 과정과 관계없이 비교 횟수는 동일 ,  최악의 경우에도 시간 복잡도는 O(n^2)
- stable하지 않다.
    
    <img src="https://github.com/user-attachments/assets/958d5c45-c9d9-4ec7-b9bc-a6f4e4480d4b" width=300/>
    
- 자바로 구현한 선택정렬
    
    ```java
    public static int[] selectionSort(int[] arr) {
          int n = arr.length;
          for (int i = 0; i < n - 1; i++) {// 외부 루프int minIdx = i;// 현재 인덱스를 최솟값 인덱스로 설정for (int j = i + 1; j < n; j++) {// 내부 루프if (arr[j] < arr[minIdx]) {// 현재 값이 최솟값보다 작다면
                      minIdx = j;// 최솟값 인덱스 업데이트
                  }
              }
    // 현재 인덱스와 최솟값 인덱스의 값 교환int temp = arr[i];
              arr[i] = arr[minIdx];
              arr[minIdx] = temp;
          }
          return arr;
      }
    
    ```
    

### 삽입정렬

삽입 정렬은 주어진 목록에서 각 요소를 이미 정렬된 부분에 알맞은 위치에 삽입하는 방식을 사용하는 알고리즘

- 목록을 두 부분, 즉 이미 정렬된 부분과 아직 정렬되지 않은 부분으로 나누는 방식을 사용한다.
- 정렬되지 않은 부분에서 요소를 선택하고 이를 이미 정렬된 부분에 알맞은 위치에 삽입하여 전체 목록을 정렬하는 작업을 수행

- 정렬이 되어있는 것일 수록 효율이 좋아진다.
- 순회보다 비교를 하기 때문에 O(n)이라고도 함.
- 데이터를 러프하게 정렬하고 삽입정렬 하면 좋다.

> **삽입 정렬의 원리**
> 
> 1. 목록의 첫 번째 요소는 이미 정렬된 것으로 간주
> 2. 그 다음 요소부터 시작하여 정렬되지 않은 부분의 첫 번째 요소를 선택
> 3. 이제 선택한 요소를 이미 정렬된 부분에서 적절한 위치에 삽입, 삽입 위치는 정렬된 부분의 요소와 비교하여 결정된다.
> 4. 이 과정을 반복하여 정렬된 부분을 점진적으로 확장한다.
> 5. 모든 요소가 정렬될 때까지 이 과정을 계속 반복

<img src="https://github.com/user-attachments/assets/600c115c-2263-4fef-a393-cf34635ba2e2" width=400/>

- 자바로 구현한 삽입정렬
    
    ```java
    public static void insertionSort(int[] arr) {
        int n = arr.length;
        for (int i = 1; i < n; i++) {// 외부 루프int current_value = arr[i];// 현재 요소 선택int j = i - 1;
            while (j >= 0 && arr[j] > current_value) {// 내부 루프
                arr[j + 1] = arr[j];// 이전 요소를 현재 위치로 이동
                j--;
            }
            arr[j + 1] = current_value;// 현재 요소를 올바른 위치에 삽입
        }
    }
    
    ```
    

### 퀵정렬✨

- 퀵 정렬은 '분할 정복' 전략을 기반으로 하는 알고리즘으로, 우리가 정렬하려는 목록을 효율적으로 정렬하는 데 사용
- 이 알고리즘은 "비교 정렬" 범주에 속하며, 평균적으로 매우 빠른 수행 속도를 자랑하여 많은 분야에서 널리 활용되고 있다.
- 퀵 정렬의 평균 시간 복잡도는 O(n log n)으로, 대부분의 경우에서 우수한 성능을 보이지만, 최악의 경우 시간 복잡도는 O(n^2)가 될 수 있다.
- 그러나 피벗 선택 방법을 최적화하거나 랜덤한 피벗을 선택하는 등의 방법을 사용하면, 최악의 시나리오에서도 효율적인 성능을 보장할 수 있습니다.
    - 피벗을 무엇으로 잡느냐에 따라 효율이 금증하거나 급감한다.
    - 퀵정렬 쓸때 정렬이 되어있다고 한다면 끝보다 가운데를 선택하는 것이 좋다.
    - 정렬할 숫자의 가장 가운데 숫자를 피벗으로 고르면 좋음
- 퀵 정렬은 내부 정렬, 외부 정렬 등 다양한 환경에서 유용하게 사용되는 알고리즘

<img src="https://github.com/user-attachments/assets/a59a026a-e064-4845-a718-0b5ca4ac6ed8" width=500/>

> **퀵 정렬의 동작 원리**
> 
> 1. 리스트에서 하나의 요소를 '피벗'으로 선정합니다. 피벗은 리스트의 중앙 요소일 수도 있고, 다양한 방식으로 선택할 수 있다.
> 2. 선택된 피벗을 중심으로, 작은 값을 피벗의 왼쪽에, 큰 값을 피벗의 오른쪽에 배치 → 이 단계를 리스트의 '분할'이라고 한다.
> 3. 이렇게 분할된 두 부분 리스트에 대해 같은 과정을 재귀적으로 수행. 즉, 부분 리스트에서 다시 피벗을 선정하고 분할합니다.
> 4. 부분 리스트의 크기가 1 이하가 될 때까지 이 과정을 반복한다. 크기가 1 이하인 리스트는 이미 정렬되었다고 간주
> 5. 모든 부분 리스트가 정렬되면, 피벗을 기준으로 왼쪽과 오른쪽 부분 리스트를 결합하여 전체 정렬 리스트를 완성한다.

- 자바로 구현한 퀵 정렬
    
    ```java
    public static List<Integer> quickSort(List<Integer> arr) {
        if (arr.size() <= 1) {// 재귀 종료 조건: 리스트의 길이가 1 이하면 종료return arr;
        }
    
        int pivot = arr.get(arr.size() / 2);// 피벗 선택
        List<Integer> left = new ArrayList<>();
        List<Integer> equal = new ArrayList<>();
        List<Integer> right = new ArrayList<>();
    
        for (int num : arr) {
            if (num < pivot) {
                left.add(num);
            } else if (num > pivot) {
                right.add(num);
            } else {
                equal.add(num);
            }
        }
    
        List<Integer> sortedLeft = quickSort(left);
        List<Integer> sortedRight = quickSort(right);
    
        sortedLeft.addAll(equal);
        sortedLeft.addAll(sortedRight);
    
        return sortedLeft;
    }
    
    ```
    

### 병합정렬

- 병합 정렬은 리스트를 절반으로 나눈 뒤, 이들을 재귀적으로 정렬하고 다시 합치는 방식으로 작동하는 알고리즘
- 이 과정에서는 '분할(Divide)'과 '병합(Merge)' 두 가지 단계가 포함되어 있다. 이러한 단계들을 순환적으로 반복하며 전체 리스트를 정렬해나감

> **병합 정렬의 동작 원리**
> 
> 1. 분할(Divide): 주어진 리스트를 절반으로 나눈다. 이 분할 과정은 리스트의 크기가 1 이하가 될 때까지 계속해서 이루어집니다. 각 분할 단계에서는 리스트가 두 부분으로 나뉜다.
> 2. 정복(Conquer): 분할된 부분 리스트를 재귀적으로 정렬. → 이는 각 부분 리스트에 대해 순환적으로 분할과 병합 과정을 수행함으로써 리스트의 크기를 점점 줄여 나가며 정렬을 수행한다.
> 3. 병합(Merge): 정렬된 부분 리스트들을 합쳐 최종적으로 정렬된 리스트를 생성한다. 이 병합 과정에서는 정렬된 부분 리스트들을 차례대로 비교하여 작은 요소부터 새로운 리스트에 합치는 방식으로 작동한다.

<img src="https://github.com/user-attachments/assets/3b7350a5-902f-4b7e-90bf-cdc5d7b1709e" width=500/>


- 자바로 구현한 병합정렬
    
    ```java
    public static List<Integer> mergeSort(List<Integer> arr) {
        if (arr.size() <= 1) {// 재귀 종료 조건: 리스트의 길이가 1 이하면 종료return arr;
        }
    
        int mid = arr.size() / 2;// 리스트를 절반으로 분할
        List<Integer> left = new ArrayList<>(arr.subList(0, mid));
        List<Integer> right = new ArrayList<>(arr.subList(mid, arr.size()));
    
        left = mergeSort(left);// 왼쪽 부분 리스트 재귀적으로 정렬
        right = mergeSort(right);// 오른쪽 부분 리스트 재귀적으로 정렬return merge(left, right);// 정렬된 부분 리스트 병합
    }
    
    public static List<Integer> merge(List<Integer> left, List<Integer> right) {
        List<Integer> merged = new ArrayList<>();
        int leftIdx = 0;
        int rightIdx = 0;
    
    // 두 부분 리스트를 순서대로 비교하면서 작은 요소를 병합while (leftIdx < left.size() && rightIdx < right.size()) {
            if (left.get(leftIdx) < right.get(rightIdx)) {
                merged.add(left.get(leftIdx));
                leftIdx++;
            } else {
                merged.add(right.get(rightIdx));
                rightIdx++;
            }
        }
    
    // 남은 요소들을 병합while (leftIdx < left.size()) {
            merged.add(left.get(leftIdx));
            leftIdx++;
        }
    
        while (rightIdx < right.size()) {
            merged.add(right.get(rightIdx));
            rightIdx++;
        }
    
        return merged;
    }
    
    ```
    

## DFS 와 BFS

그래프 순회는 그래프의 모든 정점을 방문하는 연산입니다. 일반적으로 깊이 우선 탐색(DFS)과 너비 우선 탐색(BFS)이 사용된다.

넓이 우선 탐색은 갈 수 있는 길을 먼저 간다.

DFS와 BFS 모두 지나온 길은 상태 관리를 해야 한다 - 기록 필요,  갔다 온 길을 체크할 수 있어야 함

### DFS (Depth First Search, DFS)

- 시작 정점에서부터 최대한 깊숙이 탐색을 진행하고 더 이상 탐색이 불가능 할 때 이전 단계로 돌아와 다음 분기로 탐색을 진행하는 알고리즘
- 즉, 한 분기를 탐색한 후 다음 분기로 넘어가기 전에 해당 분기를 완벽하게 탐색한다.
- DFS에서는 깊이가 없으면 앞에 것을 다시 선택해야하므로 Backtracking을 사용한다
- DFS는 최단 경로를 보장하지 않는다.
- 재귀 또는 Stack을 사용

<img src="https://github.com/user-attachments/assets/e5c0b2c3-cdb8-4f37-a46d-5ed7f3f11d8f" width=500/>

재귀는 무조건 탈출 조건이 있어야 함.

여기서는 if(시작 정점과 도착 정점이 같다면)이 탈출구

재귀는 메모리를 엄청나게 사용한다. =  실무에서 병목 현상 생겨서 절대 쓰면 안됨, 알고리즘 문제에서만 풀기

## **재귀적 접근 방법**

DFS를 재귀적으로 구현하는 방법은 다음과 같습니다:

1. 방문한 정점을 표시하기 위한 배열을 생성합니다.
2. 현재 정점을 확인합니다. 최초 실행시에는 일반적으로 root(최상위 노드)부터 탐색을 시작합니다.
3. 해당 현재 정점의 방문여부를 체크합니다. (방문했다고 표기 변경)
4. 현재 정점과 인접한 정점이 있는지 확인하고, 방문 여부를 확인합니다.
5. 인접한 정점 중에서 방문하지 않은 정점이 있다면, 해당 정점을 방문했다고 표시하고 2~5번 과정을 재귀 호출하여 해당 정점을 현재 정점으로 설정합니다.
6. 모든 정점을 방문할 때까지 2~5 단계를 반복

### BFS

- 너비 우선 탐색은 한 정점을 기준으로 인접한 모든 정점들을 방문한 후, 방문한 정점을 새로운 기준으로 설정하여 이와 인접한 정점들을 차례로 방문하는 방식의 탐색
- 주로 큐(Queue)라는 자료 구조를 활용하여 구현
1. 동작 원리
    1. 시작 정점을 큐에 넣고, 해당 정점을 방문한 것으로 표시
    2. 큐가 빌 때까지 다음의 단계를 반복한다.
        - 큐에서 하나의 정점을 가져와 현재 정점으로 설정
        - 현재 정점과 인접한 모든 정점들을 검토
        - 아직 방문하지 않은 인접한 정점들을 모두 큐에 넣고, 방문한 것으로 표시
        - (배열을 하나 만들어서 갔다온 길을 true로 체크하고 false일 때만 이동)
    3. 큐가 비었을 때 탐색을 종료

### **BFS를 활용해야 하는 경우**

1. 그래프의 모든 정점을 방문해야 하는 문제에서 DFS나 BFS 중 어느 것을 사용 가능
2. 주요 목표가 모든 정점을 방문하는 것이라면, 둘 중에서 편한 것을 선택하면 된다.
3. 최단 경로를 찾아야 하는 문제
    - 예를 들어, 미로 찾기와 같이 최단 경로를 찾아야 하는 상황에서는 BFS가 유용.
    - DFS로 경로를 검색할 경우 처음으로 발견되는 해결책이 반드시 최단 경로라는 보장이 없음.
    - 그러나 BFS는 현재 노드에서 가장 가까운 곳부터 탐색하기 때문에, 처음으로 발견되는 해결책이 바로 최단 경로가 된다.
4. 검색 대상이 크지 않고, 검색의 시작점에서 원하는 대상이 멀지 않은 경우
    - 최단 경로가 존재한다면, 하나의 경로가 무한히 이어지더라도 BFS는 방문 여부를 확인하고, 모든 경로를 검토하므로 **반드시 최단 경로를 찾을 수 있다.**

### **`BFS` 활용시 주의할 점**

1. 그래프의 크기와 밀도에 따라 성능이 달라진다.
    - 그래프의 크기나 밀도가 커질수록, 즉 정점이나 간선의 개수가 많아질수록 BFS의 성능이 저하될 수 있다.
2. 방문한 정점들을 저장해야 하므로 메모리 사용량이 크다.
    - BFS는 큐를 사용하여 방문한 정점들을 저장해야 한다. 따라서 그래프의 크기가 커질수록, 큐에 저장되는 정점의 수도 늘어나 메모리 사용량이 커진다. 이 경우, BFS보다는 DFS를 사용하는 것이 바람직
3. BFS는 시작점에서 도달 가능한 정점들만 탐색
    - BFS는 시작점에서 도달할 수 없는 정점은 탐색하지 않는다. 따라서 시작점에서 도달 가능한 정점들만 탐색하는 경우에만 사용해야 한다.
4. BFS는 '방문 여부를 체크하는 자료구조'를 사용해야 한다.
    - BFS에서는 방문 여부를 체크하는 자료구조가 필요 → 이를 사용하지 않으면 무한 루프에 빠질 수 있다.

### 알고리즘 구현

```java
/**
     * BFS 인접행렬 : 너비 우선 탐색을 구현한 템플릿 예제
     *
     * @param array : 각 정점을 행/열로 하고, 연결된 정점은 1로, 연결되지 않은 정점은 0으로 표기하는 2차원 배열
     * @param visited : 방문여부 확인을 위한 배열
     * @param src : 방문할 정점
     * @param result : 방문했던 정점 리스트를 저장한 List
     *
     **/public ArrayList<Integer> bfs_array(int[][] array, boolean[] visited, int src, ArrayList<Integer> result) {
//bfs의 경우 큐를 사용합니다.
	    Queue<Integer> queue = new LinkedList<>();
//시작 지점을 큐에 넣어주고, 해당 버택스의 방문 여부를 변경합니다.
	    queue.offer(src);
	    visited[src] = true;
//큐에 더이상 방문할 요소가 없을 경우까지 반복합니다.while (!queue.isEmpty()) {
//현재 위치를 큐에서 꺼낸 후int cur = queue.poll();
// 현재 방문한 정점을 result에 삽입합니다.
				result.add(cur);
//전체 배열에서 현재 버택스의 행만 확인합니다.for (int i = 0; i < array[cur].length; i++) {
//길이 존재하고, 아직 방문하지 않았을 경우if(adjArray[cur][i] == 1 && !visited[i]) {
//큐에 해당 버택스의 위치를 넣어준 이후
	          queue.offer(i);
//방문 여부를 체크합니다.
	          visited[i] = true;
	        }
	      }
	    }
//이어진 모든 길을 순회한 후 방문 여부가 담긴 ArrayList를 반환합니다.return result;
  }
```