# Changelog

## [v0.1.0] - 2025-05-31

### Added
- **Core Email Functionality**: Complete email sending capabilities with SMTP support
- **Message Builder**: Fluent API for building HTML and plain text emails
- **Template System**: Support for email templates with dynamic content
- **Attachment Support**: Send files as email attachments with proper MIME handling
- **Queue Integration**: Async email sending via go.fork.vn/queue for better performance
- **Configuration Management**: Flexible SMTP configuration via go.fork.vn/config
- **Multiple Recipients**: Support for To, CC, BCC recipients
- **Email Validation**: Built-in email address validation
- **Error Handling**: Comprehensive error handling and retry mechanisms
- **Dependency Injection**: Seamless integration with go.fork.vn/di
- **Testing Support**: Mock implementations and test helpers
- **Logging Integration**: Email sending logs via go.fork.vn/log
- **SSL/TLS Support**: Secure email transmission with encryption
- **Authentication**: Multiple SMTP authentication methods
- **Batch Sending**: Efficient bulk email sending capabilities
- **Email Tracking**: Basic email delivery status tracking
- **Content Types**: Support for HTML, plain text, and multipart emails
- **Character Encoding**: Proper UTF-8 and other encoding support
- **Priority Handling**: Email priority levels for queue processing
- **Connection Pooling**: Efficient SMTP connection management

### Features
- Fluent message builder API for easy email composition
- Template-based email generation with variable substitution
- Asynchronous email sending via background job queue
- Multiple SMTP provider support (Gmail, SendGrid, Mailgun, etc.)
- Comprehensive configuration options for SMTP settings
- Built-in retry logic for failed email deliveries
- Email queue management with priority and delay options
- File attachment handling with size and type validation
- Email address validation and sanitization
- Bulk email sending with rate limiting
- Email delivery status tracking and reporting
- Mock implementations for testing environments
- Thread-safe operations for concurrent usage
- Graceful error handling and recovery
- Extensive logging and debugging capabilities
