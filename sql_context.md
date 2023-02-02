#四张表

学生表 课程表 成绩表 老师表

Student(Sid,Sname,Sage,Ssex) --Sid 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

Course(Cid,Cname,Tid) --Cid --课程编号,Cname 课程名称,Tid 教师编号

SC(Sid,Cid,score) --Sid 学生编号,Cid 课程编号,score 分数

Teacher(Tid,Tname) --Tid 教师编号,Tname 教师姓名


# 1. 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
SELECT * FROM
    (SELECT `score` AS sno1, `cid`AS cno1, score FROM sc WHERE `cid`=01) a
        LEFT JOIN
    (SELECT `score` AS sno2, `cid`AS cno2, score FROM sc WHERE `cid`=02) b
        ON a.sno1 = b.sno2
        WHERE a.score > b.score

# 1.1 查询同时存在" 01 "课程和" 02 "课程的情况
SELECT * FROM
    (SELECT `score` AS sno1, `cid`AS cno1, score FROM sc WHERE `cid`=01) a
        LEFT JOIN
    (SELECT `score` AS sno2, `cid`AS cno2, score FROM sc WHERE `cid`=02) b
        ON a.sno1 = b.sno2
        WHERE sno2 IS NOT NULL
        
# 1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
SELECT * FROM
    (SELECT `score` AS sno1, `cid`AS cno1, score FROM sc WHERE `cid`=01) a
        LEFT JOIN
    (SELECT `score` AS sno2, `cid`AS cno2, score FROM sc WHERE `cid`=02) b
        ON a.sno1 = b.sno2

# 1.3 查询不存在" 01 "课程但存在" 02 "课程的情况
SELECT * FROM 
    sc WHERE `cid`='02' AND `score` NOT IN (SELECT `score` FROM sc WHERE `cid`='01')
        
# 2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
SELECT a.`score`,b.`sname`, a.avg_score FROM 
    (SELECT `score` ,AVG(score) AS avg_score FROM sc GROUP BY `score`) AS a
        LEFT JOIN student AS b
        ON a.`sscore` = b.`score`
        WHERE a.avg_score >=60

# 3. 查询在 SC 表存在成绩的学生信息
SELECT * FROM student WHERE `score` IN (SELECT DISTINCT `score` FROM sc)

# 4. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 NULL )
SELECT `score` ,sname , course_num , score_sum FROM 
    (SELECT `score`, sname FROM student ) AS a
        LEFT JOIN
    (SELECT `score` AS sno ,COUNT(`cid`) AS course_num ,SUM(score) AS score_sum  FROM sc GROUP BY sno) AS b
        ON a.`score` = b.sno
        
# 4.1 查有成绩的学生信息
# 在最外面一层select的时候，不可以用函数
# 如果两张表连接之后，有相同的字段，这时候select就需要把其中一个字段改名
SELECT `score` ,sname , course_num , score_sum FROM 
    (SELECT `score`, sname FROM student ) AS a
        LEFT JOIN
    (SELECT `score` AS sno ,COUNT(`cid`) AS course_num ,SUM(score) AS score_sum  FROM sc GROUP BY sno) AS b
        ON a.`score` = b.sno
        WHERE course_num IS NOT NULL

# 5. 查询「李」姓老师的数量 
SELECT COUNT(*) FROM teacher WHERE tname LIKE '李%'

# 6. 查询学过「张三」老师授课的同学的信息
# 张三老师是01号
SELECT * FROM student WHERE `score` IN 
    (SELECT `score` FROM sc WHERE `cid` =
        (SELECT `cid` FROM course WHERE `tid` = 
            (SELECT `tid` FROM teacher WHERE tname='张三')))

# 7. 查询没有学全所有课程的同学的信息 
SELECT `score`,COUNT(`cid`) AS course_num FROM sc GROUP BY `score` 
    HAVING course_num < (SELECT COUNT(*) FROM course)
    
# 8. 查询至少有一门课与学号为"01"的同学所学相同的同学的信息
SELECT * FROM student WHERE `score` IN 
    (SELECT DISTINCT `score` FROM sc WHERE `cid` IN
        (SELECT `cid` FROM sc WHERE `score`=01))
    AND `score`!= 01

# 9. 查询和"01"号的同学学习的课程完全相同的其他同学的信息
SELECT `score` FROM 
    (SELECT * FROM sc 
        LEFT JOIN 
    (SELECT `cid` AS cno FROM sc WHERE `score` =01) a
        ON sc.`cid` = a.cno) AS b
GROUP BY `score`        
HAVING COUNT(b.`score`) = (SELECT COUNT(`cid`) AS cno FROM sc WHERE `score` =01)

