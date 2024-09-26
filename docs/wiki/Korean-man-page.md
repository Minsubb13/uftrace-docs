uftrace supports Korean man pages for each command along with the default English man pages.

The locale setting has to be prepared beforehand as follows.
```
$ sudo locale-gen ko_KR.UTF-8
Generating locales (this might take a while)...
  ko_KR.UTF-8... done
Generation complete.

$ sudo update-locale LANG=ko_KR.UTF-8

$ sudo systemctl reboot

$ locale
LANG=ko_KR.UTF-8
LANGUAGE=
LC_CTYPE="ko_KR.UTF-8"
LC_NUMERIC="ko_KR.UTF-8"
LC_TIME="ko_KR.UTF-8"
LC_COLLATE="ko_KR.UTF-8"
LC_MONETARY="ko_KR.UTF-8"
LC_MESSAGES="ko_KR.UTF-8"
LC_PAPER="ko_KR.UTF-8"
LC_NAME="ko_KR.UTF-8"
LC_ADDRESS="ko_KR.UTF-8"
LC_TELEPHONE="ko_KR.UTF-8"
LC_MEASUREMENT="ko_KR.UTF-8"
LC_IDENTIFICATION="ko_KR.UTF-8"
LC_ALL=ko_KR.UTF-8
```
Then uftrace has to be installed with `DOCLANG=ko` macro during installation.
```
$ sudo make DOCLANG=ko install
```
It will be ready to show Korean man pages for each command.
```
$ man uftrace
UFTRACE(1)                                                  UFTRACE(1)

이름
       uftrace - 프로그램 함수 호출 분석 도구

사용법
       uftrace
       [record|replay|live|report|info|dump|recv|graph|script|tui]
       [options] COMMAND [command-options]

설명
       uftrace  는  COMMAND  에 주어지는 프로그램의 실행을 함수 단위로
       추적(trace)하는 분석 도구이다.  COMMAND 에 주어지는  프로그램은
       -pg   또는   -finstrument-function   로  컴파일된  C  또는  C++
       프로그램이다.  COMMAND 의 대상이 되는 실행 이미지는 이름을 읽을
       수 있도록 (스트립 되지 않은) ELF 심볼 테이블을 필요로 한다.
...
```