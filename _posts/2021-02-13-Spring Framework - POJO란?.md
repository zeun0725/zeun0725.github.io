---
title: "Spring Framework - POJO란?"
date: 2021-02-13 00:00:00 -0400
categories: JAVA
---
### POJO(Plain Old Java Object)란?
#### 특정 규약에 종속되지 않는 자바 객체

##### POJO 개념을 사용하지 않은 예시 (특정 환경에 결합도가 높은 코드)  
JMS로부터 메시지를 받는 경우 -> MEssageListener 인터페이스를 상속 받아야 함
_JMS라는 특정 환경에 종속되게 되고 다른 메시징 솔루션을 적용하기 어려워짐_  
~~~java
public class ExampleListener implements MessageListener {
  public void onMessage(Message message) {
    if (message instanceof TextMessage) {
      try {
        System.out.println(((TextMessage) message).getText());
        }
      catch (JMSException ex) {
        throw new RuntimeException(ex);
        }
    }
    else {
      throw new IllegalArgumentException("Message must be of type TextMessage");
    }
  }
}
~~~

##### POJO 개념을 사용한 예시 (특정 환경에 결합도가 낮은 코드)
~~~java
@Component
public class ExampleListener {
  @JmsListener(destination = "myDestination")
  public void processOrder(String message) {
    System.out.println(message);
  }
}
~~~
위의 코드는 어떠한 인터페이스에 종속되지 않음  
_@JmsListener 라는 어노테이션을 이용해 JMS 서비스와 연동함_  

스프링 프레임워크는 위의 예제처럼 우리의 **코드와 라이브러리와의 결합성을 줄이도록 도와준다.**  

+ POJO 조건
  - 특정 규약에 종속되지 않는다.
  - 특정 환경에 종속되지 않는다.
  - 단일 책임 원칙을 지키는 클래스
