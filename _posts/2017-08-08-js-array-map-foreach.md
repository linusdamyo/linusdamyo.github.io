---
layout: post
title:  "Array.prototype.map() 과 Map.prototype.forEach()"
categories: javascript
---

최근에 React.js 로 작업을 하면서, render() 안에 제일 흔하게 사용하는 형태가 array map() 인데, key-value 구조가 필요해서 Map() 을 사용하게 되었다.  

* 흔한형태
```javascript
  render() {
    return (
      <div>
        {this.state.items.map(item => <Item key={item.id} item={item}/>)}
      </div>
    )
  }
```

* items 를 Map() 으로 바꾸면, map() 을 더이상 쓸수가 없다. 그럼 forEach() 를 쓰면 되지!
```javascript
  render() {
    return (
      <div>
        {this.state.items.forEach((value, key) => <Item key={key} value={value}/>)}
      </div>
    )
  }
```

이런 멍청한 짓을 했었다.  
loop을 돌기는 하는데, 결과가 안나와서 결국 이상한모임 슬랙에 도움을 요청했더니, 어떤 분이 map() 은 array가 return된다는 말씀을 해주셨다.  
앗! MDN을 보면, `The map() method creates a new array with the results of calling a provided function on every element in the calling array.`  
반면, forEach()는 `The forEach() method executes a provided function once per each key/value pair in the Map object, in insertion order.`   
그렇다! map() 을 사용하면 새로운 array가 반환되니, render() 안에서 n개의 Item 이 표시된다.  
forEach()는 loop을 돌기만 할뿐.. render() 안에서는 당연히 표시가 안되는 것이다.    

### 결론

* 기본을 잘 알아야겠다. map() 이 어떤 아이인지, forEach() 는 어떤 아이인지, 이외에도 기본적인 공부가 덜되어 있었다.
* Map() 에 map() 을 적용하려면, Array.from() 을 이용하면 된다.
```javascript
  render() {
    return (
      <div>
        {Array.from(this.state.items).map(item => <Item key={item[0]} value={item[1]}/>)}
      </div>
    )
  }
```
* ES6 책 사놓은 것도 빨리 읽어야 되는데, 언제가 될지 모르겠다.
