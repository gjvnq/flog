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
  * Multiple sinks:
    * [ ] Stdout
    * [ ] File
    * [ ] SQL DB
    * [ ] MongoDB
    * [ ] SystemD Journal

```go
type FloggerHead interface {
  log.Logger
  Fork() FloggerHead // this will log that a new thread was created (and who the parent is)
  Close() // this will log that the thread was destroyed (and who the parent was)
  WriteLogEntry(level LogLevel, wait bool, message string, public_data, private_data map[string]interface{})
  WriteAuditEntry(wait bool, message string, retain_until time.Time, public_data, private_data map[string]interface{}, tags []string)
}

type FloggerBody interface {
  DefaultFormatter() FloggerFormatter
  SetDefaultFormatter(format FloggerFormatter)
  Sinks() []FloggerSink
  AddSink(output FloggerSink)
  DelSink(output FloggerSink)
  ThreadlessHead() FloggerHead // used when you need a global logger, ThreadNum will always be zero
  Close()
}

func NewFlogger(preset_name string) (FloggerHead, FloggerBody) {}

type FloggerSink interface {
  Type string // e.g. file, mongo, sql, journald
  MinimumLogLevel() LogLevel
  SetMinimumLogLevel(level LogLevel)
  ConnectionInfo() string
  Formatter() FloggerFormatter // if null, it will output in JSON
  SetFormater(format FloggerFormatter) errot
  IncludeSecretData() bool
  SetIncludeSecretData(flag bool)
  WriteMessages(msg []FloggerMessage) error
  Close() error
}

// basically turn a FloggerMessage into an UTF8 string
type FloggerFormatter func (msg FloggerMessage) []byte

type FloggerMessage struct {
  Timestamp time.Time // JSON key: ts
  ThreadNum int // JSON key: tn
  Level LogLevel // JSON key: lvl
  IsAudit bool // JSON key: lvl
  MsgId string // JSON key: id
  RetainUntil time.Time // JSON key: exp
  Tags []string // JSON key: tags
  Message string // JSON key: msg
  PublicData map[string]interface{} // JSON key: data
  FullData map[string]interface{} // JSON key: data
  Caller string // JSON key: caller
  Package string // JSON key: pkg
}

func (msg FloggerMessage) ToJsonBytes(include_secret bool) ([]byte, error) {}
func (msg FloggerMessage) ToBsonStruct(include_secret bool) (bson.D, error) {}
```
