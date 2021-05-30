# Bash - use getopts to parse arguments


## Introduction
Python 有 `argparse` 可用，不曉得 Bash 是否有類似的模組能用?  
結果是當然有 `"getopts"`  
事實上，之前也用過一兩次。這次特別把它記錄下來

## getopts
基本上用法如下:
```
getopts optstring optname [ arg ]
```
此篇僅記錄 short option ，若有 long-form (e.g. \-\-path) 的需求，請參考
https://www.shellscript.sh/tips/getopt/  
個人覺得相對複雜些，如果要寫得這麼完整，我可能就用 Python 改寫  
或之後有工作需要，我搞清楚了再來補齊這篇文章



### Example
```bash
#!/usr/bin/bash

a=0; b=0; c=0; d=0;
while getopts ":ab:c:dh" opt;
do
    echo "Process $opt : OPTIND is $OPTIND"
    case ${opt} in
        a)
            a=1 ;;
        b)
            b=$OPTARG ;;
        c)
            c=$OPTARG ;;
        d)
            d=1 ;;
        h | ?)
            echo 'help message'; exit 0 ;;
    esac
done
echo "a = $a"
echo "b = $b"
echo "c = $c"
echo "d = $d"
```
說明一下上述範例內容
* 有 5 個 options (a, b, c, d, h)
  * 其中 b, c 必須提供 argument。因 optstring 中的 b, c 後方有加入 `:` (e.g. `"b:c:"`)
  * 其他 (a, d, h)，我的理解是類似 true/false 的概念
* 不認得的 option ，會被當作 `?` 處理。範例中會顯示 help message
* 第 6 行 `echo "Process $opt : OPTIND is $OPTIND"` 的意思
  * 單純提供額外資訊 `$OPTIND`，不需要可以移除該行
  
</br>
以下為執行結果: 

```cmd
$ bash test.sh -a -b 5566 -d -c 56
Process a : OPTIND is 2
Process b : OPTIND is 4
Process d : OPTIND is 5
Process c : OPTIND is 7
a = 1
b = 5566
c = 56
d = 1
```
```cmd
$ bash test.sh -b 5566 -d -c 56
Process b : OPTIND is 3
Process d : OPTIND is 4
Process c : OPTIND is 6
a = 0
b = 5566
c = 56
d = 1
```
```cmd
$ bash test.sh -A
Process ? : OPTIND is 2
help message
```

### optstring 前是否有 `:` ?
這困擾我一陣子，網路上的範例有些有，有些無，搞不清楚最前面到底要不要加？  
後來看到有人解釋:
* 有 `:` 代表**關閉** default error handling
* 無 `:` 代表**啟用** default error handling
```bash
# disable default error handling
while getopts ":ab:c:dh" opt;

# enable default error handling
while getopts "ab:c:dh" opt;
```

以下是有啟用 default error handling 的範例:
```cmd
$ bash test.sh -A
test.sh: illegal option -- A
Process ? : OPTIND is 2
help message
```
輸入不存在的 option `-A`，會顯示 `test.sh: illegal option -- A` 錯誤訊息  
不需要這訊息的話，就可以選擇關閉 error handling

## Reference
* https://linuxconfig.org/how-to-use-getopts-to-parse-a-script-options
* https://www.shellscript.sh/tips/getopts/
* https://sookocheff.com/post/bash/parsing-bash-script-arguments-with-shopts/
* https://www.computerhope.com/unix/bash/getopts.htm

