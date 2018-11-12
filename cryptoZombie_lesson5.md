# [크립토좀비](https://cryptozombies.io) Lesson1를 학습하면서 정리한 내용입니다. #

# 챕터1: 이더리움 상의 토큰 #

- 이더리움에서 토큰 은 기본적으로 그저 몇몇 공통 규약을 따르는 스마트 컨트랙트이다.
- 즉 다른 모든 토큰 컨트랙트가 사용하는 표준 함수 집합을 구현한 것이다.
- 예를 들면, tarnasfer(address _to, uint256 _value) 나 balanceOf(address _owner)같은 함수가 있다.
- 내부적으로는 스마트 컨트랙트는 보통 **mapping(address => uint256) balances**와 같은 매핑을 가지고 있다. 각각의 주소에 잔액이 얼마나 있는지 기록하는 것이다.
- 즉 기본적으로 토큰은 그저 하나의 컨트랙트이며, 그 안에서 누가 얼마나 많은 토큰을 가지고 있는지 기록하고, 몇몇 함수를 가지고 사용자들이 그들의 토큰을 다른 조소로 전송할 수 있게 해주는 것이다.

## 왜 이렇게 해야 하나요?  ##

- 모든 ERC20 토큰들이 똑같은 이름의 동일한 함수 집합을 공유하기 때문에, 이 토큰들에 똑같은 방식으로 상호작용이 가능하다.
- 즉, 하나의 ERC토큰과 상호작용할 수 있는 애플리케이션 하나를 만들면, 이 앱이 다른 어떤 ERC20 토큰과도 상호작용이 가능하다.
- 이런식으로 앱에 더 많은 토큰을 추가할 수 있다. 
- 커스텀 코드를 추가하지 않고, 새로운 토큰의 컨트랙트 주소만 끼워넣으면 앱에서 사용할 수 있는 또 다른 토큰이 생긴다.
- 이러한 것의 한 예로는 거래소가 있네. 한 거래소에서 새로운 ERC20 토큰을 상장할 때, 실제로는 이 거래소에서 통신이 가능한 또 하나의 스마트 컨트랙트를 추가하는 것이네. 사용자들은 이 컨트랙트에 거래소의 지갑 주소에 토큰을 보내라고 할 수 있고, 거래소에서는 이 컨트랙트에 사용자들이 출금을 신청하면 토큰을 다시 돌려보내라고 할 수 있게 만드는 것이지.

## 다른 토큰 표준 ##

- ERC20 토큰은 화폐처럼 사용되는 토큰으로는 정말 적절하네. 하지만 우리의 좀비 게임에서 좀비를 표현할 때에는 그다지 쓸모 있지가 않지.

- 첫째로, 좀비는 화폐처럼 분할할 수가 없네 - 난 자네에게 0.237ETH를 보낼 수 있지만, 자네에게 0.237개의 좀비를 보내는 것은 말이 되지 않지.

- 둘째로, 모든 좀비가 똑같지는 않네. 자네의 레벨2 좀비 "Steve"는 내 레벨732 좀비 "H4XF13LD MORRIS 💯💯😎💯💯"와는 완전히 다르지(Steve와는 비교할 수가 없지!).

- 여기에 크립토좀비와 같은 크립토 수집품을 위해 더 적절한 토큰 표준이 있네 - 바로 ERC721 토큰이지.

- ERC721 토큰은 교체가 불가하네. 각각의 토큰이 유일하고 분할이 불가하기 때문이지. 자네는 이 토큰을 하나의 전체 단위로만 거래할 수 있고, 각각의 토큰은 유일한 ID를 가지고 있네. 그러니 이게 우리의 좀비를 거래할 수 있게 하기에는 아주 적절하지.

> ERC721과 같은 표준을 사용하면 우리의 컨트랙트에서 사용자들이 우리의 좀비를 거래/판매할 수 있도록 하는 경매나 중계 로직을 우리가 직접 구현하지 않아도 된다는 이점이 있네. 우리가 스펙에 맞추기만 하면, 누군가 ERC721 자산을 거래할 수 있도록 하는 거래소 플랫폼을 만들면 우리의 ERC721 좀비들을 그 플랫폼에서 쓸 수 있게 될 것이네. 그러니 자네만의 거래 로직을 만드느라 고생하는 것보다 토큰 표준을 사용하는 것이 명확한 이점이 있는 것이지.

