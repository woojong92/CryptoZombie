# [크립토좀비](https://cryptozombies.io) Lesson4를 학습하면서 정리한 내용입니다.<br> #

# 챕터1: Payable

#함수 제어자(function modifier)복습

1. 우린 함수가 언제, 어디서 호출될 수 있는지 제어하는 접근 제어자(visibility modifier)를 알게 되었네: **private**은 컨트랙트 내부의 다른 함수들에서만 호출될 수 있음을 의미하지. **internal**은 **private**과 비슷하지만, 해당 컨트랙트를 상속하는 컨트랙트에서도 호출될 수 있지. external은 오직 컨트랙트 외부에서만 호출될 수 있네. 마지막으로 **public**은 내외부 모두에서, 어디서든 호출될 수 있네.

2. 또한 상태 제어자(state modifier)에 대해서도 배웠네. 이 제어자는 블록체인과 상호작용 하는 방법에 대해 알려주지:** view**는 해당 함수를 실행해도 어떤 데이터도 저장/변경되지 않음을 알려주지. **pure**는 해당 함수가 어떤 데이터도 블록체인에 저장하지 않을 뿐만 아니라, 블록체인으로부터 어떤 데이터도 읽지 않음을 알려주지. 이들 모두는 컨트랙트 외부에서 불렸을 때 가스를 전혀 소모하지 않네(하지만 다른 함수에 의해 내부적으로 호출됐을 경우에는 가스를 소모하지).

3. 그리고 사용자 정의 제어자에 대해서도 배웠네. 레슨 3에서 배웠던 것이지. 예를 들자면 o**nlyOwner**와 **aboveLevel** 같은 것이지. 이런 제어자를 사용해서 우린 함수에 이 제어자들이 어떻게 영향을 줄지를 결정하는 우리만의 논리를 구성할 수 있네.

##payable 제어자
- payable 함수는 이더를 받을 수 있는 특별한 함수 유형이다.
- 일반적인 웹 서버에서 API 함수를 실행할 때에는, 함수 호출을 통해서 US달러를 보낼 수 없다.(비트고인도)

- 하지만, 이더리움에서는, 돈(_이더_), 데이터(transaction payload), 그리고 컨트랙트 코드 자체 모두 이더리움 위에 존재하기 때문에, 함수를 실행하는 동시에 컨트랙트에 돈을 지불하는 것이 가능하다.
- 이를 통해 굉장히 흥미로운 구성을 만들어 낼 수 있다. 함수를 실행하기 위해 컨트랙트에 일정 금액을 지불하게 하는 것과 같이 말이다.

- 예시:
		
		contract OnlineStore {
		  function buySomething() external payable {
		    // 함수 실행에 0.001이더가 보내졌는지 확실히 하기 위해 확인:
		    require(msg.value == 0.001 ether);
		    // 보내졌다면, 함수를 호출한 자에게 디지털 아이템을 전달하기 위한 내용 구성:
		    transferThing(msg.sender);
		  }
		}

- 여기서, msg.value는 컨트랙트로 이더가 얼마나 보내졌는지 확인하는 방법이고, ether는 기본적으로 포함된 단위이네.
 
- 여기서 일어나는 일은 누군가 web3.js(DApp의 자바스크립트 프론트엔드)에서 다음과 같이 함수를 실행할 때 발생한다:
		
		// `OnlineStore`는 자네의 이더리움 상의 컨트랙트를 가리킨다고 가정하네:
		OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})

- **value**필드를 주목해라. 자바스크립트 함수 호출에서 이 필드를 통해 **ether**를 얼마나 보낼지 결정한다. 트랜잭션을 봉투로 생각하고, 함수 호출에 전달하는 매개 변수를 편지의 내용이라 생각한다면, value는 봉투 안에 현금을 넣은 것과 같다. 즉, 편지와 돈이 모두 수령인에게 전달된다.

> 참고: 만약 함수가 payable로 표시되지 않았는데 자네가 위에서 본 것처럼 이더를 보내려 한다면, 함수에서 자네의 트랜잭션을 거부할 것이네.
		
	pragma solidity ^0.4.19;
	
	import "./zombiefeeding.sol";
	
	contract ZombieHelper is ZombieFeeding {
	
	  // 1. 여기에 levelUpFee를 정의하게
	  uint levelUpFee = 0.001 ether;
	
	  modifier aboveLevel(uint _level, uint _zombieId) {
	    require(zombies[_zombieId].level >= _level);
	    _;
	  }
	
	  // 2. 여기에 levelUp 함수를 삽입하게
	  function levelUp(uint _zombieId) external payable{
	    require( msg.value == levelUpFee);
	    zombies[_zombieId].level++;
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
	    uint counter = 0;
	    for (uint i = 0; i < zombies.length; i++) {
	      if (zombieToOwner[i] == _owner) {
	        result[counter] = i;
	        counter++;
	      }
	    }
	    return result;
	  }
	
	}


#챕터 2: 출금

- 이더를 보낸 다음에는 어떤 일이 일어날까?
- 컨트랙트로 이더를 보내면, 해당 컨트랙트의 이더리움 계좌에 이더가 저장되고 거기에 갇히게 된다.
- 따라서 컨트랙트로부터 이더를 인출하는 함수를 만들어야 한다.

-다음과 같이 컨트랙트에서 이더를 인출하는 함수를 작성할 수 있다:

		contract GetPaid is Ownable {
		  function withdraw() external onlyOwner {
		    owner.transfer(this.balance);
		  }
		}

- 우리가 **Ownable** 컨트랙트를 import 했다고 가정하고 owner와 onlyOwner를 사용하고 있다.
- **transfer**  함수를 사용하여 이더를 특정 주소로 전달할 수 있다.
- 그리고 this.balance는 컨트랙트에 저장돼 있는 전체 잔액을 반환한다. 

- **transfer** 함수를 써서 특정한 이더리움 주소에 돈을 보낼 수 있다. 
- 예를 들어, 만약 누군가 한 아이템에 대해 초과 지불을 했다면, 이더를 msg.sender로 되돌려주는 함수를 만들수 있다:
		
		uint itemFee = 0.001 ether;
		msg.sender.transfer(msg.value - itemFee);

- 혹은 구매자와 판매자가 존재하는 컨트랙트에서, 판매자의 주소를 storage에 저장하고, 누군가 판매자의 아이템을 구매하면 구매자로부터 받은 요금을 그에게 전달할 수 도 있다:

		seller.transfer(msg.value);


#챕터 3: 좀비 전투

#챕터 4: 난수(Random Number)

##keccack256을 통한 난수 생성

- 솔리디티에서 난수를 만들기에 가장 좋은 방법은 **keccak256** 해시 함수를 쓰는 것이다.
		
		// Generate a random number between 1 and 100:
		uint randNonce = 0;
		uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
		randNonce++;
		uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;

- 위 예제는 **now**의 타임스탬프 값, **msg.sender**, 증가하는 **nonce**(딱 한 번만 사용되는 숫자, 즉 똑같은 입력으로 두번이상 동일한 해시 함수를 실행 할 수 없게 함.)를 인자로 하고 있다.
- 그리고 **keccak**을 사용하여 이 입력들을 임의의 해시값으로 변환하고, 변환한 해시값은 **uint**로 바꾼 후, **% 100**을 써서 마지막 2자리 숫자만 받도록 했다. 
- 이를 통해 0과 99사이의 완전한 난수를 얻을 수 있다.

## 이 메소드는 정직하지 않은 노드의 공격에 취약하다.

이더리움에서는 자네가 컨트랙트의 함수를 실행하면 트랜잭션(transaction)으로서 네트워크의 노드 하나 혹은 여러 노드에 실행을 알리게 되네. 그 후 네트워크의 노드들은 여러 개의 트랜잭션을 모으고, "작업 증명"으로 알려진 계산이 매우 복잡한 수학적 문제를 먼저 풀기 위한 시도를 하게 되네. 그리고서 해당 트랜잭션 그룹을 그들의 작업 증명(PoW)과 함께 _블록_으로 네트워크에 배포하게 되지.

한 노드가 어떤 PoW를 풀면, 다른 노드들은 그 PoW를 풀려는 시도를 멈추고 해당 노드가 보낸 트랜잭션 목록이 유효한 것인지 검증하네. 유효하다면 해당 블록을 받아들이고 다음 블록을 풀기 시작하지.

이것이 우리의 난수 함수를 취약하게 만드네.

우리가 동전 던지기 컨트랙트를 사용한다고 해보지 - 앞면이 나오면 돈이 두 배가 되고, 뒷면이 나오면 모두 다 잃는 것이네. 앞뒷면을 결정할 때 위에서 본 난수 함수를 사용한다고 가정해보세. (random >= 50은 앞면, random < 50은 뒷면이네).

내가 만약 노드를 실행하고 있다면, 나는 오직 나의 노드에만 트랜잭션을 알리고 이것을 공유하지 않을 수 있네. 그 후 내가 이기는지 확인하기 위해 동전 던지기 함수를 실행할 수 있지 - 그리고 만약 내가 진다면, 내가 풀고 있는 다음 블록에 해당 트랜잭션을 포함하지 않는 것을 선택하지. 난 이것을 내가 결국 동전 던지기에서 이기고 다음 블록을 풀 때까지 무한대로 반복할 수 있고, 이득을 볼 수 있네.

##그럼 이더리움에서는 어떻게 난수를 안전하게 만들어낼 수 있을까?

블록체인의 전체 내용은 모든 참여자에게 공개되므로, 이건 풀기 어려운 문제이고 그 해답은 이 튜토리얼에를 벗어나네. 해결 방법들에 대해 궁금하다면 이 StackOverflow 글을 읽어보게. 하나의 방법은 이더리움 블록체인 외부의 난수 함수에 접근할 수 있도록 오라클을 사용하는 것이네.

물론, 네트워크 상의 수만 개의 이더리움 노드들이 다음 블록을 풀기 위해 경쟁하고 있으니, 내가 다음 블록을 풀 확률은 매우 낮을 것이네. 위에서 말한 부당한 방법을 쓰는 것은 많은 시간과 연산 자원을 필요로 할 것이야 - 하지만 보상이 충분히 크다면(내가 천억 원을 걸 수 있다든지?), 공격할 만한 가치가 있을 것이네.

그러니 이런 난수 생성은 이더리움 상에서 안전하지는 않지만, 실제로는 난수 함수가 즉시 큰 돈이 되지 않는 한, 자네 게임의 사용자들은 게임을 공격할 만한 충분한 자원을 들이지 않을 것이네.

이 튜토리얼에서는 시연 목적으로 간단한 게임을 만들고 있고 바로 돈이 되는 게 없기 때문에, 우린 구현하기 간단한 난수 생성기를 사용하는 것으로 타협할 것이네. 이게 완전히 안전하지는 않다는 걸 알긴 하지만 말이야.

향후 레슨에서는, 우린 oracle(이더리움 외부에서 데이터를 받아오는 안전한 방법 중 하나)을 사용해서 블록체인 밖에서 안전한 난수를 만드는 방법을 다룰 수도 있네.


#챕터 5: 좀비 싸움

좀비 전투는 다음과 같이 진행될 것이네:

- 자네가 자네 좀비 중 하나를 고르고, 상대방의 좀비를 공격 대상으로 선택하네.
- 자네가 공격하는 쪽의 좀비라면, 자네는 70%의 승리 확률을 가지네. 방어하는 쪽의 좀비는 30%의 승리 확률을 가질 것이네.
- 모든 좀비들(공격, 방어 모두)은 전투 결과에 따라 증가하는 winCount와 lossCount를 가질 것이네.
- 공격하는 쪽의 좀비가 이기면, 좀비의 레벨이 오르고 새로운 좀비가 생기네.
- 좀비가 지면, 아무것도 일어나지 않네(좀비의 lossCount가 증가하는 것 빼고 말이야).
- 좀비가 이기든 지든, 공격하는 쪽 좀비의 재사용 대기시간이 활성화될 것이네.
구현할 내용이 많으니, 다음 챕터로 진행하면서 나눠서 구현할 것이네.

		import "./zombiehelper.sol";
		
		contract ZombieBattle is ZombieHelper {
		  uint randNonce = 0;
		  // 여기에 attackVictoryProbability를 만들게
		  uint attackVictoryProbability = 70;
		
		  function randMod(uint _modulus) internal returns(uint) {
		    randNonce++;
		    return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
		  }
		
		  // 여기에 새로운 함수를 만들게
		  function attack(uint _zombieId, uint _targetId) external{
		
		  }
		}


