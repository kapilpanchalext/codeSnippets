# Code Snippets Collections

- [Code Snippets Collections](#code-snippets-collections)
  - [1. Merging an Object Bean (Two beans with same parent will concatenate into a third bean with both elements and no nulls)](#1-merging-an-object-bean-two-beans-with-same-parent-will-concatenate-into-a-third-bean-with-both-elements-and-no-nulls)
  - [2. Mockito generator](#2-mockito-generator)
  - [3. Object Copy](#3-object-copy)
  - [4. Remove Duplicates from a List using Java Streams](#4-remove-duplicates-from-a-list-using-java-streams)
  - [5. Remove from a List on condition](#5-remove-from-a-list-on-condition)
  - [6. MVC Getting data from Model](#6-mvc-getting-data-from-model)
  - [7. Java Streams Distinct](#7-java-streams-distinct)
  - [8. Filter one list from another(O(N\*\*2) Time Complexity)](#8-filter-one-list-from-anotheron2-time-complexity)
  - [9. Filter one list from another(O(N\*\*2) Time Complexity)](#9-filter-one-list-from-anotheron2-time-complexity)
  - [10. Concurrent Streams](#10-concurrent-streams)
  - [11. Executor Service](#11-executor-service)
  - [12. Logback-Config](#12-logback-config)
  - [13. Collectors To Counting](#13-collectors-to-counting)
  - [14. Map getOrDefault (remove duplicates (O(N) or O(N-log(N)) Time Complexity))](#14-map-getordefault-remove-duplicates-on-or-on-logn-time-complexity)
  - [15. Map getOrDefault (remove duplicates (O(N) or O(N-log(N)) Time Complexity))](#15-map-getordefault-remove-duplicates-on-or-on-logn-time-complexity)
  - [16. Collections Frequency(O(N\*\*2) Time Complexity)](#16-collections-frequencyon2-time-complexity)
  - [17. Execution time (nano seconds)](#17-execution-time-nano-seconds)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>



## 1. Merging an Object Bean (Two beans with same parent will concatenate into a third bean with both elements and no nulls)

```java
Class<?> formBeanClass = formBean.getClass(); 
Field[] fields = formBeanClass.getDeclaredFields(); 
Object returnValue = formBeanClass.newInstance(); 
    for (Field field : fields) { 
        field.setAccessible(true); 
        Object value1 = field.get(formBean); 
        Object value2 = field.get(parsedData); 
        Object value = (value1 != null) ? value1 : value2; 
            if(value instanceof String) { 
                value = (!StringUtils.equalsIgnoreCase((String) value1, "NA")) ? value1 : value2; 
                } 
            field.set(returnValue, value); 
        } 
        somePlanMessage = someDAO.somePlan((SomeBean) returnValue);
        }
    }
}
```

## 2. Mockito generator
```java
// SomeBean beanElement = new SomePlanBean(); 
// Class<?> somePlanClass = beanElement.getClass(); 
// Field fields [] = somePlanClass.getDeclaredFields(); 
// FIELD NAME GENERATOR  
for (Field field : fields) { 
    // field.setAccessible(true); 
    // System.err.println("Field " + field.getName() + " = testSomePlanBean.getClass().getDeclaredField(\""+ field.getName() + "\");"); 
    // System.err.println(field.getName() + ".setAccessible(true);"); 
    // System.err.println(field.getName() + ".set(testSomePlanBean,"+ "RandomStringUtils.randomAlphanumeric(20) + \" \" + RandomStringUtils.randomAlphanumeric(20));"); 
    // System.err.println(); // 
}    
```

## 3. Object Copy
```java
Class<?> formBeanClass = module.getClass(); 
Field[] fields = formBeanClass.getDeclaredFields(); 
Object returnValue = formBeanClass.newInstance(); 
for (Field field : fields) { 
    field.setAccessible(true); 
    Object value1 = field.get(module); 
    Object value = value1; 
    field.set(returnValue, value); 
    } 
    SessionPlanModuleBean moduleBean = SessionPlanModuleBean.class.cast((returnValue)); 
    moduleBean.setSessionPlanId(moduleBean.getId());
```

## 4. Remove Duplicates from a List using Java Streams
```java
List<SessionPlanModuleBean> filteredParsedList = parsedDataList
            .stream()
            .collect(collectingAndThen(toCollection(
                () -> new TreeSet<>(comparingLong((element) -> element.getSessionModuleNo()))), 
                    list -> new ArrayList<>(list)));
```

## 5. Remove from a List on condition
```java
filteredParsedList.removeIf(x -> intersectionSessionModuleNoSet.contains(x.getSessionModuleNo()));
```

## 6. MVC Getting data from Model
```java
SomeBean someModuleId = (SomeBean) m.asMap().get("someBean");
```

## 7. Java Streams Distinct
```java
List<String> someConfigIdList = someModulesList
            .stream()
            .map((element) -> { 
                return element.getsomeConfigId(); 
            })
            .distinct()
            .collect(Collectors.toList());
```

## 8. Filter one list from another(O(N**2) Time Complexity)
```java
Set<String> carNames = listCarName
        .stream()
        .map(DataCarName::getName)
        .collect(Collectors.toSet()); 
List<DataCar> listOutput = listCar
        .stream()
        .filter(e -> carNames.contains(e.getName()))
        .collect(Collectors.toList());
```

## 9. Filter one list from another(O(N**2) Time Complexity)
```java
Set<String> carNames = listCarName
        .stream()
        .map(DataCarName::getName)
        .collect(Collectors.toSet()); 
List<DataCar> listOutput = listCar
        .stream()
        .filter(e -> carNames.contains(e.getName()))
        .collect(Collectors.toList());
```

[Reference](https://stackoverflow.com/questions/36246998/stream-filter-of-1-list-based-on-another-list) Filter one list from another(O(N**2) Time Complexity)

## 10. Concurrent Streams
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5); 
// List<Integer> result = numbers.stream() 
// .peek(System.out::println) 
// .map(n -> n * 2) // 
.peek(System.out::println) 
// .collect(Collectors.toList()); 
// List<Integer> result = numbers
    .stream()
    .peek(element -> System.err.println("Pre: " + element))
    .map(element -> {
        return element*2;
        })
    .peek(element -> System.err.println("Post: " + element))
    .collect(Collectors.toList()); 
    
List<Integer> resultParallel = numbers
    .parallelStream()
    .peek(element -> System.err.println("Pre: " + element))
    .peek(n -> System.err.println("Thread Name: " + Thread.currentThread().getName()))
    .map(element -> {
        return element*2;
        })
        .peek(element -> System.err.println("Post: " + element))
        .collect(Collectors.toList()); 
System.err.println(resultParallel);
```

## 11. Executor Service
```java
ExecutorService executorService = Executors.newFixedThreadPool(5); 
executorService.execute(() -> { 
    // Code for the task to be executed concurrently 
    System.err.println("Runnable Task executed1 " + " [Thread Name: " + Thread.currentThread().getName() + "]"); 
    }); 
    executorService.execute(() -> { 
        // Code for the task to be executed concurrently 
        System.err.println("Runnable Task executed2 " + " [Thread Name: " + Thread.currentThread().getName() + "]"); 
    });
```

## 12. Logback-Config
```xml
<!-- ///////////////////////// --> 
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <!-- encoders are assigned the type ch.qos.logback.classic.encoder.PatternLayoutEncoder by default --> 
        <encoder> <pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) :: %highlight(%logger{36}) - %msg%n</pattern> 
        </encoder> 
    </appender> 
    <root level="info"> 
        <appender-ref ref="STDOUT" /> 
    </root> 
<!-- ///////////////////////// -->
```

## 13. Collectors To Counting
```java
    Map<Integer, Long> countMap = oddList1
        .stream()
        .collect(Collectors.groupingBy(e -> e, Collectors.counting()));
```

## 14. Map getOrDefault (remove duplicates (O(N) or O(N-log(N)) Time Complexity))
```java
    Map<Integer, Long> countMap = oddList1
        .stream()
        .collect(Collectors.groupingBy(e -> e, Collectors.counting()));
```

## 15. Map getOrDefault (remove duplicates (O(N) or O(N-log(N)) Time Complexity))
```java
    Map<Integer, Integer> countMap = new HashMap<>(); 
        for (Integer number : integerArray2) { 
            countMap.put(number, countMap.getOrDefault(number, 0) + 1); 
        }
```

## 16. Collections Frequency(O(N**2) Time Complexity)
```java
   uniqueElementSet
        .stream()
        .map(element -> { 
            // 
            // int frequencyValue = Collections.frequency(oddList, element); 
            // if(frequencyValue%2 != 0) { 
                // value.set(element); // 
                } // return element; // 
            })
            .collect(Collectors.toList());
```

## 17. Execution time (nano seconds)
```java
   long startEmpty = System.nanoTime();
```
## 18. Javascript Idle time indicator, active when user moves mouse
```javascript
<!DOCTYPE html>
<html>
<body>

<h1>The template Element</h1>
<button id="startButton" onclick="showContent()">Activate User Idle</button>
<button id="stopButton">Stop User Idle</button>
<template>
  <h2>Flower</h2>
</template>

<script>
    function showContent() {
        let temp = document.getElementsByTagName("template")[0];
        let clon = temp.content.cloneNode(true);
        document.body.appendChild(clon);
    }

    const controller = new AbortController();
    const signal = controller.signal;

    startButton.addEventListener("click", async () => {
    if ((await IdleDetector.requestPermission()) !== "granted") {
        console.error("Idle detection permission denied.");
        return;
    }

    try {
        const idleDetector = new IdleDetector();
            idleDetector.addEventListener("change", () => {
                const userState = idleDetector.userState;
                const screenState = idleDetector.screenState;
                console.log(`Idle change: ${userState}, ${screenState}.`);
        });
            await idleDetector.start({
            threshold: 60_000,
            signal,
        });
        console.log("IdleDetector is active.");
    } catch (err) {
        console.error(err.name, err.message);
        }
    });
        stopButton.addEventListener("click", () => {
            controller.abort();
            console.log("IdleDetector is stopped.");
    });
</script>

</body>
</html>
```

Note: [Reference](file:///D:/PROJECT_ENABLERS/markdowns/CodeReviewGeneralCommentsV1.md)