# 챕터 2: ERC721 표준, 다중 상속 #

- ERC721표준:

		contract ERC721 {
		  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
		  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);
		
		  function balanceOf(address _owner) public view returns (uint256 _balance);
		  function ownerOf(uint256 _tokenId) public view returns (address _owner);
		  function transfer(address _to, uint256 _tokenId) public;
		  function approve(address _to, uint256 _tokenId) public;
		  function takeOwnership(uint256 _tokenId) public;
		}		


> 참고: ERC721 표준은 현재 초안인 상태이고, 아직 공식으로 채택된 구현 버전은 없네. 이 튜토리얼에서 우리는 OpenZeppelin 라이브러리에서 쓰이는 현재 버전을 사용할 것이지만, 공식 릴리즈 이전에 언젠가 바뀔 가능성도 있네. 그러니 하나의 구현 가능한 버전 정도로만 생각하고, ERC721 토큰의 정식 표준으로 생각하지는 말게.

## 토큰 컨트랙트 구현하기 ##

- 토큰 컨트랙트를 구현할 때, 처음 해야 할 일은 바로 인터페이스를 솔리디티 파일로 따로 복사하여 저장하고 **import "./erc721.sol";**을 써서 임포트를 하는 것이다.

- 그리고 해당 컨트랙트를 상속하는 우리의 컨트랙트를 만들고, 각각의 함수를 오버라이딩하여 정의해야 한다.

- 솔리디티에서는 컨트랙트에 다수의 컨트랙트를 상속할 수 있다:
		
		contract SatoshiNakamoto is NickSzabo, HalFinney {
		  // 오 이런, 이 세계의 비밀이 밝혀졌군!
		}

- 다중 상속을 쓸 때는 상속하고자 하는 다수의 컨트랙트를 쉼표(,)로 구분한다.

		pragma solidity ^0.4.19;
		
		import "./zombieattack.sol";
		import "./erc721.sol";
		
		contract ZombieOwnership is ZombieAttack, ERC721 {
		
		}

# 챕터 3: BalanceOf & ownerOf #

## balanceOf ##

	function balanceOf(address _owner) public view returns (uint256 _balance);

- 이 함수는 단순히 **address**를 받아, 해당 **address**가 토큰을 얼마나 가지고 있는지 반환한다.
- 이 경우, 우리의 "토큰"은 좀비들이 된다.


## ownerOf ##

	function ownerOf(uint256 _tokenId) public view returns (address _owner);

- 이 함수에서는 토큰 ID(우리의 경우에는 좀비 ID)를 받아, 이를 소유하고 있는 사람의 **address**를 반환한다.
- 이들은 구현하기 매우 수월하다. 우리가 이 정보를 저장하는 **mapping**을 DApp에 이미 가지고 있기 때문이며, 이 함수들은 단 한줄로 구현 할 수 있다.

 
> 참고: uint256은 uint와 동일하다는 것을 기억하게. 우리 코드에서 지금까지 uint를 사용해왔지만, 여기서는 우리가 스펙을 복사/붙여넣기 했으니 uint256을 쓸 것이네.

## 직접 해보기 ##
		
		pragma solidity ^0.4.19;
		
		import "./zombieattack.sol";
		import "./erc721.sol";
		
		contract ZombieOwnership is ZombieAttack, ERC721 {
		
		  function balanceOf(address _owner) public view returns (uint256 _balance) {
		    return ownerZombieCount[_owner];
		  }
		
		  function ownerOf(uint256 _tokenId) public view returns (address _owner) {
		    return zombieToOwner[_tokenId];
		  }
		
		  function transfer(address _to, uint256 _tokenId) public {
		
		  }
		
		  function approve(address _to, uint256 _tokenId) public {
		
		  }
		
		  function takeOwnership(uint256 _tokenId) public {
		
		  }
		}

# 챕터 4: 리팩토링 #

- 이전 챕터에서 우리는 **ownerOf**라는 함수를 정의. 하지만 레슨 4에서, 우리는 **zombiefeeding.sol**에서 **ownerOf**와 똑같은 이름의 **modifier**를 만들었다.
- 이 코드를 컴파일하려 한다면, 컴파일러가 똑같은 이름의 제어자와 함수를 가질 수 없다며 에러를 만들어 낼 것이다.
- 우리는 ERC721 토큰 표준을 사용하고 있다. 이 말인즉 다른 컨트랙트들이 우리의 컨트랙트가 정확한 이름으로 정의된 함수들을 가지고 있을 것이라 예상한다는 것이다. 
- 만약 우리 컨트랙트는 ERC721을 따른다는 것을 다른 컨트랙트가 안다면, 이 다른 컨트랙트는 우리의 내부 구현 로직을 모르더라도 우리와 통신 할 수 있다.
- 따라서 우리는 레슨 4에서 만든 우리의 코드에서 modifier의 이름을 다른 것으로 바꾸도록 리팩토링 해야 한다.

