import mysql.connector
from flask import Flask, request, render_template,redirect,session,flash
from flask_session import Session
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SESSION_TYPE'] = 'filesystem'
app.config['SECRET_KEY'] = 'abc'
Session(app)

db_config = {
     'host' : 'localhost',
     'user' : 'root',
     'password' : 'Aulakh@007',
     'database' : 'librarydb'
}
conn = mysql.connector.connect(**db_config)
cursor = conn.cursor()



@app.route("/")
def home():
     if 'Name' not in session:
        return redirect('/login')
     return render_template("index.html") 

@app.route('/add_faculty', methods=['POST','GET'])
def add_faculty():

    if 'Name' not in session:
        return redirect('/login')
    
    elif request.method == 'POST':
            name = request.form.get('name')
            branch = request.form.get('branch')
            connection = mysql.connector.connect(**db_config)
            cursor = connection.cursor()
            query = "INSERT INTO faculty (Name, Branch) VALUES (%s, %s)"
            values = (name, branch)
            cursor.execute(query, values)
            connection.commit()
            cursor.close()
            connection.close()
            return redirect("/faculty")
    return render_template('addfaculty.html')
    
@app.route('/faculty')
def list_faculty():
    connection = mysql.connector.connect(**db_config)
    cursor = connection.cursor()
    query = "SELECT * FROM faculty"
    cursor.execute(query)
    faculty = cursor.fetchall()
    connection.commit()
    cursor.close()
    connection.close()
    return render_template("facultylist.html", faculty=faculty)

@app.route('/add_student', methods=['POST','GET'])
def add_student():

    if 'Name' not in session:
        return redirect('/login')
    
    elif request.method == 'POST':
            name = request.form.get('name')
            branch = request.form.get('branch')
            connection = mysql.connector.connect(**db_config)
            cursor = connection.cursor()
            query = "INSERT INTO student (Name, Branch) VALUES (%s, %s)"
            values = (name, branch)
            cursor.execute(query, values)
            connection.commit()
            cursor.close()
            connection.close()
            return redirect("/students")
    return render_template('addStudent.html')        

@app.route('/students')
def list_student():

    if 'Name' not in session:
        return redirect('/login')
    else:
        connection = mysql.connector.connect(**db_config)
        cursor = connection.cursor()
        query = "SELECT * FROM student"
        cursor.execute(query)
        student = cursor.fetchall()
        cursor.close()
        connection.close()

    return render_template('student.html', student=student)
    
@app.route('/add_book', methods=['POST','GET'])
def add_book():

    if 'Name' not in session:
        return redirect('/login')
    
    elif request.method == 'POST':
            title = request.form.get("title")
            branch = request.form.get("branch")
            author_id = request.form.get("author_id")
            connection = mysql.connector.connect(**db_config)
            cursor = connection.cursor()
            query = "INSERT INTO Book (Title, Branch, Author_ID) VALUES (%s, %s, %s)"
            values = (title, branch, author_id)
            cursor.execute(query, values)
            connection.commit()
            return redirect("/book")
    return render_template("addbook.html")
    
@app.route('/book')
def list_books():
    if 'Name' not in session:
        return redirect('/login')
    else:
        connection = mysql.connector.connect(**db_config)
        cursor = connection.cursor()
        query = '''SELECT 
                    b.Book_ID,
                    b.title,
                    b.branch,
                    COALESCE(a.Name) AS Author_Name
                    FROM book b
                    LEFT JOIN authors a ON b.Author_ID = a.Author_ID
                    '''
        cursor.execute(query)
        book = cursor.fetchall()
        cursor.close()
        connection.close()
    return render_template("listofbook.html", book=book)

