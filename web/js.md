# Javascript

가끔씩 써서 쓸때마다 한번씩 봐야하는거 모음.

## Web

### jquery 없이 element 조작

```
document.getElementById('id');
document.querySelector('.content .title');

document.querySelector('.content .text').innerHTML = '내용물 교체';
document.querySelector('.content .checkbox').classList.add('disabled');
document.querySelector('.content .checkbox').classList.toggle('checked', true);
document.querySelector('.content .checkbox').classList.contains('checked');

document.querySelector('.content .metadata').setAttribute('datetime', '2972-10-26');
document.querySelector('.content .metadata').getAttribute('datetime');

document.querySelector('.child').closest('parent');
document.querySelector('.child').form.submit(); # form에 속해있으면 작동

# <div id="test" data-content-id="123" data-bookmarked="true"></div>
document.getElementById('test').dataset.contentId
document.getElementById('test').dataset.contentId = "999"
document.getElementById('test').dataset.bookmarked === 'true'   // bookmarked type: string

# style 조작
document.getElementById('test').style.color = 'blue';
document.getElementById('test').style.backgroundColor = 'red';

# StaticNodeList에 map 때리기
Array.from(document.querySelectorAll('div')).map(x => x.innerHTML)
[...document.querySelectorAll('div')].map(x => x.innerHTML)

# global style 가져오기
getComputedStyle(document.documentElement).getPropertyValue('--responsive-mode').trim();

# parameter control
const params = new URLSearchParams(window.location.search);
params.get('q')
params.has('q')
params.append('id', 'test')
```

### dev console

- chrome console util: https://developer.chrome.com/docs/devtools/console/utilities

safari, chrome, firefox 공통으로 개발자 콘솔에서
$가 정의되어있지 않다면 document.querySelector로 $$는 document.querySelectorAll로 바인딩되어있다.

코드에서도 똑같이 사용하고 싶다면 직접 바인딩해서 쓰면 된다.

```
const $ = document.querySelector.bind(document);
const $$ = document.querySelectorAll.bind(document);
```

### Event

```
# -- bind onload

## document.readState의 상태가 변경되면 발생. 상태는 loading -> interactive -> complete
## interactive는 DOMContentLoaded, complete는 load에 해당
document.addEventListener('readystatechange', e => {
})

## HTML 문서가 로드되고 구문분석이 끝나면 트리거
## image, stylesheet 등이 로딩을 기다리지 않고 작동한다
document.addEventListener('DOMContentLoaded', e => {
  ...
});

## image, stylesheet를 포함해서 모든 리소스가 로드되면 발생
window.addEventListener('load', e => {
  ...
});


# -- bind function
<div id="id" onchange="onChangeAction()"></div>
document.querySelector('#id').addEventListener('change', onChangeAction);

function onChangeAction() {
  console.log(event.target);
  ...
}

# -- key input
elem.addEventListener('keydown', function(event) {
  if (event.code === 'Enter') {
    console.log('enter keydown');
  }
  ...
});
```

### simple fold animation

```
.area {
  max-height: 0px;
  transition: max-height .5s ease-out;

  &.opened {
    max-height: 500px;
    transition: max-height .5s ease-in;
  }
}
```

### simple image checkbox

```
input[type=checkbox].img-checkbox {
  $size: 24px;

  visibility: hidden;
  width: $size;
  height: $size;
  vertical-align: middle;

  &:before {
    visibility: visible;
    content: "";
    display: inline-block;
    width: $size;
    height: $size;

    background: {
      image: <checkbox-normal-img>;
      repeat: no-repeat;
      size: cover;
    }
  }

  &:checked:before {
    visibility: visible;
    background: {
      image: <checkbox-checked-img>;
      repeat: no-repeat;
      size: cover;
    }
  }
}
```

### simple input filter

```
<div class="popup select-category-popup">
  <div class="header">
    <div class="title">검색할 내용 입력</div>

    <input onkeyup="this.closest('.popup')
                    .querySelectorAll('.category-list > .select-item')
                    .forEach(item => item.classList.toggle('hidden', item.dataset.tabName.startsWith(this.value)))">
  </div>

  <div class="category-list">
    <% @tabs.each do |tab| %>
      <div class="select-item"
           data-tab-name="<%= tab.name %>"
           onclick="document.querySelector('input#tab_id').value = this.dataset.tabName;
                    this.closest('form').submit()">
        <%= tab.name %>
      </div>
    <% end %>
  </div>
</div>
```

### simple debounce

입력 자동 완성 같은 호출 이벤트가 짧은 시간 내에 대량으로 일어나지만, 적당하게 한번만 호출되어야할 때 사용된다.
이거 때문에 lodash를 가져오는건 닭잡는 대검이니 코드 스니펫 메모..

```js
<script>
document.getElementById('#search-input-box')
        .addEventListener('keyup', debounce(reqAutoComplete, 2000));

function debounce(callback, delay) {
    let timer;
    return function() {
        clearTimeout(timer);
        timer = setTimeout(callback, delay);
    }
}

function reqAutoComplete() {
  ...
}
</script>
```
