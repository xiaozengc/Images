> 本项目的主要用途
- 存放截图或者图片到服务端
- 用于md文件的图片访问
- 学习和熟练git指令

> 目录解析
- 记录每天保存的图片
- 图片名称按照000.xxx一直到nnn.xxx的形式来保存

---
> 第一次成功案例
### 实例说明：

1. 创建一个用户表

```
CREATE TABLE user1(
    id VARCHAR2(10) PRIMARY KEY,
    user_name VARCHAR2(10), --写法规范
    sex NUMBER(1),
    minzu NUMBER(2)
)

INSERT INTO user1 VALUES(1,'小明',1,1);
INSERT INTO user1 VALUES(2,'小芳',0,2);
COMMIT;
SELECT * FROM user1;
```

- 为什么要用数字类型？
    - 因为每个国家对于性别和民族的显示都是不同标准的
    - 所以需要通过字典来标明对应的含义

2. 创建字典表

```
CREATE TABLE dicts(
  id NUMBER PRIMARY KEY,
  display VARCHAR2(100), --显示内容
  key_value NUMBER, --真实的值
  key_type VARCHAR2(20), --值的类型
  lang VARCHAR2(10) --语言
);

INSERT INTO dicts VALUES(1,'男',1,'sex','cn');
INSERT INTO dicts VALUES(2,'女',0,'sex','cn');
INSERT INTO dicts VALUES(3,'boy',1,'sex','en');
INSERT INTO dicts VALUES(4,'girl',0,'sex','en');
INSERT INTO dicts VALUES(5,'汉族',1,'minzu','cn');
INSERT INTO dicts VALUES(6,'傣族',2,'minzu','cn');
INSERT INTO dicts VALUES(7,'hanzu',1,'minzu','en');
INSERT INTO dicts VALUES(8,'daizu',2,'minzu','en');
SELECT * FROM dicts
```

3. 要求：查询出以下图片的结果

![image](https://github.com/xiaozengc/Images/raw/master/2018.3.13/000.png)

```
SELECT
    user1.id,
    user1.user_name,
    dic_sex.display as sex,
    dic_minzu.display as minzu
FROM 
    user1,dicts dic_sex,dicts dic_minzu -- 一张表虚化出多张表使用
WHERE
    dic_sex.key_value = sex AND dic_sex.key_type = 'sex' AND dic_sex.lang = 'cn'
AND
    dic_minzu.key_value = minzu AND dic_minzu.key_type = 'minzu' AND dic_minzu.lang ='cn'
```

> 运行结果如下

![image](https://github.com/xiaozengc/Images/raw/master/2018.3.13/001.png)

- 这里就出现一个新的概念：
    - 这里的实例，需要在一张表中使用到多个字段，
    - 因为用户表在使用字典表的时候，需要查询到sex和minzu两个字段对应的字典信息
    - 而字典表只有一张
    - 所以就需要虚化出来，不要把表当成一个表，而是可以灵活操作的数据
    - 当然，表不单单可以虚化，还可以把查询出来的结果当成一个表来操作
    - 如下：

```
SELECT
    user1.id,
    user1.user_name,
    dic_sex.display as sex,
    dic_minzu.display as minzu
FROM 
    user1,
    (SELECT * FROM dicts WHERE key_type='sex' and lang='cn') dic_sex,
    (SELECT * FROM dicts WHERE key_type='minzu' and lang = 'cn') dic_minzu
    --查询出来的结果，当作一个表来操作
WHERE
    dic_sex.key_value = sex
AND
    dic_minzu.key_value = minzu;
```

> 运行结果如下

![image](https://github.com/xiaozengc/Images/raw/master/2018.3.13/002.png)

- 优化（左关联）
    - 由于字典也不一定就是可以拥有每个用户数据对应的信息
    - 可能在性别上，用户选择是 2
    - 而字典中，又没有对应的信息
    - 如果只是内关联，那那些没有匹配到的用户就不会显示
    - 这样就不符合业务逻辑了

```
SELECT
    user1.id,
    user1.user_name,
    dic_sex.display as sex,
    dic_minzu.display as minzu
FROM
    user1
LEFT JOIN
    (SELECT * FROM dicts WHERE key_type = 'sex' and lang = 'cn') dic_sex
ON
    dic_sex.key_value = sex
LEFT JOIN
    (SELECT * FROM dicts WHERE key_type = 'minzu' and lang = 'cn') dic_minzu
ON
    dic_minzu.key_value = minzu
```

> 运行结果如下

![image](https://github.com/xiaozengc/Images/raw/master/2018.3.13/003.png)
