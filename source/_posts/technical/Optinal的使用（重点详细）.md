---
title: "Java的Optional类的使用"
date: 2025-11-26
categories:
  - 技术
  - 后端
tags:
  - 后端
  - 笔记
---

## Java的Optional类的使用（重点详细）

今天在复习Java基础的时候，发现自己忽略掉的一个知识点——Java8中的Optional类

Optional是Java8引进的一个容器，用来简化空指针判断的和避免空指针异常。

## 一个实例体现Optional的用处

给出数据结构

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
class User{
    private String name;
    private Integer age;
    private Province province;
}

@Data
@AllArgsConstructor
class Province{
    private String ProvinceName;
    private City city;
}
@Data
@AllArgsConstructor
class City{
    private String CityName;
}
```

我们有一个User对象我们需要**得到他所在城市的名字**

一般，为了确保程序不报**空指针异常**，我们是这样做的

```java
@Test
    public void optt4(){
        User user = new User();
        user.setName("zhangsan");
        user.setAge(10);
        Province province = new Province("江西",new City("南昌"));
//        Province province = new Province("江西",null);
        user.setProvince(province);
        Province userProvince = null;
        if (user != null) {
            userProvince = user.getProvince();
        }
        City userCity = null;
        if (province != null) {
            userCity = province.getCity();
        }
        String userCityName = null;
        if (userCity != null) {
            userCityName = userCity.getCityName();
        }
        if(userCityName != null){
            System.out.println("userCityName = " + userCityName);
        }else {
            System.out.println("userCityName is null");
        }
    }
```

代码冗长，不够优雅！！！

**使用Optional类**

```java
@Test
    public void optt3(){
        User user = new User();
        user.setName("zhangsan");
        user.setAge(10);
        Province province = new Province("江西",new City("南昌"));
//        Province province = new Province("江西",null);
        user.setProvince(province);
        Optional<String> cityNameOpt = Optional.of(user).map(u -> u.getProvince())
                .map(p -> p.getCity())
                .map(c -> c.getCityName());
        cityNameOpt.ifPresentOrElse(
            name -> System.out.println("cityName:"+name),
            ()->{
            System.out.println("cityName为空");
        });
    }
```

不需要冗长的代码，干净简练！！

## Optional的使用

### 创建Optional对象

主要有三个方法

```java
@Test
    public void optt5(){
        //如果传入的对象是null的话，不会报空指针，而是返回一个值为空的Optional对象
        Optional<Object> o = Optional.ofNullable(new User());
        //直接返回一个值为空的Optional对象
        Optional<Object> empty = Optional.empty();
        //将传入对象当做Optional的值，但是如果传入的对象是null，会报空指针异常
        Optional.of(new User());
    }
```

### 获取值

#### get()

如果里面的值为空，会报NoSuchElementException("No value present")异常

```java
public void optt5(){
        Optional<User> user = Optional.of(new User());
        User u = user.get();
    }
```

源码

```java
public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```

#### orElse()和orElseGet()

这函数都是处理如果Optional里面的值如果是null 的情况。

```java
		Optional<User> user = Optional.of(new User());
        User u1 = user.get();
        User u2 = new User("张三",20,null);
		User temp ;
        //参数是一个对象，如果Optional里面的对象是空的话，返回这个对象
        temp = user.orElse(u2);
		//里面的参数是一个get()函数，简单点说必须是有一个返回值的函数。
        temp = user.orElseGet(()->new User("李四",20,null));
		temp = user.orElseGet(User::new);
```

### 其他几个重要的方法

#### ifPresent()和ifPresentOrElse()

处理判断Optional里面的数据是否为空，并且可以执行下一步操作

```java
public void optt5(){
        Optional<Object> o = Optional.ofNullable(new User());
        Optional<Object> empty = Optional.empty();
        Optional<User> user = Optional.of(new User());
        User u1 = user.get();
        User u2 = new User("张三",20,null);
    	//返回是否存在值
        boolean present = user.isPresent();
    	//如果存在执行逻辑行为
        user.ifPresent(user1 -> System.out.println("user1 = " + user1));
    	//如果存在执行逻辑行为1，如果存在执行逻辑行为2
        user.ifPresentOrElse(
                user2 -> System.out.println("user2 = " + user2),
                ()-> System.out.println("user2 is null")
                );
    }
```

#### map()

将Optional里面的数据提取封装成新的Optional

在最开始的实例中：

```java
Optional<String> cityNameOpt = Optional.of(user).map(u -> u.getProvince())
                .map(p -> p.getCity())
                .map(c -> c.getCityName());
```

这段代码其实可以拆分成：

```java

		//建议使用ofNullable(),不会报空指针
		Optional<User> optional= Optional.ofNullable(user);
		//Province封装到Optional
        Optional<Province> provinceOpt = optional.map(u -> u.getProvince());
		//city封装到Optional
        Optional<City> city = provinceOpt.map(p -> p.getCity());
		//最后获得cityName
        Optional<String> s = city.map(c -> c.getCityName());
```

#### orElesThrow()

如果值为空，抛出异常，不指定默认是NoSuchElementException(“No value present”)，也可以自己指定

```java
       Optional<String> cityNameOpt = city.map(c -> c.getCityName());
		//cityNameOpt.orElseThrow();
       cityNameOpt.orElseThrow(()->new NoSuchElementException("出现null"));
```

## 总结

Optional在特定的场景下，比如在对多层嵌套的对象取值的时候，可以简化代码和避免空指针异常。就我而言，Optional在日常的开发中使用的并不多，如果不是无意间了解到的这个类，可能永远也用不上，Optional的主要作用是判断空指针和避免空指针异常，空指针往往是代表着值不可以使用，但是在业务场景中值不可以使用不只是因为是null，比如空的字符串，空的集合也代表着不可以使用。在这些情况中，Optional就显得有些捉襟见肘了。该总结是作者的个人见解，如果有新的理解也可以在评论区分享一下。
