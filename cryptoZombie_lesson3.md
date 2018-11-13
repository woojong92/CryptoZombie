# [크립토좀비](https://cryptozombies.io) Lesson3를 학습하면서 정리한 내용입니다.<br> #

# 챕터 1: 컨트랙트의 불변성 #

- DApp은 일반적인 애플리케이션과 다른 특징들이 있다.

	1. 이더리움에 컨트랙트를 배포하고 나면, 컨트랙트는 변하지 안는다(Immutable). 즉, 배포된 컨트랙트를 수정하거나 업데이트 할 수 없다.
	
	2. 코드는 곧 법. 어떤 스마트 컨트랙트의 코드를 읽고 검증을 했다면, 함수를 호출할 때마다, 코드에 쓰여진 그대로 함수가 실행될 것이라고 확신할 수 있다.

## 외부 의존성 ##

레슨 2에서, 우리는 크립토키티 컨트랙트의 주소를 우리 DApp에 직접 써넣었네. 그런데 만약 크립토키티 컨트랙트에 버그가 있었고, 누군가 모든 고양이들을 파괴해버렸다면 어떻게 될 것 같은가?

그럴 일은 잘 없겠지만, 만약 그런 일이 발생한다면 우리의 DApp은 완전히 쓸모가 없어질 것이다. 우리 DApp은 주소를 코드에 직접 써넣기 때문에 어떤 고양이들도 받아올 수 없다. 우리 좀비들은 고양이를 먹을 수 없을 것이고, 우리는 그걸 고치기 위해 우리의 컨트랙트를 수정할 수도 없을 것이다.

이런 이유로, 대개의 경우 자네가 DApp의 중요한 일부를 수정할 수 있도록 하는 함수를 만들어놓는 것이 합리적이다.

예를 들자면 DApp에 크립토키티 컨트랙트 주소를 직접 써넣는 것 대신, 언젠가 크립토키티 컨트랙트에 문제가 생기면 해당 주소를 바꿀 수 있도록 해주는 setKittyContractAddress 함수를 만들 수 있다.

	pragma solidity ^0.4.19;

	contract ZombieFactory {

	    event NewZombie(uint zombieId, string name, uint dna);
	
	    uint dnaDigits = 16;
	    uint dnaModulus = 10 ** dnaDigits;
	
	    struct Zombie {
	        string name;
	        uint dna;
	    }
	
	    Zombie[] public zombies;
	
	    mapping (uint => address) public zombieToOwner;
	    mapping (address => uint) ownerZombieCount;
	
	    function _createZombie(string _name, uint _dna) internal {
	        uint id = zombies.push(Zombie(_name, _dna)) - 1;
	        zombieToOwner[id] = msg.sender;
	        ownerZombieCount[msg.sender]++;
	        NewZombie(id, _name, _dna);
	    }
	
	    function _generateRandomDna(string _str) private view returns (uint) {
	        uint rand = uint(keccak256(_str));
	        return rand % dnaModulus;
	    }
	
	    function createRandomZombie(string _name) public {
	        require(ownerZombieCount[msg.sender] == 0);
	        uint randDna = _generateRandomDna(_name);
	        randDna = randDna - randDna % 100;
	        _createZombie(_name, randDna);
	    }

	}

# 챕터 2: 소유 가능한 컨트랙트 #

- **setKittyContractAddress** 함수는 **external**이라, 누구든 이 함수를 호출할 수 있다.

- 이는 아무나 이 함수를 호출해서 크립트키티 컨트랙트의 주소를 바꿀 수 있고, 모든 사용자를 대상으로 우리 앱을 무용지물로 만들 수 있다는 것이다.

- 우리는 우리 컨트랙트에서 이 주소를 바꿀 수 있게끔 하고 싶지만, 그렇다고 모든 사람이 주소를 업데이트할 수 있기를 원하지는 않는다.

- 이런 경우에 대처하기 위해서, 최근에 주로 쓰는 하나의 방법은 **컨트랙트를 소유** 가능하게 만드는 것이다. **컨트랙트를 대상으로 특별한 권리를 가지는 소유자가 있음을 의미**하는 것이지.

# OpenZeppelin의 Ownable 컨트랙트 #

- 아래에 나와있는 것은 OpenZeppelin 솔리디티 라이브러리에서 가져온 Ownable 컨트랙트이다.

- OpenZeppelin은 자네의 DApp에서 사용할 수 있는, 안전하고 커뮤니티에서 검증받은 스마트 컨트랙트의 라이브러리 이다.
 
