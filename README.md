# Library-management-system-project-2

**Project Title: Library Management System
Level: Intermediate
Database: sql_project_p2**


![image](https://github.com/user-attachments/assets/06ec2a30-205b-403e-b698-3add9fc7bd45)




This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


Table Creation: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.


-- Library managment systerm project 2

***creating Database and managing tables**



create database sql_project_p2; 
use sql_project_p2;




create table branch(
branch_id varchar(10) primary key, 
manager_id varchar (10),
branch_address  varchar(20),
contact_no varchar(20)
);

drop table  IF exists employees;

create table employees (
emp_id varchar(10) primary key ,
emp_name varchar(25),
position varchar(15),
salary int,
branch_id varchar(25)
);

create table books(
isbn varchar(25) primary key ,
book_title varchar(75),
category varchar(15),
rental_price float,
status varchar(15),
author  varchar(25),
publisher varchar(55)
);

create table Members(
member_id varchar(10) primary key ,
member_name varchar(55),
member_address varchar(75),
reg_date date
);

create table issued_status(
issued_id varchar(10) primary key ,
issued_member_id varchar(10),
issued_book_name varchar(75),
issued_date date,
issued_book_isbn varchar(25),
issued_emp_id varchar(10)

);



create table return_status(
return_id varchar(10) primary key ,
issued_id varchar(10),
return_book_name varchar(75),
return_date date,
return_book_isbn varchar(25)

);


**-- foreign key**
alter table issued_status
add constraint fk_members
 foreign key (issued_member_id)
 references Members(member_id);
 
 alter table issued_status
add constraint fk_books
 foreign key (issued_book_isbn)
 references books(isbn);
 
 alter table issued_status
add constraint fk_empoyees
 foreign key (issued_emp_id)
 references employees (
emp_id);

alter table employees
add constraint fk_branch
 foreign key (branch_id)
 references branch (
branch_id);
 
 alter table return_status
add constraint fk_issued_status
 foreign key (issued_id)
 references issued_status (
issued_id);

**performing CRUD operations**


**-- Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird',
--  'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')**

