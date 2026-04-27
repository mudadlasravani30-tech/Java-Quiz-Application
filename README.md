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
CREATE TABLE questions(
  id INTEGER PRIMARY KEY,
  topic TEXT,
  question TEXT,
  option1 TEXT,
  option2 TEXT,
  option3 TEXT,
  option4 TEXT,
  correct_option INTEGER
);
CREATE TABLE results(
  id INTEGER PRIMARY KEY,
  username TEXT,
  topic TEXT,
  score INTEGER
);
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
Step 3: Database Connection
import java.sql.*;
public class DBConnection {
    public static Connection connect() {
        try {
            Class.forName("org.sqlite.JDBC");
            return DriverManager.getConnection("jdbc:sqlite:quiz.db");
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
Step 4: Model Class
public class Question {
    int id;
    String question, topic;
    String[] options = new String[4];
    int correctOption;
    public Question(int id, String topic, String question, String[] options, int correctOption) {
        this.id = id;
        this.topic = topic;
        this.question = question;
        this.options = options;
        this.correctOption = correctOption;
    }
}
Step 5: GUI Implementation (QuizFrame.java)
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.*;
public class QuizFrame extends JFrame {
    ArrayList<Question> questions;
    int index = 0, score = 0;
    JLabel questionLabel;
    JRadioButton[] opts = new JRadioButton[4];
    ButtonGroup bg = new ButtonGroup();
    JButton nextBtn, submitBtn;
    String topic, username;
    public QuizFrame(String topic, String username) {
        this.topic = topic;
        this.username = username;
        setTitle("Quiz - " + topic);
        setLayout(new BorderLayout());
        setSize(600, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        questionLabel = new JLabel("", JLabel.CENTER);
        add(questionLabel, BorderLayout.NORTH);
        JPanel optionsPanel = new JPanel(new GridLayout(4, 1));
        for (int i = 0; i < 4; i++) {
            opts[i] = new JRadioButton();
            bg.add(opts[i]);
            optionsPanel.add(opts[i]);
        }
        add(optionsPanel, BorderLayout.CENTER);
        JPanel bottom = new JPanel();
        nextBtn = new JButton("Next");
        submitBtn = new JButton("Submit");
        bottom.add(nextBtn);
        bottom.add(submitBtn);
        add(bottom, BorderLayout.SOUTH);
        loadQuestions();
        displayQuestion(0);
        nextBtn.addActionListener(e -> nextQuestion());
        submitBtn.addActionListener(e -> submitQuiz());
        setVisible(true);
    }
    void loadQuestions() {
        questions = new ArrayList<>();
        try (Connection con = DBConnection.connect();
             PreparedStatement ps = con.prepareStatement(
                 "SELECT * FROM questions WHERE topic=? LIMIT 20")) {
            ps.setString(1, topic);
            ResultSet rs = ps.executeQuery();
            while (rs.next()) {
                String[] opts = { rs.getString("option1"), rs.getString("option2"),
                                  rs.getString("option3"), rs.getString("option4") };
                questions.add(new Question(
                    rs.getInt("id"),
                    rs.getString("topic"),
                    rs.getString("question"),
                    opts,
                    rs.getInt("correct_option")
                ));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    void displayQuestion(int i) {
        Question q = questions.get(i);
        questionLabel.setText("Q" + (i + 1) + ": " + q.question);
        bg.clearSelection();
        for (int j = 0; j < 4; j++) {
            opts[j].setText(q.options[j]);
        }
    }
    void nextQuestion() {
        checkAnswer();
        if (++index < questions.size()) {
            displayQuestion(index);
        } else {
            submitQuiz();
        }
    }
    void checkAnswer() {
        int selected = -1;
        for (int i = 0; i < 4; i++) {
            if (opts[i].isSelected()) selected = i + 1;
        }
        if (selected == questions.get(index).correctOption) score++;
    }
    void submitQuiz() {
        try (Connection con = DBConnection.connect();
             PreparedStatement ps = con.prepareStatement(
                 "INSERT INTO results(username, topic, score) VALUES(?,?,?)")) {
            ps.setString(1, username);
            ps.setString(2, topic);
            ps.setInt(3, score);
            ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
        new ResultFrame(username, topic, score, questions.size());
        dispose();
    }
}
Step 6: Result Display
import javax.swing.*;
public class ResultFrame extends JFrame {
    public ResultFrame(String username, String topic, int score, int total) {
        setTitle("Quiz Result");
        setSize(400, 200);
        setLayout(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        JLabel lbl = new JLabel("<html><center>" +
                "User: " + username +
                "<br>Topic: " + topic +
                "<br>Score: " + score + " / " + total +
                "</center></html>", SwingConstants.CENTER);
        lbl.setBounds(50, 30, 300, 100);
        add(lbl);
        setVisible(true);
    }
}
Step 7: Start the App
import javax.swing.*;
public class QuizApp {
    public static void main(String[] args) {
        String user = JOptionPane.showInputDialog("Enter your name:");
        String[] topics = {"Python", "C"};
        String topic = (String) JOptionPane.showInputDialog(
            null, "Choose Topic:", "Select Topic",
            JOptionPane.QUESTION_MESSAGE, null, topics, topics[0]
        );
        new QuizFrame(topic, user);
    }
}
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
•	medium.com 
•	dev.to
Would you like me to include the SQL script with 20 ready-made questions (10 for Python & 10 for C) so you can populate your database instantly?
Here’s a detailed project documentation and step-by-step guide on building a Mini Quiz Application in Java that covers topics like Basics of Python and C Programming — with 15–20 questions per topic, using Java Swing for GUI and SQLite for question storage. 
📘 Mini Project Documentation
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
CREATE TABLE questions(
  id INTEGER PRIMARY KEY,
  topic TEXT,
  question TEXT,
  option1 TEXT,
  option2 TEXT,
  option3 TEXT,
  option4 TEXT,
  correct_option INTEGER
);
CREATE TABLE results(
  id INTEGER PRIMARY KEY,
  username TEXT,
  topic TEXT,
  score INTEGER
);
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
Step 3: Database Connection
import java.sql.*;
public class DBConnection {
    public static Connection connect() {
        try {
            Class.forName("org.sqlite.JDBC");
            return DriverManager.getConnection("jdbc:sqlite:quiz.db");
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
Step 4: Model Class
public class Question {
    int id;
    String question, topic;
    String[] options = new String[4];
    int correctOption;
    public Question(int id, String topic, String question, String[] options, int correctOption) {
        this.id = id;
        this.topic = topic;
        this.question = question;
        this.options = options;
        this.correctOption = correctOption;
    }
}
Step 5: GUI Implementation (QuizFrame.java)
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.*;
public class QuizFrame extends JFrame {
    ArrayList<Question> questions;
    int index = 0, score = 0;
    JLabel questionLabel;
    JRadioButton[] opts = new JRadioButton[4];
    ButtonGroup bg = new ButtonGroup();
    JButton nextBtn, submitBtn;
    String topic, username;
    public QuizFrame(String topic, String username) {
        this.topic = topic;
        this.username = username;
        setTitle("Quiz - " + topic);
        setLayout(new BorderLayout());
        setSize(600, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        questionLabel = new JLabel("", JLabel.CENTER);
        add(questionLabel, BorderLayout.NORTH);
        JPanel optionsPanel = new JPanel(new GridLayout(4, 1));
        for (int i = 0; i < 4; i++) {
            opts[i] = new JRadioButton();
            bg.add(opts[i]);
            optionsPanel.add(opts[i]);
        }
        add(optionsPanel, BorderLayout.CENTER);
        JPanel bottom = new JPanel();
        nextBtn = new JButton("Next");
        submitBtn = new JButton("Submit");
        bottom.add(nextBtn);
        bottom.add(submitBtn);
        add(bottom, BorderLayout.SOUTH);
        loadQuestions();
        displayQuestion(0);
        nextBtn.addActionListener(e -> nextQuestion());
        submitBtn.addActionListener(e -> submitQuiz());
        setVisible(true);
    }
    void loadQuestions() {
        questions = new ArrayList<>();
        try (Connection con = DBConnection.connect();
             PreparedStatement ps = con.prepareStatement(
                 "SELECT * FROM questions WHERE topic=? LIMIT 20")) {
            ps.setString(1, topic);
            ResultSet rs = ps.executeQuery();
            while (rs.next()) {
                String[] opts = { rs.getString("option1"), rs.getString("option2"),
                                  rs.getString("option3"), rs.getString("option4") };
                questions.add(new Question(
                    rs.getInt("id"),
                    rs.getString("topic"),
                    rs.getString("question"),
                    opts,
                    rs.getInt("correct_option")
                ));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    void displayQuestion(int i) {
        Question q = questions.get(i);
        questionLabel.setText("Q" + (i + 1) + ": " + q.question);
        bg.clearSelection();
        for (int j = 0; j < 4; j++) {
            opts[j].setText(q.options[j]);
        }
    }
    void nextQuestion() {
        checkAnswer();
        if (++index < questions.size()) {
            displayQuestion(index);
        } else {
            submitQuiz();
        }
    }
    void checkAnswer() {
        int selected = -1;
        for (int i = 0; i < 4; i++) {
            if (opts[i].isSelected()) selected = i + 1;
        }
        if (selected == questions.get(index).correctOption) score++;
    }
    void submitQuiz() {
        try (Connection con = DBConnection.connect();
             PreparedStatement ps = con.prepareStatement(
                 "INSERT INTO results(username, topic, score) VALUES(?,?,?)")) {
            ps.setString(1, username);
            ps.setString(2, topic);
            ps.setInt(3, score);
            ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
        new ResultFrame(username, topic, score, questions.size());
        dispose();
    }
}
Step 6: Result Display
import javax.swing.*;
public class ResultFrame extends JFrame {
    public ResultFrame(String username, String topic, int score, int total) {
        setTitle("Quiz Result");
        setSize(400, 200);
        setLayout(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        JLabel lbl = new JLabel("<html><center>" +
                "User: " + username +
                "<br>Topic: " + topic +
                "<br>Score: " + score + " / " + total +
                "</center></html>", SwingConstants.CENTER);
        lbl.setBounds(50, 30, 300, 100);
        add(lbl);
        setVisible(true);
    }
}
Step 7: Start the App
import javax.swing.*;
public class QuizApp {
    public static void main(String[] args) {
        String user = JOptionPane.showInputDialog("Enter your name:");
        String[] topics = {"Python", "C"};
        String topic = (String) JOptionPane.showInputDialog(
            null, "Choose Topic:", "Select Topic",
            JOptionPane.QUESTION_MESSAGE, null, topics, topics[0]
        );
        new QuizFrame(topic, user);
    }
}
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
•	medium.com 
•	dev.to