- 이 레슨 이후에, 자네가 레슨 4의 출시를 고대하며 기다리는 동안, 우린 자네가 저들의 사이트를 확인하고 더 학습하기를 추천하네!


		/**
		 * @title Ownable
		 * @dev The Ownable contract has an owner address, and provides basic authorization control
		 * functions, this simplifies the implementation of "user permissions".
		 */
		contract Ownable {
		  address public owner;
		  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
		
		  /**
		   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
		   * account.
		   */
		  function Ownable() public {
		    owner = msg.sender;
		  }
		
		  /**
		   * @dev Throws if called by any account other than the owner.
		   */
		  modifier onlyOwner() {
		    require(msg.sender == owner);
		    _;
		  }
		
		  /**
		   * @dev Allows the current owner to transfer control of the contract to a newOwner.
		   * @param newOwner The address to transfer ownership to.
		   */
		  function transferOwnership(address newOwner) public onlyOwner {
		    require(newOwner != address(0));
		    OwnershipTransferred(owner, newOwner);
		    owner = newOwner;
		  }
		}
	

- **생성장(Constructor)**:  **function Ownable()** 는 생성자이다. 컨트랙트와 동일한 이름을 가진, 생략할 수 있는 특별한 함수 이다. 이 함수는 컨트랙트가 생성될 때 딱 한번만 실행 된다.

- **함수 제어자(Function Modifier)** :  **modifier onlyOwner()**.  제어자는 다른 함수들에 대한 접근을 제어하기 위해 사용되는 일종의 유사 함수이다. 보통 함수 실행 전의 요구사항 충족 여부를 확인하는 데에 사용한다. **onlyOwner**의 경우에는 접근을 제한해서 오직 컨트랙트의 소유자만 해당 함수를 실행할 수 있도록 하기 위해 사용된다. 다음 챕터에서 함수제어자에 대해 더 살펴보고, **\_;**이 뭘 하는지 알아 볼 것

- **indexed** 키워드: ...

- 즉, **Ownable** 컨트랙트는 기본적으로 다음과 같은 것을 한다. :

	1. 컨트랙트가 생성되면 컨트랙트의 생성자가 **owner** 에 **msg.sender**(컨트랙트를 배포한 사람)를 대입한다.

	2. 특정한 함수들에 대해서 오직 소유자만 접근할 수 있도록 제한 가능한 **onlyOwner** 제어자를 추가한다.

	3. 새로운 소유자에게 해당 컨트랙트의 소유권을 옮길 수 있도록 한다.

- **onlyOwner**는 컨트랙트에서 흔히 쓰는 것 중 하나라, 대부분의 솔리디티 DApp들은 Ownable 컨트랙트를 복사/붙여넣기 하면서 시작한다. 그리고 첫 컨트랙트는 이 컨트랙트를 상속해서 만든다.
		
		pragma solidity ^0.4.19;
		
		// 1. 여기서 import하게
		import "./ownable.sol";
		
		// 2. 상속을 추가하게:
		contract ZombieFactory is Ownable {
		
		    event NewZombie(uint zombieId, string name, uint dna);
		
		    uint dnaDigits = 16;
		    uint dnaModulus = 10 ** dnaDigits;
		
		    struct Zombie {
		        string name;
		        uint dna;
		    }
		
		    Zombie[] public zombies;
		
		    mapping (uint => address) public zombieToOwner;
		    mapping (address => uint) ownerZombieCount;
		
		    function _createZombie(string _name, uint _dna) internal {
		        uint id = zombies.push(Zombie(_name, _dna)) - 1;
		        zombieToOwner[id] = msg.sender;
		        ownerZombieCount[msg.sender]++;
		        NewZombie(id, _name, _dna);
		    }
		
		    function _generateRandomDna(string _str) private view returns (uint) {
		        uint rand = uint(keccak256(_str));
		        return rand % dnaModulus;
		    }
		
		    function createRandomZombie(string _name) public {
		        require(ownerZombieCount[msg.sender] == 0);
		        uint randDna = _generateRandomDna(_name);
		        randDna = randDna - randDna % 100;
		        _createZombie(_name, randDna);
		    }
		}



# 챕터 3: onlyOwner 함수 제어자 #

- ZombeFactory가 Ownable을 상속하고, onlyOwner함수 제어자를 ZombieFeeding에서도 사용 할 수 이다.

		ZombieFeeding is ZombieFactory
		ZombieFactory is Ownable

- 때문에 ZombieFeeding 또한 Ownabble 이고, Ownable 컨트랙트의 함수/이벤트/제어자에 접근할 수 있다.

## 함수 제어자 ##

