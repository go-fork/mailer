# Usage Guide - Mailer Package

## Cài đặt

```bash
go get go.fork.vn/mailer@v0.1.0
```

## Import

```go
import "go.fork.vn/mailer"
```

## Quick Start

### 1. Sử dụng cơ bản

```go
package main

import (
    "log"
    
    "go.fork.vn/mailer"
)

func main() {
    // Tạo mailer instance
    mailerManager := mailer.NewMailer()
    
    // Cấu hình SMTP
    smtpConfig := mailer.SMTPConfig{
        Host:     "smtp.gmail.com",
        Port:     587,
        Username: "your-email@gmail.com",
        Password: "your-app-password",
        UseTLS:   true,
    }
    
    if err := mailerManager.SetSMTPConfig(smtpConfig); err != nil {
        log.Fatal("Lỗi cấu hình SMTP:", err)
    }
    
    // Tạo và gửi email
    message := mailerManager.NewMessage().
        From("sender@example.com", "Sender Name").
        To("recipient@example.com").
        Subject("Hello from Go Mailer").
        Body("text/html", "<h1>Hello World!</h1><p>This is a test email.</p>")
    
    if err := mailerManager.Send(message); err != nil {
        log.Fatal("Lỗi gửi email:", err)
    }
    
    log.Println("Email sent successfully!")
}
```

### 2. Sử dụng với Dependency Injection

```go
package main

import (
    "log"
    
    "go.fork.vn/di"
    "go.fork.vn/config" 
    "go.fork.vn/mailer"
)

func main() {
    // Tạo DI container
    container := di.NewContainer()
    
    // Đăng ký service providers
    container.Register(config.NewServiceProvider())
    container.Register(mailer.NewServiceProvider())
    
    // Boot container
    if err := container.Boot(); err != nil {
        log.Fatal("Lỗi boot container:", err)
    }
    
    // Resolve mailer manager
    var mailerManager mailer.Manager
    if err := container.Resolve(&mailerManager); err != nil {
        log.Fatal("Lỗi resolve mailer manager:", err)
    }
    
    // Sử dụng mailer manager
    message := mailerManager.NewMessage().
        From("sender@example.com").
        To("recipient@example.com").
        Subject("DI Integration Test").
        Body("text/plain", "Email sent via dependency injection!")
    
    if err := mailerManager.Send(message); err != nil {
        log.Fatal("Lỗi gửi email:", err)
    }
    
    log.Println("Email sent via DI!")
}
```

## Chi tiết sử dụng

### Message Composition

#### Basic Email

```go
message := mailer.NewMessage().
    From("sender@example.com", "John Doe").
    To("recipient@example.com").
    Subject("Basic Email").
    Body("text/plain", "This is a plain text email.")

err := mailerManager.Send(message)
```

#### HTML Email

```go
htmlContent := `
<!DOCTYPE html>
<html>
<head>
    <title>Welcome Email</title>
</head>
<body>
    <h1>Welcome to Our Service!</h1>
    <p>Thank you for signing up.</p>
    <p><a href="https://example.com/verify">Verify your account</a></p>
</body>
</html>
`

message := mailer.NewMessage().
    From("noreply@example.com", "Example Service").
    To("user@example.com").
    Subject("Welcome to Example Service").
    Body("text/html", htmlContent)

err := mailerManager.Send(message)
```

#### Multiple Recipients

```go
message := mailer.NewMessage().
    From("sender@example.com").
    To("user1@example.com", "user2@example.com").
    CC("manager@example.com").
    BCC("admin@example.com").
    Subject("Team Notification").
    Body("text/plain", "This is a team notification.")

err := mailerManager.Send(message)
```

### File Attachments

#### Single Attachment

```go
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Document Attached").
    Body("text/plain", "Please find the document attached.").
    Attach("/path/to/document.pdf")

err := mailerManager.Send(message)
```

#### Multiple Attachments

```go
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Multiple Documents").
    Body("text/plain", "Please find the documents attached.").
    Attach("/path/to/document1.pdf").
    Attach("/path/to/document2.docx").
    Attach("/path/to/spreadsheet.xlsx")

err := mailerManager.Send(message)
```

#### Embedded Images

```go
htmlContent := `
<html>
<body>
    <h1>Newsletter</h1>
    <p>Check out our latest product:</p>
    <img src="cid:product-image" alt="Product Image" width="300">
    <p>Company Logo: <img src="cid:logo" alt="Logo" width="100"></p>
