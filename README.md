import java.io.*;
import java.net.*;
import java.sql.*;
import java.util.*;
import javax.swing.*;
import java.awt.FlowLayout;

/* ==========================================================
   MODULE 1: CORE OOP SYSTEM (UNIT I)
   ========================================================== */

// Abstract class representing a Question
abstract class Question {
    protected String questionText;
    protected int marks;

    public Question(String questionText, int marks) {
        this.questionText = questionText;
        this.marks = marks;
    }

    public abstract boolean evaluate(String answer);
}

// Subclass for MCQ Question
class MCQQuestion extends Question {
    private String correctAnswer;

    public MCQQuestion(String questionText, int marks, String correctAnswer) {
        super(questionText, marks);
        this.correctAnswer = correctAnswer;
    }

    @Override
    public boolean evaluate(String answer) {
        return correctAnswer.equalsIgnoreCase(answer.trim());
    }
}

// Student class (Encapsulation, Constructors, Methods)
class Student {
    private String name;
    private int totalScore;

    public Student(String name) {
        this.name = name;
        this.totalScore = 0;
    }

    public void updateScore(int marks) {
        totalScore += marks;
    }

    public int getScore() {
        return totalScore;
    }

    @Override
    public String toString() {
        return "Student: " + name + ", Score: " + totalScore;
    }
}

/* ==========================================================
   MODULE 2: EXCEPTION HANDLING & FILE I/O (UNIT II)
   ========================================================== */

class InvalidAnswerException extends Exception {
    public InvalidAnswerException(String msg) {
        super(msg);
    }
}

class QuizUtility {
    public static void validateAnswer(String ans) throws InvalidAnswerException {
        if (ans == null || ans.trim().isEmpty()) {
            throw new InvalidAnswerException("Answer cannot be empty!");
        }
    }
}

// FileHandler for saving quiz results using I/O Streams
class FileHandler {
    public static void saveResult(Student s, int totalMarks) {
        try (FileWriter fw = new FileWriter("results.txt", true)) {
            fw.write(s.toString() + " out of " + totalMarks + System.lineSeparator());
        } catch (IOException e) {
            System.out.println("Error writing file: " + e.getMessage());
        }
    }
}

/* ==========================================================
   MODULE 3: GENERICS & MULTI-THREADING (UNIT III)
   ========================================================== */

class GenericList<T> {
    private List<T> items = new ArrayList<>();

    public void addItem(T item) {
        items.add(item);
    }

    public List<T> getItems() {
        return items;
    }
}

class QuizThread extends Thread {
    private Student student;
    private List<Question> questions;

    public QuizThread(Student student, List<Question> questions) {
        this.student = student;
        this.questions = questions;
    }

    @Override
    public void run() {
        for (Question q : questions) {
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println(student + " completed quiz (thread simulation).");
    }
}

/* ==========================================================
   MODULE 4: NETWORKING & JDBC (UNIT IV)
   ========================================================== */

class QuizServer {
    public static void startServer() {
        try (ServerSocket server = new ServerSocket(9999)) {
            System.out.println("Server started on port 9999");
            Socket client = server.accept();
            PrintWriter out = new PrintWriter(client.getOutputStream(), true);
            out.println("Connected to Quiz Server!");
            server.close();
        } catch (IOException e) {
            System.out.println("Server Error: " + e.getMessage());
        }
    }
}

class DBConnector {
    public static Connection connect() throws SQLException {
        String url = "jdbc:mysql://localhost:3306/quizdb";
        String user = "root";
        String pass = "password";
        return DriverManager.getConnection(url, user, pass);
    }

    public static void saveToDB(Student student, int totalMarks) {
        try (Connection con = connect()) {
            PreparedStatement ps = con.prepareStatement("INSERT INTO results(name, score) VALUES (?, ?)");
            ps.setString(1, student.toString() + " out of " + totalMarks);
            ps.setInt(2, student.getScore());
            ps.executeUpdate();
            System.out.println("Data saved to database!");
        } catch (SQLException e) {
            System.out.println("Database Error: " + e.getMessage());
        }
    }
}

/* ==========================================================
   MODULE 5: GUI PROGRAMMING (UNIT V)
   ========================================================== */

class QuizForm extends JFrame {
    JTextArea questionArea;
    JTextField answerField;
    JButton nextButton;
    JLabel scoreLabel;

