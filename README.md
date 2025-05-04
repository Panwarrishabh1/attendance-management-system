import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.ArrayList;

public class AttendanceManagementSystem extends JFrame {

    Connection con;
    ArrayList<Student> studentList = new ArrayList<>();

    public AttendanceManagementSystem() {
        // Database Connection
        try {
            Class.forName("com.mysql.jdbc.Driver");
            con = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/attendance_db", "root", "Aa11a198"
            );
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "Database Connection Error: " + e.getMessage());
        }

        setTitle("Student Attendance Management System");
        setSize(500, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new GridLayout(4, 1));

        JButton addStudentBtn = new JButton("Add Student");
        JButton markAttendanceBtn = new JButton("Mark Attendance");
        JButton viewAttendanceBtn = new JButton("View Attendance Report");
        JButton exitBtn = new JButton("Exit");

        add(addStudentBtn);
        add(markAttendanceBtn);
        add(viewAttendanceBtn);
        add(exitBtn);

        addStudentBtn.addActionListener(e -> addStudent());
        markAttendanceBtn.addActionListener(e -> markAttendance());
        viewAttendanceBtn.addActionListener(e -> viewAttendanceReport());
        exitBtn.addActionListener(e -> System.exit(0));

        setVisible(true);
    }

    // Student Class inside the same file
    class Student {
        int id;
        String name;
        String rollNo;

        Student(int id, String name, String rollNo) {
            this.id = id;
            this.name = name;
            this.rollNo = rollNo;
        }
    }

    private void addStudent() {
        String name = JOptionPane.showInputDialog(this, "Enter Student Name:");
        String rollNo = JOptionPane.showInputDialog(this, "Enter Student Roll Number:");

        if (name == null || rollNo == null || name.isEmpty() || rollNo.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Name and Roll No cannot be empty!");
            return;
        }

        try {
            String sql = "INSERT INTO students (name, roll_no) VALUES (?, ?)";
            PreparedStatement pst = con.prepareStatement(sql);
            pst.setString(1, name);
            pst.setString(2, rollNo);
            pst.executeUpdate();

            JOptionPane.showMessageDialog(this, "Student Added Successfully!");
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(this, "Error adding student: " + ex.getMessage());
        }
    }

    private void markAttendance() {
        String rollNo = JOptionPane.showInputDialog(this, "Enter Roll Number to Mark Attendance:");
        if (rollNo == null || rollNo.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Roll Number cannot be empty!");
            return;
        }

        String statusOptions[] = {"Present", "Absent"};
        String status = (String) JOptionPane.showInputDialog(this, "Select Status:",
                "Attendance", JOptionPane.QUESTION_MESSAGE, null, statusOptions, statusOptions[0]);

        if (status == null) {
            JOptionPane.showMessageDialog(this, "Attendance not marked!");
            return;
        }

        try {
            String getIdSql = "SELECT id FROM students WHERE roll_no = ?";
            PreparedStatement getIdStmt = con.prepareStatement(getIdSql);
            getIdStmt.setString(1, rollNo);
            ResultSet rs = getIdStmt.executeQuery();

            if (rs.next()) {
                int studentId = rs.getInt("id");

                String sql = "INSERT INTO attendance(student_id, date, status) VALUES (?, CURDATE(), ?)";
                PreparedStatement pst = con.prepareStatement(sql);
                pst.setInt(1, studentId);
                pst.setString(2, status);
                pst.executeUpdate();

                JOptionPane.showMessageDialog(this, "Attendance Marked Successfully!");
            } else {
                JOptionPane.showMessageDialog(this, "Student not found!");
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(this, "Error marking attendance: " + ex.getMessage());
        }
    }

    private void viewAttendanceReport() {
        try {
            String sql = "SELECT s.name, s.roll_no, a.date, a.status " +
                         "FROM students s JOIN attendance a ON s.id = a.student_id";
            Statement stmt = con.createStatement();
            ResultSet rs = stmt.executeQuery(sql);

            StringBuilder report = new StringBuilder();
            while (rs.next()) {
                report.append("Name: ").append(rs.getString("name"))
                      .append(", Roll No: ").append(rs.getString("roll_no"))
                      .append(", Date: ").append(rs.getDate("date"))
                      .append(", Status: ").append(rs.getString("status"))
                      .append("\n");
            }

            if (report.length() == 0) {
                JOptionPane.showMessageDialog(this, "No Attendance Records Found.");
            } else {
                JTextArea textArea = new JTextArea(report.toString());
                textArea.setEditable(false);
                JScrollPane scrollPane = new JScrollPane(textArea);
                scrollPane.setPreferredSize(new Dimension(400, 300));
                JOptionPane.showMessageDialog(this, scrollPane, "Attendance Report", JOptionPane.INFORMATION_MESSAGE);
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(this, "Error fetching attendance report: " + ex.getMessage());
        }
    }

    public static void main(String[] args) {
        new AttendanceManagementSystem();
    }
}