;
insert into books ( isbn ,
book_title,
category ,	
rental_price ,
status ,
author,
publisher  ) values ( '978-1-60129-456-2', 'To Kill a Mockingbird',
'Classic', 6.00 , 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');




**-- Task 2: Update an Existing Member's Address**


update Members
set member_address ='bangalore data street 786'
where member_id= 'c101';

**-- Task 3: Delete a Record from the Issued Status Table --
--  Objective: Delete the record with issued_id = 'IS121' from the issued_status table.**

delete from  Issued_Status
where issued_id = 'IS107';



**-- Task 4: Retrieve All Books Issued by a Specific Employee 
-- -- Objective: Select all books issued by the employee with emp_id = 'E1**01'.


select issued_book_name from issued_status
where issued_emp_id = 'E101';


**-- Task 5: List Members Who Have Issued More Than One Book
--  -- Objective: Use GROUP BY to find members who have issued more than one book.**


select
    issued_emp_id,
    COUNT(*)
from  issued_status
group by issued_emp_id
Having COUNT(*) > 1;


**-- Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results
--   each book and total book_issued_cnt****


create table books_issues
as(
select books.isbn,books.book_title, count(issued_status.issued_id) as total_book_issued from books 
join issued_status on  issued_status.issued_book_isbn=books.isbn
group by 1,2) ;

**Task 7. Retrieve All Books in a Specific Category:**

select *  from books
where category='classic';


**Task 8 By each category retrive total no. of books**


select books.category, count(books.isbn) from books
group by 1;


**Task 9: Find Total Rental Income by Category:**


select books.category, (sum(books.rental_price)*count(issued_status.issued_id)) as income_by_category  from books
join issued_status on issued_status.issued_book_isbn =books.isbn
group by books.category
order by income_by_category desc;



**Task 10.List Members Who Registered in the Last 180 Days:**

insert into members(member_id,member_name, member_address, reg_date) values 
('C786','Gourav jain','india by anaylst','2024-10-18');

SELECT * FROM members
WHERE reg_date >= DATE_SUB(CURDATE(), INTERVAL 180 DAY);

-- List Employees with Their Branch Manager's Name and their branch details:

 select e1.*,e2.emp_name as manager ,e2.emp_id as manger_id from employees as e1
 join branch on branch.branch_id=e1.branch_id
 join employees  as e2
 on branch.manager_id = e2.emp_id
;

**Task 11. Create a Table of Books with Rental Price Above a Certain Threshold:**

create table books_rental_Threshold as
select * from books
where rental_price > 7.00;



**Task 12: Retrieve the List of Books Not Yet Returned**

select issued_status.issued_id,return_status.return_id,  issued_status.issued_book_name from issued_status
left join return_status on issued_status.issued_id=return_status.issued_id
where return_status.return_id is null ;


**-- Task 13: Identify Members with Overdue Books**

**-- Write a query to identify members who have overdue books (assume a 30-day return period).
--  Display the member's_id, member's name, book title, issue date, and days overdue.**



select members.member_id, members.member_name, books.book_title,issued_status.issued_date,
 current_date()-issued_status.issued_date as over_dues_days
from issued_status 
join members on issued_status.issued_member_id=members.member_id
join books on books.isbn= issued_status.issued_book_isbn
left join return_status  on return_status.issued_id=issued_status.issued_id
where return_status.return_date is null  And  current_date()-issued_status.issued_date  > 30
order by 1;



**Executing advanced SQL**

**-- Task 14: Update Book Status on Return
-- Write a query to update the status of books in the books table to "Yes" 
-- when they are returned (based on entries in the return_status table).**

-- Manual process refernces 

select *from  issued_status
where issued_book_isbn='978-0-451-52994-2';

select * from books
where isbn='978-0-451-52994-2';

update books
set status='no'
where isbn='978-0-451-52994-2';

select * from return_status
where issued_id='IS130';

**---
-- if returned add entry into Return table**

insert into return_status (return_id, issued_id, return_date)
values( 'RS169',' IS130',current_date());

select *from  return_status 
where issued_id='IS130';

update books
set status='yes'
where isbn='978-0-451-52994-2';


DELIMITER 
//

CREATE PROCEDURE or replace book_status_record(IN p_return_id varchar(10), IN p_issued_id varchar(10), IN p_return_date date )
BEGIN
    DECLARE  v_isbn varchar(50);
    DECLARE  v_book_name varchar(80);
    
    insert into return_status (return_id, issued_id, return_date)
	values( P_return_id , P_issued_id , CURDATE() );

    
    SELECT issued_book_isbn, issued_book_name 
    INTO v_isbn , v_book_name
    FROM issued_status
    WHERE issued_id = P_issued_id;
    
    update books
    set status='yes'
    where isbn=v_isbn;
    
	SELECT CONCAT('Thank you for returning the book: ', v_book_name) AS return_message;

END 
//

DELIMITER ;

call book_status_record('RS120', 'IS135' ,CURDATE());



**-- Task 15: Branch Performance Report
-- Create a query that generates a performance report for each branch, 
-- showing the number of books issued, the number of books returned, and the total revenue generated from book rental**



create table  bracnh_report as

select  branch.branch_id, branch.manager_id, count(issued_status.issued_id), count(return_status.return_id), sum(books.rental_price)
from issued_status join employees on issued_status.issued_emp_id=employees.emp_id join branch on employees.branch_id=branch.branch_id
left join  return_status on  return_status.issued_id=issued_status.issued_id 
join books on books.isbn=issued_status.issued_book_isbn
group by 1,2 ;

select * from branch_reports;


**Task 16: CTAS: Create a Table of Active Members
-- Use the CREATE TABLE AS (CTAS) statement to create a new table 
-- active_members containing members who have issued at least one book in the last 6 months.**


CREATE TABLE active_members
as
select  * from  members
where  member_id IN (SELECT 
                        DISTINCT issued_member_id   
                    FROM issued_status
                    WHERE 
                       issued_status.issued_date >= subdate((curdate()) , interval 7 month) 
                    )
;

SELECT * FROM active_members;

**-- Task 17: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues.
--  Display the employee name, number of books processed, and their branch.**


select employees.emp_name , branch.*, count(issued_status.issued_id), employees.branch_id from employees
join issued_status on employees.emp_id=issued_status.issued_emp_id
join branch on branch.branch_id = employees.branch_id
group by  employees.emp_name,employees.branch_id
order by 3 desc
limit 3;





**-- Task 18: Stored Procedure Objective: Create a stored procedure to manage the status of books in a library system. 
-- Description: Write a stored procedure that updates the status of a book in the library based on its issuance. 
-- The procedure should function as follows: The stored procedure should take the book_id as an input parameter. 
-- The procedure should first check if the book is available (status = 'yes').
--  If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
--  If the book is not available (status = 'no'), 
--  the procedure should return an error message indicating that the book is currently not available.**



DELIMITER //

create  procedure issue_book( in p_issued_id  varchar(10), in p_issued_member_id varchar(30)
 , in p_issued_book_isbn varchar(50) , in p_issued_emp_id varchar(10))
 
 -- variable
begin
	declare v_status varchar(10);
    
    
    select  status into v_status from books
    where isbn=p_issued_book_isbn;
    
    if  v_status='yes' then 
		insert into issued_status(issued_id,issued_member_id,issued_date,issued_book_isbn,issued_emp_id)
        values (p_issued_id,p_issued_member_id, curdate(), p_issued_book_isbn, p_issued_emp_id);
        
        update books
        set status ='no'
        where isbn= p_issued_book_isbn;
        
        
        
		SELECT CONCAT('Book records added successfully for book isbn: ', p_issued_book_isbn) AS message;
 
	else
		SELECT CONCAT('Sorry to inform you the book you have requested is unavailable book_isbn: ', p_issued_book_isbn) AS message;


	end if;
    
end //

delimiter ;

call issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
call issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8';


**Insights**






Conclusion
This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
