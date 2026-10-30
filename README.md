# PLSQL_Assignment4_20252SEN361_20252SEN280

## 1. Introduction

The Student Registration and Grading System is a database application developed using Oracle PL/SQL. Its purpose is to manage student information, courses, enrollments and academic marks.


## 2. Database Design

The Database have 4 tables and their information:

## Students Table

Student table is created using this codes:

CREATE TABLE students (
    student_id NUMBER PRIMARY KEY,
    student_name VARCHAR2(100) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    department_id NUMBER,
    date_registered DATE DEFAULT SYSDATE
);

The table was created and insert into some information this is the output:

<img width="478" height="326" alt="Students and element" src="https://github.com/user-attachments/assets/8c34137d-d814-4f4e-ba5a-88652c08f05f" />



## Department Table

Department table is created using this codes:


CREATE TABLE departments (
    department_id NUMBER PRIMARY KEY,
    department_name VARCHAR2(100) NOT NULL
);


The table is created and inserted into some information this is the output:


<img width="472" height="331" alt="Department and elements" src="https://github.com/user-attachments/assets/39a9776a-8031-4ba2-9f51-7230c70ac355" />



## Courses Table

Courses table is created using this codes:

CREATE TABLE courses (
    course_id NUMBER PRIMARY KEY,
    course_name VARCHAR2(100) NOT NULL,
    credits NUMBER NOT NULL
);


The table is created and inserted into some info this is the output:


<img width="478" height="316" alt="courses and element" src="https://github.com/user-attachments/assets/5a325591-8bd0-4282-8da7-25c0c4be3866" />



## Enrollment table 

The Enrollment table is created using this codes:

CREATE TABLE enrollments (
    enrollment_id NUMBER PRIMARY KEY,
    student_id NUMBER,
    course_id NUMBER,
    marks NUMBER(5,2),

    CONSTRAINT fk_student
    FOREIGN KEY (student_id)
    REFERENCES students(student_id),

    CONSTRAINT fk_course
    FOREIGN KEY (course_id)
    REFERENCES courses(course_id),

    CONSTRAINT unique_enrollment
    UNIQUE (student_id, course_id),

    CONSTRAINT valid_marks
    CHECK (marks BETWEEN 0 AND 100)
);


The table is also inserted with some information this is the output:


<img width="472" height="331" alt="Department and elements" src="https://github.com/user-attachments/assets/d44d36b3-c1dc-42f7-87d0-0f02fee1c0ff" />



## Create a procedure that registers a student using a user-defined record

We used this codes :

CREATE OR REPLACE PROCEDURE register_student (
    p_id NUMBER,
    p_name VARCHAR2,
    p_email VARCHAR2,
    p_department NUMBER
)
IS
    TYPE student_record IS RECORD (    // creates our own record structure.
        student_id NUMBER,
        student_name VARCHAR2(100),
        email VARCHAR2(100),
        department_id NUMBER
    );

    v_student student_record;  // declares a variable using that structure.

BEGIN
    -- Store information in the record
    v_student.student_id := p_id;
    v_student.student_name := p_name;
    v_student.email := p_email;
    v_student.department_id := p_department;

    -- Insert record information into the table
    INSERT INTO students
    (student_id, student_name, email, department_id)
    VALUES (
        v_student.student_id,
        v_student.student_name,
        v_student.email,
        v_student.department_id
    );

    DBMS_OUTPUT.PUT_LINE('Student registered successfully.');

EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student ID or email already exists.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


We registed one student this is the output


<img width="478" height="329" alt="Student registrated" src="https://github.com/user-attachments/assets/bfc3f846-e3df-4e08-a297-c11221d8b28c" />



## create a procedure display student using %ROWTYPE

We used this code:

CREATE OR REPLACE PROCEDURE display_student (
    p_student_id NUMBER
)
IS
    v_student students%ROWTYPE;

