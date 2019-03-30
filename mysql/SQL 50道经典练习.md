# SQL 50道经典练习题

参考：

https://zhuanlan.zhihu.com/p/32137597

https://www.jianshu.com/p/476b52ee4f1b



```
create database study charset gbk;
use study;

create table student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into student values('01' , '赵雷' , '1990-01-01' , '男');
insert into student values('02' , '钱电' , '1990-12-21' , '男');
insert into student values('03' , '孙风' , '1990-05-20' , '男');
insert into student values('04' , '李云' , '1990-08-06' , '男');
insert into student values('05' , '周梅' , '1991-12-01' , '女');
insert into student values('06' , '吴兰' , '1992-03-01' , '女');
insert into student values('07' , '郑竹' , '1989-07-01' , '女');
insert into student values('09' , '张三' , '2017-12-20' , '女');
insert into student values('10' , '李四' , '2017-12-25' , '女');
insert into student values('11' , '李四' , '2017-12-30' , '女');
insert into student values('12' , '赵六' , '2017-01-01' , '女');
insert into student values('13' , '孙七' , '2018-01-01' , '女');

create table course(CId varchar(10),Cname nvarchar(10),TId varchar(10));
insert into course values('01' , '语文' , '02');
insert into course values('02' , '数学' , '01');
insert into course values('03' , '英语' , '03');

create table teacher(TId varchar(10),Tname varchar(10));
insert into teacher values('01' , '张三');
insert into teacher values('02' , '李四');
insert into teacher values('03' , '王五');

create table sc(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into sc values('01' , '01' , 80);
insert into sc values('01' , '02' , 90);
insert into sc values('01' , '03' , 99);
insert into sc values('02' , '01' , 70);
insert into sc values('02' , '02' , 60);
insert into sc values('02' , '03' , 80);
insert into sc values('03' , '01' , 80);
insert into sc values('03' , '02' , 80);
insert into sc values('03' , '03' , 80);
insert into sc values('04' , '01' , 50);
insert into sc values('04' , '02' , 30);
insert into sc values('04' , '03' , 20);
insert into sc values('05' , '01' , 76);
insert into sc values('05' , '02' , 87);
insert into sc values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into sc values('07' , '02' , 89);
insert into sc values('07' , '03' , 98);
```



```sql
// 1.查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
select *
from 
student as s
join
(select t1.SId, t1.score as score_01, t2.score as score_02 from (
    (select SId, score from sc where CId = '01') as t1
    join 
    (select SId, score from sc where CId = '02') as t2
    on (t1.SId = t2.SId and t1.score > t2.score)
)) as t
on s.SId = t.SId;
```



```sql
// 2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
select s.SId, s.SName, t.avg_score from
student s
join
(
    select SId, avg(score) as avg_score from sc
 	group by SId 
 	having avg(score) >= 60
) as t
on s.SId = t.SId;
```



```sql
// 3. 查询在 SC 表存在成绩的学生信息
select * from student
where SId in (select distinct SId from sc);
```



```sql
// 4.查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为null)
select s.SId, s.SName, t.course_cnt, t.score_sum 
from 
student s
left join
(select SId, count(*) as course_cnt, sum(score) as score_sum from sc group by SId) as t
on s.SId = t.SId;
```



```sql
// 4.1 查有成绩的学生信息
select * from student
where SId in (select distinct SId from sc);

select * from student
where exists (select 1 from sc where student.SId = sc.SId);
```



```sql
// 5. 查询「李」姓老师的数量
select count(*) from teacher
where TName like '李%';
```



```sql
// 6. 查询学过「张三」老师授课的同学的信息
select student.*
from 
	student,
	course,
	teacher,
	sc
where
	teacher.TName = '张三' and
	course.TId = teacher.TId and
	sc.CId = course.CId and
	student.SId = sc.SId;
```



```sql
// 7. 查询没有学全所有课程的同学的信息
select * from student
where SId not in (
	select SId from sc
    group by SId
    having count(*) = (select count(CId) from course) 
);
```



```sql
// 8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
select distinct * 
from student, sc
where 
	student.SId = sc.SId AND
	student.SId != '01' AND
    sc.CId in (select CId from sc where SId = '01');
    
```



