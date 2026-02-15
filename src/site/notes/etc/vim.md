---
{"author":"jx2lee","aliases":"매번 까먹는 vim 단축키나 팁..","created":"2023-12-20T00:33:04.000+09:00","last-updated":"2025-04-27 21:57","tags":["vim"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":false,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/etc/vim/","dgPassFrontmatter":true,"noteIcon":""}
---


> [!info] 매번 까먹는 vim command 를 정리한다.

NORMAL
- `:<line-number>`: line-number 로 이동한다.
- word 단위 jump
    - forward: `w`
    - backward: `b`
- blackhole registre: `"_x` or `"_y/d/dd`
    - 블랙홀 레지스터 표현은 `_`

INSERT
VISUAL
- 선택한 영역 클립보드에 복사: `"+y`
- 선택한 영역 앞뒤로 쿼터/더블쿼터 추가: `c""<Esc>P`
    - 싱글쿼터인 경우 "" > '' 변경

---
HANDLE files
- 파일로 저장하기: `:w {file_name}`
- 현재 파일을 이용하기: `!uv run %` percent 기호 사용
    - 혹은 `:terminal` or `:term` 으로 새로운 터미널 window 로 결과로 볼 수 있음([vim terminal](https://github.com/vim/vim/blob/master/runtime/doc/terminal.txt), This feature is for running a terminal emulator in a Vim window. A job can be started connected to the terminal emulator. For example, to run a shell)
    - https://stackoverflow.com/a/49658758