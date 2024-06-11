# Carve-Party 
할로윈 파티를 기념하기 위해 호박을 준비했다고한다. 호박을 10000번 클릭하면 플래그를 획득할 수 있다.

![alt text](/assets/img/Dreamhack/carve-party/image.png)

이 호박을 한번 누를때 마다 10000에서 -1씩 줄어든다. 진짜로 10000번을 다 클릭할 수도 있지만....그럴 수 없으니 한번 문제를 풀어보겠다.

페이지 소스에서 자바스크립트의 소스코드를 보니 굉장히 길고 복잡하게 되어있었는데 중점적으로 봐야할 부분은 아래에 표시한다.

```javascript
$(function() {
  $('#jack-target').click(function () {
    counter += 1;
    if (counter <= 10000 && counter % 100 == 0) {
      for (var i = 0; i < pumpkin.length; i++) {
        pumpkin[i] ^= pie;
        pie = ((pie ^ 0xff) + (i * 10)) & 0xff;
      }
    }
    make();
  });
```

위 소스코드에서 `$('#jack-target').click()` 함수가 한번 클릭할 때마다 counter를 해줘서 그것이 10000번 채우면 FLAG 를 준다.
그럼 간단히 for 문을 이용하면 가능하지 않을까 생각이 들어서 간단히 코드를 짜보았다.

```javascript
for(i = 0; i <= 10000; i++){
    $('#jack-target').click()
}
```

소스코드를 보고 어떤 함수를 써야할지 파악한 후, 코드 작성, flag 획득까지 간단히 진행하였다.

![alt text](/assets/img/Dreamhack/carve-party/image-1.png)