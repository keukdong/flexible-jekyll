![str](/assets/img/strangec120.png)
# chapter 2 (2018.10.04)
어제 보긴 다 봤는데 포스팅은 오늘 작성.
## 5. C++ 컴파일러가 자동으로 만들어 호출하는 함수들
  복사 생성자, 복사 대입 연산자, 소멸자, 생성자들은 선언되어 있지안항도 컴파일러가 자동으로 선언한다.

~~~c
private:
  std::string nameVal;
  T val;
/* blahblah */
NameObject(const string& name, const T& value);
/* blahblah */
NameObject<int> no1("smallest Prime", 2);
NameObject<int> no2(no1);
~~~
string은 자체 복사 생성자 가지고 있어서 OK
~~~c
private:
  std::string& nameVal;
/* blahblah */  
NameObject<int> no1("smallest Prime", 2);
no1 = no2;
~~~
C++의 참조자는 자신이 참조하는 것 아닌 것은 참조못한다. C++는 참조자에 대한 대입연사를 거부한다. User가 직접 연산자를 정의해줘야 한다.
복사대입연산자가 private 클래스로 선언된 class의 파생 클래스는 암시적 복사대입연산자를 가지지 못한다.

## 6. 컴파일러가 만드는 함수가 필요없으면 사용확실하게 하지 말자
1. 복사 생성자 및 복사 대입 연산자를 private 멤버로 만들어 바깥에서 호출못하게한다. -> friend 클래스에서는 여전히 호출가능
2. private 멤버로 선언 뒤 정의를 하지않는다
~~~c
class A{
private:
  A(const A&);
  A& operator=(const A&);
}
~~~
private로 만든뒤 파생클래스로 만들 수도 있다.
~~~c
class A{
private:
  A(const A&);
  A& operator=(const A&);
};
class B: private A{
};
~~~
## 7. 다형성을 가진 기본 클래스에서는 소멸자는 반드시 가상으로 만든다
~~~c
class Timekeeper{
  public:
  Timekeeper();
  ~Timekeeper();
};
class AtomicClock:public Timekeeper{
};
/* blahblah */
AtomicClock *aptr;
Timekeeper *ptr = aptr;
delete ptr;
~~~
이 상황에서 ptr은 기본클래스로 기본클래스의 소멸자를 호출하게 되는데 이 소멸자가 non virtual이면 자식클래스의 소멸자는 호출이 안된다.
그러므로 자식클래스의 멤버에서 메모리 누수 발생가능성이 있다.

C++에서는 가상함수가 있으면 클래스에는 vptr이라는 가상함수테이블 포인터 자료구조가 하나 더 들어간다.
포인터 크기만큼 메모리가 추가되겠져.
## 8. 예외가 소멸자를 떠나지 못하도록 한다.
~~~c
class DBConnection{
public:
  static DBConnection create();
  void close();
}
class DBC{
public:
  ~DBC(){
    db.close();
  }
private:
  DBConnection db;
}
~~~
이 경우 close()에서 예외가 발생했는데 ~DBC()에서 난 것처럼 보일 수 있다. 이 경우 선택지는
1. db.close()에 try catch 넣고 abort 시킨다
2. try catch넣고 아무것도 하지않는다 = 삼킨다

예외가 어디서 발생했는지 정확히 알고 뒤의 과정에 영향을 끼치는가에 따라서 적절히 처리해야한다.
## 9. 객체 소멸중에 절대로 가상함수를 호출하지 않는다
~~~c
class A{
public:
  A();
  virtual void log() const =0;
};
A::A(){
  log();
}
/* ... */
class B: public A{
public:
  virtual void log() const;
};
B b();
~~~
b객체를 생성할때 기본 클래스의 생성자가 먼저 호출된다. 하지만 A 생성자를 호출하는 과정에 가상함수로 선언된 log()를 호출하면 이 log()는 A의 로그이다. 기본클래스의 생성자가 호출되는 동안 가상함수는 파생클래스로 내려가지 않는다. C++은 초기화 되지도 않은 영역을 건드리지 못하게 막아놓았다.

해결 방법은 log()를 비가상함수로 만들고 파생클래스의 생성자가 log의 정보를 기본 생성자로 넘기게 한다. 필요한 정보를 파생클래스쪽에서 기본 클래스 생성자로 올려주도록한다.
## 10. 대입 연산자는 * *this* 의 참조자를 반환한다.
~~~c
A& operator=(const A& other){
  /* blahblah */
  return *this;
}
~~~
## 11. operator=에서는 자기 대입에 대한 처리가 빠지지 않아야한다.
~~~
private:
  B *pb;
public:
  B& operator=(const B& other){
  delete pb;
  pb = other.pb;
  return *this;
}
~~~
만약 other가 해당 객체라면 당연히 안된다. 해결방법
~~~c
//1. 일치성 검사
B& operator=(const B& other){
  if(other == *this) retrun *this
  /* blahblah */
}
//2. copy and swap
B& operator=(const B& other){
   B* temp(other);
   swap(temp); // 안전하게 정의 되어있다고 가정한다.
   return *this;
}
~~~
## 12. 객체의 모든 부분을 빠짐없이 복사한다.
~~~c
class Child: public Parent{
  Child(const Child& other): everychildmem(other.everychildmem){};
}
~~~
이때 상속된 Parent의 멤버가 복사가 안되고있다. Child복사 생성자에 기본클래스를 넘길 인자도 없어서 Parent의 기본 생성자가 호출되어 초기화한다. 여기서 Parent 기본 생성자가 선언 안되어있으면 오류가 먼저 발생한다.
~~~c
Child(const Child& other): Parent(other), everychildmem(other.everychildmem){};
~~~
기본 클래스의 복사생성자를 호출한다. 

복사 생성자와 복사 대입연산자는 비슷하게 생겼지만 코드 좀 꼼수 써서 줄여본다고 한쪽에서 다른 한쪽을 호출해보고 싶지만 절대 안된다.
생성자: 새로 만들어진 객체를 초기화
대입: 이미 초기화가 끝난 객체에 대입
서로의 역할이 다르기 때문이다. 굳이 예쁘게 만들고 싶으면 공통된 코드부분을 private 함수로 만들어 쓰는 것이 맞다.



