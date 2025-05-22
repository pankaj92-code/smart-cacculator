# smart-caculator
package com.example.calculator;
import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.speech.RecognizerIntent;
import android.view.View;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import com.google.android.material.button.MaterialButton;

import net.objecthunter.exp4j.Expression;
import net.objecthunter.exp4j.ExpressionBuilder;

import java.util.ArrayList;
import java.util.Locale;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    TextView resultTv, solutionTV;
    MaterialButton buttonVoice, buttonC, buttonBrackopen, buttonBrackClose;
    MaterialButton buttonDivide, buttonMultiply, buttonPlus, buttonMinus, buttonEquals;
    MaterialButton button0, button1, button2, button3, button4, button5, button6, button7, button8, button9;
    MaterialButton buttonAC, buttonDot;

    String dataToCalculate = ""; // Retaining state
    private static final int REQUEST_CODE_SPEECH_INPUT = 100;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        resultTv = findViewById(R.id.result_tv);
        solutionTV = findViewById(R.id.solution_tv);

        // Assign buttons correctly
        buttonVoice = assignId(R.id.button_voice);
        buttonC = assignId(R.id.button_c);
        buttonBrackopen = assignId(R.id.button_open_bracket);
        buttonBrackClose = assignId(R.id.button_close_bracket);
        buttonDivide = assignId(R.id.button_divide); // Corrected
        buttonMultiply = assignId(R.id.button_multiply); // Corrected
        buttonPlus = assignId(R.id.button_addition);
        buttonMinus = assignId(R.id.button_subtract);
        buttonEquals = assignId(R.id.button_equals);
        button0 = assignId(R.id.button_0);
        button1 = assignId(R.id.button_1);
        button2 = assignId(R.id.button_2);
        button3 = assignId(R.id.button_3);
        button4 = assignId(R.id.button_4);
        button5 = assignId(R.id.button_5);
        button6 = assignId(R.id.button_6);
        button7 = assignId(R.id.button_7);
        button8 = assignId(R.id.button_8);
        button9 = assignId(R.id.button_9);
        buttonAC = assignId(R.id.button_ac);
        buttonDot = assignId(R.id.button_dot);

        checkPermission();
    }

    MaterialButton assignId(int id) {
        MaterialButton btn = findViewById(id);
        btn.setOnClickListener(this);
        return btn;
    }

    @Override
    public void onClick(View view) {
        MaterialButton button = (MaterialButton) view;
        String buttonText = button.getText().toString();

        if (buttonText.equals("🎤 Voice Input")) {
            startVoiceInput();
            return;
        }

        switch (buttonText) {
            case "AC":
                solutionTV.setText("");
                resultTv.setText("0");
                dataToCalculate = "";
                return;
            case "=":
                resultTv.setText(getResult(dataToCalculate));
                return;
            case "C":
                if (!dataToCalculate.isEmpty()) {
                    dataToCalculate = dataToCalculate.substring(0, dataToCalculate.length() - 1);
                }
                break;
            default:
                dataToCalculate += buttonText;
                break;
        }

        solutionTV.setText(dataToCalculate);
    }

    String getResult(String data) {
        if (data.isEmpty()) return "Error";

        try {
            // Sanitize user input to replace common voice recognition errors
            data = data.replace("x", "*").replace("÷", "/").replace("multiply", "*").replace("divide", "/");

            Expression expression = new ExpressionBuilder(data).build();
            double result = expression.evaluate();

            return Double.isNaN(result) ? "ERROR" : String.valueOf(result);
        } catch (Exception e) {
            return "Error";
        }
    }
    private void startVoiceInput() {
        Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, Locale.getDefault());

        try {
            startActivityForResult(intent, REQUEST_CODE_SPEECH_INPUT);
        } catch (Exception e) {
            Toast.makeText(this, "Voice input is not supported on this device!", Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == REQUEST_CODE_SPEECH_INPUT && resultCode == RESULT_OK) {
            ArrayList<String> result = data.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS);
            if (result != null && !result.isEmpty()) {
                String voiceInput = result.get(0);
                dataToCalculate += voiceInput.replace("x", "*").replace("divide", "/"); // Handling common voice input mistakes
                solutionTV.setText(dataToCalculate);
            }
        }
    }

    private void checkPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.RECORD_AUDIO}, 1);
        }
    }
}
