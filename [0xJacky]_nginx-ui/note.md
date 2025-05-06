### 2025.5.6
该项目采用go语言作为后端语言，web框架使用gin。  
我们先从main函数开始阅读。  

```go
func main() {
	appCmd := cmd.NewAppCmd()

	confPath := appCmd.String("config")
	settings.Init(confPath)

	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM)
	defer cancel()

	err := risefront.New(ctx, risefront.Config{
		Run:       Program(ctx, confPath),  // 运行函数
		Name:      "nginx-ui",
		Addresses: []string{fmt.Sprintf("%s:%d", cSettings.ServerSettings.Host, cSettings.ServerSettings.Port)},
		ErrorHandler: func(kind string, err error) {
			if errors.Is(err, net.ErrClosed) {
				return
			}
			logger.Error(kind, err)
		},
		LogHandler: func(logLevel risefront.LogLevel, kind string, args ...any) {
			args = append([]any{kind}, args...)
			logger.Info(args...)
		},
	})
	if err != nil && !errors.Is(err, context.DeadlineExceeded) &&
		!errors.Is(err, context.Canceled) &&
		!errors.Is(err, net.ErrClosed) {
		logger.Error(err)
	}
}
```