- 함수 제어자는 함수처럼 보이지만, function 키워드 대신 **modifier** 키워드를 사용한다.

- 함수를 호출하듯이 직접 호출할 수는 없다. 

- 대신에 함수 정의부 끝에 해당 함수의 작동 방식을 바꾸도록 제어자의 이름을 붙일 수 있다.

- **onlyOwner :**

		/**
		 * @dev Throws if called by any account other than the owner.
		 */
		modifier onlyOwner() {
		  require(msg.sender == owner);
		  _;
		}

- 우리는 이 제어자를 다음과 같이 사용할 것이다: 

		contract MyContract is Ownable {
		  event LaughManiacally(string laughter);
		
		  // 아래 `onlyOwner`의 사용 방법을 잘 보게:
		  function likeABoss() external onlyOwner {
		    LaughManiacally("Muahahahaha");
		  }
		}

- **likeABoss** 함수의 **onlyOwner** 제어자 부분을 살펴보자. **likeABoss** 함수를 호출하면, **onlyOwner**의 코드가 먼저 실행된다. 그리고 **onlyOwner**의 **\_;** 부분을 **likeABoss** 함수로 되돌아가 해당 코드를 실행하게 된다.

- 당신이 제어자를 사용할 수 있는 다양한 방법이 있지만, 가장 일반적으로 쓰는 예시 중 하나는 함수 실행 전에 require 체크를 넣는 것이다.

- onlyOwner의 경우에는, 함수에 이 제어자를 추가하면 오직 컨트랙트의 소유자(자네가 배포했다면 자네겠지)만이 해당 함수를 호출할 수 있다.

> 참고: 이렇게 소유자가 컨트랙트에 특별한 권한을 갖도록 하는 것은 자주 필요하지만, 이게 악용될 수도 있다. 예를 들어, 소유자가 다른 사람의 좀비를 뺏어올 수 있도록 하는 백도어 함수를 추가할 수도 있다!
> 
> 이더리움에서 돌아가는 DApp이라고 해서 그것만으로 분산화되어 있다고 할 수는 없다. 반드시 전체 소스 코드를 읽어보고, 자네가 잠재적으로 걱정할 만한, **소유자에 의한 특별한 제어가 불가능한 상태인지 확인**하라. 개발자로서는 당신이 잠재적인 버그를 수정하고 DApp을 안정적으로 유지하도록 하는 것과, 사용자들이 그들의 데이터를 믿고 저장할 수 있는 소유자가 없는 플랫폼을 만드는 것 사이에서 균형을 잘 잡는 것이 중요하다.
> 

		
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
		
		  KittyInterface kittyContract;
		
		  // 이 함수를 수정하게:
		  function setKittyContractAddress(address _address) external onlyOwner {
		    kittyContract = KittyInterface(_address);
		  }
		
		  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    Zombie storage myZombie = zombies[_zombieId];
		    _targetDna = _targetDna % dnaModulus;
		    uint newDna = (myZombie.dna + _targetDna) / 2;
		    if (keccak256(_species) == keccak256("kitty")) {
		      newDna = newDna - newDna % 100 + 99;
		    }
		    _createZombie("NoName", newDna);
		  }
		
		  function feedOnKitty(uint _zombieId, uint _kittyId) public {
		    uint kittyDna;
		    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
		    feedAndMultiply(_zombieId, kittyDna, "kitty");
		  }
		
		}

# 챕터 4: 가스(Gas) #

## 가스 - 이더리움 DApp이 사용하는 연료 ##
솔리디티에서는 사용자들이 자네가 만든 DApp의 함수를 실행할 때마다 _가스_라고 불리는 화폐를 지불해야 하네. 사용자는 이더(ETH, 이더리움의 화폐)를 이용해서 가스를 사기 때문에, 자네의 DApp 함수를 실행하려면 사용자들은 ETH를 소모해야만 하네.

함수를 실행하는 데에 얼마나 많은 가스가 필요한지는 그 함수의 로직(논리 구조)이 얼마나 복잡한지에 따라 달라지네. 각각의 연산은 소모되는 가스 비용(gas cost)이 있고, 그 연산을 수행하는 데에 소모되는 컴퓨팅 자원의 양이 이 비용을 결정하네. 예를 들어, storage에 값을 쓰는 것은 두 개의 정수를 더하는 것보다 훨씬 비용이 높네. 자네 함수의 전체 가스 비용은 그 함수를 구성하는 개별 연산들의 가스 비용을 모두 합친 것과 같네.

