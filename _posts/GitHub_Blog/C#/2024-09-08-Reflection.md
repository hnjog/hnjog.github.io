---
title: "C#의 여러가지와 Reflection"
last_modified_at: "2024-09-08T15:30:00"
categories:
  - C#
tags:
  - C# 자료형
  - Getter / Setter
  - Reflection
---

## C#의 자료형
 최근 Unity와 C#에 대하여 다시 공부 중이며,<br>
 그 중 나중에 개념상 헷갈리지 않기에 몇가지 개념을 정리하려 한다<br>

 특히, '자료형'과 그에 따른 '전달 방식'이 조금 헷갈려서 잠시 정리하려 한다<br>
 1. C#의 자료형은 크게 2가지로 나뉜다<br>
   - 기본 자료형 (int, float, char 등) : stack에 저장
   - 참조 자료형(class, interface, delegate, array 등) : heap에 저장, Object 클래스를 상속<br>
 2. 참조 자료형이 함수의 매개변수로 전달될 때, '참조'의 방식으로 전달된다<br>
    즉, 원본 객체를 함수 내에서 수정할 수 있음<br>

 이걸 정리한 이유는, 아래의 코드를 볼 때<br>
 ```
 // C#

 void ChangeReference(MyClass obj)
{
    obj.Value = 42;
}
 ```

당연히 함수 외부의 obj 원본에 접근하는 것으로 인식되지만<br>
C++에서 같은 코드는 '지역변수'를 생성하여(외부 값을 복사한)<br>
그 지역변수에 접근하는 방식이기에 다소 혼란스러웠다<br>

```
// C++

 void ChangeReference(MyClass obj)
{
  // obj는 '지역변수'
  // 전달방식을 포인터 or 레퍼런스로 해야 C#과 동일
  // (MyClass* obj) (MyClass& obj)
    obj.Value = 42;
}
```

## C#의 Getter, Setter
 C# 은 클래스의 속성에 접근하기 위해<br>
 Getter와 Setter를 사용한다<br>

```
class Person
{
    private string name; // 필드

    public string Name
    {
        get { return name; } // Getter
        set { name = value; } // Setter
    }
}
```

 값을 가져올 때 Getter를, 값을 쓸때 Setter를 통해 사용함으로서<br>
 내부 필드를 직접적으로 수정하는 것을 막을 수 있다<br>


 - 여담으로 캡슐화와 관련하여 말이 조금 많은 편이다<br>
   특히 Getter로 '참조 자료형'을 반환하는 경우 '직접적인 필드'를 반환하는 것과<br>
   다를 바 없기 때문에 '정보 은닉에 부합하는가?'에 대하여 의견이 많다하더라...<br>

 - 필드 : 클래스 내부 데이터를 저장하는 변수
 - 속성 : 필드에 대한 '접근'을 제공하는 인터페이스
 - 메서드 : 객체가 수행하는 동작을 정의하는 함수
 - 특성(attribute) : 프로그램의 메타데이터를 정의<br>
   (메타데이터 : 데이터의 구조나 속성을 설명하는 정보,<br>
    클래스가 어떤 필드를 가지는지, 메서드가 어떠한 매개변수를 요구하는지 등을<br>
    가지고 있는 데이터이다)<br>

## C#의 Generic
 처음에는 C++의 '템플릿'과 비슷하다 생각하였으나 조금 다른 부분이 있어 정리하려 한다<br>
 1. 컴파일 시간에 '타입 검사'가 이루어진다<br>
 2. 런타임 시간에 실제 '인스턴스'가 생성된다<br>
    (C++의 경우 '컴파일' 때, 오류 검사와 '인스턴스' 동시에 진행하며,<br>
     각 타입별로 해당 코드가 생성되는 방식)<br>
 3. where 키워드를 통해 타입에 대한 제약을 지정할 수 있음<br>
    (특정 클래스 or 그 클래스를 상속받는 타입으로 지정한다 던가)<br>
  
 추가적으로, C#은 아래쪽에서 말할 '리플렉션' 기능을 통하여<br>
 제네릭의 타입에 대한 '런타임 타입 정보'를 더욱 유용하게 활용할 수 있다 <br>
 (C++은 컴파일 시간에 결정되기에 이러한 부분은 건들기 힘들다)<br>

```
using System;
using System.Collections.Generic;

public class Example<T>
{
    public void PrintType()
    {
        Console.WriteLine($"Type: {typeof(T)}");
    }
}

class Program
{
    static void Main()
    {
        var intExample = new Example<int>();
        intExample.PrintType(); // Type: System.Int32

        var stringExample = new Example<string>();
        stringExample.PrintType(); // Type: System.String
    }
}

```
## Reflection
 처음에는 '캐스팅' 혹은 '제네릭'과 비슷한 기능으로 추측하였으나<br>
 전혀 다른 독특한 기능이었다<br>

 요점은 '런타임 중 타입에 정보를 탐색하고 조작' 한다는 점이다<br>
 ('타입 정보'를 GetType() 이나 typeof() 등을 통해 얻어와 사용한다)<br>

 타입 정보를 통하여 '필드 접근','속성 접근','메서드 호출','특성 조회' 등이 가능하다<br>

 각각의 데이터를 ~Info 클래스로 받아올 수 있다<br>
 (특성의 경우는 IEnumerable)<br>

 예시(제네릭과 리플렉션을 이용한 Data 세팅)<br>

 ```
 // : new() - LoaderData 타입에 매개 변수 없는 생성자가 존재해야 함
private static List<LoaderData> ParseExcelDataToList<LoaderData>(string filename) where LoaderData : new()
{
    List<LoaderData> loaderDatas = new List<LoaderData>();

    string[] lines = File.ReadAllText($"{Application.dataPath}/@Resources/Data/ExcelData/{filename}Data.csv").Split("\n");

    for (int l = 1; l < lines.Length; l++)
    {
        string[] row = lines[l].Replace("\r", "").Split(',');
        if (row.Length == 0)
            continue;
        if (string.IsNullOrEmpty(row[0]))
            continue;

        // TestData 가 일단 위쪽에서 사용 중
        LoaderData loaderData = new LoaderData();

        // TestData의 모든 필드(변수 요소들)를 가져온다
        System.Reflection.FieldInfo[] fields = typeof(LoaderData).GetFields();
        // 필드를 순회하여 개체의 해당 필드를 가져온다
        for (int f = 0; f < fields.Length; f++)
        {
            // 해당 제네릭 타입의 타입 - 필드 정보를
            // 미리 가져온 이름으로 가져온다
            FieldInfo field = loaderData.GetType().GetField(fields[f].Name);
            Type type = field.FieldType;

            // 제네릭 타입이라면 ConvertList 호출하여 변환
            if (type.IsGenericType)
            {
                object value = ConvertList(row[f], type);
                // fieldInfo 이기에
                // 실제로 값을 써줄 대상(이 필드가 속한 인스턴스)에 값을 써준다
                field.SetValue(loaderData, value);
            }
            else
            {
                object value = ConvertValue(row[f], field.FieldType);
                field.SetValue(loaderData, value);
            }
        }

        loaderDatas.Add(loaderData);
    }

    return loaderDatas;
}

 ```
