### docker的使用学习推荐：

菜鸟教程：https://www.runoob.com/docker/docker-tutorial.html 

网上不知名博客：https://taodaling.github.io/blog/2019/01/19/docker/



### client的作用

 Docker Client是绝大部分用户使用Docker的入口 

 绕过Client，也可以向Deamon发送请求呢

 Client可以访问Deamon的API，那么用户也可以直接访问Deamon的API，而且为了方便二次开发，Docker同时提供了很多语言的SDK供开发者选择，例如Docker Client for python。 

![](/img/cli.png)

### client的入口

```go
//主函数(cli/cmd/docker/docker.go)
func main() {
	dockerCli, err := command.NewDockerCli()
	...
	if err := runDocker(dockerCli); err != nil {
		...
	}
}

//cli的定义：cli/command/cli.go:
type DockerCli struct {
	configFile         *configfile.ConfigFile //~/.docker/config.json文件的配置信息
	in                 *streams.In //stdin
	out                *streams.Out //stdout
	err                io.Writer //stderr
	client             client.APIClient //负责与daemon通讯
	serverInfo         ServerInfo
	clientInfo         *ClientInfo
	contentTrust       bool
	contextStore       store.Store
	currentContext     string
	dockerEndpoint     docker.Endpoint //看起来像daemon的api
	contextStoreConfig store.Config
}

//main函数执行 (cli/cmd/docker/docker.go)
func runDocker(dockerCli *command.DockerCli) error {
	tcmd := newDockerCommand(dockerCli)
    ...
	args, os.Args, err = processAliases(dockerCli, cmd, args, os.Args)
	...
	return cmd.Execute()
}

//runDocker函数执行(cli/cmd/docker/docker.go)
func newDockerCommand(dockerCli *command.DockerCli) *cli.TopLevelCommand {
	...
	cmd := &cobra.Command{
		Use:              "docker [OPTIONS] COMMAND [ARG...]",
		Short:            "A self-sufficient runtime for containers",
		...
        RunE: func(cmd *cobra.Command, args []string) error {
			if len(args) == 0 {
				return command.ShowHelp(dockerCli.Err())(cmd, args)
			}
			return fmt.Errorf("docker: '%s' is not a docker command.\nSee 'docker --help'", args[0])

		}
	}
	...
	commands.AddCommands(cmd, dockerCli) //在cmd所表示的命令下边增加对主执行命令的补充命令
	...
	return cli.NewTopLevelCommand(cmd, dockerCli, opts, flags)
}

// 增加命令(cli/command/commands/command.go)
func AddCommands(cmd *cobra.Command, dockerCli command.Cli) {
	cmd.AddCommand(
		...
        hide(image.NewPullCommand(dockerCli)),
        ...
    )
}    
```

### 例子：docker pull

docker pull命令的作用为：从Docker Registry中下载指定的容器镜像，并存储在本地的Graph中，以备后续创建Docker容器时的使用。

![](/img/pull.jpg)



```go
// pull指令定义(cli/command/image/pull.go)
func NewPullCommand(dockerCli command.Cli) *cobra.Command {
	//pull指令的选项，每个指令的选项规则都不一样
    var opts PullOptions

	cmd := &cobra.Command{
		Use:   "pull [OPTIONS] NAME[:TAG|@DIGEST]",
		Short: "Pull an image or a repository from a registry",
		Args:  cli.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			opts.remote = args[0]
			return RunPull(dockerCli, opts)
		},
	}

	flags := cmd.Flags()
	// 将参数解析的结果，放到options
	flags.BoolVarP(&opts.all, "all-tags", "a", false, "Download all tagged images in the repository")
	flags.BoolVarP(&opts.quiet, "quiet", "q", false, "Suppress verbose output")

	command.AddPlatformFlag(flags, &opts.platform)
	command.AddTrustVerificationFlags(flags, &opts.untrusted, dockerCli.ContentTrustEnabled())

	return cmd
}
// pull的选项，包括地址和拉取方式(cli/command/image/pull.go)
type PullOptions struct {
	remote    string
	all       bool
	platform  string
	quiet     bool
	untrusted bool
}
// pull指令的执行
func RunPull(cli command.Cli, opts PullOptions) error {
	...
	if !opts.untrusted && !isCanonical {
		err = trustedPull(ctx, cli, imgRefAndAuth, opts)
	} else {
		err = imagePullPrivileged(ctx, cli, imgRefAndAuth, opts)
	}
	...
	return nil
}

// pull的执行
func imagePullPrivileged(ctx context.Context, cli command.Cli, imgRefAndAuth trust.ImageRefAndAuth, opts PullOptions) error {
	...
	responseBody, err := cli.Client().ImagePull(ctx, ref, options)
	...
	return jsonmessage.DisplayJSONMessagesToStream(responseBody, out, nil)
}
```

然后再扩展包里实现了这个接口

```go
// vendor/github.com/docker/docker/client/interface.go
// ImageAPIClient defines API client methods for the images
type ImageAPIClient interface {
	...
	ImagePull(ctx context.Context, ref string, options types.ImagePullOptions) (io.ReadCloser, error)
	...
}
```



### 个人感受：

1.学习源码时不断思考效果会比较好，抱着要自己如何也能写出这样的代码的心态去学习，不局限于代码本身的业务。比如说关于代码能力、书写规范、设计思维、项目架构、数据结构和算法、计网的基础、操作系统和linux偏底层的知识等。尽量避免只关注纯代码流程。

2.源码中冗余代码比较多，如果代码实在过多，可以带着目的性地，为了搞清楚一个模块或者搞清楚一个原理去读

**参考网站：**

cli代码： https://www.cnblogs.com/langshiquan/p/9933624.html

docker架构： https://blog.csdn.net/sdulibh/article/details/84379853

命令行工具：cobra包 https://github.com/spf13/cobra
