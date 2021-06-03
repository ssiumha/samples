# Javascript

일단 주로 웹 관련된 내용 적어놓기, node나 순수 js는 별도로..

## Web

### jquery 없이 element 조작

```
document.getElementById('id');
document.querySelector('.content .title');

document.querySelector('.content .text').innerHTML = '내용물 교체';
document.querySelector('.content .checkbox').classList.add('disabled');
document.querySelector('.content .checkbox').classList.toggle('checked', true);

document.querySelector('.content .metadata').setAttribute('datetime', '2972-10-26');
document.querySelector('.content .metadata').getAttribute('datetime');

# <div id="test" data-content-id="123" data-bookmarked="true"></div>
document.getElementById('test').dataset.contentId
document.getElementById('test').dataset.contentId = "999"
document.getElementById('test').dataset.bookmarked === 'true'   // bookmarked type: string
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
