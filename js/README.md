# Javascript


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
