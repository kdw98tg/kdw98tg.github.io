---
title: "[Unity] Unity에서 협업 시 Addressable 관리 문제"

categories:
  -  Unity
  
tags:
  - [Unity, Addressable, Git]

toc: true
toc_sticky: true

published: true

date: 2024-09-10
last_modified_at: 2024-09-10
---

## Git 을 통해 협업 시 Addressable 에서 발생하는 문제

Addressable Asset을 통해 프로젝트를 관리하고 있을 때, 문제가 발생할 수 있습니다. 이는 유니티의 씬과 비슷한 맥락인 것 같습니다. 유니티의 Scene 은 깃으로 관리하며 협업할 때, 두사람에게 변경사항이 생겼다면, 하나를 Revert 해야 합니다.

Addressable 도 그렇더군요. 그냥  아무 의심 없이 머지를 했는데, 작업내역이 다 날아가 버렸습니다. 아 ㅋㅋㅋ

이렇게되면, 공통으로 작업한 영역중에 하나만 남거나, 둘다 Missing Reference 가 뜨는 경우가 발생합니다.

다음 에러를 뿜으면서 말이죠.

![](/images/Pasted%20image%2020240910095244.png)

```
Rename of Group failed. Cannot move asset from Assets/AddressableAssetsData/AssetGroups/sprite.asset to Assets/AddressableAssetsData/AssetGroups/Sprites.asset. Destination path name does already exist
```

```
The file 'Assets/AddressableAssetsData/AssetGroups/UI.asset' seems to have merge conflicts. Please open it in a text editor and fix the merge.
```

또한, 다른 정상적인 Asset 들과 다르게 빈 페이지 아이콘이 뜨게 됩니다. 이렇게 되면, 남아있는 참조를 없애주고, 다시 Addressable 을 등록해야 합니다.

우선 아래에 보이는 빈 페이지들을 삭제해 줍니다.

![](/images/Pasted%20image%2020240910095310.png)

그 다음으로 `Schemas` 폴더로 들어가 Schema 참조도 없애 줍니다. 그 다음 다시 Addressable 에셋을 만들어서 매핑시켜 줍니다.

![](/images/Pasted%20image%2020240910095335.png)

정말 불편했습니다. 앞으로 작업할 때 Addressable Group 을 나눠서 관리하게 해야 겠습니다 ...