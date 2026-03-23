---
title: "Mac Automator를 사용한 파일 확장자 변환"
date: 2025-03-24 00:00:00 +0900
categories: [implementation, tools]
tags: [automator, file extension]
---

자사 기업의 Cafe24 쇼핑몰 운영 내 썸네일 교체 작업에서 일반적으로 웹상에 널리 존재하는 이미지 변환 사이트를 이용하지만, 대부분 이용 횟수에 제한이 있다는 문제를 겪는다.

작업 중 CTO님이 하셨던 이야기가 생각나 macOS의 Automator를 사용해 보게 되었다.

Mac Automator란 macOS의 기본 앱으로, 특정 폴더에서 발생하는 이벤트(파일 추가, 수정 등)에 따라 지정된 자동화 작업을 실행하는 '폴더 액션' 기능을 제공.  
이를 통해 이미지 변환, 이름 변경, 파일 이동 등의 반복적이고 단순한 업무를 자동화할 수 있다.

macOS Tiger(10.4)부터 Automator가 기본 포함되어 있었고, 최근 macOS Ventura(13)부터는 "단축어(Shortcuts)" 앱이 기본 자동화 도구로 자리잡아 Automator가 점차 대체되는 추세이지만, 셸 스크립트와 같이 복잡하거나 세부적인 작업들은 아직도 Automator만이 수행할 수 있어 여전히 활용 가치가 있다.

Cafe24 작업에서 자동화에 유용했던 3가지 사례와, 폴더 액션 Automator 설정 방법 및 구체적인 코드 구현 방법을 작성한다.

이는 **각각 이미지 용량의 압축**, **.webp로의 변환**, **.gif로의 변환**이다.

## 기본 진행 방식

폴더 액션을 적용할 폴더를 적절한 위치에 적절한 작명으로 생성한다.

기본적으로 경로가 바뀌면 적용된 Automator가 풀려버린다.  
작명의 경우는 그렇지 않다고 들었는데, 나의 경우는 Automator가 해제되었다. 한번에 잘 만들어놓자.

 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FVGSCu%2FbtsMUN0AGC5%2FAAAAAAAAAAAAAAAAAAAAAJ5ky793KsCXihdvK2pTJVbgziq5tmG3WLUIn8muZ2Xg%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3DwZ1nn%252Bmzhs4CQFlJyh84I8z1wSU%253D){: .normal}  
macOS Tiger (10.4)부터 존재하는 기본 앱 Automator를 실행한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fbgwu9u%2FbtsMUl4pqdk%2FAAAAAAAAAAAAAAAAAAAAAOmWVMCZANwauWMSAtcZDgK84HEGUPOQb29a6KosQGo1%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3DPEaBes%252Br8Mclv27Px7DZJR9qE8c%253D){: .normal}
폴더적용 스크립트(=Folder Action).

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FlkWYN%2FbtsMUsI7gVJ%2FAAAAAAAAAAAAAAAAAAAAADb7VbX93mUAvrxEFW0EsMfGlXZ3CJWprn481UCabXD0%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3D%252B0%252F5Qa60uRldy5cK2FGpFbrHYHE%253D){: .normal}

1) 상단 폴더 선택에서 폴더 액션을 취할 폴더를 선택.

2) 선택된 Finder 항목 가져오기(=Get Selected Finder Items)  
셸 스크립트 실행(=Run Shell Script)을 추가. (실행 순서에 영향을 주기 때문에 적절히 위치시키기)

3) 통과 입력(=Pass input)은 인수(=as arguments)로 변경.

4) 후술하는 적절한 변환 스크립트를 입력한 후, 저장 및 종료(그냥 실행 후 이름 저장 아무렇게나 하고 닫으면 된다.)

Spotlight(cmd + space)에서 'folder actions setup'를 입력하여 기록된 폴더 액션들 확인이 가능하다.


## 사용된 실제 코드

### 이미지 용량 압축 코드

