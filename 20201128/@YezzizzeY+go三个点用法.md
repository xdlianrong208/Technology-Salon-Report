用于函数有多个不定参数的情况，可以接受多个不确定数量的参数。

```go
func NewDockerCli(ops ...DockerCliOption) (*DockerCli, error) {
	cli := &DockerCli{}
	defaultOps := []DockerCliOption{
		WithContentTrustFromEnv(),
	}
	cli.contextStoreConfig = DefaultContextStoreConfig()
	ops = append(defaultOps, ops...)
	if err := cli.Apply(ops...); err != nil {
		return nil, err
	}
	if cli.out == nil || cli.in == nil || cli.err == nil {
		stdin, stdout, stderr := term.StdStreams()
		if cli.in == nil {
			cli.in = streams.NewIn(stdin)
		}
		if cli.out == nil {
			cli.out = streams.NewOut(stdout)
		}
		if cli.err == nil {
			cli.err = stderr
		}
	}
	return cli, nil
}
```

一个切片添加到另一个切片中

```go
var strss= []string{
    "qwr",
    "234",
    "yui",

}
var strss2= []string{
    "qqq",
    "aaa",
    "zzz",
    "zzz",
}
strss=append(strss,strss2...) //strss2的元素被打散一个个append进strss		
```