# 10. 查询没学过"张三"老师讲授的任一门课程的学生姓名
# 张三是01
# 01老师是教数学，c#是02
SELECT * FROM student WHERE `score` NOT IN 
    (SELECT DISTINCT `score` FROM sc WHERE `cid` IN 
        (SELECT `cid` FROM course WHERE `tid` IN 
            (SELECT `tid` FROM teacher WHERE tname = '张三')))

# 11. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
SELECT `score`, sname, avg_score FROM 
    (SELECT `score`, sname FROM student WHERE `score` IN
        (SELECT a.`score` FROM 
            (SELECT `score`,COUNT(`cid`) AS num FROM sc WHERE score <60 GROUP BY `score`) a
            WHERE num >=2)) AS b
        LEFT JOIN
    (SELECT `score` AS sno ,AVG(score) AS avg_score FROM sc GROUP BY `score`) AS c
        ON b.`score` = c.sno

# 12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
SELECT `score`, sname, score FROM 
     student AS a
        LEFT JOIN 
    (SELECT `score` AS sno,`cid`,score FROM sc WHERE `cid`= 01 AND score <60 )b
        ON a.`score`= b.sno
    WHERE score IS NOT NULL
    ORDER BY score DESC

# 13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
SELECT `score` ,AVG(score) AS avg_score FROM sc GROUP BY `score` ORDER BY avg_score DESC

# 14. 查询各科成绩最高分、最低分和平均分：
# 以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
# 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
# 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
SELECT DISTINCT a.`cid`,cname,最高分,最低分,平均分,及格率,中等率,优良率,优秀率 FROM sc a
LEFT JOIN course ON a.`cid`=course.`cid`
LEFT JOIN (SELECT `cid`, MAX(score)最高分, MIN(score)最低分, AVG(score)平均分 FROM sc GROUP BY `cid`)b ON a.`cid`=b.`cid`
LEFT JOIN (SELECT `cid`, ROUND( r1 /cnt * 100, 2 ) AS 及格率 FROM
    (SELECT `cid`, (SUM(CASE WHEN score >=60 THEN 1 ELSE 0 END)*1.00) AS r1 , COUNT(*) AS cnt FROM sc GROUP BY `cid`) c1) c ON a.`cid`=c.`cid`
LEFT JOIN (SELECT `cid`, ROUND( r2 /cnt * 100, 2 ) AS 中等率 FROM
    (SELECT `cid`, (SUM(CASE WHEN score >=70 AND score<80 THEN 1 ELSE 0 END)*1.00) AS r2 , COUNT(*) AS cnt FROM sc GROUP BY `cid`) d1) d ON a.`cid`=d.`cid`    
LEFT JOIN (SELECT `cid`, ROUND( r3 /cnt * 100, 2 ) AS 优良率 FROM
    (SELECT `cid`, (SUM(CASE WHEN score >=80 AND score<90 THEN 1 ELSE 0 END)*1.00) AS r3 , COUNT(*) AS cnt FROM sc GROUP BY `cid`) e1) e ON a.`cid`=e.`cid`
LEFT JOIN (SELECT `cid`, ROUND( r4 /cnt * 100, 2 ) AS 优秀率 FROM
    (SELECT `cid`, (SUM(CASE WHEN score >=90 THEN 1 ELSE 0 END)*1.00) AS r4 , COUNT(*) AS cnt FROM sc GROUP BY `cid`) f1) f ON a.`cid`=f.`cid`

# 15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
# mysql中没有rank()函数
# 这种是重复时候保留名次，所以最后名次和人数是一样的
SELECT `score`, `cid`, score, rank FROM
(SELECT `score`, `cid`, score,
@currank := IF(@prevrank = score, @currank, @incrank) AS rank, 
@incrank := @incrank + 1, 
@prevrank := score
FROM sc , (
SELECT @currank :=0, @prevrank := NULL, @incrank := 1
) r 
ORDER BY score DESC) s

# 15.1 按各科成绩进行排序，并显示排名， Score 重复时合并名次
# 这种是当有重复名次的时候变成只有一个名次，所以排名的数量会变少
SELECT `score`, `cid`, score, 
CASE 
WHEN @prevrank = score THEN @currank 
WHEN @prevrank := score THEN @currank := @currank + 1
END AS rank
FROM sc, 
(SELECT @currank :=0, @prevrank := NULL) r
ORDER BY score DESC

# 16.  查询学生的总成绩，并进行排名，总分重复时保留名次空缺
# from后面不需要加表的别名
SELECT `score`, sum_score, rank FROM
(SELECT `score`, sum_score,
@currank := IF(@prevrank = sum_score, @currank, @incrank) AS rank, 
@incrank := @incrank + 1, 
@prevrank := sum_score
FROM 
(SELECT `score`, SUM(score) AS sum_score FROM sc GROUP BY `score`) c , 
(SELECT @currank :=0, @prevrank := NULL, @incrank := 1) r 
ORDER BY sum_score DESC) s

