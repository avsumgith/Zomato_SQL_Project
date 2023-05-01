# Zomato_SQL_Project
SQL_Portfolio_project


drop table if exists golduser_signup;

create table goldusers_signup(userid integer, gold_signup_date date);

insert into goldusers_signup(userid,gold_signup_date)
values(1,"2017-09-22"),(3,"2017-04-21");

drop table if exists users;

create table users(userid integer, signup_date date);

insert into users(userid,signup_date)
values (1,"2014-02-09"),(2,"2015-01-15"),(3,"2014-04-11");

drop table if exists sales;

create table sales(userid integer,created_date date,product_id integer);

insert into sales(userid,created_date,product_id)
values(1,"2017-01-19",2),
(3,"2019-12-18",1),
(2,"2020-07-20",3),
(1,"2019-10-23",2),
(1,"2018-03-19",3),
(3,"2016-12-20",2),
(1,"2016-09-11",1),
(1,"2017-09-24",1),
(2,"2017-09-24",1),
(1,"2017-11-03",2),
(1,"2016-11-03",1),
(3,"2016-11-10",1),
(3,"2017-12-07",2),
(3,"2016-12-15",2),
(2,"2017-11-08",2),
(2,"2018-10-09",3);

drop table if exists product;

create table product(product_id integer,product_name text,price integer);

insert into product(product_id,product_name,price)
values
(1,'p1',980),
(2,'p2',870),
(3,'p3',330);

select *from sales;
select *from users;
select *from product;
select *from zomato.goldusers_signup;

-- 1.What is total amount each customer spent on zomato? --

select a.userid,sum(b.price) total_amt_spent from sales a inner join product b on 
a.product_id=b.product_id group by a.userid;


-- 2.How many days has each customer visited zomato? --

select userid,count(distinct created_date) distinct_days from sales group by userid;


-- 3. What was the first product purchased by each customer? --

select * from 
(select *,rank() over(partition by userid order by created_date) rnk from sales) a where rnk=1;

-- 4.What is most purchased item on menu & how many times was it purchased by all customers? --

SELECT userid,count(product_id) cnt from sales where product_id =
(select top 1 product_id from sales group by product_id order by count(product_id)desc)
group by userid;

-- 5.Which item was most popular for each cutomer?--

select * from
(select *, rank() over(partition by userid order by cnt desc) rnk from
(select userid,product_id,count(product_id) cnt from sales group by userid,product_id)a)b
where rnk =1;

-- 6. Which item was purchased first by customer after they become a member? --

select *from 
(select c.*,rank() over(partition by userid order by created_date) rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a inner join
goldusers_signup b on a.userid=b.userid and created_date>=gold_signup_date) c)d where rnk=1;



-- 7. Which item was purchased just before customer bacome a member? --

select *from 
(select c.*,rank() over(partition by userid order by created_date) rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a inner join
goldusers_signup b on a.userid=b.userid and created_date<=gold_signup_date) c)d where rnk=1;



-- 8. What is the total orders and amount spent fro each member before they become a member? --

select userid,count(created_date) order_purchased, sum(price) total_amnt_spent from
(select c.*,d.price from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a inner join
goldusers_signup b on a.userid=b.userid and created_date<=gold_signup_date)c inner join product d on 
c.product_id=d.product_id)e group by userid;

-- 9. If buying each product generates points for eg 5rs=2 zomato point and each has different purchasing points
-- for eg for p1 5rs=1 zomato point, for p2 10rs=5 zomato point and p3 5rs=1 zomato point 2rs=1zomato point --

select userid, sum(total_points)*2.5 total_money_earned from
(select e.*,amt/points total_points from
(select d.*,case when product_id=1 then 5 when product_id=2 then 2 when product_id=3 then 5 else 0 end as points from
(select c.userid,c.product_id,sum(price) amt from
(select a.*,b.price from sales a inner join product b on a.product_id=b.product_id) c
group by userid,product_id)d)e)f group by userid;

-- 10. In the first one year after a customer joins the gold program (including their join date) irrespective of
-- what the customer has purchased they earn 5 zomato points for every 10 rs spent who earned more 1 or 3
-- and what was their points earnings in their first year? --

select c.*,d.price*0.5 total_points_earned from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a inner join
goldusers_signup b on a.userid=b.userid and created_date>=gold_signup_date and created_date<=DATEADD(year,1,gold_signup_date))c
inner join product d on c.product_id=d.product_id;

-- 11. Rank all the transaction of members? --

select *,rank() over(partition by userid order by created_date) rnk from sales;

-- 12. Rank all the transactions for each member whenever they are a zomato gold member for every non gold member
-- transactions mark as na?--

select e.*,case when rnk=0 then 'na' else rnk end as rnkk from 
(select e.*,cast((case when gold_signup_date is null then 0 else rank() over (partition by userid order by created_date desc)end) as varchar) as rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a left join
goldusers_signup b on a.userid=b.user_id and created_date>=gold_signup_date)c)e;



