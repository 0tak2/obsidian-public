
## 개요

- 민감한 데이터를 보관할 수 있는 로컬 데이터베이스의 일종
- 비밀번호에 국한하지 않고 다양한 데이터를 담을 수 있으나, 쓸 때 암호화(AES-256)되고 읽을 때 복호화되므로 읽고 쓰는 속도가 느려 보안상 필요한 데이터만 담아야한다.
- 내부에서 Core Foundation 타입(CF- 접두사)을 사용해 명시적으로 형변환을 해줘야 한다. 성능 때문인지 레거시 때문인지는 잘 모르겠다. 아마도 C로 작성된 암호화 모듈을 직접 호출하고 있는게 아닐까?

## 쿼리 딕셔너리

- Keychain에 값을 저장하기 전에 쿼리를 적절하게 생성해야 한다.
- 쿼리는 값에 대한 메타데이터이자, 나중에 저장한 값을 검색할 때 사용되는 단서가 된다.
- 필요에 따라 어트리뷰트를 고를 수 있다.
	- 아래의 경우 특정 원격 서버에 대한 암호를 저장하기 위해 kSecClass를 [kSecClassInternetPassword](https://developer.apple.com/documentation/security/ksecclassinternetpassword)로 지정했으나, 
	- 원격지 서버 등 어트리뷰트가 필요하지 않은 경우 [`kSecClassGenericPassword`](https://developer.apple.com/documentation/security/ksecclassgenericpassword)로 지정할 수도 있다.

## 기본 CRUD

```swift
let account = credentials.username
let password = credentials.password.data(using: String.Encoding.utf8)!
var query: [String: Any] = [kSecClass as String: kSecClassInternetPassword,
                            kSecAttrAccount as String: account,
                            kSecAttrServer as String: server,
                            kSecValueData as String: password]
```

- 키체인 아이템을 읽고 쓰기 위한 기본적인 CRUD API는 다음과 같다.
	- [SecItemAdd(\_:\_:)](https://developer.apple.com/documentation/security/1401659-secitemadd): Use this function to add one or more items to a keychain.
	- [SecItemCopyMatching(\_:\_:)](https://developer.apple.com/documentation/security/1398306-secitemcopymatching): This function returns one or more keychain items that match a search query. Additionally, it can copy attributes of specific keychain items.
	- [SecItemUpdate(\_:\_:)](https://developer.apple.com/documentation/security/1393617-secitemupdate): This function allows you to modify items that match a search query.
	- [SecItemDelete(\_:)](https://developer.apple.com/documentation/security/1395547-secitemdelete): This function removes items that match a search query.

## 쓰기/수정하기

-  Kodeco에서 제공하는 예제 코드임을 발힌다. ([출처](https://www.kodeco.com/9240-keychain-services-api-tutorial-for-passwords-in-swift/page/2?page=2#toc-anchor-004))

	```swift
	// 1
	guard let encodedPassword = value.data(using: .utf8) else {
	  throw SecureStoreError.string2DataConversionError
	}
	
	// 2
	var query: [String: Any] = [
	  kSecClass as String: kSecClassGenericPassword,
	  kSecAttrService as String: "MyService"
	]
	query[String(kSecAttrAccount)] = userAccount
	
	// 3
	var status = SecItemCopyMatching(query as CFDictionary, nil)
	switch status {
	// 4
	case errSecSuccess:
	  var attributesToUpdate: [String: Any] = [:]
	  attributesToUpdate[String(kSecValueData)] = encodedPassword
	  
	  status = SecItemUpdate(query as CFDictionary,
	                         attributesToUpdate as CFDictionary)
	  if status != errSecSuccess {
	    throw error(from: status)
	  }
	// 5
	case errSecItemNotFound:
	  query[String(kSecValueData)] = encodedPassword
	  
	  status = SecItemAdd(query as CFDictionary, nil)
	  if status != errSecSuccess {
	    throw error(from: status)
	  }
	default:
	  throw error(from: status)
	}
	```

1. String을 Data로 변환
2. 먼저 조회를 위한 쿼리 딕셔너리를 작성
3. 쿼리 딕셔너리를 이용해 조회
4. 성공한 경우 이미 데이터가 있다는 의미이므로 업데이트 수행
5. 데이터가 없어 실패한 경우 값 추가

## 읽기

-  역시 Kodeco에서 제공하는 예제 코드이다. ([출처](https://www.kodeco.com/9240-keychain-services-api-tutorial-for-passwords-in-swift/page/2?page=2#toc-anchor-004))

```swift
// 1
var query: [String: Any] = [
  kSecClass as String: kSecClassGenericPassword,
  kSecAttrService as String: "MyService"
]
query[String(kSecMatchLimit)] = kSecMatchLimitOne
query[String(kSecReturnAttributes)] = kCFBooleanTrue
query[String(kSecReturnData)] = kCFBooleanTrue
query[String(kSecAttrAccount)] = userAccount

// 2
var queryResult: AnyObject?
let status = withUnsafeMutablePointer(to: &queryResult) {
  SecItemCopyMatching(query as CFDictionary, $0)
}

switch status {
// 3
case errSecSuccess:
  guard 
    let queriedItem = queryResult as? [String: Any],
    let passwordData = queriedItem[String(kSecValueData)] as? Data,
    let password = String(data: passwordData, encoding: .utf8)
    else {
      throw SecureStoreError.data2StringConversionError
  }
  return password
// 4
case errSecItemNotFound:
  return nil
default:
  throw error(from: status)
}
```

1. 원하는 조건을 포함해 쿼리 딕셔너리를 만든다.
	- kSecMatchLimit: 몇 개 조회할지
	- kSecReturnAttributes: 어트리뷰트 반환 여부
	- kSecReturnData: 값 반환 여부
	- kSecAttrAccount: 비밀번호를 검색하려는 계정 지정
2. 해당 쿼리로 데이터를 조회한다. 이때 조회 결과를 담을 변수 queryResult의 포인터를 넘긴다. (status가 아니라 조회된 값이 담길 포인터)
3. 성공한 경우 queryResult를 딕셔너리로 변환해 값을 추출한다. kSecValueData가 값의 키이다. Data 타입이므로 String으로 디코딩해야 한다.
4. 실패한 경우 nil 반환

## 삭제

-  역시 Kodeco에서 제공하는 예제 코드이다. ([출처](https://www.kodeco.com/9240-keychain-services-api-tutorial-for-passwords-in-swift/page/2?page=2#toc-anchor-004))

### 단건 삭제

```swift
// 1
var query: [String: Any] = [
  kSecClass as String: kSecClassGenericPassword,
  kSecAttrService as String: "MyService"
]
query[String(kSecAttrAccount)] = userAccount

// 2
let status = SecItemDelete(query as CFDictionary)
guard status == errSecSuccess || status == errSecItemNotFound else {
  throw error(from: status)
}
```

1. 원하는 조건을 포함해 쿼리 딕셔너리를 만든다.
	- 삭제할 계정을 kSecAttrAccount 키로 추가했다.
	- 데이터를 조회하는 상황이 아니므로 kSecReturnAttributes, kSecReturnData 등은 설정할 필요가 없다.
2. 삭제 후 결과를 확인한다. 성공이거나 조회 결과가 없는 경우 에러 없이 종료한다.

### 모두 삭제

```swift
let query: [String: Any] = [
  kSecClass as String: kSecClassGenericPassword,
  kSecAttrService as String: "MyService"
]
  
let status = SecItemDelete(query as CFDictionary)
guard status == errSecSuccess || status == errSecItemNotFound else {
  throw error(from: status)
}
```

- 쿼리에 따로 조건을 추가하지 않고 SecItemDelete을 호출했다.

\* Kodeco 튜토리얼에서는 query를 위와 같이 인라인으로 바로 정의하지 않고, 
  query 프로퍼티를 제공하는 프로토콜을 만든 후 이를 준수하는 객체를 주입받아 사용한다. 반복 코드가 줄고 확장성이 높아지는 설계이다.

https://developer.apple.com/documentation/security/keychain-services
https://developer.apple.com/documentation/security/using-the-keychain-to-manage-user-secrets
https://support.apple.com/lt-lt/guide/security/secb0694df1a/web
https://medium.com/@omar.saibaa/local-storage-in-ios-keychain-668240e2670d
https://www.kodeco.com/9240-keychain-services-api-tutorial-for-passwords-in-swift