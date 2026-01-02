---
{"author":"jx2lee","aliases":null,"created":"2025-07-05T19:02:37.000+09:00","last-updated":"2025-07-05 19:07","tags":null,"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":false,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/etc/__/language/python-vscode-pyright/","dgPassFrontmatter":true,"noteIcon":""}
---



### vscode 에서 Go to Symbol 로 심볼 검색 시 설치된 외부 패키지로 스코프를 포함하는 방법

```toml
[tool.pyright]
include = [
    "**/.venv",
]
exclude = [
    "**/node_modules",
    "**/__pycache__",
]
venvPath = "."
venv = ".venv"
reportShadowedImports = false
```


### import 하지 않은 패키지의 함수/클래스를 사용하고 싶을 때 검색하지 못하는 경우

Pylance 를 사용하는 경우 User Settings 에 패키지 검색 인덱스 옵션을 추가한다.
```json
    "python.analysis.packageIndexDepths": [
        {
            "name": "",
            "depth": 4,
            "includeAllSymbols": true
        }
    ],
```

### references
- https://github.com/microsoft/pylance-release/issues/5514
- https://github.com/microsoft/pylance-release/blob/main/FAQ.md#how-can-i-increase-the-number-of-indexed-symbols-for-a-package
- https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance