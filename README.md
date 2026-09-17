import java.util.InputMismatchException;
import java.util.Scanner;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
public class Calculator extends JFrame implements ActionListener {
    private JTextField display;
    private double firstOperand;
    private String operator;
    private boolean startNewNumber;

    public Calculator() {
        setTitle("Calculator");
        setSize(300, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        display = new JTextField();
        display.setEditable(false);
        display.setHorizontalAlignment(JTextField.RIGHT);
        add(display, BorderLayout.NORTH);

        JPanel buttonPanel = new JPanel();
        buttonPanel.setLayout(new GridLayout(4, 4));

        String[] buttons = {
            "7", "8", "9", "/",
            "4", "5", "6", "*",
            "1", "2", "3", "-",
            "0", ".", "=", "+"
        };

        for (String text : buttons) {
            JButton button = new JButton(text);
            button.addActionListener(this);
            buttonPanel.add(button);
        }

        add(buttonPanel, BorderLayout.CENTER);
        startNewNumber = true;
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();

        if ("0123456789.".contains(command)) {
            if (startNewNumber) {
                display.setText(command);
                startNewNumber = false;
            } else {
                display.setText(display.getText() + command);
            }
        } else if ("+-*/".contains(command)) {
            firstOperand = Double.parseDouble(display.getText());
            operator = command;
            startNewNumber = true;
        } else if ("=".equals(command)) {
            double secondOperand = Double.parseDouble(display.getText());
            double result = 0;

            switch (operator) {
                case "+": result = firstOperand + secondOperand; break;
                case "-": result = firstOperand - secondOperand; break;
                case "*": result = firstOperand * secondOperand; break;
                case "/":
                    if (secondOperand == 0) {
                        JOptionPane.showMessageDialog(this, "Cannot divide by zero");
                        return;
                    }
                    result = firstOperand / secondOperand; break;
            }

            display.setText(String.valueOf(result));
            startNewNumber = true;
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            Calculator calculator = new Calculator();
            calculator.setLocationRelativeTo(null);
            calculator.setVisible(true);
        });
    }

    // --- Core operations (unit-testable) ---
    public static double add(double a, double b) { return a + b; }
    public static double subtract(double a, double b) { return a - b; }
    public static double multiply(double a, double b) { return a * b; }
    public static double divide(double a, double b) {
        if (b == 0.0) throw new IllegalArgumentException("Cannot divide by zero");
        return a / b;
    }

    // --- CLI App ---
    public static void main1(String[] args) {
        try (Scanner in = new Scanner(System.in)) {
            System.out.println("=== Java Calculator ===");

            while (true) {
                printMenu();
                int choice = readInt(in, "Choose option (1-5): ");

                if (choice == 5) {
                    System.out.println("Goodbye!");
                    break;
                }

                double a = readDouble(in, "Enter first number: ");
                double b = readDouble(in, "Enter second number: ");

                try {
                    double result = switch (choice) {
                        case 1 -> add(a, b);
                        case 2 -> subtract(a, b);
                        case 3 -> multiply(a, b);
                        case 4 -> divide(a, b);
                        default -> {
                            System.out.println("Invalid option. Try again.");
                            yield Double.NaN;
                        }
                    };
                    if (!Double.isNaN(result)) {
                        System.out.printf("Result: %.6f%n%n", result);
                    }
                } catch (IllegalArgumentException ex) {
                    System.out.println("Error: " + ex.getMessage() + "\n");
                }
            }
        }
    }

    private static void printMenu() {
        System.out.println("1) +  Add");
        System.out.println("2) -  Subtract");
        System.out.println("3) *  Multiply");
        System.out.println("4) /  Divide");
        System.out.println("5) Exit");
    }

    private static int readInt(Scanner in, String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                return in.nextInt();
            } catch (InputMismatchException e) {
                System.out.println("Please enter a valid integer.");
                in.nextLine(); // clear invalid token
            }
        }
    }

    private static double readDouble(Scanner in, String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                return in.nextDouble();
            } catch (InputMismatchException e) {
                System.out.println("Please enter a valid number (e.g., 12 or 12.5).");
                in.nextLine(); // clear invalid token
            }
        }
    }
}
