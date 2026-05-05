# 5-28 내부에서 사용하는 객체에 대한 '핸들'을 반환하는 코드는 되도록 피하자.

Rectangle Class를 사용해서 만드는 어떤 응용프로그램이 있다고 가정하자. <br>
사각형은 좌측 상단 및 우측 하단의 꼭짓점 2개로 나타낼 수 있고, 이것을 추상화 하였다. <br>
메모리 효율을 높이기 위해 꼭짓점(Point)을 별도의 구조체(RectData)에 넣어 사각형 클래스가 이를 가리키도록 해보자.

```cpp

class Point {
  public:
    Point(int x, int y);
    void SetX(int newVal);
    void SetY(int newVal);
};

struct RectData {
  Point ulhc; // 좌측 상단 
  Point lrhc; // 우측 하단
};

class Rectangle {

private:
  std::tr1::shared_ptr<RectData> pData;

public:
  // 둘 다 상수멤버 함수이다.
  Point& UpperLeft() const { return pData->ulhc; } // 좌측 상단 값 
  Point& LowerRight() const { return pData->lrhc; } // 우측 하단 값 
};

```

UpperLeft, LowerRight는 상수 멤버 함수로 선언하였다. <br>
상수 객체 함수를 통해 호출 객체가 수정되지 않도록 하기 위해서이다. 즉, 상수성을 유지하기 위해서이다. <br>
하지만 이 함수들을 반환하는 게 private 멤버인 내부 데이터에 대한 참조자이기 때문에, 다음 예시 코드처럼 사용하면 상수 객체가 참조하는 데이터가 수정 되어버린다. 

```cpp

Point coord1(0,0);
Point coord2(100,0);
const Rectangle rec(coord1, coord2);
rec.UpperLeft().setX(50); // 수정 됨

```

1. 데이터 멤버(ulhc, lrhc)를 private로 선언하더라도 **그 멤버의 참조자를 반환하는 함수** 들의 접근도에 따라 캡슐화의 정도가 정해진다. (UpperLeft,LowerRight가 public이기에 데이터 멤버의 접근도가 private라도 해당 함수들로 인해 접근도가 변경됨)
2. 어떤 객체에서 호출한 상수 멤버 함수 (Point& UpperLeft() const)의 참조자 반환 값의 실제 데이터가 해당 객체의 바깥에 저장되어 있다면 이 함수의 호출부에서 그 데이터가 수정이 가능하다.

이러한 이유로 **핸들(다른 객체에 손댈 수 있는 매개자 - 참조자, 포인터, 반복자)을 반환하는 코드는 최대한 피하자**라는 것을 알 수 있다. <br>
그럼 해당 문제를 어떻게 해결할까? <br>
상수 멤버 함수 UpperLeft,LowerRight의 반환 타입에 const를 붙여 상태 변경을 금지시키는 방법이 있다. 

```cpp

const Point& UpperLeft() const { return pData->ulhc; } // 좌측 상단 값 
const Point& LowerRight() const { return pData->lrhc; } // 우측 하단 값 

```

const를 앞에 붙여주면 핸들을 반환해도, 그 값을 변경할 수 없기 때문에 해결된다. 또한 캡슐화 문제라면 이 클래스는 의도적으로 Point를 들여다보도록 설계한 것이기 때문에, 의도적인 캡슐화 완화라고 볼 수 있지만 완벽하게 해결된 것은 아니다 또 다른 문제가 발생할 수 있기 때문이다. <br>
그럼 **무효 참조 핸들**에 대해서 알아보자. 

무효 참조 핸들이란 그 핸들이 가리키는 데이터를 따라갔을 때 해당 데이터가 없는 것이다. 

```cpp

class GUIObject();
const  Rectangle boundingBox(const GUIObject& obj);
GUIObject *pgo;
const Point *pUpperLeft = &(boundingBox(*pgo).uUpperLeft());

```

GUI 객체의 사각 테두리 영역을 Rectangle 객체로 반환하는 함수가 있다고 가정하자. boundingBox를 호출하면 임시 객체가 생성되는데, 이 임시 객체에 대해 UpperLeft가 호출되며 두 Point 객체 중 하나에 대한 참조자로 나온다. <br>
그 주소값이 pUpperLEft에 반환되는 형식이고, 이 문장이 끝날 무렵 임시 객체는 소멸되기 때문에 pUpperLeft가 가리키는 객체는 날아가고 없게된다. <br>
그렇기 때문에, **핸들을 반환하는 것***을 반드시 되도록 피하는게 좋으며, 어떻게든 문제가 발생할 수 있는 여지가 있기 때문에 operator[] 연사자와 같이 특별하게 필요한 경우가 아니면 지양하는 것이 좋다.

### 5-28 정리
  - 어떤 객체의 내부 요소에 대한 핸들(참조자, 포인터, 반복자)을 반환하는 것은 되도록 피하자.
  - 캡슐화 정도를 높이고, 상수 멤버 함수가 객체의 상수성을 유지한 채로 동작할 수 있도록 하며, 무효 참조 핸들이 생기는 경우를 최소화 시킬 수 있다.
