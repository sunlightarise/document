#### isAssignableFrom()������instanceof�ؼ�������
* Class.isAssignableFrom() �������ж�Class1 ����һ���� Class2 �Ƿ���ͬ��������һ����ĸ�����߽ӿڡ�

```java
    A.class.isAssignableFrom(B.class)
```
�����ߺͲ�������java.lang.Class���ͣ�����������������Ϊtrue, ���ʾA��B�ĸ�����߽ӿڣ�B������һ������߽ӿ�

* instanceof �������ж�һ������ʵ���Ƿ���һ�����ӿڶ����������ӿڵ�ʵ����

```java
    o instanceof TypeName
```
��һ�������Ƕ���ʵ�������ڶ��������Ǿ�����������߽ӿ���

#### ��������

```java
public class MyTest1 {

    public static void main(String[] args) {
        B b = new B();
        if (C.class.isAssignableFrom(B.class)) {
            System.out.println(1);
        }
        System.out.println(2);
        if (b instanceof C) {
            System.out.println(3);
        }
    }

    static class A {

    }

    static class B extends A implements C {

    }

    static interface C {

    }

}
```