#챕터 6: 공통 로직 구조 개선하기(Refactoring)


- 누가 우리의 attack 함수를 실행하든지 우리는 사용자가 공격에 사용하는 좀비를 실제로 소유하고 있다는 것을 확실히 하고 싶네. 만약 자네가 다른 사람의 좀비를 사용해서 공격할 수 있다면 보안에 문제가 되는 부분일 것이야!

- 함수를 호출하는 사람이 그가 사용한 _zombieId의 소유자인지 확인할 방법을 생각해낼 수 있겠는가?

- 좀 더 생각해보면서, 자네 스스로 답을 생각해낼 수 있는지 학인해보게.

-시간을 가지고... 아이디어를 위해 지난 레슨들을 참고해보게...

-해결책은 아래에 있지만, 생각이 나기 전에는 보지 말도록 하게.

##해결책
- 우린 이전 레슨들에서 이런 종류의 확인을 여러 번 해왔었네. changeName(), changeDna(), feedMultiply()에서, 우리는 다음과 같은 방식을 썼네:

		require(msg.sender == zombieToOwner[_zombieId]);

- 우리의 attack 함수에도 똑같은 내용을 적용할 필요가 있네. 동일한 내용을 여러 번 사용하고 있으니, 코드를 정리하고 반복을 피할 수 있도록 이 내용을 이것만의 modifier로 옮기도록 하세.

##직접 해보기

- zombiefeeding.sol을 다시 보도록 하겠네. 저 내용을 처음으로 썼던 곳이니 말이야. 확인 부분을 그 부분만의 modifier로 만들어 구조를 개선하겠네.
	
	1. modifier를 ownerOf라는 이름으로 만들게. 이 제어자는 _zombieId(uint)를 1개의 인수로 받을 것이네.<br><br> 
	제어자 내용에서는 msg.sender와 zombieToOwner[_zombieId]가 같은지 require로 확인하고, 함수를 실행해야 하네. 제어자의 문법이 기억이 나지 않는다면 zombiehelper.sol을 참고하면 되네.
	
	2. feedAndMultiply의 함수 정의 부분을 ownerOf 제어자를 사용하도록 바꾸게.
	
	3. 이제 modifier를 사용하게 됐으니, require(msg.sender == zombieToOwner[_zombieId]); 줄을 지워도 되네.


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
		
		  // 1. 여기에 제어자를 생성하게
		  modifier ownerOf(uint _zombieId){
		    require( msg.sendr == zombieToOwner[_zombieId] );
			_;
		  }
		
		  function setKittyContractAddress(address _address) external onlyOwner {
		    kittyContract = KittyInterface(_address);
		  }
		
		  function _triggerCooldown(Zombie storage _zombie) internal {
		    _zombie.readyTime = uint32(now + cooldownTime);
		  }
		
		  function _isReady(Zombie storage _zombie) internal view returns (bool) {
		      return (_zombie.readyTime <= now);
		  }
		
		  // 2. 함수 정의 부분에 제어자를 추가하게:
		  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal ownerOf(_zombieId) {
		    // 3. 이 줄을 지우게
		    //require(msg.sender == zombieToOwner[_zombieId]);
		    Zombie storage myZombie = zombies[_zombieId];
		    require(_isReady(myZombie));
		    _targetDna = _targetDna % dnaModulus;
		    uint newDna = (myZombie.dna + _targetDna) / 2;
		    if (keccak256(_species) == keccak256("kitty")) {
		      newDna = newDna - newDna % 100 + 99;
		    }
		    _createZombie("NoName", newDna);
		    _triggerCooldown(myZombie);
		  }
		
		  function feedOnKitty(uint _zombieId, uint _kittyId) public {
		    uint kittyDna;
		    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
		    feedAndMultiply(_zombieId, kittyDna, "kitty");
		  }
		}



