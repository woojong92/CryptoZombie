[크립토좀비](https://cryptozombies.io) Lesson4를 학습하면서 정리한 내용입니다.

#챕터 7: Storage vs Memory

- 솔리디티에는 변수를 저장할 수 있는 공간으로 storage와 memory 두 가지가 있다.
- **Storage**는 블록체인 상에 영구적으로 저장되는 변수를 의미한다.
- **Memory**는 임시적으로 저장되는 변수로, 컨트랙트 함수에 대한 외부 호출들이 일어나는 사이에 지워진다. 
- 상태 변수(함수 외부에 선언된 변수)는 초기 설정상 *storage*로 선언되어 블록체인에 영구적으로 저장된다.
- 함수 내에서 선언된 변수는 memory로 자동 선언되어서 함수 호출이 종료되면 사라진다.

-대부분의 경우에, 솔리디티가 알아서 처리해주기 때문에, storage와 memory와 같은 키워들을 이용할 필요가 없다.

-하지만 함수 내의 **구조체*와 배열을 처리할 때 이 키워드들이 필요로 한다.
	
	contract SandwichFactory {
	  struct Sandwich {
	    string name;
	    string status;
	  }
	
	  Sandwich[] sandwiches;
	
	  function eatSandwich(uint _index) public {
	    // Sandwich mySandwich = sandwiches[_index];
	
	    // ^ 꽤 간단해 보이나, 솔리디티는 여기서 
	    // `storage`나 `memory`를 명시적으로 선언해야 한다는 경고 메시지를 발생한다. 
	    // 그러므로 `storage` 키워드를 활용하여 다음과 같이 선언해야 한다:
	    Sandwich storage mySandwich = sandwiches[_index];
	    // ...이 경우, `mySandwich`는 저장된 `sandwiches[_index]`를 가리키는 포인터이다.
	    // 그리고 
	    mySandwich.status = "Eaten!";
	    // ...이 코드는 블록체인 상에서 `sandwiches[_index]`을 영구적으로 변경한다. 
	
	    // 단순히 복사를 하고자 한다면 `memory`를 이용하면 된다: 
	    Sandwich memory anotherSandwich = sandwiches[_index + 1];
	    // ...이 경우, `anotherSandwich`는 단순히 메모리에 데이터를 복사하는 것이 된다. 
	    // 그리고 
	    anotherSandwich.status = "Eaten!";
	    // ...이 코드는 임시 변수인 `anotherSandwich`를 변경하는 것으로 
	    // `sandwiches[_index + 1]`에는 아무런 영향을 끼치지 않는다. 그러나 다음과 같이 코드를 작성할 수 있다: 
	    sandwiches[_index + 1] = anotherSandwich;
	    // ...이는 임시 변경한 내용을 블록체인 저장소에 저장하고자 하는 경우이다.
	  }
	}

#챕터 8: 좀비 DNA

	function testDnaSplicing() public {
	  uint zombieDna = 2222222222222222;
	  uint targetDna = 4444444444444444;
	  uint newZombieDna = (zombieDna + targetDna) / 2;
	  // ^ 3333333333333333이 될 것이다
	}



#챕터 9: 함수 접근 제어자를 더 알아보기

자네가 코드를 컴파일하려고 하면 컴파일러가 에러 메시지를 출력할 거네.

문제는 ZombieFeeding 컨트랙트 내에서 _createZombie 함수를 호출하려고 했다는 거지. 그런데 _createZombie 함수는 ZombieFactory 컨트랙트 내의 private 함수이지. 즉, ZombieFactory 컨트랙트를 상속하는 어떤 컨트랙트도 이 함수에 접근할 수 없다는 뜻이지.

-솔리디티는 **public**과 **private** 이외에도 **internal**과 **external**이라는 함수 접근 제어자가 있다.

-**internal**은 함수가 정의된 컨트랙트를 상속하는 컨트랙트에서도 접근이 가능하다는 점을 제외하면 **private**과 동일하다.

-**external**은 함수가 컨트랙트 바깥에서만 호출될 수 있고 컨트랙트 내의 다른 함수에 의해 호출될 수 없다는 점을 제외하면 **public**과 동일하다.

-**internal**이나 **external**함수를 선언하는 건 **private**과 **public**함수를 선언하는 구문과 동일하다.

	contract Sandwich {
	  uint private sandwichesEaten = 0;
	
	  function eat() internal {
	    sandwichesEaten++;
	  }
	}
	
	contract BLT is Sandwich {
	  uint private baconSandwichesEaten = 0;
	
	  function eatWithBacon() public returns (string) {
	    baconSandwichesEaten++;
	    // eat 함수가 internal로 선언되었기 때문에 여기서 호출이 가능하다 
	    eat();
	  }
	}

#챕터 10: 좀비가 무엇을 먹나요?

##다른 컨트랙트와 상호 작용하기

- 블록체인 상에 있으면서 우리가 소유하지 않은 컨트랙트와 우리 컨트랙트가 상호작용을 하려면 우선 **인터페이스**를 정의해야 한다.

- 간단한 예시:
	
		contract LuckyNumber {
		  mapping(address => uint) numbers;
		
		  function setNum(uint _num) public {
		    numbers[msg.sender] = _num;
		  }
		
		  function getNum(address _myAddress) public view returns (uint) {
		    return numbers[_myAddress];
		  }
		}
- 이 컨트랙트는 아무나 자신의 행운의 수를 저장할 수 있는 간단한 컨트랙트이고, 각자의 이더리움 주소와 연관이 있다. 이 주소를 이용해서 누구나 그 사람의 행운의 수를 찾아 볼 수 있다.

- 이제 getNum 함수를 이용하여 이 컨트랙트에 있는 데이터를 읽고자 하는 external 함수가 있다고 해 보세.

- 먼저, LuckyNumber 컨트랙트의 인터페이스를 정의할 필요가 있다:
	
	contract NumberInterface {
	  function getNum(address _myAddress) public view returns (uint);
	}

- 인터페이스를 정의하는 것이 컨트랙트를 정의하는 것과 유사하다
- 먼저 컨트랙트와 상호작용하고자 하는 함수만 선언할 뿐(*getNum*함수) 다른 함수나 상태변수를 언급하지 않는다.
- 다음으로, 함수 몸체를 정의하지 않지. 중괄호 {, }를 쓰지 않고 함수 선언을 세미콜론(;)으로 간단하게 끝내지.
- 그러니 인터페이스는 컨트랙트 뼈대처럼 보인다고 할 수 있지. 컴파일러도 그렇게 인터페이스를 인식하지.


#챕터 11: 인터페이스 활용하기

- 아래와 같이 인터페이스가 정의 되면:
				
		contract NumberInterface {
		  function getNum(address _myAddress) public view returns (uint);
		}

- 다음과 같이 컨트랙트에서 인터페이스를 이용할 수 있다:

		contract MyContract {
		  address NumberInterfaceAddress = 0xab38...
		  // ^ 이더리움상의 FavoriteNumber 컨트랙트 주소이다
		  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress)
		  // 이제 `numberContract`는 다른 컨트랙트를 가리키고 있다.
		
		  function someFunction() public {
		    // 이제 `numberContract`가 가리키고 있는 컨트랙트에서 `getNum` 함수를 호출할 수 있다:
		    uint num = numberContract.getNum(msg.sender);
		    // ...그리고 여기서 `num`으로 무언가를 할 수 있다
		  }
		}

