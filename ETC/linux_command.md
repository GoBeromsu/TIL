# Linux Commands

[42](https://www.42.us.org/)에서 내준 과제들 중 리눅스를 다루는 것들이 있었다. 문자열 처리 등을 해보면서 어려웠던 것을 정리한다. 이후 차차 차근차근 조금씩 정리해나갈 예정

## 1. 구분 없이 잡다하게

- `touch -t MMDDhhmm` : 해당 파일 혹은 디렉토리의 timestamp를 특정 월, 일, 시간, 분으로 지정해줄 수 있다. 링크 파일을 변경하려면 `-h` 옵션을 추가해서 해야 함.
- `ln -s 링크할파일 새로만들어질링크파일`: symbolic link를 만든다. 
- `man command` : 커맨드에 대해서 알고싶을 때 manual이라는 의미를 가진 man 커맨드를 사용한다. 다음에 알고싶은 커맨드를 넣으면 설명 페이지가 뜬다. q로 빠져나올 수 있다.
- LDAP(The Lightweight Directory Access Protocol)이라는 것을 이용해서 서버에 저장돼있는 데이터를 필터링해서 불러오는 문제였다.
    + `ldapsearch` : command다. ldap에서 원하는 것을 검색하는 것
    + `ldapwhoami -Q`: 현재 유저의 dn(distinguished name)을 출력해준다. -Q는 불필요한 SASL 문구들을 없애준다.
    + `-x` : 간편한 결과를 얻게하는 옵션
    + `"(uid=z*)"` : 필터다. uid 값이 z로 시작하는 것을 고른다.
    + `|` : 파이프는 왼쪽의 결과를 우측 커맨드의 standard input으로 넘겨준다는 의미
    + `grep cn:` : `cn:`라는 문자열이 들어가있는 모든 라인을 찾는다.
    + `sort -rf` : 정렬 커맨드. r 옵션은 reverse order, f 옵션은 case insensitive
    + `cut -c 5-` : 문자열 자르기 커맨드. 5번째 글자 이후의 것만 잘라서 리턴. 그냥 숫자만 쓰면 그 문자만 가져오고, -5처럼 대시를 앞에 쓰면 거기까지만 잘라낸다.

    ```sh
    ldapsearch -x "(uid=z*)" | grep cn: | sort -rf | cut -c 5- 
    ```

- 현재 디렉토리의 파일들을 콤마로 구분해서 출력해보는 문제
    + (수정) 그냥 `ls -mpU` 하면 된다. manual을 잘 읽자.
    + `ls -trF` : `t`는 수정 시각이 빠른 순으로 정렬, `r`은 reverse order, `F`는 디렉토리 뒤엔 슬래쉬(/), 실행 가능한 파일 뒤엔 * 등의 문자를 표시해주는 옵션이다.
    + `tr '\n' ','` : python의 `maketrans`와 비슷하다. 왼쪽의 문자를 오른쪽의 문자로 대치시킨다. 다만 한글자를 두 글자 이상으로 바꿀 순 없다. 1:1로 바뀐다.
    + `sed 's/,$//'` : replace 커맨드다. 대치가 아니라서 글자 수 제한이 없다. `/`가 세 개 들어가고 사이 사이에 패턴과 바뀔 문자열이 들어간다. 예제는 마지막에 들어가는 콤마를 아예 없앤다는 의미다. 그리고 sed로는 개행문자 등 백슬래시나 wild 카드 문자가 들어간 놈들을 잡기가 매우 어렵다. 차라리 tr로 잡아서 특정 문자로 바꿔놓고 그걸 다시 sed로 잡아서 다르게 변경하는 방법이 쉽다. 또한 `sed 's/a/b/g'` 코드처럼 맨 뒤에 g를 적어주면 replace all이란 의미다. 안 적으면 첫 번째꺼 하나만 바뀐다.

    ```sh
    ls -trF | tr '\n' ',' | sed 's/,$//'
    ```

- 원본 파일 a와 diff 결과물 sw.diff 파일을 받아서 diff로 비교한 b 파일을 역 계산해내는 문제. 이 문제 풀면서 오픈소스의 기여가 어떻게 이뤄지는지 알 수 있었다. 문제 풀 때 참고한 [kldp wiki](https://wiki.kldp.org/wiki.php/DiffAndPatch)
    + `diff a b` : a, b 파일의 전체 내용이 아니라 다른 부분만 보여준다. < 표시는 왼쪽 파일의 내용, > 표시는 오른쪽 파일의 내용이다. 달라진 내용을 보여주기 전에 설명하는 한 줄이 추가되는데 `2d1`이라면 왼쪽 파일의 2번째 줄에서 한 줄을 삭제(delete)했다는 의미고, `6c5,7`이라면 왼쪽 파일의 6번째 줄을 아래 내용으로 바꾸는데(change) 그 결과가 5번째부터 7번째 라인까지 들어간다는 의미다.
    + `patch -p 0 a sw.diff` : sw.diff의 내용을 a에 patch한다는 의미. `diff a b`의 결과물이니 a에 적용하면 b가 된다.

- `~`로 끝나거나 or `#`으로 시작하거나 or `#`으로 끝나는 파일들을 찾고, 지우는 스크립트를 작성한다.
    + `find ./` : 현재 디렉토리부터 하위 디렉토리까지 모든 디렉토리와 파일을 나열한다.
    + `-name '*~'` : name 옵션으로 찾을 파일 이름을 지정할 수 있다.
    + `-delete`, `-print` : 파일을 찾으면 지우고, 출력하는 옵션이다.
    + `-o` : 옵션들을 or로 이을 수 있다.

    ```sh
    find ./ -name '*~' -delete -print -o -name '*#' -delete -print -o -name '#*' -delete -print
    ```

- Shell Environment variable. 환경변수를 설정하고 스크립트에서 활용하는 것을 익히는 문제다.
    + `export myvar=value` : 왼쪽 예처럼 만든다. 환경변수 이름과 값과 등호는 띄워쓰기 없이 붙여써야한다. 이후엔 해당 터미널 세션에 한해서 `$myvar` 형태로 불러쓸 수 있다.
    + `groups userid` : 지금은 쓰지 않는 커맨드. 그냥 쓰면 현재 유저가 속한 그룹을 보여주고, 뒤에 유저명을 지정해주면 그 유저에 대한 그룹을 보여준다. 아래 예제 코드에선 FT_USER라는 환경변수에 대한 결과를 사용했다.
    + `sed 's/ /,/g'` : 공백을 콤마로 모두 바꾼다.

    ```sh
    groups $FT_USER | sed 's/ /,/g'
    ```

- 현재 디렉토리(하위 디렉토리 포함)에서 `.sh`로 끝나는 파일을 모두 찾아서 파일명만 한 줄 씩 출력하는 문제
    + `find ./ -type f -exec basename {} \;` : `find ./`는 현재 디렉토리 이하 모든 디렉토리와 파일을 다 고르는 것. 근데 `-type f` 옵션을 주면 디렉토리는 제외하고 파일만 찾는다. `-exec basename {} \;` 옵션은 결과에서 `./` 표시되는 부분을 제외하고 파일명만 출력해준다.
    + `grep .sh$` : .sh로 끝나는 부분만 찾았다.
    + `sed 's/.sh//'` : 뒤 확장자를 없앴다.

    ```sh
    find ./ -type f -exec basename {} \; | grep .sh$ | sed 's/.sh//'
    ```

- 현재 디렉토리에서(하위 디렉토리 포함) 모든 파일과 디렉토리의 개수를 출력하는 문제.
    + `wc -l`: 행의 개수를 센다
    + `sed -e 's/^[ \t]*//'` : 맨 처음 튀어나오는 공백 제거

    ```sh
    find . | wc -l | sed -e 's/^[ \t]*//'
    ```

- Mac address 모두 라인 구분해서 출력
    + ifconfig로 정보를 나열하고
    + 앞에 탭으로 구분된 ether 부분을 잡아낸다.
    + 앞의 공백과 ether 부분을 잘라내고 그 이후 것을 출력

    ```sh
    ifconfig | grep '\tether' | cut -c 8-
    ```

- `"\?$*'KwaMe'*$?\"` 이름의 파일 생성하기. 알파벳을 제외한 모든 문자 앞에 백슬래시로 escape 시키면 된다.
- `ls -l`의 결과를 한 줄씩 더 띄워서 출력하기
    + bash의 while문을 사용했다. 한 라인씩 읽어서 다시 출력하는데 뒤에 개행문자를 두 개 더 붙여서 더 띄워지게 함.

    ```sh
    ls -l | while read line; do echo -n "$line\n\n"; done
    ```

- 문제가 복잡하다. 문자열을 조작해서 주석을 제외하고 뒤집고 역순으로 정렬하고 특정 라인들만 뽑아서 콤마로 구분해서 출력해야함.
    + `cat /etc/passwd` : 해당 파일을 출력하고 이걸 조작해야함.
    + `grep :` : 주석에 콜론이 없고 나머지엔 모두 있어서 이렇게 잡아냈다.
    + `cut -d ':' -f1` : 콜론을 기준으로 첫 번째 단어를 잡아내는 것.
    + `rev`: 문자들을 역으로 재배치한다.
    + `sort -r`: 라인들을 역순으로 정렬
    + `awk -v start=$FT_LINE1 -v end=$FT_LINE2 'NR==start,NR==end'` : 라인을 고를 수 있다. 명령어 내부에서 variable을 정할 수 있는데 쉘의 환경변수를 사용하려면 이렇게 내부 변수로 지정해줘야한다. 뒤에 문자열에서 NR에 값을 지정할 때 쉘 환경변수를 집어넣긴 어렵다. start부터 end까지 번호의 라인들만 골라낸다.
    + `tr '\n' ',' | sed 's/,/, /g' | sed 's/, $/./'` : sed로 개행문자가 잘 안잡혀서 tr로 먼저 개행문자를 콤마로 바꾸고, sed로 콤마를 ', ' 공백 포함하도록 바꾸고, 마지막 끝 부분의 콤마와 공백은 온점으로 바꿨다.

    ```sh
    cat /etc/passwd | grep : | cut -d ':' -f1 | rev | sort -r | awk -v start=$FT_LINE1 -v end=$FT_LINE2 'NR==start,NR==end' | tr '\n' ',' | sed 's/,/, /g' | sed 's/, $/./'
    ```

- `'\"?!` 베이스의 문자열이 들어있는 환경변수 `FT_NBR1`과 `mrdoc` 베이스의 문자열이 들어있는 환경변수 `FT_NBR2`가 있다. 결국 이건 표현방법이 베이스에 따라 다를 뿐 숫자다. 이걸 해당 베이스의 숫자로 변환해서 더한 수를 `gtaio luSnemf` 베이스로 변환했을 때 어떤 글자가 나오는지 출력하는 문제다.
    + `echo $FT_NBR1 + $FT_NBR2` : 결국 합해야하므로 왼쪽과 같이 출력한다. +도 그대로 출력된다. 아직 계산되지 않았다.
    + `sed 's/\\/1/g' | sed 's/?/3/g' | sed 's/!/4/g' | sed "s/\'/0/g" | sed "s/\"/2/g"` : 첫 번째를 숫자로 바꿨다. 결국 5진법이다.
    + `tr "mrdoc" "01234"` : 두 번째를 숫자로 바꿨다. 역시 5진법이고 특수문자가 안들어가서 좀 더 쉽게 변형.
    + `xargs echo "ibase=5; obase=23;"` : 앞에서 전달받은 입력과 함께 ibase, obase 문자열을 추가로 출력한다. 예를 들면 '123 + 342 ibase=5; obase=23;' 형태다.
    + `bc` : eval같은 놈이다. 앞에서 받은 문자열들을 그대로 계산한다. ibase는 input, obase는 output base라 생각하면 된다. 위 예에서 해당 수들을 5진법으로 계산해서 합하고, 출력은 23진법으로 하면 된다.
    + `tr "0123456789ABC" "gtaio luSnemf"` : 마지막으로 13진법처럼 수치를 바꿔주면 끝.

    ```sh
    echo $FT_NBR1 + $FT_NBR2 | sed 's/\\/1/g' | sed 's/?/3/g' | sed 's/!/4/g' | sed "s/\'/0/g" | sed "s/\"/2/g" | tr "mrdoc" "01234" | xargs echo "ibase=5; obase=23;" | bc | tr "0123456789ABC" "gtaio luSnemf"
    ```

## 2. 권한 주기

[참고 링크](http://ganus-textcube.blogspot.com/2010/03/chmod-사용법.html)

- `chmod 741 filename` 형태로 주면 된다.
- 숫자는 자리수 순서대로 사용자, 그룹, 타인을 의미한다.
- read, write, execute 순서대로 4, 2, 1 숫자가 배정되고 숫자를 합하면 된다. 예를 들어 read, write 권한만 주고 싶으면 6이고, write, execute 권한만 주고싶으면 3인 식이다.
- 사용자는 100의 자리, 그룹은 10의 자리, 타인은 1의 자리로 계산해서 더한다.
- `-R` 옵션을 가장 많이 써서 하위 디렉토리 전체에 적용되게 하는 역할을 한다.





