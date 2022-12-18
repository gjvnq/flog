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
// Use this to wrap secret values
type SecretVal interface{}

type FloggerHead interface {
  log.Logger
  Fork() FloggerHead // this will log that a new thread was created (and who the parent is)
  Close() // this will log that the thread was destroyed (and who the parent was)
  Context() map[string]interface{}
  SetContext(context map[string]interface{})
  WriteLogEntry(level LogLevel, message string, data map[string]interface{})
  WriteAuditEntry(message string, retain_until time.Time, data map[string]interface{}, tags []string)
  StartSpan(event_name string, data map[string]interface{}) FloggerSpan
}

type FloggerSpan interface {
  Data() map[string]interface{}
  SetData(data map[string]interface{})
  AddData(data map[string]interface{})
  Close(err error) // send nil if the event was successful 
  StartSpan(event_name string, data map[string]interface{}) FloggerSpan
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
  Type(context map[string]interface{}) string // e.g. file, mongo, sql, journald
  MinimumLogLevel() LogLevel
  SetMinimumLogLevel(level LogLevel)
  ConnectionInfo() string
  Formatter() FloggerFormatter // if null, it will output in JSON
  SetFormater(format FloggerFormatter) errot
  IncludeSecretData() bool
  SetIncludeSecretData(flag bool)
  WriteMessage(msg FloggerMessage) error
  WriteStartSpan(span FloggerSpanStart) error
  WriteEndSpan(span FloggerSpanEnd) error
  WriteSpan(span FloggerSpanData) error
  Close() error
}

// basically turn a FloggerMessage into an UTF8 string
type FloggerFormatter func (msg FloggerMessage) []byte

type FloggerMessage struct {
  Timestamp time.Time // JSON key: id
  ThreadNum int // JSON key: tn
  Level LogLevel // JSON key: lvl
  IsAudit bool // JSON key: lvl
  MsgId string // JSON key: id
  RetainUntil time.Time // JSON key: exp
  Tags []string // JSON key: tags
  Message string // JSON key: msg
  Context map[string]interface{} // JSON key: ctx
  Data map[string]interface{} // JSON key: data
  Caller string // JSON key: caller
  Package string // JSON key: pkg
}

func (msg FloggerMessage) ToJsonBytes(include_secret bool) ([]byte, error) {}
func (msg FloggerMessage) ToBsonStruct(include_secret bool) (bson.D, error) {}

type FloggerSpanStart struct {
  SpanId string
  ParentId string
  EventName string
  StartTime time.Time
  CallerStart string
  Context map[string]interface{}
  Data map[string]interface{}
}

type FloggerSpanEnd struct {
  SpanId string
  EndTime time.Time
  Duration time.Time
  CallerEnd string
  EndError error
  Result interface{}
}

type FloggerSpanData struct {
  FloggerSpanStart
  FloggerSpanEnd
}

func (msg FloggerSpanStart) ToJsonBytes(include_secret bool) ([]byte, error) {}
func (msg FloggerSpanStart) ToBsonStruct(include_secret bool) (bson.D, error) {}
func (msg FloggerSpanEnd) ToJsonBytes(include_secret bool) ([]byte, error) {}
func (msg FloggerSpanEnd) ToBsonStruct(include_secret bool) (bson.D, error) {}
func (msg FloggerSpanData) ToJsonBytes(include_secret bool) ([]byte, error) {}
func (msg FloggerSpanData) ToBsonStruct(include_secret bool) (bson.D, error) {}
```
