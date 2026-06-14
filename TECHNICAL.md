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

## 컬러 피커

커스텀 컬러 피커는 HSV 값을 기준으로 동작합니다.

- 채도/명도 패널: X축은 채도, Y축은 명도입니다.
- 색상 슬라이더: HSV의 hue 값을 0-360 범위로 조절합니다.
- 값 표시: 선택된 HEX를 RGB, HSL로 변환해 함께 표시합니다.
- 명도 팔레트: 현재 hue/saturation을 기준으로 9단계 lightness 색상을 보여줍니다.
- 스포이드: Chromium 계열 브라우저의 `EyeDropper` API를 지원하면 화면에서 색을 직접 선택할 수 있습니다.

색상 변경은 활성 색상 슬롯의 HEX 입력과 스와치, 미리보기, 다운로드 데이터에 즉시 반영됩니다.

## 자동 팔레트

2색 또는 3색 그라데이션으로 전환하면 첫 번째 색상을 기준으로 연관 색상이 자동 생성됩니다.

| 팔레트 | 생성 방식 |
|--------|-----------|
| 유사색 | 기준 hue 주변의 가까운 hue를 사용 |
| 보색 | 기준 hue에서 180도 회전한 hue를 사용 |
| 삼각 조화 | 기준 hue에서 120도, 240도 회전한 hue를 사용 |
| 명도 변화 | 기준 hue를 유지하고 saturation/lightness를 조정 |

모든 팔레트는 첫 번째 색상 슬롯을 기준색으로 유지합니다.

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