# 챕터 5: ERC721: 전송 로직 #

- 챕터 4에서 충돌을 해결했다.
- ERC721 스펙에서는 토큰을 전송할 때 2개의 다른 방식이 있다.
		
		function transfer(address _to, uint256 _tokenId) public;
		function approve(address _to, uint256 _tokenId) public;
		function takeOwnership(uint256 _tokenId) public;

	1. 첫 번째 방법은 토큰의 소유자가 전송 상대의 address, 전송하고자 하는 _tokenId와 함께 transfer 함수를 호출하는 것이네.
	
	2. 두 번째 방법은 토큰의 소유자가 먼저 위에서 본 정보들을 가지고 approve를 호출하는 것이네. 그리고서 컨트랙트에 누가 해당 토큰을 가질 수 있도록 허가를 받았는지 저장하지. 보통 mapping (uint256 => address)를 써서 말이지. 이후 누군가 takeOwnership을 호출하면, 해당 컨트랙트는 이 msg.sender가 소유자로부터 토큰을 받을 수 있게 허가를 받았는지 확인하네. 그리고 허가를 받았다면 해당 토큰을 그에게 전송하지.

- 자네가 눈치를 챘을지 모르겠지만, transfer와 takeOwnership 모두 동일한 전송 로직을 가지고 있네. 순서만 반대인 것이지(전자는 토큰을 보내는 사람이 함수를 호출하네; 후자는 토큰을 받는 사람이 호출하는 것이지).

- 그러니 이 로직만의 프라이빗 함수, _transfer를 만들어 추상화하는 것이 좋을 것이네. 두 함수에서 모두 쓸 수 있도록 말이야. 이렇게 하면 똑같은 코드를 두 번씩 쓰지 않아도 되지.
	
	  function _transfer(address _from, address _to, uint256 _tokenId) private {
	      ownerZombieCount[_to]++;
	      ownerZombieCount[_from]--;
	      zombieToOwner[_tokenId] = _to;
	      Transfer(_from, _to, _tokenId);
	  }

# 챕터 6: ERC721: 전송(이어서) #

- 퍼블릭 transfer함수 구현

- 우리는 토큰/좀비의 소유자만 해당 토큰/좀비를 전송할 수 있도록 하고 싶네. 자네, 어떻게 그 소유자만 이 함수에 접근할 수 있도록 제한하는지 기억하고 있나?

- 그래, 바로 그거지. 우리는 이미 이렇게 만들어주는 제어자를 가지고 있네. 그러니 이 함수에 onlyOwnerOf 제어자를 추가하게.

- 이제 함수의 내용은 진짜로 딱 한 줄이면 되네... 그저 _transfer를 호출하기만 하면 되지. address _from 인수에 msg.sender를 전달하는 것을 잊지 말게

# 챕터 7: ERC721: Approve #

- approve / takeOwnership 을 사용하는 전송은 2단계로 나뉘다.

	1. 소유자인 자네가 새로운 소유자의 **address**와 그에게 보내고 싶은 **_tokenId**를 사용하여 **approve**를 호출하네.

	2. 새로운 소유자가 **_tokenId**를 사용하여 **takeOwnership** 함수를 호출하면, 컨트랙트는 그가 승인된 자인지 확인하고 그에게 토큰을 전송하네.

- 이처럼 2번의 함수 호출이 발생하기 때문에, 우리는 함수 호출 사이에 누가 무엇에 대해 승인이 되었는지 저장할 데이터 구조가 필요하다.

# 챕터 8: ERC721: takeOwnership #

- 좋아, 이제 마지막 함수와 함께 우리의 ERC721 구현을 끝내보세!(걱정하지 말게, 이것 다음에도 레슨 5에서 다뤄야 할 것들이 더 남아 있네 😉)

