# Automation-test
script to automate the login with dynamic verification code


package logindvc;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import javax.mail.*;
//import javax.mail.internet.MimeMessage;
import java.time.Duration;
import java.util.Properties;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class logindvc {
    public static void main(String[] args) {
        // Set path to ChromeDriver (ensure you have the correct path for your system)
        System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");

        // Initialize WebDriver
        WebDriver driver = new ChromeDriver();
        
        try {
            // Navigate to the login page
            driver.get("https://yourwebsite.com/login");
            
            // Enter Email and Password
            WebElement emailField = driver.findElement(By.id("email"));
            WebElement passwordField = driver.findElement(By.id("password"));
            WebElement loginButton = driver.findElement(By.id("loginButton"));

            emailField.sendKeys("your-email@example.com");
            passwordField.sendKeys("your-password");
            
            // Click the 'Send Verification Code' button
            WebElement sendVerificationCodeButton = driver.findElement(By.id("sendVerificationCodeButton"));
            sendVerificationCodeButton.click();

            // Wait for a few seconds for the verification code email to arrive
            Thread.sleep(5000);  // You can replace this with a better waiting strategy if needed

            // Retrieve the verification code from email (using SMTP)
            String verificationCode = getVerificationCodeFromEmail();

            // Enter the verification code
            WebElement verificationCodeField = driver.findElement(By.id("verificationCode"));
            verificationCodeField.sendKeys(verificationCode);

            // Submit the login form
            WebElement submitButton = driver.findElement(By.id("submitButton"));
            submitButton.click();

            // Wait for successful login (adjust based on the page you're testing)
            WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
            wait.until(ExpectedConditions.urlContains("dashboard"));

            System.out.println("Login successful!");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            driver.quit();
        }
    }

//     Method to fetch the verification code from your email in-box
    private static String getVerificationCodeFromEmail() throws Exception {
        // Set up the email connection properties (this is for G-mail)
        Properties properties = new Properties();
        properties.put("mail.smtp.host", "smtp.gmail.com");
        properties.put("mail.smtp.port", "465");
        properties.put("mail.smtp.auth", "true");
        properties.put("mail.smtp.socketFactory.port", "465");
        properties.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");

        // Create a session with your email credentials
        Session session = Session.getInstance(properties, new javax.mail.Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication("your-email@example.com", "your-email-password");
            }
        });

        // Connect to your email in-box
        Store store = session.getStore("imaps");
        store.connect("imap.gmail.com", "your-email@example.com", "your-email-password");

        // Open the in-box folder
        Folder folder = store.getFolder("INBOX");
        folder.open(Folder.READ_ONLY);

        // Search for unread messages
        Message[] messages = folder.getMessages();
        for (Message message : messages) {
            if (message.getSubject().contains("Your Verification Code")) {
                // Extract verification code from email content
                String emailContent = message.getContent().toString();
                return extractVerificationCode(emailContent);
            }
        }
        return null;  // Return null if no email found
    }

    // Extract verification code from email content using re-gex
    private static String extractVerificationCode(String emailContent) {
        // Re-gex pattern for extracting the 6-digit verification code
        Pattern pattern = Pattern.compile("\\b\\d{6}\\b");
        Matcher matcher = pattern.matcher(emailContent);
        if (matcher.find()) {
            return matcher.group();
        }
        return null;  // Return null if no code found
    }
}