#챕터 7: 구조 더 개선하기

#챕터 8: 공격으로 돌아가자!

##직접 해보기
1. 함수를 호출하는 사람이 _zombieId를 소유하고 있는지 확인하기 위해 attack 함수에 ownerOf 제어자를 추가하게.

2. 우리 함수에서 처음으로 해야 할 것은 두 좀비의 storage 포인터를 얻어서 그들과 상호작용 하기 쉽도록 하는 것이네.
<br><br>
a. Zombie storage를 myZombie라는 이름으로 선언하고, 여기에 zombies[_zombieId]를 대입하게.
<br><br>
b. Zombie storage를 enemyZombie라는 이름으로 선언하고, 여기에 zombies[_targetId]를 대입하게.

3. 우린 전투의 결과를 결정하기 위해 0과 99 사이의 난수를 사용할 것이네. 그러니 uint를 rand라는 이름으로 선언하고, 여기에 randMod 함수에 100을 인수로 사용한 값을 대입하게.

		
		import "./zombiehelper.sol";
		
		contract ZombieBattle is ZombieHelper {
		  uint randNonce = 0;
		  uint attackVictoryProbability = 70;
		
		  function randMod(uint _modulus) internal returns(uint) {
		    randNonce++;
		    return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
		  }
		
		  // 1. 여기에 제어자를 추가하게
		  function attack(uint _zombieId, uint _targetId) external ownerOf(_zombieId) {
		    // 2. 여기서 함수 정의를 시작하게
		    Zombie storage myZombie = zombies[_zombieId];
		    Zombie storage enemyZombie = zombies[_targetId];
		
		    uint rand = randMod(100);
		  }
		}
		

#챕터 9: 좀비 승리와 패배


##직접 해보기
1. Zombie 구조체가 2개의 속성을 더 가지도록 수정하게:
<br><br>
a. winCount, uint16 타입
<br><br>
b. lossCount, 역시 uint16 타입


> 참고: 기억하게, 구조체 안에서 uint들을 압축(pack)할 수 있으니, 우리가 다룰 수 있는 가장 작은 uint 타입을 사용하는 것이 좋을 것이네. uint8은 너무 작을 것이네. 2^8 = 256이기 때문이지 - 만약 우리 좀비가 하루에 한 번씩 공격한다면, 일 년 안에 데이터 크기가 넘쳐버릴 수 있을 것이네. 하지만 2^16은 65536이네 - 그러니 한 사용자가 매일 179년 동안 이기거나 지지 않는다면, 이걸로 안전할 것이네.

2. 이제 우리는 Zombie 구조체에 새로운 속성들을 가지게 되었으니, _createZombie()의 함수 정의 부분을 수정해야 할 필요가 있네.
<br><br>
각각의 새로운 좀비가 0승 0패를 가지고 생성될 수 있도록 좀비 생성의 정의 부분을 변경하게.

		pragma solidity ^0.4.19;
		
		import "./ownable.sol";
		
		contract ZombieFactory is Ownable {
		
		    event NewZombie(uint zombieId, string name, uint dna);
		
		    uint dnaDigits = 16;
		    uint dnaModulus = 10 ** dnaDigits;
		    uint cooldownTime = 1 days;
		
		    struct Zombie {
		      string name;
		      uint dna;
		      uint32 level;
		      uint32 readyTime;
		      // 1. 여기에 새로운 속성을 추가하게
		      uint16 winCount;
		      uint16 lossCount;
		    }
		
		    Zombie[] public zombies;
		
		    mapping (uint => address) public zombieToOwner;
		    mapping (address => uint) ownerZombieCount;
		
		    function _createZombie(string _name, uint _dna) internal {
		        // 2. 여기서 새로운 좀비의 생성을 수정하게:
		        uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime), 0, 0)) - 1;
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


#챕터 10: 좀비 승

이제 우리는 winCount와 lossCount를 가지고 있으니, 어떤 좀비가 싸움에서 이기냐에 따라 이들을 업데이트할 수 있네.