- 마지막 함수인 takeOwnership에서는 msg.sender가 이 토큰/좀비를 가질 수 있도록 승인되었는지 확인하고, 승인이 되었다면 _transfer를 호출해야 하네.

## 직접 해보기 ##

1. 먼저, require 문장을 써서 zombieApprovals의 _tokenId 요소가 msg.sender와 같은지 확인해야 하네.

	이런 방식으로 만약 msg.sender가 이 토큰을 받도록 승인되지 않았다면, 에러를 만들어낼 것이네.

2. _transfer를 호출하기 위해, 우리는 그 토큰을 소유한 사람의 주소를 알 필요가 있네(함수에서 _from을 인수로 요구하기 떄문이지). 다행히 우리의 ownerOf 함수를 써서 이를 찾아낼 수 있네.

	그러니 address 변수를 owner라는 이름으로 선언하고, 여기에 ownerOf(_tokenId)를 대입하게.

3. 마지막으로, _transfer를 필요한 모든 정보와 함께 호출하게(여기서는 msg.sender를 _to에 사용하면 되네. 이 함수를 호출하는 사람이 토큰을 받을 사람이기 떄문이지).



> 참고: 2번째와 3번째 단계를 한 줄의 코드로 만들 수 있지만, 나누는 것이 조금 더 읽기 좋게 만드네. 개인적인 선호인 것이지.

# 챕터 9: 오버플로우 막기 #

축하하네, ERC721 구현을 완료했네!

너무 빡빡하진 않았을 것이네. 그렇지? 이런 이더리움에 관한 많은 것들에 대해 사람들이 말하는 걸 듣다 보면 굉장히 복잡하게 느껴지네. 그러니 이를 이해하는 가장 좋은 방법은 실제로 직접 구현해보는 것이네.

이건 그저 가장 간단한 구현 버전이라는 것을 명심하게. 구현에 더 추가할 수 있는 추가적인 기능들이 많이 있네. 예를 들면 사용자들이 의도치 않게 그들의 좀비를 0번 주소로 보내는 것(토큰을 "태운다(burning)"고 하는 것이지 - 기본적으로 누구도 개인키를 가지고 있지 않은 주소로 보내서 복구할 수 없게 하는 것이네)을 막기 위해 추가적인 확인을 할 수 있겠지. 혹은 DApp 자체에 기본적은 경매 로직을 넣는 것도 가능할 것이네(이를 구현하는 방법을 생각해낼 수 있겠나?).

하지만 우리는 이 레슨이 다루기 쉽게 유지하고 싶기 때문에, 가장 기본적인 구현만 진행하였네. 더 깊이 있는 구현의 예시를 보고 싶다면, 이 튜토리얼이 끝난 후 OpenZeppelin ERC721 컨트랙트를 참고해보도록 하게.

## 컨트랙트 보안 강화: 오버플로우와 언더플로우 ##

이제 스마트 컨트랙트를 작성할 때 자네가 인지하고 있어야 할 하나의 주요한 보안 기능을 살펴볼 것이네: 오버플로우와 언더플로우를 막는 것이지.

오버플로우가 무엇인가?

우리가 8비트 데이터를 저장할 수 있는 uint8 하나를 가지고 있다고 해보지. 이 말인즉 우리가 저장할 수 있는 가장 큰 수는 이진수로 11111111(또는 십진수로 2^8 - 1 = 255)가 되겠지.

다음 코드를 보게. 마지막에 number의 값은 무엇이 되겠나?

	uint8 number = 255;
	number++;

이 예시에서, 우리는 이 변수에 오버플로우를 만들었네 - 즉 number가 직관과는 다르게 0이 되네. 우리가 증가를 시켰음에도 말이야. 자네가 이진수 11111111에 1을 더하면, 이 수는 00000000으로 돌아가네. 시계가 23:59에서 00:00으로 가듯이 말이야.

언더플로우는 이와 유사하게 자네가 0 값을 가진 uint8에서 1을 빼면, 255와 같아지는 것을 말하네(uint에 부호가 없어, 음수가 될 수 없기 때문이지).

우리가 여기서 uint8을 쓰지는 않고, 1씩 증가시킨다고 uint256에 오버플로우가 발생하지는 않을 것 같지만(2^256은 정말 큰 숫자이네), 미래에 우리의 DApp에 예상치 못한 문제가 발생하지 않도록 여전히 우리의 컨트랙트에 보호 장치를 두는 것이 좋을 것이네.

## SafeMath 사용하기 ##