- 이런 식으로 컨트랙트는 이더리움 블록체인상의 다른 어떤 컨트랙트와도 상호 작용할 수 있다.
- 물론 상호작용하는 함수가 public이나 external로 선언되어야 한다.

#챕터 12: 다수의 반환값 처리하기

- **getKitty**함수는 우리가 살표 본 예시 중 유일하게 다수의 반환값을 갖는 함수이다.
		
		function multipleReturns() internal returns(uint a, uint b, uint c) {
		  return (1, 2, 3);
		}
		
		function processMultipleReturns() external {
		  uint a;
		  uint b;
		  uint c;
		  // 다음과 같이 다수 값을 할당한다:
		  (a, b, c) = multipleReturns();
		}
		
		// 혹은 단 하나의 값에만 관심이 있을 경우: 
		function getLastReturnValue() external {
		  uint c;
		  // 다른 필드는 빈칸으로 놓기만 하면 된다: 
		  (,,c) = multipleReturns();
		}

#챕터 13: 보너스: 키티 유전자

우리의 함수 로직이 이제 완료되었군... 하지만 한 가지를 보너스로 추가해 보도록 하세.

고양이 유전자와 조합되어 생성된 좀비가 몇 가지 독특한 특성을 가져서 고양이 좀비로 보이도록 해 보세.

