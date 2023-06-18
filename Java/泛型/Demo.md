#### 不需要强制类型转换

```java
public class Generic<T> {
  private T t;

  public T get() {
    return t;
  }

  public void set(T t) {
    this.t = t;
  }

  public void noSpecifyType() {
    Generic generic = new Generic();
    generic.set("test");
    String test = (String) generic.get(); //需要强制类型转换
  }

  public void specifyType() {
    Generic<String> generic = new Generic();
    generic.set("test");
    String test = generic.get(); //不需要强制类型转换
  }
}
```

#### 上界通配符

extends关键字声明，表示参数化的类型可能是所指定的类型，或者是此类型的子类。

```java
static int countLegs(List<? extends Animal> animals) {
  int result = 0;
  for (Animal animal : animals) {
    result += animal.countLegs();
  }
  return result;
}

static int countAnimalLegs(List<Animal> animals) {
  int result = 0;
  for (Animal animal : animals) {
    result += animal.countLegs();
  }
  return result;
}

public static void main(String[] args) {
  List<Dog> dogs = new ArrayList<>();
  countLegs(dogs); //不会报错
  countAnimalLegs(dogs); //报错
}
```

#### 下界通配符

 super关键字声明，表示参数化的类型可能是所指定的类型，或者是此类型的父类型，直至 Object。

```java
static <T> void test(List<? super T> dst, List<T> src) {
  for (T t : src) {
    dst.add(t); //dst类型“大于等于”src 的类型，这里的“大于等于”是指dst表示的范围比src要大，因此装得下dst的容器也就能装src
  }
}

public static void main(String[] args) {
  List<Dog> dogs = new ArrayList<>();
  List<Animal> animals = new ArrayList<>();
  test(animals, dogs);
}
```

