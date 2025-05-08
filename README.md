# PLP-week-8- DATABASE-assisgnment

-- Question1
CREATE DATABASE IF NOT EXISTS student_records_db;

USE student_records_db;
 
CREATE TABLE IF NOT EXISTS `student_records_db`.`departments` (
  `department_id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the department',
  `department_name` VARCHAR(100) NOT NULL UNIQUE COMMENT 'Name of the department',
  `head_of_department` VARCHAR(100) NULL COMMENT 'Name of the current head of department',
  PRIMARY KEY (`department_id`)
) ENGINE = InnoDB COMMENT = 'Academic departments';


CREATE TABLE IF NOT EXISTS `student_records_db`.`students` (
  `student_id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the student',
  `first_name` VARCHAR(50) NOT NULL COMMENT 'Student''s first name',
  `last_name` VARCHAR(50) NOT NULL COMMENT 'Student''s last name',
  `date_of_birth` DATE NULL COMMENT 'Student''s date of birth',
  `email` VARCHAR(100) UNIQUE NULL COMMENT 'Student''s email address',
  `enrollment_date` DATE NOT NULL DEFAULT (CURRENT_DATE()) COMMENT 'Date the student enrolled',
  `major_department_id` INT NULL COMMENT 'Foreign key referencing the student''s major department',
  PRIMARY KEY (`student_id`),
  CONSTRAINT `fk_students_departments`
    FOREIGN KEY (`major_department_id`)
    REFERENCES `student_records_db`.`departments` (`department_id`)
    ON DELETE SET NULL
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Information about students';


CREATE TABLE IF NOT EXISTS `student_records_db`.`courses` (
  `course_id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the course',
  `course_code` VARCHAR(20) NOT NULL UNIQUE COMMENT 'Unique code for the course (e.g., CS101)',
  `course_title` VARCHAR(100) NOT NULL COMMENT 'Full title of the course',
  `credits` INT NOT NULL COMMENT 'Number of credits for the course',
  `department_id` INT NULL COMMENT 'Foreign key referencing the department offering the course',
  PRIMARY KEY (`course_id`),
  CONSTRAINT `fk_courses_departments`
    FOREIGN KEY (`department_id`)
    REFERENCES `student_records_db`.`departments` (`department_id`)
    ON DELETE SET NULL
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Academic courses';


CREATE TABLE IF NOT EXISTS `student_records_db`.`instructors` (
  `instructor_id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the instructor',
  `first_name` VARCHAR(50) NOT NULL COMMENT 'Instructor''s first name',
  `last_name` VARCHAR(50) NOT NULL COMMENT 'Instructor''s last name',
  `email` VARCHAR(100) UNIQUE NULL COMMENT 'Instructor''s email address',
  `department_id` INT NULL COMMENT 'Foreign key referencing the instructor''s department',
  PRIMARY KEY (`instructor_id`),
  CONSTRAINT `fk_instructors_departments`
    FOREIGN KEY (`department_id`)
    REFERENCES `student_records_db`.`departments` (`department_id`)
    ON DELETE SET NULL
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Information about instructors';


CREATE TABLE IF NOT EXISTS `student_records_db`.`enrollments` (
  `enrollment_id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the enrollment record',
  `student_id` INT NOT NULL COMMENT 'Foreign key referencing the student',
  `course_id` INT NOT NULL COMMENT 'Foreign key referencing the course',
  `enrollment_date` DATE NOT NULL DEFAULT (CURRENT_DATE()) COMMENT 'Date of enrollment',
  `grade` VARCHAR(5) NULL COMMENT 'Grade received in the course (e.g., A, B+, Pass)',
  PRIMARY KEY (`enrollment_id`),
  UNIQUE INDEX `uq_student_course` (`student_id` ASC, `course_id` ASC) COMMENT 'Ensures a student is enrolled in a course only once',
  INDEX `fk_enrollments_courses1_idx` (`course_id` ASC),
  CONSTRAINT `fk_enrollments_students1`
    FOREIGN KEY (`student_id`)
    REFERENCES `student_records_db`.`students` (`student_id`)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT `fk_enrollments_courses1`
    FOREIGN KEY (`course_id`)
    REFERENCES `student_records_db`.`courses` (`course_id`)
    ON DELETE CASCADE
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Records of student enrollments in courses';



CREATE TABLE IF NOT EXISTS `student_records_db`.`course_offerings` (
  `offering_id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the course offering',
  `course_id` INT NOT NULL COMMENT 'Foreign key referencing the course being offered',
  `instructor_id` INT NOT NULL COMMENT 'Foreign key referencing the instructor teaching the course',
  `academic_year` YEAR NOT NULL COMMENT 'Academic year the course was offered (e.g., 2023)',
  `semester` VARCHAR(20) NOT NULL COMMENT 'Semester the course was offered (e.g., Fall, Spring)',
  PRIMARY KEY (`offering_id`),
  UNIQUE INDEX `uq_course_instructor_year_semester` (`course_id` ASC, `instructor_id` ASC, `academic_year` ASC, `semester` ASC) COMMENT 'Ensures a unique offering by instructor, year, and semester',
  INDEX `fk_course_offerings_instructors1_idx` (`instructor_id` ASC),
  CONSTRAINT `fk_course_offerings_courses1`
    FOREIGN KEY (`course_id`)
    REFERENCES `student_records_db`.`courses` (`course_id`)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT `fk_course_offerings_instructors1`
    FOREIGN KEY (`instructor_id`)
    REFERENCES `student_records_db`.`instructors` (`instructor_id`)
    ON DELETE CASCADE
    ON UPDATE CASCADE
) ENGINE = InnoDB COMMENT = 'Records of which instructors teach which courses in which semesters';

-- Question2
CREATE DATABASE IF NOT EXISTS task_manager_db;

USE task_manager_db;

CREATE TABLE IF NOT EXISTS `task_manager_db`.`tasks` (
  `id` INT NOT NULL AUTO_INCREMENT COMMENT 'Unique identifier for the task',
  `title` VARCHAR(100) NOT NULL COMMENT 'Title of the task',
  `description` TEXT NULL COMMENT 'Detailed description of the task',
  `status` ENUM('pending', 'in_progress', 'completed') NOT NULL DEFAULT 'pending' COMMENT 'Current status of the task',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Timestamp when the task was created',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Timestamp when the task was last updated',
  PRIMARY KEY (`id`)
) ENGINE = InnoDB COMMENT = 'Information about tasks';

import mysql.connector
from mysql.connector import Error

# Replace with your MySQL credentials and database name
DB_HOST = "localhost"
DB_USER = "your_mysql_user"
DB_PASSWORD = "your_mysql_password"
DB_NAME = "task_manager_db"

def get_db_connection():
    """Creates and returns a database connection."""
    try:
        connection = mysql.connector.connect(
            host=DB_HOST,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASSWORD
        )
        if connection.is_connected():
            print("Successfully connected to MySQL database")
            return connection
    except Error as e:
        print(f"Error connecting to MySQL database: {e}")
        return None

def close_db_connection(connection):
    """Closes the database connection."""
    if connection and connection.is_connected():
        connection.close()
        print("MySQL connection closed")
        
        from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.responses import JSONResponse
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

from database import get_db_connection, close_db_connection
import mysql.connector

app = FastAPI()

class TaskBase(BaseModel):
    title: str
    description: Optional[str] = None
    status: str = "pending" # Default status

class TaskCreate(TaskBase):
    pass

class Task(TaskBase):
    id: int
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True # Allow ORM model attributes to be accessed

def get_db():
    db = get_db_connection()
    try:
        yield db
    finally:
        close_db_connection(db)


@app.post("/tasks/", response_model=Task, status_code=status.HTTP_201_CREATED)
def create_task(task: TaskCreate, db: mysql.connector.MySQLConnection = Depends(get_db)):
    if db is None:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Database connection error")

    cursor = db.cursor(dictionary=True)
    try:
        sql = "INSERT INTO tasks (title, description, status) VALUES (%s, %s, %s)"
        values = (task.title, task.description, task.status)
        cursor.execute(sql, values)
        db.commit()

        task_id = cursor.lastrowid

        cursor.execute("SELECT * FROM tasks WHERE id = %s", (task_id,))
        created_task = cursor.fetchone()

        if created_task:
            return created_task
        else:
            raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Failed to fetch created task")

    except mysql.connector.Error as err:
        db.rollback()
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Database error: {err}")
    finally:
        cursor.close()

# Read all tasks
@app.get("/tasks/", response_model=List[Task])
def read_tasks(db: mysql.connector.MySQLConnection = Depends(get_db)):
    if db is None:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Database connection error")

    cursor = db.cursor(dictionary=True)
    try:
        cursor.execute("SELECT * FROM tasks")
        tasks = cursor.fetchall()
        return tasks
    except mysql.connector.Error as err:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Database error: {err}")
    finally:
        cursor.close()

# Read a single task by ID
@app.get("/tasks/{task_id}", response_model=Task)
def read_task(task_id: int, db: mysql.connector.MySQLConnection = Depends(get_db)):
    if db is None:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Database connection error")

    cursor = db.cursor(dictionary=True)
    try:
        cursor.execute("SELECT * FROM tasks WHERE id = %s", (task_id,))
        task = cursor.fetchone()

        if task is None:
            raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Task not found")

        return task
    except mysql.connector.Error as err:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Database error: {err}")
    finally:
        cursor.close()

@app.put("/tasks/{task_id}", response_model=Task)
def update_task(task_id: int, task: TaskCreate, db: mysql.connector.MySQLConnection = Depends(get_db)):
    if db is None:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Database connection error")

    cursor = db.cursor()
    try:
        # Check if the task exists first
        cursor.execute("SELECT id FROM tasks WHERE id = %s", (task_id,))
        existing_task = cursor.fetchone()
        if existing_task is None:
            raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Task not found")

        sql = "UPDATE tasks SET title = %s, description = %s, status = %s WHERE id = %s"
        values = (task.title, task.description, task.status, task_id)
        cursor.execute(sql, values)
        db.commit()

        cursor.execute("SELECT * FROM tasks WHERE id = %s", (task_id,))
        updated_task = cursor.fetchone() # Use fetchone() after getting the updated row

        if updated_task:
             # Convert tuple to dictionary for Pydantic model
            updated_task_dict = {
                "id": updated_task[0],
                "title": updated_task[1],
                "description": updated_task[2],
                "status": updated_task[3],
                "created_at": updated_task[4],
                "updated_at": updated_task[5],
            }
            return updated_task_dict
        else:
             # This case should ideally not happen if commit was successful and task existed
             raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Failed to fetch updated task")


    except mysql.connector.Error as err:
        db.rollback()
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Database error: {err}")
    finally:
        cursor.close()

# Delete a task by ID
@app.delete("/tasks/{task_id}", status_code=status.HTTP_200_OK)
def delete_task(task_id: int, db: mysql.connector.MySQLConnection = Depends(get_db)):
    if db is None:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Database connection error")

    cursor = db.cursor()
    try:
        sql = "DELETE FROM tasks WHERE id = %s"
        cursor.execute(sql, (task_id,))
        db.commit()

        if cursor.rowcount == 0:
            raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Task not found")

        return {"message": "Task deleted successfully"}

    except mysql.connector.Error as err:
        db.rollback()
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Database error: {err}")
    finally:
        cursor.close()

@app.get("/")
def read_root():
    return {"message": "Task Manager API is running!"}