함수를 실행하는 것은 자네의 사용자들에게 실제 돈을 쓰게 하기 때문에, 이더리움에서 코드 최적화는 다른 프로그래밍 언어들에 비해 훨씬 더 중요하네. 만약 자네의 코드가 엉망이라면, 사용자들은 자네의 함수를 실행하기 위해 일종의 할증료를 더 내야 할 걸세. 그리고 수천 명의 사용자가 이런 불필요한 비용을 낸다면 할증료가 수십 억 원까지 쌓일 수 있지.

## 가스는 왜 필요한가? ##
이더리움은 크고 느린, 하지만 굉장히 안전한 컴퓨터와 같다고 할 수 있네. 자네가 어떤 함수를 실행할 때, 네트워크상의 모든 개별 노드가 함수의 출력값을 검증하기 위해 그 함수를 실행해야 하지. 모든 함수의 실행을 검증하는 수천 개의 노드가 바로 이더리움을 분산화하고, 데이터를 보존하며 누군가 검열할 수 없도록 하는 요소이지.
<br>
이더리움을 만든 사람들은 누군가가 무한 반복문을 써서 네트워크를 방해하거나, 자원 소모가 큰 연산을 써서 네트워크 자원을 모두 사용하지 못하도록 만들길 원했다네. 그래서 그들은 연산 처리에 비용이 들도록 만들었고, 사용자들은 저장 공간 뿐만 아니라 연산 사용 시간에 따라서도 비용을 지불해야 한다네.
<br>

> 참고: **사이드체인**에서는 반드시 이렇지는 않다네. 크립토좀비를 만든 사람들이 Loom Network에서 만들고 있는 것들이 좋은 예시가 되겠군. 이더리움 메인넷에서 월드 오브 워크래프트 같은 게임을 직접적으로 돌리는 것은 절대 말이 되지 않을 걸세. 가스 비용이 엄청나게 높을 것이기 때문이지. 하지만 다른 합의 알고리즘을 가진 사이드체인에서는 가능할 수 있지. 우린 다음에 나올 레슨에서 DApp을 사이드체인에 올릴지, 이더리움 메인넷에 올릴지 판단하는 방법들에 대해 더 얘기할 걸세.

## 가스를 아끼기 위한 구조체 압축 ##

- 기본적으로는 이런 하위 타입들을 쓰는 것은 아무런 이득이 없다. 왜냐하면 솔리디티에서는 uint의 크기에 상관없이 256비트의 저장 공간을 미리 잡아놓기 때문이다. 예를 들자면, uint(uint256) 대신에 uint8을 쓰는 것은 가스 소모를 줄이는 데에 아무 영향이 없다.

- 하지만 **예외**가 있다. 바로 **struct**안에서 이다.
- 솔리디티에서는 struct 안의 변수들을 더 적은 공간을 차지하도록 압축할 것이다.

		struct NormalStruct {
		  uint a;
		  uint b;
		  uint c;
		}
		
		struct MiniMe {
		  uint32 a;
		  uint32 b;
		  uint c;
		}
		
		// `mini`는 구조체 압축을 했기 때문에 `normal`보다 가스를 조금 사용할 것이다.
		NormalStruct normal = NormalStruct(10, 20, 30);
		MiniMe mini = MiniMe(10, 20, 30); 

# 챕터 5: 시간 단위 #

## 시간 단위(Time units) ##

- 솔리디티는 시간을 다룰 수 있는 단위계를 기본적으로 제공한다.

- **now** 변수를 쓰면 현재의 유닉스 타임스탬프(1970년 1월 1일부터 지금까지의 초 단위 합)값을 얻을 수 있다.

	
	> 참고: 유닉스 타임은 전통적으로 32비트 숫자로 저장되네. 이는 유닉스 타임스탬프 값이 32비트로 표시가 되지 않을 만큼 커졌을 때 많은 구형 시스템에 문제가 발생할 "Year 2038" 문제를 일으킬 것이네. 그러니 만약 우리 DApp이 지금부터 20년 이상 운영되길 원한다면, 우리는 64비트 숫자를 써야 할 것이네. 하지만 우리 유저들은 그동안 더 많은 가스를 소모해야 하겠지. 설계를 보고 결정을 해야 하네!
	
