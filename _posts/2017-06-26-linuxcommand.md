---
layout: post
title: 리눅스 명령어정리
date: 2017-06-26 10:04
categories: Unix/Linux
---

# 자주 사용하는 리눅스 명령어 정리

### 1. `mutt` - 서버에서 메일 보내기 기능

- 설치방법 : yum install mutt
- 사용 예 : `mutt -s "제목" "받는이 메일주소" -x -a "첨부파일"

### 2. `zcat` - 압축안풀고 내용보기

같은 명령어 `gzip -d -c`

### 3. `wc -l` - 파일 라인수 세기

```
Usage: wc [OPTION]... [FILE]...
  or:  wc [OPTION]... --files0-from=F
Print newline, word, and byte counts for each FILE, and a total line if
more than one FILE is specified.  With no FILE, or when FILE is -,
read standard input.
  -c, --bytes            print the byte counts
  -m, --chars            print the character counts
  -l, --lines            print the newline counts
      --files0-from=F    read input from the files specified by
                           NUL-terminated names in file F;
                           If F is - then read names from standard input
  -L, --max-line-length  print the length of the longest line
  -w, --words            print the word counts
```

### 4. `sort` - 정렬

- 기본 사용법 : `sort [옵션] "fileName"`
- ex) `sort -k2 -r data` : data파일의 2번째 필드을 기준으로 내림차순 정렬

 **옵션종류**
 
* `-f` => 대소문자를 구분하지않음
* `-r` => 내림차순정렬
* `-k` => 필드 번호를 나타냄
* `-t` => 필드 구분자 지정
* `-n` =>숫자 순서로 정렬

### 5. `file -bi` - 파일 인코딩 확인

* ex) `> file -bi test.txt` => text/html; charset=utf-8

### 6. `iconv` - 파일 인코딩 변환

* ex) `iconv -c -f utf-8 -t euc-kr ttt.php > ttt2.php` : utf-8 인코딩이었던 ttt.php 를  euc-kr 로 변환하여 ttt2.php 로 저장

### 7. bash `^M`문자 제거

`sed -i -e "s/\r//g" fileName`

### 8. `ln -Tfs` - 심볼릭링크 타겟변경

ex) 

```
#심볼릭 링크 설정
ln -s /home/data1 /home/data
lrwxrwxrwx 1 root root    5 12월 13 19:28 data -> data1

#심볼릭 링크 타겟 변경
ln -Tfs /home/data2 /home/data 
lrwxrwxrwx 1 root root    5 12월 13 19:28 data -> data2
```

### 9. `sed` - 스트리밍 편집기

`sed [-e script][-f script-file][file...]`

기본적인 기능은 ed에서 따 왔으며, 이 기능들은 모두 sed에 적용이 된다. 다만 ed는 대화형 편집기이며,
sed는 스트리밍 편집기이다. 대화형 편집기와 스트리밍 편집기의 차이점은 대화형 편집기는 입력 및 출력이
하나로 이루어지며, 스트리밍 편집기는 하나의 입력이 하나의 출력을 낸다는 것이다.
\n 을 개행문자로 사용하는 스트리밍 에디터이다. 

#### 찾기(search), 출력(print)
`sed -n '/abd/p' list.txt` : list.txt 파일을 한줄씩 읽으면서(-n : 읽은 것을 출력하지 않음) abd 문자를 찾으면 그 줄을 출력(p)한다.

#### 치환(substitute)
`sed 's/addrass/address/' list.txt` : addrass를 address로 바꾼다. 단, 원본파일을 바꾸지 않고 출력을 바꿔서 한다. `sed 's/addrass/address/' list.txt > list2.txt`

`sed 's/\t/\ /' list.txt` : 탭문자를 엔터로 변환

`sed 's/□□*/□/' list.txt` : ( *표시: □ 는 공백 문자를 표시한다. ) 위의 구문은 한개이상의 공백문자열을 하나의 공백으로 바꾼다.


#### 추가(insert)
scriptfile - s/

#### 삭제(delete)

`sed '/TD/d' 1.html` : TD 문자가 포함된 줄을 삭제하여 출력한다.

`sed '/Src/!d' 1.html` : Src 문자가 있는 줄만 지우지 않는다.

`sed '1,2d' 1.html` : 처음 1줄, 2줄을 지운다.

`sed '/^$/d 1.html` : 공백라인을 삭제하는 명령이다