@app.route('/borrow_book', methods=['POST','GET'])
def borrow_book():
    connection = mysql.connector.connect(**db_config)
    cursor = connection.cursor()

    if 'Name' not in session:
        return redirect('/login')
    
    elif request.method == 'POST':
        student_id = request.form.get('student_id')
        faculty_id = request.form.get('faculty_id')
        admin_Name = session['Name']
        book_id = request.form.get('book_id')
        borrow_date = request.form.get('borrow_date')


        if (student_id and faculty_id) or (not student_id and not faculty_id):
             return "Select either a student or a faculty member, not both.", 400
        
        student_id = int(student_id) if student_id else None
        faculty_id = int(faculty_id) if faculty_id else None
        
        query = """
                INSERT INTO BorrowedBooks (Student_ID, Faculty_ID, Book_ID, Borrow_Date,admin_Name)
                VALUES (%s, %s, %s, %s,%s)
            """
        values = (student_id, faculty_id, book_id, borrow_date,admin_Name)
        cursor.execute(query, values)
        connection.commit()
        cursor.close()
        connection.close()

        return redirect("/borrow_list")
    return render_template("Book_issued.html")

@app.route("/borrow_list")
def borrow_list():
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()

     if 'Name' not in session:
        return redirect('/login')
     else:
        query = '''SELECT
                i.Borrow_ID, 
                b.title,
                COALESCE(s.Name, 'N/A') AS Student_Name,
                COALESCE(f.Name, 'N/A') AS Faculty_Name, 
                i.borrow_date,
                i.renew_date,
                i.return_date,
                admin_Name
                FROM borrowedbooks i
                LEFT JOIN student s ON i.Student_ID = s.Student_ID
                LEFT JOIN faculty f ON i.Faculty_ID = f.Faculty_ID
                LEFT JOIN book b ON i.Book_ID = b.Book_ID;
                '''
        cursor.execute(query)
        borrowedbooks = cursor.fetchall()
        cursor.close()
        connection.close()
     return render_template("borrow_list.html",borrowedbooks=borrowedbooks)

@app.route("/update_borrow/<int:Borrow_ID>", methods=["GET", "POST"])
def update_borrow(Borrow_ID):
     connection = mysql.connector.connect(**db_config)
     cursor=connection.cursor()

     if 'Name' not in session:
        return redirect('/login')
     else:
        query = '''SELECT
                i.Borrow_ID, 
                b.title,
                COALESCE(s.Name, 'N/A') AS Student_Name,
                COALESCE(f.Name, 'N/A') AS Faculty_Name, 
                i.borrow_date,
                i.renew_date,
                i.return_date
                FROM borrowedbooks i
                LEFT JOIN student s ON i.Student_ID = s.Student_ID
                LEFT JOIN faculty f ON i.Faculty_ID = f.Faculty_ID
                LEFT JOIN book b ON i.Book_ID = b.Book_ID
                where i.Borrow_ID=%s; 
                '''
        cursor.execute(query, (Borrow_ID,))
        borrow_record = cursor.fetchone()

        if request.method=='POST':
            renew_date = request.form.get("renew_date")
            return_date = request.form.get("return_date")

            renew_date = renew_date if renew_date else None
            return_date = return_date if return_date else None

            update_query = """
                UPDATE borrowedbooks 
                SET renew_date= %s, return_date = %s 
                WHERE Borrow_ID = %s
                """
            cursor.execute(update_query, (renew_date, return_date, Borrow_ID))
            connection.commit()
        
            cursor.close()
            connection.close()
            return redirect(("/borrow_list"))
        cursor.close()
        connection.close()
     return render_template("update_table.html", borrow=borrow_record, Borrow_ID=Borrow_ID)

@app.route('/update_book/<int:Book_ID>', methods=['POST','GET'])
def update_book(Book_ID):
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()

     if 'Name' not in session:
        return redirect('/login')
     else:
        query = '''SELECT 
                    b.Book_ID,
                    b.title,
                    b.branch,
                    b.author_id
                    FROM book b 
                    Where b.Book_ID = %s
                '''
        cursor.execute(query,(Book_ID,))
        book_record = cursor.fetchone()

        if request.method == 'POST':
            title = request.form.get("title")
            branch = request.form.get("branch")
            author_id = request.form.get("author_id")

            update_query = """
                UPDATE book 
                SET title = %s, branch = %s, author_id = %s 
                WHERE Book_ID = %s
                """
            
            cursor.execute(update_query, (title, branch, author_id, Book_ID))
            connection.commit()

            cursor.close()
            connection.close()
            return redirect("/book")
        cursor.close()
        connection.close()
     return render_template("update_book.html",book=book_record, Book_ID=Book_ID)

