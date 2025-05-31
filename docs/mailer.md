# Mailer Package Documentation

## Tổng quan

Package `go.fork.vn/mailer` cung cấp một hệ thống gửi email mạnh mẽ và linh hoạt cho các ứng dụng Go, được thiết kế để tích hợp dễ dàng với hệ thống Dependency Injection `go.fork.vn/di` và Queue system `go.fork.vn/queue`.

### Đặc điểm chính

- **Email Composition**: API fluent để tạo email HTML và plain text
- **Template Support**: Hỗ trợ template engine với `text/template` và `html/template`
- **Queue Integration**: Gửi email bất đồng bộ qua hệ thống queue
- **Attachment Handling**: Gửi file đính kèm và embed hình ảnh
- **Multiple Recipients**: Hỗ trợ To, CC, BCC recipients
- **SMTP Configuration**: Cấu hình linh hoạt cho nhiều SMTP provider
- **Error Handling**: Xử lý lỗi và retry mechanism toàn diện
- **Testing Support**: Mock implementations cho unit testing

## Kiến trúc

### Core Components

1. **Manager Interface**: Định nghĩa API chính cho việc gửi email
2. **Mailer Implementation**: Implementation thực tế sử dụng gomail.v2
3. **Message Builder**: Fluent API để xây dựng email messages
4. **ServiceProvider**: Tích hợp với DI container
5. **Queue Integration**: Async email sending via queue

### Interface Manager

Interface `Manager` định nghĩa các nhóm phương thức sau:

#### Email Sending Methods
```go
Send(message *Message) error
SendNow(message *Message) error
SendQueue(message *Message) error
SendTemplate(template string, data interface{}, to ...string) error
SendBatch(messages []*Message) error
```

#### Message Building Methods  
```go
NewMessage() *Message
From(email, name string) *Message
To(emails ...string) *Message
CC(emails ...string) *Message
BCC(emails ...string) *Message
Subject(subject string) *Message
Body(contentType, body string) *Message
Attach(filepath string) *Message
Embed(filepath, cid string) *Message
```

#### Configuration Management
```go
SetSMTPConfig(config SMTPConfig) error
GetSMTPConfig() SMTPConfig
TestConnection() error
```

## Message Builder

Message builder cung cấp fluent API để xây dựng email:

```go
message := mailer.NewMessage().
    From("sender@example.com", "Sender Name").
    To("recipient@example.com").
    CC("cc@example.com").
    BCC("bcc@example.com").
    Subject("Email Subject").
    Body("text/html", "<h1>Hello World</h1>").
    Attach("/path/to/file.pdf").
    Embed("/path/to/image.png", "image1")
```

### Message Properties

- **Recipients**: To, CC, BCC email addresses
- **Subject**: Email subject line
- **Body**: Email content (HTML or plain text)
- **Attachments**: File attachments
- **Embedded Files**: Inline images and files
- **Headers**: Custom email headers
- **Priority**: Email priority level

## SMTP Configuration

Package hỗ trợ cấu hình SMTP linh hoạt:

```go
type SMTPConfig struct {
    Host     string // SMTP server host
    Port     int    // SMTP server port  
    Username string // SMTP username
    Password string // SMTP password
    UseTLS   bool   // Use TLS encryption
    UseSSL   bool   // Use SSL encryption
    SkipVerify bool // Skip certificate verification
}
```

### Supported SMTP Providers

- **Gmail**: smtp.gmail.com:587
- **Outlook**: smtp-mail.outlook.com:587
- **Yahoo**: smtp.mail.yahoo.com:587
- **SendGrid**: smtp.sendgrid.net:587
- **Mailgun**: smtp.mailgun.org:587
- **Custom SMTP**: Any SMTP server

## Queue Integration

Package tích hợp với `go.fork.vn/queue` để gửi email bất đồng bộ:

### Queue Benefits

- **Performance**: Không block main thread khi gửi email
- **Reliability**: Retry mechanism cho failed emails
- **Scalability**: Xử lý hàng loạt email hiệu quả
- **Priority**: Ưu tiên email quan trọng
- **Delay**: Lên lịch gửi email

### Queue Configuration

```yaml
mailer:
  queue:
    enabled: true
    queue_name: "emails"
    priority: "normal"
    max_retries: 3
    retry_delay: "5m"
```

## Template System

### Template Types

1. **Text Templates**: Plain text emails với variable substitution
2. **HTML Templates**: Rich HTML emails với styling
3. **Multipart Templates**: Kết hợp text và HTML

### Template Usage

```go
// Load template
template := `
Hello {{.Name}},

Welcome to our service! Your account has been created successfully.