```sql
// 10.查询没学过"张三"老师讲授的任一门课程的学生姓名
select * from student
where student.SId not in (
	select sc.SId 
    from course, teacher, sc 
    where teacher.TName = '张三' and 
    	  course.TId = teacher.TId and 
    	  sc.CId = course.CId
);
```



```sql
// 11.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

select s.SID, s.SName, t.avg_score from 
student s
join 
(
    select SId, avg(score) as avg_score from sc
	where score < 60
	group by SId
	having count(*) >= 2
) as t
on s.SId = t.SId;

```



```sql
// 12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
select student.*, sc.score from student, sc, course
where student.SId = sc.SId
	and sc.CId = '01'
	and sc.score < 60
order by sc.score desc;
```



```sql
// 13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
select student.*, sc.CId, sc.score, t.avg_score 
from 
student 
join
sc 
on student.SId = sc.SId
join (
	select SId, avg(score) as avg_score from sc
	group by SId
) as t
on sc.SId = t.SId
order by t.avg_score desc;
```



```sql
// 14. 查询各科成绩最高分、最低分和平均分, 以如下形式显示：
课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
(及格为: >=60，中等为：70-80，优良为：80-90，优秀为：>=90)
要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

select
	sc.CId,
	course.CName, 
	max(score) as 最高分, 
	min(score) as 最低分, 
	avg(score) as 平均分, 
	count(*) as 选修人数,
	sum(case when sc.score>=60 then 1 else 0 end) / count(*) as 及格率,
	sum(case when sc.score>=70 and score < 80 then 1 else 0 end) / count(*) as 中等率,
	sum(case when sc.score>=80 and score < 90 then 1 else 0 end) / count(*) as 优良率,
	sum(case when sc.score>=90 and score < 80 then 1 else 0 end) / count(*) as 优秀率
from sc, course
where sc.CId = course.CId
group by sc.CId
order by 选修人数 desc, sc.CId;
```



```sql
// *15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
select sc1.CId, sc1.SId, sc1.score, count(sc2.score) + 1 as rank
from sc as sc1 left join sc as sc2 on sc1.score < sc2.score and sc1.CId = sc2.CId
group by sc1.CId, sc1.SId, sc1.score
order by sc1.CId, rank;
```



```sql
// 16. 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
set @rank = 0;
select SId, score_sum, @rank := @rank + 1 as rank
from (
    select SId, sum(score) as score_sum from sc
	group by SId
	order by score_sum desc
) as t;


// *16.1 查询学生的总成绩，并进行排名，总分重复时保留名次空缺
select t1.SId, t1.total, count(t2.SId) + 1 as rank from
(select SId, sum(score) as total from sc group by SId) as t1
left join
(select SId, sum(score) as total from sc group by SId) as t2
on t1.total < t2.total
group by t1.SId, t1.total
order by rank;
```



```sql
// 17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，(85-70]，(70-60]，(60-0] 及所占百分比

select
	sc.CId, 
	course.CName,
	sum(case when sc.score >= 85 then 1 else 0 end) as '[100-85]',
	sum(case when sc.score >= 70 and sc.score < 85 then 1 else 0 end) as '[85-70]',
	sum(case when sc.score >= 60 and sc.score < 70 then 1 else 0 end) as '[70-60]',
	sum(case when sc.score < 60 then 1 else 0 end) as '[60-0]'
from sc, course
where sc.CId = course.CId
group by CId;

```



```sql
// 18. 查询各科成绩前三名的记录 (思路：前三名转化为若大于此成绩的数量少于3即为前三名)
select t1.CId, t1.SId, t1.score from sc as t1
where (select count(*) from sc as t2 where t2.CId = t1.CId and t2.score > t1.score) + 1 <= 3
order by t1.CId, t1.score desc;
```



```sql
// 19. 查询每门课程被选修的学生数
select CId, count(SId) from sc
group by CId;
```



```sql
// 20. 查询出只选修两门课程的学生学号和姓名
select student.SId, student.SName from student, sc
where student.SId = sc.SId
group by sc.SId
having count(CId) = 2;
```



```sql
// 21. 查询男生、女生人数
select Ssex, count(*) from student
group by Ssex;
-------------------------------------------------------------
select 
	sum(case when Ssex = '男' then 1 else 0 end) as 男生人数,
	sum(case when Ssex = '女' then 1 else 0 end) as 女生人数
from student;
```



