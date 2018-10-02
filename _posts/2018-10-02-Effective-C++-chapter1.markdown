![str](/assets/img/strangec120.png)
# chapter 1 (2018.10.02)
## 
## define 대신 const, enum, inline
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

## 낌새만 보이면 cosnst를 들이대라
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

상수 비상수 차이만있고 비슷하게 생긴 함수를 두번이나 중복해서 쓸경우 만약 코드가 길다면?
//짤방
~~~c
const char& operator[](int idx) const{
  ... 블라블라 블라
  return text[idx];
}
char& operator[](int idx){
  return const_cast<char&>(static_cast<const Textblock&> (*this)[idx]);
}
~~~
비상수에 const casting을 하여 상수를 만들고 그 상수 함수를 호출한다.
//짤방 뭔소리래


