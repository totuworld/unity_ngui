
표 3-19: UISprite 컴포넌트 설정

| 속성 | 값 |
| ---|:---:|
|Atlas|InGameAtlas|
|Sprite|BarBorder|
|Type|Sliced|
|Fill Center|체크|
|Depth|2|
|Size|150x38|
|Color Tint|73, 44, 0, 255|


표 3-20

| 속성 | 값 |
| ---|:---:|
|Atlas|InGameAtlas|
|Sprite|BarPattern|
|Type|Filled|
|Depth|1|
|Size|140x24|
|Color Tint|255, 216, 0, 255|


표 3-21

| 속성 | 값 |
| ---|:---:|
|Atlas|InGameAtlas|
|Sprite|FillRect|
|Type|Sliced|
|Fill Center|체크|
|Depth|0|
|Size|140x32|
|Color Tint|174, 0, 0, 255|


표 3-22

| 속성 | 값 |
| ---|:---:|
|Atlas|InGameAtlas|
|Sprite|ShadowRect|
|Type|Sliced|
|Fill Center|체크|
|Depth|3|
|Color Tint|128, 82, 0, 255|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|HPBar 게임 오브젝트|
|Left|Target’s Left|-55|
|Right|Target’s Right|15|
|Bottom|Target’s Bottom|-5|
|Top|Target’s Top|5|


표 3-23
| 속성 | 값 |
| ---|:---:|
|Font|NanumBarunGothicBold 끌어다 놓기|
|Font Size|30|
|Text|HP|
|Overflow|ResizedFreely|
|Depth|4|
|Color Tint|255, 255, 255, 255|



표 3-24
| 속성 | 값 |
| ---|:---:|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|Panel 게임 오브젝트|
|Left|Target’s Left|60|
|Right|Target’s Left|210|
|Bottom|Target’s Top|-48|
|Top|Target’s Top|-10|

표 3-25
| 속성 | 값 |
| ---|:---:|
|Text|점수|


표 3-26
| 속성 | 값 |
| ---|:---:|
|Font|NanumBarunGothicBold 끌어다 놓기|
|Font Size|28|
|Text|0|
|Overflow|ResizedFreely|
|Depth|5|
|Color Tint|255, 255, 255, 255|

표 3-27
| 속성 | 값 |
| ---|:---:|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|Panel 게임 오브젝트|
|Left|Target’s Center|-75|
|Right|Target’s Center|75|
|Bottom|Target’s Top|-48|
|Top|Target’s Top|-10|

### 적 웨이브 표시

표 3-28
| 속성 | 값 |
| ---|:---:|
|Text|Wave|

표 3-29
| 속성 | 값 |
| ---|:---:|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|WaveBar 게임 오브젝트|
|Left|Target’s Left|-85|
|Right|Target’s Left|15|
|Bottom|Target’s Bottom|-5|
|Top|Target’s Top|5|


표 3-30
| 속성 | 값 |
| ---|:---:|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|Panel 게임 오브젝트|
|Left|Target’s Right|-300|
|Right|Target’s Right|-200|
|Bottom|Target’s Top|-48|
|Top|Target’s Top|-10|


### 일시 정지, 배속 변경 버튼 추가


표 3-31
| 속성 | 값 |
| ---|:---:|
|Atlas|InGameAtlas|
|Sprite|ShadowRect|
|Type|Sliced|
|Fill Center|체크|
|Depth|2|
|Color Tint|255, 195, 0 , 255|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|Panel 게임 오브젝트|
|Left|Target’s Right|-80|
|Right|Target’s Right|-10|
|Bottom|Target’s Top|-80|
|Top|Target’s Top|-10|

표 3-32
| 속성 | 값 |
| ---|:---:|
|Atlas|InGameAtlas|
|Sprite|pause|
|Type|Sliced|
|Fill Center|체크|
|Depth|3|
|Size|35x40|
|Color Tint|0, 0, 0 , 255|


표 3-33
| 속성 | 값 |
| ---|:---:|
|Color Tint|255, 111, 0, 255|
|Anchors Type|Unified|
|Execute|OnEnable|
|Target|PauseButton 게임 오브젝트|
|Left|Target’s Left|-80|
|Right|Target’s Left|-10|
|Bottom|Target’s Bottom|0|
|Top|Target’s Top|0|