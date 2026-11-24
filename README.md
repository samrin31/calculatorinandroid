## Ex.No:5 Develop a simple calculator using android studio.
## DATE : 24.11.26
## AIM:
To develop a program to develop a simple calculator in Android Studio.

## EQUIPMENTS REQUIRED:
Android Studio(Min.required Artic Fox)

## ALGORITHM:
Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as calculator and click Next.

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout using UI components in activity_main.xml.

Step 6: Display the calculator operation in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
/* Program to print the text “calculator operation”. Developed by: SAMRIN T Registeration Number : 212225100042 */

## MainActivity.java
```
package com.example.calculator;

import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import java.math.BigDecimal;

public class MainActivity extends AppCompatActivity {

    private TextView tvExpression, tvResult;

    private final StringBuilder expression = new StringBuilder();
    private boolean justEvaluated = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvExpression = findViewById(R.id.tvExpression);
        tvResult = findViewById(R.id.tvResult);

        // Number buttons
        int[] numberIds = {
                R.id.btn0, R.id.btn1, R.id.btn2,
                R.id.btn3, R.id.btn4, R.id.btn5,
                R.id.btn6, R.id.btn7, R.id.btn8,
                R.id.btn9
        };

        for (int id : numberIds) {
            findViewById(id).setOnClickListener(v -> {
                Button button = (Button) v;
                appendNumber(button.getText().toString());
            });
        }

        // Arithmetic operators
        setButton(R.id.btnAdd, () -> appendOperator("+"));
        setButton(R.id.btnSubtract, () -> appendOperator("-"));
        setButton(R.id.btnMultiply, () -> appendOperator("*"));
        setButton(R.id.btnDivide, () -> appendOperator("/"));

        // Decimal
        setButton(R.id.btnDot, this::appendDecimal);

        // Percentage
        setButton(R.id.btnPercent, this::appendPercent);

        // Positive / Negative
        setButton(R.id.btnSign, this::toggleSign);

        // Clear
        setButton(R.id.btnClear, this::clearAll);

        // Delete
        setButton(R.id.btnDelete, this::deleteLast);

        // Equal
        setButton(R.id.btnEquals, this::calculate);
    }

    private void setButton(int id, Runnable action) {
        findViewById(id).setOnClickListener(v -> action.run());
    }

    private void appendNumber(String number) {
        if (justEvaluated) {
            expression.setLength(0);
            justEvaluated = false;
        }

        int length = expression.length();

        if (length > 0 && expression.charAt(length - 1) == '%') {
            expression.append("*");
        }

        expression.append(number);
        updateDisplay();
    }

    private void appendOperator(String operator) {
        int initialLength = expression.length();
        if (initialLength == 0) {
            if ("-".equals(operator)) {
                expression.append("-");
                updateDisplay();
            }
            return;
        }

        char last = expression.charAt(initialLength - 1);

        if (last == '.') {
            expression.deleteCharAt(initialLength - 1);
            int newLength = expression.length();
            if (newLength == 0) {
                if ("-".equals(operator)) {
                    expression.append("-");
                    updateDisplay();
                }
                return;
            }
            last = expression.charAt(newLength - 1);
        }

        int currentLen = expression.length();
        if (isOperator(last)) {
            if ("-".equals(operator) && last != '-') {
                expression.append("-");
            } else if (currentLen > 1 && expression.charAt(currentLen - 2) == '(') {
                return;
            } else {
                expression.setCharAt(currentLen - 1, operator.charAt(0));
            }
        } else {
            expression.append(operator);
        }

        justEvaluated = false;
        updateDisplay();
    }

    private void appendDecimal() {
        if (justEvaluated) {
            expression.setLength(0);
            justEvaluated = false;
        }

        int length = expression.length();

        if (length == 0 || isOperator(expression.charAt(length - 1))) {
            expression.append("0.");
            updateDisplay();
            return;
        }

        if (expression.charAt(length - 1) == '%') {
            expression.append("*0.");
            updateDisplay();
            return;
        }

        int start = length - 1;

        while (start >= 0 &&
                (Character.isDigit(expression.charAt(start))
                        || expression.charAt(start) == '.')) {
            start--;
        }

        String currentNumber = expression.substring(start + 1);

        if (!currentNumber.contains(".")) {
            expression.append(".");
        }

        updateDisplay();
    }

    private void appendPercent() {
        if (expression.length() == 0) return;

        char last = expression.charAt(expression.length() - 1);

        if (Character.isDigit(last)) {
            expression.append("%");
            justEvaluated = false;
            updateDisplay();
        }
    }

    private void toggleSign() {
        if (expression.length() == 0) {
            expression.append("-");
            updateDisplay();
            return;
        }

        int end = expression.length() - 1;

        if (!Character.isDigit(expression.charAt(end))
                && expression.charAt(end) != '.') {
            return;
        }

        int start = end;

        while (start >= 0 &&
                (Character.isDigit(expression.charAt(start))
                        || expression.charAt(start) == '.')) {
            start--;
        }

        if (start >= 0 && expression.charAt(start) == '-'
                && (start == 0 || isOperator(expression.charAt(start - 1)))) {
            expression.deleteCharAt(start);
        } else {
            expression.insert(start + 1, "-");
        }

        justEvaluated = false;
        updateDisplay();
    }

    private void deleteLast() {
        if (expression.length() > 0) {
            expression.deleteCharAt(expression.length() - 1);
        }

        justEvaluated = false;
        updateDisplay();
    }

    private void clearAll() {
        expression.setLength(0);
        justEvaluated = false;

        tvExpression.setText("");
        tvResult.setText("0");
    }

    private void calculate() {
        if (expression.length() == 0) return;

        try {
            String original = expression.toString();

            Parser parser = new Parser(original);
            double result = parser.parse();

            if (Double.isNaN(result) || Double.isInfinite(result)) {
                throw new ArithmeticException();
            }

            String answer = formatResult(result);

            tvExpression.setText(String.format("%s =", original));
            tvResult.setText(answer);

            expression.setLength(0);
            expression.append(answer);

            justEvaluated = true;

        } catch (ArithmeticException e) {
            tvResult.setText(R.string.error_text);
            Toast.makeText(this, R.string.cannot_divide_by_zero, Toast.LENGTH_SHORT).show();
            expression.setLength(0);
            justEvaluated = false;
        } catch (Exception e) {
            tvResult.setText(R.string.error_text);
            Toast.makeText(this, R.string.invalid_expression, Toast.LENGTH_SHORT).show();
            expression.setLength(0);
            justEvaluated = false;
        }
    }

    private void updateDisplay() {
        tvExpression.setText(expression.toString());

        if (expression.length() == 0) {
            tvResult.setText("0");
        } else {
            tvResult.setText(expression.toString());
        }
    }

    private boolean isOperator(char c) {
        return c == '+' || c == '-' || c == '*' || c == '/';
    }

    private String formatResult(double value) {
        return BigDecimal.valueOf(value)
                .stripTrailingZeros()
                .toPlainString();
    }

    // Mathematical expression parser
    private static class Parser {

        private final String input;
        private int position = 0;

        Parser(String input) {
            this.input = input;
        }

        double parse() {
            double result = parseExpression();

            if (position != input.length()) {
                throw new IllegalArgumentException();
            }

            return result;
        }

        // Addition and subtraction
        private double parseExpression() {
            double value = parseTerm();

            while (position < input.length()) {
                char operator = input.charAt(position);

                if (operator == '+') {
                    position++;
                    value += parseTerm();
                } else if (operator == '-') {
                    position++;
                    value -= parseTerm();
                } else {
                    break;
                }
            }

            return value;
        }

        // Multiplication and division
        private double parseTerm() {
            double value = parseFactor();

            while (position < input.length()) {
                char operator = input.charAt(position);

                if (operator == '*') {
                    position++;
                    value *= parseFactor();
                } else if (operator == '/') {
                    position++;

                    double divisor = parseFactor();

                    if (divisor == 0) {
                        throw new ArithmeticException();
                    }

                    value /= divisor;
                } else {
                    break;
                }
            }

            return value;
        }

        // Positive, negative and percentage
        private double parseFactor() {
            boolean negative = false;

            while (position < input.length()) {
                char c = input.charAt(position);

                if (c == '-') {
                    negative = !negative;
                    position++;
                } else if (c == '+') {
                    position++;
                } else {
                    break;
                }
            }

            double value = parsePrimary();

            while (position < input.length()
                    && input.charAt(position) == '%') {
                value /= 100.0;
                position++;
            }

            return negative ? -value : value;
        }

        // Numbers and brackets
        private double parsePrimary() {
            if (position >= input.length()) {
                throw new IllegalArgumentException();
            }

            if (input.charAt(position) == '(') {
                position++;

                double value = parseExpression();

                if (position >= input.length()
                        || input.charAt(position) != ')') {
                    throw new IllegalArgumentException();
                }

                position++;
                return value;
            }

            int start = position;

            while (position < input.length()) {
                char c = input.charAt(position);

                if (Character.isDigit(c) || c == '.') {
                    position++;
                } else {
                    break;
                }
            }

            if (start == position) {
                throw new IllegalArgumentException();
            }

            return Double.parseDouble(input.substring(start, position));
        }
    }
}
```
## activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#101114"
    android:padding="12dp">

    <!-- Application Heading -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:text="@string/app_heading"
        android:textColor="#FF9800"
        android:textSize="20sp"
        android:textStyle="bold"
        android:gravity="center_vertical"
        android:paddingStart="12dp"/>

    <!-- Display Section -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="vertical"
        android:gravity="bottom"
        android:padding="10dp">

        <TextView
            android:id="@+id/tvExpression"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text=""
            android:textColor="#999BA5"
            android:textSize="22sp"
            android:gravity="end"
            android:maxLines="2"
            android:padding="8dp"/>

        <TextView
            android:id="@+id/tvResult"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="0"
            android:textColor="#FFFFFF"
            android:textSize="46sp"
            android:textStyle="bold"
            android:gravity="end"
            android:maxLines="1"
            android:padding="8dp"/>

    </LinearLayout>

    <!-- Divider -->
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#303139"
        android:layout_marginBottom="10dp"/>

    <!-- Keypad -->
    <GridLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1.8"
        android:columnCount="4"
        android:rowCount="5"
        android:alignmentMode="alignMargins"
        android:useDefaultMargins="true">

        <!-- Row 1 -->
        <Button
            android:id="@+id/btnClear"
            style="@style/CalcButton"
            android:text="C"
            android:textColor="#FF8585"
            android:backgroundTint="#56515D"/>

        <Button
            android:id="@+id/btnDelete"
            style="@style/CalcButton"
            android:text="DEL"
            android:textSize="17sp"
            android:backgroundTint="#56515D"/>

        <Button
            android:id="@+id/btnPercent"
            style="@style/CalcButton"
            android:text="%"
            android:backgroundTint="#56515D"/>

        <Button
            android:id="@+id/btnDivide"
            style="@style/CalcButton"
            android:text="÷"
            android:backgroundTint="#FF9800"/>

        <!-- Row 2 -->
        <Button
            android:id="@+id/btn7"
            style="@style/CalcButton"
            android:text="7"/>

        <Button
            android:id="@+id/btn8"
            style="@style/CalcButton"
            android:text="8"/>

        <Button
            android:id="@+id/btn9"
            style="@style/CalcButton"
            android:text="9"/>

        <Button
            android:id="@+id/btnMultiply"
            style="@style/CalcButton"
            android:text="×"
            android:backgroundTint="#FF9800"/>

        <!-- Row 3 -->
        <Button
            android:id="@+id/btn4"
            style="@style/CalcButton"
            android:text="4"/>

        <Button
            android:id="@+id/btn5"
            style="@style/CalcButton"
            android:text="5"/>

        <Button
            android:id="@+id/btn6"
            style="@style/CalcButton"
            android:text="6"/>

        <Button
            android:id="@+id/btnSubtract"
            style="@style/CalcButton"
            android:text="−"
            android:backgroundTint="#FF9800"/>

        <!-- Row 4 -->
        <Button
            android:id="@+id/btn1"
            style="@style/CalcButton"
            android:text="1"/>

        <Button
            android:id="@+id/btn2"
            style="@style/CalcButton"
            android:text="2"/>

        <Button
            android:id="@+id/btn3"
            style="@style/CalcButton"
            android:text="3"/>

        <Button
            android:id="@+id/btnAdd"
            style="@style/CalcButton"
            android:text="+"
            android:backgroundTint="#FF9800"/>

        <!-- Row 5 -->
        <Button
            android:id="@+id/btnSign"
            style="@style/CalcButton"
            android:text="+/−"
            android:textSize="17sp"
            android:backgroundTint="#56515D"/>

        <Button
            android:id="@+id/btn0"
            style="@style/CalcButton"
            android:text="0"/>

        <Button
            android:id="@+id/btnDot"
            style="@style/CalcButton"
            android:text="."/>

        <Button
            android:id="@+id/btnEquals"
            style="@style/CalcButton"
            android:text="="
            android:backgroundTint="#FF9800"/>

    </GridLayout>

    <!-- Footer -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="35dp"
        android:text="@string/app_heading"
        android:textColor="#777777"
        android:textSize="11sp"
        android:gravity="center"/>

</LinearLayout>
```    
## AndroidManifest.xml
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Calculator">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>

            </intent-filter>

        </activity>

    </application>

</manifest>
```
## OUTPUT
## MAINACTIVITY.JAVA:
<img width="1672" height="941" alt="Profile-Free Android Studio Calculator Workspace" src="https://github.com/user-attachments/assets/cb27fd0f-0ea9-4fa6-813d-6c4c88f4535a" />

## ACTIVITY_MAIN.XML:
<img width="1672" height="941" alt="Dark Theme Calculator IDE Preview" src="https://github.com/user-attachments/assets/b48fbb4c-0518-4c92-bb7f-1528868ac29a" />

## Addition operation:
<img width="1677" height="938" alt="Dark Android Studio Calculator Preview" src="https://github.com/user-attachments/assets/64cf3744-601f-4e95-8844-3f5a9a49db13" />

## Subtraction Operation:
<img width="1666" height="944" alt="Dark Android Studio Calculator Workspace" src="https://github.com/user-attachments/assets/e4d09fbd-f5e0-43fd-afd7-a208274ea231" />

## RESULT
Thus a Simple Android Application develop a program to create simple calculator in Android Studio is developed and executed successfully.