이를 막기 위해, OpenZeppelin에서 기본적으로 이런 문제를 막아주는 SafeMath라고 하는 라이브러리를 만들었네.

이것을 살펴보기 전에 먼저... 라이브러리가 무엇인가?

라이브러리(Library)는 솔리디티에서 특별한 종류의 컨트랙트이네. 이게 유용하게 사용되는 경우 중 하나는 기본(native) 데이터 타입에 함수를 붙일 때이네.

예를 들어, SafeMath 라이브러리를 쓸 때는 using SafeMath for uint256이라는 구문을 사용할 것이네. SafeMath 라이브러리는 4개의 함수를 가지고 있네 - add, sub, mul, 그리고 div가 있네. 그리고 이제 우리는 uint256에서 다음과 같이 이 함수들에 접근할 수 있네.

		using SafeMath for uint256;
		
		uint256 a = 5;
		uint256 b = a.add(3); // 5 + 3 = 8
		uint256 c = a.mul(2); // 5 * 2 = 10

이 함수들이 어떤 것들인지는 다음 챕터에서 알아볼 것이네. 지금은 우리 컨트랙트에 SafeMath 라이브러리를 추가하도록 하지.


# 챕터 10: Safemath 파트 2 #

- SafeMath 내부 코드 :
	
		library SafeMath {
		
		  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
		    if (a == 0) {
		      return 0;
		    }
		    uint256 c = a * b;
		    assert(c / a == b);
		    return c;
		  }
		
		  function div(uint256 a, uint256 b) internal pure returns (uint256) {
		    // assert(b > 0); // Solidity automatically throws when dividing by 0
		    uint256 c = a / b;
		    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
		    return c;
		  }
		
		  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
		    assert(b <= a);
		    return a - b;
		  }
		
		  function add(uint256 a, uint256 b) internal pure returns (uint256) {
		    uint256 c = a + b;
		    assert(c >= a);
		    return c;
		  }
		}

- **library**키워드. 라이브러리는 contract 와 비승하지만 조금 다르다.
- 라이브러리는 using 키워드를 사용하게 해준다.
- 이를 통해 라이브러리의 메소드들을 다른 데이터 타입에 적용할 수 있다.

		using SafeMath for uint;
		// 우리는 이제 이 메소드들을 아무 uint에서나 쓸 수 있네.
		uint test = 2;
		test = test.mul(3); // test는 이제 6이 되네
		test = test.add(5); // test는 이제 11이 되네

- mul과 add 함수는 각각 2개의 인수를 필요로 한다는 것에 주목하게. 하지만 우리가 using SafeMath for uint를 선언할 때, 우리가 함수를 적용하는 uint(test)는 첫 번째 인수로 자동으로 전달되네.

- SafeMath가 어떤 것을 하는지 보기 위해 add 함수의 내용을 한번 살펴보도록 하지:

		function add(uint256 a, uint256 b) internal pure returns (uint256)	 {		
		 	uint256 c = a + b;
		 	assert(c >= a);
		 	return c;
		}


- 기본적으로 add는 그저 2개의 uint를 +처럼 더한다. 하지만 그 안에 assert 구문을 써서 그 합이 a보다 크도록 보장한다. 이것이 오버플로우를 막아준다.

- assert는 조건을 만족하지 않으면 에러를 발생시킨다는 점에서 require와 비슷하다. assert와 require의 차이점은, require는 함수 실행이 실패하면 남은 가스를 사용자에게 되돌려 주지만, assert는 그렇지 않다. 즉 대부분 자네는 자네의 코드에 require를 쓰고 싶을 것이네; assert는 일반적으로 코드가 심각하게 잘못 실행될 때 사용하네(uint 오버플로우처럼 말이야).

- 간단히 말해, SafeMath의 add, sub, mul, 그리고 div는 4개의 기본 수학 연산을 하는 함수이지만, 오버플로우나 언더플로우가 발생하면 에러를 발생시키는 것이네.

## 우리의 코드에 SafeMath 사용하기 ##

- 오버플로우나 언더플로우를 막기 위해, 우리의 코드에서 +, -, * 또는 /을 쓰는 곳을 찾아 add, sub, mul, div로 교체할 것이네.

- 예를 들어, 아래처럼 하는 대신:

		myUint++;

- 이렇게 할 것이네:

		myUint = myUint.add(1);


# 챕터 11: SafeMath 파트 3 #

