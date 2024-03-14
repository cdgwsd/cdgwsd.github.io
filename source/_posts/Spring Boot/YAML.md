---
categories:
  - Spring Boot
---
# YAML

## 基本语法

- 语法格式为 `key: value`；<font color=red>冒号后有空格</font>
- 大小写敏感
- 以缩进代表层级关系
- 缩进不允许使用 tab 只能使用空格
- 空格的个数不重要，只要相同层级的元素左对齐即可
- 字符串不需要使用引号包裹
  - 使用单引号包裹时会转义特殊字符
  - 使用双引号包裹时不转义特殊字符
## 数据类型
### 字面量
单个的，不可再分的值，比如：数字、字符串、布尔值

语法：`key: value`

```yml
name: 张三
age: 10
birth: 2000/10/10
```
### 对象
键值对的集合，比如 map、set、hash、object

语法：

1. 行内写法：`k: {k1: v1,k2: v2,k3: v3}`

2. 换行写法：

   ```yaml
   k:
       k1: V1
       K2: V2
       K3: V3
   ```

示例：

```yaml
person:
    name: 张三
    age: 10
    car:
        name: 奔驰
        price: 99999
    student: {name: 李四,age: 11}
```

### 数组
一组按顺序排列的值，比如：array、list、queue

语法：

1. 行内写法：`k: [v1,v2,v3]`

2. 换行写法：

   ```yaml
   k:
       - v1
       - v2
       - v3
   ```

示例：

   ```yaml
   hobby: [篮球,足球,排球]
   student:
       - 张三
       - 李四
       - 王二
   ```

## 应用实例
使用 YAML 配置文件与 `@ConfigurationProperties` 注解完成 bean 的属性注入与返回

定义实体类

```java
@Data  
public class Car {  
    private String name;  
    private int price;  
}
```
```java
@Data  
@Component  
@ConfigurationProperties("monster")  
public class Monster {  
    private Integer id;  
    private String name;  
    private Integer age;  
    private Boolean isMarried;  
    private Date birth;  
    private Car car;  
    private String[] skill;  
    private List<String> hobby;  
    private Map<String, Object> wife;  
    private Set<Double> salaries;  
    private Map<String, List<Car>> cars;  
}
```
定义控制器
```java
@RestController  
public class HelloController {  

    @Resource  
    private Monster monster;  

    @RequestMapping("/monster")  
    public Monster hello(){  
        return monster;  
    }  
}
```
定义配置文件
```yaml
monster:  
    id: 100  
    name: 牛魔王  
    age: 500  
    isMarried: true  
    birth: 253/3/23  
    car: {name: 奔驰,price: 9999999}  
    skill:  
        - 芭蕉扇  
        - 牛魔拳  
    hobby: [喝酒,吃肉]  
    wife: {no1: 玉面狐狸,no2: 铁扇公主}  
    salaries: [10000,200000]  
    cars:  
    	group1: [{name: 宝马, price: 1000000},{name: 法拉利, price: 1200000}]  
    	group2:  
            - {name: 奔驰, price: 2000000}  
            - name: 迈巴赫
              price: 3000000
```
测试
![](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202304041733379.png)
