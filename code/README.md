# overflow_with_joy `code/` 사용 가이드

이 디렉터리의 예제들은 이제 32비트 x86 리눅스를 기준으로 정리되어 있습니다.  
빌드와 실행은 `code/` 디렉터리에서 진행하면 됩니다.

## 1. 준비물

- `gcc-multilib`
- `libc6-dev-i386`
- `gdb`
- `binutils` (`nm`, `objdump` 용도)

Ubuntu / Debian 예시:

```bash
sudo apt update
sudo apt install gcc-multilib libc6-dev-i386 gdb binutils
```

## 2. 빌드

```bash
cd /home/isl_hwpark/overflow_with_joy_hw/code
chmod +x compile-all.sh makeshellcode.sh
./compile-all.sh
```

`compile-all.sh`가 하는 일:

- 모든 예제를 `-m32`로 빌드
- `-fno-stack-protector`, `-z execstack`, `-fno-pie`, `-no-pie`, `-O0` 적용
- 가능하면 ASLR 비활성화

`gcc -m32`가 실패하면 32비트 개발 패키지가 아직 설치되지 않은 상태입니다.

## 3. shellcode 생성

`hackme1` / `hackme5`에 쓰는 `/bin/sh` shellcode는 `shellcode-creator.c`에 들어 있습니다.

```bash
./makeshellcode.sh
```

`hackme4`용 `"hacked!\n"` 출력 shellcode는 `shellcode.bin`에 들어 있습니다.

## 4. 실행 예시

### Hackme 1

```bash
./exploit1
```

프로그램 내부에서 32비트 `execve("/bin/sh")` shellcode를 바로 실행합니다.

### Hackme 2

```bash
./hackme2 "$(./exploit2)"
```

`exploit2`는 `password_buffer[16]` 뒤의 `correct` 값을 덮을 수 있도록 20바이트를 출력합니다.

### Hackme 3

기본 사용:

```bash
./hackme3 "$(./exploit3)"
```

`exploit3`는 기본적으로 `./hackme3`에서 `jackpot` 심볼 주소를 읽어와서 payload를 만듭니다.  
직접 주소를 넘기고 싶다면:

```bash
./hackme3 "$(./exploit3 0x080491d6)"
```

### Hackme 4

환경 변수에 shellcode를 넣고 리턴 주소를 그쪽으로 덮습니다.

```bash
export SHELLCODE=$(cat shellcode.bin)
./hackme4 "$(./exploit4 SHELLCODE)"
```

기본 보정값은 `exploit4`와 `hackme4`의 프로그램 이름 길이 차이를 기준으로 잡혀 있습니다.

### Hackme 5

기본 사용:

```bash
./hackme5 "$(./exploit5)"
```

스택 주소가 다르면 리턴 주소를 직접 넘길 수 있습니다.

```bash
./hackme5 "$(./exploit5 0xffffd6c0)"
```

`exploit5`는 NOP sled + 32비트 `/bin/sh` shellcode + 4바이트 리턴 주소를 출력합니다.

### Hackme 6

```bash
./hackme6 "$(./exploit6)"
```

heap overflow로 `public.txt` 대신 `secret.txt`를 읽게 만듭니다.

## 5. 참고

- `hackme2`, `hackme3`는 오버플로우 대상이 더 명확하게 보이도록 로컬 상태를 구조체로 묶어 두었습니다.
- `exploit3`는 하드코딩 주소 대신 `nm`으로 심볼 주소를 읽습니다.
- `exploit5`의 기본 리턴 주소는 예시값입니다. 환경이 다르면 `gdb`로 `buf` 근처 주소를 보고 조정하는 편이 안전합니다.
