# Library_managment--SQL-SERVER
CREATE DATABASE LibraryManagement;
USE LibraryManagement;
--create tables
-- Table to store book details
CREATE TABLE Books (
    book_id INT PRIMARY KEY IDENTITY(1,1),
    ISBN VARCHAR(13) UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    author_id INT,
    genre VARCHAR(50),
    publication_year INT,
    is_available BIT DEFAULT 1
);

-- Table to store author details
CREATE TABLE Authors (
    author_id INT PRIMARY KEY IDENTITY(1,1),
    name VARCHAR(255) NOT NULL,
    nationality VARCHAR(50),
    birth_year INT
);

-- Table to store member information
CREATE TABLE Members (
    member_id INT PRIMARY KEY IDENTITY(1,1),
    name VARCHAR(100) NOT NULL,
    contact_info VARCHAR(100),
    join_date DATE DEFAULT GETDATE()
);

-- Table to store borrow records
CREATE TABLE BorrowRecords (
    borrow_id INT PRIMARY KEY IDENTITY(1,1),
    book_id INT FOREIGN KEY REFERENCES Books(book_id),
    member_id INT FOREIGN KEY REFERENCES Members(member_id),
    borrow_date DATE DEFAULT GETDATE(),
    due_date DATE NOT NULL,
    return_date DATE,
    fine DECIMAL(10, 2) DEFAULT 0
);

-- Table to store fine details
CREATE TABLE Fines (
    fine_id INT PRIMARY KEY IDENTITY(1,1),
    member_id INT FOREIGN KEY REFERENCES Members(member_id),
    amount_due DECIMAL(10, 2),
    date_issued DATE DEFAULT GETDATE(),
    date_paid DATE
);
--Insert_records
-- Adding books
INSERT INTO Books (ISBN, title, author_id, genre, publication_year)
VALUES 
    ('9780747532743', 'Harry Potter and the Philosopher''s Stone', 1, 'Fantasy', 1997),
    ('9780553103540', 'A Game of Thrones', 2, 'Fantasy', 1996),
    ('9780062073488', 'Murder on the Orient Express', 3, 'Mystery', 1934),
    ('9780261103573', 'The Hobbit', 4, 'Fantasy', 1937),
    ('9780385504201', 'The Da Vinci Code', 5, 'Thriller', 2003),
    ('9780307277671', 'Inferno', 5, 'Thriller', 2013),
    ('9780261102385', 'The Lord of the Rings', 4, 'Fantasy', 1954),
    ('9780007118359', 'The Two Towers', 4, 'Fantasy', 1954),
    ('9780007136599', 'The Return of the King', 4, 'Fantasy', 1955),
    ('9780007141326', 'The Fellowship of the Ring', 4, 'Fantasy', 1954);
	------------------------------------------------------------------------------------------
-- Adding authors
INSERT INTO Authors (name, nationality, birth_year)
VALUES 
    ('J.K. Rowling', 'British', 1965),
    ('George R.R. Martin', 'American', 1948),
    ('Agatha Christie', 'British', 1890),
    ('J.R.R. Tolkien', 'British', 1892),
    ('Dan Brown', 'American', 1964);
------------------------------------------------------------------------------------------------
-- Adding members
INSERT INTO Members (name, contact_info)
VALUES 
    ('Alice Johnson', 'alice@example.com'),
    ('Bob Smith', 'bob@example.com'),
    ('Charlie Brown', 'charlie@example.com'),
    ('Diana Prince', 'diana@example.com'),
    ('Evan Peters', 'evan@example.com'),
    ('Frank Castle', 'frank@example.com'),
    ('Grace Lee', 'grace@example.com'),
    ('Hannah Moore', 'hannah@example.com'),
    ('Ian Fleming', 'ian@example.com'),
    ('Julia Roberts', 'julia@example.com');
-------------------------------------------------------------------------------------
-- Adding borrow records with various due and return dates
INSERT INTO BorrowRecords (book_id, member_id, borrow_date, due_date, return_date, fine)
VALUES 
    (1, 1, '2024-09-15', '2024-09-30', '2024-09-29', 0),         -- Returned on time
    (2, 2, '2024-09-20', '2024-10-05', '2024-10-07', 1.00),      -- 2 days late
    (3, 3, '2024-09-25', '2024-10-10', NULL, 0),                 -- Not returned yet
    (4, 4, '2024-09-27', '2024-10-12', NULL, 0),                 -- Not returned yet
    (5, 5, '2024-10-01', '2024-10-16', '2024-10-15', 0),         -- Returned on time
    (6, 6, '2024-10-02', '2024-10-17', NULL, 0),                 -- Not returned yet
    (7, 7, '2024-10-03', '2024-10-18', '2024-10-19', 0.50),      -- 1 day late
    (8, 8, '2024-10-04', '2024-10-19', NULL, 0),                 -- Not returned yet
    (9, 9, '2024-10-05', '2024-10-20', '2024-10-22', 1.00),      -- 2 days late
    (10, 10, '2024-10-06', '2024-10-21', NULL, 0);               -- Not returned yet
----------------------------------------------------------------------------------------------------
-- Adding fine records for overdue books
INSERT INTO Fines (member_id, amount_due, date_issued, date_paid)
VALUES 
    (2, 1.00, '2024-10-07', '2024-10-10'),     -- Fine paid
    (7, 0.50, '2024-10-19', NULL),             -- Fine pending
    (9, 1.00, '2024-10-22', NULL);             -- Fine pending
------------------------------------------------------------------------------
--List of all avl boooks on store..
SELECT title, genre, publication_year 
FROM Books
WHERE is_available = 1;

--------------------------------------------------------------------------------
--Records of that books i.e Currently Borrowed and Overdue:
SELECT b.title, m.name, br.due_date, DATEDIFF(DAY, br.due_date, GETDATE()) AS days_overdue
FROM BorrowRecords br
JOIN Books b ON br.book_id = b.book_id
JOIN Members m ON br.member_id = m.member_id
WHERE br.return_date IS NULL AND br.due_date < GETDATE();
-----------------------------------------------------------------------------------
--pending fines from members..
SELECT m.name, SUM(f.amount_due) AS total_pending_fines
FROM Fines f
JOIN Members m ON f.member_id = m.member_id
WHERE f.date_paid IS NULL
GROUP BY m.name;
------------------------------------------------------------------------------------
--Most liked books...
SELECT b.title, COUNT(br.borrow_id) AS borrow_count
FROM BorrowRecords br
JOIN Books b ON br.book_id = b.book_id
GROUP BY b.title
ORDER BY borrow_count DESC;

---------------------------------------------------------
--Members with overdues days
SELECT m.name, b.title, br.due_date, DATEDIFF(DAY, br.due_date, GETDATE()) AS days_overdue
FROM BorrowRecords br
JOIN Members m ON br.member_id = m.member_id
JOIN Books b ON br.book_id = b.book_id
WHERE br.return_date IS NULL AND br.due_date < GETDATE();