- 솔리디티는 또한 **seconds**, **minutes**, **hours**, **days**, **weeks**, **years** 같은 시간 단위 또한 포함하고 있다네. 이들은 그에 해당하는 길이 만큼의 **초 단위 uint 숫자로 변환**된다. 즉 1 minutes는 60, 1 hours는 3600(60초 x 60 분), 1 days는 86400(24시간 x 60분 x 60초) 같이 변환된다.

		uint lastUpdated;
		
		// `lastUpdated`를 `now`로 설정
		function updateTimestamp() public {
		  lastUpdated = now;
		}
		
		// 마지막으로 `updateTimestamp`가 호출된 뒤 5분이 지났으면 `true`를, 5분이 아직 지나지 않았으면 `false`를 반환
		function fiveMinutesHavePassed() public view returns (bool) {
		  return (now >= (lastUpdated + 5 minutes));
		}


	> 참고: now가 기본적으로 uint256을 반환하기 때문에, uint32(...) 부분이 필수적이네. 이렇게 함으로써 해당 데이터를 uint32로 명시적으로 변환하는 것이지.
		
		pragma solidity ^0.4.19;
		
		import "./ownable.sol";
		
		contract ZombieFactory is Ownable {
		
		    event NewZombie(uint zombieId, string name, uint dna);
		
		    uint dnaDigits = 16;
		    uint dnaModulus = 10 ** dnaDigits;
		    // 1. `cooldownTime`을 여기에 정의하게
		    uint cooldownTime = 1 days;
		
		
		    struct Zombie {
		        string name;
		        uint dna;
		        uint32 level;
		        uint32 readyTime;
		    }
		
		    Zombie[] public zombies;
		
		    mapping (uint => address) public zombieToOwner;
		    mapping (address => uint) ownerZombieCount;
		
		    function _createZombie(string _name, uint _dna) internal {
		        // 2. 아래 줄을 업데이트하게:
		        uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime) )) - 1;
		        zombieToOwner[id] = msg.sender;
		        ownerZombieCount[msg.sender]++;
		        NewZombie(id, _name, _dna);
		    }
		
		    function _generateRandomDna(string _str) private view returns (uint) {
		        uint rand = uint(keccak256(_str));
		        return rand % dnaModulus;
		    }
		
		    function createRandomZombie(string _name) public {
		        require(ownerZombieCount[msg.sender] == 0);
		        uint randDna = _generateRandomDna(_name);
		        randDna = randDna - randDna % 100;
		        _createZombie(_name, randDna);
		    }

- **now + cooldownTime**은 현재 유닉스 타임스탬프(초 단위)에 1일을 초 단위로 바꾼 것의 합과 같을 것이다. 바꿔 말해 지금부터 하루 뒤의 유닉스 타임스탬프 값과 같은 것이다. 이후에 우리는 좀비를 다시 사용하기 위해 충분한 시간이 지났는지 확인할 수 있도록 좀비의 readyTime이 now보다 큰지 비교할 것이다.

# 챕터 6: 좀비 재사용 대기 시간 #

## 구조체를 인수로 전달하기 ##

- private 또는 interanl 함수에 인수로서 구조체의 storage 포인터를 전달 할 수 있다.
 
- 문법의 예:
		
		function _doStuff(Zombie storage _zombie) internal {
		  // _zombie로 할 수 있는 것들을 처리
		}

## 직접해보기 ##

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
		
		  KittyInterface kittyContract;
		
		  function setKittyContractAddress(address _address) external onlyOwner {
		    kittyContract = KittyInterface(_address);
		  }
		
		  // 1. `_triggerCooldown` 함수를 여기에 정의하게
		  function _triggerCooldown(Zombie storage _zombie) internal {
		      _zombie.readyTime = uint32(now + cooldownTime);
		  }
		
		  // 2. `_isReady` 함수를 여기에 정의하게
		  function _isReady(Zombie storage _zombie) internal view returns(bool){
		      return (_zombie.readyTime <= now);
		  }
				
		  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    Zombie storage myZombie = zombies[_zombieId];
		    _targetDna = _targetDna % dnaModulus;
		    uint newDna = (myZombie.dna + _targetDna) / 2;
		    if (keccak256(_species) == keccak256("kitty")) {
		      newDna = newDna - newDna % 100 + 99;
		    }
		    _createZombie("NoName", newDna);
		  }
		
		  function feedOnKitty(uint _zombieId, uint _kittyId) public {
		    uint kittyDna;
		    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
		    feedAndMultiply(_zombieId, kittyDna, "kitty");
		  }
		
		}

# 챕터 7: Public 함수 & 보안 #

- 이제 **feedAndMultiply**를 우리의 재사용 대기 시간 타이머를 고려하도록 수정해보자.

- 이 함수를 다시 살펴보면, 우리가 이전 레슨에서 이 함수를 **public**으로 만들었던 것을 볼 수 있다. 

- **보안을 점검하는 좋은 방법**은 당신의 모든 public과 external 함수를 검사하고, 사용자들이 그 함수들을 남용할 수 있는 방법을 생각해보는 것이다. 