    private List<Question> questions;
    private Student student;
    private int currentIndex = 0;
    private int totalMarks = 0;

    public QuizForm(Student s, List<Question> qs) {
        this.student = s;
        this.questions = qs;
        for (Question q : qs) totalMarks += q.marks;

        setTitle("Quiz and Assessment System");
        setLayout(new FlowLayout());
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        questionArea = new JTextArea(4, 30);
        questionArea.setEditable(false);
        answerField = new JTextField(20);
        nextButton = new JButton("Next");
        scoreLabel = new JLabel("Score: 0");

        add(questionArea);
        add(answerField);
        add(nextButton);
        add(scoreLabel);

        showQuestion();

        nextButton.addActionListener(e -> handleAnswer());

        setSize(430, 270);
        setVisible(true);
    }

    private void showQuestion() {
        if (currentIndex < questions.size()) {
            Question q = questions.get(currentIndex);
            questionArea.setText("Q" + (currentIndex + 1) + ": " + q.questionText);
            answerField.setText("");
        } else {
            JOptionPane.showMessageDialog(this,
                    "Quiz completed!\nYour Score: " + student.getScore() + " out of " + totalMarks);
            FileHandler.saveResult(student, totalMarks);
            DBConnector.saveToDB(student, totalMarks);
            dispose();
        }
    }

    private void handleAnswer() {
        try {
            String ans = answerField.getText();
            QuizUtility.validateAnswer(ans);
            Question q = questions.get(currentIndex);
            if (q.evaluate(ans)) {
                student.updateScore(q.marks);
            }
            scoreLabel.setText("Score: " + student.getScore());
            currentIndex++;
            showQuestion();
        } catch (InvalidAnswerException ex) {
            JOptionPane.showMessageDialog(this, ex.getMessage());
        }
    }
}

/* ==========================================================
   MAIN APPLICATION – INTEGRATION OF ALL MODULES
   ========================================================== */

public class QuizSystem {
    public static void main(String[] args) {

        // --- UNIT I: OOP Setup ---
        GenericList<Question> questionList = new GenericList<>();

        // Add 10 MCQs
        questionList.addItem(new MCQQuestion("1. What is OOP in Java?", 5, "Object Oriented Programming"));
        questionList.addItem(new MCQQuestion("2. Which keyword is used for inheritance?", 5, "extends"));
        questionList.addItem(new MCQQuestion("3. Which class is the parent of all Java classes?", 5, "Object"));
        questionList.addItem(new MCQQuestion("4. Which keyword prevents method overriding?", 5, "final"));
        questionList.addItem(new MCQQuestion("5. What is used to handle exceptions?", 5, "try catch"));
        questionList.addItem(new MCQQuestion("6. Which interface is used for multithreading?", 5, "Runnable"));
        questionList.addItem(new MCQQuestion("7. Which package is used for GUI components?", 5, "javax.swing"));
        questionList.addItem(new MCQQuestion("8. What does JDBC stand for?", 5, "Java Database Connectivity"));
        questionList.addItem(new MCQQuestion("9. Which class handles input and output streams?", 5, "IOStream"));
        questionList.addItem(new MCQQuestion("10. What keyword is used to create objects in Java?", 5, "new"));

        Student student = new Student("Atchaya");

        // --- UNIT III: Multithreading Simulation ---
        new QuizThread(student, questionList.getItems()).start();

        // --- UNIT IV: Server Networking (runs in background) ---
        new Thread(() -> QuizServer.startServer()).start();

        // --- UNIT V: Launch GUI ---
        javax.swing.SwingUtilities.invokeLater(() -> new QuizForm(student, questionList.getItems()));
    }
}