# 16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
SELECT c.*,
CASE 
WHEN @prevrank = c.sum_score THEN @currank 
WHEN @prevrank := c.sum_score THEN @currank := @currank + 1
END AS rank
FROM 
(SELECT a.`score`,a.sname,SUM(score) AS sum_score
FROM (student AS a  RIGHT JOIN sc AS b ON a.`score` = b.`score`) 
GROUP BY a.`score` ) c , 
(SELECT @currank := 0 , @prevrank :=NULL ) d 
ORDER BY sum_score DESC

# 17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
SELECT a.`cid` , b.cname, 
    SUM(CASE WHEN score >=85 AND score <=100 THEN 1 ELSE 0 END ) '[100-85]',
    SUM(CASE WHEN score >=85 AND score <=100 THEN 1 ELSE 0 END )*1.00/COUNT(*) AS '[100-85]percent',
        SUM(CASE WHEN score < 85 AND score >= 70 THEN 1 ELSE 0 END ) '(85-70]',
        SUM(CASE WHEN score < 85 AND score >= 70 THEN 1 ELSE 0 END )*1.00/COUNT(*) AS '(85-70]percent',
        SUM(CASE WHEN score < 70 AND score >= 60 THEN 1 ELSE 0 END ) '(70-60]',
        SUM(CASE WHEN score < 70 AND score >= 60 THEN 1 ELSE 0 END )*1.00/COUNT(*) AS '(85-70]percent',
        SUM(CASE WHEN score < 60 AND score >= 0 THEN 1 ELSE 0 END ) '(60-0]',
        SUM(CASE WHEN score < 60 AND score >= 0 THEN 1 ELSE 0 END )*1.00/COUNT(*) AS '(85-70]percent',
        COUNT(*) AS counts
FROM sc a LEFT JOIN course b ON a.`cid` = b.`cid`
GROUP BY `cid`

# 18. 查询各科成绩前三名的记录
SELECT * FROM sc a WHERE 
    (SELECT COUNT(*) FROM sc WHERE `cid`=a.`cid` AND score>a.score)<3  
    ORDER BY a.`cid`, a.score DESC;

# 19. 查询每门课程被选修的学生数
SELECT `cid`, COUNT(`score`) FROM 
(SELECT `score`,`cid` FROM sc ORDER BY `cid`)a
GROUP BY `cid` 

SELECT   a.`cid` , b.cname ,COUNT(*) AS num  FROM sc a LEFT JOIN course b ON a.`cid` = b.`cid`
GROUP BY a.`cid`;

# 20. 查询出只选修两门课程的学生学号和姓名 
SELECT a.`score`, a.sname ,cnt FROM 
student a
LEFT JOIN 
(SELECT `score`,COUNT(`cid`) AS cnt FROM sc GROUP BY `score`) b
ON a.`score`=b.`score`
WHERE cnt=2

# 21. 查询男生、女生人数
SELECT ssex,COUNT(ssex) FROM student GROUP BY ssex

# 22. 查询名字中含有「风」字的学生信息
SELECT * FROM student WHERE sname LIKE '%风%'

# 23. 查询同名同性学生名单，并统计同名人数
SELECT a.*,b.同名人数 FROM student a
LEFT JOIN (SELECT sname,ssex,COUNT(*) AS 同名人数 FROM student GROUP BY sname,ssex)b 
ON a.sname=b.sname AND a.ssex=b.ssex
WHERE b.同名人数>1

# 24. 查询 1990 年出生的学生名单
SELECT * FROM student WHERE YEAR(sage) = 1990

# 25. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
SELECT `cid`, ROUND(AVG(score),2) AS avg_score FROM sc GROUP BY `cid` ORDER BY `cid` ASC

# 26. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
SELECT c.`score`,sname ,avg_score FROM
(student c 
LEFT JOIN
(SELECT `score`, avg_score FROM 
(SELECT `score` ,ROUND(AVG(score),2) AS avg_score FROM sc 
GROUP BY `score` ORDER BY avg_score DESC)a 
WHERE avg_score >=85) b
ON c.`score` =b.`score`)
WHERE avg_score IS NOT NULL

# 27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
SELECT a.`score`,a.sname,b.math, b.score FROM
student a
LEFT JOIN
(SELECT `score`,`cid` AS math ,score FROM sc WHERE `cid` IN 
    (SELECT `cid`  FROM course WHERE cname = '数学')
    AND sc.score <60) b
ON a.`score`=b.`score`
WHERE b.score IS NOT NULL 