- **함수들이 onlyOwner 같은 제어자를 갖지 않는 이상, 어떤 사용자든 이 함수들을 호출하고 자신들이 원하는 모든 데이터를 함수에 전달할 수 있다.**

- **feedAndMultiply** 함수를 다시 살펴보면, 사용자들은 이 함수를 직접적으로 호출할 수 있고 그들이 원하는 아무 **\_targetDna**나 **\_species**를 전달할 수 있네. 

- 좀 더 자세히 들여보면, **feedAndMultiply** 함수는 오직 **feedOnKitty()**에 의해서만 호출이 될 필요가 있다. 그러니 이런 남용을 막을 가장 쉬운 방법은 이 함수를 **internal**로 만드는 것이지.
		
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
		
		  KittyInterface kittyContract;
		
		  function setKittyContractAddress(address _address) external onlyOwner {
		    kittyContract = KittyInterface(_address);
		  }
		
		  function _triggerCooldown(Zombie storage _zombie) internal {
		    _zombie.readyTime = uint32(now + cooldownTime);
		  }
		
		  function _isReady(Zombie storage _zombie) internal view returns (bool) {
		      return (_zombie.readyTime <= now);
		  }
		
		  // 1. 이 함수를 internal로 만들게
		  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    Zombie storage myZombie = zombies[_zombieId];
		    // 2. 여기에 `_isReady`를 확인하는 부분을 추가하게
		    require(_isReady(myZombie));
		    _targetDna = _targetDna % dnaModulus;
		    uint newDna = (myZombie.dna + _targetDna) / 2;
		    if (keccak256(_species) == keccak256("kitty")) {
		      newDna = newDna - newDna % 100 + 99;
		    }
		    _createZombie("NoName", newDna);
		    // 3. `triggerCooldown`을 호출하게
		    _triggerCooldown(myZombie);
		  }
		
		  function feedOnKitty(uint _zombieId, uint _kittyId) public {
		    uint kittyDna;
		    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
		    feedAndMultiply(_zombieId, kittyDna, "kitty");
		  }
		
		}

# 챕터 8: 함수 제어자의 또 다른 특징 #

- **추가적인 헬퍼 메소드**를 추가하자.
- 좀비들이 특정 레벨에 도달하면 특별한 능력들을 얻을 수 있도록 만든다.

## 인수를 가지는 함수 제어자 ##

- 이전에는 onlyOwner라는 간단한 예시를 살펴보았다. 하지만 **함수 제어자는 사실 인수 또한 받을 수 있다.** 예를 들면:

		
		// 사용자의 나이를 저장하기 위한 매핑
		mapping (uint => uint) public age;
		
		// 사용자가 특정 나이 이상인지 확인하는 제어자
		modifier olderThan(uint _age, uint _userId) {
		  require (age[_userId] >= _age);
		  _;
		}
		
		// 차를 운전하기 위햐서는 16살 이상이어야 하네(적어도 미국에서는).
		// `olderThan` 제어자를 인수와 함께 호출하려면 이렇게 하면 되네:
		function driveCar(uint _userId) public olderThan(16, _userId) {z
		  // 필요한 함수 내용들
		}

- 여기서 당신은 **olderthan 제어자**가 함수와 비슷하게 인수를 받는 것을 볼 수 있다. 그리고 **driveCar** 함수는 받은 인수를 제어자로 전달하고 있다.

- 이제 특별한 능력에 제한을 걸 수 있도록 좀비의 level 속성을 사용하는 우리만의 **modifier**를 만들어보자.
		
		pragma solidity ^0.4.19;
		
		import "./zombiefeeding.sol";
		
		contract ZombieHelper is ZombieFeeding {
		
		  modifier aboveLevel(uint _level, uint _zombieId){
		      require (zombies[_zombieId].level >= _level);
		      _;
		  }
		
		}

# 챕터 9: 좀비 제어자 #


		pragma solidity ^0.4.19;
		
		import "./zombiefeeding.sol";
		
		contract ZombieHelper is ZombieFeeding {
		
		  modifier aboveLevel(uint _level, uint _zombieId) {
		    require(zombies[_zombieId].level >= _level);
		    _;
		  }
		
		  function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    zombies[_zombieId].name = _newName;
		  }
		
		  function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    zombies[_zombieId].dna = _newDna;
		  }
		
		}


# 챕터 10: 'View' 함수를 사용해 가스 절약하기 #

## View 함수는 가스를 소모하지 않는다. ##

