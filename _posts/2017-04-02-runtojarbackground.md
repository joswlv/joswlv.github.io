---
layout: post
title: jar 백그라운드로 실행하기
date: 2017-04-02 09:04
categories: Etc
---

# Jar 백그라운드로 실행하기

```bash
JARFile="KafkaOffsetMonitor-assembly-0.2.1.jar"
DIR="/home1/irteam/seungwan"
function start {
    if pkill -0 -f $JARFile > /dev/null 2>&1
    then echo "Service [$JARFile] is already running. Ignoring startup request/"
      exit 1
    fi
    echo "Staring [$JARFile] Application!!"
    nohup java -cp KafkaOffsetMonitor-assembly-0.2.1.jar com.quantifind.kafka.offsetapp.OffsetGetterWeb --zk tcambkaf-91b901.svr.toastmaker.net,tcambkaf-91b902.svr.toastmaker.net --port 8080 --refresh 10.seconds --retain 2.days> $DIR/log.txt 2>&1 &

}

function stop {
    if ! pkill -0 -f $JARFile > /dev/null 2>&1
    then
        echo "Service [$JARFile] is not running. Ignoring shutdown request."
        return 0
    fi

    echo "Stopping [$JARFile]!!"

    # First, we will try to trigger a controlled shutdown using

    # Wait until the server process has shut down
    attempts=0
    while pkill -0 -f $JARFile > /dev/null 2>&1
    do
        attempts=$[$attempts + 1]
        if [ $attempts -gt 5 ]
        then
            # We have waited too long. Kill it.
            pkill -f $JARFile > /dev/null 2>&1
        fi
        sleep 1s
    done
    echo "Stop [$JARFile] is completed!!"
}

function status {
    if pkill -0 -f $JARFile > /dev/null 2>&1
    then echo "Service [$JARFile] is running."
      exit 1
    fi
    echo "Service [$JARFile] is not running"
}

case "$1" in
  status)
    status
    ;;
  stop)
        stop
    ;;
  start)
        start
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit 1
esac

exit 0
```

* `/dev/null`은 어디에도 콘솔로그를 출력하지 않겠다는 뜻
* `2>&1`은 표준에러는 표준출력에 출력하겠다는 뜻
* `$DIR/log.txt 2>&1`은 표준에러를 파일에 적지 않고 콘솔에 출력하겠다는 뜻 이렇게 2>&1 리다이렉션을 시켜 줌으로 인해 stderr > stdout 으로 출력이 되고 log.txt파일에 기록됨
* `nohup`명령어는 프로세스 중단(hangup)을 무시하고 명령어를 실행한다는 뜻
* `nohup`명령어 끝에 `&`를 추가 하면 백그라운드로 실행
* `> /dev/null 2>&1`도 있는데, 이는 표준 출력을 무시하고, 표준에러만 화면에 출력하겠다는 뜻
* `>`연산자는 redirection을 의미함


### Reference

* [http://reebok.tistory.com/56](http://reebok.tistory.com/56)
* [https://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean/](https://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean/)
