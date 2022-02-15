- 環境變數
	~~~
	GOPATH = f:\server
	~~~
	
- golang資料夾結構
	~~~
	server //ex: f:\server
	- bin
	- pkg
	- src
	  - server
	    - cg
	      - base
	      - gameServer
	      - dataServer
	      - walletServer
	~~~

- 插件
	~~~
	go get -v github.com/ramya-rao-a/go-outline
	vscode go 擴充插件
	~~~

- 專案初始化
	~~~
	go mod init server
	~~~

- 基本指令
	~~~
	go build xxx.go	
	go build -o xxx.exe xxx.go
	go run xxx.go
	go get github.com/xxx
	~~~
	