- **view**는 사용자에 의해 외부에서 호출되었을 때 가스를 전혀 소모하지 않는다.
- 이건 view 함수가 블록체인 상에서 실제로 어떤 것도 수정하지 않기 때문이다. 
- view표시를 하는 것은 web3.js에 이렇게 말하는 것과 같다. "이 함수는 실행할 때 자네 로컬 이더리움 노드에 질의만 날리면 되고, 블록체인에 어떤 트랜잭션도 만들지 않아"(트랜잭션은 모든 개별 노드에서 실행되어야 하고, 가스를 소모한다.)


> 참고: 만약 view 함수가 동일 컨트랙트 내에 있는, view 함수가 아닌 다른 함수에서 내부적으로 호출될 경우, 여전히 가스를 소모할 것이네. 이것은 다른 함수가 이더리움에 트랜잭션을 생성하고, 이는 모든 개별 노드에서 검증되어야 하기 때문이네. 그러니 view 함수는 외부에서 호출됐을 때에만 무료라네.

# 챕터 11: Storage는 비싸다. ##

- 솔리디티에서 더 비싼 연산 중 하나는 바로 storage를 쓰는 것이며, 그 중에서도 쓰기 연산이 가장 비싸다.

- 데이터의 일부를 쓰거나 바꿀 때마다, 블록체인에 영구적으로 기록되기 때문에 지구상의 수천개의 노드들이 그들의 하드 드라이브에 그 데이터를 저장해야하고, 블록체인이 커져 가면서 이 데이터의 양 또한 같이 커져간다. 

- 비용을 최소화 하기 위해선, 진짜 필요한 경우가 아니면 storage에 데이터를 쓰지 않는 것이 좋다.

- 이를 위해 때때로 겉보기에 비효율적으로 보이는 프로그래밍 구성을 할 필요가 있다. 

- 어떤 배열에서 내용을 빠르게 찾기 위해, 단순히 변수에 저장하는 것 대신 함수가 호출 될 때마다 배열을 memory에 다시 만드는 것처럼.

- 대부분의 프로그래밍 언어에서는, 큰 데이터 집합의 개별 데이터에 모두 접근하는 것은 비용이 비싸다. 하지만 솔리디티에서는 그 접근이 external view함수라면, storage를 사용하는 것보다 더 저렴한 방법이다.

## 메모리에 배열 선언하기 ##

- Storage에 아무것도 쓰지 않고도 함수 안에 새로운 배열을 만들려면 배열에 **memory** 키워드를 쓰면 된다. 

- 이 배열은 함수가 끝날 때까지만 존재할 것이고, 이는 storage의 배열을 직접 업데이트하는 것보다 가스 소모 측면에서 훨씬 저렴하다.

- 외부에서 호출되는 view 함수라면 무료이다.

- 메모리에 배열을 선언하는 방법:
	
		function getArray() external pure returns(uint[]) {
		  // 메모리에 길이 3의 새로운 배열을 생성한다.
		  uint[] memory values = new uint[](3);
		  // 여기에 특정한 값들을 넣는다.
		  values.push(1);
		  values.push(2);
		  values.push(3);
		  // 해당 배열을 반환한다.
		  return values;
		}	

> 참고: 메모리 배열은 반드시 길이 인수와 함께 생성되어야 한다(이 예시에서는, 3). 메모리 배열은 현재로서는 storage 배열처럼 array.push()로 크기가 조절되지는 않느다. 이후 버전의 솔리디티에서는 변경될 수도 있다.

## 직접해보기 ##

- **getZombiesByOwner** 함수에서, 우리는 특정한 사용자가 소유한 모든 좀비를 **uint[]** 배열로 반환하기를 원하네.

1. result라는 이름의 **uint[] memory 변수**를 선언하게.

2. 해당 변수에 uint 배열을 대입하게. 배열의 길이는 이 _owner가 소유한 좀비의 개수여야 하고, 이는 우리의 mapping인 ownerZombieCount[_owner]를 통해서 찾을 수 있네.

3. 함수의 끝에서 result를 반환해라. 지금 당장은 빈 배열이지만, 다음 챕터에서 이를 채울 것이다.
		
		pragma solidity ^0.4.19;
		
		import "./zombiefeeding.sol";
		
		contract ZombieHelper is ZombieFeeding {
		
		  modifier aboveLevel(uint _level, uint _zombieId) {
		    require(zombies[_zombieId].level >= _level);
		    _;
		  }
		
		  function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    zombies[_zombieId].name = _newName;
		  }
		
		  function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
		    require(msg.sender == zombieToOwner[_zombieId]);
		    zombies[_zombieId].dna = _newDna;
		  }
		
		  function getZombiesByOwner(address _owner) external view returns(uint[]) {
		      uint[] memory result = new uint[](ownerZombieCount[_owner]);
		      return result
		  }
		
		}