챕터 6에서 우린 0부터 100까지의 난수를 계산했네. 이제 그 숫자를 누가 싸움에서 이길지 결정하는 데에 사용하고, 그에 따라 상태를 업데이트하세.

##직접 해보기

1. rand가 attackVictoryProbability와 같거나 더 작은지 확인하는 if 문장을 만들게.

2. 만약 이 조건이 참이라면, 우리 좀비가 이기게 되네! 그렇다면:
	
	a. myZombie의 winCount를 증가시키게.
	
	b. myZombie의 level을 증가시키게. (레벨업이다!!!!!!!)
	
	c. enemyZombie의 lossCount를 증가시키게. (이 패배자!!!!!!! 😫 😫 😫)

	d. feedAndMultiply 함수를 실행하게. 실행을 위한 문법을 보려면 zombiefeeding.sol을 확인하게. 3번째 인수(_species)로는 "zombie"라는 문자열을 전달하게(이건 지금 이 순간에는 실제로 아무 것도 하지 않지만, 이후에 우리가 원한다면 좀비 기반의 좀비를 만들어내는 부가적인 기능을 추가할 수도 있을 것이네).

		
		import "./zombiehelper.sol";
		
		contract ZombieBattle is ZombieHelper {
		  uint randNonce = 0;
		  uint attackVictoryProbability = 70;
		
		  function randMod(uint _modulus) internal returns(uint) {
		    randNonce++;
		    return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
		  }
		
		  function attack(uint _zombieId, uint _targetId) external ownerOf(_zombieId) {
		    Zombie storage myZombie = zombies[_zombieId];
		    Zombie storage enemyZombie = zombies[_targetId];
		    uint rand = randMod(100);
		    if (rand <= attackVictoryProbability) {
		      myZombie.winCount++;
		      myZombie.level++;
		      enemyZombie.lossCount++;
		      feedAndMultiply(_zombieId, enemyZombie.dna, "zombie");
		    } 
		  }
		}

#챕터 11: 좀비 패배


이제 우리는 좀비가 이겼을 때 어떤 일이 발생할지에 대해 작성했으니, 좀비가 지면 어떤 일이 발생할지 생각해보세.

우리 게임에서, 좀비가 진다고 좀비의 레벨이 떨어지지는 않네 - 단순히 좀비의 lossCount에 그들의 패배를 기록하고, 다시 공격하기 전에 하루를 기다려야만 하도록 그들의 재사용 대기시간이 활성화될 것이네.

이러한 구조를 구현하기 위해서, 우리는 else 문장이 필요할 것이네.

	else 문장은 자바스크립트나 다른 많은 언어들에서 사용하듯이 쓸 수 있네:
	
	if (zombieCoins[msg.sender] > 100000000) {
	  // 엄청난 부자다!!!
	} else {
	  // 더 많은 좀비 코인이 필요해...
	}

##직접 해보기
1. else 문장을 추가하게. 만약 우리의 좀비가 진다면:

	a. myZombie의 lossCount를 증가시키게.
	
	b. enemyZombie의 winCount를 증가시키게.

2. else 문장의 밖에서, myZombie에 대해 _triggerCooldown 함수를 실행하게. 이러한 방법으로 해당 좀비는 하루에 한 번만 공격할 수 있네.
		
		import "./zombiehelper.sol";
		
		contract ZombieBattle is ZombieHelper {
		  uint randNonce = 0;
		  uint attackVictoryProbability = 70;
		
		  function randMod(uint _modulus) internal returns(uint) {
		    randNonce++;
		    return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
		  }
		
		  function attack(uint _zombieId, uint _targetId) external ownerOf(_zombieId) {
		    Zombie storage myZombie = zombies[_zombieId];
		    Zombie storage enemyZombie = zombies[_targetId];
		    uint rand = randMod(100);
		    if (rand <= attackVictoryProbability) {
		      myZombie.winCount++;
		      myZombie.level++;
		      enemyZombie.lossCount++;
		      feedAndMultiply(_zombieId, enemyZombie.dna, "zombie");
		    } // 여기서 시작하게
		    else {
		      myZombie.lossCount++;
		      enemyZombie.winCount++;
		    }
		    
		    _triggerCooldown(myZombie);
		  }
		}
































