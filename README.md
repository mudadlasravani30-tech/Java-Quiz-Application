Project Documentation
Project Title:
Quiz Application using Java (Topics: Python & C Programming) 
Objective:
To develop a Java-based desktop quiz system that allows users to choose a topic (Python or C), answer a limited set of multiple-choice questions (15–20 per topic), and automatically receive a score at the end. 
1. Problem Statement
Traditional quiz systems need manual grading or constant internet connectivity.
This project aims to:
•	Provide an offline, GUI-based quiz platform. 
•	Include topic selection (Python or C). 
•	Present 15–20 randomly selected questions per topic. 
•	Show automatic scoring and feedback once completed. 
•	Save user scores in an SQLite database for reference.
2. Technology Stack
Component	Technology Used
Programming Language	Java
GUI Framework	Java Swing
Database	SQLite (sqlite-jdbc)
IDE	Eclipse / IntelliJ / VS Code
Version Control	GitHub (optional)
3. Application Structure
Layers:
1.	GUI Layer: Swing-based interface for users. 
2.	Logic Layer: Handles question loading, navigation, and scoring. 
3.	Database Layer: Stores questions and results using SQLite and JDBC.
4. Database Design
questions Table
Column	Type	Description
id	INTEGER PRIMARY KEY	
topic	TEXT	'Python' or 'C'
question	TEXT	The quiz question
option1	TEXT	Option A
option2	TEXT	Option B
option3	TEXT	Option C
option4	TEXT	Option D
correct_option	INTEGER	Correct option index (1–4)
results Table
Column	Type	Description
id	INTEGER PRIMARY KEY	
username	TEXT	Student’s name
topic	TEXT	Quiz topic
score	INTEGER	Final score
5. Workflow
Diagram:
graph TD
A[Start Application] --> B[Enter Username]
B --> C[Select Topic: Python / C]
C --> D[Load 15–20 Questions from DB]
D --> E[Display Question + 4 Options]
E --> F[Next / Back Buttons]
F --> G[Submit Quiz]
G --> H[Calculate Score]
H --> I[Display Result + Save to DB]
I --> J[Exit or Restart]
6. Implementation Steps
Step 1: Create Database
Use SQLite with two tables (questions and results).
Populate Python and C questions manually using a .sql script or via Java insert statements.

Example Data (Python)
INSERT INTO questions (topic, question, option1, option2, option3, option4, correct_option)
VALUES 
('Python', 'Who developed Python?', 'James Gosling', 'Guido van Rossum', 'Dennis Ritchie', 'Bjarne Stroustrup', 2),
('Python', 'What is the file extension of Python files?', '.java', '.py', '.c', '.txt', 2);
Step 2: Java Class Structure
src/
│
├── QuizApp.java            → main class
├── DBConnection.java        → handles SQLite connection
├── Question.java            → model class
├── QuizFrame.java           → GUI for quiz
└── ResultFrame.java         → shows final score        
            
7. Sample Output
GUI Flow
1.	User enters name → selects “Python”. 
2.	Questions appear one by one with “Next” and “Submit” buttons. 
3.	After last question → “Your Score: 17 / 20” shown. 
4.	Result stored in the results table.
8. Enhancements (Optional)
•	Add Timer (15 seconds/question). 
•	Create Admin Panel for adding questions (projectgurukul.org). 
•	Enable review of previous results. 
•	Randomized question order.
9. References
•	projectgurukul.org 
•	codewithcurious.com 
•	medium.