#### 파일 이름만을 뽑아내는 정규식

`s/^.*\/\([a-zA-Z0-9.]*\)".*$/\1/` : ^는 라인의 맨 처음, .* 아무문자열, \(, \)은 정규표현식을 그룹화, $ 는 라인의 맨 끝. `( s;^.*\/\([a-zA-Z0-9.]*\)".*$;\1;)` \1는 그룹화된 첫번째 요소를 말한다.

`[a-zA-Z0-9.]` 는 알파벳과 숫자 및 .(콤마)를 표현하는 문자(character)를 말한다.

즉 GF02.jpg와 같은 문자열을 첫번째 그룹화하고 난 다음 라인 전체를 그룹화된 내용으로 바꾸는 것이다. 

`/g` : global을 의미 한줄에 대상문자가 여러개일 때도 처리하도록 한다.

`who | sed -e 's; .*$;;'` : 각 라인의 첫 번째 공백에서부터 마지막까지 삭제하라.

`who | sed -e 's;^.* ;;'` : 각 라인의 처음부터 맨 마지막 공백까지 삭제하라.

`who | sed -e 's;^.*:;;'` : 각 라인의 처음부터 : 문자가 있는 곳(:문자포함)까지 삭제하라. 

#### -n 옵션

sed는 항상 표준 출력에서 입력 받은 각 라인을 나타낸다는 것을 알아냈다. 그러나 때때로 한 파일로부터 몇 개의 라인들을 추출해 내기 위해 sed를 사용하기를 원할 때도 있다. 이러한 경우에 -n옵션을 사용한다. 이 옵션은 사용자가 만약 해야 할 일을 정확히 말해 주지 않는다면 임의의 라인을 프린트하는 것을 원하지 않는다고 sed에게 말한다. 따라서 p명령이 같이 쓰인다. 라인 번호와 라인 번호의 범위를 나타냄으로써 sed를 사용하여 텍스트의 라인들을 선택적으로 프린트할 수 있게 한다. 다음에서 볼 수 있는 바와 같이, 한 파일로부터 첫 번째 두 라인이 프린트되었다.

`$ sed -n '1,2p' intro Just print the first 2 lines from intro file.`

만약 라인 번호 대신에 슬래시로 에워 싸인 문자열과 함께 p명령이 쓰인다면 sed는 이들 문자들이 포함하고 있는 표준 입력을 통해서 라인들을 프린트하게 된다. 따라서 하나의 파일로부터 처음의 두 라인을 프린트하기 위하여 다음과 같이 사용될 수 있다.

`$ sed -n '/UNIX/p' intro Just print lines containing UNIX`

`sed '5d'` : 라인 5를 삭제

`sed '/[Tt]est/d'` : Test 또는 test를 포함하는 모든 라인들을 삭제

`sed -n '20,25p' text` : text로부터 20에서 25까지의 라인들만 프린트

`sed '1,10s/unix/UNIX/g' intro` : intro의 처음 10개의 라인들의 unix를 UNIX로 변경

`sed '/jan/s/-1/-5'` : jan을 포함하는 모든 라인들 위의 첫 번째 -1을 -5로 변경

`sed 's/...//' data` : 각 data라인으로부터 처음 세 개의 문자들을 삭제

`sed 's/...$//' data` : 각 데이터 라인으로부터 마지막 3문자들을 삭제

`sed -n '1' text` : 비 프린트 문자들을 \nn으로 (여기서 nn은 그 문자의 8진수 값임),
그 리고 탭 문자들을 > 로 나타내는 각 텍스트로부터의 모든 라인들을 프린트

