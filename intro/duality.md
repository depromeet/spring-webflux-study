# Duality
쌍대성이라고 불리는 용어. Pull 방식과 Push 방식을 쌍대성이라고 한다. (궁극적인 기능은 똑같으나, 반대 방향으로 표현)<br/>
List<Integer> list = Arrays.asList(1,2,3 ... 10); 을 통해서 1~10을 찍는 예제를 통해서 Pull 방식과 Push 방식을 알아보자.

## Pull 방식

```
	Iterable<Integer> iter = () -> new Iterator<Integer>() {
		int i = 0;
		final static int MAX = 10;

		@Override
		public Integer next() {
			// TODO Auto-generated method stub
			return ++i;
		}

		@Override
		public boolean hasNext() {
			// TODO Auto-generated method stub
			return i < MAX;
		}
	};
	
	for(Integer i : iter) {
		System.out.println(i);
	}
	
	for(Iterator<Integer> it = iter.iterator(); it.hasNext();) {
		System.out.println(it.next());
	}
```

* 위의 방식으로 구현할 수 있는데, 이런 방식을 Pulling 방식이라고 한다. 내가 그 다음 것을 줘! 해서 갖고오고 있다. -> it.next()<br/>
* 어떤 소스를 사용하는 쪽에서 끌어오는 방식 <br/>
* 이와 반대로 Observable이라는 개념이 있는데 이 방식으로 구현한 것이 Pushing 방식이라고 한다. 
<br/>
위의 예제를 대표적인 Push방식인 Observer Pattern을 이용해 구현을 해보자.

## Push 방식

```
	// Observable  Source -> Event/Data -> Observer : Observer를 Observable에 등록. 
	// Observable은 새로운 정보가 발생할 때마다 Observer에 notify한다. (Observer 여러 개 등록 가능)
	
	static class IntObservable extends Observable implements Runnable {
		@Override
		public void run() {
			for(int i=1; i<=10; i++) {
				// 변화가 생겼다는 것을 설정하는 메서드
				// setChaged()를 설정하지 않으면 notify를 실행해도 전파되지 않는다.
				setChanged();
				// 변경 내용을 전파하는 메서드
				notifyObservers(i); // publisher
				// int i = it.next(); 대응관계
			}
		}
	}
	
	public static void main(String args[]) {
		// subscriber (= observer)
		// Observable에게 등록되어 변화가 생기면 전파받는다.
		// update 메서드를 통하여 전파 받은 내용으로 작업을 한다.
		Observer ob = new Observer() {
			@Override
			public void update(Observable o, Object arg) {
				System.out.println(arg);
			}
		};
		
		IntObservable io = new IntObservable();
		io.addObserver(ob); // IntObservable이 던지는 이벤트는 ob 옵저버가 받는다.
		io.run();
	}
```

자바에서는 기본적으로 Observer Pattern을 위한 API를 제공한다. (자바 1.0부터 존재하던 API)</br>
실행하면 출력내용은 동일하지만 여러 개의 observer들이 동시에 변경 내용을 받을 수 있다는 장점이 있다. 위의 예제는 Duality의 예인데 옵저버 패턴은 <b>Iterator</b>처럼 필요한 데이터를 가져와서(pull) 출력하는 것이 아니라 필요한 데이터를 전달(push)해서 출력을 한다는 것이 핵심이다.</br></br>

추가적으로 위의 예를 수정할 점이 무엇일까? 바로 이벤트가 언제 발생하는지 모르는데 메인 스레드를 가로막고 있다는 것이다. 따라서 observable이 비동기적으로 동작하도록 수정할 수 있다. 다음 코드를 보자. </br>

```
	static class IntObservable extends Observable implements Runnable {
	       ... 생략 (위 코드와 동일)
	}
	
	public static void main(String args[]) {
		// subscriber (= observer)
		// Observable에게 등록되어 변화가 생기면 전파받는다.
		// update 메서드를 통하여 전파 받은 내용으로 작업을 한다.
		Observer ob = new Observer() {
			@Override
			public void update(Observable o, Object arg) {
				System.out.println(Thread.currentThread().getName() +  " " + arg);
			}
		};
		
		IntObservable io = new IntObservable();
		io.addObserver(ob); // IntObservable이 던지는 이벤트는 ob 옵저버가 받는다.
		
		ExecutorService es = Executors.newSingleThreadExecutor(); // 스레드를 하나 할당 받는다.
		es.execute(io); // Runnable 인터페이스를 구현한 객체를 실행하면 run이 돌아간다.

		System.out.println("EXIT");
		es.shutdown();
	}
```
</br>
위의 코드를 보면 Push 방식으로 observer 패턴을 이용하면 별개의 스레드에서 동작하는 코드를 손쉽게 작성할 수 있다. 반대로, Pulling 방식(Iterator)으로 멀티 스레딩 코드를 작성하려면 꽤나 까다로운 코드가 나올 것이다. </br></br>

이 두 방식의 차이가 리액티브 프로그래밍의 시발점이라고 할 수 있다.