BEGIN
    SELECT *
    INTO v_student
    FROM students
    WHERE student_id = p_student_id;

    DBMS_OUTPUT.PUT_LINE('Student ID: ' || v_student.student_id);
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_student.student_name);
    DBMS_OUTPUT.PUT_LINE('Email: ' || v_student.email);
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_student.department_id);

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Student not found.');

    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Multiple students found.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


This is the output of test:


<img width="466" height="344" alt="Display workig proc" src="https://github.com/user-attachments/assets/31d289ab-5dc7-4c07-918c-41124a63f3ff" />



Why do we use %ROWTYPE?

Because it automatically inherits all the columns and data types from the students table.



## Implement collection using Nested table


Objective: Store multiple student marks and calculate their average.

Codes used :

CREATE OR REPLACE PROCEDURE calculate_marks
IS
    TYPE marks_table IS TABLE OF NUMBER;

    v_marks marks_table := marks_table();

    v_total NUMBER := 0;
    v_average NUMBER;

BEGIN
    -- Add elements
    v_marks.EXTEND(5);

    v_marks(1) := 85;
    v_marks(2) := 75;
    v_marks(3) := 90;
    v_marks(4) := 80;
    v_marks(5) := 95;

    -- Calculate total
    FOR i IN 1..v_marks.COUNT LOOP
        v_total := v_total + v_marks(i);
    END LOOP;

    v_average := v_total / v_marks.COUNT;

    DBMS_OUTPUT.PUT_LINE('Total Marks: ' || v_total);
    DBMS_OUTPUT.PUT_LINE('Average Marks: ' || v_average);

EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Cannot divide by zero.');

    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


The output got is :

<img width="479" height="356" alt="Calculating marks collection" src="https://github.com/user-attachments/assets/0719d555-99f0-4dbd-b220-37c98de6ab97" />



## Display student names using Association Array

Objective: Store student names using their IDs as indexes.


The codes  used :

CREATE OR REPLACE PROCEDURE display_student_names
IS
    TYPE student_array IS TABLE OF VARCHAR2(100)
    INDEX BY PLS_INTEGER;

    v_students student_array;

BEGIN
    v_students(101) := 'Alice';
    v_students(102) := 'John';
    v_students(103) := 'Peter';
    v_students(104) := 'Mary';

    DBMS_OUTPUT.PUT_LINE('Student 101: ' || v_students(101));
    DBMS_OUTPUT.PUT_LINE('Student 102: ' || v_students(102));
    DBMS_OUTPUT.PUT_LINE('Student 103: ' || v_students(103));
    DBMS_OUTPUT.PUT_LINE('Student 104: ' || v_students(104));

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


The output got is :

<img width="475" height="341" alt="Display names association" src="https://github.com/user-attachments/assets/6c1aac84-4997-4e78-bb76-64e0d40d2845" />




## Implement IMPLICIT CURSORS

We are going to Create a procedure that updates a student's email 

codes used:

CREATE OR REPLACE PROCEDURE update_student_email (
    p_id NUMBER,
    p_email VARCHAR2
)
IS
BEGIN
    UPDATE students
    SET email = p_email
    WHERE student_id = p_id;

    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Student updated successfully.');
        DBMS_OUTPUT.PUT_LINE('Rows affected: ' || SQL%ROWCOUNT);
    ELSE
        DBMS_OUTPUT.PUT_LINE('Student not found.');
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;

The output got:

<img width="481" height="333" alt="Update student emails implicit cursor" src="https://github.com/user-attachments/assets/b4c7a33f-904a-4c8e-8c4f-fccdf78ea6aa" />



We are going to create a procedure that delete a student

we used this codes:

CREATE OR REPLACE PROCEDURE delete_student (
    p_id NUMBER
)
IS
BEGIN
    DELETE FROM students
    WHERE student_id = p_id;

    IF SQL%NOTFOUND THEN
        DBMS_OUTPUT.PUT_LINE('Student not found.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Student deleted successfully.');
        DBMS_OUTPUT.PUT_LINE('Rows deleted: ' || SQL%ROWCOUNT);
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;

The output got:


<img width="482" height="343" alt="Delete student implicit cursor" src="https://github.com/user-attachments/assets/d3a5fbcd-588f-4435-929d-fb77dd4fdaa9" />


