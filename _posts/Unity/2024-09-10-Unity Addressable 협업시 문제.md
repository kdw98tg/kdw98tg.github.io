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

Addressable Asset을 통해 프로젝트를 관리하고 있을 때, 문제가 발생할 수 있습니다.

```
Rename of Group failed. Cannot move asset from Assets/AddressableAssetsData/AssetGroups/sprite.asset to Assets/AddressableAssetsData/AssetGroups/Sprites.asset. Destination path name does already exist
```

```
The file 'Assets/AddressableAssetsData/AssetGroups/UI.asset' seems to have merge conflicts. Please open it in a text editor and fix the merge.
```

![](/images/Pasted%20image%2020240910095244.png)

![](/images/Pasted%20image%2020240910095310.png)


![](/images/Pasted%20image%2020240910095335.png)