- 일반적으로 기본 수학 연산보다 SafeMath를 쓰는 것이 좋다. 향후의 솔리디티 버전에서는 이게 기본으로 포함될 수도 있겠지만, 지금은 우리 코드에 추가적인 보안 조치를 해야 한다.
- 하지만 여기에 작은 문제가 있네 - winCount와 lossCount는 uint16이고, level은 uint32인 것이지. 즉 우리가 이런 인수들을 써서 SafeMath의 add 메소드를 사용하면, 이 타입들을 uint256으로 바꿀 것이기 때문에 실제로 오버플로우를 막아주지 못하네

		function add(uint256 a, uint256 b) internal pure returns (uint256) {
		  uint256 c = a + b;
		  assert(c >= a);
		  return c;
		}
		
		// 만약 `uint8`에 `.add`를 호출한다면, 타입이 `uint256`로 변환되네.
		// 그러니 2^8에서 오버플로우가 발생하지 않을 것이네. 256은 `uint256`에서 유효한 숫자이기 때문이지.

- 이는 즉 uint16과 uint32에서 오버플로우/언더플로우를 막기 위해 2개의 라이브러리를 더 만들어야 한다는 것을 의미하네. 우리는 이들을 SafeMath16과 SafeMath32라고 부를 것이네.

- 코드는 모든 uint256들이 uint32 또는 uint16으로 바뀐다는 것 말고는 SafeMath와 정확히 같을 것이네.


> Year 2038 문제??
// 참고: 우리는 Year 2038 문제를 막지 않기로 하겠네... 그러니 readyTime에서 오버플로우를 걱정할 필요는 없네.
// 우리 앱은 2038년에는 좀 꼬이겠지 ;)

# 챕터 12: SafeMath 파트 4 #

- 좋아, 이제 우리의 DApp에서 사용한 모든 uint 타입에 대해 SafeMath를 적용할 수 있네!

- ZombieAttack에서 이 모든 잠재적 문젯거리들을 고쳐보도록 하지(ZombieHelper에서도 고쳐져야 할 zombies[_zombieId].level++; 이런 부분이 있었지만, 우리가 이걸 하기 위해 추가적으로 챕터를 쓰지 않도록 내가 자네를 위해 처리해 놓았네 😉).

# 챕터 13: 주석(Comment) # 


## 주석을 위한 문법 ##

- // : 한줄 주석
- /* */ : 여러줄 주석

- 솔리디티 커뮤니티에서 표준으로 쓰이는 형식은 **natspec**이라 불리네. 아래와 같이 생겼지:

		/// @title 기본적인 산수를 위한 컨트랙트
		/// @author H4XF13LD MORRIS 💯💯😎💯💯
		/// @notice 지금은 곱하기 함수만 추가한다.
		contract Math {
		  /// @notice 2개의 숫자를 곱한다.
		  /// @param x 첫 번쨰 uint.
		  /// @param y 두 번째 uint.
		  /// @return z (x * y) 곱의 값
		  /// @dev 이 함수는 현재 오버플로우를 확인하지 않는다.
		  function multiply(uint x, uint y) returns (uint z) {
		    // 이것은 일반적인 주석으로, natspec에 포함되지 않는다.
		    z = x * y;
		  }
		}

- @title과 @author는 따로 설명이 필요 없을 것 같군.
 
- @notice는 사용자에게 컨트랙트/함수가 무엇을 하는지 설명하네. @dev는 개발자에게 추가적인 상세 정보를 설명하기 위해 사용하지.
 
- @param과 @return은 함수에서 어떤 매개 변수와 반환값을 가지는지 설명하네.
 
- 자네가 모든 함수에 대해 꼭 이 모든 태그들을 항상 써야만 하는 것은 아니라는 점을 명심하게 모든 태그는 필수가 아니네. 하지만 적어도, 각각의 함수가 어떤 것을 하는지 설명하도록 @dev는 남기도록 하게.

# 챕터 14: 마무리하기 #

## 요약하자면: ##
- 이번 레슨에서 우리는 다음을 배웠네:

		1. 토큰, ERC721 표준, 그리고 거래할 수 있는 자산/좀비
		2. 라이브러리와 이를 사용하는 방법
		3. SafeMath 라이브러리를 사용하여 오버플로우와 언더플로우를 막는 방법
		4. 코드에 주석을 추가하는 방법과 natspec 표준