# 챕터 12: For 반복문 #
 
- 배열을 다룰 때, 그냥 storage에 해당 배열을 저장하는 것이 아니라 **for** 반복문을 사용해서 구성해야 한다.

- **getZombiesByOwner**를 구현할 때, 기초적인 구현 방법은 **zombieFactory 컨트랙트**에서 소유자의 좀비 군대에 대한 **mapping**을 만들어 저장하는 것이다.

		mapping (address => uint[]) public ownerToZombies

- 그리고나서 새로운 좀비를 만들 때마다, 해당 소유자의 좀비 배열에 **ownerToZombies[owner].push(zombieId)**를 사용해서 새 좀비를 추가 하는 것이다:

		function getZombiesByOwner(address _owner) external view returns (uint[]) {
		  return ownerToZombies[_owner];
		}

## 이 방식의 문제 ##
- 이러한 접근 방법은 구현의 간단함 때문에 매력적으로 보인다.
 
- 하지만 나중에 한 좀비를 원래 소유자에서 다른 사람에게 전달하는 함수를 구현하게 된다면 어떤 일이 일어날지 생각해 보라.

- 좀비 전달 함수에서 필요한 내용:

	1. 전달할 좀비를 새로운 소유자의 ownerToZombies 배열에 넣는다.
	1. 기존 소유자의 ownerToZombies 배열에서 해당 좀비를 지운다.
	1. 좀비가 지워진 구멍을 메우기 위해 기존 소유자의 배열에서 모든 좀비를 한 칸씩 움직인다.
	1. 배열의 길이를 1 줄인다.

- 솔리디티에서 storage에 쓰는 것은 가장 비용이 높은 연산 중 하나이기 때문에, 이 전달 함수에 대한 모든 호출은 가스 측면에서 굉장히 비싸다. 

- 더 안 좋은 점은, 이 함수가 실행될 때마다 다른 양의 가스를 소모할 것이라는 점이다. 사용자가 자신의 군대에 얼마나 많은 좀비를 가지고 있는지, 또 거래되는 좀비의 인덱스에 따라 달라진다. 즉 사용자들은 거래에 가스를 얼마나 쓰게 될지 알 수 없게 된다.

	>참고: 물론, 빈 자리를 채우기 위해 마지막 좀비를 움직인 다음, 배열의 길이를 하나 줄여도 되겠지. 하지만 그렇게 하면 교환이 일어날 때마다 좀비 군대의 순서가 바뀌게 될 것이다.

- **view** 함수는 외부에서 호출될 때 가스를 사용하지 않기 때문에, 우린 **getZombiesByOwner** 함수에서 **for** 반복문을 사용해서 좀비 배열의 모든 요소에 접근한 후 특정 사용자의 좀비들로 구성된 배열을 만들 수 있있다. 그러고 나면 **transfer** 함수는 훨씬 비용을 적게 쓰게 된다. 왜냐하면 **storage**에서 어떤 배열도 재정렬할 필요가 없기 때문이다. 일반적인 직관과는 반대로 이런 접근법이 전체적으로 비용 소모가 더 적다.

## for 반복문 사용하기 ##

- 솔리디티에서 for 반복문의 문법은 자바스크립트와 비슷하다.
		
		function getEvens() pure external returns(uint[]) {
		  uint[] memory evens = new uint[](5);
		  // 새로운 배열의 인덱스를 추적하는 변수
		  uint counter = 0;
		  // for 반복문에서 1부터 10까지 반복함
		  for (uint i = 1; i <= 10; i++) {
		    // `i`가 짝수라면...
		    if (i % 2 == 0) {
		      // 배열에 i를 추가함
		      evens[counter] = i;
		      // `evens`의 다음 빈 인덱스 값으로 counter를 증가시킴
		      counter++;
		    }
		  }
		  return evens;
		}

# 챕터 13: 마무리하기 #

- 요약하자면:

	- CryptoKitties 컨트랙트를 변경하는 한 가지 방법을 배웠다.
	- onlyOwner를 이용해 핵심적인 함수를 보호하는 방법을 배웠다.
	- 가스와 가스 사용 최적화에 대해 배웠다.
	- 우리 좀비들에게 레벨과 대기시간 개념을 적용했다.
	- 좀비가 특정 레벨이 되면 좀비의 이름과 DNA를 재설정할 수 있는 함수를 만들었다.
	- 마지막으로, 사용자의 좀비 군대를 반환하는 함수를 만들었다.
	
