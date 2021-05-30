# My Command Line Cheat Sheet



## tar
* create tar.gz
    ```
    $ tar -zcvf tar-archive-name.tar.gz source-folder-name
    ```

* extract tar.gz
    ```
    $ tar -zxvf tar-archive-name.tar.gz
    ```
* create tar.gz (pigz)
    ```
    $ tar cv filename | pigz -6 -p 10 -k > filename.tar.gz
    ```
* decompress
    ```
    $ tar -I pigz -xv filename.tar.gz
    ```

## split
```
$ split -b 200M linux-mint-18.tar.gz "ISO-archive.part"
```
```
$ cat home.tar.bz2.parta* > backup.tar.gz.joined
```

## pcap
* check file type (pcap vs pcapng)
    ```
    $ capinfos -t [file]
    ```
* turn pcapng into pcap
    ```
    $ editcap -F pcap <input-pcapng-file> <output-pcap-file> 
    ```

* change packet's timestamp (reduce 1 hour; 60x60)
    ```
    $ editcap -t -3600 input_dump output_dump
    ```
* install bittwist (centos: build from src code)

* replace old ip (10.0.2.15) with new ip (1.144.118.53)
    ```
    $ bittwiste -I input_dump -O output_dump -T ip -s 10.0.2.15,1.144.118.53 -d 10.0.2.15,1.144.118.53
    ```
* merge 2 pcap files (ctu20.pcap ctu29.pcap)
    ```
    $ mergecap -w <outfile> [<infile> ...i]
    ```
* extract packets between a specific timeperiod 
    ```
    $ editcap -v -A "1970-01-01 08:00:00" -B "1970-01-01 11:00:00" input_dump output_dump
    ```
* split pcap file based on specific time inverval (second)
    ```
    $ editcap -i <seconds> <input_file> <output_prefix>.pcapng
    ```
* scapy
  * https://github.com/secdev/scapy

* IP address display filter
    ```
    $ tshark -r <pcap_file> -T fields -e ip.addr -Y "ip.addr == 1.1.1.1/24"
    ```


## 7zip
```
$ 7za x ?
```

## grep
* Search whether there are flows using  port 514
    ```
    grep '", \(514, [0-9]*\|[0-9]*, 514\)], "byte"' *
    find ./*/* -type f -exec grep -l '", \(514, [0-9]*\|[0-9]*, 514\)], "byte"' {} +
    ```
* -v :invert -E: use regex
    ```
    $ grep -v -E "(xuite|hamicloud|googleapis|hinet)"
    ```


## awk
* split line by "\t"; print all columns (1 to NF)
    ```
    $ cat [log] | awk -F "\t" '{for(i=1;i<NF;i++){printf "%s", $i} {printf "\n"}}'
    ```
* similar to the previous one and use **`","`** as separator  
  * `OFS`: output field separator
  * `ORS`: output record separator; default `"\n"`
    ```
    $ cat [log] | awk -F "\t" 'BEGIN{OFS=","} {printf "%s |", $6; for(i=10;i<=17;i++){printf "%s,", $i}; printf "\n"}'|sort|uniq
    ```
* print all columns except i==2 and i==3
    ```
    $ cat [log] | awk -F "\t" '{for(i=1;i<=NF;i++){if(i!=2 && i!=3){printf "%s", $i}} {printf "\n"}}'
    ```
* print line if column 4 or column 6 match the pattern
    ```
    $ cat [log] | awk -F "\t" "{if (\$4 ~ /$pattern/ || \$6 ~ /$pattern/) print}"
    ```


## tail
* print [file] starting from `3rd` line
```
$ tail -n+3 [file]
```


## sed
* print [file] except line `2` ~ line `5`
```
$ sed -n '2,5d;p' [file]
```


## vim
* format(Indent) the whole file
  * gg: from first line; G: to last line; =: Indent
    ```
    gg=G    
    ```

* format(Indent) line 100 to line 200
    ```
    100G=200G
    ```