# 28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
SELECT a.`score`,a.`sname`,a.`sage`,a.`ssex`,b.`cid`,b.score FROM 
student a LEFT JOIN sc b ON a.`score` = b.`score`
LEFT JOIN course c ON c.`cid` = b.`cid`

# 29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
SELECT a.`score`,a.`sname`,a.`sage`,a.`ssex`,b.`cid`,b.score FROM 
student a 
LEFT JOIN 
(SELECT `score`,`cid`,score FROM sc WHERE score >70) b ON a.`score`=b.`score`
LEFT JOIN course c 
ON c.`cid`=b.`cid`
WHERE score IS NOT NULL

# 30. 查询不及格的课程
SELECT * FROM sc WHERE score < 60

# 31. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
SELECT a.`score`, a.sname ,b.score FROM 
    student a
        LEFT JOIN
    (SELECT * FROM sc WHERE `cid`='01' AND score >= 80) b
        ON a.`score` = b.`score`
    WHERE score IS NOT NULL

# 32. 求每门课程的学生人数
SELECT `cid`,COUNT(`cid`) FROM sc GROUP BY `cid`

# 33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
SELECT a.`score`, a.`sname` ,b.`cid`, b.max_score FROM
student a
LEFT JOIN
(SELECT `score` AS sid,`cid` ,MAX(score) AS max_score FROM sc WHERE `cid` IN 
    (SELECT `cid` FROM course WHERE `tid` IN 
        (SELECT `tid` FROM teacher WHERE tname = '张三'))) b
ON a.`score`=b.sid
WHERE max_score IS NOT NULL

# 34. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
SELECT * FROM
(SELECT dd.*,
CASE 
WHEN @prevrank = dd.score THEN @currank 
WHEN @prevrank := dd.score THEN @currank := @currank + 1
END AS rank
 FROM (SELECT a.*,b.score FROM
student a 
LEFT JOIN sc b ON a.`score` = b.`score`
LEFT JOIN course c ON b.`cid` = c.`cid`
LEFT JOIN teacher d ON c.`tid` = d.`tid` WHERE d.tname = '张三' ) dd,(SELECT @currank := 0 , @prevrank :=NULL ) ff 
ORDER BY score DESC) AS dddddddd
WHERE rank = 1;

# 35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 
SELECT DISTINCT a.`score`, a.`cid`, a.score FROM sc AS a JOIN sc AS b 
WHERE a.`cid` != b.`cid` AND a.score = b.score AND a.`score` != b.`score`
ORDER BY a.`score`, a.`cid`, a.score

# 36. 查询每门功课成绩最好的前两名
# 此题和18题相同
SELECT * FROM sc a WHERE 
    (SELECT COUNT(*) FROM sc WHERE `cid`=a.`cid` AND score>a.score)<2  
    ORDER BY a.`cid`, a.score DESC;

# 37. 统计每门课程的学生选修人数（超过 5 人的课程才统计）
# 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列 
SELECT a.`cid`, COUNT(*) AS num FROM 
course a LEFT JOIN sc b ON a.`cid` = b.`cid`
GROUP BY a.`cid` HAVING num > 5
ORDER BY num,a.`cid`

# 38. 检索至少选修两门课程的学生学号
SELECT DISTINCT`score`,COUNT(`cid`) AS num FROM sc GROUP BY `score` HAVING num >=2

# 39. 查询选修了全部课程的学生信息
SELECT * FROM 
    (SELECT `score`,COUNT(*) AS num FROM sc GROUP BY `score` ) b
    WHERE num = (SELECT COUNT(*) FROM course)

# 40. 查询各学生的年龄，只按年份来算
SELECT *, YEAR(NOW()) - YEAR(sage) AS age FROM student

# 41. 查询本周过生日的学生
SELECT * FROM
(SELECT * , WEEK(sage), MONTH(sage),DAY(sage),
WEEK(STR_TO_DATE(CONCAT_WS(',',YEAR(NOW()),MONTH(sage),DAY(sage)),'%y,%m,%d')) AS w  FROM student) a
WHERE w = WEEK(NOW())

# 42. 查询下周过生日的学生
SELECT * FROM
(SELECT * , WEEK(sage), MONTH(sage),DAY(sage),WEEK(NOW()),
WEEK(STR_TO_DATE(CONCAT_WS(',',YEAR(NOW()),MONTH(sage),DAY(sage)),'%y,%m,%d')) AS w  FROM student) a
WHERE w + 2 = WEEK(NOW())

# 43. 查询本月过生日的学生
SELECT * , MONTH(sage),MONTH(NOW()) FROM student
WHERE MONTH(sage) = MONTH(NOW())

# 44. 查询下月过生日的学生        
SELECT * , MONTH(sage),MONTH(NOW()) FROM student
WHERE MONTH(sage) = MONTH(NOW()) + 1