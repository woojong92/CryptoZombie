# [크립토좀비](https://cryptozombies.io) Lesson4를 학습하면서 정리한 내용입니다.

# 챕터 3: 상태 변수&정수

- 상태 변수 : 컨트랙트 저장소에 영구적으로 저장되는 변수. 즉, 이더리움 블록체인에 기록됨

	contract Example {
		// 이 변수는 블록체인에 영구적으로 저장된다.
		uint myUnsignedInteger = 100;
	}

- uint자료형은 부호 없는 정수로, 값이 음수가 아이어야 한다.
- 솔리디티에서 uint는 실제로 uint256, 즉 256비트 부호 없는 정수의 다른 표현이다.

# 챕터 4: 수학 연산
- 솔리디티는 지수 연산되 지원한다. 

	uint x = 5 ** 2; // 즉, 5^2 = 25

# 챕터 5: 구조체

- 솔리디티는 구조체를 제공한다.

	struct Person {
		uint age;
		string name;
	}

- string자료형은 임의의 길이를 가진 utf-8 데이터를 위해 활용됨.  

# 챕터 6:  배열

- 솔리디티에는 정적 배열과 동적 배열, 두 종류의 배열이 있다.

	// 2개의 원소를 담을 수 있는 고정 길이의 배열:
	uint[2] fixedArray;
	// 또다른 고정 배열으로 5개의 스트링을 담을 수 있다:
	string[5] stringArray;
	// 동적 배열은 고정된 크기가 없으며 계속 크기가 커질 수 있다:
	uint[] dynamicArray;

- 구조체의 배열을 생성할 수도 있다. 

	Person[] people; // 이는 동적 배열로, 원소를 계속 추가 할 수 있다.

- 상태 변수가 블록체인에 영구적으로 저장되며, 이처럼 구제체의 동적 배열을 생성하면 마치 데이터베이스처럼 컨트랙트에 구조화된 데이터를 저장하는데 유용하다.

	##Public 배열
	
	- public으로 배열을 선언. 솔리디티는 이런 배열을 위해 getter메소드를 자동적으로 생성한다. 구문은 다음과 같다.
	
		Person[] public people;
	
	- 그러면 다른 컨트랙트들이 이 배열을 읽을 수 있게 된다.( 단, 쓸 수는 없다.) 이는 컨트랙트에 공개 데이터를 저장할 때 유용한 패턴이다.


#챕터 7: 함수 선언

-솔리디티의 함수 선언은 다음과 같다.

	function eatHamburgers(string _name, uint _amount) {

	}

-참고: 함수 인자명을 언더스코어(_)로 시작해서 전역변수와 구별하는 것이 관례이다.(의무x)

-위 함수는 다음과 같이 호출할 수 있다. 
	eatHambergers("vitalik", 100);

#챕터 8: 구조체와 배열 활용하기

	## 새로운 구조체 생성하기.
		
	- Person 구조체 생성 및 public 구조체 배열 생성
		struct Person {
		  uint age;
		  string name;
		}
		
		Person[] public people;
	
	- 새로운 Person를 생성하고, people 배열에 추가하는 방법
	
		// 새로운 사람을 생성한다:
		Person satoshi = Person(172, "Satoshi");
		
		// 이 사람을 배열에 추가한다:
		people.push(satoshi);
	
	- 이 두 코드를 조합하여 깔끔하게 한줄로 표현 할 수 있다.
	
		people.push(Person(16, "Vitalik"));

#챕터 9: Private/Public 함수

- 솔리디티에서는 기본적으로 public으로 선언된다. 즉, 누구나 (혹은 다른 어느 컨트랙트가) 생성된 컨트랙트의 함수를 호출하고 코드를 실행할 수 있다.

- 이는 항상 바람직하지 않으며, 컨트랙트를 공격에 취약하게 만들 수 있다. 따라서 기본적으로 함수를 *private*으로 선언하고, 공개할 함수만 *public*으로 선언하는 것이 좋다.

- private 함수를 선언하는 방법
	
	uint[] numbers;
	
	function _addToArray(uint _number) private {
	  numbers.push(_number);
	}

- private는 컨트랙트 내의 다른 함수들만이 이 함수를 호출하여 *number* 배열로 무언가를 추가할 수 있다는 것을 의미한다.

