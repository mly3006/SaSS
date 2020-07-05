# 开发教务系统--部分随堂记录
## Django部分
* 安装django`pip install`
* 构建一个基于Django的基本框架的Web的程序
   * `django-adminstartproject mysite`
   * `cd mysite`
   * `pyhthon manage.py startapppolls`
   * `python manage.py runserver`
 * 访问`http://127.0.0.1:8000/`
![](images/6.jpg)
* 下载`sqlite-tools-win32-x86-3310100.zip`,并添加环境变量，执行`sqlite3.exe db.sqlite3`，进入命令后在执行table命令可以看到自己定义的三个表
![](images/5.jpg)
-----------------------------
## 实验要求
* 不使用Django的情况下，使用更底层的pyhton的sqlite库来编程操作数据库
* 写两个页面，一个是教师录入成绩页面，一个是学生查询成绩页面。教师录入成绩页面表单有三个字段，课程id，学生id，成绩
* httpserver调用sqlite库使用sql语句写入数据库。然后是学生查询成绩表单，学生输入学生id，课程id，httpserver使用sql语句查询成绩后返回给用户
## 实验过程
* 创建教务数据库和新建teacher表格
  代码部分
```python
# -*- coding: utf-8 -*-
import sqlite3
conn = sqlite3.connect('./edu_admin.db')
c = conn.cursor()

# Create teacher table 
c.execute('''CREATE TABLE teacher(courseid,studentid,grade)''')
# Insert a row of data
# c.execute("INSERT INTO teacher VALUES ('1','1','90')")
# Save (commit) the change
conn.commit()
# We can also close the connection if we are done with it.
# Just be sure any changes have been committed or they will be lost.
conn.close()
```
* 编写教师录入成绩页面和学生查询成绩页面
代码部分
```python
# -*- coding: utf-8 -*-

import sys
import cgi
from http.server import HTTPServer, BaseHTTPRequestHandler
import sqlite3

def insert_into_db(course_id,student_id,grade):
    print("insert database")
    conn = sqlite3.connect('./edu_admin.db')
    c = conn.cursor()
    try:
        c.execute("select * from teacher where courseid = %s and studentid = %s " % (course_id,student_id))
        jud = c.fetchall()
        # 如果学生已有成绩，不允许重复插入
        if len(jud) == 0:
            c.execute("INSERT INTO teacher VALUES (%s,%s,%s)" %(course_id,student_id,grade))
            # Save (commit) the change
            conn.commit()
    except:
        print("already exist")
        conn.rollback()
    conn.close()

def select_from_db(course_id,student_id):
    print("select database")
    conn = sqlite3.connect('./edu_admin.db')
    c = conn.cursor()
    try:
        c.execute("select * from teacher where courseid = %s and studentid = %s " % (course_id,student_id))
        res = c.fetchall()
        if len(res) == 0:
            print("no student / no course")
            conn.close()
            return "error"
        else:
            conn.close()
            return res
    except:
        print("error")
        conn.close()
        return "error"

class MyHTTPRequestHandler(BaseHTTPRequestHandler):
    
    course_id = 'course_id'
    student_id = 'student_id'
    grade = 'grade'
    
    def do_GET(self):
        
        print("get success")

        path = self.path
        print(path)
        if path == "/grade":
            form_html = \
                '''
                <html>
                <body>
                <form method='post' enctype='multipart/form-data'>
                query the scores </br>
                course id:</br>
                <input type='text' name='%s'></br>
                student id:</br>
                <input type='text' name='%s'></br>
                <input type='submit'>
                </form>
                </body>
                </html>
                ''' % ('course_id','student_id')
            
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()
            self.wfile.write(form_html.encode())
                
        elif path == "/teacher": 
            form_html = \
                '''
                <html>
                <body>
                <form method='post' enctype='multipart/form-data'>
                course id:</br>
                <input type='text' name='%s'></br>
                student id:</br>
                <input type='text' name='%s'></br>
                grade:<br>
                <input type='text' name='%s'></br>
                <input type='submit'>
                </form>
                </body>
                </html>
                ''' % ('course_id','student_id','grade')
                
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()
            self.wfile.write(form_html.encode())

    def do_POST(self):
        print("post success")
        form_data = cgi.FieldStorage(
            fp=self.rfile,
            headers=self.headers,
            environ={
                'REQUEST_METHOD': 'POST',
                'CONTENT_TYPE': self.headers['Content-Type'],
            })
        fields = form_data.keys()
        if len(fields) == 2:
            if self.course_id in fields and self.student_id in fields:
                course = form_data[self.course_id].value
                print(course)
                student = form_data[self.student_id].value
                print(student)
                # database
                res = select_from_db(course,student)
                print(res)
            
                if res == "error":
                    self.send_response(200)
                    self.send_header("Content-type", "text/html")
                    self.end_headers()
                    self.wfile.write(b"<html><body>query error </body></html>")
                else:
                    result = res[0]
                    result = result[2]
                    self.send_response(200)
                    self.send_header("Content-type", "text/html")
                    self.end_headers()
                    str = "<html><body>scores: %s </body></html>" % result
                    self.wfile.write(str.encode())
        else :
            if self.course_id in fields and self.student_id in fields and self.grade in fields:
                course = form_data[self.course_id].value
                print(course)
                student = form_data[self.student_id].value
                print(student)
                gra = form_data[self.grade].value
                print(gra)
                # file = open("."+self.path, "wb")
                # file.write("course_id: %s student_id: %s grade: %s \n" % (course,student,gra))
                
                # database
                insert_into_db(course,student,gra)

            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()
            self.wfile.write(b"<html><body>OK</body></html>")

# MyHTTPServer类，是继承自原生的HTTPSever
class MyHTTPServer(HTTPServer):
    def __init__(self, host, port):
        print("run app server by python!")
        HTTPServer.__init__(self,  (host, port), MyHTTPRequestHandler)

if '__main__' == __name__:
    server_ip = "127.0.0.1"
    server_port = 8080
    if len(sys.argv) == 2:
        server_port = int(sys.argv[1])
    if len(sys.argv) == 3:
        server_ip = sys.argv[1]
        server_port = int(sys.argv[2])
    print("App server is running on http://%s:%s " % (server_ip, server_port))

    server = MyHTTPServer(server_ip, server_port)
    server.serve_forever()
```
## 实验结果
没有运行出来正确的结果，也没找到问题出在哪里。