## Implement Explicit cursors

we created a procedure display all students using explicit cursors 

using this codes:

CREATE OR REPLACE PROCEDURE display_all_students
IS
    CURSOR student_cursor IS
        SELECT student_id, student_name, email
        FROM students;

    v_student student_cursor%ROWTYPE;

BEGIN
    OPEN student_cursor;

    LOOP
        FETCH student_cursor INTO v_student;

        EXIT WHEN student_cursor%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(
            'ID: ' || v_student.student_id ||
            ' Name: ' || v_student.student_name ||
            ' Email: ' || v_student.email
        );

    END LOOP;

    CLOSE student_cursor;

EXCEPTION
    WHEN OTHERS THEN
        IF student_cursor%ISOPEN THEN
            CLOSE student_cursor;
        END IF;

        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


The output got is :

<img width="707" height="398" alt="display all explicit" src="https://github.com/user-attachments/assets/bab80ab0-ea39-4f53-99f9-478737aab05f" />



## Implement Bulk collect and forall


BULK COLLECT retrieves multiple database rows into a collection.

codes used to show Bulk collect:

CREATE OR REPLACE PROCEDURE get_all_students_bulk
IS
    TYPE student_table IS TABLE OF students%ROWTYPE;

    v_students student_table;

BEGIN
    SELECT *
    BULK COLLECT INTO v_students
    FROM students;

    DBMS_OUTPUT.PUT_LINE(
        'Total students: ' || v_students.COUNT
    );

    FOR i IN 1..v_students.COUNT LOOP

        DBMS_OUTPUT.PUT_LINE(
            v_students(i).student_id || ' - ' ||
            v_students(i).student_name
        );

    END LOOP;

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


The Output got is :

<img width="479" height="367" alt="Collect bulk" src="https://github.com/user-attachments/assets/c87acd11-e419-4042-8273-5f9f6688b0f5" />



FORALL performs multiple DML operations using collection elements.

Objective:  we will use FORALL to update multiple enrollment records.
We will increase the marks of all students by 2 points.

Codes used are :

CREATE OR REPLACE PROCEDURE increase_all_marks
IS
    TYPE id_table IS TABLE OF NUMBER;
    TYPE marks_table IS TABLE OF NUMBER;

    v_ids id_table;
    v_marks marks_table;

BEGIN
    -- Retrieve enrollment IDs and marks
    SELECT enrollment_id, marks
    BULK COLLECT INTO v_ids, v_marks
    FROM enrollments;

    -- Increase marks in bulk
    FOR i IN 1..v_ids.COUNT LOOP
        v_marks(i) := v_marks(i) + 2;
    END LOOP;

    -- Update all records using FORALL
    FORALL i IN 1..v_ids.COUNT
        UPDATE enrollments
        SET marks = v_marks(i)
        WHERE enrollment_id = v_ids(i);

    DBMS_OUTPUT.PUT_LINE(
        'Total updated enrollments: ' || SQL%ROWCOUNT
    );

    COMMIT;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;


The output got is :

<img width="485" height="348" alt="Forall" src="https://github.com/user-attachments/assets/06c6affb-8ec9-4fc5-81ee-b31f9c0bbd9f" />



## Conclusion

The application demonstrates how Oracle PL/SQL concepts can be combined to develop a practical student management system.
## Bulk exception handling

For this we used chartGPT for codes :

DECLARE
    TYPE id_array IS TABLE OF NUMBER;

    v_ids id_array := id_array(101, 999, 102);

    e_bulk_errors EXCEPTION;

    PRAGMA EXCEPTION_INIT(e_bulk_errors, -24381);

BEGIN
    FORALL i IN 1..v_ids.COUNT SAVE EXCEPTIONS
        UPDATE students
        SET student_name = student_name || '_Updated'
        WHERE student_id = v_ids(i);

    DBMS_OUTPUT.PUT_LINE(
        'Rows updated: ' || SQL%ROWCOUNT
    );

