---
title: "zTree 예제 데모 jquery 트리 구현"
excerpt: "트리 구조/폴더 구조/계층형 구조... 형태의 데이타를 다루고 화면에 표현하는 프로젝트"
categories: 
  - toyproject
tags: 
  - toyproject
  - JPA
  - zTree
last_modified_at: 2018-12-11T00:00:00+09:00
toc: true
toc_sticky: true
---

## Intro
트리 구조/폴더 구조/계층형 구조의 데이터를 다루고 zTree라는 plugin을 사용해 이를 UI로 표현하는 프로젝트입니다.  
해당 프로젝트의 github [link](https://github.com/moregorenine/toyproject/tree/master/tree-structure-ztree "github link")  
Demo 테스트 사이트 [link](http://w4springrain.cafe24.com/menus "w4springrain homepage")
특징은 아래와 같습니다.  
- [zTree plugin](http://www.treejs.cn/ "homepage link")은 MIT License라 자유로운 이용이 가능합니다.
- zTree는 순수하게 javascript와 jquery로 만들어진 plugin입니다.
- 심플한 디자인을 지원합니다.
- JPA를 사용하여 DB환경에 제약을 받지 않아 호환성이 뛰어납니다.

## 트리구조 특징

### 설계사상
tree struct를 표현하는 단어는 참 많습니다. 트리 구조, 폴더 구조, 계층형 구조, 이진화 트리형... 이런 구조와 화면UI는 데이터를 체계적이고 직관적으로 관리할수 있도록 사용자에게 많은 도움을 줍니다. 그래서 실제 다양한 프로젝트에서는 메뉴 관리나 공통 코드 관리, 문서 관리... 등등 많은 화면에서 사용되어지고 있습니다. 이를 구현하는 방법도 다양하며 각각의 방법에 따라 장단점이 존재합니다. 아래는 각각의 설계사상과 이에 따른 장단점을 나열합니다.

### Data 구조
트리 구조를 DB로 표현하는 방식은 다양합니다. 그리고 사용하는 DB에 따라 또 다양한 방법으로 구현이 가능합니다. 예를 들어 oracle DB를 사용할 경우 트리 구조를 간단히 표현할 수 있도록 자체적으로 사용하는 함수(connect by)를 이용할 수도 있습니다. 허나 해당 프로젝트는 이런 특정 DB에 종속되는 유형은 배제를 했습니다. JPA를 사용한 이유도 이런 연유입니다. 트리 구조를 표현하기 위해서 데이타 구조를 잡는데 크게 아래의 2가지 방법을 생각했습니다.

#### 엄격한 구조 및 ordering 관리
* logic이 단순하고 읽는 속도가 빠릅니다.
* 트리 구조상에서의 변경이 Save Action 작업단위로 Transaction 관리가 가능합니다.
* All Delete 후 All Insert라 저장 속도가 느립니다.
  * 데이타 변경이 자주 일어나는 경우 추천하지 않습니다.
  * 데이타 변경에 대한 책임유무를 관리해야하는 경우 추천하지 않습니다.

#### linked list 형식의 유연한 관리
* 트리구조 화면에서 일어나는 변경에 대한 처리 logic이 복잡하고 읽는 속도라 느립니다.
* 트리 구조상에서의 변경을 Save Action 작업단위로 Transaction 관리하기 어렵습니다.
  * 이를 해결하기 위해서는 모든 변경 이벤트 히스토리를 추적관리 해야합니다.
* 변경 노드만 update 함으로써 저장 속도가 빠를 수도 있습니다.
  * 절대적인게 아닌 이유가 형제노드들 중 마지막 노드를 최상위로 이동 시킬시 모든 형제 노드들의 ordering이 일어나야 합니다.
  
해당 프로젝트는 엄격한 구조 및 ordering 관리를 선택했습니다. linked list 형식이 확장성 있고 유연하긴 하지만 이를 위해 너무 많은 비지니스 로직이 필요합니다. 몇몇 사이트에서는 이를 Save Action 작업단위 Transaction 처리가 아닌 Even Trigger 처리 형식으로 간단히 구현하기도 하지만 사용자가 의도하지 않았던 Event를 무조건 적으로 Save 처리하는 방식은 사용자 친화적인 방법은 아니라 배제하게 되었습니다. 대신 공통메뉴 처럼 데이타 변경이 자주 일어나고 데이타 건수가 많을 경우 부모노드별로 트리 구조를처리하는 방식으로 절충 가능하다고 생각합니다.