Best regards,
The Team
`

// Send with template
data := map[string]interface{}{
    "Name": "John Doe",
}

err := mailer.SendTemplate(template, data, "user@example.com")
```

### Template Files

```go
// Load from file
template, err := mailer.LoadTemplate("welcome.html")
if err != nil {
    return err
}

err = mailer.SendTemplate(template, data, "user@example.com")
```

## Attachment Handling

### File Attachments

```go
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Document Attached").
    Body("text/plain", "Please find the document attached.").
    Attach("/path/to/document.pdf")
```

### Embedded Images

```go
message := mailer.NewMessage().
    From("sender@example.com").
    To("recipient@example.com").
    Subject("Newsletter").
    Body("text/html", `
        <h1>Newsletter</h1>
        <img src="cid:logo" alt="Logo">
    `).
    Embed("/path/to/logo.png", "logo")
```

### Attachment Validation

- **File Size**: Configurable maximum file size
- **File Types**: Allowed/blocked file extensions
- **Security**: Virus scanning integration
- **Compression**: Automatic compression for large files

## Error Handling

### Error Types

```go
type EmailError struct {
    Type    ErrorType
    Message string
    Cause   error
}

const (
    ErrorTypeSMTP ErrorType = iota
    ErrorTypeTemplate
    ErrorTypeAttachment
    ErrorTypeValidation
    ErrorTypeQueue
)
```

### Retry Logic

```go
type RetryConfig struct {
    MaxRetries    int
    RetryDelay    time.Duration
    BackoffFactor float64
    MaxDelay      time.Duration
}
```

## Testing Support

### Mock Implementation

```go
type MockMailer struct {
    SentMessages []*Message
    ShouldError  bool
    ErrorType    ErrorType
}

func (m *MockMailer) Send(message *Message) error {
    if m.ShouldError {
        return &EmailError{Type: m.ErrorType}
    }
    
    m.SentMessages = append(m.SentMessages, message)
    return nil
}
```

### Test Helpers

```go
func TestEmailSending(t *testing.T) {
    mockMailer := &MockMailer{}
    
    message := mockMailer.NewMessage().
        From("test@example.com").
        To("recipient@example.com").
        Subject("Test Email").
        Body("text/plain", "Test content")
    
    err := mockMailer.Send(message)
    assert.NoError(t, err)
    assert.Len(t, mockMailer.SentMessages, 1)
}
```

## Performance & Monitoring

### Connection Pooling

```go
type ConnectionPool struct {
    MaxConnections  int
    IdleTimeout     time.Duration
    MaxLifetime     time.Duration
    HealthCheck     bool
}
```

### Metrics

- **Email Count**: Sent, failed, queued emails
- **Delivery Time**: Average time to send
- **Error Rates**: SMTP, template, validation errors
- **Queue Metrics**: Queue size, processing time

### Logging

```go
type LogLevel int

const (
    LogLevelError LogLevel = iota
    LogLevelWarn
    LogLevelInfo
    LogLevelDebug
)
```

## Security Features

### Email Validation

- **Format Validation**: RFC 5322 compliant
- **Domain Validation**: MX record checking
- **Blacklist**: Block suspicious domains
- **Rate Limiting**: Prevent spam

### Content Security

- **HTML Sanitization**: Remove malicious scripts
- **Attachment Scanning**: Virus detection
- **Content Filtering**: Spam detection
- **Encryption**: TLS/SSL support

## Best Practices

### Email Composition

1. **Clear Subject Lines**: Descriptive and concise
2. **Mobile-Friendly HTML**: Responsive design
3. **Alt Text**: For images and attachments
4. **Unsubscribe Links**: Compliance with regulations

### Performance

1. **Batch Sending**: Group similar emails
2. **Template Caching**: Cache compiled templates
3. **Connection Reuse**: SMTP connection pooling
4. **Queue Usage**: Async for non-critical emails

### Deliverability

1. **SPF Records**: Domain authentication
2. **DKIM Signing**: Email signatures
3. **Reputation Management**: Monitor send rates
4. **List Hygiene**: Remove invalid addresses

## Configuration Examples

### Basic SMTP

```yaml
mailer:
  driver: "smtp"
  smtp:
    host: "smtp.gmail.com"
    port: 587
    username: "your-email@gmail.com"
    password: "your-app-password"
    use_tls: true
```

### Advanced Configuration

```yaml
mailer:
  driver: "smtp"
  
  smtp:
    host: "smtp.example.com"
    port: 587
    username: "mailer@example.com"
    password: "secure-password"
    use_tls: true
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
    allowed_types: ["pdf", "doc", "docx", "jpg", "png"]
    
  security:
    validate_emails: true
    sanitize_html: true
    rate_limit: 100
```
