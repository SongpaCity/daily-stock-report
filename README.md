# 데일리 증시 리포트

매일 갱신하는 증시 요약 리포트. GitHub Pages로 자동 배포됩니다.

## 공유 링크 (친구에게 이 주소만 보내세요)

> https://songpacity.github.io/daily-stock-report/

링크는 항상 고정입니다. 내용만 매일 바뀝니다.

## 매일 업데이트 방법

1. Claude로 증시 리포트 HTML을 뽑는다.
2. 이 폴더의 `index.html` 내용을 새 리포트로 교체한다.
   - (팁) `<!-- ↓↓↓ ... ↓↓↓ -->` 주석 사이 영역만 바꾸면 스타일/헤더는 유지됩니다.
3. 커밋 & 푸시:

   ```bash
   cd ~/daily-stock-report
   git add -A
   git commit -m "report: YYYY-MM-DD"
   git push
   ```

4. 약 1분 뒤 위 링크에 반영됩니다.

## (선택) 과거 리포트 보관하고 싶다면

`index.html`을 덮어쓰기 전에 그날 파일을 날짜 이름으로 복사해 두세요:

```bash
cp index.html 2026-10-16.html
```

그러면 `https://songpacity.github.io/daily-stock-report/2026-10-16.html` 로도 접근 가능합니다.
