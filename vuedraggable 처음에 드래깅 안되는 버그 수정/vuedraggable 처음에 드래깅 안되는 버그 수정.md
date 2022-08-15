![예약 화면](./reservation.png)

서핏 어드민에는 이렇게 아티클들의 예약 발행 내역을 볼 수 있는 기능이 있다. 8월 둘째주에는 여기에 드래그 기능으로 발행 순서를 수정할 수 있는 기능을 넣게 되었다.

마음같아서는 브라우저 기본 이벤트들을 이용해서 드래그 기능을 구현하고 싶었지만, 백오피스 기능이기도 하고 이미 프로젝트에서 vuedraggable이라는 드래깅 라이브러리를 사용하고 있었기에 해당 라이브러리를 이용해서 작업을 진행하였다.

[vuedraggable github](https://github.com/SortableJS/Vue.Draggable)

사용법은 간단하다. vuedraggable에서 제공하는 컴포넌트에 드래그 기능의 대상이 되는 리스트 데이터를 v-model로 넘기고 해당 컴포넌트 내부에는 v-for를 이용해서 리스트를 렌더링하면 끝이다.

```vue
<draggable v-model="myArray">
    <div v-for="element in myArray" :key="element.id">
        {{element.name}}
    </div>
</draggable>
```

예약 발행 내역을 보여주는 페이지에는 아티클의 카테고리를 분류해서 볼 수 있는 기능이 있다. 해당 기능을 사용하려면 백엔드로부터 카테고리 데이터를 가져올 수 있어야 한다.

프론트엔드 로직에서는 이 카테고리 데이터가 있으면 카드를 노출하게 만드는 조건이 있었다.

```vue
<div v-if="categorys" v-for="element in myArray" :key="element.id">
    {{element.name}}
</div>
```

이번 작업을 하면서 카테고리 관련 부분을 건드릴 필요가 없어서 단순히 위 코드에 draggable 컴포넌트를 덧대는 형식으로 코드 작업을 진행했다.

```vue
<draggable v-model="myArray">
    <div v-if="categorys" v-for="element in myArray" :key="element.id">
        {{element.name}}
    </div>
</draggable>
```

그랬더니 새로고침하고 처음 드래깅을 할 때 드래그 기능이 작동하지 않는 버그를 발견했다.

![버그](./bug.mov)

한참을 vuedraggable 관련 문서를 읽고 수정한 코드에 대해서 디버깅하는 시간을 가졌다. 도저히 원인을 찾지 못하다가 선태님과 같이 시간을 쓰게 되었는데,

```vue
<draggable v-if="categorys" v-model="myArray">
    <div v-for="element in myArray" :key="element.id">
        {{element.name}}
    </div>
</draggable>
```

```v-if="categorys"``` 코드를 draggable 바깥으로 빼보니 제대로 작동하는 것을 볼 수 있었다. 어짜피 카테고리 데이터가 없으면 아티클을 보여줄 수가 없으니 이런식으로 코드를 작성하는 것이 좀 더 매끄러운 것 같다는 생각이 들었다.

이 원인이 드래그 기능에 어떤 영향을 미쳤는지 생각해봤는데, 비동기적으로 바인딩되는 데이터를 조건부 렌더링의 조건으로 건다면 라이브러리의 기능이 적용이 되지 않을 수 있다는 결론을 냈다.

```vue
<draggable v-if="categorys" v-model="myArray">
    // v-if로 인해 엘리먼트 없음
</draggable>
```

```vue
<draggable v-if="categorys" v-model="myArray">
    // 갑자기 생김. 드래그 기능이 적용되지 않음.
    <div>{{name}}</div>
    <div>{{name}}</div>
    <div>{{name}}</div>
    <div>{{name}}</div>
</draggable>
```

**왜 첫 드래깅만 안되는 것인가?**

첫 드래그 기능 후 리렌더링 되면서 하위 엘리먼트들에게 드래그 기능들이 적용되기에 다음번 드래깅에서는 정상적으로 작동한다.