- *private* 키워드는 함수명 다음에 적히며, 함수 인자명과 마찬가지로 private 함수명도 언더바(_)로 시작하는 것이 관례이다.

# 챕터 10: 함수를 더 알아보기

	##반환값
	- 함수에서는 어떤 값을 반환 받으려면 다음과 같이 선언해야 한다.
	
		string greeting = "What's up dog";
		
		function sayHello() public returns (string) {
		  return greeting;
		}
	
	- 솔리디티에서 함수 선언은 반환값 종류를 포함한다.(이 경우에는 string이다.)

	##함수 제어자
	- 위에서 살펴 본 함수 sayHello()는 솔리디티에서 상태를 변화시키지 않는다. 즉, 어떤 값을 변경하거나 무언가를 쓰지 않는다.
	- 이 경우에는 **view** 함수로 선언한다. 이는 함수가 데이터를 보기만 하고 변경하지 않는다는 뜻이다.
	
		function sayHello() public view returns (string) {	
	
	- 솔리디티는 **pure** 함수도 갖는다. 이는 함수가 앱에서 어떤 데이터도 접근하지 않는 것을 의미한다.
	
		function _multiply(uint a, uint b) private pure returns (uint) {
		  return a * b;
		}
	
	- 이 함수는 앱에서 읽는 것도 하지 않고, 다만 반환값이 함수에 전달된 인자값에 따라서 달라진다. 이 경우에 함수를 **pure**로 선언한다.
	
	- 솔리디티 컴파일러는 어떤 제어자를 써야 하는지 경고 메시지를 통해 잘 알려준다.

#챕터 11: Keccak256과 형 변환

- 이더리움은 SHA3의 한 버전인 keccak256를 내장 해시 함수로 가지고 있다. 해시 함수는 기본적으로 입력 스트링을 랜덤 256비트 16진수로 매핑한다. 

- 예시:

	//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
	keccak256("aaaab");
	//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
	keccak256("aaaac");

	##형 변환
	
		uint8 a = 5;
		uint b = 6;
		// a * b가 uint8이 아닌 uint를 반환하기 때문에 에러 메시지가 난다:
		uint8 c = a * b; 
		// b를 uint8으로 형 변환해서 코드가 제대로 작동하도록 해야 한다:
		uint8 c = a * uint8(b); 
	
	- 위의 예시에서 a * b는 uint를 반환하지. 하지만 우리는 이 반환값을 uint8에 저장하려고 하니 잠재적으로 문제를 야기할 수 있네. 반환값을 uint8으로 형 변환하면 코드가 제대로 작동하고 컴파일러도 에러 메시지를 주지 않을 걸세.

#챕터 12: 종합하기

1. createRandomZombie라는 public함수를 생성한다. 이 함수는 _name이라는 string형 인자를 하나 전달받는다. (참고: 함수를 private로 선언한 것과 마찬가지로 함수를 public로 생성할 것)

2. 이 함수의 첫 줄에서는 _name을 전달받은 _generateRandomDna 함수를 호출하고, 이 함수의 반환값을 randDna라는 uint형 변수에 저장해야 한다.

3. 두번째 줄에서는 _createZombie 함수를 호출하고 이 함수에 _name와 randDna를 전달해야 한다.

4. 함수의 내용을 닫는 }를 포함해서 코드가 4줄이어야 한다.

	pragma solidity ^0.4.19;
	
	contract ZombieFactory {
	
	    uint dnaDigits = 16;
	    uint dnaModulus = 10 ** dnaDigits;
	
	    struct Zombie {
	        string name;
	        uint dna;
	    }
	
	    Zombie[] public zombies;
	
	    function _createZombie(string _name, uint _dna) private {
	        zombies.push(Zombie(_name, _dna));
	    } 
	
	    function _generateRandomDna(string _str) private view returns (uint) {
	        uint rand = uint(keccak256(_str));
	        return rand % dnaModulus;
	    }
	    
	     function createRandomZombie(string _name) public {
	        uint randDna = _generateRandomDna(_name);
	        _createZombie(_name, randDna);
	    }
	}


#챕터 13: 이벤트

