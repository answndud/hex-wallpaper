# Hex Wallpaper 기술 문서

## 구조

```text
hex-wallpaper/
├── index.html
├── README.md
└── TECHNICAL.md
```

`index.html` 안에 HTML, CSS, JavaScript가 모두 포함되어 있으며 서버나 빌드 도구가 필요 없습니다.

## 입력 처리

HEX 입력은 다음 형식을 허용합니다.

- `#RRGGBB`
- `RRGGBB`
- `#RGB`
- `RGB`

3자리 HEX는 각 문자를 두 번 반복해 6자리 HEX로 변환합니다.

```javascript
#2A9 -> #22AA99
```

## 렌더링

다운로드 시 임시 Canvas를 만들고 선택된 해상도로 이미지를 그립니다.

- 단색: `fillStyle`에 HEX 색상을 지정해 전체 Canvas를 채웁니다.
- 그라데이션: `createLinearGradient`로 2색 또는 3색 스톱을 추가합니다.

미리보기 영역은 동일한 색상 값을 CSS `background`로 반영합니다.

## 파일 크기 맞춤

Canvas가 생성하는 PNG는 단색과 단순 그라데이션에서 매우 작게 압축됩니다. 1 MB, 2 MB, 3 MB 다운로드 옵션을 맞추기 위해 PNG 파일 안의 `IEND` 청크 앞에 비표준 보조 청크를 삽입합니다.

처리 순서:

1. Canvas를 PNG Blob으로 변환
2. PNG 바이너리에서 `IEND` 청크 위치 탐색
3. 목표 용량까지 필요한 바이트 수 계산
4. private ancillary chunk인 `npAd` 청크 생성
5. 청크 타입과 데이터에 대한 CRC32 계산
6. `IEND` 앞에 청크를 삽입해 새 Blob 생성

이 방식은 이미지 픽셀을 바꾸지 않고 PNG 파일 크기만 늘립니다.

## 제한 사항

- 목표 용량보다 원본 PNG가 이미 크면 패딩하지 않고 원본 PNG를 다운로드합니다.
- 파일 크기 단위는 십진 MB 기준입니다. 1 MB는 1,000,000 bytes입니다.
- 다운로드는 브라우저의 Blob, Object URL, Canvas API 지원이 필요합니다.
