# PCB Vision — GitHub / Netlify 화면 배포

이 폴더 **안의 파일과 public 폴더**를 새 GitHub 저장소의 최상위에 올리세요. 기존 Seongpil_RealPCB 전체를 올릴 필요는 없습니다. 원본 로컬 v3는 그대로 보존했고, 업로드본은 웹용 안내와 API 주소 설정을 분리했습니다.

## 올릴 파일

```text
저장소 최상위/
├── public/
│   ├── index.html       # 최신 v3 화면; CSS·JS·SVG 내장
│   └── config.js        # 공개 HTTPS 검사 서버 주소
├── netlify.toml         # public 폴더를 배포하도록 설정
├── README.md
└── .gitignore
```

## Netlify 설정

GitHub 저장소를 선택한 뒤 아래 값으로 배포합니다.

| 항목 | 값 |
|---|---|
| Base directory | 비워두기 |
| Build command | 비워두기 |
| Publish directory | public |

netlify.toml에도 publish를 넣었습니다. npm 설치, package.json, Python 빌드 명령은 필요 없습니다. ZIP 파일 자체를 GitHub에 올리는 것이 아니라 위 파일과 폴더 구조를 올립니다.

## 화면 배포와 실제 검사는 별도

**이 폴더만 Netlify에 배포하면 화면을 공개할 수 있지만 YOLO·DINO 사진 검사는 아직 실행되지 않습니다.** Netlify Functions의 지원 런타임은 JavaScript/TypeScript 및 Go이며 현재 프로젝트의 Python server.py를 그대로 서비스하지 않습니다. Python을 빌드 과정에서 쓸 수 있다는 사실도 이 검사 서버가 실행된다는 뜻은 아닙니다.

공식 안내: https://www.netlify.com/platform/core/functions/

실제 검사에는 모델을 실행하는 별도 공개 HTTPS 검사 API가 필요합니다. 준비되면 public/config.js에 **기본 주소만** 입력하고 GitHub에 커밋하세요.

```js
window.PCB_API_URL = "https://실제-검사서버-도메인";
```

마지막에 /api를 붙이지 않습니다. 클라이언트가 /api/status, /api/inspect 등을 붙입니다. API 키·비밀번호를 이 파일에 넣지 마세요. config.js는 웹 방문자에게 공개됩니다. 서버 주소가 비어 있으면 상태를 ‘검사 서버 설정 필요’로 표시하고 검사 요청을 보내지 않습니다.

검사 서버는 아래 계약을 지원해야 합니다.

- GET /api/status: ready, error, localization_version: 2 이상
- POST /api/inspect: 기존 JSON {image: base64 또는 data URL, filename, cropped}; 기존 판정·점수·crop·heatmap·regions·localization 응답
- GET /api/examples, GET /api/example/:id: 예시 체험용; 사진 업로드 검사와 별도
- JPG/PNG/WEBP 최대 20MB, 2,400만 화소; JSON base64 확장까지 수용할 요청 제한
- 최종 Netlify 사이트 origin에 대한 CORS 허용 및 OPTIONS 응답

**기존 로컬 server.py는 127.0.0.1에만 바인딩하고 로컬 origin만 허용합니다.** 이를 그대로 서버 호스팅하거나 주소만 바꿔서는 공개 검사가 완성되지 않습니다. 실행 포트/바인딩, 허용 origin, 상대 파일 경로를 서버용으로 조정해야 합니다. 현재 업로드 폴더에는 서버 호스팅이나 CORS 변경을 적용하지 않았습니다.

## 별도 Python 검사 서버에 필요한 원본 파일

아래는 Netlify 화면 폴더에 넣을 파일 목록이 아니라, 별도 검사 서버를 준비할 때의 목록입니다.

| 원본 프로젝트 경로 | 용도 |
|---|---|
| server.py | 로컬 HTTP API; 공개 서버용 바인딩·CORS 수정 필요 |
| runtime.py | YOLO 크롭 → DINO 특징 → 판정·히트맵 |
| engine.py | 특징 추출과 nearest-neighbour 계산 |
| prepare.py | 런타임에서 사용하는 square_crop 함수 |
| localization.py | 의심 위치·우선 확인 위치 |
| requirements.txt | Python 의존성 |
| models/blue_grid.pt | 고정 뱅크·PCA·이미지 판정 기준 |
| models/localization_profile.json | 위치 표시 기준 |
| data/manifest.json | 등록·보정 사진 해시와 역할; 서버 공유용은 절대 경로 정리 필요 |
| data/source_paths.json | YOLO 경로; 현재 Mac 절대 경로이므로 서버 경로로 변경 필요 |
| data/features/dino_448_r0/의 profile.refs 7개 .pt | 타입 구분용 기준 특징 |
| ver5_best.pt | Week2의 ‘욜로 ver5 공유용/.../models/’에 있는 실제 가중치 |
| DINOv2-B 코드·사전학습 가중치 | 공식 Torch Hub 다운로드 또는 서버용으로 고정한 캐시 |

server.py의 예시 API까지 유지하려면 data/quarantine.json과 선택한 원본 예시 사진, 그 사진 경로도 서버에 준비해야 합니다. 전체 Week2/Week3 데이터나 평가 결과는 사진 업로드 검사에 모두 필요한 것은 아닙니다. 가중치를 GitHub에 올릴지는 별도 서버의 저장소·모델 공급 방식에 따라 결정하며, 이 정적 배포본에는 포함하지 않았습니다.

## 업로드하지 않을 것

- 기존 원본/v2/v3의 별도 복사본과 .command 실행 파일
- models, data, reports, diagnostics, next_validation, output, tmp 전체
- 회의록·논문·전체 데이터셋·불량 원본 사진
- .env, 토큰·API 키·비밀번호, 캐시 및 Python 환경

기존 빨강·파랑 히트맵과 번호 상자 동작은 유지합니다. 주황 상자는 국소 의심 위치, 파랑 상자는 우선 확인할 참고 위치입니다. 참고 위치는 정상 사진에도 나타날 수 있으며 실제 결함 정답이나 전체 불량 판정이 아닙니다.
