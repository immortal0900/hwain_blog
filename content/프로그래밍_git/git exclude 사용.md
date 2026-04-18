---
생성날짜:
- 2026-02-03 17:40
마지막수정날짜:
- 2026-02-03-화요일 17:40
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
Windows CMD에서는 `echo` 명령이 따옴표까지 포함해서 출력되기 때문에 문제가 발생합니다. 다음 방법들을 시도해보세요:[[stackoverflow](https://stackoverflow.com/questions/46383506/add-files-to-gitignore-directly-from-git-shell)]​

## Windows CMD에서 올바른 방법

bash

`# 방법 1: 따옴표 없이 사용 echo 파일명.txt > .git\info\exclude echo *.log >> .git\info\exclude echo node_modules/ >> .git\info\exclude`

Windows에서는 슬래시(`/`) 대신 백슬래시(`\`)를 사용해야 합니다.[[stackoverflow](https://stackoverflow.com/questions/46383506/add-files-to-gitignore-directly-from-git-shell)]​

## PowerShell에서 사용하는 방법

powershell

`# 방법 2: PowerShell 사용 (더 권장) Add-Content -Path .git\info\exclude -Value "파일명.txt" Add-Content -Path .git\info\exclude -Value "*.log" Add-Content -Path .git\info\exclude -Value "node_modules/"`

또는:

powershell

`# 여러 줄 한번에 추가 @" *.log node_modules/ .env __pycache__/ "@ | Out-File -FilePath .git\info\exclude -Encoding UTF8`

## 텍스트 에디터 사용 (가장 확실한 방법)

bash

`# 방법 3: 메모장으로 직접 편집 notepad .git\info\exclude # 또는 VS Code 사용 code .git\info\exclude`

메모장이나 에디터가 열리면 무시할 파일 패턴을 직접 입력하고 저장하세요.[[stackoverflow](https://stackoverflow.com/questions/43593697/how-do-i-create-add-to-a-git-info-exclude-file-to-ignore-files-locally)]​

## 확인 방법

bash

`# 파일 내용 확인 (CMD) type .git\info\exclude # 파일 내용 확인 (PowerShell) Get-Content .git\info\exclude`

Windows CMD의 `echo`는 따옴표를 문자 그대로 출력하므로, 따옴표 없이 사용하거나 PowerShell/에디터를 사용하는 것이 더 안전합니다.[[stackoverflow](https://stackoverflow.com/questions/46383506/add-files-to-gitignore-directly-from-git-shell)]​