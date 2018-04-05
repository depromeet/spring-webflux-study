# Observer 패턴의 문제점
이전에 Duality를 통해서 push 방식과 pull 방식을 살펴보았다. 그런데 이전의 Observer 패턴에는 단점이 있었는데, 단점은 다음과 같다.<br/>
* Complete의 부재
  * Observer 패턴은 Observable이 데이터를 다 주었다는 complete의 개념이 없다.
* Error 처리
  * 예외가 전파되는 방식과 예외를 받아서 어떻게 처리할까라는 것이 패턴에 녹아있지 않다.
<br/>

-> 위의 두 가지가 확장된 것이 리액티브 프로그래밍의 한 축이 된다! <br/>
다음은 reactive-streams에 표준을 정의해놓은 것을 정리한 것. <br/>

# Publisher와 Subscriber
* Publisher : 옵저버 패턴의 Observable의 역할 (데이터를 주는 쪽)
* Subscriber : Observer 역할 (데이터를 받는 쪽) 
둘의 관계는 구독을 통해서 연결된다. <br/>


```
public static void main(String[] args) throws InterruptedException {
	Iterable<Integer> iter = Arrays.asList(1,2,3,4,5);
	
	Publisher p = new Publisher() {
		@Override
		public void subscribe(Subscriber subscriber) {
			Iterator<Integer> it = iter.iterator();
			
			subscriber.onSubscribe(new Subscription() {
				@Override
				public void request(long n) {
					int i = 0;
					try {
						while(i++ < n) {
							if(it.hasNext()) {
								subscriber.onNext(it.next());
							} else {
								subscriber.onComplete();
								break;
							}
						}							
					} catch(RuntimeException e) {
						subscriber.onError(e);
					}							
				}
				
				@Override
				public void cancel() {
					
				}
			});
		}
	};
	
	// 오버라이딩 할 메서드 4가지는 reactive-streams의 프로토콜이다.
	// onSubscribe는 subscribe 하는 즉시 호출해야 한다.
	// onNext는 자유
	// onError | onComplete
	Subscriber<Integer> s = new Subscriber<Integer>() {
		Subscription subscription;
		
		@Override
		public void onSubscribe(Subscription subscription) {
			System.out.println("onSubscribe");
			this.subscription = subscription;
			this.subscription.request(1);
		}
		
		@Override
		public void onComplete() {
			System.out.println("onComplete");
		}

		@Override
		public void onError(Throwable throwable) {
			System.out.println("onError");
		}

		@Override
		public void onNext(Integer n) {
			System.out.println("onNext : " + n);
			this.subscription.request(1);
		}

	};

	p.subscribe(s); // subscriber는 publisher를 구독해야 한다.
}	
```

위는 reactive-streams의 프로토콜을 구현한 것이다. 코드를 정리하자면 <br/>
1. Subscriber는 Publisher를 subscriber(구독)해야 한다.
2. Publisher는 구독이 들어오면 반드시 구독자의 onSubscribe를 호출해야 한다.
3. onSubscribe의 파라미터로 Subscription이 존재하는데, 이 오브젝트를 통해서 Publisher와 Subscriber의 요청을 조절할 것이다. (마치 TCP에서 윈도우처럼)
4. Subscriber는 onSubscribe의 subcription을 통해서 request를 호출하고, request 파라미터로 현재 몇 개의 데이터를 처리할 것인지 알려준다.
5. Publisher는 subscription을 통해 request 개수를 넘겨받고, 해당 개수만큼 처리를 한 뒤 Subscriber의 onNext로 데이터를 넘겨준다.
6. SubScriber는 onNext로 넘겨받은 데이터를 처리하고, 데이터를 처리할 만큼 subscription의 request(n)으로 다시 넘겨준다.
7. 이 과정을 반복하고, 데이터를 모두 전달했을 시에 Publisher는 Subscriber의 onComplete을 실행하여 데이터 전달이 완료되었음을 알려준다.

옵저버 패턴과는 다르게 Publisher와 Subscriber는 subscription이라는 중개자를 두어서 요청을 조절할 수 있다는 것과 onComplete, onError를 통해서 데이터를 전송 완료와 에러를 처리할 수 있다는 것도 알게 되었다.
