![str](/assets/img/strangec120.png)
# chapter 1 (2018.10.02)
요약
## 2. define 대신 const, enum, inline
클래스 안 정적 상수
~~~c
//선언부
class A{
  private:
    static const int num = 5; //선언
    ...
}
//구현부
const int A::num ; //정의 
~~~

나열자 둔갑술
~~~c
private:
  enum{ num=5; };
  int score[num]; //가능
~~~

~~~c
const int* p //포인터가 가리키는 데이터가 상수가 된다. p가 가리키는 내용을 바꿀 수 없다.
int* const p //포인터의 주소가 상수가 된다. 
const int* const p // "완전체"
~~~

## 3. 낌새만 보이면 const를 들이대라
~~~c
const vecotr<int>::iterator it= vec.begin();
*it = 10; // ok : iter가 가리키는 대상을 변경
++it; // no

vector<int>::iterator const cit = vec.begin();
*cit = 10; // no
cit++; // ok
~~~
상수멤버함수를 쓰는 이유
1. 객체의 멤버 변수값을 변경 가능한 함수와 변경 불가능한 함수를 나누어 클래스의 인터페이스를 이해하기 쉽게한다.
2. 상수 객체를 사용하기 위해서

~~~c
const char& operator[](int position) const{
  return text[position];
}
const TextBlock ctb("World");
cout<<ctb[0]; //상수 멤버함수 호출
~~~
~~~c
char& operator[](int position){
  return text[position];
}
TextBlock tb("World");
cout<<tb[0]; //비상수 멤버함수 호출
~~~
만약 char operator[](int position){ ~ } tb[0]= 'x'라도 값이 수정되지 않는다.
~~~c
char& operator[](int position) const { ~ }
...
ctb[0] = 'x'
~~~
혼종이다. 제대로 수행되지만 이래서는 const 객체가 호출하는 의미가 없다.
그런데 굳이 호출해서 수정해야겠다면 mutable 을 쓰면 된다
~~~c
mutuable int textlength;
int length() const{
  textlength = strlen(pText);
  return textlength;
}
~~~

상수 비상수 차이만있고 비슷하게 생긴 함수를 두번이나 중복해서 쓸경우 만약 코드가 길다면?
~~~c
const char& operator[](int idx) const{
  ...// 블라블라 블라
  return text[idx];
}
char& operator[](int idx){
  return const_cast<char&>(static_cast<const Textblock&> (*this)[idx]);
  //const_cast : op[] 반환타입에 const 떼어내는 캐스팅 적용
  //(*this)에 const 붙인다
  //op[]의 상수버전을 호출한다.
}
~~~
비상수 함수는 상수함수를 호출하여 상수 반환 받고 그걸 const casting을해서 const 캐스팅을 떼어낸다.
반대의 경우: 상수함수는 안에서 비상수 멤버를 호출하여 수정하겠다는 컴파일러와의 약속을 어김 어기면? error 발생.

## 4. 객체 사용 전에 반드시 객체 초기화
객체 초기화는 생성자 안에 """대입"""하지말고 멤버 초기화 리스트를 사용하여 초기화하자.

불변의 데이터 초기화 순서
1. 기본 클래스는 파생 클래스보다 먼저 초기화 된다.
2. 클래서 data 멤버는 선언 순서대로 초기화 된다.

non-local static 객체의 초기화 순서는 개별 번역 단위에서 결정된다. C++에서는 서로 다른 번역 단위에 정의된 비지역 static객체끼리는 초기화 순서는 정해져있지 않다.
~~~c
class A(){
  void func();
  ...
};
extern A a;
~~~
~~~c
class B(){
  ...
  A.func(); // B.cpp에서 A를 호출할때
};
~~~
A.cpp보다 B.cpp가 먼저 컴파일되서 초기화되는 경우: 와~안된다~ ->해결: 함수 안에서 정적 객체 선언후 참조자 return
why? 지역정적객체는 정의에 최초로 닿았을 때 초기화 된다.
~~~c
class A(){
  ...
  void func();
};
A& funA(){
  static A as;
  return as;
};
~~~
~~~c
...
funA().func();
B& dir(){
  static B bs;
  return bs;
};
~~~
이 경우 다중 스레드에서 문제를 일으킬 수 있다. 이 경우 시동단계에서 참조자 반환 함수를 전부 직접 호출해야 race condition문제가 없어진다.

