# HIVE50道经典练习题，带答案以及查询结果

![img](https://csdnimg.cn/release/phoenix/template/new_img/original.png)

[浅谈_](https://me.csdn.net/wangqinyi574110) 2020-05-30 17:43:18 ![img](https://csdnimg.cn/release/phoenix/template/new_img/articleReadEyes.png) 425 ![img](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollect.png) 收藏 2

分类专栏： [hive](https://blog.csdn.net/wangqinyi574110/category_10030610.html)

版权

##### 建表语句

```sql
CREATE TABLE STUDENT(
S_ID STRING,
S_NAME STRING,
S_BIRTH STRING,
S_SEX STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

CREATE TABLE COURSE(
 C_ID STRING,
 C_NAME STRING,
 T_ID STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

CREATE TABLE TEACHER(
T_ID STRING,
T_NAME STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

CREATE TABLE SCORE (
S_ID STRING,
C_ID STRING,
S_SCORE INT
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
123456789101112131415161718192021222324252627
```

##### 数据

- ```sh
  vi /export/data/hivedatas/student.csv
  
  01 赵雷 1990-01-01 男
  02 钱电 1990-12-21 男
  03 孙风 1990-05-20 男
  04 李云 1990-08-06 男
  05 周梅 1991-12-01 女
  06 吴兰 1992-03-01 女
  07 郑竹 1989-07-01 女
  08 王菊 1990-01-20 女
  12345678910
  ```

- ```sh
  vi /export/data/hivedatas/course.csv
  01	语文	02
  02	数学	01
  03	英语	03
  1234
  ```

- ```sh
  vi /export/data/hivedatas/teacher.csv
  01	张三
  02	李四
  03	王五
  1234
  ```

- ```sh
  vi /export/data/hivedatas/score.csv
  01	01	80
  01	02	90
  01	03	99
  02	01	70
  02	02	60
  02	03	80
  03	01	80
  03	02	80
  03	03	80
  04	01	50
  04	02	30
  04	03	20
  05	01	76
  05	02	87
  06	01	31
  06	03	34
  07	02	89
  07	03	98
  12345678910111213141516171819
  ```

##### 题

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530174143473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdxaW55aTU3NDExMA==,size_16,color_FFFFFF,t_70#pic_center)

1. **查询"01"课程比"02"课程成绩高的学生的信息及课程分数:**

   - ```sql
     WITH A AS(
      SELECT * FROM SCORE WHERE C_ID ='01'
     ),
     B AS(
      SELECT * FROM SCORE WHERE C_ID ='02'
     ),
     C AS (
      SELECT S.*,A.S_SCORE AS FIRST_SCORE, B.S_SCORE AS SECOND_SCORE FROM STUDENT S,A,B WHERE S.S_ID = A.S_ID AND S.S_ID = B.S_ID
     )
      SELECT * FROM C WHERE  C.FIRST_SCORE>C.SECOND_SCORE
      
      +---------+-----------+-------------+----------+----------------+-----------------+--+
     | c.s_id  | c.s_name  |  c.s_birth  | c.s_sex  | c.first_score  | c.second_score  |
     +---------+-----------+-------------+----------+----------------+-----------------+--+
     | 02      | 钱电        | 1990-12-21  | 男        | 70             | 60              |
     | 04      | 李云        | 1990-08-06  | 男        | 50             | 30              |
     +---------+-----------+-------------+----------+----------------+-----------------+--+
     1234567891011121314151617
     ```

2. **查询"01"课程比"02"课程成绩低的学生的信息及课程分数:**

   - ```sql
     WITH A AS(
      SELECT * FROM SCORE WHERE C_ID ='01'
     ),
     B AS(
      SELECT * FROM SCORE WHERE C_ID ='02'
     ),
     C AS (
      SELECT S.*,A.S_SCORE AS FIRST_SCORE, B.S_SCORE AS SECOND_SCORE FROM STUDENT S,A,B WHERE S.S_ID = A.S_ID AND S.S_ID = B.S_ID
     )
      SELECT * FROM C WHERE  C.FIRST_SCORE<C.SECOND_SCORE
      
      +---------+-----------+-------------+----------+----------------+-----------------+--+
     | c.s_id  | c.s_name  |  c.s_birth  | c.s_sex  | c.first_score  | c.second_score  |
     +---------+-----------+-------------+----------+----------------+-----------------+--+
     | 01      | 赵雷        | 1990-01-01  | 男        | 80             | 90              |
     | 05      | 周梅        | 1991-12-01  | 女        | 76             | 87              |
     +---------+-----------+-------------+----------+----------------+-----------------+--+
     1234567891011121314151617
     ```

3. **查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩:**

   - ```sql
     WITH A AS (
       SELECT S_ID,AVG(S_SCORE) AS AVG_SCORE FROM SCORE GROUP BY S_ID HAVING AVG_SCORE>=60  
     )
     SELECT A.S_ID,B.S_NAME,ROUND(A.AVG_SCORE,2) AVG_SCORE FROM A LEFT JOIN STUDENT B ON A.S_ID = B.S_ID;  
     
     +---------+-----------+--------+--+
     | a.s_id  | b.s_name  |  avg_score|
     +---------+-----------+--------+--+
     | 01      | 赵雷        | 89.67  |
     | 02      | 钱电        | 70.0   |
     | 03      | 孙风        | 80.0   |
     | 05      | 周梅        | 81.5   |
     | 07      | 郑竹        | 93.5   |
     +---------+-----------+--------+--+
     1234567891011121314
     ```

4. **查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩:– (包括有成绩的和无成绩的)**

   - ```sql
     SELECT B.S_ID,B.S_NAME,ROUND(IF(AVG(A.S_SCORE) IS NULL,0,AVG(A.S_SCORE)),2) AS AVG_SCORE FROM SCORE A RIGHT JOIN STUDENT B ON A.S_ID = B.S_ID GROUP BY B.S_ID,B.S_NAME HAVING IF(AVG(A.S_SCORE) IS NULL,0,AVG(A.S_SCORE))<60 
     
     +---------+-----------+------------+--+
     | b.s_id  | b.s_name  | avg_score  |
     +---------+-----------+------------+--+
     | 04      | 李云        | 33.33      |
     | 06      | 吴兰        | 32.5       |
     | 08      | 王菊        | 0.0        |
     +---------+-----------+------------+--+
     123456789
     ```

5. **查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩:**

   - ```sql
     SELECT S.S_ID,S.S_NAME,SIZE(COLLECT_SET(C.C_ID)) CHANGE_COURSE_SUM,IF(SUM(C.S_SCORE) IS NULL,0, SUM(C.S_SCORE)) SUM_SCORE FROM STUDENT S LEFT JOIN SCORE C ON S.S_ID = C.S_ID GROUP BY S.S_ID,S.S_NAME
     
     +---------+-----------+--------------------+------------+--+
     | s.s_id  | s.s_name  | change_course_sum  | sum_score  |
     +---------+-----------+--------------------+------------+--+
     | 01      | 赵雷        | 3                  | 269        |
     | 02      | 钱电        | 3                  | 210        |
     | 03      | 孙风        | 3                  | 240        |
     | 04      | 李云        | 3                  | 100        |
     | 05      | 周梅        | 2                  | 163        |
     | 06      | 吴兰        | 2                  | 65         |
     | 07      | 郑竹        | 2                  | 187        |
     | 08      | 王菊        | 0                  | 0          |
     +---------+-----------+--------------------+------------+--+
     1234567891011121314
     ```

6. **查询"李"姓老师的数量:**

   - ```sql
     SELECT COUNT(*) AS COUNT FROM TEACHER WHERE T_NAME LIKE '李%';
     
     +------+--+
     | COUNT|
     +------+--+
     | 1    |
     +------+--+
     1234567
     ```

7. **查询学过"张三"老师授课的同学的信息:**

   - ```sql
     WITH T AS(
       SELECT C.C_ID FROM COURSE C LEFT JOIN TEACHER T ON C.T_ID = T.T_ID WHERE T.T_NAME = '张三'
     ),
     T1 AS (
      SELECT DISTINCT S.S_ID FROM T LEFT JOIN SCORE S ON T.C_ID = S.C_ID  
     )
     SELECT * FROM T1 LEFT JOIN STUDENT S ON S.S_ID = T1.S_ID;
     
     +-----------+-------------+--------------+------------+--+
     | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  |
     +-----------+-------------+--------------+------------+--+
     | 01        | 赵雷          | 1990-01-01   | 男          |
     | 02        | 钱电          | 1990-12-21   | 男          |
     | 03        | 孙风          | 1990-05-20   | 男          |
     | 04        | 李云          | 1990-08-06   | 男          |
     | 05        | 周梅          | 1991-12-01   | 女          |
     | 07        | 郑竹          | 1989-07-01   | 女          |
     +-----------+-------------+--------------+------------+--+
     123456789101112131415161718
     ```

8. **查询没学过"张三"老师授课的同学的信息:**

   - ```sql
     WITH T AS(
       SELECT C.C_ID FROM COURSE C LEFT JOIN TEACHER T ON C.T_ID = T.T_ID WHERE T.T_NAME = '张三'
     ),
     T1 AS (
      SELECT DISTINCT S.S_ID FROM T LEFT JOIN SCORE S ON T.C_ID = S.C_ID  
     )
     SELECT S.* FROM T1 RIGHT JOIN STUDENT S ON  S.S_ID = T1.S_ID WHERE T1.S_ID IS NULL
     
     +---------+-----------+-------------+----------+--+
     | s.s_id  | s.s_name  |  s.s_birth  | s.s_sex  |
     +---------+-----------+-------------+----------+--+
     | 06      | 吴兰        | 1992-03-01  | 女        |
     | 08      | 王菊        | 1990-01-20  | 女        |
     +---------+-----------+-------------+----------+--+
     1234567891011121314
     ```

9. **查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息:**

   - ```sql
     WITH A AS(
      SELECT * FROM SCORE WHERE C_ID ='01'
     ),
     B AS(
      SELECT * FROM SCORE WHERE C_ID ='02'
     )
     SELECT STU.* FROM STUDENT STU JOIN B ON B.S_ID = STU.S_ID JOIN A ON A.S_ID = STU.S_ID;
     
     +-----------+-------------+--------------+------------+--+
     | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  |
     +-----------+-------------+--------------+------------+--+
     | 01        | 赵雷          | 1990-01-01   | 男          |
     | 02        | 钱电          | 1990-12-21   | 男          |
     | 03        | 孙风          | 1990-05-20   | 男          |
     | 04        | 李云          | 1990-08-06   | 男          |
     | 05        | 周梅          | 1991-12-01   | 女          |
     +-----------+-------------+--------------+------------+--+
     1234567891011121314151617
     ```

10. **查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息:**

    - ```sql
      WITH A AS(
       SELECT STU.* FROM SCORE A LEFT JOIN STUDENT STU ON A.S_ID = STU.S_ID WHERE A.C_ID ='01'
      ),
      B AS(
        SELECT STU.* FROM SCORE A LEFT JOIN STUDENT STU ON A.S_ID = STU.S_ID WHERE A.C_ID ='02'
      )
      SELECT A.* FROM A LEFT JOIN B ON A.S_ID = B.S_ID WHERE B.S_ID IS NULL
      
      +---------+-----------+-------------+----------+--+
      | a.s_id  | a.s_name  |  a.s_birth  | a.s_sex  |
      +---------+-----------+-------------+----------+--+
      | 06      | 吴兰        | 1992-03-01  | 女        |
      +---------+-----------+-------------+----------+--+
      12345678910111213
      ```

11. **查询没有学全所有课程的同学的信息:**

    - ```sql
      WITH A AS(
       SELECT COUNT(DISTINCT C_ID) COUNT FROM COURSE
      ),
      B AS (
       SELECT SIZE(COLLECT_SET(SCORE.C_ID)) NUM,STUDENT.S_ID FROM SCORE RIGHT JOIN STUDENT ON SCORE.S_ID = STUDENT.S_ID GROUP BY STUDENT.S_ID
      ),
      C AS (
        SELECT S_ID FROM B,A WHERE A.COUNT != B.NUM
      )
      SELECT STUDENT.* FROM C ,STUDENT WHERE C.S_ID = STUDENT.S_ID  
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 05            | 周梅              | 1991-12-01       | 女              |
      | 06            | 吴兰              | 1992-03-01       | 女              |
      | 07            | 郑竹              | 1989-07-01       | 女              |
      | 08            | 王菊              | 1990-01-20       | 女              |
      +---------------+-----------------+------------------+----------------+--+
      12345678910111213141516171819
      ```

12. **查询至少有一门课与学号为"01"的同学所学相同的同学的信息:**

    - ```sql
      WITH A AS(
        SELECT C_ID FROM SCORE WHERE S_ID ='01' 
      ),
      B AS (
          SELECT DISTINCT S.S_ID  FROM SCORE S LEFT JOIN A ON A.C_ID = S.C_ID WHERE S.S_ID !='01' 
      )
      SELECT STUDENT.* FROM B LEFT JOIN STUDENT ON B.S_ID = STUDENT.S_ID
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 02            | 钱电              | 1990-12-21       | 男              |
      | 03            | 孙风              | 1990-05-20       | 男              |
      | 04            | 李云              | 1990-08-06       | 男              |
      | 05            | 周梅              | 1991-12-01       | 女              |
      | 06            | 吴兰              | 1992-03-01       | 女              |
      | 07            | 郑竹              | 1989-07-01       | 女              |
      +---------------+-----------------+------------------+----------------+--+
      123456789101112131415161718
      ```

13. **查询和"01"号的同学学习的课程完全相同的其他同学的信息:**

    - ```sql
      WITH A AS(
        SELECT COLLECT_SET(C_ID) FIRST_C_ID FROM SCORE GROUP BY S_ID HAVING S_ID ='01' 
      ),
      B AS(
       SELECT COLLECT_SET(C_ID) OTHER_C_ID,S_ID FROM SCORE GROUP BY S_ID HAVING S_ID !='01' 
      ),
      C AS (
        SELECT B.S_ID FROM B RIGHT JOIN A ON B.OTHER_C_ID = A.FIRST_C_ID
      )
      SELECT STUDENT.* FROM C LEFT JOIN STUDENT ON C.S_ID = STUDENT.S_ID
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 02            | 钱电              | 1990-12-21       | 男              |
      | 03            | 孙风              | 1990-05-20       | 男              |
      | 04            | 李云              | 1990-08-06       | 男              |
      +---------------+-----------------+------------------+----------------+--+
      123456789101112131415161718
      ```

14. **查询没学过"张三"老师讲授的任一门课程的学生姓名:**

    - ```sql
      WITH T AS(
        SELECT C.C_ID FROM COURSE C LEFT JOIN TEACHER T ON C.T_ID = T.T_ID WHERE T.T_NAME = '张三'
      ),
      T1 AS (
       SELECT DISTINCT S.S_ID FROM T LEFT JOIN SCORE S ON T.C_ID = S.C_ID  
      )
      SELECT S.* FROM T1 RIGHT JOIN STUDENT S ON  S.S_ID = T1.S_ID WHERE T1.S_ID IS NULL
      
      +---------+-----------+-------------+----------+--+
      | s.s_id  | s.s_name  |  s.s_birth  | s.s_sex  |
      +---------+-----------+-------------+----------+--+
      | 06      | 吴兰        | 1992-03-01  | 女        |
      | 08      | 王菊        | 1990-01-20  | 女        |
      +---------+-----------+-------------+----------+--+
      1234567891011121314
      ```

15. **查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩:**

    - ```sql
      WITH A AS (
       SELECT S_ID,ROUND(AVG(S_SCORE),2) AVG_SCORE FROM SCORE WHERE S_SCORE<60 GROUP BY S_ID HAVING COUNT(S_ID)>1 
      )
      SELECT STUDENT.S_ID,STUDENT.S_NAME,A.AVG_SCORE FROM A LEFT JOIN STUDENT ON A.S_ID = STUDENT.S_ID
      
      +---------------+-----------------+--------------+--+
      | student.s_id  | student.s_name  | a.avg_score  |
      +---------------+-----------------+--------------+--+
      | 04            | 李云              | 33.33        |
      | 06            | 吴兰              | 32.5         |
      +---------------+-----------------+--------------+--+
      1234567891011
      ```

16. **检索"01"课程分数小于60，按分数降序排列的学生信息:**

    - ```sql
      SELECT * FROM SCORE S LEFT JOIN STUDENT STU ON STU.S_ID = S.S_ID WHERE S.C_ID ='01' AND S.S_SCORE < 60 ORDER BY S.S_SCORE DESC ;
      
      +---------+---------+------------+-----------+-------------+--------------+------------+--+
      | s.s_id  | s.c_id  | s.s_score  | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  |
      +---------+---------+------------+-----------+-------------+--------------+------------+--+
      | 04      | 01      | 50         | 04        | 李云          | 1990-08-06   | 男          |
      | 06      | 01      | 31         | 06        | 吴兰          | 1992-03-01   | 女          |
      +---------+---------+------------+-----------+-------------+--------------+------------+--+
      12345678
      ```

17. **按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩:**

    - ```sql
      WITH B AS (
        SELECT C.C_NAME ,S.* FROM COURSE C RIGHT JOIN SCORE S ON S.C_ID = C.C_ID     
      )
      SELECT S.S_NAME,
      MAX(CASE WHEN B.C_NAME='英语' THEN B.S_SCORE ELSE 0 END) AS `英语`,
      MAX(CASE WHEN B.C_NAME='数学' THEN B.S_SCORE ELSE 0 END) AS `数学`,
      MAX(CASE WHEN B.C_NAME='语文' THEN B.S_SCORE ELSE 0 END) AS `语文`,
      ROUND(IF(AVG(B.S_SCORE) IS NULL,0,AVG(B.S_SCORE)),2) AVG_SCORE
      FROM B RIGHT JOIN STUDENT S ON S.S_ID = B.S_ID GROUP BY S.S_NAME ORDER BY AVG_SCORE DESC
      
      +-----------+-----+-----+-----+------------+--+
      | s.s_name  | 英语  | 数学  | 语文  | avg_score  |
      +-----------+-----+-----+-----+------------+--+
      | 郑竹        | 98  | 89  | 0   | 93.5       |
      | 赵雷        | 99  | 90  | 80  | 89.67      |
      | 周梅        | 0   | 87  | 76  | 81.5       |
      | 孙风        | 80  | 80  | 80  | 80.0       |
      | 钱电        | 80  | 60  | 70  | 70.0       |
      | 李云        | 20  | 30  | 50  | 33.33      |
      | 吴兰        | 34  | 0   | 31  | 32.5       |
      | 王菊        | 0   | 0   | 0   | 0.0        |
      +-----------+-----+-----+-----+------------+--+
      12345678910111213141516171819202122
      ```

18. **查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率:**

    - ```sql
      SELECT S.C_ID,C.C_NAME,MAX(S.S_SCORE) MAX_SCORE,MIN(S.S_SCORE) MIN_SCORE,ROUND(AVG(S.S_SCORE),2) AVG_SCORE ,ROUND(SUM(CASE WHEN S.S_SCORE>=60 THEN 1 ELSE 0 END)/COUNT(*),2) AS `及格率`,
      ROUND(SUM(CASE WHEN S.S_SCORE>=60 AND S.S_SCORE<80 THEN 1 ELSE 0 END)/COUNT(*),2) AS `中等率`,
      ROUND(SUM(CASE WHEN S.S_SCORE>=80 AND S.S_SCORE<90 THEN 1 ELSE 0 END)/COUNT(*),2) AS `优良率`,
      ROUND(SUM(CASE WHEN S.S_SCORE>=90 THEN 1 ELSE 0 END)/COUNT(*),2) AS `优秀率`
      FROM SCORE S LEFT JOIN COURSE C ON S.C_ID = C.C_ID  GROUP BY S.C_ID,C.C_NAME 
      
      +---------+-----------+------------+------------+------------+-------+-------+-------+-------+--+
      | s.c_id  | c.c_name  | max_score  | min_score  | avg_score  |  及格率  |  中等率  |  优良率  |  优秀率  |
      +---------+-----------+------------+------------+------------+-------+-------+-------+-------+--+
      | 01      | 语文        | 80         | 31         | 64.5       | 0.67  | 0.33  | 0.33  | 0.0   |
      | 02      | 数学        | 90         | 30         | 72.67      | 0.83  | 0.17  | 0.5   | 0.17  |
      | 03      | 英语        | 99         | 20         | 68.5       | 0.67  | 0.0   | 0.33  | 0.33  |
      +---------+-----------+------------+------------+------------+-------+-------+-------+-------+--+
      12345678910111213
      ```

19. **按各科成绩进行排序，并显示排名:**

    - ```sql
      SELECT *,ROW_NUMBER() OVER (PARTITION BY C_ID ORDER BY S_SCORE DESC) AS CN FROM SCORE; 
      
      +-------------+-------------+----------------+-----+--+
      | score.s_id  | score.c_id  | score.s_score  | cn  |
      +-------------+-------------+----------------+-----+--+
      | 03          | 01          | 80             | 1   |
      | 01          | 01          | 80             | 2   |
      | 05          | 01          | 76             | 3   |
      | 02          | 01          | 70             | 4   |
      | 04          | 01          | 50             | 5   |
      | 06          | 01          | 31             | 6   |
      | 01          | 02          | 90             | 1   |
      | 07          | 02          | 89             | 2   |
      | 05          | 02          | 87             | 3   |
      | 03          | 02          | 80             | 4   |
      | 02          | 02          | 60             | 5   |
      | 04          | 02          | 30             | 6   |
      | 01          | 03          | 99             | 1   |
      | 07          | 03          | 98             | 2   |
      | 02          | 03          | 80             | 3   |
      | 03          | 03          | 80             | 4   |
      | 06          | 03          | 34             | 5   |
      | 04          | 03          | 20             | 6   |
      +-------------+-------------+----------------+-----+--+
      123456789101112131415161718192021222324
      ```

20. **查询学生的总成绩并进行排名:**

    - ```sql
      WITH T AS (
      SELECT S_ID,SUM(S_SCORE) AS SUM_SCORE FROM SCORE GROUP BY S_ID
      ),
       T1 AS(
       SELECT *,ROW_NUMBER() OVER (ORDER BY SUM_SCORE DESC) CN FROM T
       )
       SELECT STU.*,T1.SUM_SCORE,T1.CN FROM T1 LEFT JOIN STUDENT STU ON T1.S_ID = STU.S_ID
      
      +-----------+-------------+--------------+------------+---------------+--------+--+
      | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  | t1.sum_score  | t1.cn  |
      +-----------+-------------+--------------+------------+---------------+--------+--+
      | 01        | 赵雷          | 1990-01-01   | 男          | 269           | 1      |
      | 03        | 孙风          | 1990-05-20   | 男          | 240           | 2      |
      | 02        | 钱电          | 1990-12-21   | 男          | 210           | 3      |
      | 07        | 郑竹          | 1989-07-01   | 女          | 187           | 4      |
      | 05        | 周梅          | 1991-12-01   | 女          | 163           | 5      |
      | 04        | 李云          | 1990-08-06   | 男          | 100           | 6      |
      | 06        | 吴兰          | 1992-03-01   | 女          | 65            | 7      |
      +-----------+-------------+--------------+------------+---------------+--------+--+
      12345678910111213141516171819
      ```

21. **查询不同老师所教不同课程平均分从高到低显示:**

    - ```sql
      SELECT COURSE.T_ID,SCORE.C_ID,ROUND(AVG(S_SCORE),2) AS AVG_SCORE FROM SCORE LEFT JOIN COURSE ON SCORE.C_ID = COURSE.C_ID GROUP BY COURSE.T_ID,SCORE.C_ID ORDER BY  AVG_SCORE DESC;
      
      +--------------+-------------+------------+--+
      | course.t_id  | score.c_id  | avg_score  |
      +--------------+-------------+------------+--+
      | 01           | 02          | 72.67      |
      | 03           | 03          | 68.5       |
      | 02           | 01          | 64.5       |
      +--------------+-------------+------------+--+
      123456789
      ```

22. **查询所有课程的成绩第2名到第3名的学生信息及该课程成绩:**

    - ```sql
      WITH A AS(
        SELECT *,ROW_NUMBER() OVER(PARTITION BY C_ID ORDER BY S_SCORE DESC  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) CN  FROM SCORE
      )
      SELECT * FROM A WHERE CN BETWEEN 2 AND 3;
      
      +---------+---------+------------+-------+--+
      | a.s_id  | a.c_id  | a.s_score  | a.cn  |
      +---------+---------+------------+-------+--+
      | 01      | 01      | 80         | 2     |
      | 05      | 01      | 76         | 3     |
      | 07      | 02      | 89         | 2     |
      | 05      | 02      | 87         | 3     |
      | 07      | 03      | 98         | 2     |
      | 02      | 03      | 80         | 3     |
      +---------+---------+------------+-------+--+
      123456789101112131415
      ```

23. **统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比**

    - ```sql
      SELECT C_ID ,
      SUM((CASE WHEN S_SCORE<=100 AND S_SCORE>85 THEN 1 ELSE 0 END)) AS `[100-85]人数`,
      SUM((CASE WHEN S_SCORE<=85 AND S_SCORE>70 THEN 1 ELSE 0 END)) AS `[85-70]人数`,
      SUM((CASE WHEN S_SCORE<=70 AND S_SCORE>60 THEN 1 ELSE 0 END)) AS `[70-60]人数`,
      SUM((CASE WHEN S_SCORE<=60 AND S_SCORE>0 THEN 1 ELSE 0 END)) AS `[0-60]人数`,
      ROUND(SUM((CASE WHEN S_SCORE<=100 AND S_SCORE>85 THEN 1 ELSE 0 END))/COUNT(*),2) AS `[100-85]`,
      ROUND(SUM((CASE WHEN S_SCORE<=85 AND S_SCORE>70 THEN 1 ELSE 0 END))/COUNT(*),2) AS `[85-70]`,
      ROUND(SUM((CASE WHEN S_SCORE<=70 AND S_SCORE>60 THEN 1 ELSE 0 END))/COUNT(*),2) AS `[70-60]`,
      ROUND(SUM((CASE WHEN S_SCORE<=60 AND S_SCORE>0 THEN 1 ELSE 0 END))/COUNT(*),2) AS `[0-60]`
      FROM SCORE GROUP BY C_ID  
      
      +-----+------------+----------+-----------+----------+--------+-------+-------+------+--+
      | c_id|[100-85]人数|[85-70]人数|[70-60]人数|[0-60]人数|[100-85]|[85-70]|[70-60]|[0-60]|
      +-----+----- ------+----------+-----------+----------+--------+- -----+-------+------+--+
      | 01  |     0      |    3     |     1     |    2     |  0.0   | 0.5   | 0.17  | 0.33 |
      | 02  |     3      |    1     |     0     |    2     |  0.5   | 0.17  | 0.0   | 0.33 |
      | 03  |     2      |    2     |     0     |    2     | 0.33   | 0.33  | 0.0   | 0.33 |
      +-----+------------+----------+-----------+----------+--------+-------+-------+------+--+
      123456789101112131415161718
      ```

24. **查询学生平均成绩及其名次:**

    - ```sql
      WITH T AS(
          SELECT  S_ID ,AVG(S_SCORE) AS AVG_SCORE FROM SCORE GROUP BY S_ID
      ),
      T3 AS(
      SELECT STU.*,ROUND(CAST(T.AVG_SCORE AS DOUBLE),2) AVG_SCORE FROM STUDENT STU RIGHT JOIN T ON STU.S_ID = T.S_ID
      )
      SELECT *,ROW_NUMBER() OVER(ORDER BY AVG_SCORE DESC) AS RANK FROM T3;
      
      +----------+------------+-------------+-----------+---------------+-------+--+
      | t3.s_id  | t3.s_name  | t3.s_birth  | t3.s_sex  | t3.avg_score  | rank  |
      +----------+------------+-------------+-----------+---------------+-------+--+
      | 07       | 郑竹         | 1989-07-01  | 女         | 93.5          | 1     |
      | 01       | 赵雷         | 1990-01-01  | 男         | 89.67         | 2     |
      | 05       | 周梅         | 1991-12-01  | 女         | 81.5          | 3     |
      | 03       | 孙风         | 1990-05-20  | 男         | 80.0          | 4     |
      | 02       | 钱电         | 1990-12-21  | 男         | 70.0          | 5     |
      | 04       | 李云         | 1990-08-06  | 男         | 33.33         | 6     |
      | 06       | 吴兰         | 1992-03-01  | 女         | 32.5          | 7     |
      +----------+------------+-------------+-----------+---------------+-------+--+
      12345678910111213141516171819
      ```

25. **查询各科成绩前三名的记录**

    - ```sql
      WITH T AS(
        SELECT *,ROW_NUMBER() OVER (PARTITION BY C_ID ORDER BY S_SCORE DESC) AS CN FROM SCORE
      )
      SELECT * FROM T WHERE T.CN = 1;    
      
      +---------+---------+------------+-------+--+
      | t.s_id  | t.c_id  | t.s_score  | t.cn  |
      +---------+---------+------------+-------+--+
      | 03      | 01      | 80         | 1     |
      | 01      | 02      | 90         | 1     |
      | 01      | 03      | 99         | 1     |
      +---------+---------+------------+-------+--+
      123456789101112
      ```

26. **查询每门课程被选修的学生数:**

    - ```sql
      SELECT C_ID ,COUNT(S_ID) AS NUM FROM SCORE GROUP BY C_ID ;
      
      +-------+------+--+
      | c_id  | num  |
      +-------+------+--+
      | 01    | 6    |
      | 02    | 6    |
      | 03    | 6    |
      +-------+------+--+
      123456789
      ```

27. **查询出只有两门课程的全部学生的学号和姓名:**

    - ```sql
      WITH A AS (
       SELECT S_ID ,COUNT(C_ID) AS NUM  FROM SCORE GROUP BY S_ID HAVING NUM = 2
      )
      SELECT A.S_ID,STUDENT.S_NAME FROM A LEFT JOIN STUDENT ON STUDENT.S_ID  = A.S_ID
      
      +---------+-----------------+--+
      | a.s_id  | student.s_name  |
      +---------+-----------------+--+
      | 05      | 周梅              |
      | 06      | 吴兰              |
      | 07      | 郑竹              |
      +---------+-----------------+--+
      123456789101112
      ```

28. **查询男生、女生人数:**

    - ```sql
      SELECT SIZE(COLLECT_SET(S_ID)) COUNT_NUM,S_SEX FROM STUDENT GROUP BY S_SEX;
      
      +------------+--------+--+
      | count_num  | s_sex  |
      +------------+--------+--+
      | 4          | 女      |
      | 4          | 男      |
      +------------+--------+--+
      12345678
      ```

29. **查询名字中含有"风"字的学生信息:**

    - ```sql
      SELECT * FROM STUDENT WHERE S_NAME LIKE '%风%'
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 03            | 孙风              | 1990-05-20       | 男              |
      +---------------+-----------------+------------------+----------------+--+
      1234567
      ```

30. **查询同名同性学生名单，并统计同名人数:**

    - ```sql
      WITH A AS (
        SELECT *,COUNT(S_NAME) OVER (PARTITION BY S_NAME,S_SEX ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) COUNT_S_NAME FROM STUDENT
      )
      SELECT * FROM A WHERE COUNT_S_NAME>1
      
      +---------+-----------+------------+----------+-----------------+--+
      | a.s_id  | a.s_name  | a.s_birth  | a.s_sex  | a.count_s_name  |
      +---------+-----------+------------+----------+-----------------+--+
      +---------+-----------+------------+----------+-----------------+--+
      123456789
      ```

31. **查询1990年出生的学生名单:**

    - ```sql
      SELECT * FROM STUDENT WHERE YEAR(S_BIRTH) = '1990'; 
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 01            | 赵雷              | 1990-01-01       | 男              |
      | 02            | 钱电              | 1990-12-21       | 男              |
      | 03            | 孙风              | 1990-05-20       | 男              |
      | 04            | 李云              | 1990-08-06       | 男              |
      | 08            | 王菊              | 1990-01-20       | 女              |
      +---------------+-----------------+------------------+----------------+--+
      1234567891011
      ```

32. **查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列:**

    - ```sql
      SELECT C_ID,ROUND(AVG(S_SCORE),2) AS AVG_SCORE FROM SCORE GROUP BY C_ID ORDER BY AVG_SCORE DESC ,C_ID ASC 
      
      +-------+------------+--+
      | c_id  | avg_score  |
      +-------+------------+--+
      | 02    | 72.67      |
      | 03    | 68.5       |
      | 01    | 64.5       |
      +-------+------------+--+
      123456789
      ```

33. **查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩:**

    - ```sql
      WITH T AS(
        SELECT AVG(S_SCORE) AS AVG_SCORE,S_ID  FROM SCORE GROUP BY S_ID HAVING AVG_SCORE>=85
      )
      SELECT STU.*,ROUND(AVG_SCORE,2) AS AVG_SCORE FROM T LEFT JOIN STUDENT STU ON T.S_ID = STU.S_ID;
      
      +-----------+-------------+--------------+------------+------------+--+
      | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  | avg_score  |
      +-----------+-------------+--------------+------------+------------+--+
      | 01        | 赵雷          | 1990-01-01   | 男          | 89.67      |
      | 07        | 郑竹          | 1989-07-01   | 女          | 93.5       |
      +-----------+-------------+--------------+------------+------------+--+
      1234567891011
      ```

34. **查询课程名称为"数学"，且分数低于60的学生姓名和分数:**

    - ```sql
      WITH T AS (
        SELECT S.S_ID ,S.S_SCORE FROM SCORE S LEFT JOIN COURSE C ON C.C_ID = S.C_ID WHERE S.S_SCORE < 60 AND C.C_NAME ='数学'
      )
      SELECT S.S_NAME,T.S_SCORE FROM T LEFT JOIN STUDENT S ON S.S_ID = T.S_ID
      
      +-----------+------------+--+
      | s.s_name  | t.s_score  |
      +-----------+------------+--+
      | 李云        | 30         |
      +-----------+------------+--+
      12345678910
      ```

35. **查询所有学生的课程及分数情况:**

    - ```sql
      SELECT S.*,CO.C_NAME,C.S_SCORE FROM STUDENT S LEFT JOIN SCORE C ON S.S_ID = C.S_ID LEFT JOIN COURSE CO ON CO.C_ID = C.C_ID 
      
      +---------+-----------+-------------+----------+------------+------------+--+
      | s.s_id  | s.s_name  |  s.s_birth  | s.s_sex  | co.c_name  | c.s_score  |
      +---------+-----------+-------------+----------+------------+------------+--+
      | 01      | 赵雷        | 1990-01-01  | 男        | 语文         | 80         |
      | 01      | 赵雷        | 1990-01-01  | 男        | 数学         | 90         |
      | 01      | 赵雷        | 1990-01-01  | 男        | 英语         | 99         |
      | 02      | 钱电        | 1990-12-21  | 男        | 语文         | 70         |
      | 02      | 钱电        | 1990-12-21  | 男        | 数学         | 60         |
      | 02      | 钱电        | 1990-12-21  | 男        | 英语         | 80         |
      | 03      | 孙风        | 1990-05-20  | 男        | 语文         | 80         |
      | 03      | 孙风        | 1990-05-20  | 男        | 数学         | 80         |
      | 03      | 孙风        | 1990-05-20  | 男        | 英语         | 80         |
      | 04      | 李云        | 1990-08-06  | 男        | 语文         | 50         |
      | 04      | 李云        | 1990-08-06  | 男        | 数学         | 30         |
      | 04      | 李云        | 1990-08-06  | 男        | 英语         | 20         |
      | 05      | 周梅        | 1991-12-01  | 女        | 语文         | 76         |
      | 05      | 周梅        | 1991-12-01  | 女        | 数学         | 87         |
      | 06      | 吴兰        | 1992-03-01  | 女        | 语文         | 31         |
      | 06      | 吴兰        | 1992-03-01  | 女        | 英语         | 34         |
      | 07      | 郑竹        | 1989-07-01  | 女        | 数学         | 89         |
      | 07      | 郑竹        | 1989-07-01  | 女        | 英语         | 98         |
      | 08      | 王菊        | 1990-01-20  | 女        | NULL       | NULL       |
      +---------+-----------+-------------+----------+------------+------------+--+
      12345678910111213141516171819202122232425
      ```

36. **查询任何一门课程成绩在70分以上的学生姓名、课程名称和分数:**

    - ```sql
      WITH T AS (
        SELECT STU.*,S.S_SCORE,S.C_ID FROM SCORE  S LEFT JOIN STUDENT STU ON STU.S_ID =S.S_ID  WHERE S.S_SCORE >= 70
      )
      SELECT T.*,C.C_NAME FROM T LEFT JOIN COURSE C ON C.C_ID = T.C_ID ;
      
      +---------+-----------+-------------+----------+------------+---------+-----------+--+
      | t.s_id  | t.s_name  |  t.s_birth  | t.s_sex  | t.s_score  | t.c_id  | c.c_name  |
      +---------+-----------+-------------+----------+------------+---------+-----------+--+
      | 01      | 赵雷        | 1990-01-01  | 男        | 80         | 01      | 语文        |
      | 01      | 赵雷        | 1990-01-01  | 男        | 90         | 02      | 数学        |
      | 01      | 赵雷        | 1990-01-01  | 男        | 99         | 03      | 英语        |
      | 02      | 钱电        | 1990-12-21  | 男        | 70         | 01      | 语文        |
      | 02      | 钱电        | 1990-12-21  | 男        | 80         | 03      | 英语        |
      | 03      | 孙风        | 1990-05-20  | 男        | 80         | 01      | 语文        |
      | 03      | 孙风        | 1990-05-20  | 男        | 80         | 02      | 数学        |
      | 03      | 孙风        | 1990-05-20  | 男        | 80         | 03      | 英语        |
      | 05      | 周梅        | 1991-12-01  | 女        | 76         | 01      | 语文        |
      | 05      | 周梅        | 1991-12-01  | 女        | 87         | 02      | 数学        |
      | 07      | 郑竹        | 1989-07-01  | 女        | 89         | 02      | 数学        |
      | 07      | 郑竹        | 1989-07-01  | 女        | 98         | 03      | 英语        |
      +---------+-----------+-------------+----------+------------+---------+-----------+--+
      123456789101112131415161718192021
      ```

37. **查询课程不及格的学生:**

    - ```sql
      WITH T AS (
        SELECT STU.*,S.S_SCORE,S.C_ID FROM SCORE  S LEFT JOIN STUDENT STU ON STU.S_ID =S.S_ID  WHERE S.S_SCORE < 60
      )
      SELECT T.*,C.C_NAME FROM T LEFT JOIN COURSE C ON C.C_ID = T.C_ID ;
      
      +---------+-----------+-------------+----------+------------+---------+-----------+--+
      | t.s_id  | t.s_name  |  t.s_birth  | t.s_sex  | t.s_score  | t.c_id  | c.c_name  |
      +---------+-----------+-------------+----------+------------+---------+-----------+--+
      | 04      | 李云        | 1990-08-06  | 男        | 50         | 01      | 语文        |
      | 04      | 李云        | 1990-08-06  | 男        | 30         | 02      | 数学        |
      | 04      | 李云        | 1990-08-06  | 男        | 20         | 03      | 英语        |
      | 06      | 吴兰        | 1992-03-01  | 女        | 31         | 01      | 语文        |
      | 06      | 吴兰        | 1992-03-01  | 女        | 34         | 03      | 英语        |
      +---------+-----------+-------------+----------+------------+---------+-----------+--+
      1234567891011121314
      ```

38. **查询课程编号为01且课程成绩在80分以上的学生的学号和姓名:**

    - ```sql
      WITH T AS(
        SELECT S_ID FROM SCORE WHERE C_ID ='01' AND S_SCORE >= 80
      )
      SELECT T.S_ID,STU.S_NAME FROM T LEFT JOIN STUDENT STU ON STU.S_ID = T.S_ID 
      
      +---------+-------------+--+
      | t.s_id  | stu.s_name  |
      +---------+-------------+--+
      | 01      | 赵雷          |
      | 03      | 孙风          |
      +---------+-------------+--+
      1234567891011
      ```

39. **求每门课程的学生人数:**

    - ```sql
      WITH T AS (
        SELECT SIZE(COLLECT_SET(S_ID)) AS CN,C_ID FROM SCORE GROUP BY C_ID 
      )
      SELECT C.C_ID,C.C_NAME,T.CN FROM T LEFT JOIN COURSE C ON C.C_ID = T.C_ID;
      
      +---------+-----------+-------+--+
      | c.c_id  | c.c_name  | t.cn  |
      +---------+-----------+-------+--+
      | 01      | 语文        | 6     |
      | 02      | 数学        | 6     |
      | 03      | 英语        | 6     |
      +---------+-----------+-------+--+
      123456789101112
      ```

40. **查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩:**

    - ```sql
      WITH T AS (
        SELECT C.C_ID FROM TEACHER T LEFT JOIN COURSE C ON C.T_ID = T.T_ID WHERE T.T_NAME = '张三'
      ),
       T1 AS (
        SELECT S.S_ID,S.S_SCORE FROM T LEFT JOIN SCORE S ON T.C_ID = S.C_ID ORDER BY S_SCORE DESC LIMIT 1 
       )
      SELECT STU.*,T1.S_SCORE FROM T1 LEFT JOIN STUDENT STU ON STU.S_ID = T1.S_ID ;
      
      +-----------+-------------+--------------+------------+-------------+--+
      | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  | t1.s_score  |
      +-----------+-------------+--------------+------------+-------------+--+
      | 01        | 赵雷          | 1990-01-01   | 男          | 90          |
      +-----------+-------------+--------------+------------+-------------+--+
      12345678910111213
      ```

41. **查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩:**

    - ```sql
      WITH A AS(
       SELECT S_ID,C_ID,S_SCORE,ROW_NUMBER() OVER (PARTITION BY S_SCORE) CN FROM SCORE
      )
      SELECT DISTINCT B.S_ID,B.C_ID,B.S_SCORE FROM A JOIN SCORE B ON A.S_SCORE = B.S_SCORE WHERE A.CN>1
      
      +---------+---------+------------+--+
      | b.s_id  | b.c_id  | b.s_score  |
      +---------+---------+------------+--+
      | 01      | 01      | 80         |
      | 02      | 03      | 80         |
      | 03      | 01      | 80         |
      | 03      | 02      | 80         |
      | 03      | 03      | 80         |
      +---------+---------+------------+--+
      1234567891011121314
      ```

42. **查询每门课程成绩最好的前三名:**

    - ```sql
      WITH T AS(
        SELECT *,ROW_NUMBER() OVER(PARTITION BY C_ID ORDER BY S_SCORE DESC) AS CN  FROM SCORE
      ),
      T1 AS (
        SELECT STU.*,T.C_ID FROM T LEFT JOIN STUDENT STU ON T.S_ID = STU.S_ID WHERE T.CN <4
      )
      SELECT T1.*,C.C_NAME FROM T1 LEFT JOIN COURSE C ON T1.C_ID = C.C_ID;
      
      +----------+------------+-------------+-----------+----------+-----------+--+
      | t1.s_id  | t1.s_name  | t1.s_birth  | t1.s_sex  | t1.c_id  | c.c_name  |
      +----------+------------+-------------+-----------+----------+-----------+--+
      | 03       | 孙风         | 1990-05-20  | 男         | 01       | 语文        |
      | 01       | 赵雷         | 1990-01-01  | 男         | 01       | 语文        |
      | 05       | 周梅         | 1991-12-01  | 女         | 01       | 语文        |
      | 01       | 赵雷         | 1990-01-01  | 男         | 02       | 数学        |
      | 07       | 郑竹         | 1989-07-01  | 女         | 02       | 数学        |
      | 05       | 周梅         | 1991-12-01  | 女         | 02       | 数学        |
      | 01       | 赵雷         | 1990-01-01  | 男         | 03       | 英语        |
      | 07       | 郑竹         | 1989-07-01  | 女         | 03       | 英语        |
      | 02       | 钱电         | 1990-12-21  | 男         | 03       | 英语        |
      +----------+------------+-------------+-----------+----------+-----------+--+
      123456789101112131415161718192021
      ```

43. **统计每门课程的学生选修人数（超过5人的课程才统计）:– 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列**

    - ```sql
      WITH T AS (
       SELECT SIZE(COLLECT_SET(S_ID)) AS CN,C_ID FROM SCORE GROUP BY C_ID 
      )
      SELECT * FROM T WHERE CN > 5 ORDER BY CN DESC ,C_ID ASC
      
      +-------+---------+--+
      | t.cn  | t.c_id  |
      +-------+---------+--+
      | 6     | 01      |
      | 6     | 02      |
      | 6     | 03      |
      +-------+---------+--+
      123456789101112
      ```

44. **检索至少选修两门课程的学生学号:**

    - ```sql
      WITH T AS(
       SELECT SIZE(COLLECT_SET(C_ID)) AS CN ,S_ID FROM SCORE GROUP BY S_ID 
      )
      SELECT S_ID,CN FROM T WHERE CN >=2
      
      +-------+-----+--+
      | s_id  | cn  |
      +-------+-----+--+
      | 01    | 3   |
      | 02    | 3   |
      | 03    | 3   |
      | 04    | 3   |
      | 05    | 2   |
      | 06    | 2   |
      | 07    | 2   |
      +-------+-----+--+
      12345678910111213141516
      ```

45. **查询选修了全部课程的学生信息:**

    - ```sql
      --方法1
      WITH T AS(
        SELECT COUNT(C_ID) COUNT FROM COURSE
      ),
      T1 AS(
          SELECT S_ID,COUNT(S_ID) OVER(PARTITION BY S_ID ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS COUNT_NUM FROM SCORE
      ),
      T2 AS(
      SELECT DISTINCT T1.S_ID FROM T1 JOIN T ON T1.COUNT_NUM=T.COUNT
      )
      SELECT * FROM STUDENT STU LEFT SEMI JOIN T2 ON STU.S_ID = T2.S_ID
      -----------------------------------------------------------------------------------------------
      --方法2
      WITH T AS(
      SELECT S_ID,SIZE(COLLECT_SET(C_ID)) AS COUNT_NUM FROM SCORE  GROUP BY S_ID
      ),
      T1 AS (
      SELECT COUNT(DISTINCT C_ID) AS COUNT FROM COURSE
      ),
      T2 AS(
      SELECT STU.*,T.COUNT_NUM FROM STUDENT STU LEFT JOIN T ON STU.S_ID=T.S_ID
      )
      SELECT T2.S_ID,T2.S_NAME,T2.S_BIRTH,T2.S_SEX FROM T2 LEFT SEMI JOIN T1 ON T2.COUNT_NUM = T1.COUNT
      
      +-----------+-------------+--------------+------------+--+
      | stu.s_id  | stu.s_name  | stu.s_birth  | stu.s_sex  |
      +-----------+-------------+--------------+------------+--+
      | 01        | 赵雷          | 1990-01-01   | 男          |
      | 02        | 钱电          | 1990-12-21   | 男          |
      | 03        | 孙风          | 1990-05-20   | 男          |
      | 04        | 李云          | 1990-08-06   | 男          |
      +-----------+-------------+--------------+------------+--+
      1234567891011121314151617181920212223242526272829303132
      ```

46. **查询各学生的年龄(周岁):
    – 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一**

    - ```sql
      SELECT *,(
          YEAR(CURRENT_DATE)-YEAR(S_BIRTH)-(
           CASE 
              WHEN MONTH(CURRENT_DATE)<MONTH(S_BIRTH) THEN 1   
              WHEN MONTH(CURRENT_DATE) = MONTH(S_BIRTH) AND DATE(CURRENT_DATE)<DATE(S_BIRTH) THEN 1 
           ELSE 0 
           END
          )
        ) AS AGE 
      FROM STUDENT; 
      
      +---------------+-----------------+------------------+----------------+------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  | age  |
      +---------------+-----------------+------------------+----------------+------+--+
      | 01            | 赵雷             | 1990-01-01      | 男              | 30   |
      | 02            | 钱电             | 1990-12-21      | 男              | 29   |
      | 03            | 孙风             | 1990-05-20      | 男              | 30   |
      | 04            | 李云             | 1990-08-06      | 男              | 29   |
      | 05            | 周梅             | 1991-12-01      | 女              | 28   |
      | 06            | 吴兰             | 1992-03-01      | 女              | 28   |
      | 07            | 郑竹             | 1989-07-01      | 女              | 30   |
      | 08            | 王菊             | 1990-01-20      | 女              | 30   |
      +---------------+-----------------+------------------+----------------+------+--+
      1234567891011121314151617181920212223
      ```

47. **查询本周过生日的学生:**

    - ```sql
      SELECT * FROM STUDENT WHERE WEEKOFYEAR(S_BIRTH) = WEEKOFYEAR(CURRENT_DATE);
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      +---------------+-----------------+------------------+----------------+--+
      123456
      ```

48. **查询下周过生日的学生:**

    - ```sql
      SELECT * FROM STUDENT WHERE WEEKOFYEAR(S_BIRTH) = WEEKOFYEAR(DATE_ADD(CURRENT_DATE,7));
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      +---------------+-----------------+------------------+----------------+--+
      123456
      ```

49. **查询本月过生日的学生:**

    - ```sql
      SELECT * FROM STUDENT WHERE MONTH(S_BIRTH)=MONTH(CURRENT_DATE);
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 03            | 孙风              | 1990-05-20       | 男              |
      +---------------+-----------------+------------------+----------------+--+
      1234567
      ```

50. **查询12月份过生日的学生:**

    - ```sql
      SELECT * FROM STUDENT WHERE MONTH(S_BIRTH)='12';
      
      +---------------+-----------------+------------------+----------------+--+
      | student.s_id  | student.s_name  | student.s_birth  | student.s_sex  |
      +---------------+-----------------+------------------+----------------+--+
      | 02            | 钱电              | 1990-12-21       | 男              |
      | 05            | 周梅              | 1991-12-01       | 女              |
      +---------------+-----------------+------------------+----------------+--+
      12345678
      ```

      **有更好的sql解题思路请留言**