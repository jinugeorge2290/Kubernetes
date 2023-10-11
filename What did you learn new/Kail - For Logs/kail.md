
## Installation Steps

wget https://github.com/boz/kail/releases/tag/v0.16.1\
tar -xvzf kail_v0.16.1_linux_amd64.tar.gz\
sudo cp kail /usr/local/bin/\
rm -rf kail_v0.16.1_linux_amd64.tar.gz LICENSE.txt README.md


## kail help 
```
[jigeorge@chi3-mgtxjump-1 ~]$ kail --help
usage: kail [<flags>] <command> [<args> ...]

Tail for kubernetes pods

Flags:
  -h, --help                  Show context-sensitive help (also try --help-long and --help-man).
      --ignore=SELECTOR ...   ignore selector
  -l, --label=SELECTOR ...    label
  -p, --pod=NAME ...          pod
  -n, --ns=NAME ...           namespace
      --ignore-ns=NAME ...    ignore namespace
      --svc=NAME ...          service
      --rc=NAME ...           replication controller
      --rs=NAME ...           replica set
      --ds=NAME ...           daemonset
  -d, --deploy=NAME ...       deployment
      --sts=NAME ...          statefulset
  -j, --job=NAME ...          job
      --node=NAME ...         node
      --ing=NAME ...          ingress
      --context=CONTEXT-NAME  kubernetes context
      --current-ns            use namespace from current context
  -c, --containers=NAME ...   containers
      --dry-run               print matching pods and exit
      --log-file=LOG-FILE     log file output
      --log-level=error       log level
      --since=DURATION        Display logs generated since given duration, like 5s, 2m, 1.5h or 2h45m. Defaults to 1s.
  -o, --output=default        Log output mode (default, raw, json, or json-pretty, zerolog)
      --zerolog-timestamp-field="time"
                              sets the zerolog timestamp field name, works with --output=zerolog
      --zerolog-level-field="level"
                              sets the zerolog level field name, works with --output=zerolog
      --zerolog-message-field="message"
                              sets the zerolog message field name, works with --output=zerolog
      --zerolog-error-field="error"
                              sets the zerolog error field name, works with --output=zerolog

Commands:
  help [<command>...]
    Show help.

  run*
    Display logs

  version
    Display current version
    
```