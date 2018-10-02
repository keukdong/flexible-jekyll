![str](/assets/img/strangec120.png)
# chapter 1
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


