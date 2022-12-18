# flog
A golang logging library for concurrent timelines.

The names comes from _forking_ + _logging_.

Features:

  * Support for multiple concurrent timelines through `log.Fork()`
  * Distinction between logging and auditing
    * [ ] Taggable audit messages
    * [ ] Minimum audit retention
    * [ ] Both async and sync auditing
  * [ ] PII and secrets auto hiding in the logs
    * [ ] Discard secret data when running in production mode
    * [ ] Store secrets in separate output targets.
  * Multiple output formats:
    * [ ] JSON
    * [ ] Human readable
    * [ ] Colorful human readable
  * Multiple outputs targets:
    * [ ] Stdout
    * [ ] File
    * [ ] SQL DB
    * [ ] MongoDB
    * [ ] SystemD Journal

```go
type FloggerHead interface {
  log.Logger
  Fork() FloggerHead
  Close()
  WriteLogEntry(level LogLevel, wait bool, message string, public_data, private_data map[string]interface{})
  WriteAuditEntry(wait bool, message string, retain_until time.Time, public_data, private_data map[string]interface{}, tags []string)
}

type FloggerBody interface {
  DefaultFormatter() FloggerFormatter
  SetDefaultFormatter(format FloggerFormatter)
  Outputs() []FloggerOutput
  AddOutput(output FloggerOutput)
  DelOutput(output FloggerOutput)
  RootHead() FloggerHead
  Close()
}

type FloggerOutput interface {
  Kind() string // e.g. file, mongo, sql, journald
  MinimumLogLevel() LogLevel
  SetMinimumLogLevel(level LogLevel)
  ConnectionInfo() string
  Formatter() FloggerFormatter // if null, it will output in JSON
  SetFormater(format FloggerFormatter) errot
  AllowSecretData() bool
  SetAllowSecretData(flag bool)
  WriteMessages(msg []FloggerMessage) error
  Close() error
}

// basically turn a FloggerMessage into an UTF8 string
type FloggerFormatter func (msg FloggerMessage) []byte

type FloggerMessage struct {
  Level LogLevel
  IsAudit bool
  RetainUntil time.Time
  Tags []string
  Message string
  PublicData map[string]interface{}
  PrivateData map[string]interface{}
  Caller string
}

func (msg FloggerMessage) ToJsonBytes(include_secret bool) ([]byte, error) {}
func (msg FloggerMessage) ToBsonStruct(include_secret bool) (bson.D, error) {}
```