- 이벤트는 컨트랙트가 블록체인 상에서 앱의 사용자 단에서 무언가 액션이 발생했을 때 의사소통하는 방법이다. 컨트랙트는 특정 이벤트가 일어나는지 "귀를 기울이고" 그 이벤트가 발생하면 행동을 취하지.

- 컨트랙트는 특정 이벤트가 일어나는지 "귀를 기울이고" 그 이벤트가 발생하면 행동을 취한다.

- 예시: 

	// 이벤트를 선언한다
	event IntegersAdded(uint x, uint y, uint result);
	
	function add(uint _x, uint _y) public {
	  uint result = _x + _y;
	  // 이벤트를 실행하여 앱에게 add 함수가 실행되었음을 알린다:
	  IntegersAdded(_x, _y, result);
	  return result;
	}

- 그러면 앱의 사용자 단은 해당 이벤트가 일어나는지 귀 기울이며, 자바스트립트로 구현하면 다음과 같다.

	YourContract.IntegersAdded(function(error, result) { 
	  // 결과와 관련된 행동을 취한다
	}

-이벤트를 위해 좀비의 id가 필요할 것이다. array.push()는 배열의 새로운 길이를 uint형으로 반환한다. 배열의 첫 원소가 0이라는 인덱스를 갖기 때문에, array.push() - 1은 막 추가된 좀비의 인덱스가 될 것이다. zombies.push() - 1의 결과값을 uint형인 id로 저장하고 이를 다음 줄에서 NewZombie 이벤트를 위해 활용한다

#챕터 14: Web3.js

- 솔리디티 컨트랙트가 완성되면 이제 이 컨트랙트와 상호작용하는 사용자 단의 자바스크립트 코드를 작성해야 한다.
- 이더리움은 Web3.js 라고 하는 자바스크립트 라이브러리를 가지고 있다.
		
		// 여기에 우리가 만든 컨트랙트에 접근하는 방법을 제시한다:
		var abi = /* abi generated by the compiler */
		var ZombieFactoryContract = web3.eth.contract(abi)
		var contractAddress = /* our contract address on Ethereum after deploying */
		var ZombieFactory = ZombieFactoryContract.at(contractAddress)
		// `ZombieFactory`는 우리 컨트랙트의 public 함수와 이벤트에 접근할 수 있다.
		
		// 일종의 이벤트 리스너가 텍스트 입력값을 취한다:
		$("#ourButton").click(function(e) {
		  var name = $("#nameInput").val()
		  // 우리 컨트랙트의 `createRandomZombie`함수를 호출한다:
		  ZombieFactory.createRandomZombie(name)
		})
		
		// `NewZombie` 이벤트가 발생하면 사용자 인터페이스를 업데이트한다
		var event = ZombieFactory.NewZombie(function(error, result) {
		  if (error) return
		  generateZombie(result.zombieId, result.name, result.dna)
		})
		
		// 좀비 DNA 값을 받아서 이미지를 업데이트한다
		function generateZombie(id, name, dna) {
		  let dnaStr = String(dna)
		  // DNA 값이 16자리 수보다 작은 경우 앞 자리를 0으로 채운다
		  while (dnaStr.length < 16)
		    dnaStr = "0" + dnaStr
		
		  let zombieDetails = {
		    // 첫 2자리는 머리의 타입을 결정한다. 머리 타입에는 7가지가 있다. 그래서 모듈로(%) 7 연산을 하여
		    // 0에서 6 중 하나의 값을 얻고 여기에 1을 더해서 1에서 7까지의 숫자를 만든다. 
		    // 이를 기초로 "head1.png"에서 "head7.png" 중 하나의 이미지를 불러온다:
		    headChoice: dnaStr.substring(0, 2) % 7 + 1,
		    // 두번째 2자리는 눈 모양을 결정한다. 눈 모양에는 11가지가 있다:
		    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
		    // 셔츠 타입에는 6가지가 있다:
		    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
		    // 마지막 6자리는 색깔을 결정하며, 360도(degree)까지 지원하는 CSS의 "filter: hue-rotate"를 이용하여 아래와 같이 업데이트된다:
		    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
		    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
		    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
		    zombieName: name,
		    zombieDescription: "A Level 1 CryptoZombie",
		  }
		  return zombieDetails
		}