```sql
// 22. 查询名字中含有「风」字的学生信息
select * from student
where SName like '%风%';
```



```sql
// 23. 查询同名同性别学生名单，并统计同名人数
select student.*, t.同名同性别人数
from student join (
    select SName, Ssex, count(*) as 同名同性别人数 from student
	group by SName, Ssex
	having 同名同性别人数 > 1
) as t
on student.SName = t.SName and student.Ssex = t.Ssex;
```



```sql
// 24. 查询 1990 年出生的学生名单
select * from student
where year(Sage) = 1990;
```



```
// 25. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
select CId, avg(score) as 平均成绩 from sc
group by CId
order by 平均成绩 desc, CId;
```



```sql
// 26. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
select student.SId, student.SName, t.平均成绩
from student inner join (
	select SId, avg(score) as 平均成绩 from sc
	group by SId
	having 平均成绩 >= 85
) as t
on student.SId = t.SId;
```



```sql
// 27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数

select student.SName, sc.score
from student, course, sc
where course.CName = '数学'
and sc.CId = course.CId
and sc.score < 60
and student.SId = sc.SId;
```



```sql
// 28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
select student.*, sc.CId, sc.score
from student left join sc
on student.SId = sc.SId;
```



```sql
// 29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
select student.SName, course.CName, sc.score
from student, course, sc
where student.SId = sc.SId
and sc.score > 70
and sc.CId = course.CId;
```



```sql
// 30. 查询存在不及格的课程
select distinct sc.CId from sc where score < 60;
```



```sql
// 31. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
select student.SId, student.SName, sc.score
from student, sc
where student.SId = sc.SId
and sc.CId = '01'
and sc.score >= 80;
```



```sql
// 32. 求每门课程的学生人数
select CId, count(*) from sc
group by CId;
```



```sql
// 33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
select student.*, sc.score from student, sc, teacher, course
where teacher.TName = '张三'
and course.TId = teacher.TId 
and sc.CId = course.CId
and student.SId = sc.SId
order by sc.score desc
limit 1;
```



```sql
// 34. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

// 为了验证这一题，先修改原始数据
update sc 
set score = 90
where SId = '07' and CId ='02';

select student.*, sc.score from student, sc, course, teacher
where teacher.TName = '张三'
and course.TId = teacher.TId
and sc.CId = course.CId
and student.SId = sc.SId
and sc.score = (
    select max(score) from sc, course, teacher
	where teacher.TName = '张三'
	and course.TId = teacher.TId
	and sc.CId = course.CId
);
```



```sql
// 35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
select * from sc sc1 join sc sc2
on sc1.score = sc2.score and sc1.SId = sc2.SId and sc1.CId != sc2.CId;
```



```sql
// 36. 查询每门功成绩最好的前两名
select * from sc as t1
where (select count(*) from sc t2 where t1.CId = t2.CId and t2.score > t1.score) + 1 <= 2
order by t1.CId, t1.score desc;
```



```sql
// 37.统计每门课程的学生选修人数（超过 5 人的课程才统计）
select CId, count(*) as 选修人数 from sc
group by CId having 选修人数 > 5;
```



```sql
// 38.检索至少选修两门课程的学生学号
select SId, count(*) as 选修课程数 from sc
group by SId having 选修课程数 >= 2;
```



```sql
// 39. 查询选修了全部课程的学生信息
select student.* from student inner join (
	select SId from sc
	group by SId having count(*) = (select count(*) from course)
) as t
on student.SId = t.SId;
----------------------------------------------------------------------------------------
select student.* from sc, student
where sc.SId = student.SId
group by SId having count(*) = (select count(*) from course);
```




```sql
// 40. 查询各学生的年龄，只按年份来算
select *, year(now()) - year(Sage) as age from student;
----------------------------------------------------------------------------------------
select *, timestampdiff(YEAR, Sage, now()) as age from student;
```




```sql
// 41. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
select *, year(now()) - year(Sage) as age from student;
----------------------------------------------------------------------------------------
select *, timestampdiff(YEAR, Sage, now()) as age from student;
```




```sql
// 42.查询本周过生日的学生
```



```sql
// 43. 查询下周过生日的学生
```



```sql
// 44.查询本月过生日的学生
select * from student
where Month(Sage) = Month(curDate());
```




```sql
// 45.查询下月过生日的学生
```
