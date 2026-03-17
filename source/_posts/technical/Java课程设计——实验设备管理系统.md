---
cover:
title: Java课程设计实验设备管理系统
date: 2025-05-01
categories:
  - 杂
tags:
  - 课程设计
  - 笔记
---

# Java课程设计实验设备管理系统

## 课程设计题目

**实验设备管理系统设计**

实验设备信息包括：设备编号，设备种类(如：微机、打印机、扫描仪等等)，设备名称，设备价格，设备购入日期，是否报废，报废日期等。

主要功能：

1、能够完成对设备的录入和修改

2、对设备进行分类统计

3、设备的破损耗费和遗损处理。根据“是否报废”进行删除

4、设备的查询

## 系统设计构思

先设计一个父类Equipment类,然后Printer类，Scan类,Microcomputer类分别继承Equipment类（这是最基本的）。主体Database类，最后设计一个接口Handler，对于各个处理设计一个类，如Handler_add，Handler_count,Handler_del,Handler_check,Handler_set等。最后在Main里面执行

###

![外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传](https://img-home.csdnimg.cn/images/2023![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f1174fcfd1984cb4b237665798cf4e5e.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f83e36329eed4f838e6e6f7f32ee869d.png#pic_center)

# 代码展示

## Database类

**字段**

1. 存放所有设备的ArrayList集合 equipment
2. 对每种设备分开存储的HashMap集合 type_map
3. 处理集cd
4. 类集合Class_map（用来后面动态生成类）

**文件读写**

Java课只教了文件读写，虽然有点low，但也只能用这个了，连接数据库还没学······

```java
import java.io.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Scanner;

public class DataBase {
    private ArrayList<Equipment> equipment;
    private HashMap<String ,ArrayList<Equipment>> type_map;//分类
    private HashMap<String ,Handler>cd;//处理集
    private HashMap<String ,Class> Class_map;//类集
    public DataBase()
    {
        equipment = new ArrayList<Equipment>();
        type_map = new HashMap<String ,ArrayList<Equipment>>();
        Class_map = new HashMap<>();
        type_map.put("扫描机", new ArrayList<Equipment>());
        Class_map.put("扫描机", Scan.class);
        type_map.put("打印机",new ArrayList<Equipment>());
        Class_map.put("打印机",Printer.class);
        type_map.put("微机",new ArrayList<Equipment>());
        Class_map.put("微机",Microcomputer.class);
        cd = new HashMap<>();
        cd.put("1", new Handler_add(this));
        cd.put("3", new Handler_count(this));
        cd.put("2",new Handler_del(this));
        cd.put("4",new Handler_check(this));
        cd.put("5",new Handler_query(this));
        cd.put("6",new Handler_set(this));
        Load_resource();
    }
/**
 * 导入数据
 */
    public void Load_resource() {
        File file = new File("D:\\IDEA-workspace\\Laboratory_Managment\\src\\scoure.txt");
        try (ObjectInputStream in = new ObjectInputStream(
                new BufferedInputStream(new FileInputStream(file)))) {
            while (true) {
                Equipment x = (Equipment) in.readObject();  // 只读取一次
                equipment.add(x);
                String type=x.getTypeName();
                type_map.get(type).add(x);
            }
        } catch (EOFException e) {
            // 正常结束：文件读取完毕
            //设置起始id
            if(!equipment.isEmpty()) Equipment.set_Id(equipment.getLast().getNumber()+1);
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 保存数据
     */
    public void put_resource(){
        File file = new File("D:\\IDEA-workspace\\Laboratory_Managment\\src\\scoure.txt");
        try (ObjectOutputStream out = new ObjectOutputStream(
                new BufferedOutputStream((new FileOutputStream(file))));
            ){
            for (Equipment x : equipment)
            {
                out.writeObject(x);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    //初始面板
    public void start_panel()
    {
        System.out.println("——————————————————————————");
        System.out.println("欢迎使用实验设备管理系统");
        System.out.println("1.添加设备");
        System.out.println("2.删除设备");
        System.out.println("3.设备统计");
        System.out.println("4.设备检查");
        System.out.println("5.设备查询");
        System.out.println("6.设备修改");
        System.out.println("0.退出");
        System.out.println("——————————————————————————");
    }
    public void print(){
        for (Equipment x : equipment){
            System.out.println(x.toString());
        }
    }
    //清空文件
    public void Clear(){
        File file = new File("D:\\IDEA-workspace\\Laboratory_Managment\\src\\scoure.txt");
        try (ObjectOutputStream out = new ObjectOutputStream(
                new BufferedOutputStream((new FileOutputStream(file))));
        ){
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    //运行
    public void  run(){
        Scanner sc = new Scanner(System.in);
        Handler handler = null;
        String command;
        while(true)
        {
            start_panel();
            Handler hd = null;
                command = sc.next();
                if(command.equals("0"))break;
                while (!cd.containsKey(command))
                {
                    System.out.println("输入格式错误：重新输入");
                    command = sc.next();
                }
                hd = cd.get(command);
                hd.work();

        }
        put_resource();
    }

    public ArrayList<Equipment> getEquipment() {
        return equipment;
    }

    public HashMap<String, ArrayList<Equipment>> getType_map() {
        return type_map;
    }

    public HashMap<String, Handler> getCd() {
        return cd;
    }

    public HashMap<String, Class> getClass_map() {
        return Class_map;
    }
}

```

## Equipment类

这里生成了太多设置字段和取字段的函数了，如果觉得代码太过长了，大家可以去下载Lombok插件，它可以节省代码，看起来更整洁，具体使用可以去网上查。

```java
import java.io.Serializable;
import java.time.LocalDate;

public class Equipment implements Serializable{
    private static int id=1;
    private int number;
    private String name;
    private int price;
    private LocalDate start_date;
    private LocalDate end_date;
    private boolean bool_scrap=false;

    public Equipment(String name, int price, LocalDate start_date, LocalDate end_date) {
        id = id++;
        this.name = name;
        this.price = price;
        this.start_date = start_date;
        this.end_date = end_date;
    }
    public static void set_Id(int start){
        id = start;
    }
    public  Equipment(){
        this.number = id++;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }

    public int getNumber() {
        return number;
    }

    public LocalDate getStart_date() {
        return start_date;
    }

    public LocalDate getEnd_date() {
        return end_date;
    }

    public boolean isBool_scrap() {
        return bool_scrap;
    }


    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public void setStart_date(LocalDate start_date) {
        this.start_date = start_date;
    }

    public void setEnd_date(LocalDate end_date) {
        this.end_date = end_date;
    }

    public void setBool_scrap(boolean bool_scrap) {
        this.bool_scrap = bool_scrap;
    }

    @Override
    public String toString() {
        return  "编号 = " + number +
                ", 设备名 = " + name +
                ", 购入价格 = " + price +
                ", 购入日期 = " + start_date +
                ", 报废日期 = " + end_date +
                ", 是否报废 = " + (bool_scrap?"是":"否")+" "
                ;
    }

    public String toString2() {
        return "Equipment{" +
                "number=" + number +
                ", name='" + name + '\'' +
                ", price=" + price +
                ", start_date=" + start_date +
                ", end_date=" + end_date +
                ", bool_scrap=" + bool_scrap +
                '}';
    }
    public String getTypeName() {
        return "";
    }
}

```

### Printer类

```java
import java.time.LocalDate;

public class Printer extends Equipment{
    public final  String TypeName = "打印机";//设备种类名

    public String getTypeName() {
        return TypeName;
    }

    public Printer(int id, String name, int price, LocalDate start_date, LocalDate end_date) {
        super( name, price, start_date, end_date);
    }

    @Override
    public String toString() {
        return "设备类型："+getTypeName()+", "+super.toString();
    }
    public Printer() {

    }
}

```

### Microcomputer类

```java
import java.time.LocalDate;

public class Microcomputer extends Equipment{
    public  final String TypeName="微机";//设备种类名

    public String getTypeName() {
        return TypeName;
    }

    public Microcomputer(int id, String name, int price, LocalDate start_date, LocalDate end_date)
    {
        super(name, price, start_date, end_date);
    }

    @Override
    public String toString() {
        return "设备类型："+getTypeName()+", "+super.toString();
    }

    public Microcomputer() {

    }
}
```

### Scan类

```java
import java.time.LocalDate;

public class Scan extends Equipment {

    public final  String TypeName = "扫描机";//设备种类名

    public String getTypeName() {
        return TypeName;
    }

    @Override
    public String toString() {
        return "设备类型："+getTypeName()+", "+super.toString();
    }

    public Scan() {

    }

    public Scan(String name, int price, LocalDate start_date, LocalDate end_date) {
        super(name, price, start_date, end_date);
    }
}
```

## Handler接口

实现这些业务并不难，令人麻烦的是如何保持程序的**健壮性**（程序在非预期输入、异常条件或错误操作下，仍能维持正常运行或优雅失败的能力）课程设计老师一直强调这个（别再展示的时候，输入一个错误格式就崩溃了），也是成功被烦到了。

```java
public interface Handler {
    void work();

}
```

### Handler_add类

添加功能：

提供选择添加设备类型，可设置设备名，购入价格，购入日期，报废日期。

添加设备：必须要传这三个参数

```java
    private ArrayList<Equipment> equipment;
    private HashMap<String, ArrayList<Equipment>> type_map;
    private HashMap<String, Class> class_mp;
```

添加的新设备必须加入equipment和type_map。当时设计type_map的时候，想的是做个分类集合好统计每种设备的数量，省的每次统计还要遍历集合equipment，写到后面才发现，woc，有点麻烦了。添加的时候你不只要添加进equipment，也要添加进type_map里面去，删除的时候也是这样的。增删的时候equipment和type_map是连在一起的。不然数据**不同步**就糟了。

至于class_map的使用，它会用到反射机制。不知道反射机制的小伙伴可以先去了解一下反射机制。

```java
import java.lang.reflect.InvocationTargetException;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.InputMismatchException;
import java.util.Scanner;

public class Handler_add implements Handler{
    private ArrayList<Equipment> equipment;
    private HashMap<String, ArrayList<Equipment>> type_map;
    private HashMap<String, Class> class_mp;
    @Override
    public void work() {
        Scanner sc = new Scanner(System.in);
         To: while (true){
              print_Choices();//显示选择
              String choice = sc.next();
              while(!class_mp.containsKey(choice)){
                  if(choice.equals("0"))break;
                  System.out.println("输入错误！");
                  choice = sc.next();
              }
              if(choice.equals("0")){
                  break;
              }
              add(choice);
              System.out.println("是否继续：1.继续 0.停止");
              String choice1 = "";
              while(true){
                  choice1 = sc.next();
                  if(choice1.equals("0"))break To;
                  else if(choice1.equals("1"))continue To;
                  else {
                      System.out.println("您输入的格式错误");
                  }
              }
//              if (sc.hasNextInt()){
//                  int x = sc.nextInt();
//                  if (x == 1){
//                      continue;
//                  }else if (x == 0){
//                      break;
//                  }
//              }else {
//                  System.out.println("您输入的格式错误");
//                  continue;
//              }
          }
    }
    /*
    * 添加信息
    * */
    public void add(String type) {
        Class c = class_mp.get(type);
        Equipment tamp = null;
        try {
            System.out.println(c.getName());
            //调用无参构造器
            tamp = (Equipment) c.getDeclaredConstructor().newInstance();
        } catch (InstantiationException e) {
            throw new RuntimeException(e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        } catch (InvocationTargetException e) {
            throw new RuntimeException(e);
        } catch (NoSuchMethodException e) {
            throw new RuntimeException(e);
        }
        Scanner sc = new Scanner(System.in);
        System.out.println("输入设备名：");
        tamp.setName(sc.next());
        System.out.println("输入设备价格：");
        while (true){
             if (sc.hasNextInt()){
                 int choice = sc.nextInt();
                tamp.setPrice(choice);
                break;
            }else {
                System.out.println("格式格式错误，重新输入：");
                sc.next();
            }
        }
//        System.out.println("是否报废：1.报废 0.不报废");
//        while (true){
//            int choice = sc.nextInt();
//            if (choice == 1){
//                tamp.setBool_scrap(true);
//                break;
//            }else if (choice == 0){
//                tamp.setBool_scrap(false);
//                break;
//            }else {
//                System.out.println("格式格式错误，重新输入：");
//            }
//        }
        tamp.setStart_date(in_date("购入"));
        tamp.setEnd_date(in_date("报废"));
        equipment.add(tamp);
        type_map.get(tamp.getTypeName()).add(tamp);
    }
    /*
    * 添加选项
    * */
    public void print_Choices(){
        System.out.print("可选择的添加的设备有：");
        int cnt=1;
        for(var e:type_map.keySet()){
            System.out.print(cnt+"."+e+"\t");
            cnt++;
        }
        System.out.println("0.退出");
    }
/*
* 添加日期
* */
    public LocalDate in_date(String s){
        Scanner scanner = new Scanner(System.in);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("y-M-d");

        LocalDate date = null;
        boolean validInput = false;

        while (!validInput) {
            try {
                System.out.print("请输入"+s+"日期（格式：YYYY-MM-DD，例如 2023-10-25）: ");
                String input = scanner.nextLine(); // 读取用户输入

                date = LocalDate.parse(input, formatter); // 解析为 LocalDate
                validInput = true; // 输入正确，退出循环

            } catch (DateTimeParseException e) {
                System.out.println("错误：日期格式无效，请按格式重新输入！");
            }
        }
        return date;
    }

    public Handler_add(DataBase dataBase) {
        this.equipment = dataBase.getEquipment();
        this.type_map = dataBase.getType_map();
        int cnt=1;
        class_mp = new  HashMap<String, Class>();
        for(var e:dataBase.getClass_map().keySet()){
              class_mp.put(Integer.toString(cnt),dataBase.getClass_map().get(e));
              cnt++;
        }
    }
}
```

### Handler_check类

设备检查

以当天时间为标准，检查有哪些设备已经达到报废，用户可以选择是否删除报废设备。

```java
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Scanner;

public class Handler_check implements Handler{
    private ArrayList<Equipment> equipment;
    private HashMap<String, ArrayList<Equipment>> type_map;

    @Override
    public void work() {
          System.out.println("今天是"+ LocalDate.now().toString()+"达到报废日期的设备有：");
          ArrayList<Equipment> baofei = new ArrayList<Equipment>();
          for(Equipment e:equipment)
              if (e.getEnd_date().compareTo(LocalDate.now()) < 0) {
                  baofei.add(e);
                  e.setBool_scrap(true);
                  System.out.println(e.toString());
              }
          if(baofei.size()==0){System.out.println("总共：0 台");}
          else {System.out.println("总共："+baofei.size()+"台");}
          System.out.println("是否删除报废设备：1.是 0或任意字符.否");
          Scanner sc = new Scanner(System.in);
          String choice = sc.next();
          if(choice.equals("1")){
              for(var x:baofei){
                  equipment.remove(x);
                  type_map.get(x.getTypeName()).remove(x);
              }
              System.out.println("成功删除！");
          }
          System.out.println("按任意键继续>>>>>>>>");
          sc.nextLine();
          sc.nextLine();
    }

    public Handler_check(DataBase dataBase) {
        this.equipment = dataBase.getEquipment();
        this.type_map = dataBase.getType_map();
    }
}
```

### handler_count类

设备统计

提供统计总设备数，和每种设备数目，用户可以选择性打印每种设备集合的各个信息。

```java
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Scanner;

public class Handler_count implements Handler{
    private ArrayList<Equipment> equipment;
    private HashMap<String, ArrayList<Equipment>> type_map;
    private HashMap<String, String> show_map;
    Scanner sc = new Scanner(System.in);
    @Override
    public void work() {
         int cnt = 0;
         cnt=equipment.size();
         System.out.println("总共有"+cnt+"台设备");
         System.out.println("每种设备如下：");
         for(String key:type_map.keySet()){
             System.out.println(key+":"+type_map.get(key).size()+"(台)");
         }
         while(true){
             chioce_print();
             String chioce="";
             while(!show_map.containsKey(chioce)){
                 chioce=sc.next();
                 if(chioce.equals("0")){
                     break;
                 }
                 if(!show_map.containsKey(chioce))System.out.println("格式错误，重新输入！");
             }
             if(chioce.equals("0")){
                 break;
             }
             show_part(chioce);
         }
         System.out.println("按回车键继续>>>>>>>");
        sc.nextLine();
        sc.nextLine();
    }

    public void show_all(){
        System.out.println("");
    }

    public void show_part(String type){
        type=show_map.get(type);
        for(var x:type_map.get(type)){
            System.out.println(x.toString());
        }
    }

    public void chioce_print(){
        System.out.println("选择打印：");
        for(var x:show_map.keySet()){
            System.out.print(x+"."+show_map.get(x)+" ");
        }
        System.out.println("0.退出");
    }

    public Handler_count(DataBase dataBase) {
        this.equipment = dataBase.getEquipment();
        this.type_map = dataBase.getType_map();
        show_map=new HashMap<String,String>();
        int cnt=1;
        for(var x:type_map.keySet()){
            show_map.put(Integer.toString(cnt),x);
            cnt++;
        }
    }
}
```

### handler_set类

设备修改

提供以编号名确定设备并提供修改字段的选项。

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Scanner;

public class Handler_set implements Handler{
    private ArrayList<Equipment> equipment;
    private Handler_add handler;
    Scanner scanner = new Scanner(System.in);
    @Override
    public void work() {
        To:while(true){
            System.out.println("输入要修改的设备编号：(0)退出");
            while (!scanner.hasNextInt()) {
                String s = scanner.next();
                if(s.equals("0"))break To;
                System.out.println("格式错误，重新输入：");
            }
            int id = scanner.nextInt();
            if(id==0)break To;
            Equipment tamp = new Equipment();
            if ((tamp=check(id))!=null) {
                System.out.println("存在该设备");
                set(tamp);
            }else{
                System.out.println("不存在该设备");
            }
        }
    }
    public Equipment check(int num){
        int start=0;int end=equipment.size()-1;
        if(num<equipment.get(start).getNumber()&&num>equipment.get(end).getNumber()){
            return null;
        }
        while(start<=end){
            int mid = (start+end)/2;
            if(equipment.get(mid).getNumber()==num){
                return equipment.get(mid);
            }
            if(num<equipment.get(mid).getNumber()){
                end = mid-1;
            }else if(num>equipment.get(mid).getNumber()){
                start = mid+1;
            }
        }
        return null;
    }
    public void set(Equipment tamp)
    {
        String chioce = "修改选项：1.设备名 2.购入价格 3.购入日期 4.报废日期 0.退出";
        while (true) {
            System.out.println(chioce);
            String ch = scanner.next();
            switch (ch) {
                case "1":
                    set_name(tamp);
                    break;
                case "2":
                    set_price(tamp);
                    break;
                case "3":
                    tamp.setStart_date(handler.in_date("修改的购入"));
                    break;
                case "4":
                    tamp.setEnd_date(handler.in_date("修改的报废"));
                    break;
                case "0": {
                    break;
                }
                default: {
                    System.out.println("格式错误,重新输入！");
                    break;
                }
            }
            if (ch.equals("0")) {
                break;
            }
        }
    }
    void set_name(Equipment tamp){
        System.out.println("输入修改设备名：");
        tamp.setName(scanner.next());
        System.out.println("修改成功");
    }
    void set_price(Equipment tamp){
        System.out.println("输入修改购入价格：");
        while (true){
            if (scanner.hasNextInt()){
                int choice = scanner.nextInt();
                tamp.setPrice(choice);
                break;
            }else {
                System.out.println("格式格式错误，重新输入：");
                scanner.next();
            }
        }
        System.out.println("修改成功");
    }
    Handler_set(DataBase dataBase) {
        this.equipment=dataBase.getEquipment();
        this.handler= (Handler_add) dataBase.getCd().get("1");
    }
}
```

### Handler_del类

删除功能：

提供按照编号（编号采取自增设置）和设备名删除。

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Scanner;

public class Handler_del implements Handler{
    private ArrayList<Equipment> equipment;
    private HashMap<Integer,Equipment>mpNum;
    private HashMap<String,Equipment>mpName;
    private HashMap<String,ArrayList<Equipment>>type_map;
    @Override
    public void work() {
        init();
        Scanner scanner = new Scanner(System.in);
        To:while(true) {
            System.out.println("选择删除模式：1，按编号删除 2.按设备名删除 0.退出");
            String type = scanner.next();
            if(type.equals("0"))break;
            if (type.equals("1")) {
                System.out.println("输入编号：");
                int sign=0;
                int id=-1;
                while(sign==0) {
                    while (!scanner.hasNextInt()) {
                        String s = scanner.next();
                        if(s.equals("0"))break To;
                        System.out.println("格式错误，重新输入：");
                    }
                     id = scanner.nextInt();
                    if(id==0)break To;
                    if (!mpNum.containsKey(id)) {
                        System.out.println("不存在该编号，重新输入：");
                        //id = scanner.nextInt();
                        if(id==0)break To;
                    }else{
                        sign=1;
                        System.out.println("该编号存在，是否删除，1.是 0或任意字符. 否");
                    }
                }
                String s = scanner.next();
                if (s.equals("1")) {
                        Equipment remove = mpNum.get(id);
                        Remove(remove);
                        System.out.println("成功删除！");
                    }else continue;

            } else if (type.equals("2")) {
                  String name="";
                  System.out.println("输入设备名：");
                  while(!mpName.containsKey(name)) {
                      name = scanner.next();
                      if(name.equals("0"))break To;
                      if(!mpName.containsKey(name))System.out.println("不存在该设备,重新输入：");
                  }
                System.out.println("该设备号存在，是否删除，1.是 0或任意字符. 否");
                String s = scanner.next();
                  if (s.equals("1")) {
                      Equipment remove = mpName.get(name);
                      Remove(remove);
                      System.out.println("成功删除！");
                  }else continue;

            } else {
                System.out.println("格式错误，重新输入：");
            }
            System.out.println("按回车键继续>>>>>>>");
            scanner.nextLine();
            scanner.nextLine();
        }
    }
    public void init(){
        this.mpNum = new HashMap<Integer,Equipment>();
        this.mpName = new HashMap<String,Equipment>();
        for(Equipment e: equipment) {
            mpNum.put(e.getNumber(), e);
            mpName.put(e.getName(), e);
        }
    }
    public void Remove(Equipment remove){
        equipment.remove(remove);
        mpNum.remove(remove.getNumber(),remove);
        mpName.remove(remove.getName(),remove);
        type_map.get(remove.getTypeName()).remove(remove);
    }
    public Handler_del(DataBase dataBase) {
        this.equipment = dataBase.getEquipment();
        this.type_map = dataBase.getType_map();
    }
}
```

### Handler_query类

设备查询

提供以编号和设备名查询方式，以及全部查询功能。

```java
import java.util.ArrayList;
import java.util.Scanner;

public class Handler_query implements Handler{
    private ArrayList<Equipment> equipment;
    Scanner sc=new Scanner(System.in);
    @Override
    public void work() {
        String choice="";
        while(!choice.equals("0")){
            System.out.println("1.按编号查询 2.按设备名查询 3.查询全部 0.退出");
            choice=sc.next();
            switch(choice){
                case "1":
                    find();
                    break;
                case "2":
                    find2();
                    break;
                case "0":
                    break;
                case "3":
                    find_3();
                    break;
                default:
                    System.out.println("错误！");
            }
        }
    }
    public void find(){
        System.out.println("输入编号：");
        int num=0;
        while(!sc.hasNextInt()){
            sc.next();
            System.out.println("数字！数字！数字！");
        }
        num=sc.nextInt();
//        sc.nextLine();
        int sign=0;
        for(Equipment e: equipment) {
            if (e.getNumber()==num) {
                sign=1;
                System.out.println(e.toString());
                break;
            }
        }
        if(sign==0){
            System.out.println("没有该编号。");
        }
    }
    public void find2(){
        System.out.println("输入设备名：");
        String name=sc.next();
        int sign=0;
        for(Equipment e: equipment) {
            if(name.equals(e.getName())){
                sign=1;
                System.out.println(e.toString());
                break;
            }
        }
        if(sign==0){
            System.out.println("没有该设备。");
        }
    }
    public void find_3(){
        for (Equipment x : equipment){
            System.out.println(x.toString());
        }
    }

    public Handler_query(DataBase dataBase) {
        equipment = dataBase.getEquipment();
    }
}
```

# 小结

本人的这个课程设计没有使用Java图形化组件。虽然课程设计老师表示写出个界面就可以加分，但写完这些的我早已精疲力尽，累累累。

有实力的小伙伴，可以尝试一下用图形化界面来实现这个。如果觉得上面的功能写的不太好的，小伙伴们也可以试试自己去添加一下新功能，Database类的内容基本不需要这么改，只需要动这么几个地方就可以了。

```Java
        //添加型类型的设备
        type_map.put("新设备名",new ArrayList<Equipment>());
        Class_map.put("新设备名"",new_Equipment.class);
        //添加新的处理命令
        cd.put("新处理命令", new Handler_new(this));
```

对于面向对象的语言，可拓展性强确实是它的魅力。

最后，本人第一次写博客，有什么地方写的不好的可以提出来，希望大家多多包涵。