@app.route('/update_student/<int:Student_ID>', methods=['POST','GET'])
def update_student(Student_ID):
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()

     if 'Name' not in session:
        return redirect('/login')
     else:
        query = '''SELECT 
                    s.Student_ID,
                    s.name,
                    s.branch
                    FROM student s 
                    Where s.Student_ID = %s
                '''
        cursor.execute(query,(Student_ID,))
        student_record = cursor.fetchone()

        if request.method == 'POST':
            name = request.form.get("name")
            branch = request.form.get("branch")

            update_query = """
                UPDATE student 
                SET name = %s, branch = %s 
                WHERE Student_ID = %s
                """
            
            cursor.execute(update_query, (name, branch, Student_ID))
            connection.commit()

            cursor.close()
            connection.close()
            return redirect("/students")
        cursor.close()
        connection.close()
     return render_template("update_student.html",student=student_record, Student_ID=Student_ID)

@app.route('/update_faculty/<int:Faculty_ID>', methods=['POST','GET'])
def update_faculty(Faculty_ID):
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()

     if 'Name' not in session:
        return redirect('/login')
     else:
        query = '''SELECT 
                    s.Faculty_ID,
                    s.name,
                    s.branch
                    FROM faculty s 
                    Where s.Faculty_ID = %s
                '''
        cursor.execute(query,(Faculty_ID,))
        faculty_record = cursor.fetchone()

        if request.method == 'POST':
            name = request.form.get("name")
            branch = request.form.get("branch")

            update_query = """
                UPDATE faculty 
                SET name = %s, branch = %s 
                WHERE Faculty_ID = %s
                """
            
            cursor.execute(update_query, (name, branch, Faculty_ID))
            connection.commit()

            cursor.close()
            connection.close()
            return redirect("/faculty")
        cursor.close()
        connection.close()
     return render_template("update_faculty.html",faculty=faculty_record, Faculty_ID=Faculty_ID)

@app.route("/delete_student/<int:Student_ID>", methods=['DELETE','POST','GET'])
def delete_student(Student_ID):
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()
     if 'Name' not in session:
        return redirect('/login')
     else:
        query = "DELETE FROM student WHERE Student_ID = %s"

        cursor.execute(query,(Student_ID,))
        connection.commit()

        cursor.close()
        connection.close()

        return redirect("/students")


@app.route("/delete_faculty/<int:Faculty_ID>", methods=['POST','GET'])
def delete_faculty(Faculty_ID):
     
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()
     
     if 'Name' not in session:
        return redirect('/login')
     elif request.method=='POST' and request.form.get('DELETE'):
          
          query = "DELETE FROM faculty WHERE Faculty_ID = %s"
          
          cursor.execute(query,(Faculty_ID,))
          connection.commit()
          
          cursor.close()
          connection.close()
          return ("/faculty")
     
@app.route('/login', methods=['POST','GET'])
def login():
     connection = mysql.connector.connect(**db_config)
     cursor = connection.cursor()

     if request.method == 'POST':
          Name = request.form.get("Name")
          Password = request.form.get("Password")

          query = "SELECT * FROM admin WHERE Name = %s and Password = %s"
          cursor.execute(query,(Name,Password))
          user = cursor.fetchone()

          if user:
               session['Name'] = Name
               session_data = str(session)

               update_query = "UPDATE admin SET session_data = %s WHERE Name = %s"
               cursor.execute(update_query, (session_data, Name))
               connection.commit()
               return redirect('/loginu')
          else:
               return('invalid name or password','danger')
     cursor.close()
     connection.close()          
     return render_template("login.html")
          
@app.route('/logout')
def logout():
    session.clear()
    return redirect("/login")

@app.route('/loginu')
def login_success():
    if 'Name' in session:
        return redirect("/")
    else:
        return redirect('/login')

if __name__ == "__main__":
    app.run(debug=True)