</body>
</html>
`

message := mailer.NewMessage().
    From("newsletter@example.com").
    To("subscriber@example.com").
    Subject("Latest Newsletter").
    Body("text/html", htmlContent).
    Embed("/path/to/product.jpg", "product-image").
    Embed("/path/to/logo.png", "logo")

err := mailerManager.Send(message)
```

### Template System

#### Text Templates

```go
// Template content
textTemplate := `
Hello {{.Name}},

Welcome to {{.ServiceName}}! Your account has been created successfully.

Account Details:
- Username: {{.Username}}
- Email: {{.Email}}
- Registration Date: {{.RegistrationDate}}

To get started, please visit: {{.LoginURL}}

Best regards,
The {{.ServiceName}} Team
`

// Template data
data := map[string]interface{}{
    "Name":             "John Doe",
    "ServiceName":      "Example Service",
    "Username":         "johndoe",
    "Email":            "john@example.com",
    "RegistrationDate": "2025-05-31",
    "LoginURL":         "https://example.com/login",
}

// Send template email
err := mailerManager.SendTemplate(textTemplate, data, "john@example.com")
```

#### HTML Templates

```go
htmlTemplate := `
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to {{.ServiceName}}</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .header { background-color: #007bff; color: white; padding: 20px; }
        .content { padding: 20px; }
        .button { background-color: #28a745; color: white; padding: 10px 20px; text-decoration: none; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Welcome to {{.ServiceName}}</h1>
    </div>
    <div class="content">
        <p>Hello {{.Name}},</p>
        <p>Thank you for joining {{.ServiceName}}! We're excited to have you on board.</p>
        
        <h3>Your Account Details:</h3>
        <ul>
            <li><strong>Username:</strong> {{.Username}}</li>
            <li><strong>Email:</strong> {{.Email}}</li>
            <li><strong>Member Since:</strong> {{.RegistrationDate}}</li>
        </ul>
        
        <p>
            <a href="{{.LoginURL}}" class="button">Get Started</a>
        </p>
        
        <p>If you have any questions, feel free to contact our support team.</p>
        
        <p>Best regards,<br>The {{.ServiceName}} Team</p>
    </div>
</body>
</html>
`

err := mailerManager.SendTemplate(htmlTemplate, data, "john@example.com")
```

#### Template from File

```go
// Load template from file
template, err := ioutil.ReadFile("templates/welcome.html")
if err != nil {
    log.Fatal("Error loading template:", err)
}

// Send template email
err = mailerManager.SendTemplate(string(template), data, "user@example.com")
```

### Queue Integration

#### Async Email Sending

```go
// Send email via queue (asynchronous)
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Async Email").
    Body("text/plain", "This email is sent asynchronously via queue.")

err := mailerManager.SendQueue(message)
if err != nil {
    log.Printf("Error queuing email: %v", err)
} else {
    log.Println("Email queued successfully!")
}
```

#### Immediate vs Queue Sending

```go
// Immediate sending (synchronous)
err := mailerManager.SendNow(message)

// Queue sending (asynchronous)  
err = mailerManager.SendQueue(message)

// Default sending (follows configuration)
err = mailerManager.Send(message)
```

#### Batch Email Sending

```go
messages := []*mailer.Message{
    mailer.NewMessage().
        From("sender@example.com").
        To("user1@example.com").
        Subject("Batch Email 1").
        Body("text/plain", "First email in batch"),
        
    mailer.NewMessage().
        From("sender@example.com").
        To("user2@example.com").
        Subject("Batch Email 2").
        Body("text/plain", "Second email in batch"),
        
    mailer.NewMessage().
        From("sender@example.com").
        To("user3@example.com").
        Subject("Batch Email 3").
        Body("text/plain", "Third email in batch"),
}

err := mailerManager.SendBatch(messages)
```

### SMTP Configuration

#### Gmail Configuration

```go
gmailConfig := mailer.SMTPConfig{
    Host:     "smtp.gmail.com",
    Port:     587,
    Username: "your-email@gmail.com",
    Password: "your-app-password", // Use App Password for 2FA accounts
    UseTLS:   true,
}

err := mailerManager.SetSMTPConfig(gmailConfig)
```

#### Outlook Configuration

```go
outlookConfig := mailer.SMTPConfig{
    Host:     "smtp-mail.outlook.com",
    Port:     587,
    Username: "your-email@outlook.com",
    Password: "your-password",
    UseTLS:   true,
}

err := mailerManager.SetSMTPConfig(outlookConfig)
```

#### SendGrid Configuration

```go
sendGridConfig := mailer.SMTPConfig{
    Host:     "smtp.sendgrid.net",
    Port:     587,
    Username: "apikey",
    Password: "your-sendgrid-api-key",
    UseTLS:   true,
}

err := mailerManager.SetSMTPConfig(sendGridConfig)
```

#### Custom SMTP Configuration

```go
customConfig := mailer.SMTPConfig{
    Host:       "mail.example.com",
    Port:       465,
    Username:   "mailer@example.com",
    Password:   "secure-password",
    UseSSL:     true,
    SkipVerify: false,
}

err := mailerManager.SetSMTPConfig(customConfig)
```

### Configuration với config package

```go
package main

import (
    "log"
    
    "go.fork.vn/config"
    "go.fork.vn/di"
    "go.fork.vn/mailer"
)

func main() {
    // Tạo container
    container := di.NewContainer()
    
    // Đăng ký service providers
    container.Register(config.NewServiceProvider())
    container.Register(mailer.NewServiceProvider())
    
    // Boot container
    if err := container.Boot(); err != nil {
        log.Fatal("Lỗi boot container:", err)
    }
    
    // Resolve services
    var cfg config.Manager
    var mailerManager mailer.Manager
    
    container.Resolve(&cfg)
    container.Resolve(&mailerManager)
    
    // Mailer sẽ tự động đọc config từ config manager
    
    // Sử dụng mailer
    message := mailerManager.NewMessage().
        From("sender@example.com").
        To("recipient@example.com").
        Subject("Config Integration").
        Body("text/plain", "Email with config integration!")
    
    err := mailerManager.Send(message)
    if err != nil {
        log.Fatal("Lỗi gửi email:", err)
    }
}
```

### File cấu hình mẫu (config.yaml)

```yaml
mailer:
  driver: "smtp"
  
  smtp:
    host: "smtp.gmail.com"
    port: 587
    username: "your-email@gmail.com"
    password: "your-app-password"
    use_tls: true
    use_ssl: false
    skip_verify: false
    
  pool:
    max_connections: 10
    idle_timeout: "30s"
    max_lifetime: "1h"
    
  queue:
    enabled: true
    queue_name: "emails"
    priority: "normal"
    max_retries: 3
    retry_delay: "5m"
    
  templates:
    directory: "./templates"
    cache: true
    
  attachments:
    max_size: "10MB"
    allowed_types: ["pdf", "doc", "docx", "jpg", "png", "gif"]
    
  security:
    validate_emails: true
    sanitize_html: true
    rate_limit: 100
    
  defaults:
    from_email: "noreply@example.com"
    from_name: "Example Service"
```

## Advanced Usage

### Custom Message Headers

```go
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Custom Headers").
    Body("text/plain", "Email with custom headers").
    SetHeader("X-Priority", "1").
    SetHeader("X-Mailer", "Go Mailer v0.1.0").
    SetHeader("Reply-To", "support@example.com")

err := mailerManager.Send(message)
```

### Email Priority

```go
// High priority email
highPriorityMessage := mailer.NewMessage().
    From("alert@example.com").
    To("admin@example.com").
    Subject("URGENT: Server Alert").
    Body("text/plain", "Server is down!").
    SetPriority(mailer.PriorityHigh)

// Low priority email
lowPriorityMessage := mailer.NewMessage().
    From("newsletter@example.com").
    To("subscriber@example.com").
    Subject("Weekly Newsletter").
    Body("text/html", newsletterContent).
    SetPriority(mailer.PriorityLow)
```

### Email Validation

```go
// Validate email addresses before sending
validEmails := []string{}
invalidEmails := []string{}

emails := []string{
    "valid@example.com",
    "invalid-email",
    "user@domain.com",
    "bad@",
}

for _, email := range emails {
    if mailerManager.ValidateEmail(email) {
        validEmails = append(validEmails, email)
    } else {
        invalidEmails = append(invalidEmails, email)
    }
}

fmt.Printf("Valid emails: %v\n", validEmails)
fmt.Printf("Invalid emails: %v\n", invalidEmails)
```

### Connection Testing

```go
// Test SMTP connection
if err := mailerManager.TestConnection(); err != nil {
    log.Printf("SMTP connection failed: %v", err)
} else {
    log.Println("SMTP connection successful!")
}
```

### Email Templates với Functions

```go
import (
    "text/template"
    "time"
)

// Template với custom functions
templateContent := `
Hello {{.Name | title}},

Today is {{now | date}}.
Your balance is {{.Balance | currency}}.

Best regards,
The Team
`

// Custom template functions
funcMap := template.FuncMap{
    "title": strings.Title,
    "date": func(t time.Time) string {
        return t.Format("January 2, 2006")
    },
    "currency": func(amount float64) string {
        return fmt.Sprintf("$%.2f", amount)
    },
    "now": func() time.Time {
        return time.Now()
    },
}

// Parse template với functions
tmpl, err := template.New("email").Funcs(funcMap).Parse(templateContent)
if err != nil {
    log.Fatal(err)
}

// Execute template
var buf bytes.Buffer
data := map[string]interface{}{
    "Name":    "john doe",
    "Balance": 1234.56,
}

err = tmpl.Execute(&buf, data)
if err != nil {
    log.Fatal(err)
}

// Send email
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Account Statement").
    Body("text/plain", buf.String())

err = mailerManager.Send(message)
```

### Multipart Email (Text + HTML)

```go
textContent := `
Hello John,

Welcome to our service! Visit https://example.com/welcome to get started.

Best regards,
The Team
`

htmlContent := `
<html>
<body>
    <h1>Hello John,</h1>
    <p>Welcome to our service!</p>
    <p><a href="https://example.com/welcome">Get Started</a></p>
    <p>Best regards,<br>The Team</p>
</body>
</html>
`

message := mailer.NewMessage().
    From("sender@example.com").
    To("john@example.com").
    Subject("Welcome").
    AddAlternative("text/plain", textContent).
    AddAlternative("text/html", htmlContent)

err := mailerManager.Send(message)
```

## Error Handling

### Basic Error Handling

```go
err := mailerManager.Send(message)
if err != nil {
    switch e := err.(type) {
    case *mailer.SMTPError:
        log.Printf("SMTP error: %v", e)
    case *mailer.ValidationError:
        log.Printf("Validation error: %v", e)
    case *mailer.AttachmentError:
        log.Printf("Attachment error: %v", e)
    default:
        log.Printf("Unknown error: %v", e)
    }
}
```

### Retry Logic

```go
func sendWithRetry(mailerManager mailer.Manager, message *mailer.Message, maxRetries int) error {
    var lastErr error
    
    for i := 0; i <= maxRetries; i++ {
        err := mailerManager.Send(message)
        if err == nil {
            return nil
        }
        
        lastErr = err
        
        // Exponential backoff
        if i < maxRetries {
            delay := time.Duration(math.Pow(2, float64(i))) * time.Second
            log.Printf("Send attempt %d failed: %v. Retrying in %v...", i+1, err, delay)
            time.Sleep(delay)
        }
    }
    
    return fmt.Errorf("failed to send email after %d attempts: %w", maxRetries+1, lastErr)
}

// Usage
err := sendWithRetry(mailerManager, message, 3)
```

## Testing

### Unit Testing với Mock

```go
package main

import (
    "testing"
    
    "github.com/stretchr/testify/assert"
    "go.fork.vn/mailer"
)

func TestEmailSending(t *testing.T) {
    // Tạo mock mailer
    mockMailer := &mailer.MockMailer{}
    
    // Tạo message
    message := mockMailer.NewMessage().
        From("test@example.com").
        To("recipient@example.com").
        Subject("Test Email").
        Body("text/plain", "Test content")
    
    // Gửi email
    err := mockMailer.Send(message)
    
    // Kiểm tra kết quả
    assert.NoError(t, err)
    assert.Len(t, mockMailer.SentMessages, 1)
    assert.Equal(t, "Test Email", mockMailer.SentMessages[0].Subject)
}

func TestEmailSendingWithError(t *testing.T) {
    mockMailer := &mailer.MockMailer{
        ShouldError: true,
        ErrorType:   mailer.ErrorTypeSMTP,
    }
    
    message := mockMailer.NewMessage().
        From("test@example.com").
        To("recipient@example.com").
        Subject("Test Email").
        Body("text/plain", "Test content")
    
    err := mockMailer.Send(message)
    
    assert.Error(t, err)
    assert.IsType(t, &mailer.SMTPError{}, err)
}
```

### Integration Testing

```go
func TestEmailIntegration(t *testing.T) {
    // Skip if no test email config
    if testing.Short() {
        t.Skip("Skipping integration test")
    }
    
    // Setup real mailer với test config
    mailerManager := mailer.NewMailer()
    
    testConfig := mailer.SMTPConfig{
        Host:     "smtp.gmail.com",
        Port:     587,
        Username: os.Getenv("TEST_EMAIL"),
        Password: os.Getenv("TEST_PASSWORD"),
        UseTLS:   true,
    }
    
    err := mailerManager.SetSMTPConfig(testConfig)
    require.NoError(t, err)
    
    // Test connection
    err = mailerManager.TestConnection()
    require.NoError(t, err)
    
    // Send test email
    message := mailerManager.NewMessage().
        From(os.Getenv("TEST_EMAIL")).
        To(os.Getenv("TEST_EMAIL")).
        Subject("Integration Test").
        Body("text/plain", "This is an integration test email.")
    
    err = mailerManager.Send(message)
    assert.NoError(t, err)
}
```

## Best Practices

### Email Composition

```go
// ✅ Good: Clear and descriptive subject
message := mailer.NewMessage().
    From("noreply@example.com", "Example Service").
    To("user@example.com").
    Subject("Your password has been reset").
    Body("text/html", htmlContent)

// ❌ Bad: Vague subject
message := mailer.NewMessage().
    Subject("Update").
    Body("text/plain", "Something happened")
```

### Template Management

```go
// ✅ Good: Reusable template structure
type EmailTemplate struct {
    Name    string
    Subject string
    Text    string
    HTML    string
}

var templates = map[string]EmailTemplate{
    "welcome": {
        Name:    "welcome",
        Subject: "Welcome to {{.ServiceName}}",
        Text:    "...",
        HTML:    "...",
    },
    "reset-password": {
        Name:    "reset-password",
        Subject: "Reset your password",
        Text:    "...",
        HTML:    "...",
    },
}

func sendWelcomeEmail(mailerManager mailer.Manager, userEmail string, data interface{}) error {
    template := templates["welcome"]
    return mailerManager.SendTemplate(template.HTML, data, userEmail)
}
```

### Resource Management

```go
// ✅ Good: Proper resource cleanup
func sendEmailWithAttachment(mailerManager mailer.Manager, filePath string) error {
    // Check file exists
    if _, err := os.Stat(filePath); os.IsNotExist(err) {
        return fmt.Errorf("attachment file not found: %s", filePath)
    }
    
    message := mailerManager.NewMessage().
        From("sender@example.com").
        To("recipient@example.com").
        Subject("Document Attached").
        Body("text/plain", "Please find the document attached.").
        Attach(filePath)
    
    return mailerManager.Send(message)
}
```

### Error Handling

```go
// ✅ Good: Comprehensive error handling
func sendNotificationEmail(mailerManager mailer.Manager, userEmail, content string) error {
    // Validate email
    if !mailerManager.ValidateEmail(userEmail) {
        return fmt.Errorf("invalid email address: %s", userEmail)
    }
    
    message := mailerManager.NewMessage().
        From("notifications@example.com", "Example Notifications").
        To(userEmail).
        Subject("New Notification").
        Body("text/html", content)
    
    // Try immediate send first, fallback to queue
    err := mailerManager.SendNow(message)
    if err != nil {
        log.Printf("Immediate send failed: %v. Trying queue...", err)
        return mailerManager.SendQueue(message)
    }
    
    return nil
}
```

## Troubleshooting

### Common Issues

1. **Authentication Failed**
   - Check username và password
   - Enable "Less secure app access" cho Gmail
   - Use App Passwords cho 2FA accounts

2. **Connection Timeout**
   - Check SMTP host và port
   - Verify firewall settings
   - Try different ports (25, 465, 587)

3. **TLS/SSL Issues**
   - Check TLS/SSL settings
   - Try with `SkipVerify: true` for testing
   - Verify certificate validity

4. **Large Attachments**
   - Check file size limits
   - Compress files before attaching
   - Use cloud storage links instead

### Debug Mode

```go
// Enable debug logging
mailerManager.SetDebug(true)

// Hoặc qua config
config:
  mailer:
    debug: true
    log_level: "debug"
```

### Connection Diagnostics

```go
// Test connection với chi tiết
diagnostics := mailerManager.Diagnose()
fmt.Printf("SMTP Host: %s\n", diagnostics.Host)
fmt.Printf("Port: %d\n", diagnostics.Port) 
fmt.Printf("TLS: %v\n", diagnostics.TLS)
fmt.Printf("Auth: %v\n", diagnostics.Auth)
fmt.Printf("Connection: %s\n", diagnostics.Status)
```