```
# 압축 코드
for f in "$@"; do
    # 원본 파일 크기 확인
    file_size=$(stat -f%z "$f")

    # 1.5MB 이하라면 압축하지 않고 건너뛰기
    if [ "$file_size" -le 1500000 ]; then
        echo "⚠️ 이미 1.5MB 이하입니다. 압축을 건너뜁니다: $f"
        continue
    fi

    # 원본 파일명 유지
    output_file="${f%.*}.${f##*.}"

    # 임시 파일명을 만들어서 원본을 덮어쓰지 않도록 함
    tmp_file="${f%.*}_tmp.${f##*.}"

    # 1차 압축 (해상도 80% 축소, 품질 조정)
    /opt/homebrew/bin/ffmpeg -i "$f" -vf "scale=iw*0.9:ih*0.9" -qscale:v 3 "$tmp_file"

    # 변환 성공 여부 확인
    if [ ! -f "$tmp_file" ]; then
        echo "❌ 첫 번째 압축 실패: $f"
        continue  # 변환 실패 시 다음 파일로 넘어감
    fi

    # 변환 후 파일 크기 확인
    file_size=$(stat -f%z "$tmp_file")

    # 파일 크기가 1.5MB 초과하면 추가 압축 반복
    while [ "$file_size" -gt 1500000 ]; do
        tmp_file_next="${f%.*}_tmp2.${f##*.}"
        /opt/homebrew/bin/ffmpeg -i "$tmp_file" -vf "scale=iw*0.9:ih*0.9" -qscale:v 4 "$tmp_file_next"

        # 압축이 실패했다면 루프 중단
        if [ ! -f "$tmp_file_next" ]; then
            echo "❌ 추가 압축 실패: $tmp_file"
            break
        fi

        # 기존 임시 파일 삭제 후 새 파일 교체
        rm -f "$tmp_file"
        mv "$tmp_file_next" "$tmp_file"

        # 새로운 파일 크기 확인
        file_size=$(stat -f%z "$tmp_file")

        # 1.5MB 이하로 줄어들면 중단
        if [ "$file_size" -le 1500000 ]; then
            break
        fi
    done

    # 최종 압축된 파일을 원래 파일명으로 저장
    mv "$tmp_file" "$output_file"
done
```
기본적으로 존재하는 반복 압축 코드를 개선하여 용량을 원하는 기준 근처까지만 압축하며,  
파일명을 그대로 유지할 수 있는 로직을 추가했다.

### .webp 확장자로 변환하는 코드

```
for f in "$@"; do  
    # 원본 파일명 유지 + 변환된 파일을 "다운로드" 폴더에 저장
    output_file="$HOME/Downloads/$(basename "${f%.*}").webp"

    # 이미지 퀄리티 100으로 변환
    /opt/homebrew/bin/cwebp -q 100 "$f" -o "$output_file"

    # 변환 성공 여부 확인 후 원본 삭제
    if [ -f "$output_file" ]; then
        echo "✅ 변환 성공: $output_file"
        rm -f "$f"
    else
        echo "❌ 변환 실패: $f (원본 삭제 안 함)"
    fi
done
```

### .gif 확장자로 변환하는 코드

우선 터미널에서 ffmpeg와 gifsicle을 다운로드 한다.
(각각 비디오 변환 도구, GIF 최적화 도구)

`brew install ffmpeg gifsicle`
```
# 환경 변수 PATH 설정
export PATH=$PATH:/usr/local/bin:/opt/homebrew/bin

# ffmpeg와 gifsicle 경로 확인
FFMPEG_PATH=$(which ffmpeg)
GIFSICLE_PATH=$(which gifsicle)

if [ -z "$FFMPEG_PATH" ] || [ -z "$GIFSICLE_PATH" ]; then
    echo "ffmpeg 또는 gifsicle을 찾을 수 없습니다."
    echo "설치하려면: brew install ffmpeg gifsicle"
    exit 1
fi

for f in "$@"
do
    # 파일이 비디오 파일인지 확인
    if file -b --mime-type "$f" | grep -q "video"; then
        filename=$(basename "$f")
        dirname=$(dirname "$f")
        output_file="${dirname}/${filename%.*}.gif"
        
        echo "변환 중: $f -> $output_file"
        
        # 디버그 정보 추가
        "$FFMPEG_PATH" -v debug -i "$f" -vf "fps=10,scale=480:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -f gif - | "$GIFSICLE_PATH" --optimize=3 > "$output_file"
        
        # 파일 생성 확인
        if [ -s "$output_file" ]; then
            echo "변환 완료: $output_file"
        else
            echo "오류: 빈 GIF 파일이 생성되었습니다."
        fi
    else
        echo "$f는 비디오 파일이 아닙니다."
    fi
done
```
 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbKvPsI%2FbtsMT9C3bJ8%2FAAAAAAAAAAAAAAAAAAAAAHAy0suYZi6MboUh9eRQdOP-SSdzvv3HtK9dll26RwBt%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1774969199%26allow_ip%3D%26allow_referer%3D%26signature%3D9Jjl66ZfAQTUH2L67AwC4jPu7og%253D){: .normal}
모두 적절한 결과를 뱉어냄을 확인할 수 있다.