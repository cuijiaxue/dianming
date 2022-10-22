//前端

// dianming.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

#include <stdio.h>//c语言
#include <iostream>//c++语言
#include <string.h>//字符串
#include <conio.h>//_getch()在这个文件中
#include "include\mysql.h"//数据库
#pragma comment(lib,"./lib/libmysql.lib")//数据库
using namespace std;
int main()
{
    MYSQL mysql; //一个数据库结构体
    MYSQL_RES* res; //一个结果集结构体
    MYSQL_ROW row; //char** 二维数组，存放一条条记录
    char strsql[1024];//存放SQL语句
    //初始化数据库
    mysql_init(&mysql);
    //设置编码方式
    mysql_options(&mysql, MYSQL_SET_CHARSET_NAME, "gbk");//有汉字，gbk编码不乱码

    //连接数据库
    //判断如果连接失败就把连接失败的信息显示出来，我们好进行对应修改。
    // mysql_real_connect参数：2.本地地址 3.你的mysql用户名 4.你的mysql密码 5.数据库名字 6.端口号

    if (mysql_real_connect(&mysql, "localhost", "root", "CuiJiaxue0904", "dm", 3306, NULL, 0) == NULL) {
        cout << (mysql_error(&mysql)) << endl;
    }
    //输出所有课程，选择当前签到课程
    //查询数据
    mysql_query(&mysql, "SELECT * from course");
    //获取结果集
    res = mysql_store_result(&mysql);
    cout << "当前开设了以下课程:" << endl;
    //显示数据
    //给ROW赋值，判断ROW是否为空，不为空就打印数据。
    while (row = mysql_fetch_row(res))
    {
        printf("%s ", row[0]);//打印课程ID
        printf("%s ", row[1]);//打印课程名称
        cout << endl;
    }
    int c_id;
    printf("请选择当前的课程ID:");
    cin >> c_id;
    cout << endl;
    //逐一点名
    //查询学生表
    mysql_query(&mysql, "select * from student");
    res = mysql_store_result(&mysql);
    while (row = mysql_fetch_row(res))
    {
        printf("%s ", row[0]);//打印ID
        printf("%s ", row[1]);//打印姓名
        printf("%s ", row[2]);//打印年龄
        printf("%s ", row[3]);//打印性别
        printf("这位同学来上课了吗?(y/n):");
        char ch = _getch();//不用回车就可读入字符
        putchar(ch);
        //构造查询字符串
        if (ch == 'n' || ch == 'N') {
            sprintf_s(strsql, 1024, "insert into sign(s_id,c_id,sig,sign_time) values('%s',%d,0,now())", row[0], c_id);
        }
        else {
            sprintf_s(strsql, 1024, "insert into sign(s_id,c_id,sig,sign_time) values('%s',%d,1,now())", row[0], c_id);
        }
        //执行查询,到课情况写入数据库
        mysql_query(&mysql, strsql);
        cout << endl;
    }
    //统计一下学生缺课情况
    cout << endl << "到目前为止，学生缺课情况如下:" << endl;
    sprintf_s(strsql, 1024, "select s_name, c_name, sign_time, count(s_name) from student,course,sign where student.s_id=sign.s_id and course.c_id=sign.c_id and sign.sig=0 group by s_name");
    mysql_query(&mysql, strsql);
    res = mysql_store_result(&mysql);
    while (row = mysql_fetch_row(res))
    {
        printf("%s ", row[0]);//打印学生姓名
        printf("%s ", row[1]);//打印课程名称
        printf("%s ", row[2]);//最近一次缺课时间
        printf("%s次", row[3]);
        cout << endl;
    }
    //释放结果集
    mysql_free_result(res);
    //关闭数据库
    mysql_close(&mysql);
    //停留等待，观看结果，任意键退出
    cout << endl << "任意键退出...";
    system("pause > nul");
    return 0;
}


//数据库部分

# 创建数据库
create database if not exists dm ;
use dm;

# 学生表
CREATE TABLE if not exists Student(
s_id int,
s_name VARCHAR(20) NOT NULL DEFAULT '',
s_birth VARCHAR(20) NOT NULL DEFAULT '',
s_sex VARCHAR(10) NOT NULL DEFAULT '',
PRIMARY KEY(s_id)
);

# 课程表
CREATE TABLE if not exists Course(
c_id int,
c_name VARCHAR(20) NOT NULL DEFAULT '',
PRIMARY KEY(c_id)
);

#签到表
create table if not exists sign
(
id int auto_increment not null primary key,
s_id varchar(20) not null,
c_id varchar(20) not null,
sig bool default true,
sign_time datetime default now(),
foreign key(s_id) references student(s_id),
foreign key(c_id) references course(c_id)
);

# 插入学生表测试数据
insert into Student values(1 , '赵雷' , '1990-01-01' , '男');
insert into Student values(2 , '钱电' , '1990-12-21' , '男');
insert into Student values(3 , '孙风' , '1990-05-20' , '男');
insert into Student values(4 , '李云' , '1990-08-06' , '男');
insert into Student values(5 , '周梅' , '1991-12-01' , '女');
insert into Student values(6, '吴兰' , '1992-03-01' , '女');
insert into Student values(7 , '郑竹' , '1989-07-01' , '女');
insert into Student values(8 , '王菊' , '1990-01-20' , '女');


# 课程表测试数据
insert into Course values(1 , '语文' );
insert into Course values(2 , '数学' );
insert into Course values(3 , '英语' );

