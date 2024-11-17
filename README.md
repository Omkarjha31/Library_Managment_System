# Library_Managment_System
Mini project used to maintain library database using python and Mysql

import mysql.connector

try:
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="pass",
        database="library_db"
    )
    print("Connection successful!")
except mysql.connector.Error as err:
    print(f"Error: {err}")
    
def create_tables():
    cursor = connection.cursor()
    # Create Books table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS books (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        author VARCHAR(255) NOT NULL,
        quantity INT NOT NULL
    )
    ''')
    # Create Issued Books table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS issued_books (
        id INT AUTO_INCREMENT PRIMARY KEY,
        book_id INT NOT NULL,
        issued_to VARCHAR(255) NOT NULL,
        FOREIGN KEY (book_id) REFERENCES books (id)
    )
    ''')
    connection.commit()
    connection.close()

def add_book(title, author, quantity):
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="Pass",
        database="library_db"
    )
    cursor = connection.cursor()
    cursor.execute("INSERT INTO books (title, author, quantity) VALUES (%s, %s, %s)", (title, author, quantity))
    connection.commit()
    connection.close()

def view_books():
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="Pass",
        database="library_db"
    )
    cursor = connection.cursor()
    cursor.execute("SELECT * FROM books")
    books = cursor.fetchall()
    connection.close()
    return books

def issue_book(book_id, issued_to):
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="Pass",
        database="library_db"
    )
    cursor = connection.cursor()
    cursor.execute("SELECT quantity FROM books WHERE id = %s", (book_id,))
    book = cursor.fetchone()
    if book and book[0] > 0:
        cursor.execute("INSERT INTO issued_books (book_id, issued_to) VALUES (%s, %s)", (book_id, issued_to))
        cursor.execute("UPDATE books SET quantity = quantity - 1 WHERE id = %s", (book_id,))
        connection.commit()
        print("Book issued successfully.")
    else:
        print("Book not available.")
    connection.close()

def return_book(issue_id):
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="Pass",
        database="library_db"
    )
    cursor = connection.cursor()
    cursor.execute("SELECT book_id FROM issued_books WHERE id = %s", (issue_id,))
    issued_book = cursor.fetchone()
    if issued_book:
        cursor.execute("DELETE FROM issued_books WHERE id = %s", (issue_id,))
        cursor.execute("UPDATE books SET quantity = quantity + 1 WHERE id = %s", (issued_book[0],))
        connection.commit()
        print("Book returned successfully.")
    else:
        print("Issued book not found.")
    connection.close()

def main():
    create_tables()
    while True:
        print("\nLibrary Management System")
        print("1. Add Book")
        print("2. View Books")
        print("3. Issue Book")
        print("4. Return Book")
        print("5. Exit")
        choice = input("Enter your choice: ")
        if choice == "1":
            title = input("Enter book title: ")
            author = input("Enter book author: ")
            quantity = int(input("Enter quantity: "))
            add_book(title, author, quantity)
        elif choice == "2":
            books = view_books()
            print("\nBooks in Library:")
            for book in books:
                print(f"ID: {book[0]}, Title: {book[1]}, Author: {book[2]}, Quantity: {book[3]}")
        elif choice == "3":
            book_id = int(input("Enter book ID to issue: "))
            issued_to = input("Enter name of the person: ")
            issue_book(book_id, issued_to)
        elif choice == "4":
            issue_id = int(input("Enter issued book ID to return: "))
            return_book(issue_id)
        elif choice == "5":
            print("Exiting...")
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()
