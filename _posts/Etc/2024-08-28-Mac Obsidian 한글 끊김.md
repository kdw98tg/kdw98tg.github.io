---
title: "[Mac] 맥에서 Obsidian 에디터 한글 작성 시 끊기는 현상 해결"

categories:
  -  etc
  
tags:
  - [Obsidian, Electron]

toc: true
toc_sticky: true

published: true

date: 2024-08-29
last_modified_at: 2024-08-29
---

## 맥북에서 obsidian 사용 시 한글이 끊기는 문제 해결법

맥에서 `Obsidian`툴 사용 시 한글이 계속 끊기는 문제가 발생하였습니다. 저는 깃헙 블로그를 작성할 때 `Obisidian`으로 편집을 하기 때문에 굉장히 불편하였습니다. 

그렇게 자료를 찾던 중 어떤 게시글의 댓글에서 해결책을 찾았습니다.

`Obisidian`은 Electron 이라는 프레임워크로 만들어 졌고, 이녀석들에게는 특징이 있다고 합니다.
어떠한 캐시 폴더가 만들어지는데, 그것이 한글을 끊기게 하는 주범이라고 합니다.

그 캐시파일의 경로는 다음과 같습니다.
```bash
~/Library/Application\ Support
```

Terminal로 지우는 것을 해보면
```bash
rm -rf ~/Library/Application\ Support/obsidian/Cache/Cache_Data/
```

이렇게 입력해 주시면 됩니다.

저는 이렇게 하고 나서 말끔하게 문제가 해결 되었습니다.