이를 위해 좀비 DNA에 몇 가지 특별한 키티 코드를 추가할 수 있네.

레슨 1에서 배운 내용을 떠올려 보면, 좀비의 외모를 결정하는 데 있어서 16자리 DNA 중에서 처음 12자리만 이용되지. 그러니 마지막에서 2자리 숫자를 활용하여 "특별한" 특성을 만들어 보세.

고양이 좀비는 DNA 마지막 2자리로 99를 갖는다고 해 보세 (고양이는 9개의 목숨을 가졌다고 할 만큼 생명력이 강하므로). 그러면 우리 코드에서는 만약(if) 좀비가 고양이에서 생성되면 좀비 DNA의 마지막 2자리를 99로 설정한다.

	pragma solidity ^0.4.19;
	
	import "./zombiefactory.sol";
	
	contract KittyInterface {
	  function getKitty(uint256 _id) external view returns (
	    bool isGestating,
	    bool isReady,
	    uint256 cooldownIndex,
	    uint256 nextActionAt,
	    uint256 siringWithId,
	    uint256 birthTime,
	    uint256 matronId,
	    uint256 sireId,
	    uint256 generation,
	    uint256 genes
	  );
	}
	
	contract ZombieFeeding is ZombieFactory {
	
	  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
	  KittyInterface kittyContract = KittyInterface(ckAddress);
	
	  // 여기에 있는 함수 정의를 변경:
	  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
	    require(msg.sender == zombieToOwner[_zombieId]);
	    Zombie storage myZombie = zombies[_zombieId];
	    _targetDna = _targetDna % dnaModulus;
	    uint newDna = (myZombie.dna + _targetDna) / 2;
	    // 여기에 if 문 추가
	    if(keccak256(_species) == keccak256("kitty")){
	      newDna = newDna - newDna % 100 + 99;
	    }
	    _createZombie("NoName", newDna);
	  }
	
	  function feedOnKitty(uint _zombieId, uint _kittyId) public {
	    uint kittyDna;
	    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
	    // 여기에 있는 함수 호출을 변경: 
	    feedAndMultiply(_zombieId, kittyDna, "kitty");
	  }
	
	}

#챕터 14: 마무리하기 Wrapping it up

	
	var abi = /* abi generated by the compiler */
	var ZombieFeedingContract = web3.eth.contract(abi)
	var contractAddress = /* our contract address on Ethereum after deploying */
	var ZombieFeeding = ZombieFeedingContract.at(contractAddress)
	
	// 우리 좀비의 ID와 타겟 고양이 ID를 가지고 있다고 가정하면 
	let zombieId = 1;
	let kittyId = 1;
	
	// 크립토키티의 이미지를 얻기 위해 웹 API에 쿼리를 할 필요가 있다. 
	// 이 정보는 블록체인이 아닌 크립토키티 웹 서버에 저장되어 있다.
	// 모든 것이 블록체인에 저장되어 있으면 서버가 다운되거나 크립토키티 API가 바뀌는 것이나 
	// 크립토키티 회사가 크립토좀비를 싫어해서 고양이 이미지를 로딩하는 걸 막는 등을 걱정할 필요가 없다 ;) 
	let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
	$.get(apiUrl, function(data) {
	  let imgUrl = data.image_url
	  // 이미지를 제시하기 위해 무언가를 한다 
	})
	
	// 유저가 고양이를 클릭할 때:
	$(".kittyImage").click(function(e) {
	  // 우리 컨트랙트의 `feedOnKitty` 메소드를 호출한다 
	  ZombieFeeding.feedOnKitty(zombieId, kittyId)
	})
	
	// 우리의 컨트랙트에서 발생 가능한 NewZombie 이벤트에 귀를 기울여서 이벤트 발생 시 이벤트를 제시할 수 있도록 한다: 
	ZombieFactory.NewZombie(function(error, result) {
	  if (error) return
	  // 이 함수는 레슨 1에서와 같이 좀비를 제시한다: 
	  generateZombie(result.zombieId, result.name, result.dna)
	})