EXCEPTION
    WHEN e_bulk_errors THEN
        DBMS_OUTPUT.PUT_LINE(
            'Some bulk operations failed.'
        );

        FOR i IN 1..SQL%BULK_EXCEPTIONS.COUNT LOOP
            DBMS_OUTPUT.PUT_LINE(
                'Failed index: ' ||
                SQL%BULK_EXCEPTIONS(i).ERROR_INDEX ||
                ' Error code: ' ||
                SQL%BULK_EXCEPTIONS(i).ERROR_CODE
            );
        END LOOP;
END;


But the Explaination is Here, the example uses a student ID that does not exist. That simply affects zero rows; it is not itself a bulk exception. The example demonstrates the bulk exception-handling structure, while actual SQL errors are reported through SQL%BULK_EXCEPTIONS.


The output got is :


<img width="482" height="362" alt="Exception bulk chartgt" src="https://github.com/user-attachments/assets/b7cd4c61-d92b-4f12-b63c-10c0383cfe41" />



## Implement STORED FUNCTIONS

Objective: Our function will calculate the average marks of a particular student.

The codes used :

CREATE OR REPLACE FUNCTION get_student_average (
    p_student_id NUMBER
)
RETURN NUMBER
IS
    v_average NUMBER;
    v_count NUMBER;

    v_student_name students.student_name%TYPE;

BEGIN
    -- Check whether the student exists
    SELECT student_name
    INTO v_student_name
    FROM students
    WHERE student_id = p_student_id;

    -- Count enrollments
    SELECT COUNT(*)
    INTO v_count
    FROM enrollments
    WHERE student_id = p_student_id;

    IF v_count = 0 THEN
        RAISE_APPLICATION_ERROR(
            -20001,
            'This student has no recorded marks.'
        );
    END IF;

    -- Calculate average
    SELECT AVG(marks)
    INTO v_average
    FROM enrollments
    WHERE student_id = p_student_id;

    RETURN v_average;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RAISE_APPLICATION_ERROR(
            -20002,
            'Student does not exist.'
        );

    WHEN OTHERS THEN
        RAISE;
END;


The output got is :

<img width="482" height="360" alt="Implement store function 2" src="https://github.com/user-attachments/assets/76807ee6-382f-490b-84b0-547b9a40ac41" />




## Implement EXCEPTION HANDLING

1. Predefined exceptions: these are already provided by oracle.

Example: 
NO_DATA_FOUND	: Used when no record was found
TOO_MANY_ROWS	: Used when more than one row returned by SELECT INTO
ZERO_DIVIDE : Used when	Dividing by zero
DUP_VAL_ON_INDEX	: Used when Duplicate primary key or unique value
OTHERS	: Used when handles other unexpected errors

2. User-defined exception

Let's use this code :

CREATE OR REPLACE PROCEDURE record_student_marks (
    p_enrollment_id NUMBER,
    p_marks NUMBER
)
IS
    e_invalid_marks EXCEPTION;

BEGIN
    -- Check marks
    IF p_marks < 0 OR p_marks > 100 THEN
        RAISE e_invalid_marks;
    END IF;

    UPDATE enrollments
    SET marks = p_marks
    WHERE enrollment_id = p_enrollment_id;

    IF SQL%NOTFOUND THEN
        RAISE_APPLICATION_ERROR(
            -20003,
            'Enrollment not found.'
        );
    END IF;

    DBMS_OUTPUT.PUT_LINE('Marks recorded successfully.');

EXCEPTION
    WHEN e_invalid_marks THEN
        RAISE_APPLICATION_ERROR(
            -20004,
            'Invalid marks. Marks must be between 0 and 100.'
        );

    WHEN OTHERS THEN
        RAISE;
END;


Output when valid marks is entered:


<img width="443" height="342" alt="Exception handling user defined" src="https://github.com/user-attachments/assets/24b04872-4f5f-473d-8c03-098eefe2f2c2" />



Output when Invalid marks is entered:


<img width="472" height="355" alt="22" src="https://github.com/user-attachments/assets/3975f266-c41d-4b65-b212-deee6549e288" />


